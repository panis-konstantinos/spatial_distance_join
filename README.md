## Overview
A distributed Spark application for analyzing spatial proximity between data points of two large datasets using Apache Spark (PySpark) for the computations and Hadoop File System for file storage. Given two CSV files (denoted as R and S) containing data poins (id, longitude, latitude), the app computes:

* **queryA**: The number of point pairs (r in R, s in S) within a given distance ε 

* **queryB**: All points in R that have at least k neighbors from S within distance ε 

To address these queries, algorithms were designed and implemented using point grid partitioning techniques, aiming to reduce computational cost and avoid exhaustive pairwise comparisons. The implementation was based on the Spark DataFrame API and executed on real large-scale datasets. Through experimental evaluation, the performance of the algorithms was recorded for different parameter values (ε, k), highlighting the advantages of parallel processing and spatial proximity-based optimization. The results confirm the effectiveness of the approach in terms of both execution time and scalability.

## Dataset
The dataset consists of two tab-separated CSV files, namely RAILS and AREALM, which are subset of TIGER dataset. Each file contains approximately 3.5 million records with three columns: id, longitude and latitude. For exhibition reasons, a subset of each file has been uploaded to [data](data) folder.

## Development & Test Environment
The project was originally developed and tested on a 4-node cluster. Each node ran:

* **OS**: Ubuntu 16.04.3 LTS

* **Resources**: 4 CPU cores, 6 GB RAM, 10 GB disk

* **Software**: Apache Spark v3.5.1, Apache Hadoop v3.2.1

This setup was used to evaluate performance and scalability of the algorithms on real distributed infrastructure.
## Methodology
The algorithm was implemented in the Python programming language using the Spark API for Python, PySpark, with a focus on leveraging Spark’s parallel processing capabilities. Key goals included execution performance and scalability. Data processing was implemented using Spark DataFrames, in order to optimize both physical and logical execution plans (through the Catalyst Optimizer) and improve in-memory storage efficiency.

The algorithm consists of two main parts. The first part involves partitioning the points into groups by dividing the space into grid cells and assigning each point to a cell. At this stage, it is important to replicate points into neighboring cells that are within a distance less than or equal to the threshold ε, in order to ensure that no candidate pair (r, s) is missed. The second part concerns the join of the two datasets, R and S, to form candidate pairs and compute the distance between them. After the distance is calculated, filtering based on ε and k thresholds is straightforward.

## Experiment results
In order to assess the algorithm's efficiency we measured its execution time for different parameter values and available resources distribution. It is noted that execution time was measured from the start of the application to the computation of the final result. The time required to export the results to files in HDFS was not included in the measurements. You can find the detailed experiment results along with visualisations under the folder [experiment_results](experiment_results).

## How to run
In order to run the application run the following commands for queryA and queryB respectively in bash.

- **queryA** (eps=0.003, cellFactor=4):
```
/usr/local/spark/bin/spark-submit --master yarn --deploy-mode client --driver-memory 1g --executor-memory 3g --executor-cores 4 --num-executors 4 --conf spark.query.pathA="hdfs:///input/AREALM.csv" --conf spark.query.pathR="hdfs:///input/RAILS.csv" --conf spark.query.output="hdfs:///resultsA" --conf spark.query.eps=0.003 --conf spark.query.cellFactor=4 spatial_join.py
```

- **queryB** (eps=0.003, k=200, cellFactor=4):
```
/usr/local/spark/bin/spark-submit --master yarn --deploy-mode client --driver-memory 1g --executor-memory 3g --executor-cores 4 --num-executors 4 --conf spark.query.pathA="hdfs:///input/AREALM.csv" --conf spark.query.pathR="hdfs:///input/RAILS.csv" --conf spark.query.output="hdfs:///resultsB" --conf spark.query.eps=0.003 --conf spark.query.cellFactor=4 --conf spark.query.k=200 spatial_join.py
```

You can find the query results of the above executions under the folder [output](output).

**NOTE**: This project assumes a working Spark and Hadoop environment. If you don't have one, consider using a single-node setup with Spark standalone for testing purposes.
