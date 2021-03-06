lcmap-spark
===========

A simple, portable environment for executing science models and performing exploratory analysis at scale.

What is Spark?
--------------
`From the source <https://spark.apache.org/>`_, Apache Spark is a fast and general engine for large scale data processing.  It can run on a laptop or on thousands of machines, processes data too big to fit in memory, and moves functions to data rather than data to functions.

Spark has connectors to many data sources, offers interactive development and is open source.

Read more about Spark: https://spark.apache.org.

What is LCMAP-Spark?
--------------------
lcmap-spark is a ready to go Docker base image for the LCMAP Science Execution Environment (SEE).

It contains Apache Spark, the Spark-Cassandra Connector, and a Jupyter Notebook server to quickly allow science developers to get up and running on the LCMAP SEE.

A base set of data access and manipulation libraries (lcmap-merlin & numpy with MKL) are already installed, so time series creation works out of the box.  Conda and pip3 are configured and available for installing additional packages.

lcmap-spark provides a consistent and portable runtime environment: Applications developed on a laptop can be published and run at scale through simple configuration values with zero code changes.  No more worrying about scaling your applications.

Just write your application to use the Apache Spark API, test it, package it, publish it, then turn it loose on the SEE.

lcmap-spark uses Apache Mesos as its cluster manager for distributed computing.

Anatomy of A Spark Job
----------------------
1. Create SparkContext
2. Load and partition input data
3. Construct execution graph
4. Save calculation results
5. Shut down SparkContext

.. code-block:: python

   # Assumes read_timeseries_data, calculate_change_detection and save_to_cassandra
   # exist elsewhere in your codebase... they are not part of Spark.

   import pyspark

   # create Spark context
   sc = pyspark.SparkContext()

   # load and partition input data (10 partitions)
   rdd1 = sc.parallelize(read_timeseries_data(), 10)

   # construct execution graph
   rdd2 = rdd1.map(calculate_change_detection)

   # save calculation results
   save_to_cassandra(rdd2)

   # stop Spark context
   sc.stop()

Apache Spark builds a directed acyclic graph of functions to be applied against the input data and only begins executing these functions when an action, such as saving data to Cassandra, is performed.

The fundamental data structure used is a Resilient Distributed Dataset, which is a `"collection of elements partitioned across the nodes of the cluster that can be operated on in parallel." <https://spark.apache.org/docs/latest/rdd-programming-guide.html>`_.

The `laziness <https://en.wikipedia.org/wiki/Lazy_evaluation>`_ of RDDs is key because it allows Spark to avoid realizing the full dataset at once.  This means datasets much larger than available physical memory may be operated on.

Ways to Run
-----------
Spark jobs may be executed from a Jupyter Notebook, a Spark shell, or from the command line.

* ``spark-submit`` runs Spark jobs from a command line
* ``pyspark`` is a Python shell
* ``jupyter notebook`` is a Jupyter Notebook server

See https://spark.apache.org/docs/latest/quick-start.html and https://jupyter.org for more information.

Full examples with working configurations are in `running.rst <running.rst>`_.

.. code-block:: bash

    # Run any job from the command line
    docker run -it \
               --rm \
               --user=`id -u` \
               --net=host \
               --pid=host \
               usgseros/lcmap-spark:1.0 \
               spark-submit your_spark_job.py

    # Run Python jobs interactively from the PySpark shell
    docker run -it \
               --rm \
               --user=`id -u` \
               --net=host \
               --pid=host \
               usgseros/lcmap-spark:1.0 \
               pyspark

    # Run any job interactively from the Jupyter Notebook server
    docker run -it \
               --rm \
               --user=`id -u` \
               --net=host \
               --pid=host \
               --volume=/path/to/your/notebooks/:/home/lcmap/notebook/yours \
               usgseros/lcmap-spark:1.0 \
               jupyter --ip=$HOSTNAME notebook

               
Shippable Artifacts
-------------------
The shippable artifact for lcmap-spark is a Docker image published to https://hub.docker.com/r/usgseros/lcmap-spark/.

* Contains all code and libraries necessary to connect to LCMAP SEE
* Provides a consistent, immutable execution environment
* Is a base image, suitable for exploratory analysis or as starting points for derivative images

LCMAP SEE applications are independent software projects, publishing their own Docker images derived from lcmap-spark.


Modes
-----
There are two modes for lcmap-spark: ``cluster`` and ``local``.

* ``cluster`` mode executes Spark applications in parallel across many physical hosts
* ``local`` mode executes Spark applications on the local host system only
* Switching modes is achieved by setting parameters during SparkContext creation

               
