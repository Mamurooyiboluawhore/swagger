# IMPLEMENTING SWAGGER IN DRF(DJANGO REST FRAMEWORK)

### INTRODUCTION
### PREREQUISITE
### BODY
### CONCLUSION

## INTRODUCTION:
What is swagger documentation?
Swagger is an open-source framework that helps developers design, build,  document, and consume RESTFUL web services.You can say swagger is used to document APIs. This helps the developers to understand the basic behavior of every (API) created at the backend. 

It also outlines all available models in a particular project. This is very helpful to developers as it saves long hours of meetings. 
Swagger uses a JSON and YAML called openAPI specification(OAS) to define APIs. This specification includes details such as endpoints, methods, requests, responses, etc.
In this article …..

## IMPORTANCE OF USING SWAGGER DOCS

Some of the importance of integrating  swagger docs in the backend of your code includes:
Automated documentation of API 
Testing and debugging

## PREREQUISITE
Basic knowledge of DJANGO and DRF is required to start this tutorial.
To continue, we will need to install the necessary dependencies needed for the swagger implementation. You must have Python installed on your local machine. If you don’t yet, you need to install it from https://www.python.org/downloads/.


Next, we need to create a virtual environment. It can be created with the code below. Depending on the version of Python you downloaded. For python 3.12. 1. Use this command below.
```bash
    Python -m venv {venv name}
```
Where the ‘{venv name}’ refers to any desired names you give to your virtual environment(venv). The convention is using Venv but whatever name you choose to use is totally up to you.
It should look like this
```bash
  Python -m venv venv
```
Then run this command on Windows to activate your virtual environment 
Venv/Script/activate
On Linux, run this command to activate your Venv 
venv/bin/activate

Now, you need to install Django and Django rest framework (drf) within your virtual environment. To install Django, use this command 
```bash
      \\ commnand to install django
      Pip install Django
```
Then this is to install DRF
```bash
    Pip install Django rest framework.
```
These are the necessary packages you need to start up a simple DRF application 

## CREATE A SAMPLE APPLICATION
After setting up your virtual environment (Venv), installing Django and Django rest framework within your virtual environment, you will need to start a new project.
Run the command below to set up a new project.
```bash
    django-admin startproject myproject
```
This will create a new directory called __myproject__ with the basic structure of Django in it. You can run your server to make sure everything is working fine.
You should create a simple model for products.
There are several ways to do this but I prefer creating a new app for each functionality in the project. 
Run this command on your terminal to do so.
```bash
    Python manage.py createapp products
```

This will create a new directory with the basic structure of an app in it. 
Navigate to your __“settings.py”__ file and include the __‘products’__ to your installed app. 
Navigate to your app directory in this case, it is products and create a simple model for it in your app's models.py file
```bash
    Class Product(model. Models)
      id = models.UIIDfield()
      user = models.Foreignkey(User, on delete=Cascade)
      createdat = models.datetimefield(auto_add)
      updatedat = models.datetimefield(auto_add_now)
  ```
Create a serializer class to serialize your data. You need to create a serializer.py file in your app directory(products directory). In your serializer file, create a serializer class


```bash
From .Models import products 


Class ProductSerializer(serializer.Serializers):
      Class Meta:
            model = products
            fields = ‘__all__’

```

In your product views.py, create a simple crud. 
```bash
    from django.http import JsonResponse
from django.shortcuts import get_object_or_404
from django.shortcuts import render
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status
from accounts.models import Product
from .serializers import ProductSerializer

class ProductListCreateAPIView(APIView):
    """
    API endpoint for listing and creating products.

    GET:
    Returns a list of all products.

    POST:
    Creates a new product.

    Raises:
    HTTP_500_INTERNAL_SERVER_ERROR: If an internal server error occurs.
    """
    def get(self, request, format=None):
        """
        Retrieves a list of all products.

        Args:
        request: HTTP request object.
        format: The requested data format. Default is None.

        Returns:
        Response containing a list of serialized products.
        """
        try:
            products = Product.objects.all()
            serializer = ProductSerializer(products, many=True)
            return Response(serializer.data)
        except Exception as e:
            return Response({'error': str(e)}, status=status.HTTP_500_INTERNAL_SERVER_ERROR)

    def post(self, request, format=None):
        """
        Creates a new product.

        Args:
        request: HTTP request object.
        format: The requested data format. Default is None.

        Returns:
        Response containing information about the created product or validation errors.
        """
        try:
            serializer = ProductSerializer(data=request.data)
            if serializer.is_valid():
                serializer.save()
                response = {
                    "message": "Product added successfully",
                    "status_code": 201,
                    "data" : serializer.data
                }
                return Response(response, status=status.HTTP_201_CREATED)
            return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
        except Exception as e:
            response= {
                "message": "internal server error",
                "status_code": 500,
                "data": {'detail': str(e)},
            }
            return Response(response, status=status.HTTP_500_INTERNAL_SERVER_ERROR)


class ProductDetailAPIViews(APIView):
    """
    API endpoint for retrieving, updating, and deleting individual products.

    GET:
    Retrieves details of a specific product.

    PUT:
    Updates details of a specific product.

    DELETE:
    Deletes a specific product.

    Raises:
    HTTP_404_NOT_FOUND: If the requested product does not exist.
    HTTP_500_INTERNAL_SERVER_ERROR: If an internal server error occurs.
    """
    def get_object(self, pk):
        """
        Helper function to retrieve a product object by its primary key (pk).

        Args:
        pk: The primary key of the product.

        Returns:
        Product object.

        Raises:
        HTTP_404_NOT_FOUND: If the product with the given primary key does not exist.
        """
        try:
            return Product.objects.get(pk=pk)
        except Product.DoesNotExist:
            return get_object_or_404(Product, pk=pk)
    
    
    def get(self, request, pk, format=None):
        """
        Retrieves details of a specific product.

        Args:
        request: HTTP request object.
        pk: The primary key of the product.
        format: The requested data format. Default is None.

        Returns:
        Response containing details of the specified product.
        """
        product = self.get_object(pk)
        serializer = ProductSerializer(product)
        return Response(serializer.data)
    
    def put(self, request, pk, format=None):
        """
        Updates details of a specific product.

        Args:
        request: HTTP request object.
        pk: The primary key of the product.
        format: The requested data format. Default is None.

        Returns:
        Response containing updated details of the product or validation errors.
        """
        product = self.get_object(pk)
        serializer = ProductSerializer(product, data=request.data)
        if serializer.is_valid():
            serializer.save()
            response = {
                "message": "Product successfully updated",
                "status_code": 200,
                "data": serializer.data
            }
            return Response(response, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    

    def delete(self, request, pk):
        """
        Deletes a specific product.

        Args:
        request: HTTP request object.
        pk: The primary key of the product.

        Returns:
        Response confirming the deletion of the product.
        """
        try:
            product = get_object_or_404(Product, pk=pk)
            product.delete()
            return Response({'message': 'Product deleted successfully.'})
        except Exception as e:
            return JsonResponse({'error': str(e)}, status=500)
```
Then create the URLs for product in your product urls.py file.
```bash
    from django.urls import path
from .views import ProductListCreateAPIView, ProductDetailAPIViews

urlpatterns = [
    # URL pattern for listing all products and creating new products
    path('', ProductListCreateAPIView.as_view(), name='product-list-create'),

    # URL pattern for retrieving, updating, and deleting a specific product
    path('<uuid:pk>/', ProductDetailAPIViews.as_view(), name='product-detail'),

    # URL pattern for updating a specific product
    path('<uuid:pk>/update/', ProductDetailAPIViews.as_view(), name='product-update'),

    # URL pattern for deleting a specific product
    path('<uuid:pk>/delete/', ProductDetailAPIViews.as_view(), name='product-delete'),
]

```

Navigate to your core urls.py file, and do the basic settings for Swagger docs. First, you need to import all the necessary modules.


“from rest_framework_swagger.views import get_swagger_view
from drf_yasg.views import get_schema_view
from drf_yasg import openapi
from rest_framework import permissions”

Then proceed to set up Swagger with the code below.

“schema_view = get_schema_view(
    openapi.Info(
        title="First Collectionz API",
        default_version='v1',),
    public=True,
    permission_classes=(permissions.AllowAny,),
)”


After which, include swagger docs in your urlspatterns with this line of code.

urlpatterns = [
       path('api/docs/', schema_view.with_ui('swagger', cache_timeout=0),name='schema-swagger-ui'),

]. 

Then run your django server…. Congratulations, you have successfully
  implemented Swagger. PS. For deployment purposes, please configure your static files. You can do this by running this command below. 
Python manage.py collectstatic

This will create a static file directory. 

Conclusion 
Documenting with swagger is a great way to 






