# Serving & Fetching Data From Django
## Building the Backend
### Virtual Environment
- Go to the `ecommerce` root directory.
- `pip install virtualenv`
- `virtualenv myenv`
- Activate the virtual environment `myenv\scripts\activate` - might be different with ubuntu, check the `activate` file for further instructions.
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
  return JsonResponse("Hello", safe=False)
```

- Create `urls.py` in the `base` directory and fill as below...

```
from django.urls import path
from . import views

urlpattern
```
