# Traffic Data Analysis Using PySpark and Cassandra

## Table of Contents
- [Overview](#overview)
- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Usage](#usage)
- [Queries and Insights](#queries-and-insights)
- [Data Source](#data-source)
- [Integration with Cassandra](#integration-with-cassandra)
- [License](#license)

## Overview

This project is a traffic data analysis application using **PySpark** for large-scale data processing and **Apache Cassandra** for data storage. The data consists of traffic records collected from multiple lanes, including the vehicle type, speed, and other metadata. The project includes multiple queries to generate insights about traffic distribution, rush hours, average speed by location, and more.

## Features

- **Data Ingestion**: Traffic data from multiple CSV files is loaded into PySpark for analysis.
- **Query Execution**: Spark SQL is used to execute queries and retrieve insights.
- **Data Storage**: Results of queries are saved in Cassandra.
- **Traffic Insights**: Insights include vehicle types, rush hour analysis, speed analysis, and vehicle distribution across various junctions and locations.

## Requirements

- Python 3.x
- Apache Spark
- Apache Cassandra
- PySpark
- Cassandra-Spark Connector
- Hadoop HDFS (optional for data storage)

## Installation

1. **Clone the Repository**
   ```bash
   git clone https://github.com/yourusername/traffic-analysis-pyspark.git
   cd traffic-analysis-pyspark
   ```

2. **Set Up Spark**
   Install Apache Spark and make sure it is added to your environment variables.

3. **Set Up Cassandra**
   Install Cassandra and create a keyspace named `assignment2`:
   ```cql
   CREATE KEYSPACE assignment2 WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1};
   ```

4. **Install Required Python Packages**
   ```bash
   pip install pyspark cassandra-driver
   ```

5. **Start the Cassandra Cluster**
   Make sure Cassandra is up and running on your local machine or on a specified host.

## Usage

1. **Run the PySpark Application**
   ```bash
   spark-submit traffic_analysis.py
   ```

2. **View Results in Cassandra**
   After running the PySpark application, the query results will be available in the `assignment2` keyspace within Cassandra.

## Queries and Insights

### Query 1: Vehicle Distribution by Class
This query groups vehicles by their class (e.g., cars, heavy goods vehicles) and calculates the percentage of each class.

```sql
SELECT classname, COUNT(classname) AS count, 
ROUND(COUNT(classname) * 100 / (SELECT COUNT(*) FROM tables), 1) AS percentage
FROM tables
GROUP BY classname
ORDER BY percentage DESC;
```

### Query 2: Hourly Traffic Volume
This query provides traffic counts by hour to determine peak traffic times during the day.

```sql
SELECT hour, COUNT(hour) AS count 
FROM tables 
GROUP BY hour 
ORDER BY count DESC;
```

### Query 3: Morning and Evening Rush Hours
- **Morning Rush (8 AM - 11 AM)**:
    ```sql
    SELECT hour, COUNT(hour) AS count 
    FROM tables 
    WHERE hour IN (8,9,10,11) 
    GROUP BY hour;
    ```
  
- **Evening Rush (5 PM - 8 PM)**:
    ```sql
    SELECT hour, COUNT(hour) AS count 
    FROM tables 
    WHERE hour IN (17,18,19,20) 
    GROUP BY hour;
    ```

### Query 4: Average Speed by Junction
This query calculates the average speed of vehicles at different junctions:

```sql
SELECT cosit, ROUND(AVG(speed), 1) AS avgspeed, q4jun.junction
FROM tables 
JOIN q4jun ON tables.cosit = q4jun.cosit 
GROUP BY tables.cosit, q4jun.junction;
```

### Query 5: Heavy Goods Vehicles Distribution by Location
This query filters and counts heavy goods vehicles (HGVs) across different locations:

```sql
SELECT DISTINCT tables.cosit, COUNT(tables.cosit) AS count, q5loclist.location
FROM tables 
JOIN q5loclist ON tables.cosit = q5loclist.cosit 
WHERE tables.classname = 'HGV_RIG' OR tables.classname = 'HGV_ART' 
GROUP BY tables.cosit, q5loclist.location 
ORDER BY count DESC;
```

## Data Source

The data used for this analysis consists of traffic records with the following fields:
- `cosit`: Unique identifier for the location.
- `year`, `month`, `day`, `hour`: Time details of the traffic record.
- `classname`: Vehicle class (e.g., CAR, LGV, HGV).
- `speed`: Vehicle speed in km/h.
- `lanename`: Lane from which the data is recorded.
- `weight`, `temperature`, `duration`: Additional metadata about the vehicle and environment.

## Integration with Cassandra

The project uses Apache Cassandra for storing query results. Cassandra tables are created dynamically, and data is saved using the PySpark Cassandra connector. The query results are written back into Cassandra using the following format:

```python
q1.write.format("org.apache.spark.sql.cassandra").options(table="q1", keyspace="assignment2").save(mode="append")
```

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
