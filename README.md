# Django Rest Framework (DRF) — Complete Cheat Sheet

A beginner-friendly guide + commented code reference

---

## **📌 Table of Contents**

1. [Introduction to Django Rest Framework](#introduction-to-django-rest-framework)
2. [Installing DRF](#installing-drf)
3. [Basic APIView Example](#basic-apiview-example)

   - [views.py (with full comments)](#viewspy-with-full-comments)
   - [urls.py (with full comments)](#urlspy-with-full-comments)

4. [Serializers](#serializers)

   - [Creating the app](#creating-the-app)
   - [Model](#model)
   - [Serializer](#serializer)
   - [Connecting serializer to views](#connecting-serializer-to-views)

5. [GET Method With Serializer](#get-method-with-serializer)
6. [Authentication](#authentication)

   - [Permissions](#permissions)
   - [Settings Configuration](#settings-configuration)
   - [Token Creation](#token-creation)
   - [Postman Authentication](#postman-authentication)

7. [External Framework Token Generation](#external-framework-token-generation)
8. [Postman Token Request Example](#postman-token-request-example)

---

# Introduction to Django Rest Framework

Django Rest Framework (**DRF**) is a powerful and flexible library that makes it easier to build RESTful APIs inside Django projects.
It reduces boilerplate code and provides tools such as:

- Serializers
- Authentication
- Class-based API views
- Browsable API interface
- Permissions
- Validations

---

# Installing DRF

```bash
pip install djangorestframework
```

After installation, add it to Django settings (explained later).

---

# Basic APIView Example

You must create or edit the `views.py` file inside your project or app.

---

## **views.py (with full comments)**

```python
'''
Importing render (not used here but useful for templates)
Importing APIView: allows us to create class-based API endpoints
Importing Response: used to return JSON data to the client
'''
from django.shortcuts import render
from rest_framework.views import APIView
from rest_framework.response import Response

class TestView(APIView):
    # Handles GET requests to the endpoint
    def get(self, request, *args, **kwargs):
        data = {
            'username': 'admin',   # sample data we want to return
            'years_active': 10     # another sample field
        }
        return Response(data)       # returns data in JSON format
```

---

## **urls.py (with full comments)**

```python
from django.contrib import admin                   # admin site
from django.urls import path, include              # path() to define routes
from .views import TestView                        # importing our TestView

urlpatterns = [
    path('admin/', admin.site.urls),               # Django admin route

    # Enables DRF's built-in login/logout views for browsable API
    path('api-auth', include('rest_framework.urls')),

    # Root path of project → our TestView
    # .as_view() is required because TestView is a class
    path('', TestView.as_view(), name='test'),
]
```

---

# Serializers

Serializers convert Django model instances ↔ JSON format.
They work similarly to Django ModelForms.

They help with:

- validation
- deserialization (input → Python)
- serialization (Python → JSON)

---

## Creating the app

```bash
python manage.py startapp drfapp
```

Register the app in `settings.py`.

---

## Model

Create a model inside `drfapp/models.py`.

```python
from django.db import models   # importing required base class

class Student(models.Model):
    name = models.CharField(max_length=100)    # text field for student's name
    age = models.IntegerField()                # integer field for age
    description = models.TextField()           # long text field
    date_enrolled = models.DateTimeField(auto_now=True)
    # auto_now=True → updates timestamp on each save

    def __str__(self):                         # readable representation
        return self.name
```

---

## Migrate the model

```bash
python manage.py makemigrations
python manage.py migrate
```

---

## Serializer

Create a file `drfapp/serializer.py`:

```python
from rest_framework import serializers          # importing Serializer classes
from .models import Student                      # importing the model

class StudentSerializer(serializers.ModelSerializer):
    class Meta:
        model = Student                         # telling serializer which model to use
        fields = ('name', 'age')                # fields we want to expose in API
```

> **Note:** We can choose which fields to send back in JSON even if the model contains more fields.

---

## Connecting Serializer to Views

Update `views.py`:

```python
from django.shortcuts import render
from rest_framework.views import APIView
from rest_framework.response import Response

# Importing serializer and model
from drfapp.serializers import StudentSerializer
from drfapp.models import Student

class TestView(APIView):
    def get(self, request, *args, **kwargs):     # GET request
        data = {
            'username': 'admin',
            'years_active': 10,
        }
        return Response(data)

    def post(self, request, *args, **kwargs):    # POST request handler
        # Creating serializer instance with incoming JSON data
        serializer = StudentSerializer(data=request.data)

        # Validate incoming data against fields and model rules
        if serializer.is_valid():
            serializer.save()                   # creates Student object
            return Response(serializer.data)    # return validated data

        return Response(serializer.errors)       # return validation errors
```

---

# GET Method With Serializer

To return all students in database:

```python
def get(self, request, *args, **kwargs):
    qs = Student.objects.all()                   # query all students
    serializer = StudentSerializer(qs, many=True)
    # many=True → tells serializer to handle a list of objects

    return Response(serializer.data)             # return serialized list
```

---

# Authentication

Authentication protects API endpoints from unauthorized users.

---

## Permissions

Inside your `views.py`:

```python
from rest_framework.permissions import IsAuthenticated  # import permission class

# Restrict access to authenticated users only
permission_classes = (IsAuthenticated, )

# Leaving this empty would mean "open to the public"
```

---

# Settings Configuration

Add DRF and token auth to your installed apps:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    'rest_framework',                # Django Rest Framework
    'rest_framework.authtoken',      # Token authentication package
    'drfapp',                        # your app
]
```

At bottom of settings:

```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication'
    ]
}
```

---

## Migrate Again

```bash
python manage.py makemigrations
python manage.py migrate
```

---

## (Optional) Flush the database

```bash
python manage.py flush
```

---

## Create a Superuser

```bash
python manage.py createsuperuser
```

---

## Create Token for User

```bash
python manage.py drf_create_token admin
```

This generates an auth token that can be used in Postman.

---

# Postman Authentication

1. Open Postman
2. Go to **Authorization** tab
3. Choose **API Key**
4. Settings:

   ```
   key = authorization
   value = Token <paste token here>
   ```

The request will now include auth token.

---

# External Framework Token Generation

DRF includes an API endpoint to automatically generate tokens by sending username + password.

Add this to your `urls.py`:

```python
from django.contrib import admin
from django.urls import path, include
from .views import TestView
from rest_framework.authtoken.views import obtain_auth_token  # built-in token generator

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api-auth', include('rest_framework.urls')),
    path('', TestView.as_view(), name='test'),

    # POST username and password → returns auth token
    path('api/token/', obtain_auth_token, name='obtain')
]
```

---

# Postman Token Request Example

**Method:** `POST`
**URL:** `/api/token/`

**Body → raw → JSON**

```json
{
  "username": "admin",
  "password": ""
}
```

Response will contain:

```json
{
  "token": "<your_token_here>"
}
```

---
