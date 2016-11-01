#ODPi Application Installation and Management Specification
##Abstract
This specification outlines the requirements for ODPi-Interoperable applications to be installed, managed, and monitored. It starts with giving a definition of common expectations that are applicable to all Hadoop Management tools and then breaks into subsections describing behaviour and expectations specific to particular Hadoop Management tools. Adherence to the common part of the spec provides guarantees that consumers, ISVs, and service developers can count on to develop custom applications, etc. Tool-specific parts of the spec are meant to be a guidance.

##Objective
The Application Installation and Management specification covers requirements and guarantees for consumers, ISVs, and service developers while packaging their Hadoop application that provide custom service specifications and views. An application can further rely on functionality provided by a particular implementation of a Hadoop Management tool. Currently this specification only covers those extensions provided by Apache Ambari but we expect to cover other Hadoop Management tools as the spec evolves. All of the tools are expected to provide common set of services that, at runtime, are guaranteed to be compliant with the [ODPi Runtime Specification](https://github.com/odpi/specs/blob/master/ODPi-Runtime.md). 

##Glossary
###Component

A component is a managed entity. It is installed, configured, upgraded and if it’s a daemon it can be started, stopped, etc. HDFS DataNode is a component.

###Custom Service

A custom service is a service that does not belong to the ODPi core and typically developed to be used against the ODPi runtime. In this spec, a custom service and application are used interchangeably.

###Cluster

A cluster is a collection of hosts and deployed services including ones that are part of ODPi run time or are custom services.

###Host

A host is a machine where components are physically deployed.

###Hadoop Management Tool

A tool that can manage services including the ODPi core services as well as custom services. 

###ODPi Core Services

The services which are included in the [ODPi Runtime Specification](https://github.com/odpi/specs/blob/master/ODPi-Runtime.md).  Currently this includes HDFS, MAPRED and YARN.

###Service

A service is a collection of components. Operations against the service translates to operations against individual components. HDFS is a service.

###Service Definition

A set of declarative files and scripts that encapsulate the management and monitoring requirements of a service.

###Stack

A stack is a collection of services for a specific version which includes a common release/delivery mechanism. 

###Stack Extension

A stack extension refers to a stack that is being added to another stack.

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

##Hadoop Management Tool Specifications
This section covers the set of requirements expected out of Hadoop Management tools managing services belonging to the ODPi runtime and custom services developed against the core runtime. Later the spec covers details specific to Apache Ambari that are applicable if custom services are being developed for Apache Ambari. 

###Packaging
A Hadoop Management tool MUST provide a packaging and package delivery mechanism for third-party developers to deliver, install the custom services. There are two parts to a custom service - service definition(s) and service package (files that are installed on disk). Usually, RPM and deb packaging is a good way to package files and can be used by standard installers available on linux platforms. It’s not atypical to see alternate packaging mechanism based on tarball, zip, etc. or even Docker images. A Hadoop Management tool MAY support well-known packaging solution such as RPM or deb packages. 

Service definition MAY be packaged and installed separately from the service package.

####Naming requirements:
* The naming requirements covers the name of the Services, Components, and Service Packages.
* A name MUST be valid for use as a file or directory name on the supported platforms
* A name MUST be unique across all services and components included in the cluster
* One component MUST be able to refer to another component or service using the unique name
* Hadoop Management tool MUST use the unique names when displaying or listing services and components.  A service or a component MAY specify a display name and a description

###Runtime Environment Specification
A Hadoop Management tool MUST specify the JRE versions it supports. It MUST support either JRE 7 or JRE 8 runtime environments (64-bit only).  It SHOULD support both JRE 7 and JRE 8.  Additionally, a Hadoop Management tool SHOULD provide a way for custom services to retrieve JAVA_HOME.

ODPi Applications MUST work in at least one of these runtime environments, and SHOULD be clear when they don’t support both.

###Management Operations
A Hadoop Management tool MUST have well-defined life-cycle support that the custom services can integrate with. Life-cycle operations are defined at the level of components. User should be able to install and configure any component. If the component is a daemon, it can be started and stopped. In this spec, the focus is to identify the basic set of operations required. 

A Hadoop Management tool 
* MUST support Install and Configure operations for all components

If the component is a daemon/process then a Hadoop Management tool 
* MUST support Start, Stop, and ability to query status (up or down). 
* MAY support Reconfigure, Graceful-Stop, Restart
* MAY support Upgrade and Downgrade
* MAY support customer commands

###Hadoop Management Tool Stack
The Hadoop Management tool MUST either ship a stack with the management tool or provide a mechanism for users to acquire a stack.   The stack MUST include all the services which are included in the ODPi runtime spec and MAY contain one or more custom services.

###Secured Deployment
A Hadoop Management tool MUST support deployment of secured services using Kerberos. The tool MAY support the deployment of the secured environment such as the availability of KDC server. Once the secured environment is made available to the management too it MUST support creation of keytabs, distribution or the keytabs, and the regeneration of the keytabs.

###Discoverability
It is critical for users to be able to discover what is deployed and what can be deployed.

The Hadoop Management tool  
* MUST allow querying of installed service and their components and on which host they are installed
* MUST allow querying of the latest desired configuration as well as the applied actual configurations
* MUST allow querying of the current status of all components 
* MUST allow listing of all hosts available on the cluster and the components that are deployed on the hosts
* MUST list all services and components that are available but not yet installed
* MAY list all stacks that are available and MUST list the stack that is installed
* MAY provide the version of all services that are installed

###Configuration Management
Configuration management is a critical feature in any management too. An ODPi Hadoop Management tool
* MUST manage configurations and push configurations to all hosts based on the components deployed on the host
* MUST support config versioning to allow comparison of config changes
* MUST report the version of the configuration that is applied to individual components
* MAY support reverting configurations to a previous version
* MAY support configuration recommendation and validation
* MAY support multiple repositories to include the various extra packages that may be required by the services

###Metrics
A Hadoop Management tool MUST support configurations of services to emit metrics. The emitted metrics can target any user preferred metrics store. Towards that end, a management tool MAY support storage and presentation of the metrics. It MAY also allow emission of metrics to an external store.

###Alerts
A Hadoop Management tool MUST support configuration of the services to generate alerts and MUST support mechanism to notify users when alerts are generated. It MAY support maintaining alert history for a reasonable time.

##Management Tool Specific

###Ambari

####Glossary

#####Ambari Stack

An Ambari stack is a stack as defined in the general section of the specification (a collection of services for a specific version which includes a common release/delivery mechanism).  Ambari only supports installing a single stack.

Any services which are not shipped in the stack SHOULD be installed as an extension.  In this first release of the operations specification, installing custom services by modifying the stack is allowed.

Generally the Platform Vendor ships a stack with the Ambari release including all of their services.

#####Extension

An extension is a collection of services for a specific version which includes a common release/delivery mechanism.  An extension can only be installed if a specific stack (or specific version of a stack) will be installed with the extension or has already been installed.

ISV custom services SHOULD be packaged as extensions instead of being added directly into a stack version.  In this first release of the operations specification, installing custom services by modifying the stack is allowed.

#####Management Pack

A management pack is a collection of services packaged as a tarball.  A management pack can be used to ship an Ambari stack, extensions or to add custom services to an Ambari stack.

####Runtime Environment Specification
In addition to the common runtime environment, any application can assume the following for the runtime environment on each node of the cluster:
* Python 2.6 or 2.7 is REQUIRED
* JAVA_HOME MUST be explicitly provided to Ambari (or Ambari can be used to setup Java)
* JAVA_HOME MUST be the same on all hosts

####Detailed Ambari Information

More details on stacks, extensions, services and management packs can be found in the [Apache Ambari Stack Definition PDF document](https://github.com/apache/ambari/raw/branch-2.4/contrib/docs/Apache_Ambari_Stack_Definition.pdf)

####Packaging and Repository URLs
Ambari supports installation of services via RPM and debian packages only. It is expected that the packages are available at runtime via a well-known repository solution such as YUM, Zypper or Apt (depending on the cluster’s operating system).

* Custom services MUST make the packages available via YUM, Zypper, or Apt repositories
* Custom services MUST make the repository URL available to Ambari

Repository URLs may be made available to Ambari in one of the following ways:
* Prior to installation of the cluster, custom repositories URLs MAY be added to the list of repository URLs in the ```stacks/<name>/<version>/repos/repoinfo.xml```.  Repository URLs would need to be added for each operating system required by the cluster.

```
<os family="redhat6">
  <!-- Leave all the existing repos -->
  <repo>
    <baseurl>http://<server>/path/to/repo</baseurl>
    <repoid>YOUR_REPO_ID</repoid>
    <reponame>YOUR_REPO_NAME</reponame>
  </repo>
</os>
```

* Post installation of the cluster, Ambari REST APIs MAY be used to add additional repository URLs

####Service Definition Requirements
For Ambari to recognize a custom service definition it needs to be developed per [Apache Ambari Stack Definition](https://github.com/apache/ambari/raw/branch-2.4/contrib/docs/Apache_Ambari_Stack_Definition.pdf). Once installed, Ambari will consume the service definition and present it as an option while deploying the cluster or adding a new service to the cluster.

In addition to the service definition requirements in the common section above:

* Custom Services MUST specify <versionAdvertised>false</versionAdvertised> in metainfo.xml. This ensures that adding Custom Service does not break the upgrade features.
* Although it is possible that multiple services can be defined in one metainfo.xml, Services other than YARN/MAPRED MUST be declared in their own metainfo.xml.
* The Python scripts MUST be compatible with both Python 2.6 and 2.7

####Inheritance
Inheritance in Ambari reduces duplication between different versions of Stacks, Extensions and Services.  Stacks, Extensions and Services all use the same mechanism to declare their inheritance.  In their corresponding metainfo.xml, a Stack or Extension may extend a previous version:

```
<metainfo>
    <extends>1.0</extends>
```

A service can inherit through the stack but may also inherit directly from common-services:

```
<metainfo>
  <schemaVersion>2.0</schemaVersion>
  <services>
    <service>
      <name>ZOOKEEPER</name>
      <version>3.4.5.2.0</version>
      <extends>common-services/ZOOKEEPER/3.4.5</extends>
```

When including the following services: HDFS, MAPRED, YARN, HIVE and ZOOKEEPER into a stack, they SHOULD be inherited from the common-services directory defined in Ambari.  In addition, any services, which have definitions in common-services, SHOULD inherit from their common-services definition.

For more information see the Service Inheritance or the Stack Inheritance sections of the [Apache Ambari Stack Definition](https://github.com/apache/ambari/raw/branch-2.4/contrib/docs/Apache_Ambari_Stack_Definition.pdf).

####Creating Stack Versions
A Hadoop Management Tool when packaging a stack for Ambari should follow the guidelines for defining stacks as specified in the [Apache Ambari Stack Definition](https://github.com/apache/ambari/raw/branch-2.4/contrib/docs/Apache_Ambari_Stack_Definition.pdf) and should make sure to include all mandatory requirements including the stack properties.

####Recommended Application Packaging
Each individual Application SHOULD be created as one or more extensions and SHOULD be packaged as a single management pack.  In this first release of the operations specification, installing custom services by modifying the stack is allowed.

For more information on packaging applications as extensions using management packs see the Packaging Custom Services section in the [Apache Ambari Stack Definition](https://github.com/apache/ambari/raw/branch-2.4/contrib/docs/Apache_Ambari_Stack_Definition.pdf).

####Defining Custom Services
Each custom service definition MUST be created following the guidelines for custom services in the [Apache Ambari Stack Definition](https://github.com/apache/ambari/raw/branch-2.4/contrib/docs/Apache_Ambari_Stack_Definition.pdf).

####Kerberos
If an Application needs to communicate with other services or Applications that require Kerberos authentication, the Application SHOULD provide a "Kerberos descriptor" metadata file (kerberos.json) in its service definition. This is optional, but if this is omitted, it puts the burden on the end user for performing Kerberos principal generation, Kerberos keytab generation and distribution, etc.

If an Application needs to communicate with other services or Applications that require Kerberos authentication, the Application MUST explicitly invoke Kerberos authentication calls (i.e., kinit invocations) in its Service Definition scripts.

For more information about Kerberos see the Kerberos section in the [Apache Ambari Stack Definition](https://github.com/apache/ambari/raw/branch-2.4/contrib/docs/Apache_Ambari_Stack_Definition.pdf).

####Metrics Monitoring
An Application MAY expose its application-level metrics via Ambari REST API by defining a metrics.json file in its Service Definition.

An Application MAY provide its own Hadoop Metrics2 Sink implementation or custom code to emit its metrics into Ambari Metrics Collector.

For more information about Metrics see the Metrics section in the [Apache Ambari Stack Definition](https://github.com/apache/ambari/raw/branch-2.4/contrib/docs/Apache_Ambari_Stack_Definition.pdf).

####Alerts
An Application MAY expose its application-level alerts via Ambari REST API and Ambari Web UI by defining an alerts.json file in its Service Definition.

In case an alert definition contains any script-based alerts, it MUST be written in Python and MUST be compatible with Python 2.6 and 2.7.

For more information about Alerts see the Alerts section in the [Apache Ambari Stack Definition](https://github.com/apache/ambari/raw/branch-2.4/contrib/docs/Apache_Ambari_Stack_Definition.pdf).

####Quicklinks
An Application MAY expose quick links to the Ambari UI by including a quicklinks.json file in its Service Definition.  For more information see the Quick Links section in the [Apache Ambari Stack Definition](https://github.com/apache/ambari/raw/branch-2.4/contrib/docs/Apache_Ambari_Stack_Definition.pdf).
