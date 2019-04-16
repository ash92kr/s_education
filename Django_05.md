[TOC]

## Django_05

**Content**

0. CRUD Review
1. URL Names
2. RESTful

> 190313 Wed

---

### 0. CRUD Review

Django_03~04 참고해서 CRUD 다시 만들어보기

```bash
$ mkdir PROJECT03
$ cd PROJECT03
$ pyenv local django-venv 
$ django-admin startproject crud .
# 이후 과정 수업자료와 동일
# 삭제와 수정 버튼은 index 가 아닌 detail 페이지에 작성
```

---

### 1. URL Names

#### 1.1 하드코딩 URL 제거

```python
# example
path('<int:pk>/edit/', views.edit, name='edit'),
```

- 우리는 지금까지 사용한적은 없지만 urls.py 에 path 를 설정할 때, **주소의 이름**을 설정했다. path 에 이름을 설정하면, 우리가 만든 다른 주소에 요청을 보내거나 할 때, `/boards/{{ board.pk }}/edit/` 처럼 하드코딩할 필요없이 이름으로 사용할 수 있다.
- `{% url %}` template 태그를 사용하여 url 설정에 정의된 특정한 URL 경로들의 의존성을 제거할 수 있다.

```django
<!-- 이전 방식 -->
<a href="/boards/{{ board.pk }}/edit/">[수정]</a>

<!-- 새로운 방식 -->
<a href="{% url 'edit' board.pk %}">[수정]</a>
```

- 만약 edit view 의 URL 을 `/boards/ssafy/{{ board.pk }}/edit/` 으로 바꾸게 된다면, template 에서 모두 찾아가면서 바꾸는 것이 아니라 `boards/urls.py` 에서만 바꾸면 된다.

> 또한 `/` 를 빼먹는 실수를 차단할 수 있다.

#### 1.2 URL namespace 나누기

- 지금은 하나의 app 만 가지고 있지만 실제로 프로젝트에는 여러개의 app 이 만들어질 것이다. 
- 결국 동일한 url 이름을 가진 app 들이 많아지면,  위 방법은 Django 가 어떤 app 의 url 이름인지 구별할 수 없다.
- 그래서 우리는 static 이나 templates 에서 사용한 방법처럼 중간에 이름공간(namespace) 추가하는 것이다.
- 이름공간은 `urls.py` 에 **app_name** 을 통해 app 의 이름공간을 설정한다.

```python
# boards/urls.py
from django.urls import path
from . import views

app_name = 'boards'

urlpatterns = [
	...
]
```

- 이제 기존 `{% url %}` 코드를 다음과 같이 변경할 수 있다.

```django
<!-- 이전 방식 -->
<a href="{% url 'edit' board.pk %}">[수정]</a>

<!-- 새로운 방식 -->
<a href="{% url 'boards:edit' board.pk %}">[수정]</a>
```

- 하드코딩된 모든 url 을 깔끔하게 변경해보자.

```python
# boards/views.py
def create(request):
	...
    return redirect('boards:detail', board.pk)

def delete(request, pk):
	...
    return redirect('boards:index')

def update(request, pk):
	...
    return redirect('boards:detail', board.pk)
```

```django
<!-- boards/index.html -->
{% extends 'boards/base.html' %}
{% block body %}
    <h1>Board</h1>
    <a href="{% url 'boards:new' %}">글 작성하기</a>
	...
        <a href="{% url 'boards:detail' board.pk %}">[글 보러가기]</a>
        <hr>
    {% endfor %}
{% endblock %}
```

```django
<!-- boards/detail.html -->
{% extends 'boards/base.html' %}
{% block body %}
	...
    <a href="{% url 'boards:index' %}">Back</a>
	<a href="{% url 'boards:delete' board.pk %}" onclick="return confirm('R U SURE?')">[삭제]</a>
	<a href="{% url 'boards:edit' board.pk %}">[수정]</a>
{% endblock %}
```

```django
<!-- boards/new.html -->
{% extends 'boards/base.html' %}
{% block body %}
    <h1>NEW</h1>
    <form action="{% url 'boards:create' %}" method="POST">
	...
    <a href="{% url 'boards:index' %}">BACK</a>
{% endblock %}
```

```django
<!-- boards/edit.html -->
{% extends 'boards/base.html' %}
{% block body %}
    <h1>EDIT</h1>
    <form action="{% url 'boards:update' board.pk %}" method="POST">
	...
    <a href="{% url 'boards:index' %}">BACK</a>
{% endblock %}
```

- 변경이 끝났다면 동일하게 동작하는지 확인해보자.

---

### 2. RESTful

> 슬라이드로 개념 설명
>
> https://meetup.toast.com/posts/92
>
> [URL 과 URI](https://blog.lael.be/post/61)
>
> [언제나 헷갈리는 URL과 URI](http://sunychoi.github.io/java/2015/04/27/uri-url.html)

- 개요

  - REST는 `Representational State Transfer`라는 용어의 약자로서 2000년도에 로이 필딩 (Roy Fielding)의 박사학위 논문에서 최초로 소개. 로이 필딩은 HTTP의 주요 저자 중 한 사람으로 그 당시 웹(HTTP) 설계의 우수성에 비해 제대로 사용되어지지 못하는 모습에 안타까워하며 웹의 장점을 최대한 활용할 수 있는 아키텍처로써 REST를 발표했다.
  - REST 원리를 따르는 시스템은 종종 RESTful 이란 용어로 지칭한다.

- REST 구성

  - 자원(Resource) - URI
  - 행위(Verb) - HTTP Method
  - 표현(Representations)

- **Rest API 디자인 가이드**

  1. URI 는 정보의 자원을 **표현**해야한다.

     ```bash
     GET /boards/show/1 (X)
     POST /boards/1 (O)
     
     # URI는 자원을 표현하는데 중점을 두기 때문에 show 와 같은 행위에 대한 표현이 들어가서는 안된다.
     ```

  2. 자원에 대한 행위는 HTTP Method(GET, POST, PUT, DELETE)로 **표현**한다.

     ```bash
     GET /boards/1/update (X)
     PUT /boards/1 (O)
     
     GET /boards/1/delete (X)
     DELETE /boards/1 (O)
     
     # GET Method 는 리소스 생성에 부적합하다.
     ```

  > django 에서는 PUT, DELETE 같은 비공식적 요청을 default 로 지원하고 있지 않아 다음과 같이 절충한다.
  >
  > ```bash
  > GET /boards/2/edit/ 	# 수정 페이지 보여줌
  > POST /boards/2/edit/	# 수정 작업 수행
  > ```
  >


- REST 는 URI 마지막에 `/`(슬래시) 를 쓰지 않는 걸 권장하는데, django 는 권장하고 있다. (권장을 넘어 쓰지 않으면 에러가 발생하기도 한다.)

#### 2.1 RESTful 적용하기

- 기존 urls.py

  ```python
  urlpatterns = [
      path('<int:pk>/update/', views.update, name='update'),
      path('<int:pk>/edit/', views.edit, name='edit'),
      path('<int:pk>/delete/', views.delete, name='delete'),
      path('<int:pk>/', views.detail, name='detail'),
      path('create/', views.create, name='create'),
      path('new/', views.new, name='new'),
      path('', views.index, name='index'),
  ]
  ```

- RESTful 적용 후

  ```python
  urlpatterns = [
      path('<int:pk>/edit/', views.edit, name='edit'), # GET(edit), POST(update)
      path('<int:pk>/delete/', views.delete, name='delete'), # POST(delete)
      path('<int:pk>/', views.detail, name='detail'), # GET
      path('new/', views.new, name='new'), # GET(new), POST(create)
      path('', views.index, name='index'), # GET
  ]
  ```

#### 2.2 url, views 정리하기

- new + create

  ```python
  # views.py
  def new(request):
      if request.method == 'POST':
  		# create
          title = request.POST.get('title')
          content = request.POST.get('content')
  
          board = Board(title=title, content=content)
          board.save()
          return redirect('boards:detail', board.pk)
      else:
          # new
          return render(request, 'boards/new.html')
  ```

  ```django
  <!-- boards/new.html -->
  ...
  	<form action="" method="POST"> 
          ...
  ```

  - form tag 의 action 이 없는데, 이렇게하면 **form 데이터가 현재 URL 페이지로 다시 POST 된다. (action 을 아예 쓰지 않아도 작동한다.)**
  - 헷갈린다면 그냥 `action="{% url 'boards:new' %}"` 를 사용해도 무방하다.

- edit + update

  ```PYTHON
  # views.py
  def edit(request, pk):
      if request.method == 'POST':
          # update
          board = Board.objects.get(pk=pk)
          board.title = request.POST.get('title')
          board.content = request.POST.get('content')
          board.save()
          return redirect('boards:detail', board.pk)
      else:
          # edit
          board = Board.objects.get(pk=pk)
          return render(request, 'boards/edit.html', {'board': board})	
  ```

  ```django
  <!-- boards/edit.html -->
  ...
  	<form action="" method="POST">
          ...
  ```

- delete

  - 현재 GET 방식이기 때문에 url 에 직접 삭제 주소를 입력하면 삭제가 된다.
  - 그래서 기존에 GET 으로 구현 했었던 것을 의미에 맞도록 POST 로 변경하고 조건을 만든다.

  ```python
  # views.py
  def delete(request, pk):
      board = Board.objects.get(pk=pk)
      if request.method == 'POST':
          board.delete()
          return redirect('boards:index')
      else:
          return redirect('boards:detail', board.pk)
  ```

  ```django
  <!-- boards/detail.html -->
  {% extends 'boards/base.html' %}
  {% block body %}
  	...
      <form action="{% url 'boards:delete' board.pk %}" method="POST" style="display: inline" onsubmit="return confirm('R U SURE?');">
          {% csrf_token %}
          <input type="submit" value="삭제">
      </form>
  {% endblock %}
  ```

  > 완성했다면 동일하게 동작하는지 최종 확인 해보자.