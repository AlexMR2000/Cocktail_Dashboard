# Cocktail Dashboard

Prepared by Alexander Reisenauer (Matr.Nr.: 03712872)

- Supervisor: Dr. Jürgen Mangler
- Chair: TUM School of Computation, Information and Technology (Prof. Rinderle-Ma)

## Architectural Design

### Overview

The main goal of this project was to develop a management dashboard for the cocktail robot developed by the TUM CIT chair based on a [CPEE](https://cpee.org/) model. The requirements for the project stipulate that the system should persist the logs resulting from the CPEE process in a SQLite database and, in the next step, be connected to a web-based dashboard via a Server-Sent Events (SSE) connection. In this way, this project provides an extensible and easily scalable system that gives the user feedback on the actions of one or more cocktail robots in real time. Further architectural details of this project can be found in the following graphic: 

![alt text](https://github.com/AlexMR2000/Cocktail_Dashboard/blob/main/docs/Cocktail_Dashboard_ArchitecturalDesign_Overview.jpg)

### Components

This project incorporates the following components: 

1. **Simulator:** The CPEE process `CPEE_Cocktail_Dashboard_Process.xml` serves as a simulator that generates data on the actions of the cocktail robot in its typical working environment. This means: setting up the robot, receiving and accepting orders, preparing and serving cocktails and switching off. Please refer to the next chapter for further information. 
2. **Logger:** The Python script `persist_logs_in_sqlite.py` serves as an interface between the logging interface of CPEE and the backend. Event logs that are generated while the simulator is running are parsed and persisted in a SQLite database (`log_database.db`) located on my account on the server [lehre.bpm.in.tum.de](https://lehre.bpm.in.tum.de/~ge49vav/).  
3. **Dashboard Backend:** The Python script `backend_cocktail_dashboard.py` includes a Flask Python web framework that prepares relevant logs as soon as new logs are written to the database. It therefore constantly searches the database for updates in order to send new data to the frontend (SSE).
4. **Dashboard Frontend:** The html script `frontend_cocktail_dashboard.html` visualizes the logs in a user-friendly and comprehensive way that allows developers and non-developers to get an overview of the status of one or more cocktail robots and various relevant statistics. The dashboard is limited to view-only functionalities, which meets the requirements of the project.

## Installation

1. Navigate and log into your account space on the server [lehre.bpm.in.tum.de](https://lehre.bpm.in.tum.de/)
2. Clone the repository with the following command:

`git clone https://github.com/AlexMR2000/Cocktail_Dashboard`

3. Install the required packages with the following command:

`pip install -r requirements.txt`

4. Create a SQLite database file on your account space on the server ["lehre.bpm.in.tum.de"](https://lehre.bpm.in.tum.de/) with the following commands:

`touch log_database.db`

`chmod 666 log_database.db`

5. If required: Adjust the link `db_path` in the scripts `persist_logs_in_sqlite.py` and `backend_cocktail_dashboard.py` to point to the database you created

6. Run the project with the following commands:

`python3 persist_logs_in_sqlite.py` to persist logs that are generated by the CPEE model in SQLite

`python3 backend_cocktail_dashboard.py` to set up and send the event logs to a web Dashboard 

7. The Dashboard can be accessed at the configured port.
8. Create a new instance on [CPEE](https://cpee.org/) and load the [CPEE model](https://github.com/AlexMR2000/Cocktail_Dashboard/blob/main/cpee_model/CPEE_Cocktail_Dashboard_Process.xml). Note that multiple instances can be created, which means that "multiple robots can work simultaneously".
9. Start one or more CPEE models and watch the Dashboard's magic.

## CPEE Model

The [CPEE model](https://github.com/AlexMR2000/Cocktail_Dashboard/blob/main/cpee_model/CPEE_Cocktail_Dashboard_Process.xml) serves as a simulator of a single cocktail robot and is designed to be a placeholder for more detailed (sub)processes in future projects. It is possible to start several processes in parallel, which would mean that several cocktail robots work simultaneously. Note that the process is based on local data elements that control the process, e.g. by determining the fill levels of ingredients or by randomly selecting cocktail orders. The process exists for demonstration purposes only. In this project, the basic functionalities of a cocktail robot are implemented, which consist of the following actions: 

### 1. Setting up the robot

![alt text](https://github.com/AlexMR2000/Cocktail_Dashboard/blob/main/docs/01_cpee_model_doc_setting_up_the_robot.png)

The process uses a random parameter to decide whether the robot builds up (boots) successfully or whether it "crashes" and shuts down (see Step 5). The latter results in a new reboot. If the robot boots successfully, the process continues to Step 2.  

### 2. Getting an order

![alt text](https://github.com/AlexMR2000/Cocktail_Dashboard/blob/main/docs/02_cpee_model_doc_getting_an_order.png)

The process "orders" a cocktail at random, i.e. in this example Gin Tonic, Negroni or Old Fashioned. There is also a chance that a cocktail will be ordered that is not on the "menu", i.e. for which no process/recipe is defined. In this case, refer to Step 4. If the cocktail is "on the menu", the process continues with Step 3. 

### 3. Preparing a cocktail or filling up the ingredients

![alt text](https://github.com/AlexMR2000/Cocktail_Dashboard/blob/main/docs/03_cpee_model_preparing_cocktail_or_fill_up_ingredients.png)

The system checks whether the ingredients are sufficient to make the cocktail ordered. If so, the cocktail recipe, as implemented into the process, is executed, whereby the addition of an ingredient, e.g. Angostura to an Old Fashioned, is a process step. Finally, each cocktail is "mixed and served". If the level of any of the required ingredients is insufficient, the process selects the alternative lane, i.e. rejecting the order and replenishing the ingredients. The process continues with Step 5.

### 4. Cocktail is unavailable

![alt text](https://github.com/AlexMR2000/Cocktail_Dashboard/blob/main/docs/04_cpee_model_cocktail_unavailable.png)

If the order from Step 2 results in a cocktail that is not available, i.e. for which no process is defined, the order is rejected and the process continues with Step 5.

### 5. Finishing an order and shutting down

![alt text](https://github.com/AlexMR2000/Cocktail_Dashboard/blob/main/docs/05_cpee_model_finishing_an_order_and_shutting_down.png)

In both cases, i.e. when an order has been successfully executed or rejected, the process ends the current order cycle by resetting various status variables that are required for the process to run in an endless simulation loop. Here too, the process randomly determines whether the robot can continue working, i.e. accepts the next order (jump to Step 2), or whether it "crashes" and has to be switched off. In the latter case, the robot shuts down and rebuilds itself (Step 1).  

To summarize, the CPEE model is designed to generate data for demonstration porpuses and to be easily scalable. As the dashboard is not based on the data elements of the process, but on the annotations associated with the different process elements, the dashboard is robust to changes in the names of individual data elements, for example. These 'annotations' are a CPEE feature enabling more efficient logging by abstracting single process steps. The dashboard backend and frontend heavily relies on these annotations as can be seen in the following chapters. 

## Demo with Images and Videos 

In the folder [docs](https://github.com/AlexMR2000/Cocktail_Dashboard/tree/main/cpee_model) there are useful images and videos documenting the functionalities of this project including a process and user perspective. Files that were to large to upload on Git can be accessed from the Sync&Share system of TUM (#TODO upload files and adjust name)

- The video (#TODO adjust name) documents how to install and set up the project
- The video (#TODO adjust name) documents how the CPEE process behaves (process perspective)
- The video (#TODO adjust name) documents how the Dashboard behaves as the CPEE model produces more and more data (user perspective)

## API Documentation

### Logger

The application serves as a logging service, capturing event data sent through HTTP POST requests to a specified endpoint ´/writelogtodb´. It processes the incoming data and stores it in a structured format within an SQLite database. The service is built using Flask, a micro web framework for Python, making it lightweight and suitable for both development and production environments.

#### Application Setup

The database is configured to store logs in a table named logs, containing various fields such as timestamps, event types, instance details, activities, and annotations among others. The ´setup_database´ function is responsible for creating this table if it does not already exist.

#### Endpoint 

The endpoint `/writelogtodb` accepts POST requests containing event log data. The data is expected to be in a form-encoded format with a key named notification holding a JSON string of the event details. The application parses this data, along with other form fields, to extract relevant information such as event timestamp, type, instance details, activity information, and annotations

#### Key Functions

- `get_db_connection()` establishes a connection to the SQLite database and sets the row factory to sqlite3.Row for convenient row access.
- `setup_database()` ensures the existence of the logs table within the database, creating it with the appropriate schema if necessary.
- `write_log_to_db()` is the main function for processing incoming POST requests. It extracts data from the request, formats it appropriately, and inserts it into the logs table in the database. In case of errors during database operations, it returns an HTTP 500 response.

#### Error Handling

The application includes basic error handling, particularly for database operations within the write_log_to_db function. Errors are caught, logged to the console, and result in an HTTP 500 response to indicate a server error.

### Dashboard Backend

The application is a Flask-based web application designed for real-time monitoring and analysis of cocktail-making robots. It fetches and streams data from an SQLite database to a dashboard, displaying statistics such as cocktail counts, ingredient usage, robot status, and failures. The application is structured around two main Flask routes: the dashboard view and a streaming endpoint for real-time updates. The dashboard serves as the user interface, displaying various statistics and the current status of cocktail-making robots. The stream endpoint continuously polls the database for new log entries, processes them, and sends updates to the dashboard via Server-Sent Events (SSE).

#### Application Setup

The application defines a global `Flask` instance and specifies the path to the SQLite database that contains the log data. It also initializes two dictionaries, `last_activity_by_uuid` and `first_out_of_order_time`, to track the latest activity timestamps and the first occurrence of out-of-order status for each robot, respectively.

#### Endpoints

- Dashboard (`/`) serves the main dashboard page by rendering a template named `frontend_cocktail_dashboard.html`. The dashboard is the user interface where the real-time statistics and robot statuses are displayed.
- The Stream (`/stream`) endpoint is a generator function wrapped in a Flask `Response` object with the MIME type set to `text/event-stream`, enabling SSE. This endpoint is responsible for continuously querying the database for new log entries, processing the data, and sending updates to the client in real-time.

#### Key Functions

The `generate` function is the core of the streaming endpoint. It enters an infinite loop, periodically querying the database for new logs since the last processed entry. The function updates various statistics, such as cocktail counts, ingredient usage, robot failures, and the current status of each robot. It then formats these statistics as JSON and yields them as SSE data to the client.

#### Real-time Data Processing

The application processes various types of events logged by the robots, such as cocktail orders (`calling` event type) and robot status updates. It tracks the number of cocktails made, the usage of individual ingredients, and instances of robot failures. The function also manages the status of each robot, marking robots as out of order if they have been inactive for more than a minute and removing them from the tracking list if the out-of-order status persists for more than five minutes.

#### Error Handling

The application includes basic error handling for database operations and JSON parsing. It ensures that the application continues to run even if individual log entries contain malformed data. The streaming function uses a `try-except` block around database operations to handle any potential exceptions gracefully.

### Dashboard Frontend

The dashboard serves as a real-time monitoring interface for tracking the performance and activities of a cocktail robot system. It displays metrics such as the number of cocktails made, ingredients used, robot failures, and the current status of robots. Additionally, it includes visual representations of data through charts.

#### Structure and Style

##### HTML Structure

The HTML document is structured into several main components:

- Header: Displays the title of the dashboard and the last reset timestamp.
- Main Content: Comprises an image container and a dashboard container. The dashboard container includes sections for displaying key metrics and charts.
- Footer: Shows the last update timestamp and copyright information.

##### CSS Styling

The styling defines a clean and modern look for the dashboard, with a focus on readability and clear data presentation. Key styling elements include:

- Background colors for the header and footer.
- Flexbox layout for positioning dashboard elements.
- Custom styles for dashboard metrics (cards) to highlight key data points.
- Chart containers styled to fit within the dashboard layout harmoniously.

#### Key Functions

##### JavaScript and EventSource

The dashboard leverages JavaScript to dynamically update its content based on real-time data streamed from the server:

- EventSource: Used to establish a persistent connection with the server, listening for updates sent to the /stream endpoint.
- Data Processing: Incoming data is parsed and used to update various parts of the dashboard, such as metrics, robot statuses, and charts.
- Charts: Utilizes Chart.js to display and update bar charts representing the number of cocktails made and the ingredients used.

##### Dynamic Updates

Key dynamic elements updated through JavaScript include:

- Cocktails Made and Ingredients Used: Metrics are displayed prominently and updated in real-time as data is received.
- Robot Failures: A counter indicating the number of times robots have encountered failures.
- Robot Status: Individual sections are dynamically added or removed based on the active status of each robot, displaying current activities or indicating if a robot is out of order.
- Charts: The cocktails and ingredients charts are updated with new data points as they arrive, providing a visual representation of the usage trends over time.

#### Real-time Data Processing

The dashboard is designed to handle real-time data efficiently:

- Event Handling: Listens for new data via the EventSource connection and updates the dashboard components accordingly.
- Error Handling: Includes basic error handling for the EventSource connection, logging errors to the console.
- Cleanup and Removal: Old data, such as inactive robot statuses, are removed from the dashboard to maintain relevance and clarity.
