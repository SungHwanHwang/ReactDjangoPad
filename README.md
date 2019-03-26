%% 리액트 & 장고 연결 절차 %%
cf) < > 내 아이템은 사용자 마음

1. Back단 - python(django) 
	# 최상단 디렉토리 생성
		1) django-react
	
	# 가상 환경 생성
		1) python -m venv <django-react-venv>
	
	# 가상 환경 실행
		1) django-react-venv\Scripts\activate
	
''' 이하 가상환경 내에서 진행 '''
	
	# 가상 환경 내 pip 업그레이드
		1) python -m pip install --upgrade pip
		
	# django 설치
		1) pip install django
		
	# backend 디렉토리 생성 및 backend project 생성
		1) mkdir <backend>
		2) cd backend
		3) django-admin startproject <djangoreactapi>
		
	# api로 호출 시킬 app 생성(post)
		1) python manage.py startapp <post>
		
	# migrate 실행
		1) python manage.py migrate
		
	# djangoreactapi/settings.py에 post를 추가
		# backend/djangoreactapi/settings.py
		...
		INSTALLED_APPS = [
			'django.contrib.admin',
			'django.contrib.auth',
			'django.contrib.contenttypes',
			'django.contrib.sessions',
			'django.contrib.messages',
			'django.contrib.staticfiles',
			
			'post', #추가
		]
	
	# 테스트 실행
		1) python manage.py runserver
	
	# post앱의 model 정의
		#backend/post/models.py
		from django.db import models

		class Post(models.Model):
			title = models.CharField(max_length=200)
			content = models.TextField()

			def __str__(self):
				"""A string representation of the model."""
				return self.title
	
	# post앱의 admin 정의
		#backend/post/admin.py
		from django.contrib import admin

		from .models import Post

		admin.site.register(Post)
				
	# migration을 통한 superuser 생성(아이디, 비번)
		1) python manage.py makemigrations
		2) python manage.py migrate
		3) python manage.py createsuperuser
			- 아이디 입력
			- 이메일 입력
			- 비번 입력
			- 비번 확인
			
	# 서버 실행 후 테스트 진행 
		1) python manage.py runserver
		
	# django-rest-framework 구성 
		1) pip install djangorestframework
		
	# setting.py에 새로 추가된 djangorestframework 추가
		#backend/djangoreactapi/settings.py
		...
		INSTALLED_APPS = [
			'django.contrib.admin',
			'django.contrib.auth',
			'django.contrib.contenttypes',
			'django.contrib.sessions',
			'django.contrib.messages',
			'django.contrib.staticfiles',
			
			'post',
			'rest_framework', #추가
			
		]
		...
		
		#이하 추가
		REST_FRAMEWORK = {
			'DEFAULT_PERMISSION_CLASSES': [
				'rest_framework.permissions.AllowAny',
			]
		}
		
	# JSON형식으로의 직렬화(serializers)를 위한 serializers.py 파일 생성
		#backend/post/serializers.py
		from rest_framework import serializers
		from .models import Post

		class PostSerializer(serializers.ModelSerializer):
			class Meta:
				fields = (
					'id',
					'title',
					'content',
				)
				model = Post
		
	# post의 views.py 작성
		#backend/post/views.py
		from django.shortcuts import render
		from rest_framework import generics

		from .models import Post
		from .serializers import PostSerializer

		class ListPost(generics.ListCreateAPIView):
			queryset = Post.objects.all()
			serializer_class = PostSerializer

		class DetailPost(generics.RetrieveUpdateDestroyAPIView):
			queryset = Post.objects.all()
			serializer_class = PostSerializer
	
	# post의 urls.py 작성
		#backend/post/urls.py
		from django.urls import path

		from . import views

		urlpatterns = [
			path('', views.ListPost.as_view()),
			path('<int:pk>/', views.DetailPost.as_view()),
		]
		
	# 루트 디렉토리의 urls.py 내용 변경
		#backend/djangoreactapi/urls.py
		from django.contrib import admin
		from django.urls import path, include

		urlpatterns = [
			path('admin/', admin.site.urls),
			path('api/', include('post.urls')),
		]
		
	# 리액트의 script에 접근하기 위한 패키지 설치
		1) pip install django-cors-headers
		2)
		#backend/djangoreactapi/settings.py
		...
		INSTALLED_APPS = [
			'django.contrib.admin',
			'django.contrib.auth',
			'django.contrib.contenttypes',
			'django.contrib.sessions',
			'django.contrib.messages',
			'django.contrib.staticfiles',
			
			'post',
			'rest_framework',
			'corsheaders', # 추가
		]
		...
		MIDDLEWARE = [
			'corsheaders.middleware.CorsMiddleware',     # 추가
			'django.middleware.common.CommonMiddleware', # 추가
			'django.middleware.security.SecurityMiddleware',
			'django.contrib.sessions.middleware.SessionMiddleware',
			'django.middleware.common.CommonMiddleware',
			'django.middleware.csrf.CsrfViewMiddleware',
			'django.contrib.auth.middleware.AuthenticationMiddleware',
			'django.contrib.messages.middleware.MessageMiddleware',
			'django.middleware.clickjacking.XFrameOptionsMiddleware',
		]
		...
		CORS_ORIGIN_WHITELIST = (
			'localhost:3000/'
		)
		#script안에서의 리소스 요청을 허용할 도메인 추가
		
''' 이하 리액트 내에서 진행 '''		
	# create-react-app 을 통해 frontend 생성
		1) django-react\ 경로 내 create-react-app frontend
		2) cd frontend
		3) yarn start
	
	# django 서버 실행
		1) python manage.py runserver
		
	# react 내 app.js 파일 수정
		#frontend/src/app.js
		import React, { Component } from 'react';

		class App extends Component {
			state = {
				posts: []
			};

			async componentDidMount() {
				try {
					const res = await fetch('http://127.0.0.1:8000/api/');
					const posts = await res.json();
					this.setState({
						posts
					});
				} catch (e) {
					console.log(e);
				}
			}

			render() {
				return (
					<div>
						{this.state.posts.map(item => (
							<div key={item.id}>
								<h1>{item.title}</h1>
								<span>{item.content}</span>
							</div>
						))}
					</div>
				);
			}
		}

		export default App;
