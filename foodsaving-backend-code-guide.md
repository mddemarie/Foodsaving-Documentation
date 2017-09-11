# foodsaving.world

Website: <https://foodsaving.world/>  
Model predecessor: <https://foodsharing.de>    
Repository on GitHub: <https://github.com/yunity/>

### Repository Structure

There are two separated Repos for Frontend and Backend.

**Frontend in JavaScript, Node.js**  
<https://github.com/yunity/foodsaving-frontend>

You don't need to do the setup for the frontend, but it might be useful to try out your backend through the frontend.

**Backend in Python Django REST**  
<https://github.com/yunity/foodsaving-backend>

- Python – object-oriented programming language  
- Django – Python framework (for faster web development)  
         Tutorial: <https://docs.djangoproject.com/en/1.11/intro/tutorial01/> 
- REST   – Framework on top of django for building Web APIs  
         Tutorial: <http://www.django-rest-framework.org/tutorial/1-serialization/>

*Both Repositories are not directly connected – the data exchange works via an API.*
         
## 01  Setup

First deside if you want to use Docker for the setup or try to install everything without it. Both options are described in the README.md in the foodsaving-backend repository. 

If you want to work with Docker, we would suggest to use __3 tabs in the shell__:

1. __Tab for Communicating with git / GitHub__ (doing that inside the docker container might raise errors)  
  
2. __Tab with Docker running__ for run manage.py commands:

	_After starting Docker your lines in the shell start with: `(env)`_

	Running tests:   
	`python manage.py test`

	_Please run the tests after your setup and every time you make a little change in code._

	After changing a model you have to migrate them:   
	`python manage.py makemigrations`    
	`python manage.py migrate`

	Leave Docker:   
	`exit`

3. __Tab with Docker running__ to check what your server is doing

	Find out the name of your Docker container:   
	`docker ps`

	Execute your running container in a second window:   
	`docker exec -it <container_name> bash`

	Show the last 12 lines of the server output with email address for Swagger:
	`docker logs -f <container_name> --tail "12"`

## 02 Project Architecture
### Foodsaving App Relationships in Backend
First of all, you have to have a group. You create the group of members. You can create and join a store as member (collector/user). Then, you can create a pickup event in future - this is called pickup-date (=one time event) or pickup-series (=repetitive event). As a member of group, you can join the pickup event, leave the pickup event before the event starts, miss the pickup event, delete the pickup event or do the pickup. Whatever you do with group (create/modify/join/leave), store (create/modify/delete) or pickup, will be moved from group, store or pickup and saved into history. You have also an option after food pickup to leave a feedback.

**Group** -> **Store** -> **Pickup-Date**/**Pickup-Series** -> (**Feedback**) -> **History**

You can play with this in Swagger or after Login to https://foodsaving.world

- Foodsaving Apps in Code
- Groups = groups of users
- Stores = space where is the food collected and offered for pickup
- History = Pickups in the past or any other actions (with Stores, Series, Groups) in the past
- UserAuth = authentification of user
- Users = reset password, change password etc.

#### Groups
As member of group you get a notification a few hours before pickup event you joined.

#### Stores
In models.py in Stores, you can find besides the data structure of Stores and Feedback also the PickupDate and PickupDateSeries. Both PickupDate and PickupDateSeries refer to pickup-date event and pickup-series event in Swagger and contain appropriate data fields. `PickupDateManager` with the method `process_finished_pickup_dates` is the most interesting class because it processes old pickups and moves them into History (even empty ones) - as result you find `PICKKUP_MISSED` or `PICKUP_DONE` in database.

#### History
In History you find any action regarding stores, groups or pickup-dates/pickup-series from the past. As result you find here different HistoryTypus (just “typus” in database), e.g. PICKUP_JOIN and additional data about that action. This helps to keep a track of all actions the user/member/collector has done.


## 03 Server and Swagger

#### Why do you need the server output?

On one hand it's good to notice when the server is not running anymore to avoid errors, on the other hand you can display with some sample data in Swagger and better understand relations between models. In particular, you use a mail address and the password 123. Use this to login in to Swagger:

<http://127.0.0.1:8000/docs/>

#### Why Swagger?

Swagger shows you the API endpoints you generate in models.py. One of the API endpoint is `pickup-dates`. You can there HTTP methods like `GET`, `POST`, `PATCH` and `DELETE`.

* **GET**: displays all data from database  
* **POST**: saves new data into database  
* **PATCH**: modifies data in database based on given id  
* **DELETE**: deletes data from database based on gived id

You can also add additional functionalities to your API endpoint like:

* **GET /api/..../{id}/**: displays all data from database based on given (e.g. pickup-date) id  
* **POST /api/..../{join}/**: the user/member joins the group/store/pickup  
* **POST /api/..../{leave}/**: the user/member can leave the group/store/pickup  
* any other functionality added to **GET**, **POST**, **PATCH** or **DELETE**   

Database is automatically populated with sample data as described above. But there are missing connections between: being user -> being member -> being collector -> pick up the food. You have to create these connections in Swagger for testing purposes.

> **TIP**: If you want, you can populate first the database writting some querysets in Django Shell and then look it up in Swagger. Or you can open PostgreSQL and populate the database there.

#### Response-request Cycle

Whenever you paste the url http://127.0.0.1:8000/docs/ into browser and hit Enter, you send a request to your local server sitting on your computer (live web sites have their own host server with domain). It depends if you want to GET data or POST data. The server (with own IP address) will use the given URL (with own IP address), execute some functionality on server with usage of data in database and will respond with view that you can see in your browser.


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
Video about how to populate a database in Django shell and display it in Swagger /// maybe with how to navigate direcly through tables
