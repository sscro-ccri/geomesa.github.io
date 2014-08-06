---
title: GeoMesa Command Line Tools
author: jake
layout: tutorial
---

{% include tutorial-header.html %}

<!-- add some style to fix the xml formatting color -->
<style>
code.xml { color:#93a1a1 }
</style>

## Introduction

Geomesa Tools is a set of command line tools to add feature management functions, query planning and explanation, 
ingest, and export abilities from 
the command line.  

## Configuration

To begin using the command line tools, first build the full Geomesa project with `mvn clean install`. This will build 
the project and geomesa-tools JAR file.  
 
Geomesa Tools relies on environment variables to connect to the Accumulo and Zookeeper instances. To set these, edit 
your `~/.bashrc` file and 
add the following lines at the end, adding the information specific to your instance:  

    export GEOMESA_USER=Your.user.here  
    export GEOMESA_PASSWORD=Your.password.here  
    export GEOMESA_INSTANCEID=Your.instanceId.here  
    export GEOMESA_ZOOKEEPERS=Your.zookeepers.string.here
   
Source your `~/.bashrc` file by running `source ~/.bashrc`. After this step, you will likely need to log out and log 
back in for your 
environment variables to be set correctly.  

Next, run the command line tools jar with the following command from your root Geomesa directory:  
{% highlight bash %}
java -jar geomesa-tools/target/geomesa-tools-accumulo1.5-1.0.0-SNAPSHOT-shaded.jar --help
{% endhighlight %}
This should print out the following usage text:

    GeoMesa Tools 1.0
    Usage: geomesa-tools [export|list|explain|delete|create|ingest] [options]
     --help
           show help command
    Command: export [options]
    Export all or a set of features in csv, geojson, gml, or shp format
     --catalog <value>
           the name of the Accumulo table to use -- or create, if it does not already exist -- to contain the new data
     --typeName <value>
           the name of the feature to export
     --format <value>
           the format to export to (e.g. csv, tsv)
     --attributes <value>
           attributes to return in the export
     --idAttribute <value>
           feature ID attribute to query on
     --latAttribute <value>
           latitude attribute to query on
     --lonAttribute <value>
           longitude attribute to query on
     --dateAttribute <value>
           date attribute to query on
     --maxFeatures <value>
           max number of features to return
     --query <value>
           ECQL query to run on the features
    Command: list [options]
    List the features in the specified Catalog Table
     --catalog <value>
           the name of the Accumulo table to use
    Command: explain [options]
    Explain and plan a query in Geomesa
     --catalog <value>
           the name of the Accumulo table to use
     --typeName <value>
           the name of the new feature to be create
     --filter <value>
           the filter string
    Command: delete [options]
    Delete a feature from the specified Catalog Table in Geomesa
     --catalog <value>
           the name of the Accumulo table to use
     --typeName <value>
           the name of the new feature to be create
    Command: create [options]
    Create a feature in Geomesa
      --catalog <value>
            the name of the Accumulo table to use -- or create, if it does not already exist -- to contain the new data
      --typeName <value>
            the name of the new feature to be create
      --sft <value>
            the string representation of the SimpleFeatureType
    Command: ingest [options]
    Ingest a feature into GeoMesa
      --file <value>
            the file you wish to ingest, e.g.: ~/capelookout.csv
      --format <value>
            the format of the file, it must be csv or tsv
      --table <value>
            the name of the Accumulo table to use -- or create, if it does not already exist -- to contain the new data
      --typeName <value>
            the name of the feature type to be ingested
      -s <value> | --spec <value>
            the sft specification for the file
      --datetime <value>
            the name of the datetime field in the sft
      --dtformat <value>
            the format of the datetime field
            
This usage text gives a brief overview of how to use each command. To learn more about each command the the flags 
associated with them,
browse to the 
[Github Tools README](https://github.com/locationtech/geomesa/tree/accumulo1.5.x/1.x/geomesa-tools#geomesa-tools).

## Feature Management

To begin, let's start by creating a new feature in Geomesa with the `create` command. The `create` command takes three
flags, `--catalog` for the name of the catalog table, `--typeName` for the name of the feature, and `--sft` for the 
SimpleFeatureType schema. From the root Geomesa directory, run the command:
{% highlight bash %}
java -jar geomesa-tools/target/geomesa-tools-accumulo1.5-1.0.0-SNAPSHOT-shaded.jar create --catalog cmd_tutorial 
--typeName feature --sft id:String:indexed=true,dtg:Date,geom:Point:srid=4326
{% endhighlight %}

This will create a new feature, "feature," on catalog table "cmd_tutorial." The catalog table stores metadata 
information about each feature,
and it will be used to prefix each table name in Accumulo. 

If the above command was successful, you should see output text similar to the following:
    
    2014-08-06 16:50:28,347 [main] INFO  geomesa.tools.Tools$ - Creating 'feature' with schema 'id:String:indexed=true,dtg:Date,geom:Point:srid=4326'. Just a few moments...
    2014-08-06 16:51:00,064 [main] WARN  geomesa.core.index.TemporalIndexCheck$ - 
    __________Possible problem detected in the SimpleFeatureType_____________
    SF_PROPERTY_START_TIME points to no existing SimpleFeature attribute, or isn't defined.
    However, the following attribute(s) could be used in GeoMesa's temporal index:
    
    dtg
    
    Please note that while queries on a temporal attribute will still work,
    queries will be faster if SF_PROPERTY_START_TIME, located in the SimpleFeatureType's UserData,
    points to the attribute's name
    
    GeoMesa will now point SF_PROPERTY_START_TIME to the first temporal attribute found:
    dtg
    so that the index will include a temporal component.
    
          
    2014-08-06 16:51:00,202 [main] INFO  geomesa.tools.Tools$ - Feature 'feature' with schema 'id:String:indexed=true,dtg:Date,geom:Point:srid=4326' successfully created.

Now that you've seen how to create features, create another feature on catalog table "cmd_tutorial" using your 
own first name for the `--typeName` and the above schema for the `--sft`.

You should have two features on catalog table "cmd_tutorial." To verify, we'll use the `list` command. The `list` 
command takes one flag, `--catalog` where you will specify the catalog table to be "cmd_tutorial." Run the 
following command:

{% highlight bash %}
java -jar geomesa-tools/target/geomesa-tools-accumulo1.5-1.0.0-SNAPSHOT-shaded.jar list --catalog cmd_tutorial 
{% endhighlight %}

The output text should be something like:

    2014-08-06 16:54:52,144 [main] INFO  geomesa.tools.Tools$ - Listing features on 'cmd_tutorial'. Just a few moments...
    2014-08-06 16:54:57,480 [main] INFO  geomesa.tools.FeaturesTool - feature: [AttributeDescriptorImpl id <id:String> nillable 0:1
    userData=(
            index ==> false), AttributeDescriptorImpl dtg <dtg:Date> nillable 0:1
    userData=(
            index ==> false), GeometryDescriptorImpl geom <geom:Point> nillable 0:1
    userData=(
            index ==> false)]
    2014-08-06 16:54:57,503 [main] INFO  geomesa.tools.FeaturesTool - {YourFirstNameHere}: [AttributeDescriptorImpl id <id:String> nillable 0:1
    userData=(
            index ==> false), AttributeDescriptorImpl dtg <dtg:Date> nillable 0:1
    userData=(
            index ==> false), GeometryDescriptorImpl geom <geom:Point> nillable 0:1
    userData=(
            index ==> false)]
            
Continuing on, let's delete the first feature we created with the `delete` command. The `delete` command takes two
flags, `--catalog` for the name of the catalog table and `--typeName` for the name of the feature to delete. Run the 
following command:

{% highlight bash %}
java -jar geomesa-tools/target/geomesa-tools-accumulo1.5-1.0.0-SNAPSHOT-shaded.jar delete --catalog cmd_tutorial --typeName feature
{% endhighlight %}

NOTE: Running this command will take a bit longer than the previous two, as it will delete three tables in Accumulo, as 
well as remove the metadata rows in the catalog table associated with the feature.

The output should resemble the following:

    2014-08-06 17:00:10,736 [main] INFO  geomesa.tools.Tools$ - Deleting 'feature.' Just a few moments...
    2014-08-06 17:01:06,147 [main] INFO  geomesa.tools.Tools$ - Feature 'feature' successfully deleted.

Great! You've learned about the feature management commands contained in the Geomesa Command Line Tools. Now, let's 
move onto ingesting features into Geomesa to use the `ingest`, `export`, and `explain` commands.

## Ingesting Features

## Exporting Features

Let's export your newly ingested features in a couple of file formats. Currently, the `export` command supports exports
to CSV, TSV, Shapefile, GEOJSON, and GML. We'll do one of each format in this next section.

The `export` command has 3 required flags, `--catalog` to specify the name of the catalog table, `--typeName` to give
the name of the feature to export, and `--format` to specify the output format. Additionally, you can specify more 
details about the kind of export you would like to perform with `export`'s optional flags. Those are:
 
`--attributes` - the attributes of the feature to return
`--maxFeatures` - the maximum number of features to return in an export
`--query` - a CQL query to perform on the features to return only subset of features matching the query

We'll use the `--maxFeatures` flag to ensure our dataset is small and quick to export. First, we'll export to CSV with 
the following command:

{% highlight bash %}
java -jar geomesa-tools/target/geomesa-tools-accumulo1.5-1.0.0-SNAPSHOT-shaded.jar export --catalog {TODO: name} --typeName {TODO: name} --format csv --maxFeatures 50
{% endhighlight %}

Now, run the above command four additional times, changing the `--format` flag to `tsv`, `shp`, `geojson`, and `gml`. 

The created files should now be under the "export" directory in your root Geomesa directory. Inspect these files to
ensure your data was properly exported (and if it wasn't, be sure to [submit a bug to our listserv](mailto:geomesa-users@locationtech.org)).

## Explaining Queries

One last command in the Geomesa Command Line Tools module is the `explain` command, which is covered in detail in the post 
[Understanding Queries by James Hughes](no-link-yet).

## Conclusion
We hope the release of the Command Line Tools module simplifies working with Geomesa and remove much of the boilerplate
from past versions. If you have ideas for additional functionality to include in the Command Line Tools module, please 
don't hesitate to [reach out on our listserv](mailto:geomesa-users@locationtech.org).