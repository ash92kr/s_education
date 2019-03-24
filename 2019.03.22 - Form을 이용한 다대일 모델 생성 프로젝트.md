# 2019.03.22





zzu.li/08_seed

기본 - Movie CRUD, Score CRD

선택 - 종합평점 구하기(annotate) - 평점 0 ~ 5점(widget 활용)

뷰함수 - index, create, detail, update, delete, scores_create, scores_delete

어제 배운 개념 활용하기(같은 form을 edit과 new가 나눠 가짐)

```
+ 평점 없을 때 None 출력 없애기
+ 종합평점 소수점 첫째자리까지 출력
+ 상황에 따른 링크 action 변화
```



```
project_08
190322 | json data seed를 이용한 Djanog 1:N 모델 관계 구현 + Form class 사용
```





### 1. DB 설계



```bash
mkdir 190322_pjt08
cd 190322_pjt08

pyenv local django-venv
django-admin startproject crud .
python manage.py startapp movies
```



* settings.py

```python
ALLOWED_HOSTS = ['*']

INSTALLED_APPS = [
    'movies.apps.MoviesConfig',
    
LANGUAGE_CODE = 'ko-kr'

TIME_ZONE = 'Asia/Seoul'

USE_TZ = False
```



* models.py

```python
from django.db import models

# Create your models here.
class Genre(models.Model):
    name = models.CharField(max_length=30)
    
    def __str__(self):
        return self.name
        
class Movie(models.Model):
    title = models.CharField(max_length=50)
    audience = models.IntegerField()
    poster_url = models.TextField()
    description = models.TextField()
    genre = models.ForeignKey(Genre, on_delete=models.CASCADE)
    
    def __str__(self):
        return f'{self.title}, {self.audience}, {self.poster_url}, {self.description}'
        
class Score(models.Model):
    content = models.CharField(max_length=200)
    score = models.IntegerField()
    movie = models.ForeignKey(Movie, on_delete=models.CASCADE)
    
    def __str__(self):
        return f'{self.content}, {self.score}'
```



```bash
python manage.py makemigrations
python manage.py migrate
```



### 2. seed data 넣기



```bash
# movies 앱 폴더 아래에 fixtures 폴더를 만들고 movie.json과 genre.json 파일을 넣는다
# https://twpower.github.io/20-how-to-use-fixture-in-django 참고

python manage.py loaddata genre.json
python manage.py loaddata movie.json
```



* admin.py

```python
from django.contrib import admin
from .models import Genre, Movie, Score

# Register your models here.
class GenreAdmin(admin.ModelAdmin):
    list_display = ('name',)
    
admin.site.register(Genre, GenreAdmin)

class MovieAdmin(admin.ModelAdmin):
    list_display = ('title', 'audience', 'poster_url', 'description', 'genre_id',)
    
admin.site.register(Movie, MovieAdmin)

class ScoreAdmin(admin.ModelAdmin):
    list_display = ('content', 'score', 'movie_id',)
    
admin.site.register(Score, ScoreAdmin)
```



```bash
python manage.py createsuperuser
```



### 3. Movie CRUD



* 프로젝트 폴더 urls.py

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('movies/', include('movies.urls')),
    path('admin/', admin.site.urls),
]
```



* Index

```python
# views.py
from django.shortcuts import render
from .models import Genre, Movie, Score

# Create your views here.
def index(request):
    movies = Movie.objects.order_by('-pk')
    context = {
        'movies': movies,
    }
    return render(request, 'index.html', context)
```



```python
# urls.py
from django.urls import path
from . import views

app_name = 'movies'

urlpatterns = [
    path('', views.index, name='index'),
]
```



```django
<!--base.html-->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css" integrity="sha384-ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T" crossorigin="anonymous">
    <title>{% block title %} {% endblock %}</title>
</head>
<body>
    {% block body %}
    
    {% endblock %}
    <script src="https://code.jquery.com/jquery-3.3.1.slim.min.js" integrity="sha384-q8i/X+965DzO0rT7abK41JStQIAqVgRVzpbzo5smXKp4YfRvH+8abtTE1Pi6jizo" crossorigin="anonymous"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.7/umd/popper.min.js" integrity="sha384-UO2eT0CpHqdSJQ6hJty5KVphtPhzWj9WO1clHTMGa3JDZwrnQq4sF86dIHNDz0W1" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js" integrity="sha384-JjSmVgyd0p3pXB1rRibZUAYoIIy6OrQ6VrjIEaFf/nJGzIxFDsf4x0xIM+B07jRM" crossorigin="anonymous"></script>
</body>
</html>
```



```django
<!--index.html-->
{% extends 'movies/base.html' %}

{% block title %} Index {% endblock %}

{% block body %}
    <div class="container">
        
        <h1>영화목록</h1>
        <a href="#">[NEW - 영화 입력]</a>
        <hr>
        
        {% for movie in movies %}
            <p><a href="#">{{ movie.title }}</a></p>
            <img src="{{ movie.poster_url }}" alt="{{ movie.poster_url }}"></img>
            <hr>
        {% endfor %}

    </div>
{% endblock %}
```



* CREATE

```python
# views.py
from django.shortcuts import render, redirect

def create(request):
    if request.method == "POST":
        title = request.POST.get('title')
        audience = request.POST.get('audience')
        poster_url = request.POST.get('poster_url')
        description = request.POST.get('description')
        movie = Movie(title=title, audience=audience, poster_url=poster_url, description=description)
        movie.save()
        return redirect('movies:index')
    else:
        return render(request, 'movies/form.html')
```



```python
# urls.py
    path('new/', views.create, name='create'),
```



```django
<!--form.html-->
{% extends 'movies/base.html' %}

{% block title %} Form {% endblock %}

{% block body %}

    <h1>NEW Movie 입력</h1>
    
    <form method="POST">
        {% csrf_token %}
        <input type="text" name="title" id="title"/>
        <input type="number" name="audience" id="audience"/>
        <input type="text" name="poster_url" id="poster_url"/>
        <textarea name="description" id="description"></textarea>
        <input type="submit" value="입력하기"/>
    </form>


{% endblock %}
```



```django
<!--index.html-->
		<a href="{% url 'movies:create' %}">[NEW - 영화 입력]</a>
```



* DETAIL

```python
# views.py
def detail(request, movie_pk):
    movie = Movie.objects.get(pk=movie_pk)
    context = {
        'movie': movie
    }
    return render(request, 'movies/detail.html', context)
```



```python
# urls.py
	path('<int:movie_pk>/', views.detail, name='detail'),
```



```django
<!--detail.html-->
{% extends 'movies/base.html' %}

{% block title %} DETAIL {% endblock %}

{% block body %}

    <div class="container">
        
        <p>영화 제목 : {{ movie.title }}</p>
        <p>관객수 : {{ movie.audience }}</p>
        <p>포스터 : <img src="{{ movie.poster_url }}" alt="{{ movie.title }}포스터"></img></p>
        <p>설명 : {{ movie.description }}</p>
        <p>장르 : {{ movie.genre.name }}</p>   <!--movie 안의 genre_id를 가지고 부름-->
        <hr>
        <a href="{% url 'movies:index' %}">[뒤로]</a>
        
    </div>

{% endblock %}
```



```django
<!--index.html-->  <!--html에서는 .pk-->
            <p><a href="{% url 'movies:detail' movie.pk %}">{{ movie.title }}</a></p>
```



* DELETE

```python
# views.py
def delete(request, movie_pk):
    movie = Movie.objects.get(pk=movie_pk)
    if request.method == 'POST':
        movie.delete()
        return redirect('movies:index')
    else:
        return redirect('movies:detail', movie_pk)
```



```django
# urls.py
	path('<int:movie_pk>/delete', views.delete, name='delete'),
```



```django
# detail.html
        <form action="{% url 'movies:delete' movie.pk %}" method="POST">
            {% csrf_token %}
            <input type="submit" value="삭제"/>
        </form>
```



* EDIT - form.html로 create를 공유해 template은 생략

```python
# views.py
def edit(request, movie_pk):
    movie = Movie.objects.get(pk=movie_pk)
    if request.method == "POST":
        movie.title = request.POST.get('title')
        movie.audience = request.POST.get('audience')
        movie.poster_url = request.POST.get('poster_url')
        movie.description = request.POST.get('description')
        movie.genre_id = request.POST.get('genre')
        movie.save()
        return redirect('movies:detail', movie_pk)
    else:
        genres = Genre.objects.all()
        context = {
            'movie': movie,
            'genres': genres,
        }
        return redirect('movies:')
```



```django
# urls.py
	path('<int:movie_pk>/edit', views.edit, name='edit'),
```





### 4. Form 적용하기



* forms.py

```python
from django import forms
from .models import Genre, Movie, Score

class MovieForm(forms.ModelForm):
    class Meta:
        model = Movie
        fields = '__all__'
        widgets = {
            'title': forms.TextInput(attrs={
                                        'placeholder': '영화 제목 입력',
                                        'class': 'title',}),
            'audience': forms.NumberInput(attrs={
                                        'placeholder': '관객수 입력',
                                        'class': 'audience',}),
            'poster_url': forms.Textarea(attrs={
                                        'placeholder': '포스터 url 입력',
                                        'class': 'poster_url',
                                        'rows': 2,
                                        'cols': 100,}),
            'description': forms.Textarea(attrs={
                                        'placeholder': '영화 설명 입력',
                                        'class': 'description',
                                        'rows': 5,
                                        'cols': 100,}),
            'genre': forms.NumberInput(attrs={
                                        'placeholder': '장르 번호 입력',
                                        'class': 'genre',
                                        'max_value': 11,
                                        'min_value': 1,}),
                    }
                    
class ScoreForm(forms.ModelForm):
    class Meta:
        model = Score
        fields = '__all__'
        widgets = {
            'content': forms.TextInput(attrs={
                                        'placeholder': '감상평 입력',
                                        'class': 'content',}),
            'score': forms.NumberInput(attrs={
                                        'placeholder': '평점 입력',
                                        'class': 'score',
                                        'max_value': 5,
                                        'min_value': 0,}),
        }
```



* views.py

```python
def create(request):
    if request.method == "POST":
        form = MovieForm(request.POST)
        if form.is_valid():
            title = form.cleaned_data.get('title')
            audience = form.cleaned_data.get('audience')
            poster_url = form.cleaned_data.get('poster_url')
            description = form.cleaned_data.get('description')
            genre = form.cleaned_data.get('genre')
            movie = Movie.objects.create(title=title, audience=audience, poster_url=poster_url, description=description, genre=genre)
            movie.save()
            return redirect('movies:detail', movie.pk)
    else:
        form = MovieForm()
    context = {'form': form}
    return render(request, 'movies/form.html', context)
```



* views.py

```python
def edit(request, movie_pk):
    movie = get_object_or_404(Movie, pk=movie_pk)
    if request.method == "POST":
        form = MovieForm(request.POST, instance=movie)
        if form.is_valid():
            movie = form.save()
            return redirect('movies:detail', movie_pk)
    else:
        form = MovieForm(instance=movie)
    context = {'form': form, 'movie': movie,}
    return render(request, 'movies/form.html', context)
```



* form.html

```django
    <h1>NEW Movie 입력</h1>
    
    <form method="POST">
        {% csrf_token %}
        {{ form }}
        <input type="submit" value="입력하기"/>
    </form>
```



* views.py

```python
from django.shortcuts import render, redirect, get_object_or_404

def detail(request, movie_pk):
    movie = get_object_or_404(Movie, pk=movie_pk)
    
def delete(request, movie_pk):
    movie = get_object_or_404(Movie, pk=movie_pk)
```



* form.html

```django
{% if request.resolver_match.url_name == 'create' %}
    <h1>NEW Movie 입력</h1>
{% else %}
    <h1>Movie 수정하기</h1>
{% endif %}
    
    <form method="POST">
        {% csrf_token %}
        {{ form }}
        <input type="submit" value="입력하기"/>
    </form>

{% if request.resolver_match.url_name == 'create' %}
    <a href="{% url 'movies:index' %}">뒤로</a>
{% else %}
    <a href="{% url 'movies:detail' movie.pk %}">뒤로</a>
{% endif %}
```



```bash
pip install django-crispy-forms
pip freeze > requirements.txt
```



* settings.py

```python
INSTALLED_APPS = [
    'movies.apps.MoviesConfig',
    'crispy_forms',

CRISPY_TEMPLATE_PACK = 'bootstrap4'
```



* forms.html

```django
{% load crispy_forms_tags %}

    <form method="POST">
        {% csrf_token %}
        {{ form|crispy }}
        <input type="submit" value="입력하기"/>
    </form>
```



* tree

```
.
├── crud
│   ├── __init__.py
│   ├── __pycache__
│   │   ├── __init__.cpython-36.pyc
│   │   ├── settings.cpython-36.pyc
│   │   ├── urls.cpython-36.pyc
│   │   └── wsgi.cpython-36.pyc
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── db.sqlite3
├── manage.py
├── movies
│   ├── __init__.py
│   ├── __pycache__
│   │   ├── __init__.cpython-36.pyc
│   │   ├── admin.cpython-36.pyc
│   │   ├── apps.cpython-36.pyc
│   │   ├── forms.cpython-36.pyc
│   │   ├── models.cpython-36.pyc
│   │   ├── urls.cpython-36.pyc
│   │   └── views.cpython-36.pyc
│   ├── admin.py
│   ├── apps.py
│   ├── fixtures
│   │   ├── genre.json
│   │   └── movie.json
│   ├── forms.py
│   ├── migrations
│   │   ├── 0001_initial.py
│   │   ├── __init__.py
│   │   └── __pycache__
│   │       ├── 0001_initial.cpython-36.pyc
│   │       └── __init__.cpython-36.pyc
│   ├── models.py
│   ├── templates
│   │   └── movies
│   │       ├── base.html
│   │       ├── detail.html
│   │       ├── form.html
│   │       └── index.html
│   ├── tests.py
│   ├── urls.py
│   └── views.py
└── requirements.txt
```

