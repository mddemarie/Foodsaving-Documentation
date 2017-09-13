# foodsaving.world

Website: <https://foodsaving.world/>  
Model predecessor: <https://foodsharing.de>    
Repository on GitHub: <https://github.com/yunity/>

### Repository Structure

There are two separated Repos for Frontend and Backend.

**Karrot-Frontend in JavaScript, Node.js**  
<https://github.com/yunity/karrot-frontend>

You don't need to do the setup for the frontend, but it might be useful to try out your backend through the frontend.

**Karrot-Backend in Python Django REST**  
<https://github.com/yunity/karrot-backend>

- Python – object-oriented programming language  
- Django – Python framework (for faster web development)  
         Tutorial: <https://docs.djangoproject.com/en/1.11/intro/tutorial01/> 
- REST   – Framework on top of django for building Web APIs  
         Tutorial: <http://www.django-rest-framework.org/tutorial/1-serialization/>

*Both Repositories are not directly connected – the data exchange works via an API.*
         
## 01  Setup

We use Docker for the setup. How to build a Docker container is described in the README.md in the [karrot-backend repository](https://github.com/yunity/karrot-backend). 

We would suggest to use __3 tabs in the shell__:

1. __Tab for Communicating with git / GitHub__ (doing that inside the docker container might raise errors)  
  
2. __Tab with Docker running for run manage.py commands__

	Find out the name of your Docker Container: `docker ps`  
	_(examples: `young_curie` or `amazing_lovelace`)_

	Run Docker with: `docker exec -it <container_name> bash`  
	_(After starting Docker your lines in the shell start with: `(env)`)_
	
	Running __tests__:   
	
	`python manage.py test`  
	_(Please run the tests after your setup and every time you make a change in code.)_

	After changing a model you have to __migrate__ them:   
	
	`python manage.py makemigrations`    
	`python manage.py migrate`

	Leave Docker: `exit`

3. __Tab with Docker running to check what your server is doing__

	Show the last 12 lines of the server output:  
	`docker logs -f <container_name> --tail "12"`
	
	_Note: The first line shown is an email address. Store it – we will need it for Swagger._

## 02 Project Architecture
### Relationships in Backend

First of all, you have to have a `Group` Model, allowing to create objects like "Foodsavers Berlin". One Group usually has may `Stores`, like "Bakery Smith" and each store can define events where foodsavers can come by and save food. These events are called `PickupDate` (=one time event) or `PickupDateSeries` (=repetitive event).

![core elements of foodsaving backend](material/foodsaving-core-elements.jpg)

As loggin-in user you can create and join a Group. Then, you can join or create a PickupDate event in the future. Futher actions are for example:

- for member in `Group`: create/modify/join/leave
- for member in `Store`: create/update/delete
- for collector in `PickupDate`/`PickupDateSeries`: create/join/update/delete

Users have also an option after food pickup to leave a feedback.

### Foodsaving Apps

At the moment (September 2017) there are 15 Apps (= Folders) in foodsaving. Not all of them are in use or critical for foodsaving.world since the project is under development and the dev team tries different approaches. 

Important are for example: `groups` (see above), `users` and `userauth` (authentification, reset user password, change password etc.), `base` (most models in the code inherit from the models created there) and `tests` (the test coverage is very high - some of the tests are in the test app – others in the other apps). `stores` and `history` need a bit more explanation:

#### Stores
In models.py in `stores`, you can find besides the data structure of `Stores` and `Feedback` also the `PickupDate` and `PickupDateSeries`. They refer to pickup-date event and pickup-series event in Swagger (see desciption in the chapter "Server and Swagger") and contain appropriate data fields. `PickupDateManager` with the method `process_finished_pickup_dates` is an interesting class because it processes old pickups and moves them into `history` (even empty ones) - as result you find `PICKKUP_MISSED` or `PICKUP_DONE` in database.

#### History
In `history` you find any action regarding stores, groups or pickup-dates/pickup-series from the past. As result you find here different HistoryTypus (just “typus” in database), e.g. PICKUP_JOIN and additional data about that action. This helps to keep a track of all actions the user/member/collector has done.

### Stores app in detail

We want to dig a bit deeper in the app stores a) to give you an example how the foodsaving apps work and b) because there is a lot functionality inside that you might like to know. If you didn't already opened the code in your editor: do it now! Open the stores app and have a look on the files:


1. __models.py__ Here you define which database tables you want to have and what the fields/columns should store in the database. One model (or class) defines one database table. Let's have a look on the Model `Feedback` which creates four database fields (and two fields for the id and a time stamp, but these are created automatically here). The following line creates a field with the name `comment`.

	`comment = models.CharField(max_length=settings.DESCRIPTION_MAX_LENGTH, blank=True)`  
	
	The type [CharField](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.CharField) says that `comment` will be stored as string in database. That string could be maximal as long as defined in the file settings.py under `DESCRIPTION_MAX_LENGTH`. The entry can be saved even if the comment field is `blank`.


2. __serializers.py__ The models we created in models.py are Python objects. But these are not very useful for other users in the API – so we convert them into JSON data format with serializers. Our Feedback model has a `FeedbackSerializer` which inherits a lot of functions from ModelSerializers. But there are also new functions like `validate_about`. This validator checks if an user is allowed to give feedback to the pickup under certain circumstances (user is a member of group, that member joined the pickup and the pickup is in the future). 

3. __permissions.py__ Another possibility to check if something is allowed are permissions. They are imported to (and used in) `api.py`. Here is for example the permission `IsNotFull` that permits a member to join the pickup event only if it is not full.

4. __api.py__ The api defines how the data stored in the database can be accessed via API. The used HTTP methods (like `GET`, `CREATE` or `JOINED`) are described in the capter _03 Server and Swagger_.

	Instead of normal `Views` we use whole `ViewSets` which allow to combine the logic for a set of related views. Have a look on the class `FeedbackViewSet`. You will notice that most HTTP methods (like `GET`) are not defined there but in an imported [mixin](http://www.django-rest-framework.org/api-guide/generic-views/#mixins). Each mixin contains whole logic for creating a single HTTP request. The ViewSets are connected with `urls.py` and defined there in form of an url.

5. __factories.py__ In a Factory you can create sample data used in the tests.

6. A folder with __tests__ The test coverage of the project is very good and [Circle CI](https://circleci.com/) will answer in angry red if you try to push untested and non-funtioning code. 

	Have a look on the class FeedbackTest in `test_feedback_api.py`. First we create all data we need in the setUpClass - we generate here the test data with factories and whole logic of being user, member and collector. Then we test step by step if the expected result is `assertEqual` to the actual result. (The chapter '01 Setup' explains how to run the tests in the shell.)
	
7. A folder with __migrations__: You don't have to care about them a lot here. They are generated automatically when you run `python manage.py makemigrations` in the shell with Docker active.

Please also have a look on the used urls in _config/urls.py_ and on the archive functions in _foodsaving/history_. 



## 03 Server and Swagger

#### Why do you need the server output in the shell?

On one hand it's good to notice when the server is not running anymore (to avoid errors), on the other hand you can display with some sample data in Swagger and better understand relations between models. In particular, you use a mail address (see 3. Tab in 01 Setup chapter) and the password 123. Use this to login in to Swagger in your browser:

<http://127.0.0.1:8000/docs/>

#### Why Swagger?

[Swagger](https://swagger.io/docs/specification/about/) shows you the API endpoints that are defined in the api.py files in the apps groups, stores etc. One of the API endpoint is `pickup-dates`. 

You can use [HTTP methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods) like:

* **GET**: list all data from database  
* **POST**: submits new entry into database  
* **PATCH**: modifies one entry in database based on given id  
* **DELETE**: deletes one entry from database based on gived id

You can also add additional functionalities to your API endpoint like:

* **GET /api/..../{id}/**: displays one entry from database based on given (e.g. pickup-date) id  
* **POST /api/..../{join}/**: the user/member joins the group/store/pickup  
* **POST /api/..../{leave}/**: the user/member can leave the group/store/pickup  
* any other functionality added to **GET**, **POST**, **PATCH** or **DELETE**   

The Database is automatically populated with sample data if you use Docker. But there are missing connections between: being user -> being member -> being collector -> pick up the food. You can create these connections in Swagger for testing purposes.

> **TIP**: If you want, you can populate first the database writting some [querysets](https://docs.djangoproject.com/en/1.11/topics/db/queries/) in Django Shell and then look it up in Swagger. Or you can [open PostgreSQL](https://www.postgresql.org/docs/8.3/static/tutorial-accessdb.html) and populate the database there.

#### Response-request Cycle

Whenever you paste the url [http://127.0.0.1:8000/docs/](http://127.0.0.1:8000/docs/) into browser and hit Enter, you send a request to your local server sitting on your computer (live web sites have their own host server with domain). It depends if you want to GET data or POST data. The server (with own IP address) will use the given URL (with own IP address), execute some functionality on server with usage of data in database and will respond with view that you can see in your browser.



## Index
03 API endpoints  
04 Test Data  
	_ 041 Generate Test Data in the tests itself  
	_ 042 Generate Test Data with Factories  
	_ 043 Generate Test Data create_sample-data.py  
	
05 How to write a test  
06 How decorators are used  
07 Validators and Permissions  
08 Users, Members and Collectors  
09 History 
10? Signals
11? Filtering
12? How to contribute to the project in general

another idea:
Video about how to populate a database in Django shell and display it in Swagger /// maybe with how to navigate direcly through tables




## Delete?

__Foodsaving Apps in Code__ _that is already explained in the text above, is it???_
 
- Groups = groups of users
- Stores = space where is the food collected and offered for pickup
- History = Pickups in the past or any other actions (with Stores, Series, Groups) in the past
- UserAuth = authentification of user
- Users = reset password, change password etc.

#### Groups
As member of group you get a notification a few hours before pickup event you joined. _do we really need that???_
