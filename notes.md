# ====== Setup virtual environment ======
 - install virtualenv software if not installed in the system: 
 `$pip install virtualenv`
 - create virtual environment inside the project folder by going to the project folder and running: 
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
## 1. Django project Settings
- template configuration

# ====== Django contribute app ======
## 1. admin and superuser


# ====== Create our own app ======
## 1. first view and URL routing
- understand how the url is routed to the view function
    - everytime you enter url in browser, you are making a http request to the server
    - the view function takes the request, and return a response to show webpage on the browser
- understand how the template is looked and loaded behind the hood, based on the template settings:
    - Filesystem loader (look for the folder manually defined, usually we make templates under the base folder): `'DIRS': [os.path.join(BASE_DIR, "templates")]`
    - App_directory loader(looking for tempaltes folder inside each app folder): `'APP_DIRS': True,` 

