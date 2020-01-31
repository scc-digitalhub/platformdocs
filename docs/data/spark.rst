Apache Spark
===============

Spark (https://spark.apache.org/) is a general-purpose distributed data processing engine, 
suitable for use in a wide range of circumstances.

The official tagline is *unified analytics engine for large-scale data processing.*

On top of the its core data processing engine, 
Spark offers libraries for SQL, machine learning, graph computation, and stream processing,
which can be used together in an application. 

Programming languages supported by Spark include: Java, Python, Scala, and R. 

Application developers and data scientists integrate Spark into their applications
to rapidly query, analyze, and transform data at scale, also thanks to 
the availability of a number of high-level operators that make it easy to 
build and scale parallel applications.

Some of the tasks most frequently associated with Spark are:

- ETL and SQL batch jobs across huge data sets
- processing of streaming data from sensors (e.g. IoT)
- machine learning tasks
- complex analytics
- data integration



Spark is able to handle tasks which span several petabytes of data at a time, 
distributed across a cluster of thousands of physical or virtual servers. 

The main advantage of Spark is the ability to build complex and very performing
data pipelines, which combine very different techniques and processes into a single,
coherent whole. 
In modern cloud computing, the discrete tasks of acquiring, selecting, transforming 
and analyzing data usually requires the integration of many different systems and 
processing frameworks. Thanks to Spark, developers have the 
ability to combine these together, crossing the boundaries between batch, streaming, and interactive workflows.

Furthermore, Spark executes tasks as in-memory operations, with the option to *spill to disk* 
intermediate results when memory constrained. This approach guarantees a fast data pipeline,
easy to configure, run and maintain.