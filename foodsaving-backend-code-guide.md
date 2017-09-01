# foodsaving.world

Website: https://foodsaving.world/  
Repository on GitHub: https://github.com/yunity/foodsaving-backend


**Written in Python Django REST**

- Python – object-oriented programming language  
- Django – Python framework (for faster web development)  
         Tutorial: <https://docs.djangoproject.com/en/1.11/intro/tutorial01/> 
- REST   – Framework on top of django for building Web APIs  
         Tutorial: <http://www.django-rest-framework.org/tutorial/1-serialization/>

         
## 01  Setup

First deside if you want to use Docker for the setup or try to install everything without it. Both options are described in the README.md in the foodsaving-backend repository. 

If you want to work with Docker, we would suggest to use 3 tabs in the shell:

1. Tab for Communicating with git / GitHub (doing that inside the docker container might raise weird errors)  
2. Tab (with Docker running) for run manage.py commands:

After starting Docker your lines in the shell start with (env)

Running tests:   
> python manage.py test

After changing a model/class you have to migrate them:   
> python manage.py makemigrations   
> python manage.py migrate

Leave Docker   
> exit

3, Tab (with Docker running) to check what your server is doing

Find out the name of your Docker container   
> docker ps

Execute your running container in a second window   
> docker exec -it <container_name> bash

Show the last 12 lines of the server output with email address for Swagger
> docker logs -f <container_name> --tail "12"

## 02 Server and Swagger

**Why do you need the server output?**

On the one hand it's good to notice when it's not running anymore to avoid errors, on the other hand it shows you some sample data. In particular a mail address and the password 123. Use this to login in to Swagger:

<http://127.0.0.1:8000/docs/>

**Why Swagger?**

Swagger shows you the API endpoints you generate ...

Response-request-cycle

02 Project Architecture   
03 API endpoints  
04 Test Data  
_041 Generate Test Data in the tests itself  
_042 Generate Test Data with Factories  
_043 Generate Test Data create_sample-data.py  
05 How to write a test  
06 How decorators are used  
07 Validators and Permissions  
08 Users, Members and Collectors  
09 History 
10? Signals
11? Filtering

another idea:
Video about hot to populate a database in django shell and display it in swagger /// maybe vs. navigate direcly through tables
