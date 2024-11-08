Step 1: Setup
Create a virtual environment:

bash
python -m venv venv
source venv/bin/activate   # On Windows use `venv\Scripts\activate`
Install necessary packages:

bash
pip install django djangorestframework djangorestframework-simplejwt psycopg2-binary
Step 2: Create Django Projects
Task Management Microservice
Create the Django project and app:

bash
django-admin startproject task_management
cd task_management
django-admin startapp tasks
Define the Task model in tasks/models.py:

python
from django.db import models
from django.contrib.auth.models import User

class Task(models.Model):
    STATUS_CHOICES = [
        ('To Do', 'To Do'),
        ('In Progress', 'In Progress'),
        ('Completed', 'Completed'),
    ]

    title = models.CharField(max_length=200)
    description = models.TextField()
    status = models.CharField(choices=STATUS_CHOICES, max_length=20)
    due_date = models.DateField()
    assigned_to = models.ForeignKey(User, on_delete=models.SET_NULL, null=True)

    def __str__(self):
        return self.title
Create serializers and views for CRUD operations in tasks/serializers.py and tasks/views.py:

Serializers:

python
from rest_framework import serializers
from .models import Task

class TaskSerializer(serializers.ModelSerializer):
    class Meta:
        model = Task
        fields = '__all__'
Views:

python
from rest_framework import viewsets
from .models import Task
from .serializers import TaskSerializer

class TaskViewSet(viewsets.ModelViewSet):
    queryset = Task.objects.all()
    serializer_class = TaskSerializer
Add URL routing in task_management/urls.py:

python
from django.contrib import admin
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from tasks.views import TaskViewSet

router = DefaultRouter()
router.register(r'tasks', TaskViewSet)

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include(router.urls)),
]
User Management Microservice
Create the Django project and app:

bash
django-admin startproject user_management
cd user_management
django-admin startapp users
Set up JWT Authentication in user_management/settings.py:

python
INSTALLED_APPS = [
    ...
    'rest_framework',
    'rest_framework_simplejwt',
    'users',
]

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ],
}
Define custom user roles in users/models.py:

python
from django.contrib.auth.models import AbstractUser

class CustomUser(AbstractUser):
    is_admin = models.BooleanField(default=False)
    is_regular_user = models.BooleanField(default=True)
Create views and serializers for user registration and login in users/views.py and users/serializers.py:

Serializers:

python
from rest_framework import serializers
from django.contrib.auth.models import User

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ('username', 'password', 'is_admin', 'is_regular_user')
    extra_kwargs = {'password': {'write_only': True}}

    def create(self, validated_data):
        user = User(
            username=validated_data['username'],
            is_admin=validated_data.get('is_admin', False),
            is_regular_user=validated_data.get('is_regular_user', True)
        )
        user.set_password(validated_data['password'])
        user.save()
        return user
Views:

python
from rest_framework import generics
from .serializers import UserSerializer
from django.contrib.auth.models import User

class UserCreate(generics.CreateAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer
Add URL routing in user_management/urls.py:

python
from django.contrib import admin
from django.urls import path, include
from users.views import UserCreate
from rest_framework_simplejwt.views import TokenObtainPairView, TokenRefreshView

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/users/', UserCreate.as_view(), name='user-create'),
    path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
]
Step 3: Implement Notification System
Create Django signals for task updates in tasks/signals.py:

python
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.core.mail import send_mail
from .models import Task

@receiver(post_save, sender=Task)
def send_task_update_notification(sender, instance, **kwargs):
    if instance.status != 'To Do':
        send_mail(
            'Task Update',
            f'The task "{instance.title}" is now {instance.status}.',
            'from@example.com',
            [instance.assigned_to.email],
            fail_silently=False,
        )
Connect the signals in tasks/apps.py:

python
from django.apps import AppConfig

class TasksConfig(AppConfig):
    name = 'tasks'

    def ready(self):
        import tasks.signals
Step 4: Configure PostgreSQL Database
Update settings.py to use PostgreSQL:

python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'your_db_name',
        'USER': 'your_db_user',
        'PASSWORD': 'your_db_password',
        'HOST': 'localhost',
        'PORT': '',
    }
}
Step 5: Deployment
Create a Procfile for Heroku:

Procfile
web: gunicorn task_management.wsgi
Install Gunicorn and Psycopg2:

bash
pip install gunicorn psycopg2-binary
Push to Heroku:

bash
heroku create your-app-name
git add .
git commit -m "Deploy to Heroku"
git push heroku master
Set environment variables on Heroku:

bash
heroku config:set DJANGO_SECRET_KEY='your_secret_key'
heroku config:set DATABASE_URL='your_database_url'
GitHub Repository
Initialize and Push to GitHub:

bash
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/yourusername/your-repository-name.git
git push -u origin masters
