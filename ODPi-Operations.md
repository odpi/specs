# ODPi Application Installation and Management Specification
## Abstract
This specification outlines the requirements for ODPi-compliant applications to be installed, managed, and monitored by Ambari and the guarantees that consumers, ISVs, and service developers can count on to develop custom applications, etc.

## Objective
The Application Installation and Management specification covers requirements and guarantees for consumers, ISVs, and service developers while developing custom service specification and views. An application can be a custom service that can be managed by Ambari or a View that can be hosted by Ambari deployments conforming to this spec.

This specification covers the Ambari requirements/expectation for applications to be compatible with the [ODPi Runtime Spec](https://github.com/odpi/specs/blob/master/ODPi-Runtime.md).

## Ambari Version
The specification is based on Ambari 2.2.0.0.

## Ambari Runtime Environment Specification
Any Ambari user can assume the following for the runtime environment.

* JRE 7 or 8 is REQUIRED for Ambari
* Python 2.6 or 2.7 is REQUIRED for Ambari
* JAVA_HOME MUST be explicitly provided to Ambari and it MUST be the same on all hosts managed by Ambari

## Custom Service Specification
### Packaging Requirements
Ambari supports installation of services via rpm and debian packages only. It is expected that the packages are available at runtime via well-known repository solution such as YUM, Zypper, Apt.

* Custom service SHOULD be deployable on the following operating systems:  CentOS/Red Hat 6 and 7, Ubuntu 12 and 14, Debian 7, and SLES 11.  It SHOULD be made clear when not all of these operating systems are supported.

### Repository URL Specification
Ambari supports specification of the repository base URLs only at the stack level.

* Custom service MUST make the packages available via YUM, Zypper, or Apt repositories
* Custom service MUST make the repository URL available to Ambari in one of the following ways:
  * Prior to installation of the cluster, custom repositories URLs MAY be added to the list of repository URLs in the stack(s)'s `repoinfo.xml` 
  * Post installation of the cluster, Ambari REST APIs MAY be used to add additional repository URLs

### Service Definition Requirements
For Ambari to recognize a custom service definition it needs to be developed per Ambari 2.2.0 specification and installed to the appropriate stack definition directory under `/var/lib/ambari-server/resources/stacks/`*stack-name*`/`*stack-version*`/services`.  Once installed, Ambari will consume the service definition and present it as an option while deploying the cluster or adding a new service to the cluster.

* Service MUST include the `metainfo.xml` as required by Ambari
* Service MUST include the implementation of the following commands for each non-client component: install, configure, start, stop, status, and security_status
* Service MUST include the implementation of the following commands for each client component: install, configure
* Service MUST use Python based scripts to provide the implementation of its lifecycle commands
* The Python scripts MUST be compatible with both Python 2.6 and 2.7
* Service MUST specify <versionAdvertised>false</versionAdvertised> in `metainfo.xml`.  This ensures that adding Custom Service does not break Rolling Upgrade and Express Upgrade features.

### Role Command Order
Role command order allows Ambari to be aware of the start order of a custom service if the start of any of its component or service check needs other services to be started.

* Custom service MAY specify a role command order (via `role_command_order.json` file) in the service definition
* Custom service MUST not influence the relative command order between components that are not included in the service definition

### Kerberos
If an Application needs to communicate with other services or Applications that require Kerberos authentication, the Application SHOULD provide a "Kerberos descriptor" metadata file (`kerberos.json`) in its service definition.  This is optional, but if this is omitted, it puts the burden on the end user for performing Kerberos principal generation, Kerberos keytab generation and distribution, etc.

If an Application needs to communicate with other services or Applications that require Kerberos authentication, the Application MUST explicitly invoke Kerberos authentication calls (i.e., kinit invocations) in its Service Definition scripts.  

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
As of Ambari 2.2.0.0, an application-specific, custom dashboard cannot be created unless you modify ambari-web code, though some elements such as widgets are declarative and do not require code changes.
