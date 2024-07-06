# portfolio-web
Backend (Django with MySQL)
1. Set up a Django project with a MySQL database
Install Django and MySQL client:
sh
Copy code
pip install django mysqlclient
Create a new Django project:
sh
Copy code
django-admin startproject portfolio
cd portfolio
Configure MySQL database in settings.py:
python
Copy code
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'portfolio_db',
        'USER': 'your_username',
        'PASSWORD': 'your_password',
        'HOST': 'localhost',
        'PORT': '3306',
    }
}
Create the database:
sql
Copy code
CREATE DATABASE portfolio_db;
2. Create models for User and Project
Create a new app:
sh
Copy code
python manage.py startapp cms
Define the models in cms/models.py:
python
Copy code
from django.contrib.auth.models import AbstractUser
from django.db import models

class User(AbstractUser):
    pass

class Project(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    title = models.CharField(max_length=255)
    description = models.TextField()
    image = models.ImageField(upload_to='project_images/')
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    def _str_(self):
        return self.title
3. Implement user authentication
Add rest_framework and rest_framework.authtoken to INSTALLED_APPS in settings.py:
python
Copy code
INSTALLED_APPS = [
    ...
    'rest_framework',
    'rest_framework.authtoken',
    'cms',
]
Configure Django REST framework in settings.py:
python
Copy code
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
}
4. Develop a simple CMS with API endpoints
Create serializers in cms/serializers.py:

python
Copy code
from rest_framework import serializers
from .models import User, Project

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'username', 'email']

class ProjectSerializer(serializers.ModelSerializer):
    class Meta:
        model = Project
        fields = ['id', 'title', 'description', 'image', 'created_at', 'updated_at']
Create views in cms/views.py:

python
Copy code
from rest_framework import viewsets
from rest_framework.permissions import IsAuthenticated
from .models import Project
from .serializers import ProjectSerializer

class ProjectViewSet(viewsets.ModelViewSet):
    queryset = Project.objects.all()
    serializer_class = ProjectSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return self.queryset.filter(user=self.request.user)

    def perform_create(self, serializer):
        serializer.save(user=self.request.user)
Add URL routes in cms/urls.py:

python
Copy code
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from .views import ProjectViewSet

router = DefaultRouter()
router.register(r'projects', ProjectViewSet)

urlpatterns = [
    path('', include(router.urls)),
]
Include CMS URLs in the project's urls.py:

python
Copy code
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('cms.urls')),
]
5. Migrate and run the server
Apply migrations:
sh
Copy code
python manage.py makemigrations
python manage.py migrate
Create a superuser for admin access:
sh
Copy code
python manage.py createsuperuser
Run the development server:
sh
Copy code
python manage.py runserver
Frontend (Next.js)
1. Set up a Next.js project
Create a new Next.js project:
sh
Copy code
npx create-next-app@latest portfolio-frontend
cd portfolio-frontend
Install necessary packages:
sh
Copy code
npm install axios react-hook-form react-redux @reduxjs/toolkit next-auth
2. Create pages
User registration and login pages:
Create pages/register.js, pages/login.js
Portfolio pages:
Create pages/index.js for the portfolio list and pages/project/[id].js for individual project details.
Project management pages:
Create pages/project/create.js and pages/project/edit/[id].js.
3. Implement state management
Set up Redux for state management in store.js:

javascript
Copy code
import { configureStore } from '@reduxjs/toolkit';
import userReducer from './features/userSlice';
import projectReducer from './features/projectSlice';

export const store = configureStore({
    reducer: {
        user: userReducer,
        project: projectReducer,
    },
});
Create user slice in features/userSlice.js:

javascript
Copy code
import { createSlice } from '@reduxjs/toolkit';

export const userSlice = createSlice({
    name: 'user',
    initialState: {
        user: null,
        token: null,
    },
    reducers: {
        setUser: (state, action) => {
            state.user = action.payload.user;
            state.token = action.payload.token;
        },
        logout: (state) => {
            state.user = null;
            state.token = null;
        },
    },
});

export const { setUser, logout } = userSlice.actions;

export default userSlice.reducer;
Create project slice in features/projectSlice.js:

javascript
Copy code
import { createSlice } from '@reduxjs/toolkit';

export const projectSlice = createSlice({
    name: 'project',
    initialState: {
        projects: [],
        currentProject: null,
    },
    reducers: {
        setProjects: (state, action) => {
            state.projects = action.payload;
        },
        setCurrentProject: (state, action) => {
            state.currentProject = action.payload;
        },
    },
});

export const { setProjects, setCurrentProject } = projectSlice.actions;

export default projectSlice.reducer;
4. Integrate API endpoints
Use axios to interact with the backend API:
javascript
Copy code
import axios from 'axios';

const API = axios.create({ baseURL: 'http://localhost:8000/api' });

API.interceptors.request.use((req) => {
    if (localStorage.getItem('token')) {
        req.headers.Authorization = Token ${localStorage.getItem('token')};
    }
    return req;
});

export const registerUser = (userData) => API.post('/register/', userData);
export const loginUser = (userData) => API.post('/login/', userData);
export const fetchProjects = () => API.get('/projects/');
export const createProject = (projectData) => API.post('/projects/', projectData);
export const updateProject = (id, projectData) => API.put(/projects/${id}/, projectData);
export const deleteProject = (id) => API.delete(/projects/${id}/);
