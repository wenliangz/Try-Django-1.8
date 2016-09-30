# ====== Setup virtual environment ======
## 1. overivew
## 2. Steps to setup
 - install virtualenv software if not installed in the system: 
 `$pip install virtualenv`
 - make a project folder and create virtual environment inside the project folder by going to the project folder and running: 
 `$virtualenv .`
 - If using Pycharm, you need to change the interpreter to virtual environment folder: "Scripts" folder for windows; 'bin' folder for linux. And within the project, you may need to mark the right folder as root folder
 - activate the virtual environment: 
    - for windows: `$.\Scripts\activate`
    - for linux: `$source bin/activate`
 - Install required software within the virtual environment
    - install all required software individually. 
    - freeze all softwares into a requirements.txt file for future easy installation: `$pip freeze > requirements.txt`
    - If there is a created requirement.txt for the project, install them all at once by : `$pip install -r requirements.txt`
    
# ====== Start django project ======
## 1. Create project: 
- First, make sure you are in the project folder you want, and have activated the virtual environment and have all required softwre installed. 
- run django command by start a new project. This will create a folder, projectname, in which contains all the initial files necessary for the project including another folder called projectname: 
`$django-admin.py startproject projectname`
- For best practice(not necessary), rename the projectname to 'src' in order to prevent confusions between virtual env, project folder manually created  and project folder created by command. 
- go to the folder of 'src' for running 'manage.py' command.

## 2. Django project Settings
- template configuration


# =====Create Django Apps  ==========

## ====== Django contribute apps : admin and auth ======
When you create a django project, django automatically create and installed some apps for you, which are called 'contirbute apps'
```
INSTALLED_APPS = (
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
)
```
## 1. admin app
- one of the contribute apps that have built-in interfaces and views useful for managing and viewing all databases within the project(site administration). 
- you will need to register the user custom app in the admin.py file, in order for the admin app to manage it. 
- the default app url: /admin
## 2. auth app
- this app is connected to the admin app, so that you can create superuser to log into the admin app for managing databases

## ====== Create our own app ======
## 1. create and install app
- to create:
`$python manage.py startapp appname`
- to install: add the app to the settings
```
INSTALLED_APPS = (
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
)
```
## 2. URL routing and view
Everytime you enter url in browser, you are making a http request to the server. It is important to understand how the url is routed and request is passed to the view function.
- The URLConf will send the request to the corresponding view based on the url information on the request. 
- The view function takes the request, and return a response to show webpage on the browser.


## 3. template configuration
- understand how the template is looked and loaded behind the hood, based on the template settings:
    - Filesystem loader (look for the folder manually defined, usually we make templates under the base folder): `'DIRS': [os.path.join(BASE_DIR, "templates")]`
    - App_directory loader(looking for tempaltes folder inside each app folder): `'APP_DIRS': True,` 

    
# ======== Work with Admin Views ===============    
The django admin app built-in interfaces(different views) useful for managing and viewing all databases within the project(site administration). We are first going to use the admin app to demonstrate how to workk with data using the model and model forms in our admin views before we build and use our own views. 
    
## ======== Model and admin views ===============

##1. Model Overview
Django use Object Relational mapper(ORM) to map a model class to a database table. Each model(a class object) generally corresponds to the whole data table in a database. 
- Each model field (class attribute) corresponds to the fields in the database.
- Each instance of that model(a instance object) corresponds to represents a particular record in the database table.
## 2. Add data to the model in different ways at different situations
- Through django admin app, this interface is usually for the site administor to use.
    - register our model into django admin app, within the file admin.py
    - go site/admin in browser and log into admin to add data to our model
        - notice how the name of the model show up based on the model name registered
        - the default fields shows up in admin is the __unicode__ field returned in the model
    - To customize how the user model fields show up in the admin app
        - within admin.py, create another class inherit from admin.ModelAdmin class.This class will take the model and fields defined in the class attribute to create an model form for displaying in the admin View. 
        - we can customize the display by overriding the class attributes.
        - instead of passing the model, we can even define our own model form, and directly pass to the class by overriding the form attribute to let admin app to use our own customized model form within the admin form view. 
        ```
        from django.contrib import admin
        from .models import SignUp
        from .forms import SignUpForm
        # Register your models here.
        class SignUpAdmin(admin.ModelAdmin):
            list_display = ['__unicode__','timestamp','updated']
            form =SignUpForm
            # class Meta:
            #     model = SignUp
        
        admin.site.register(SignUp,SignUpAdmin)
        ```
- Use commands in the shell. This interface is normally used for developers when testing database
`$python manage.py shell`
    - create instance of the model class, which represents one record in the database
    - run save method of the instance to save it into the database
- Build our own views to add data. This is for the end users. We will have to spend most of the time on building these views for different operations: CRUD!
    
## ======== ModelForm and admin views ===============
Now we want to create our own ModelForm, very like the ModelAdmin, to display and customize the model fields in our own view to be created.

## 1. create a form.py file and define a model form, inherited from form.ModelForm
```
from django import forms
from .models import SignUp

class SignUpform(forms.ModelForm):
    class Meta:
        model = SignUp
        Fields = ['email']

```

## 2. Use our custom model form in the admin app
- we can use our own model form to override the form in the ModelAdmin class 
```
from django.contrib import admin
from .models import SignUp
from .forms import SignUpForm
# Register your models here.
class SignUpAdmin(admin.ModelAdmin):
    list_display = ['__unicode__','timestamp','updated']
    form =SignUpForm
    # class Meta:
    #     model = SignUp

admin.site.register(SignUp,SignUpAdmin)
```

## 3. form validation
The django form class have some built-in fields validation for each fields. We can override the fied validation  by adding custom method in the form class.
- the form data in the form will be saved into the attribute, self.cleaned_data, in a dict format everytime the form is submitted.
- To override validation of each specific field, define method as: def clean_field(). For example, to override the default validation for email field:
```
from django import forms
from .models import SignUp

class SignUpForm(forms.ModelForm):
    class Meta:
        model = SignUp
        fields = ['full_name','email']

    def clean_email(self):
        email = self.cleaned_data.get('email')
        email_base,provider = email.split('@')
        domain,extension = provider.split('.')
        if not domain == 'uga':
            raise forms.ValidationError('Please make sure you use uga email')
        if not extension =='edu':
            raise forms.ValidationError('Please use a valid .EDU email address')
        return email
    
    def clean_full_name(self):
        full_name = self.cleaned_data.get('full_name')
        # write validation code here
        return full_name

```
- to override validation of the whole form, define method as: def form_isvalid()


# ======== Work with our own Views (CRUD) ===============   
Now we need to build our own views to work with data. These views are the most important as they are directly used by end users. Based on the functions of views ( the type request the views take), we can divided them into four different kinds: CRUD
- CREATE
- RETRIEVE: List
- Update/Edit
- Delete

## 1. View and template


