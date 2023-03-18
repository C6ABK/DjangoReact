# Serving & Fetching Data From Django
## Building the Backend
### Virtual Environment
- Go to the `ecommerce` root directory.
- `pip install virtualenv`
- `virtualenv myenv` or `python -m venv myenv`
- Activate the virtual environment `myenv\scripts\activate` - use `source myenv/bin/activate` from bash
- `pip list` to view dependencies.
- `pip install django`
- `django-admin startproject backend`
- `cd backend`
- `python manage.py runserver` - add `0.0.0.0:8000` for the codeanywhere IDE

### Create API App
- In the backend folder...
- `python manage.py startapp base`
- Go to `backend/settings.py`
- Add the new app to `INSTALLED_APPS`

```
 INSTALLED_APPS = [
  ...
  'django.contrib.staticfiles',
  'base.apps.BaseConfig',
 ]
```

- Go to `base/views.py` and modify as below...

```
from django.http import JsonResponse

def getRoutes(request):
  routes = [
    "/api/products/",
    "/api/products/create/",
    "/api/products/upload/",
    "/api/products/<id>/reviews/",
    "/api/products/top/",
    "/api/products/<id>/",
    "/api/products/delete/<id>/",
    "/api/products/<update>/<id>/",
  ]
  return JsonResponse(routes, safe=False)
```

- Create `urls.py` in the `base` directory and fill as below...

```
from django.urls import path
from . import views

urlpatterns = [
 path("", views.getRoutes, name="routes"),
]
```

- Go to `backend/urls.py` and modify as below...

```
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
 path("admin/", admin.site.urls),
 path("api/", include("base.urls")),
]
```

- Copy `products.js` from the `frontend`, create `products.py` in base and paste in.
- Get rid of `const` and `export` to create a python dictionary.

```
products = [
 {
   ...
 },
 {
   ...
 }
]
```

- Go to `base/views.py`, `from .products import products`
- Create a new view...

```
def getProducts(request):
  return JsonResponse(products, safe=False)
```

- Go to `base/urls.py` and add a new path `path("products/", views.getProducts, name="products"),`

## Django REST Framework
