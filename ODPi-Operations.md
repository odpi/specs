# ODPi Application Installation and Management Specification
## Abstract
This specification outlines the requirements for ODPi-compliant applications to be installed, managed, and monitored by Ambari and the guarantees that consumers, ISVs, and service developers can count on to develop custom applications, etc.

## Objective
The Application Installation and Management specification covers requirements and guarantees for consumers, ISVs, and service developers while developing custom service specification and views. An application can be a custom service that can be managed by Ambari or a View that can be hosted by Ambari deployments conforming to this spec.

This specification covers the Ambari requirements/expectation for applications to be compatible with the [ODPi Runtime Spec](https://github.com/odpi/specs/blob/master/ODPi-Runtime.md).

## Ambari Version
The specification is based on Ambari 2.2.1.0.

## Ambari Runtime Environment Specification
Any Ambari user can assume the following for the runtime environment.

* JRE 7 or 8 is REQUIRED for Ambari
* Python 2.6 or 2.7 is REQUIRED for Ambari
* JAVA_HOME MUST be explicitly provided to Ambari and it MUST be the same on all hosts managed by Ambari

## Custom Service Specification
### Packaging Requirements
Ambari supports installation of services via rpm and debian packages only. It is expected that the packages are available at runtime via well-known repository solution such as YUM, Zypper, Apt.

* Custom service artifacts MUST be deployable via YUM, Zypper, or Apt

### Repository URL Specification
Ambari supports specification of the repository base URLs only at the stack level.

* Custom service MUST make the packages available via YUM, Zypper, or Apt repositories
* Custom service MUST make the repository URL available to Ambari in one of the following ways:
  * Prior to installation of the cluster, custom repositories URLs MAY be added to the list of repository URLs in the stack(s)'s `repoinfo.xml` (see [Apache Ambari 2.2.1 specification](https://cwiki.apache.org/confluence/display/AMBARI/Defining+a+Custom+Stack+and+Services))
  * Post installation of the cluster, Ambari REST APIs MAY be used to add additional repository URLs

###Deploying Custom Service
For Ambari to recognize a custom service definition it needs to be developed per [Ambari 2.2.1 specification](https://cwiki.apache.org/confluence/display/AMBARI/Defining+a+Custom+Stack+and+Services) and installed to the appropriate stack definition directory under `/var/lib/ambari-server/resources/stacks/<stack-name>/<stack-version>/services`. `<stack-name>` and `<stack-version>` uniquely identify the stack that is being customized. Ambari needs to be restarted post deployment to recognize the added custom stacks and/or services.

### Service Definition Requirements
For Ambari to recognize a custom service definition it needs to be developed per [Ambari 2.2.1 specification](https://cwiki.apache.org/confluence/display/AMBARI/Defining+a+Custom+Stack+and+Services). Once installed, Ambari will consume the service definition and present it as an option while deploying the cluster or adding a new service to the cluster. The Apache Ambari 2.2.1 specification provides additional details on how to write a custom service.

* A service MUST be named using an alphanumeric string
* Service name MUST be unique across all services included in the stack

A `metainfo.xml` file is an xml formatted declarative definition of a service. It provides the top level description of service definition. `metainfo.xml` contains one or more section for services with each service containing one or more components. See [Apache Ambari 2.2.1 specification](https://cwiki.apache.org/confluence/display/AMBARI/Writing+metainfo.xml) for details on how to author `metainfo.xml`.

```
<metainfo>
  <schemaVersion>2.0</schemaVersion>
  <services>
    <service>
      <name>MyService</name>
      ...
      <components>
        <component>
          <name>MyComponent</name>
          ...
```

* A component MUST be named using an alphanumeric string
* Component name MUST be unique across all components in the stack
* A component MUST be identified as `MASTER`, `SLAVE`, or `CLIENT`
* A component MAY include a cardinality specifying minimum and maximum number of instances for deployment
* Service MUST include the implementation of the following commands for each non-client component: `install`, `configure`, `start`, `stop`, `status`, and `security_status`
* Service MUST include the implementation of the following commands for each client component: `install`, `configure`
* Service MUST use Python based scripts to provide the implementation of its lifecycle commands
* The Python scripts MUST be compatible with both Python 2.6 and 2.7
* Service MUST specify `<versionAdvertised>false</versionAdvertised>` in `metainfo.xml`.  This ensures that adding Custom Service does not break Rolling Upgrade and Express Upgrade features.

```
<component>
  <name>MyComponent</name>
  <category>SLAVE</category>
  <cardinality>1+</cardinality>     
  <versionAdvertised>false</versionAdvertised>
    <commandScript>
    <script>scripts/hbase_regionserver.py</script>
    <scriptType>PYTHON</scriptType>
  </commandScript>
</component>
```

### Service Dependency Requirements
A service definition can describe various dependency requirements with other services, components, and/or configs. The details of how dependencies are specified can be found in [Apache Ambari 2.2.1 specification](https://cwiki.apache.org/confluence/display/AMBARI/Writing+metainfo.xml). This spec covers the naming requirement/expectations for the ODPi runtime components, HDFS, YARN, and ZOOKEEPER. The following table lists the names of such service and component as well as the type of the component.

|service|component|type|
|-------|---------|----|
|HDFS|DATANODE|SLAVE|
||NAMENODE|MASTER|
||SECONDARY_NAMENODE|MASTER|
||HDFS_CLIENT|CLIENT|
||JOURNALNODE|SLAVE|
||ZKFC|SLAVE|
|YARN|NODEMANAGER|SLAVE|
||RESOURCEMANAGER|MASTER|
||YARN_CLIENT|CLIENT|
||APP_TIMELINE_SERVER|MASTER|
|ZOOKEEPER|ZOOKEEPER_SERVER|MASTER|
||ZOOKEEPER_CLIENT|CLIENT|

* A custom application MAY not use any of the reserved service and component names above for its own service or components being defined
* A custom application MAY use any of the reserved service and component names above to specify dependencies

### Creating Custom Stack
A custom stack is a set of service definitions grouped for deployment/management. A custom stack may include services that are part of ODPi and zero or more custom service definitions.

When including ODPi services (`HDFS`, `YARN`, `ZOOKEEPER`) into a custom stack, they MUST be inherited from the `common-services` defined in the management infrastructure.

`common-services` is a folder that contains a set of service definitions that can be shared across different stacks. Including services that are derived from services within common-service folder ensures that runtime behavior and management operations are uniform across all stacks

### Role Command Order
Role command order allows Ambari to be aware of the start order of a custom service if the start of any of its component or service check needs other services to be started.

* Custom service MAY specify a role command order (via `role_command_order.json` file) in the service definition
* Custom service MUST not influence the relative command order between components that are not included in the service definition

### Kerberos
If an Application needs to communicate with other services or Applications that require Kerberos authentication, the Application SHOULD provide a "Kerberos descriptor" metadata file (`kerberos.json`) in its service definition.  This is optional, but if this is omitted, it puts the burden on the end user for performing Kerberos principal generation, Kerberos keytab generation and distribution, etc.

If an Application needs to communicate with other services or Applications that require Kerberos authentication, the Application MUST explicitly invoke Kerberos authentication calls (i.e., `kinit` invocations) in its Service Definition scripts.  

## Monitoring
### Metrics Monitoring
An Application MAY expose its application-level metrics via Ambari REST API by defining `metrics.json` file in its Service Definition.

An Application MAY provide its own Hadoop Metrics2 Sink implementation or custom code to emit its metrics into Ambari Metrics Collector.

### Alerts
An Application MAY expose its application-level alerts via Ambari REST API and Ambari Web UI by defining `alerts.json` file in its Service Definition.  

In case an alert definition contain any script-based alerts, it MUST be written in Python and MUST be compatible with Python 2.6 and 2.7.

## Views
An Application MAY provide one or more Views as custom UIs that can be deployed, configured, and accessed via Ambari Web UI.

View's UI code MAY be written in any standard web technologies including HTML, CSS, JavaScript, etc.  A View MAY use any JavaScript libraries as the View itself runs in its own sandbox inside an iframe.

A View SHOULD work against Internet Explorer 10+, Chrome 47+, Safari 9+, and Firefox 43+ and SHOULD make it clear when not all these browsers are supported.

A View SHOULD be compiled and packaged as a JAR file that can be run on both JRE 7 and JRE 8.  It SHOULD be made clear when either JRE 7 or JRE 8 is not supported.

A View MUST specify `view.xml` in the root of the JAR.

A View's name attribute in `view.xml` MUST not be the same as with other Views available.

## Glossary
<dl>
  <dt>Application</dt>
  <dd>A custom Service or a View written to work with Ambari.</dd>

  <dt>Component</dt>
  <dd>A component is a managed entity.  Typically, it is either a daemon that can be started and stopped, or a client that is invoked at runtime.  The component is installed, configured, upgraded, monitored, and if it is a daemon, it can also be started and stopped.</dd>

  <dt>Cluster</dt>
  <dd>A cluster is a collection of hosts and deployed services.</dd>

  <dt>Host</dt>
  <dd>A host is a machine where components are physically deployed.</dd>

  <dt>Host Component</dt>
  <dd>A host component is a deployed component instance on a specific host.</dd>

  <dt>Service</dt>
  <dd>A service is a collection of components. Operations against the service translates to operations against individual components.</dd>

  <dt>Service Definition</dt>
  <dd>A set of declarative files and scripts that encapsulate the management and monitoring requirements of a service.</dd>

  <dt>Stack</dt>
  <dd>A stack is a collection of services for a specific version.</dd>

  <dt>Stack Definition</dt>
  <dd>A set of service definitions that collectively define a stack.</dd>

  <dt>View</dt>
  <dd>A pluggable UI that can be installed, configured, and accessed via Ambari. </dd>

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

## Out of Scope
The following topics are out of scope for the initial draft specification.

### QuickLinks
QuickLinks are not customizable in Ambari 2.2.0.0 unless Ambari Web's core code is modified.  [AMBARI-11268](https://issues.apache.org/jira/browse/AMBARI-11268) added the customization capability.

### Upgrade
What a custom service can expect to do so that it can seamlessly plug-in into the Ambari Rolling Upgrade, Express Upgrade, Patch Upgrade, or some other upgrade functionality.

### Configuration Recommendation / Validation
This is done via stack advisor, but this exists at the stack level, so there’s no good story around exposing this to applications (though we have a similar situation with role command order).

### Dashboard Customization
As of Ambari 2.2.1.0, an application-specific, custom dashboard cannot be created unless you modify ambari-web code, though some elements such as widgets are declarative and do not require code changes.
