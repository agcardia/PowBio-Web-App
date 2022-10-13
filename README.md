# Pow Bio Fermentation Web App

This is a fullstack web application that uses a [ReactJS](https://reactjs.org) frontend with a [Flask](http://flask.pocoo.org/) backend that executes fermentation scripts and collects fermentation data. The flask backend uses [Celery](https://docs.celeryq.dev/en/stable/) and [Redis](https://redis.io) to manage an asynchronous task queue to manage scripting and collecting tasks. 

# Installation

```bash
git clone https://github.com/POWBiosci/new_web_app.git
```

# Project Structure 

```bash
.
├── backend
│   ├── Dockerfile
│   ├── __init__.py
│   ├── dependencies
│   │   ├── FW_utils.py
│   │   ├── __init__.py
│   │   ├── reactor_class.py
│   │   └── run_experiment.py
│   ├── main.py
│   └── requirements.txt
├── docker-compose.yml
└── frontend
    ├── Dockerfile
    ├── package.json
    ├── public
    │   ├── index.html
    │   └── manifest.json
    ├── src
    │   ├── App.css
    │   ├── App.js
    │   ├── Components
    │   │   ├── FeedingScript
    │   │   │   ├── FeedingScript.css
    │   │   │   └── FeedingScript.jsx
    │   │   ├── Layout
    │   │   │   ├── TopBar.css
    │   │   │   └── TopBar.jsx
    │   │   └── RunExperiment
    │   │       ├── RunExperiment.css
    │   │       └── RunExperiment.jsx
    │   ├── index.css
    │   ├── index.js
    │   └── setupTests.js
    └── yarn.lock
```

# Build and Launch App

Make sure you are in the same directory that the github branch was cloned into (i.e if you clone into users/documents make sure you cd into users/documents before building app)

```bash
docker-compose up
```
The web app will now be acessible on [localhost:3000](http://localhost:3000)

# Starting an Experiment

![Text](/images/Experiment.png?raw=True)

To start an experiment fill out the form data shown in the image and click submit.
* Name: Experiment name, should be the three letter project code followed by the run number (ex. ABC002 for run 2 of project ABC)
* Reactors: The name of the reactors which will be running for the experiment, if more than one type them all in one line no space (ex. F1F2F3 for reactors F1,F2,F3)
* Time: The time in hours that the data collection script will be running for.
* Rate: The rate at which data is queried from Influx. (ex. 30 seconds will give a timepoint every 30 seconds for the dataframe)

# Starting a Feeding Script

![Text](/images/Script.png?raw=True)

To start a feeding script fill out the form data as follows.
* Mode: Feeding mode for reactor, the choices are fed-batch, continuous, exponential, and pulse feed.
* Reactor: The name of the reactor which will be running for the script, only type in one reactor!
* DO Setpoint: Dissolved oxygen setpoint for fermentation, given as a percent of maximum saturation 
* DO Spike Setpoint: The DO value at which to trigger feeding, (ex. 45% would be a hunger spike trigger for a 30% setpoint experiment)
* Temperature Setpoint: Temperature setpoint for experiment 
* pH Setpoint: pH setpoint for experiment
* Tubing: The type of tubing used for the substrate peristlatic pump, choices are Watson Marlow Bioprene 112, Masterflex LS-13, and Masterflex LS-14

Feed mode parameters

## Fed-Batch
* Flowrate: flowrate at which to pump in feed solution,in ml/hour 

## Continuous:
* µ: Dilution rate at which chemostat is being run at
* Volume: Tank volume at which reacto is kept constant at 

## Exponential:
* µ: Growth rate at which to run exponential feed at
* F_0: Initial flowrate that feed starts at
* Length: Length in hours of exponential feeding profile 

## Pulse:
* Flowrate: Rate at which to flow feed in during pulse (ml/hour)
* Length: Length of pulse in seconds
* Frequency: Frequency of pulse feeding in seconds

# Tracking Tasks

![Text](/images/Flower.png?raw=True)

To see if your data collection or feeding script has been properly recieved, go to [localhost:8888](http://localhost:8888) to acess the celery flower dashboard. This is a dashboard that tracks each tasks recieved and completed by the celery worker. The final status of each task is also shown to allow easier debugging.

# Contents
The app consists of five main parts, the web app, the frontend server, the task worker, the flower task tracker, and the redis broker. The five parts are bundled together in a docker-compose file that allows the entire application to be run as a docker container. 

# Web App
This web app consists of the flask application and the python and html code that generates the endpoints and defines the function behavior based on user specified inputs on the html forms. An example of creating a basic flask app is shown below.

```python
app = Flask(__name___)

@app.route('/',methods=['POST'])
def hello():
  input = request.json['input']
  return {'Your input is':input}
```

Routes define url endpoints for the app to access and the functions defined below explain what the app does when each endpoint is called (GET request) or queried (POST request) 

# Celery Worker

Celery is a system designed to process backgroudn tasks for web applications. It works by creating a worker that waits and listens for tasks from an app. Once the worker has recieved a task the function is executed in the background by the celery worker. 

The worker functions by connecting to a broker (Redis) that serves as the intermediary between the app and celery. The worker is created in the command line by the line 

```bash
celery -A main.celery worker --pool=eventlet --concurrency=12 --loglevel=INFO
```
Generally the amount of tasks a celery worker can run in parallel is limited by the number of CPU cores on the machine. However, because the tasks being computed are not computionally intensive we use eventlet as our worker pool, which lets us run our celery worker without being limited by CPU cores.

# Celery Flower

Celery flower is a package designed to track all celery worker tasks in a dashboard. This is implemented so we can see what tasks have been recieved, and of those tasks which have been succesfully completed or failed. 

# Redis

Redis is an open-source data structure store, and can serve as a cache and message broker for task queues. To create a redis server we simply pull the redis image on Docker and run it as a dependency for our web app and worker.

```yaml
 redis: 
    image: redis
    ports:
      - "6379:6379" 
```





