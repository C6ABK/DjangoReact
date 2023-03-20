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
- `pip install djangorestframework`
- Import into `INSTALLED_APPS` in `backend/settings.py`

```
INSTALLED_APPS = [
  ...
  'base.apps.BaseConfig',
  'rest_framework',
]
```

- Go to `base/views.py` and modify as below...

```
...
from rest_framework.decorators import api_view
from rest_framework.response import Response
...

@api_view(["GET"])
def getRoutes(request):
  routes = [
     ...
  ]
  return Response(routes)
  
@api_view(["GET"]) 
def getProducts(request):
  return Response(products)

...
```

- Also in `base/views.py`, add the single product view...

```
...

@api_view(["GET"])
def getProduct(request, pk):
  product = None
  
  for i in products:
    if i["_id"] == pk:
      product = i
      break
  
  return Response(product)
```

- Add the url in `base/urls.py` - `path("products/<str:pk>/, views.getProduct, name="product")`

## Fetching Data
### HomeScreen.js
- Run the django server and open another terminal
- In the `frontend` directory, `npm install axios`
- `npm start`
- Go to `frontend/src/screens/HomeScreen.js` and modify as below (remove the import of the static products file).

```
...
import { useState, useEffect } from "react"
import axios from "axios"

function HomeScreen() {
 const [products, setProducts] = useState([])
 
 useEffect(() => {
   async function fetchProducts() {
     const { data } = await axios.get("/api/products/")
     setProducts(data)
   }
   
   fetchProducts()
 }, [])
 
 ...
}

```

- `pip install django-cors-headers`
- Go to `backend/settings.py` and add to `INSTALLED_APPS`

```
INSTALLED_APPS = [
  ...
  'rest_framework',
  'corsheaders',
]
```

- Place the corsheaders middleware at the top of the `MIDDLEWARE` group as below

```
MIDDLEWARE = [
  'corsheaders.middleware.CorsMiddleware',
  'django.middleware.security.SecurityMiddleware',
  ...
]
```

- Add `CORS_ALLOW_ALL_ORIGINS = True` at the bottom of `settings.py`

### Proxy URL
- Go to `package.json` and add `"proxy": "http://yourdomain.com"` below `"name"` - `0.0.0.0:8000`?

### ProductScreen.js
- Modify the products page as below

```
...
import { useState, useEffect } from "react"
import axios from "axios"

function ProductScreen({ match }) {
  const [product, setProduct] = useState([])
  
  useEffect(() => {
    async function fetchProduct() {
      const { data } = await axios.get(`/api/products/${match.params.id}`)
      setProduct(data)
    }
    
    fetchProduct()
  }
}
```
