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

## 1. View in general
- django View process HttpRequest and return HttpResponse. 
    - Inputs for View fuction or class: 
        - request: context provided by middleware
        - template: 
        - context: context provided by user, python dict
    - Ouput: HttpResponse
        - render(request, template, context)
        
```
from django.shortcuts import render

# Create your views here.
def home(request):
    title = 'Homepage of %s'%(request.user)
    context = {
        "template_title": title,
    }
    return render(request, "home.html", context)
```        

- Template is written in a certain template language, which can use template variables that can come from context provided in the view function
- Another source of contex variable for template is from the sources defined in settting. for example, {{user}} {{request.user}} ,not provided in the view function 


```
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, "templates"), "/Users/jmitch/desktop/trydjango18/src/abctemplates"],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

## 2. Form in a our own view ( ususally for Create and Update view)
- import our form class and create an instance of our ModelForm class by passing request.POST or None
- add the form instance to the context for rendering in the view
    - we still need to use the form tag in our view
    - if method is not specified in the form tag, the default is GET method. the action is where it goes after submit form.
    ```
    <form method='POST' action =''> {%csrf_token%}
    {{ form.as_p }}
    <input type="submit" value="Sign Up"/>
    </form>
    ```
   - CSRF token is needed if POST method is used for the security reason 

## 3. Regular Form in a View
- if your don't need to save the user input information into the database, regular form class is enough, instead of modelform.
- to catch the information sent by the form, e.g. for sending a email, you just need to get the info from cleaned_data.

## 4. Sending email from django
-  in settings:
```
ALLOWED_HOSTS = []
EMAIL_HOST = 'smtp.gmail.com'
EMAIL_HOST_USER = 'bluebirdbioMA@gmail.com'
EMAIL_HOST_PASSWORD = 'BBBbio@MA'
EMAIL_PORT = 587 
EMAIL_USE_TLS = True
```
- in the view function:
 ```
def contact(request):
    form = ContactForm(request.POST or None)

    if form.is_valid():
        form_email = form.cleaned_data.get('email')
        form_message = form.cleaned_data.get('message')
        form_full_name = form.cleaned_data.get('full_name')
        subject = 'Site contact form'
        from_email = settings.EMAIL_HOST_USER
        to_email = [from_email,'wenliangz@gmail.com']
        contact_message = '''
        %s:%s via %s
        '''%(form_full_name,form_message,form_email)

        send_mail(subject,
                  contact_message,
                  from_email,
                  to_email,
                  fail_silently=False
                  )
    context = {
        "form":form
    }
    return render(request, 'form.html',context)
```

# ======== Static files ===============  
## 1. STATIC_URL and STATIC_ROOT 
Static files are served at different server. when in production, there are two server: one is django related file server and one is static file server.during development, we are trying to keep them separate. 
- location of static files during production: STATIC_ROOT. All the static files in the project( from all apps) will be collected to that location by running: python manage.py collectstatic. The settings might be a little different depending on the server, however, this is the place where all files in STATIC_DIRS are going to send to. The folder is better to be seperated outside of django project folder.
- location of static files during development: STATIC_DIRS. All the static files in this place will be collected into the STATIC_ROOT in production
- one tip: os.path.dirname(BASE_DIR) will give the one-level up DIR of the BASE_DIR. 
## 2. MEDIA_URL and MEDIA_ROOT
 Media are files uploaded by end users. MEDIA_ROOT is the place where all the files uploaded by users are going to be sent to. In production, the folder is also better to be seperated outside of django project folder.
## 3. Serving Static and Media files
## 4. Add Bootstrap to django

# ======== About Template and Styling===============  
- django template tags: Include, inheritance, blocks. Need {% load staticfiles %} template tag loader
- use third party app, crispy forms, to make forms look better on the template: {% load crispy_forms_tags %} template tag loader
- Bootstrap Grid System
- CSS with style blocks in the template
- use URL names as links in the template
- Styling page with CSS 
- logo image in Navbar
- use Font Awesome, little icon, all in css, not image,

# ======== Django Registration Redux =============== 
- installation and setup registration redux
- Authentication and login form in the Navbar
- promo video and images

