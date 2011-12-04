---
layout: post
title: Multiple database validation in Django
published: true
tags:
  - django
  - router
  - replication
  - master-slave
  - mysql
  - models
  - validation
---

**Code in the post tested with django1.2, and 1.3.1**

When you start django server with [runfcgi] it will perform
a [model validation] which requires your [django.db.DEFAULT_DB_ALIAS]
(i.e. 'default') to be available for reading.

The problem kicks in when you have multiple databases: master and slaves.

Assume the following configuration:
{% highlight python %}
DATABASES = { 
    # HACK: force runfcgi start when default db isn't available
    'default': {
        'ENGINE': 'django.db.backends.dummy',
    },  
    'master': {
        'ENGINE': 'django.db.backends.mysql',
        'OPTIONS': {
            'read_default_file': '/place/mysql/.my_master.cnf',
            'init_command': 'SET storage_engine=INNODB',
        },  
    },  
    'slave': {
        'ENGINE': 'django.db.backends.mysql',
        'OPTIONS': {
            'read_default_file': '/place/mysql/.my_slave.cnf',
            'init_command': 'SET storage_engine=INNODB',
        },  
    },  
    # for test purposes
    'fake_slave': {
        'ENGINE': 'django.db.backends.mysql',
        'OPTIONS': {
            'read_default_file': '/place/mysql/.my_fake_slave.cnf',
            'init_command': 'SET storage_engine=INNODB',
        },  
    },  
}
{% endhighlight %}
Here, you **have to** use *django.backends.dummy* to force *runfcgi*
start even if your *'default'* database isn't available.

Here is router I use, is_alive taken from [django_replicated]:


{% highlight python %}
from datetime import datetime, timedelta

from django.conf import settings

class MasterSlaveRouter(object):
    """A router that sets up a simple master/slave configuration"""
    def __init__(self):
        from django.db import connections
        self.connections = connections
        self.downtime = timedelta(seconds=getattr(settings, 'DATABASE_DOWNTIME', 60))
        self.dead_slaves = {}

    def is_alive(self, slave):
        death_time = self.dead_slaves.get(slave)
        if death_time:
            if death_time + self.downtime > datetime.now():
                return False
            else:
                del self.dead_slaves[slave]
        db = self.connections[slave]
        try:
            if db.connection is not None and hasattr(db.connection, 'ping'):
                db.connection.ping()
            else:
                db.cursor()
            return True
        except StandardError:
            self.dead_slaves[slave] = datetime.now()
            return False

    def db_for_read(self, model, **hints):
        "Point all read operations to a random slave"
        for db in ('slave', 'fake_slave'):
            if self.is_alive(db):
                return db
        return self.db_for_write(model, **hints)

    def db_for_write(self, model, **hints):
        "Point all write operations to the master"
        return 'master'

def get_alive_db():
    r = MasterSlaveRouter()
    return r.db_for_read(None)
{% endhighlight %}

As you can see, model validation in django is broken!
It only tries to validate 'default' database, and **will not start** if it is unavailable.

Here is the patch for django1.3.1 to fix this behaviour:

{% highlight diff %}
Index: django/core/management/base.py
===================================================================
--- django/core/management/base.py    (revision 17142)
+++ django/core/management/base.py    (working copy)
@@ -241,18 +241,33 @@

         """
         from django.core.management.validation import get_validation_errors
+        from django.db import connections
         try:
             from cStringIO import StringIO
         except ImportError:
             from StringIO import StringIO
         s = StringIO()
-        num_errors = get_validation_errors(s, app)
+        valid_databases = 0
+        num_errors = 0
+        for db_alias in connections:
+            conn = connections[db_alias]
+            try:
+                num_errors += get_validation_errors(s, app, conn)
+            except Exception as e:
+                num_errors += 1
+                err_text = "Exception while validating '%s' db alias:\n%s\n" % (db_alias, e)
+                s.write(err_text)
+            else:
+                valid_databases += 1
         if num_errors:
             s.seek(0)
             error_text = s.read()
-            raise CommandError("One or more models did not validate:\n%s" % error_text)
-        if display_num_errors:
-            self.stdout.write("%s error%s found\n" % (num_errors, num_errors != 1 and 's' or ''))
+            if display_num_errors:
+                self.stdout.write("%s error%s found\n" % (num_errors, num_errors != 1 and 's' or ''))
+            if not valid_databases:
+                raise CommandError("One or more models did not validate in all connections:\n%s" % error_text)
+            else:
+                self.stderr.write("Some connection did not validate:\n%s" % error_text)

     def handle(self, *args, **options):
         """
Index: django/core/management/validation.py
===================================================================
--- django/core/management/validation.py    (revision 17142)
+++ django/core/management/validation.py    (working copy)
@@ -18,18 +18,22 @@
         self.errors.append((context, error))
         self.outfile.write(self.style.ERROR("%s: %s\n" % (context, error)))

-def get_validation_errors(outfile, app=None):
+def get_validation_errors(outfile, app=None, connection=None):
     """
     Validates all models that are part of the specified app. If no app name is provided,
     validates all models of all installed apps. Writes errors, if any, to outfile.
     Returns number of errors.
+    If no connecton is specified, the django.db.DEFAULT_DB_ALIAS will be used.
     """
     from django.conf import settings
-    from django.db import models, connection
+    from django.db import models
     from django.db.models.loading import get_app_errors
     from django.db.models.fields.related import RelatedObject
     from django.db.models.deletion import SET_NULL, SET_DEFAULT

+    if connection is None:
+        from django.db import connection
+
     e = ModelErrorCollection(outfile)

     for (app_name, error) in get_app_errors().items():
{% endhighlight %}


[django_replicated]: https://github.com/isagalaev/django_replicated
[runfcgi]: https://docs.djangoproject.com/en/1.3/ref/django-admin/#runfcgi-options
[model validation]: https://code.djangoproject.com/browser/django/tags/releases/1.3.1/django/core/management/validation.py#L28 
[django.db.DEFAULT_DB_ALIAS]: https://code.djangoproject.com/browser/django/tags/releases/1.3.1/django/db/utils.py#L9
