# Setup Steps
1. Run `docker-compose build` to build image
2. Run `docker-compose run web pipenv run django-admin startproject app .` to build project
3. Run `docker-compose restart`
4. Update `app/app/setting.py` for database connection
```python 
  DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'postgres',
        'USER': 'postgres',
        'PASSWORD': 'postgres',
        'HOST': 'db',
        'PORT': '5432',
     }
   }
```
5. Run db migration `docker-compose run web pipenv run python manage.py migrate`
6. Setup super user `docker-compose run web pipenv run python manage.py createsuperuser`
7. Collection static files `docker-compose run web pipenv run python manage.py manage.py collectstatic`
8. Update  `app/app/setting.py` for allowed host 
```python  
ALLOWED_HOSTS = ['ip of host']
```
9. Update `app/app/setting.py` for celery
```python
  CELERY_BROKER_URL = 'redis://redis:6379/0'
  CELERY_RESULT_BACKEND = 'redis://redis:6379/0'
  CELERY_ACCEPT_CONTENT = ['application/json']
  CELERY_TASK_SERIALIZER = 'json'
  CELERY_RESULT_SERIALIZER = 'json'
  CELERY_IMPORTS = ['app.tasks',]
  from celery.schedules import crontab
  CELERY_BEAT_SCHEDULE = {
    'printHello': {
        'task': 'core.tasks.printHello',
        #'schedule': 5.0
    },
  }
```
10. Add `app/app/celery.py` for registering celery app and tasks discovery
```python  
  from __future__ import absolute_import, unicode_literals
  import os
  from celery import Celery
  from celery import shared_task

  # set the default Django settings module for the 'celery' program.
  os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'app.settings')

  app = Celery('app')

  # Using a string here means the worker doesn't have to serialize
  # the configuration object to child processes.
  # - namespace='CELERY' means all celery-related configuration keys
  #   should have a `CELERY_` prefix.
  app.config_from_object('django.conf:settings', namespace='CELERY')

  # Load task modules from all registered Django app configs.
  app.autodiscover_tasks()

  @app.task(bind=True)
  def debug_task(self):
      print('Request: {0!r}'.format(self.request))
```
11. Add `/app/app/tasks,py` for demo schedule task 
```python   
  from __future__ import absolute_import, unicode_literals
  from celery import task

  @task()
  def printHello():
    return "Hello"
```
12. Run `docker-compose down` and `docker-compose up -d` to recreate containers
13. Go to 
    - Home: http://localhost:8000
    - Admin panel: http://localhost:8000/admin
    - Flower Celery monitoring tools: http://localhost:8888
    - PGadmin: http://localhost:5050




