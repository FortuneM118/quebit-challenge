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


- **GitHub Submission**:
The easiest method is to fork this repository on GitHub and then clone it to your local machine.
- **Non-GitHub Submission**:
If you are not using GitHub, please ensure that your repository is hosted in a location where we can easily access it.
- **Work on the Challenge**:
Throughout the challenge, commit regularly to document your progress. Strive for structured, meaningful commits, with each one adding functionality in a coherent manner. Where possible, aim to keep individual commits small and concise, and try to break up your solutions into several small commits.
- **Submission Link**:
Once you have completed the challenge, email us a link to your repository at: cobus.louw@bytefuse.ai.

<span style="color: red;">NOTE: </span> **Be sure to watch the repo for bug fixes**

# Challenge

Quebit is currently deployed at an intersection in Stellenbosch, where we are collecting traffic data. For this task, we would like you to extract various insights from the data and process it to run a short simulation by the end of the challenge.

## Background

### Camera dataset

We have four FLIR cameras deployed at our data-collection site. Vehicle counts are streamed over websockets and stored in a database on AWS. The figures below illustrate the setup.

<div align="center">
  <div >
    <figure class="image">
      <img src="resources/sb.jpeg" width="300" alt="Screenshot 1">
    </figure>
    <figure class="image">
      <img src="resources/bk.jpeg" width="300" alt="Screenshot 2">
    </figure>
  </div>
  <div >
    <figure sclass="image"">
      <img src="resources/tp.jpeg" width="300" alt="Screenshot 3">
    </figure>
    <figure class="image">
      <img src="resources/sw.jpeg" width="300" alt="Screenshot 4">
    </figure>
  </div>
</div>

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


The following YAML allows us to map the detection zones of each camera to a turning movement. Be aware that some turning movements are counted by multiple detection zones/cameras.

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

## Challenge Questions/Problems

### 1. Query data from s3

Dumps of both the camera vehicle counts dataset and the controller's dataset have been uploaded to a public bucket on S3. 

The vehicle counts dataset is available at the following address: 

<span style="color: blue;"> s3://quebit-challenge/vehicle_counts.json </span>

The controller's dataset is available at the following address:

<span style="color: blue;"> s3://quebit-challenge/controller.json</span>

You need only use the data in the following timeframe: 
Start: `2024-03-08T17:00:00+02:00` End: `2024-03-08T18:00:00+02:00`

Both of these `.json` files are large, so it's recommended to query only the required subset. Consider using the AWS `SELECT` method to accomplish this task.


#### 1.1. Query vehicle counts
Let's start by querying the vehicle dataset for vehicle counts over the previously specified timeframe as well as for a subset of the detection zones. Remember that some turning movements are counted by multiple cameras and detection zones. The table below specifies which detection zones may be ignored for each of the cameras:


| Camera IP Address | Zones to ignore |
|---------------|-------------|
| 172.30.15.52  | 5, 6        |
| 172.30.15.53  | 1, 2, 3, 8  |
| 172.30.15.54  | 3, 4, 5, 6  |
| 172.30.15.55  | 7, 8        |


Only select rows within the specified time frame - `2024-03-08T17:00:00+02:00` to `2024-03-08T18:00:00+02:00`. Use the `time` column to make a selection based on time. The time format here is [ISO_8601](https://en.wikipedia.org/?title=ISO_8601).

#### 1.2. Query stage informations
Query the `controller.json` file for information regarding stage changes. Rows where the `oid` column have a value of `3.6.1.4.1.13267.3.2.5.1.1.3` contain information regarding stage selections. Again select only information within the specified timeframe. Use the `timestamp` column to select rows based time. Keep in mind that the format here is [Unix time](https://en.wikipedia.org/wiki/Unix_time).


### 2. Data processing and analysis

#### 2.1. Vehicle counts

- Create a bar plot of the total volume of cars served for each turning movement
- Create a rolling mean line plot of 30-second throughput for each turning movement. Here, throughput refers to the number of vehicles passing through the intersection.
- Construct a suitable visual to provide insight on the gap time distribution. Here, gap time refers to the time between consecutive vehicle counts.

#### 2.2. Traffic light visualization

- Visualize the distribution of green time extensions for each stage. Report and describe these distributions.
- Create a transition matrix to showcase the frequency at which the traffic light switched from one stage to another. After obtaining the transition matrix, use this matrix to construct a heatmap of the stage transitions.

#### 2.3 Vehicle counts and traffic light stages joined

- Implement a suitable method to combine (join) staging data with the vehicle count dataset. The resulting dataframe should assign a stage to each counted vehicle, indicating the stage that was active when the vehicle was observed.

- Both Stage 2 and Stage 4 served vehicles traveling from Somerset West to Technopark. Determine which stage served the most vehicles and which stage had the highest vehicle rate.

- Vehicles should only traverse the intersection during the designated stage corresponding to their turning movement. Identify trips (vehicle counts) recorded outside of valid periods, where the vehicle's route was not served during intersection crossing. That is, an invalid trip occurs when a vehicle passes through the intersection when it's route is not being served (e.g. vehicles driving over red lights). Report on the route and stage with the highest number of invalid trips.

- Analyze green time utilization. Identify the stage with the longest duration during which no vehicles were served, and report the duration of this period.


### 3. Create a simulation based on vehicle counts and traffic light stage data (bonus question)
   
In this question you are going to create a digital twin of the scenario using the provided dataset. We've already provided the simulation and dependencies. You need to create two `.xml` files. The first will be responsible for generating traffic within the simulation. The second `.xml` file is going to dictate the stage of the traffic light... 
