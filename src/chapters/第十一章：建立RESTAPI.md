# 第十一章：建立RESTAPI.md

## 安裝 Django REST framework

```shell
$ pipenv install djangorestframework
```

## core/settings.py

> 註冊 Django REST framework 到 Django

```python
INSTALLED_APPS = [
    ...

    'rest_framework',
    'bootstrap4',

    ...
]
```

## 建立一個新的 app，專門給 API 使用

```shell
$ python manage.py startapp api
```

## core/settings.py

> 註冊 API app 到 Django

```python
INSTALLED_APPS = [
    ...

    'coffees.apps.CoffeesConfig',
    'api.apps.ApiConfig',
]
```

## api/serializers.py

```python
from rest_framework.serializers import ModelSerializer

from coffees.models import Coffee


class CoffeeSerializer(ModelSerializer):
    class Meta:
        model = Coffee
        fields = '__all__'
```

## api/views.py

```python
from rest_framework.viewsets import ModelViewSet

from api.serializers import CoffeeSerializer
from coffees.models import Coffee


# Create your views here.
class CoffeeViewSet(ModelViewSet):
    queryset = Coffee.objects.all()
    serializer_class = CoffeeSerializer
```

## core/urls.py

```python
from django.contrib import admin
from django.urls import path, include
from rest_framework.routers import DefaultRouter

from api import views as api_views

router = DefaultRouter()
router.register('coffees', api_views.CoffeeViewSet)

urlpatterns = [
    path('coffees/', include('coffees.urls')),
    path('api/', include(router.urls)),
    path('admin/', admin.site.urls),
]
```
