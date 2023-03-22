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

### Proxy URL - (unreliable on cloud IDE?)
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

## Database Setup & Admin Panel
- `python manage.py migrate`
- `python manage.py createsuperuser`
- Start server and go to `https://domain.com/admin` to access admin panel.
- Use `CSRF_TRUSTED_ORIGINS=['https://*.yourdomain.com/']` is you get CSRF error while in development.

## Models
- Go to `base/models.py` and import the user model - `from django.contrib.auth.models import User`
- Add the product model below to `models.py`

### Product Model
```
class Product(models.Model):
  user = models.ForeignKey(User, on_delete=models.SET_NULL, null=True)
  name = models.CharField(max_length=200, null=True, blank=True)
  brand = models.CharField(max_length=200, null=True, blank=True)
  category = models.CharField(max_length=200, null=True, blank=True)
  description = models.TextField(null=True, blank=True)
  rating = models.DecimalField(max_digits=7, decimal_places=2, null=True, blank=True)
  numReviews = models.IntegerField(null=True, blank=True, default=0)
  price = models.DecimalField(max_digits=7, decimal_places=2, null=True, blank=True)
  countInStock = models.IntegerField(null=True, blank=True, default=0)
  createAt = models.DateTimeField(auto_now_add=True)
  _id = models.AutoField(primary_key=True, editable=False)
  
  def __str__(self):
    return self.name
```

- `python manage.py makemigrations`
- `python manage.py migrate`
- Go to `base/admin.py` and modify as below

```
...
from .models import Product

admin.site.register(Product)
```

### Review Model
- Add to `base/models.py`

```
class Review(models.Model):
  product = models.ForeignKey(Product, on_delete=models.SET_NULL, null=True)
  user =  models.ForeignKey(User, on_delete=models.SET_NULL, null=True)
  name = models.CharField(max_length=200, null=True, blank=True)
  rating = models.IntegerField(null=True, blank=True, default=0)
  comment = models.TextField(null=True, blank=True)
  _id = models.AutoField(primary_key=True, editable=False)
  
  def __str__(self):
    return str(self.rating)

```

### Order Model
- Add to `base/models.py`

```
class Order(models.Model):
  user = models.ForeignKey(User, on_delete=models.SET_NULL, null=True)
  paymentMethod = models.CharField(max_length=200, null=True, blank=True)
  taxPrice = models.DecimalField(max_digits=7, decimal_places=2, null=True, blank=True)
  shippingPrice = models.DecimalField(max_digits=7, decimal_places=2, null=True, blank=True)
  totalPrice = models.DecimalField(max_digits=7, decimal_places=2, null=True, blank=True)
  isPaid = models.BooleanField(default=False)
  paidAt = models.DateTimeField(auto_now_add=False, null=True, blank=True)
  isDelivered = models.BooleanField(default=False)
  deliveredAt = models.DateTimeField(auto_now_add=False, null=True, blank=True)
  createdAt = models.DateTimeField(auto_now_add=True)
  _id = models.AutoField(primary_key=True, editable=False)
  
  def __str__(self):
    return str(self.createdAt)
  
```

### Order Item Model
- Add to `base/models.py`

```
class OrderItem(models.Model):
  product = models.ForeignKey(Product, on_delete=models.SET_NULL, null=True)
  order = models.ForeignKey(Order, on_delete=models.SET_NULL, null=True)
  name = models.CharField(max_length=200, null=True, blank=True)
  qty = models.IntegerField(null=True, blank=True, default=0)
  price = models.DecimalField(max_digits=7, decimal_places=2, null=True, blank=True)
  image = models.CharField(max_length=200, null=True, blank=True)
  _id = models.AutoField(primary_key=True, editable=False)
  
  def __str__(self):
    return str(self.name)
```

### Shipping Address Model
- Add to `base/models.py`

```
class ShippingAddress(models.Model):
  order = models.OneToOneField(Order, on_delete=models.CASCADE, null=True, blank=True)
  address = models.CharField(max_length=200, null=True, blank=True)
  city = models.CharField(max_length=200, null=True, blank=True)
  postalCode = models.CharField(max_length=200, null=True, blank=True)
  country = models.CharField(max_length=200, null=True, blank=True)
  shippingPrice = models.DecimalField(max_digits=7, decimal_places=2, null=True, blank=True)
  _id = models.AutoField(primary_key=True, editable=False)
  
  def __str__(self):
    return str(self.address)
```

- Go to `base/admin.py` and register the models

```
from django.contrib import admin
from .models import *

admin.site.register(Product)
admin.site.register(Review)
admin.site.register(Order)
admin.site.register(OrderItem)
admin.site.register(ShippingAddress)
```

- `python manage.py makemigrations`
- `python manage.py migrate`
- Check admin panel to test all tables are displayed correctly.

## Product Image Field
- Go to `base/models.py` again and add the image field

```
class Product(models.Model):
  ...
  image = models.ImageField(null=True, blank=True)
  ...
```

- `pip install pillow`
- `python manage.py makemigrations`
- `python manage.py migrate`
- This will save images in the `backend` directory - this is fixed in the next section.

## Static Files
- Create a folder `static` in the root of the `backend`
- Go to `backend/backend/settings.py` and go to the bottom where `STATIC_URL...` should already exist.
- Add the following options to `settings.py`

```
...
STATIC_URL = '/static/'
MEDIA_URL = '/images/'

STATICFILES_DIRS = [
  BASE_DIR / 'static'
]

MEDIA_ROOT = 'static/images'
...
```

- Go to `backend/backend/urls.py` and add `from django.conf.urls.static import static`
- Add the following below the existing `urlpatterns` section...

```
urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

## Serializing Data
- Go to `base/views.py` and add `from .models import Product`
- Modify `def getProducts` as below

```
@api_view(['GET'])
def getProducts(request):
  products = Product.objects.all()
  return Response(products)
```

- Create `serializers.py` in `base` and fill as below...

```
from rest_framework import serializers
from django.contrib.auth.models import User
from .models import Product

class ProductSerializer(serializers.ModelSerializer):
  class Meta:
    model = Product
    fields = '__all__'
```

- Note: you can return specific fields using the format below...
```
class ProductSerializer(serializers.ModelSerializer):
  class Meta:
    model = Product
    fields = ['name', 'price']
```

- Go back to `base/views.py` and `from .serializers import ProductSerializer`
- Modify `def getProducts` as below...

```
@api_view(['GET'])
def getProducts(request):
  products = Product.objects.all()
  serializer = ProductSerializer(products, many=True)
  return Response(serializer.data)
```

- Modify the individual `def getProduct` as below. Because we're only returning the specific product with `/api/products/1`, many is set to False.
```
@api_view(['GET'])
def getProduct(request, pk):
  product = Product.objects.get(_id=pk)
  serializer = ProductSerializer(product, many=False)
  return Response(serializer.data)
```
