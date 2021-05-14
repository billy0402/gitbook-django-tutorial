                        # API 權限

## core/settings.py

```python
INSTALLED_APPS = [
    ...
    'rest_framework.authtoken',
    ...
]

...

# REST Framework
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework.authentication.TokenAuthentication',
        'rest_framework.authentication.SessionAuthentication',
    ),
}
```

## core/urls.py

```python
from rest_framework.authtoken.views import obtain_auth_token

urlpatterns = [
    ...
    path('api/auth/', obtain_auth_token, name='api_auth'),
    ...
]
```

## api/views.py

```python
from rest_framework.permissions import IsAuthenticated

class CoffeeViewSet(ModelViewSet):
    ...
    permission_classes = (IsAuthenticated,)
```
