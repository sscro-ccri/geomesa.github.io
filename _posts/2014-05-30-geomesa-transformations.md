---
title: GeoMesa Transformations
author: emilio
layout: tutorial
---

{% include tutorial-header.html %}

### Background

GeoMesa allows users to perform [relational projections](http://en.wikipedia.org/wiki/Projection_%28relational_algebra%29) on query results.  We call these 'transformations' to distinguish them from the overloaded term 'projection' which has a different meaning in a spatial context.  These transformations have the following uses and advantages

1. Subset to specified columns - reduces network overhead of returning results
2. Rename specified columns - alters the schema of data on the fly
3. Compute new attributes from one or more original attributes - adds derived fields to results

The transformations are applied in parallel across the cluster thus making them very fast.  They are analogous to the map tasks in a map-reduce job.  Transformations are also extensible; developers can implement new functions and plug them into the system using standard mechanisms from Geotools.  

### This tutorial will show you how to:

Write custom Java code using GeoMesa to do the following:

1.  query previously-ingested data
2.  apply relational projections to your query results
3.  apply transformations to your query results

**Note:** when this tutorial refers to ```projections```, it means in the relational sense - see [Projection - Relational Algebra](http://en.wikipedia.org/wiki/Projection_(relational_algebra)). Projection also has [many other meanings](http://en.wikipedia.org/wiki/Projection_(disambiguation)) in spatial discussions - they are not used in this tutorial.

Although projections can also modify an attribute's value, in this tutorial we will refer to such modifications as ```transformations``` to keep things clearer.

<div class="callout callout-warning">
    <span class="glyphicon glyphicon-exclamation-sign"></span>
    You will need access to a Hadoop 2.2 installation as well as an Accumulo 1.5.x database.
</div>

<div class="callout callout-warning">
    <span class="glyphicon glyphicon-exclamation-sign"></span>
    You will need to have ingested the GDELT data set using GeoMesa. Instructions are available <a href="/2014/04/17/geomesa-gdelt-analysis/">here</a>.
</div>

#### Other prerequisites

Before you begin, you should also have these:

* a locally-built version of GeoMesa - see [GeoMesa Quickstart](/2014/05/28/geomesa-quickstart/) - (DOWNLOAD AND BUILD GEOMESA)
* an Accumulo user that has appropriate permissions to query your data
* a local copy of the Java Development Kit 1.7.x
* Apache Maven installed
* a GitHub client installed

### Download and Build the Tutorial Code

Pick a reasonable directory on your machine, and run:

```
git clone git@github.com:geomesa/geomesa-tutorial-transformations.git
```

The ```pom.xml``` file contains an explicit list of dependent libraries that will be bundled together into the final tutorial. You should confirm that the versions of Accumulo and Hadoop match what you are running; if it does not match, change the value in the POM. (NB: The only reason these libraries are bundled into the final JAR is that this is easier for most people to do this than it is to set the classpath when running the tutorial. If you would rather not bundle these dependencies, mark them as provided in the POM, and update your classpath as appropriate.)

From within the root of the cloned tutorial, run:

```
mvn clean install
```

When this is complete, it will have built a JAR file that contains all of the code you need to run the tutorial.

### Run the Tutorial

On the command-line, run:

```
java -cp ./target/geomesa-tutorial-transformations-1.0.jar geomesa.tutorial.QueryTutorial -instanceId <instance> -zookeepers <zoos> -user <user> -password <pwd> -tableName <table> -featureName <feature>
```

where you provide the following arguments:

* ```<instance>``` - the name of your Accumulo instance
* ```<zoos>``` - comma-separated list of your Zookeeper nodes, e.g. zoo1:2181,zoo2:2181,zoo3:2181
* ```<user>``` - the name of an Accumulo user that will execute the scans, e.g. root
* ```<pwd>``` - the password for the previously-mentioned Accumulo user
* ```<table>``` - the name of the Accumulo table that has the GeoMesa GDELT dataset, e.g. gdelt
* ```<feature>``` - the feature name used to ingest the GeoMesa GDELT dataset, e.g. gdelt

You should see several queries run and the results printed out to your console.

### Insight Into How the Tutorial Works

The code for querying and projections is available in the following class:

* ```geomesa.tutorial.QueryTutorial```

The source code is meant to be accessible, but here is a high-level breakdown of the relevant methods:

* basicQuery - executes a base filter without any further options. All attributes are returned in the data set.
* basicProjectionQuery - executes a base filter but specifies a subset of attributes to return.
* basicTransformationQuery - executes a base filter and transforms one of the attributes that is returned.
* renamedTransformationQuery - executes a base filter and transforms one of the attributes, returning it in a separate derived attribute.
* mutliFieldTransformationQuery - executes a base filter and transforms two attributes into a single derived attributes.
* geometricTransformationQuery - executes a base filter and transforms the geometry returned from a point into a polygon by buffering it. 

Additional transformation functions are listed [here](http://docs.geotools.org/latest/userguide/library/main/filter.html) (scroll to 'Function List').
*Please note that currently not all functions are supported by GeoMesa.*

Additionally, there are two helper classes included in the tutorial:

* ```geomesa.tutorial.GdeltFeature``` - Contains the properties (attributes) available in the GDELT data set.
* ```geomesa.tutorial.SetupUtil``` - Handles reading command-line arguments

### Sample Code

The following code snippets show the basic aspects of creating queries for GeoMesa.

#### Create a basic query with no projections

{% highlight java %}
Query query = new Query(simpleFeatureTypeName, cqlFilter);
{% endhighlight %}

#### Create a query with a projection for attributes 'a' and 'b'

{% highlight java %}
String[] properties = new String[] {"a", "b"};
Query query = new Query(simpleFeatureTypeName, cqlFilter, properties);
{% endhighlight %}

#### Create a query with a transformation for attribute 'a'

{% highlight java %}
String[] properties = new String[] {"a=strConcat('hello ', a)"};
Query query = new Query(simpleFeatureTypeName, cqlFilter, properties);
{% endhighlight %}

#### Create a query with a derived attribute 'c'

{% highlight java %}
String[] properties = new String[] {"c=strConcat('hello ', a)"};
Query query = new Query(simpleFeatureTypeName, cqlFilter, properties);
{% endhighlight %}
