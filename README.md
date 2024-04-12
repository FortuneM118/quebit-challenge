# Quebit Data Science Challenge
<p align="center">
<img src="resources/quebit.png" alt="Quebit Logo" class="center"/>
</p>

# Introduction

## Who we are

Quebit is a subsidiary of Bytefuse that specializes in transport and traffic light optimization. Our solutions are data-driven, and we continually gather substantial amounts of traffic data. As a result, we are seeking skilled data engineers and analysts to construct and maintain pipelines for managing, processing, and extracting insights from this data.

## Purpose of this challenge

The purpose of this challenge is to assess the programming skills of prospective Quebit Data Scientists and Engineers.

# Challenge overview & requirements

When completing this assignment, please adhere to the requirements listed below.

- **Use Python**: Please complete the assignment using Python, as it is the primary language used at Quebit.

- **Utilize Python Packages**: Feel free to leverage any Python package (e.g., pandas, numpy, etc.) during this challenge.

- **Structured Approach**: We seek candidates who can work in a structured manner and utilize version control effectively to allow easy rollback in case of breaking changes. Therefore, you should organize your code into logical directories and files to facilitate readability for reviewers.

- **Documentation and Commit Messages**: Document your functions thoroughly and use descriptive commit messages throughout the completion of the challenge.

- **Prioritize Readability**: We prioritize code readability over conciseness. Hence, feel free to use verbose variable and function names since most IDEs have autocomplete capabilities.

- **Logical and Efficient Data Handling**: Ensure that your code queries and processes data in a logical and efficient manner.

- **Statistical Insights**: Provide insightful descriptions of the results and graphs obtained during the challenge. That is, your solutions should contain explanations of all results produced.

- **Clear Instructions for Execution**: Include clear instructions in your final submission on how to install and run your code. Your submission can be in the form of a Jupyter Notebook or a Python script, but it should be executable by following the provided instructions. Specify the version of Python required to run your code. Note the use of `pip freeze` or `pipreqs` for generating a .txt file containing the requirements for executing your code.


# Submitting your solutions


- **GitHub submission**:
The easiest method is to fork this repository on GitHub and then clone it to your local machine.
- **Non-GitHub submission**:
If you are not using GitHub, please ensure that your repository is hosted in a location where we can easily access it.
- **Work on the challenge**:
Throughout the challenge, commit regularly to document your progress. Strive for structured and meaningful commits, with each commit adding functionality in a coherent manner. Where possible, aim to keep individual commits small and concise, and try to break up your solutions into several small commits. Although not necessary, have a look at the [gitmoji tags](https://gitmoji.dev/) which are used to add an emoji to commit messages for ease of readability (an example commit message that was used in one of our Quebit packages: `git commit -m ":bug: fix wandb artifact problem related to issue #372"`)
Once you have completed the challenge, email us a link to your repository at: cobus.louw@bytefuse.ai.

<span style="color: red;">NOTE: </span> **Be sure to watch the repo for bug fixes**

# Challenge

Quebit is currently deployed at an intersection in Stellenbosch, where we are collecting traffic data. For this task, we would like you to extract various insights from the data.

## Background

## Database
A sample dataset has been uploaded to a database on AWS. We will email you the information to connect to the DB. The database is equiped with 'readonly' permissions and you should not attempt to write to the database. You can connect to the database using a tool like DBeaver, pgAdmin4, SQLConnect, or any other tool of your choice. Use the tool to inspect the structure of the database and the tables in the database. Using Python and the Peewee Python package create database models that represent the structure of the database tables. Use these models to read and manupulate the data to complete the tasks below. Avoid typing out RAW SQL queries in your code. More background on the data follows in the next two sections.  

### Cameras

We have four FLIR cameras deployed at our data-collection site. These are mainly used for detecting vehicles. Vehicle counts are streamed over websockets and stored in the database in the `individual_data` table. The figures below illustrate the setup at the site.

<div align="center">
  <div style="display: flex; flex-wrap: wrap; justify-content: center;">
    <figure style="margin: 10px;">
      <img src="resources/sb.jpeg" width="200" alt="Screenshot 1">
      <!-- <figcaption>Stellenbosch | IP: 172.30.15.52</figcaption> -->
    </figure>
    <figure style="margin: 10px;">
      <img src="resources/bk.jpeg" width="200" alt="Screenshot 2">
      <!-- <figcaption>Blaauwklippen | IP: 172.30.15.53</figcaption> -->
    </figure>
  </div>
  <div style="display: flex; flex-wrap: wrap; justify-content: center;">
    <figure style="margin: 10px;">
      <img src="resources/tp.jpeg" width="200" alt="Screenshot 3">
      <!-- <figcaption>Technopark | IP: 172.30.15.54</figcaption> -->
    </figure>
    <figure style="margin: 10px;">
      <img src="resources/sw.jpeg" width="200" alt="Screenshot 4">
      <!-- <figcaption>Somerset West | IP: 172.30.15.55</figcaption> -->
    </figure>
  </div>
</div>


Counting zones are strategically placed to count the different turning movements at the intersection. Each row in this table represents a vehicle that was counted. The `detectorZone` column indicate the zone number where the vehicle was counted and the `time` column can be used to get the timestamp for when the count was made. Note that the counts of all 4 cameras are combined in this one table. 

There are 12 turning movements we want to track:
The counting zones (red boxes in the above figures) are strategically placed to count the different turning movements at the intersection. There are 12 turning movements we want to track:

```
stellenbosch_to_blaauwklippen
stellenbosch_to_somersetwest
stellenbosch_to_technopark

blaauwklippen_to_somersetwest
blaauwklippen_to_technopark
blaauwklippen_to_stellenbosch

somersetwest_to_technopark
somersetwest_to_stellenbosch
somersetwest_to_blaauwklippen

technopark_to_stellenbosch
technopark_to_blaauwklippen
technopark_to_somersetwest
```


The following YAML allows us to map the detector zones of each camera to a turning movement. Be aware that some turning movements are counted by multiple detection zones/cameras.

```yaml
172.30.15.52:
    "1": "stellenbosch_to_blaauwklippen"
    "2": "stellenbosch_to_somersetwest"
    "3": "stellenbosch_to_somersetwest"
    "4": "stellenbosch_to_somersetwest"
    "5": "stellenbosch_to_technopark"
    "6": "stellenbosch_to_technopark"
    "7": "blaauwklippen_to_somersetwest"
172.30.15.53:
    "1": "blaauwklippen_to_somersetwest"
    "2": "blaauwklippen_to_technopark"
    "3": "blaauwklippen_to_stellenbosch"
    "4": "technopark_to_somersetwest"
    "5": "technopark_to_somersetwest"
    "6": "stellenbosch_to_technopark"
    "7": "stellenbosch_to_technopark"
    "8": "_to_blaauwklippen" # only counts cars driving towards Blaauwklippen
172.30.15.54:
    "1": "technopark_to_stellenbosch"
    "2": "technopark_to_blaauwklippen"
    "3": "technopark_to_somersetwest"
    "4": "technopark_to_somersetwest"
    "5": "stellenbosch_to_technopark"
    "6": "stellenbosch_to_technopark"
    "7": "blaauwklippen_to_stellenbosch"
    "8": "somersetwest_to_blaauwklippen"
172.30.15.55:
    "1": "somersetwest_to_technopark"
    "2": "somersetwest_to_technopark"
    "3": "somersetwest_to_stellenbosch"
    "4": "somersetwest_to_stellenbosch"
    "5": "somersetwest_to_stellenbosch"
    "6": "somersetwest_to_blaauwklippen"
    "7": "somersetwest_to_technopark"
    "8": "somersetwest_to_technopark"
```

There are various other fields in this table. Use your Database tool or ORM to explore these.

### Traffic controller dataset

We are integrated with the traffic light controller at the site. As the name suggests, the controller is responsible for controlling the traffic light, which entails serving stages based on demand as well as extending stages to clear vehicle queues. Each stage defines a unique permutation of red and green lights corresponding to different turning movements.

The following stages are currently defined at the site:
<div align="center">
  <div style="display: grid; grid-template-columns: repeat(3, 1fr); grid-gap: 5px; width: 70%">
    <figure style="margin: 0;">
      <img src="resources/stage_1.png" width="200" alt="Stage 1">
    </figure>
    <figure style="margin: 0;">
      <img src="resources/stage_2.png" width="200" alt="Stage 2">
    </figure>
    <figure style="margin: 0;">
      <img src="resources/stage_3.png" width="200" alt="Stage 3">
    </figure>
    <figure style="margin: 0;">
      <img src="resources/stage_4.png" width="200" alt="Stage 4">
    </figure>
    <figure style="margin: 0;">
      <img src="resources/stage_5.png" width="200" alt="Stage 5">
    </figure>
    <figure style="margin: 0;">
      <img src="resources/stage_6.png" width="200" alt="Stage 6">
    </figure>
  </div>
</div>

As an example, stage 1 above serves each of the following turning movements:
  - vehicles entering Technopark from Stellenbosch
  - vehicles driving toward Somerset West from Stellenbosch, and
  - several turning movements for serving pedestrian traffic


The YAML below defines the turning movements that are served by each of the above stages:

```yaml
stage_route_map:
  1:
    - "stellenbosch_to_blaauwklippen"
    - "stellenbosch_to_somersetwest"
    - "stellenbosch_to_technopark"
    - "technopark_to_stellenbosch"
    - "_to_blaauwklippen"
  2:
    - "stellenbosch_to_blaauwklippen"
    - "stellenbosch_to_somersetwest"
    - "stellenbosch_to_technopark"
    - "somersetwest_to_blaauwklippen"
    - "somersetwest_to_stellenbosch"
    - "somersetwest_to_technopark"
    - "technopark_to_stellenbosch"
    - "_to_blaauwklippen"
  3:
    - "stellenbosch_to_technopark"
    - "somersetwest_to_blaauwklippen"
    - "technopark_to_stellenbosch"
    - "_to_blaauwklippen"
  4:
    - "technopark_to_blaauwklippen"
    - "technopark_to_somersetwest"
    - "somersetwest_to_technopark"
    - "technopark_to_stellenbosch"
    - "_to_blaauwklippen"
  5:
    - "technopark_to_blaauwklippen"
    - "technopark_to_stellenbosch"
    - "technopark_to_somersetwest"
    - "blaauwklippen_to_stellenbosch"
    - "blaauwklippen_to_somersetwest"
    - "blaauwklippen_to_technopark"
    - "_to_blaauwklippen"
  6:
    - "technopark_to_somersetwest"
    - "somersetwest_to_technopark"
    - "technopark_to_stellenbosch"
# technopark_to_stellenbosch is "served" by every stage as these vehicles use a slip and do not cross the intersection
```
---

Information from the traffic light is streamed using a SNMP Trap. The stream contains a variety of different messages. This includes information on when the traffic light changed stages, detector faults, detector readings etc. Data regarding the traffic light are located in the `trap_table`. A few important things to note:

- There are actually two traffic lights at this site. A main traffic light with `stream_site_id=51.51.50.48.57.54` and a secondary traffic light that controls a pedestrian crossing with `stream_site_id=51.51.50.48.57.55`. **Only** use data from the main traffic light i.e. `stream_site_id=51.51.50.48.57.54`.
- The `oid` column is used to distinguish different types of messages. We are only interested in stage confirmation bits. The message contains information of the current stage being served at the traffic light. The oid is `3.6.1.4.1.13267.3.2.5.1.1.3` .
- The timestamp column indicates the time when the message was received. This is [Unix time](https://en.wikipedia.org/wiki/Unix_time).
- The value of a message can be found in the `value` column. Stage information is given in hex and ascii. More information will follow in the questions section.

## Challenge Questions/Problems

### 1. Traffic volume

#### 1.1. Query vehicle counts
Use Peewee to write a query to select rows from the `individual_data` table within a timeframe of `2024-03-08T06:00:00+02:00` to `2024-03-08T17:00:00+02:00`. Recall that some turning movements are counted by multiple cameras and detector zones. The table below specifies the detector zones that should be ignored for each of the cameras.

| Camera IP Address | Zones to ignore |
|---------------|-------------|
| 172.30.15.52  | 5, 6        |
| 172.30.15.53  | 1, 2, 3, 8  |
| 172.30.15.54  | 3, 4, 5, 6  |
| 172.30.15.55  | 7, 8        |

#### 1.2 Map detector zones to turning movements
Use the information provided in the previous section to map the remaining detector zones of each camera to a turning movement (route). Use Python functions, classes or structures of your choice to obtain the resulting turning movements. Do not attempt to add the route information to the remote DB. It should be sufficient to store it locally in memory.

#### 1.3 Aggregate counts
It is sensible to resample these counts to look at volumes over longer timeframes. Resample the data in the `individual_data` table to obtain hourly counts for each route in the previously specified timeframe. Answer the following questions:
- Create a plot that describes the distribution of hourly vehicle counts for each turning movement. 
- What hour of the day had peak traffic (maximum counts) on the `somersetwest_to_technopark` turning movment? What was the volume of cars for that hour?

#### 1.4 Investigate speed information
1.4.1 We are interseted in the speed of vehicles driving from Stellenbosch to Somerset West i.e. `stellenbosch_to_somersetwest` turning movement. Create a suitable visualization to provide insight into this distribtion. Explain why you have chosen the specific data visualization.

1.4.2 What are some basic approaches to outlier detection? Implement some method to detect potential outliers in terms of vehicle speed. (Feel free to make distributional assumptions on vehicle speed for simplicity).

### 2. Traffic light

#### 2.1 Query stage information
Query the `trap_table` Table for information regarding stage changes. Recall that rows where the `oid` column have a value of `3.6.1.4.1.13267.3.2.5.1.1.3` contain information regarding stage selections. Again select only information within the specified timeframe. Use the `timestamp` column to select rows that fall within this timeframe. Keep in mind that the format here is [Unix time](https://en.wikipedia.org/wiki/Unix_time).

#### 2.2 Convert stage values to integers
Recall that the values of stages are given in hex or ascii. Use the `value` column to obtain these values and use the `value_type` column to determine if the value was given as a hex or a string. A few steps are required for converting these values to human-readable integers (e.g. stage 1 to stage 7). The steps are as follows:
- Convert the value to a binary number
- The resulting binary number should always be one-hot encoded; that is, only a single bit should take on the value 1, and the remaining bits take on the value 0 (with the exception of stage 0 where all bits take on a value of 0). Stage 0 also indicates an intergreen stage, i.e. the traffic light is transitioning from one green stage to the next. The index of the one-hot-encoded bit is the stage number. For example (`0000 0001` -> 1, `0000 0010` -> 2, `0000 0100` -> 3 and so on)
  
You can test your function by using the following table:
| Value Type | Value         | Stage Number |
|------------|-------------|--------------|
| HEX-String | 0x00        | 0            |
| HEX-String | 0x01        | 1            |
| HEX-String | 0x02        | 2            |
| HEX-String | 0x04        | 3            |
| HEX-String | 0x08        | 4            |
| HEX-String | 0x10        | 5            |
| STRING     | " "         | 6            |
| STRING     | @           | 7            |


#### 2. Visualise stage information

- Visualize the distribution of green time extensions for each stage.
- Create a transition matrix to showcase the frequency at which the traffic light switched from one stage to another (i.e. a matrix containing the frequency of stage transitions from stage $i$ to stage $j$ for each value of $i, j = 1, 2, \dots, 7, i\not=j$). After obtaining the transition matrix, use this matrix to construct a heatmap of the stage transition frequencies.
  

### 3 Join Vehicle counts and stages
- Implement a suitable method to combine (join) staging data with the vehicle count dataset. The resulting dataframe should assign a stage to each counted vehicle, indicating the stage that was active when the vehicle was observed.

- Both Stage 2 and Stage 4 served vehicles traveling from Somerset West to Technopark. Determine which stage served the most vehicles and which stage had the highest vehicle rate.

- Vehicles should only traverse the intersection during the designated stage corresponding to their turning movement. Identify trips (vehicle counts) recorded outside of valid periods, where the vehicle's route was not served during intersection crossing. That is, an invalid trip occurs when a vehicle passes through the intersection when it's route is not being served (e.g. vehicles driving over red lights). Report on the route and stage with the highest number of invalid trips.

- Analyze green time utilization. Identify the stage with the longest duration during which no vehicles were served, and report the duration of this period.


### 4. General questions:
1. What is a basic approach to approximate/estimate the distribution of a discrete random variable?
2. What are some approaches to visualizing high-dimensional data?
3. Suppose you fit a linear regression model to some training data and find that the model performs very well on this data (in the sense of having a small mean squared error), but when you evaluate the model on some held-out validation set, you find that the model performs very poorly (in the sense of having a very large mean squared error). What can you do to improve the generalisation of the model to unseen data?
4. Suppose you have a feature $x$ and a response variable $y$ that you wish to model using a linear regression model. Furthermore, suppose you visualize the relationship between $x$ and $y$ and observe a highly non-linear relationship, in which case a simple linear regression model will surely fail to provide an accurate fit to the data. How can you improve the model to account for the non-linearity, while still using a linear model?