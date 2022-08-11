---
title: Big Data Analysis on Million Song Dataset
date: 2022-08-12 00:49:00 +0800
categories: [project]
tags: [big-data, MapReduce, Hadoop, Spark, Drill, Avro, HDFS, query, database, Python]
image:
  src: /assets/img/blog/MSD-analysis/workflow.png
  width: 1000 # in pixels
  alt: project workflow
---

This project is a course project in *ECE4721J Methods and Tools for Big Data* [@UM-SJTU Joint Institute](https://www.ji.sjtu.edu.cn/){:target="\_blank"}. In general, we perform big data analysis with several tools and algorithms on [Million Song Dataset](http://millionsongdataset.com){:target="\_blank"}.

You can find the source code in my [Github repo](https://github.com/kx-Huang/ECE4721J/tree/master/project/pj_1){:target="\_blank"}.

---

# Project Goals

- Work with Hadoop, Drill and Spark
- Compare MapReduce and Spark
- Retrieve specific information in big data
- Perform advanced data analysis on big data

Table of content:

  - [Milestone 0: HDF5 Data Pre-process](#milestone-0-hdf5-data-pre-process)
  - [Milestone 1: Drill Database Query](#milestone-1-drill-database-query)
  - [Milestone 2: Advanced Data Analysis](#milestone-2-advanced-data-analysis)

---
## Milestone 0: HDF5 Data Pre-process

### 0.1 Compact small `hdf5` files into larger one

```bash
python3 create_aggregate_file.py <IN> <OUT>
```

- Input: a directory contains `hdf5` song files

- Output: an aggregate `hdf5` song file

![Compact 10000 `hdf5` files into larger one](/assets/img/blog/MSD-analysis/compact.jpeg)

---

### 0.2 Read `hdf5` file and extract the information

```bash
python3 display_song.py [FLAGS] <HDF5> <idx> <field>
```

- Input: an `hdf5` song file

- Output: specified field content

![Get artist name of the second song in compacted `hdf5` file](/assets/img/blog/MSD-analysis/extract.png)

---

### 0.3 Convert `hdf5` to `Avro` with `Apache Avro™`

```bash
hdf5_to_avro.py [-h] -s <SCHEMA> -i <HDF5> -o <AVRO>
```

- Input:

  - an `Avro` schema file

  - an `hdf5` song file to be converted

- Output: an `Avro` song file

- Sample schema file in `json` format:

```json
  {
    "namespace": "song.avro",
    "type": "record",
    "name": "Song",
    "fields": [
      {
        "name": "artist_name",
        "type": ["string", "null"]
      },
      {
        "name": "title",
        "type": ["string", "null"]
      }
    ]
  }
```

![Convert compacted `hdf5` file to `Avro`](/assets/img/blog/MSD-analysis/convert.jpeg)

---

## Milestone 1: Drill Database Query

Query [Million Song Dataset (MSD)](http://millionsongdataset.com) with `Drill`:

### 1.1 Find the range of dates covered by the songs in the dataset

- SQL:

  ```sql
  -- Age of the oldest songs
  SELECT 2022 - MAX(year) AS Age
  FROM hdfs.`/pj/m0/output.avro`;

  -- Age of the youngest songs
  SELECT 2022 - MIN(year) AS Age
  FROM hdfs.`/pj/m0/output.avro`
  WHERE year > 0;
  ```

- Results:

  ```log
  +--------+                          +--------+
  |  Age   |                          |  Age   |
  +--------+                          +--------+
  | 12     |                          | 96     |
  +--------+                          +--------+
  1 rows selected (0.864 seconds)     1 rows selected (0.642 seconds)
  ```

The oldest song's age is **96** and the youngest is **12**.

As a result, the range of dates covered by the songs is **84** years.

---

### 1.2 Find the hottest song that is the shortest and shows highest energy with lowest tempo

```log
+------------------------------------------------------+
|                        title                         |
+------------------------------------------------------+
| b'Immigrant Song (Album Version)'                    |
| b"Nothin' On You [feat. Bruno Mars] (Album Version)" |
| b'This Christmas (LP Version)'                       |
| b'If Today Was Your Last Day (Album Version)'        |
| b'Harder To Breathe'                                 |
| b'Blue Orchid'                                       |
| b'Just Say Yes'                                      |
| b'They Reminisce Over You (Single Version)'          |
| b'Exogenesis: Symphony Part 1 [Overture]'            |
| b'Inertiatic Esp'                                    |
+------------------------------------------------------+
10 rows selected (0.471 seconds)
```

---

### 1.3 Find the name of the album with the most tracks

- SQL:

  ```sql
  SELECT release, COUNT(release) AS NumTrack
  FROM hdfs.`/pj/m0/output.avro`
  GROUP BY release
  ORDER BY NumTrack desc
  LIMIT 1;
  ```

- Results:

  ```log
  +------------------+----------+
  |     release      | NumTrack |
  +------------------+----------+
  | b'Greatest Hits' | 21       |
  +------------------+----------+
  1 row selected (0.695 seconds)
  ```

---

### 1.4 Find the name of the band who recorded the longest song

- SQL:

  ```sql
  SELECT artist_name, duration
  FROM hdfs.`/pj/m0/output.avro`
  ORDER BY duration DESC
  LIMIT 1;
  ```

- Results:

  ```log
  +-------------+-----------+
  | artist_name | duration  |
  +-------------+-----------+
  | b'UFO'      | 1819.7677 |
  +-------------+-----------+
  1 row selected (0.27 seconds)
  ```

---

## Milestone 2: Advanced Data Analysis

### 2.1 BFS with MapReduce - A Simple Example

Let's say we want to find artists similar to **A** with distance **3**, and we have the following relationship graph (each edge has distance **1**)

![Similarity Graph](/assets/img/blog/MSD-analysis/graph.png){: width="40%"}

With **n** MapReduce BFS iterations, we can get similar artist with distance **n**.

---

#### 2.1.1 Initialize Graph File with Target Artist

**Format**: each line contains `Node | Distance | Neighbours`

![Initialize Graph File](/assets/img/blog/MSD-analysis/graph-init.png)

---

#### 2.1.2 Generate Distance Pairs in Mapper

**Mapper**: Generate **Neighbours, Node, Distance+1** if not itself

![Mapper in Iteraion 1](/assets/img/blog/MSD-analysis/mapper-1.png)

---

#### 2.1.3 Merge Distance Pairs in Reducer

**Reducer**: Merge the same **Neighbour**, keep distance **minimum**

![Reducer in Iteraion 1](/assets/img/blog/MSD-analysis/reducer-1.png)

---

#### 2.1.4 Iteraiton 2: Mapper

![Mapper in Iteraion 2](/assets/img/blog/MSD-analysis/mapper-2.png)

---

#### 2.1.4 Iteraiton 2: Reducer

![Reducer in Iteraion 2](/assets/img/blog/MSD-analysis/reducer-2.png)

---

### 2.2 BFS with Spark

- Same algorithm as is proposed before

- Implemented using Python with `PySpark`

- For the implementation, we:

    1. Convert data into RDD map

    2. Using Spark `sortByKey()` to sort RDD aggregated by node index and then combining the neighbours

    3. Using Spark `reduce()` to pick the minimum distance of different neighbours towards the central node

- Spark is assumed to be faster since subsequent steps are retained in memory with a trade-off of much more memory consumption

---

### 2.3 Benchmark

- Data size: around **20GB**

- Server: SJTU cluster with three machines
  - CPU: Dualcore Intel Xeon Processor (Skylake, IBRS)
  - Memory: 4GB

|                     MapReduce: around **250s**                      |                    Spark: around **45s**                    |
| :-----------------------------------------------------------------: | :---------------------------------------------------------: |
| ![MapReduce Time](/assets/img/blog/MSD-analysis/time-mapreduce.png) | ![Spark Time](/assets/img/blog/MSD-analysis/time-spark.png) |

---

## Reference

1. Million Song Dataset. [`http://millionsongdataset.com`](http://millionsongdataset.com){:target="\_blank"}

2. Apache Avro Documentation. [`https://avro.apache.org/docs/current/index.html`](https://avro.apache.org/docs/current/index.html){:target="\_blank"}

3. Apache Spark Documentation. [`https://spark.apache.org/docs/latest/`](https://spark.apache.org/docs/latest/){:target="\_blank"}


**Thanks for your attention!**

![Never Gonna Give You Up (by Rick Astley)](/assets/img/blog/MSD-analysis/thank-you.png){: width="75%"}
