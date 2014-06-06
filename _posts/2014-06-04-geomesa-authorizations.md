---
title: GeoMesa Authorizations
author: emilio
layout: tutorial
---

{% include tutorial-header.html %}

### Background

One of the most powerful features of Accumulo is the ability to apply cell-level ```visibilities``` to you data. 
A set of ```authorizations``` can be applied to your queries, which will limit the data returned based on visibility.

*Note: authorizations are distinct from table-level permissions, and operate at a much finer grain.*

Full details on authorizations can be seen in the [Accumulo](http://accumulo.apache.org/1.5/accumulo_user_manual.html#_security) documentation,
with more examples [here](http://accumulo.apache.org/1.5/examples/visibility.html).

### This tutorial will show you how to:

1.  Set visibilities on your data during ingestion through GeoMesa
2.  Apply authorizations to your queries through GeoMesa
3.  Implement user authorizations through the GeoMesa GeoServer plugin, using PKI certs to login to GeoServer and LDAP to store authorizations

<div class="callout callout-warning">
    <span class="glyphicon glyphicon-exclamation-sign"></span>
    You will need access to a Hadoop 2.2 installation as well as an Accumulo 1.5.x database.
</div>

#### Other prerequisites

Before you begin, you should also have these:

* a locally-built version of GeoMesa - see [GeoMesa Quickstart](/2014/05/28/geomesa-quickstart/) - (DOWNLOAD AND BUILD GEOMESA)
* an Accumulo user that has appropriate permissions to query your data
* a local copy of the Java Development Kit 1.7.x
* Apache Maven installed
* a GitHub client installed

### Ingest GDELT Data with Visibilities

If you have never ingested GDELT data, or you have previously ingested it **without** visibilities, you will need to ingest it again.
Follow the instructions [here](/2014/04/17/geomesa-gdelt-analysis/), with the following changes:

* ensure that you have the latest version of the GDELT tutorial code from github
* when executing the map/reduce job, include the following parameter:

{% highlight bash %}
   -visibility <visibility-label-string>
{% endhighlight %}

**Note: You will need to go through the entire GDELT tutorial, including the GeoServer setup, before attempting the GeoServer authorizations section of this tutorial.**

The entire command will be as follows:

{% highlight bash %}
hadoop jar /path/to/geomesa-gdelt-1.0-SNAPSHOT.jar \
   geomesa.gdelt.GDELTIngest                       \
   -instanceId <accumulo-instance-id>              \
   -zookeepers <zookeeper-hosts-string>            \
   -user <username> -password <password>           \
   -visibility <visibility-label-string>           \
   -tableName <table> -featureName <feature>       \
   -ingestFile hdfs:///gdelt/uncompressed/gdelt.tsv
{% endhighlight %}

The visibility string can be anything valid for your Accumulo instance. For the rest of this exercise,
we are going to assume the visibility string is 'user'. You can see the visibilities that are currently
enabled for your user through the accumulo shell:

{% highlight bash %}
$ accumulo shell -u myuser -p mypassword

Shell - Apache Accumulo Interactive Shell
- 
- version: 1.5.1-SNAPSHOT
- instance name: mycloud
- instance id: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
- 
- type 'help' for a list of available commands
- 
myuser@mycloud> getauths
user,admin
{% endhighlight %}

If your user does not already have authorizations, you can add them as follows:
*Note: A user cannot set authorizations unless the user has the System.ALTER_USER permission*

{% highlight bash %}
myuser@mycloud> getauths
user
myuser@mycloud> addauths -s admin -u myuser
myuser@mycloud> getauths
user,admin
{% endhighlight %}

Once the GDELT data is ingested, you should see a visibility label when you scan the table:

{% highlight bash %}
myuser@mycloud> table gdelt_auths
myuser@mycloud gdelt_auths> scan
00~gdelt~04e~20080125 169881494:SimpleFeatureAttribute [user]    \x02\x12169881494\x00\xAC\xBE\x81\xA2\x01\x00\x80\xB2\x88\xF5\xF5E\x00\xC2\xC1\x18\x00\xB0\x1F\x001\x02\xFBD\x00\x06JPN\x00\x0AJAPAN\x00\x06JPN\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x02\x00\x06090\x00\x06090\x00\x0409\x00\x04\x00\x00\x00\x00\xC0\x00\x06\x00\x02\x00\x06\x00\x00\x00\x00\x00\x00\x00\x00DRoss Sea, Oceans (general), Oceans\x00\x04OS\x00\x08OS00\x00\x00\x00\x96\xC2\x00\x00\x00/\xC3\x00\xE5\xF1\xB7\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x08\x00FDabei, Dubayy, United Arab Emirates\x00\x04AE\x00\x08AE03\x00\x81\x04\xCAA\x00\xB8\x1E]B\x00\xDD\xC7_\x00\xB6\xA6\x99\x13\x00 POINT (-175 -75)
{% endhighlight %}

### Download and Build the Tutorial Code

Pick a reasonable directory on your machine, and run:

{% highlight bash %}
git clone git@github.com:geomesa/geomesa-tutorial-authorizations.git
{% endhighlight %}

The ```pom.xml``` file contains an explicit list of dependent libraries that will be bundled together into the final tutorial. You should confirm that the versions of Accumulo and Hadoop match what you are running; if it does not match, change the value in the POM. (NB: The only reason these libraries are bundled into the final JAR is that this is easier for most people to do this than it is to set the classpath when running the tutorial. If you would rather not bundle these dependencies, mark them as provided in the POM, and update your classpath as appropriate.)

From within the root of the cloned tutorial, run:

{% highlight bash %}
mvn clean install
{% endhighlight %}

When this is complete, it will have built a JAR file that contains all of the code you need to run the tutorial.

### Run the Tutorial

On the command-line, run:

{% highlight bash %}
java -cp ./target/geomesa-tutorial-authorizations-1.0.jar \
   geomesa.tutorial.AuthorizationsTutorial \
   -instanceId <instance> \
   -zookeepers <zoos> \
   -user <user> \
   -password <pwd> \
   -tableName <table> \
   -featureName <feature>
{% endhighlight %}

where you provide the following arguments:

* ```<instance>``` - the name of your Accumulo instance
* ```<zoos>``` - comma-separated list of your Zookeeper nodes, e.g. zoo1:2181,zoo2:2181,zoo3:2181
* ```<user>``` - the name of an Accumulo user that will execute the scans, e.g. root
* ```<pwd>``` - the password for the previously-mentioned Accumulo user
* ```<table>``` - the name of the Accumulo table that has the GeoMesa GDELT dataset, e.g. gdelt
* ```<feature>``` - the feature name used to ingest the GeoMesa GDELT dataset, e.g. gdelt

You should see two queries run and the results printed out to your console. You should see output similar to the following:

{% highlight bash %}
Executing query with AUTHORIZED data store: auths are 'user,admin'
Results:
1|geom=POINT (33.9744 45.2908)

Executing query with UNAUTHORIZED data store: auths are ''
No results
{% endhighlight %}

The first query should return 1 or more results. The second query should return 0 results, since they are hidden by visibilities.

### AuthorizationsProvider

GeoMesa delegates the retrieval of authorizations to service providers that implement the following interface:

{% highlight java %}
geomesa.core.security.AuthorizationsProvider
{% endhighlight %}

When a GeoMesa DataStore is instantiated, it will scan the classpath for service providers. Third-party implementations
can be enabled by simply placing them in the classpath. See the Oracle [javadocs](http://docs.oracle.com/javase/7/docs/api/javax/imageio/spi/ServiceRegistry.html) for details on implementing a service provider.

To ensure that the correct AuthorizationsProvider is used, GeoMesa will throw an exception if multiple service providers are found
on the classpath. In this scenario, the particular service provider class to use can be specified by the following system property:

{% highlight java %}
geomesa.core.security.AuthorizationsProvider.AUTH_PROVIDER_SYS_PROPERTY = "geomesa.auth.provider.impl";
{% endhighlight %}

For simple scenarios, the set of authorizations to apply to all queries can be specified when creating
the GeoMesa DataStore by using the 'auths' configuration parameter. This will use the DefaultAuthorizationsProvider implementation provided by GeoMesa.

{% highlight java %}
// create a map containing initialization data for the GeoMesa data store
Map<String, String> configuration = ...
configuration.put("auths", "user,admin");
DataStore dataStore = DataStoreFinder.getDataStore(configuration);
{% endhighlight %}

If there are no AuthorizationsProviders found on the classpath, and the 'auths' parameter is not set, GeoMesa will default to
using the authorizations associated with the Accumulo connection (i.e. the 'user' configuration value).

```Note: this is not a recommended approach for a production system.```

In addition, please note that the authorizations used in any scenario cannot exceed the authorizations of the Accumulo connection.

### Insight Into How the Tutorial Works

The code for querying with authorizations is available in the following class:

* ```geomesa.tutorial.AuthorizationsTutorial```

The interesting code for this tutorial is contained in the 'main' method:

{% highlight java %}
// get an instance of the data store that uses the default authorizations provider, which will use whatever auths the connector has available
System.setProperty(AuthorizationsProvider.AUTH_PROVIDER_SYS_PROPERTY, DefaultAuthorizationsProvider.class.getName());
AccumuloDataStore authDataStore = (AccumuloDataStore) DataStoreFinder.getDataStore(dsConf);

// get another instance of the data store that uses our authorizations provider that always returns empty auths
System.setProperty(AuthorizationsProvider.AUTH_PROVIDER_SYS_PROPERTY, EmptyAuthorizationsProvider.class.getName());
AccumuloDataStore noAuthDataStore = (AccumuloDataStore) DataStoreFinder.getDataStore(dsConf);
{% endhighlight %}

This code snippet shows how you can specify the AuthorizationProvider to use with a system property.
The DefaultAuthorizationsProvider class is provided by GeoMesa, and used when no other implementations are found.
The EmptyAuthorizationsProvider class is included in the tutorial:

* ```geomesa.tutorial.EmptyAuthorizationsProvider```

The EmptyAuthorizationsProvider will always return an empty Authorizations object, which means that any data stored with visibilities will not be returned.

There is a more useful implementation of AuthorizationsProvider that will be explored in more detail in the next section:

* ```geomesa.tutorial.LdapAuthorizationsProvider```

Additionally, there are two helper classes included in the tutorial:

* ```geomesa.tutorial.GdeltFeature``` - Contains the attributes available in the GDELT data set.
* ```geomesa.tutorial.SetupUtil``` - Handles reading command-line arguments

### Applying Visibilities to GeoServer using PKIs and LDAP 

This section will show you how to configure GeoServer to authenticate users with PKIs, use LDAP to store visibilities, then apply visibilities on a per-user basis.

```Note: It is assumed for the rest of the tutorial that you have created the GeoServer data stores and layers outlined in the [GDELT tutorial](/2014/04/17/geomesa-gdelt-analysis/)```

#### Run GeoServer in Tomcat

**Note: If you are already running GeoServer in Tomcat, you can skip this step.**

GeoServer ships by default with an embedded Jetty servlet. In order to use PKI login, we need to install it in Tomcat instead.

1. Download and install [Tomcat 7](http://tomcat.apache.org/download-70.cgi)
2. Create an environment variable pointing to your tomcat installation:
(note: you may want to add this to your bash init scripts)

{% highlight bash %}
export CATALINA_HOME=/path/to/tomcat
{% endhighlight %}

3. IF you want to re-use your existing GeoServer configuration, create an environment variable pointing to your GeoServer data directory:
(note: you may want to add this to your bash init scripts)

{% highlight bash %}
export GEOSERVER_DATA_DIR=/path/to/geoserver/data_dir
{% endhighlight %}

4. Copy the GeoServer webapp from the GeoServer distribution into the tomcat servlet:

{% highlight bash %}
cp -r /path/to/geoserver/webapps/geoserver/ $CATALINA_HOME/webapps/
{% endhighlight %}

5. Increase the memory allocated to tomcat, which you will need for running complex queries in GeoServer:
(note: the values here may not be applicable for every installation)

{% highlight bash %}
cd $CATALINA_HOME/bin
echo 'CATALINA_OPTS="-Xmx2g -XX:MaxPermSize=128m"' >> setenv.sh
{% endhighlight %}

6. Start tomcat, either as a service or through the start scripts, and ensure that GeoServer is available at [http://localhost:8080/geoserver/web/]

#### Congfigure GeoServer for PKI Login

Follow the instructions located [here](http://docs.geoserver.org/stable/en/user/security/tutorials/cert/index.html) in order to enable PKI login to GeoServer.

In the step where you add the 'cert' filter to the 'Filter Chains', also add it to the 'rest', 'gwc' and 'default' chains (in addition to web).

For the rest of the tutorial, we will assume that the user is logged in as 'rod', from the sample PKIs.

**Note: There is a bug in the current version of GeoServer (2.5.1) that sometimes does not save authentication filters properly.**

If, after going through the above steps, you do not get logged in properly, do the following:

1. Shut down GeoServer
2. Navigate to the GeoServer data directory ($GEOSERVER_DATA_DIR or $GEOSERVER_HOME/data_dir)
3. Edit the file ./security/config.xml by adding 4 lines:

{% highlight xml %}
<filterChain>
  <filters name="web" class="org.geoserver.security.HtmlLoginFilterChain" interceptorName="interceptor" exceptionTranslationName="exception" path="/web/**,/gwc/rest/web/**,/" disabled="false" allowSessionCreation="true" ssl="false" matchHTTPMethod="false">
    <filter>rememberme</filter>
    <filter>cert</filter> <!--add this line -->
    <filter>form</filter>
    <filter>anonymous</filter>
  </filters>
  ...
  <filters name="rest" class="org.geoserver.security.ServiceLoginFilterChain" interceptorName="restInterceptor" exceptionTranslationName="exception" path="/rest/**" disabled="false" allowSessionCreation="false" ssl="false" matchHTTPMethod="false">
    <filter>cert</filter> <!--add this line -->
    <filter>basic</filter>
    <filter>anonymous</filter>
  </filters>
  <filters name="gwc" class="org.geoserver.security.ServiceLoginFilterChain" interceptorName="restInterceptor" exceptionTranslationName="exception" path="/gwc/rest/**" disabled="false" allowSessionCreation="false" ssl="false" matchHTTPMethod="false">
    <filter>cert</filter> <!--add this line -->
    <filter>basic</filter>
  </filters>
  <filters name="default" class="org.geoserver.security.ServiceLoginFilterChain" interceptorName="interceptor" exceptionTranslationName="exception" path="/**" disabled="false" allowSessionCreation="false" ssl="false" matchHTTPMethod="false">
    <filter>cert</filter> <!--add this line -->
    <filter>basic</filter>
    <filter>anonymous</filter>
  </filters>
</filterChain>
{% endhighlight %}

4. Re-start GeoServer
5. Verify that the 'web' filter chain has the 'cert' filter selected:

!["Web Filter Panel"](/img/tutorials/2014-06-04-geomesa-authorizations/cert9.jpg)

#### Install an LDAP Server for Storing Authorizations

**Note: If you are already have an LDAP server to use, you can skip this step.**

1. Download and install [ApacheDS](http://directory.apache.org/apacheds/downloads.html)
2. Either run as a service, or run through the start scripts:

{% highlight bash %}
cd apacheds-2.0.0-M16/bin
chmod 755 *.sh
./apacheds.sh 
{% endhighlight %}

#### Configure LDAP for Storing Authorizations

We want to configure LDAP with a user to match the Spring Security PKIs we are testing with. The end result we want is to create the following user:

```DN: cn=rod,ou=Spring Security,o=Spring Framework```

In order to do that, we will use Apache Directory Studio.

1. Download and run [Apache Directory Studio](http://directory.apache.org/studio/downloads.html)
2. Connect to the your LDAP instance (ApacheDS), using the instructions [here](http://directory.apache.org/apacheds/basic-ug/1.4.2-changing-admin-password.html)
(note: you do not need to change the password unless you want to)
3. Create a partition for our data:
    a. Right-click the 'ApacheDS (localhost)' entry under the 'Connection' tab and select 'Open Configuration'
    b. Click 'Advanced Partitions Configuration...'
    c. Click 'Add'
    d. Set the ID field to be 'Spring Framework'
    e. Set the Suffix field to be 'o=Spring Framework'
    f. Uncheck 'Auto-generate context entry from suffix DN'
    g. Set the following attributes in Context Entry:
        objectclass: extensibleObject
        objectclass: top
        objectclass: domain
        dc: Spring Framework2
        o: Spring Framework2
    h. Hit ctrl-S to save the partition
    !["ApacheDS Partition"](/img/tutorials/2014-06-04-geomesa-authorizations/apache-ds-partition.png)
4. **Re-start ApacheDS** - otherwise the partition will not be available and the LDIF import will fail.
5. Load the following LDIF file, which will create the Spring Security OU and the rod user:
    a. Right-click THe 'Root DSE' node in the LDAP browser, and select 'Import->LDIF import...'
    [Spring Security LDIF](/assets/tutorials/2014-06-04-geomesa-authorizations/spring-security-rod.ldif)

#### Test LDAP Connection Using Tutorial Code

The 
