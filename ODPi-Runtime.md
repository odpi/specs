ODPi Technical Working Group

ODPi Runtime Specification: 2.0

Date of Publication:

Status: Draft


---

Abstract
========

Specifications covering ODPi Platforms based upon Apache Hadoop and Apache Hive. Compatibility guidelines for applications running on ODPi Platforms.

Included Projects
=================
This specification covers:
* Apache Hadoop 2.7, including all maintenance releases.
* Apache Hadoop 2.7 compatible filesystems (HCFS).
* Apache Hive.

Maintenance releases indicate bug fix releases connected to the indicated minor release.  For example, at the time of this writing the Hadoop 2.7 line has two maintenance releases, 2.7.1 and 2.7.2.  Thus versions 2.7.0, 2.7.1, and 2.7.2 all satisfy this specification.

Objective
=========

Objectives of the ODPi TWG is to achieve the following:

1.  **For consumers:** ability to run any “ODPi-compatible” software on any “ODPi-compliant” platform and have it work.

2.  **For ISVs:** compatibility guidelines that allow them to “test once, run everywhere.”

3.  **For Hadoop platform providers:** compliance guidelines that enable ODPi-compatible software to run successfully on their solutions. But the guidelines must allow providers to patch their customers in an expeditious manner, to deal with emergencies.

The goal of this document is to define the interface between ODPi-compliant Apache Hadoop Runtime Services (such as HDFS) and ODPi-compatible applications that achieves the above goal. This interface in turn can be used by ISVs to properly build their software, and will be used as the basis of a compliance test suite that can be used by ODPi-compliant platform providers to test compliance.

Technical Context
=================

At this time the ODPi specification is strongly based on the source-code of the underlying Apache projects.  Part of compliance is specified as shipping a platform built from a specific line of Apache Hadoop, namely 2.7.  It is expected that the Apache Hadoop version the spec is based on will evolve as both Apache Hadoop and this specification evolve.

Even with a source code based specification, the Hadoop implementation leaves many degrees of freedom in how Hadoop is deployed and configured--and also how it is used (e.g., nothing stops applications from calling private interfaces). These degrees of freedom interfere with the goal of “test once, run everywhere” (TONE). The goal of this spec is to close enough of those freedoms to achieve TONE.

The source code approach is not followed in the same way for Apache Hive.  Instead a set of interfaces that are deemed to be important for applications and users are specified to be fully compatible with Apache Hive 1.2.


Specification Format
====================

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

Each entry will have a designation (free text in square brackets) in order to pinpoint which parts of the specification are in violation during certification.

The designation [TEST_ENVIRONMENT] is reserved for entries that are defining constraints on the environment in which ODPi-compliant Hadoop platforms are expected to run. The output of the reference implementation validation testsuite is expected to capture enough information to identify whether the test execution environment conforms to the specification.

Build Specifications
===========================

To help achieve TONE, ODPi-compliant platforms MUST conform to the following build specifications.

Version Specifications
-----------------------------

-   **[HADOOP_VERSION]** For this version of the specification, ODPi Platforms MUST include a descendent of the Apache Hadoop 2.7 branch.  Future versions MAY increase the base Apache Hadoop version.

-   The Apache components in an ODPi reference release MUST have their source be 100% committed to an Apache source tree.

Patch Specifications
---------------------------

While ODPi can be more prescriptive when it comes to the source-code and release-timing of major and minor releases of Apache components, platform providers need more flexibility in dealing with patch releases.  In particular, to deal with urgent security or availability problems for their customers, providers need to be able to do just about anything to triage an emergency situation.  Even after an emergency is dealt with, some customers and/or vendors are very conservative about change-management and providers need flexibility to work with such customers.

-   **[ODPi_PATCH1]** ODPi platform providers have full flexibility to release fixes to customers who are facing urgent security or availability issues.  Once operations are restored to normal, however, these emergency fixes MUST eventually be replaced with more permanent patches that comply with the specifications listed here.

-   **[ODPi_PATCH2]** Patches to Apache components MUST have the source available to the Apache community, posted via the project-specific bug-tracking system (like JIRA).  The vendor SHOULD make reasonable efforts to get the patch committed.

-   **[ODPi_PATCH3]** Patches to Apache components MUST be to deal with major security, availability, compatibility, or correctness issues.  Patches MUST be 100% backward compatible (as defined by Apache Hadoop's compatibility guidelines) and MUST NOT be used to add features of any kind.

- ODPi MUST itself issue official patch releases to the reference release to deal with very major security, availability, or correctness issues.


Minimum Native build specifications
-----------------------------------

The native libraries of Hadoop have historically been a particular point of pain for ISVs. The specifications in this subsection should reduce that pain. These options guarantee a minimum set of basic functionalities that MUST be available for each of these components, including Apache Hadoop native operating system resources required for enabling Kerberos, many Java/OS performance and functionality enhancements, and the GZip and Snappy codec compression libraries. ODPi Platforms MAY enable other features such as file system encryption, however they are considered optional and not part of the base specification.

### Common

-   hadoop-common-project MUST be built with:

    -   **[HADOOP_CNATIVE1]** `-Pnative` or `-Pnative-win`   (build `libhadoop.so`, which also enables ZLib/gzip compression codec)

    -   **[HADOOP_CNATIVE2]** `-Drequire.snappy`             (enable Snappy compression)

### HDFS

-   hadoop-hdfs-project MUST be built with:

    -   **[HADOOP_HNATIVE1]** `-Pnative` or `-Pnative-win`   (enable `libhdfs.so`)

### YARN

-   hadoop-yarn-project MUST be built with:

    -   **[HADOOP_YNATIVE1]** `-Pnative` or `-Pnative-win`   (build and include an appropriate container executor)

### MapReduce

-   hadoop-mapreduce-project MUST be built with:

    -   **[HADOOP_MNATIVE1]** `-Drequire.snappy`             (enable Snappy support in the MapReduce native client)

Runtime Environment for Application Code
========================================


Minimum Versions
----------------

Applications on Unix platforms need to understand the base specification of some key components of which they write software. Two of those components are the Java runtime environment and the shell environment.

-   **[TEST_ENVIRONMENT]** **Java:** ODPi Platforms SHOULD provide jars that are JDK 1.7 bytecode compatible, thus allowing them to run in both JRE 7 and JRE 8 runtime environments.  Only
support for 64-bit environments is required.  ODPi Applications SHOULD work in at least one of JRE 7 or JRE 8, and SHOULD be clear when they don’t support both.

-   **[TEST_ENVIRONMENT]**  **Shell scripts:** On Unix and Unix-like systems, ODPi Platforms and Applications SHOULD use either POSIX sh or GNU bash with the appropriate bang path configured for that operating system. GNU bash usage SHOULD NOT require any version of GNU bash later than 3.2.  On Windows, ODPi platforms and Applications SHOULD use Microsoft batch or PowerShell.


Environment Variables
---------------------

Apache Hadoop uses several critical environment variables to determine the Java class path and location of configuration information.  As a result, they become the glue that holds together not only Hadoop itself but also anything that connects to it. (See [*this document*](https://github.com/apache/hadoop/blob/0bc15cb6e60dc60885234e01dec1c7cb4557a926/hadoop-common-project/hadoop-common/src/main/bin/hadoop-layout.sh.example) for related Apache Hadoop documentation.)

In order to fulfill the goals of this specification, the discovery and content of several key environment variables are covered. This enables applications the capability to locate where the various Apache Hadoop components are located (user-level binaries and Java JAR files) in an ODPi Platform consistent way.

The following environment variables are noted by this spec:

| Test Designation | Environment Variable | Type | Description |
|:-----------------|:---------------------|:-----|:------------|
| **[HADOOP_EJH1]** | JAVA_HOME            | Absolute Dir  | Location of Java |
| **[HADOOP_EC1]** | HADOOP_TOOLS_PATH | Class Path | Supplemental Apache Hadoop jars for extra functionality |
| **[HADOOP_EC2]** | HADOOP_COMMON_HOME   | Absolute Dir  | Home directory of the Apache Hadoop 'common' component |
| **[HADOOP_EC3]** | HADOOP_COMMON_DIR    | Relative Dir  | Apache Hadoop common jars |
| **[HADOOP_EC4]** | HADOOP_COMMON_LIB_JARS_DIR | Relative Dir | Apache Hadoop common jar dependencies |
| **[HADOOP_EC5]** | HADOOP_CONF_DIR | Absolute Dir | Location of Apache Hadoop configuration files |
| **[HADOOP_EH1]** | HADOOP_HDFS_HOME   | Absolute Dir  | Home directory of the Apache Hadoop HDFS component |
| **[HADOOP_EH2]** | HDFS_DIR    | Relative Dir  | Apache Hadoop HDFS jars |
| **[HADOOP_EH3]** | HDFS_LIB_JARS_DIR | Relative Dir | Additional Apache Hadoop HDFS jar dependencies |
| **[HADOOP_EY1]** | HADOOP_YARN_HOME   | Absolute Dir  | Home directory of the Apache Hadoop YARN component |
| **[HADOOP_EY2]** | YARN_DIR    | Relative Dir  | Apache Hadoop YARN jars |
| **[HADOOP_EY3]** | YARN_LIB_JARS_DIR | Relative Dir | Additional Apache Hadoop YARN jar dependencies |
| **[HADOOP_EM1]** | HADOOP_MAPRED_HOME   | Absolute Dir  | Home directory of the Apache Hadoop MapReduce component |
| **[HADOOP_EM2]** | MAPRED_DIR    | Relative Dir  | Apache Hadoop MapReduce jars |
| **[HADOOP_EM3]** | MAPRED_LIB_JARS_DIR | Relative Dir | Additional Apache Hadoop MapReduce jar dependencies |


-   The content of the `*_DIR` directories SHOULD be the same as the ODPi Reference Implementation and the Apache Hadoop distribution of the appropriate platform.  In a future release, this will become a MUST.

-   **[HADOOP_ENVVAR]** All previously named environment variables mentioned in this section MUST be either explicitly set or readable via running the appropriate bin command with the `envvars` parameter.  In the situation where these variables are not explicitly set, the appropriate commands MUST be available on the path.    For example, `hadoop envvars` should provide output similar to the following:

```bash
$ hadoop envvars
HADOOP_COMMON_HOME='/opt/hadoop'
HADOOP_COMMON_DIR='share/hadoop/common'
HADOOP_COMMON_LIB_JARS_DIR='share/hadoop/common/lib'
HADOOP_CONF_DIR='/etc/hadoop'
JAVA_HOME='/usr/local/jdk1.8.0_45'
HADOOP_TOOLS_PATH='/opt/hadoop/share/hadoop/tools/lib'
```

-   **[HADOOP_EJH2]** An ODPi Platform MUST either explicitly set `JAVA_HOME` or configure it in `hadoop-env.sh` and `yarn-env.sh`. In a future specification, `yarn-env.sh` will be removed.

-   **[HADOOP_EJH2]** An ODPi Platform MUST set the `HADOOP_CONF_DIR` environment variable to point to Apache Hadoop’s configuration directory if config files aren’t being stored in `*_HOME/etc/hadoop`.

-   **[HADOOP_TOOLS]** The location of the tools jars and other miscellaneous jars SHOULD be set to the `HADOOP_TOOLS_PATH` environment variable.  This is used as input for setting Java class paths, therefore this MUST be an absolute path. It MAY contain additional content above and beyond what ships with Apache Hadoop and the reference implementation. The entire directory SHOULD NOT be included in the default hadoop class path.  Individual jars MAY be specified.

Compliance
----------

### Hadoop Compliance

-   **[HADOOP_SUBPROJS]** ODPi Platforms MUST have all of the base Apache Hadoop components installed.

-   **[HADOOP_BIGTOP]** ODPi Platforms MUST pass the Apache Big Top 1.0.0 Hadoop smoke tests.

-   **[TEST_ENVIRONMENT]** ODPi Platforms MUST NOT change public APIs, where an API is defined as either a Java API (aka "Apache Hadoop ABI") or a REST API. See the [Apache Hadoop Compatibility guidelines](http://hadoop.apache.org/docs/r2.7.1/hadoop-project-dist/hadoop-common/Compatibility.html#Java_Binary_compatibility_for_end-user_applications_i.e._Apache_Hadoop_ABI) for more information.

-   **[HADOOP_PLATVER]** ODPi Platforms MUST modify the version string output by Hadoop components, such as those displayed in log files, or returned via public API's such that they contain `-(vendor string)` or `_(vendor string)` where `(vendor string)` matches the regular expression [A-Za-z_0-9]+ and appropriately identifies the ODPi Platform vendor in the output.

-   An ODPi Platform MUST keep the same basic directory layout with regards to directory and filenames as the equivalent Apache component. Changes to that directory layout MUST be enabled by the component itself with the appropriate configurations for that layout configured.  For example, if Apache Hadoop YARN's package distribution contains a libexec directory with content, then that libexec directory with the equivalent content must be preset.  Additionally:

    -   **[HADOOP_DIRSTRUCT_COMMON]** Contents of HADOOP_COMMON_HOME should match the reference implementation. 
    -   **[HADOOP_DIRSTRUCT_HDFS]** Contents of HADOOP_HDFS_HOME should match the reference implementation.
    -   **[HADOOP_DIRSTRUCT_MAPREDUCE]** Contents of HADOOP_MAPRED_HOME should match the reference implementation.
    -   **[HADOOP_DIRSTRUCT_YARN]** Contents of HADOOP_YARN_HOME should match the reference implementation.

    -   **[HADOOP_BINCONTENT]**`HADOOP_COMMON_HOME/bin`, `HADOOP_HDFS_HOME/bin`, `HADOOP_MAPRED_HOME/bin`, and `HADOOP_YARN_HOME/bin` MUST contain the same binaries and executables that they contain in the ODPi Reference Implementation and the Apache Hadoop distribution of the appropriate platform, with exceptions granted for bug fixes.  Therefore, there SHOULD NOT be any additional content in order to avoid potential future conflicts.  In future versions of this spec this will become a MUST NOT.

    -   **[HADOOP_LIBJARSCONTENT]** `HADOOP_COMMON_LIB_JARS_DIR`, `HDFS_LIB_JARS_DIR`, `MAPRED_LIB_JARS_DIR`, and `YARN_LIB_JARS_DIR` MUST contain the same binaries and executables that they contain in the ODPi Reference Implementation and the Apache Hadoop distribution. They MAY be modified to be either fix bugs or have enhanced features.  There SHOULD NOT be any additional content in order to avoid potential future conflicts.  In future versions of this spec this will become a MUST NOT.

-   **[HADOOP_GETCONF]** It MUST be possible to determine key Hadoop configuration values by using `${HADOOP_HDFS_HOME}/bin/hdfs getconf` so that directly reading the XML via Hadoop’s Configuration object SHOULD NOT be required.

-   **[HADOOP_COMPRESSION]** The native compression codecs for gzip and snappy MUST be available.

-   A common application-architecture is one where there’s a fair bit of stuff running on the “Client Host” -- a Web server, all kinds of app logic, maybe even a database. They interact with Hadoop using client-libraries and cluster-config files installed locally on the client host. These apps tend to have a lot of requirements in terms of the packages installed locally. A good ODPi Platform implementation SHOULD NOT get in the way: at most, the implementation SHOULD only care about the version of Java and Bash, and nothing else.

-   ODPi Platforms SHOULD publish all modified (i.e., not-default) Apache Hadoop configuration entries, regardless of client, server, etc applicability to all nodes unless it is known to be node hardware specific, private to a service, security-sensitive, or otherwise problematic.  The list of variables that SHOULD NOT be shared are listed in Appendix A.

Requirements we’d like to push upstream from a compatibility perspective:

-   Don’t assume GNU userland -- POSIX please -- to increase cross-platform compatibility.

Best practices for ODPi Platforms:

-   ODPi Platforms SHOULD avoid using randomized ports when possible. For example, the NodeManager RPC port SHOULD NOT use the default ‘0’ (or random) value. Using randomized ports may make firewall setup extremely difficult as well as makes some parts of Apache Hadoop function incorrectly.  Be aware that users MAY change these port numbers, including back to randomization.

-   Future versions of this specification MAY require other components to set the environment variable *component*_HOME to the location in which the component is installed and *component*_CONF_DIR to the directory in which the component's configuration can be found, unless the configuration directory is located in *component*_HOME/conf.

### HCFS (Hadoop-compatible filesystem) Compliance

ODPi Platforms MAY include alternative filesystem implementations compatible with Apache Hadoop (HCFS). Even if they do, ODPi Platforms MUST still include Apache Hadoop HDFS as a viable choice. HCFS implementations MUST follow the guidelines laid out in the Hadoop FileSystem API Definition available as part of compliant Apache Hadoop release and also available [*online*](https://hadoop.apache.org/docs/r2.7.0/hadoop-project-dist/hadoop-common/filesystem/index.html). ODPi Platform providers MAY include results of compliance testing with HCFS implementations in the submission presented to ODPi. These results MUST at least include execution of tests created in accordance with guidelines provided by "Testing with the Filesystem specification" section of the Hadoop FileSystem API Definition. ISV vendors SHOULD NOT be expected to provide test results with HCFS implementations when certifying their ODPi interoperability. 

The following requirements provide guidance around best practices of integrating HCFS implementations with the rest of ODPi Platforms:

* **[HCFS_CLI]** Hadoop CLI tools MAY have different output when working with HCFS implementations 
* **[HCFS_INTEGRATION1]** First piece of Matt's doc goes here
* **[HCFS_INTEGRATION2]** Second piece of Matt's doc goes here

#### Requirements Relevant to HCFS Covered Elsewhere
It is important for HCFS implementations to be able to seamlessly integrate with tools covered by Operations Specification. Note that from a standpoint of an Operations Specification HCFS implementation is yet another instance of a generic service. All of the provisions laid out by Operations Specification will apply to an HCFS implementation when it comes to installation, configuration, monitoring, upgrades and decomissioning. 

### Hive Compliance

Apache Hive 1.2 is taken as the reference version for this version of ODPi specification.  This does not mean that ODPi Platforms must include Apache Hive 1.2.  Rather it means that Apache Hive 1.2 is the reference that all ODPi Platforms will be tested against.

* **[HIVE_SQL]** ODPi Platforms MUST provide SQL that is compatible with the reference version of Hive's SQL.  All valid statements in the reference version should produce the same results in the ODPi Platform.  The ODPi Platform MAY include additional SQL that is not supported in the reference version of Hive, subject to other requirements in this document.
* **[HIVE_JDBC]** ODPi Platforms MUST include access via JDBC and support all of the same JDBC functionality as the reference version of Hive.  All methods implemented in the reference version's JDBC client must be implemented and have the same signature in the ODPi Platform.  The ODPi Platform MAY implement additional JDBC methods that are not supported in the reference version of Hive, subject to other requirements of this document.
* **[HIVE_CLI]** ODPi Platforms MAY include the `bin/hive` command line interface.  If the platform includes the CLI it MUST accept all of the same arguments as the reference version.
* **[HIVE_BEELINE]** ODPi Platforms MUST include the `bin/beeline` command line script.  The platform MUST accept all of the same arguments as the reference version.
* **[HIVE_THRIFT]** ODPi Platforms MAY allow applications to connect to the Hive Metastore Thrift server via the thrift interface.  If the platform allows this it MUST accept all of the same thrift method calls as the reference version.
* **[HIVE_HCATALOG]** ODPi Platforms MAY support HCatalog.  If they do they MUST support the `HCatInputFormat`, `HCatOutputFormat`, `HCatReader`, and `HCatWriter` APIs and they MUST be binary compatible with the reference version.

#### Requirements Relevant to Hive Covered Elsewhere
It is important for applications to be able to find the Hive ODBC/JDBC endpoint on a cluster.  Discoverability of services in the ODPi Platform is covered in the Operations Specification, in the section titled Discoverability.


Compatibility
-------------

ODPi Compatible Applications must follow these guidelines:

-   Applications that need a different version of Java MUST NOT change the ODPi Platform’s `JAVA_HOME` setting. Instead, they SHOULD set it appropriately for their specific code in an appropriate way (either own startup scripts,
custom-to-the-application configuration file, etc) that does not impact the ODPi Platform.

-   Applications SHOULD get the Java version via `${JAVA_HOME}/bin/java` -version or via Java system property detection.

-   Applications SHOULD use `${HADOOP_CONF_DIR}` or `${*_HOME}/etc/hadoop` as the location of the configuration directory.

-   Applications SHOULD use the REST interfaces in lieu of direct RPC calls. Applications MUST use the Hadoop client libraries or command-line tools to access the non-REST interfaces.  Applications SHOULD use stable interfaces when possible. Interfaces marked as evolving interfaces MAY be used however they are not preferred.

-   Applications SHOULD NOT use traditionally human consumable interfaces such as log file output or shell command output.

-   YARN applications SHOULD use the Web App proxy to surface their UIs to users, rather than asking users to connect directly to the Application Manager.

-   Applications SHOULD use the Java client libraries or `${HADOOP_HDFS_HOME}/bin/hdfs getconf` to obtain configuration information, rather than reading config files directly. This includes getting the YARN Resource Manager address and port information.

-   Applications SHOULD NOT depend upon the configuration entries listed in Appendix A, as they are known to be node specific, private to a service, security-sensitive, or otherwise problematic.

-   Applications SHOULD only use the `HADOOP_CLASSPATH` environment variable hook (2.x) or the shellprofile.d infrastructure (3.x) to manipulate the runtime content of the Java classpath. Applications SHOULD NOT inject themselves into the classpath other than manipulation of this environment variable.

-   Applications SHOULD obtain the Java classpath via the `${HADOOP_COMMON_HOME}/bin/hadoop classpath` command with the understanding that users and platforms may add or upgrade objects in that classpath.

-   Applications SHOULD obtain the version of a specific Apache Hadoop component via the appropriate `$_HOME/bin/cmd version` command.

-   Applications SHOULD NOT assume that HDFS is the currently configured distributed file system. They SHOULD use `hadoop fs` commands instead of `hdfs dfs` commands. Future specifications MAY include the ability to use any file system that is compatible with the Hadoop Compatible File System (HCFS) specification.

-   Applications SHOULD either launch via the Apache Hadoop YARN ResourceManager REST API or via `${HADOOP_YARN_HOME}/bin/yarn jar`

-   Applications SHOULD use JDBC, ODBC, or SQL to determine the structure of Hive metadata rather than directly querying Hive's metastore thrift interface whenever possible.

Best practices for compatible apps to be portable and operator friendly:

-   Applications SHOULD NOT assume that an Intel processor is being used.

-   Applications SHOULD NOT assume that Linux is being used.

-   Applications SHOULD NOT assume that an Oracle JRE is being used.

-   Applciations SHOULD include both Microsoft batch or PowerShell as well as Unix-compatible shell scripts.

-   Applications SHOULD NOT install their own dependent packages (e.g., Ruby, Python, Apache Web Server) unless absolutely necessary. They SHOULD list them as system requirements and let the operator install them.

-   Similarly, applications SHOULD NOT ship with “fat jars” that include Hadoop Java libraries. They SHOULD pick them up from their runtime environment.

-   In order to avoid conflicting with other services, applications SHOULD use distributed cache as much as possible to distribute execution objects to compute nodes. Pre-installation SHOULD be avoided as much as possible.

-   Hive allows users to set some configuration values as part of their Hive session, via the *set* command.  Applications SHOULD only depend on setting those values that Hive by default allows users to set.  These values are listed in Appendix B.

Glossary
========

-   **Service** - A *service* refers to a software package that is installable within the Hadoop stack. A service can be comprised of one or more *components* (such as a NameNode, DataNode, etc.) or may be as simple as a single library.

-   **Component** - A *service* is comprised of one or more components. For example, HDFS has three components: NameNode, Secondary NameNode, and DataNode. A single component may be installed across multiple nodes in the cluster, such as in the case of the HDFS data node.

-   **Distribution** - A collection of components.

-   **ISV vendor** - Individual or company that created an ISV application.

-   **ISV application** - Non-ODPi application or process that runs on top of or beside an ODPi platform.

-   **ODPi Platform** - A distribution of software that includes all of the required components and is compliant with this specification.

-   **ODPi Runtime** - ODPi specification and platforms geared towards holistic management.

-   **ODPi Core** - ODPi specification and platforms geared towards components outside of any management requirements.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

Appendix A, Hadoop Configuration Values Not Shared
==================================================
The following Hadoop configuration values need not be shared by a compliant distribution and should not be depended on by an application:

| Configuration Value | Reason for Not Sharing |
|:--------------------|:-----------------------|
| dfs.block.access.key.update.interval | internal |
| dfs.block.invalidate.limit | internal |
| dfs.block.local-path-access.user | internal |
| dfs.block.misreplication.processing.limit | internal |
| dfs.block.replicator.classname | internal |
| dfs.block.scanner.volume.bytes.per.second | internal |
| dfs.blockreport.initialDelay | internal |
| dfs.blockreport.intervalMsec | internal |
| dfs.blockreport.split.threshold | internal |
| dfs.cachereport.intervalMsec | internal |
| dfs.cluster.administrators | internal |
| dfs.content-summary.limit | internal |
| dfs.content-summary.sleep-microsec | internal |
| dfs.corruptfilesreturned.max | internal |
| dfs.datanode.available-space-volume-choosing-policy.balanced-space-preference-fraction | internal |
| dfs.datanode.available-space-volume-choosing-policy.balanced-space-threshold | internal |
| dfs.datanode.balance.bandwidthPerSec | internal |
| dfs.datanode.balance.max.concurrent.moves | internal |
| dfs.datanode.block.id.layout.upgrade.threads | internal |
| dfs.datanode.cache.revocation.polling.ms | internal |
| dfs.datanode.cache.revocation.timeout.ms | internal |
| dfs.datanode.data.dir | internal |
| dfs.datanode.data.dir.perm | internal |
| dfs.datanode.directoryscan.interval | internal |
| dfs.datanode.directoryscan.threads | internal |
| dfs.datanode.dns.interface | internal |
| dfs.datanode.dns.nameserver | internal |
| dfs.datanode.drop.cache.behind.reads | internal |
| dfs.datanode.drop.cache.behind.writes | internal |
| dfs.datanode.du.reserved | internal |
| dfs.datanode.duplicate.replica.deletion | internal |
| dfs.datanode.failed.volumes.tolerated | internal |
| dfs.datanode.fsdataset.factory | internal |
| dfs.datanode.fsdataset.volume.choosing.policy | internal |
| dfs.datanode.fsdatasetcache.max.threads.per.volume | internal |
| dfs.datanode.handler.count | internal |
| dfs.datanode.hdfs-blocks-metadata.enabled | internal |
| dfs.datanode.keytab.file | security |
| dfs.datanode.lazywriter.interval.sec | internal |
| dfs.datanode.network.counts.cache.max.size | internal |
| dfs.datanode.oob.timeout-ms | internal |
| dfs.datanode.ram.disk.low.watermark.bytes | internal |
| dfs.datanode.ram.disk.low.watermark.percent | internal |
| dfs.datanode.ram.disk.replica.tracker | internal |
| dfs.datanode.readahead.bytes | internal |
| dfs.datanode.restart.replica.expiration | internal |
| dfs.datanode.scan.period.hours | internal |
| dfs.datanode.shared.file.descriptor.paths | internal |
| dfs.datanode.slow.io.warning.threshold.ms | internal |
| dfs.datanode.socket.reuse.keepalive | internal |
| dfs.datanode.socket.write.timeout | internal |
| dfs.datanode.startup | internal |
| dfs.datanode.sync.behind.writes | internal |
| dfs.datanode.sync.behind.writes.in.background | internal |
| dfs.datanode.synconclose | internal |
| dfs.datanode.transferTo.allowed | internal |
| dfs.datanode.use.datanode.hostname | internal |
| dfs.datanode.xceiver.stop.timeout.millis | internal |
| dfs.ha.fencing.ssh.private-key-files | security |
| dfs.ha.log-roll.period | internal |
| dfs.ha.log-roll.rpc.timeout | internal |
| dfs.ha.standby.checkpoints | internal |
| dfs.ha.tail-edits.period | internal |
| dfs.ha.zkfc.port | internal |
| dfs.heartbeat.interval | internal |
| dfs.https.server.keystore.resource | internal |
| dfs.image.compress | internal |
| dfs.image.compression.codec | internal |
| dfs.image.transfer.bandwidthPerSec | internal |
| dfs.image.transfer.chunksize | internal |
| dfs.image.transfer.timeout | internal |
| dfs.journalnode.edits.dir | internal |
| dfs.journalnode.http-address | internal |
| dfs.journalnode.https-address | internal |
| dfs.journalnode.keytab.file | security |
| dfs.metrics.percentiles.intervals | internal |
| dfs.metrics.session-id | internal |
| dfs.namenode.accesstime.precision | internal |
| dfs.namenode.audit.log.async | internal |
| dfs.namenode.audit.log.token.tracking.id | internal |
| dfs.namenode.audit.loggers | internal |
| dfs.namenode.avoid.read.stale.datanode | internal |
| dfs.namenode.avoid.write.stale.datanode | internal |
| dfs.namenode.blocks.per.postponedblocks.rescan | internal |
| dfs.namenode.checkpoint.check.period | internal |
| dfs.namenode.checkpoint.dir | internal |
| dfs.namenode.checkpoint.edits.dir | internal |
| dfs.namenode.checkpoint.max-retries | internal |
| dfs.namenode.checkpoint.period | internal |
| dfs.namenode.checkpoint.txns | internal |
| dfs.namenode.datanode.registration.ip-hostname-check | internal |
| dfs.namenode.decommission.blocks.per.interval | internal |
| dfs.namenode.decommission.interval | internal |
| dfs.namenode.decommission.max.concurrent.tracked.nodes | internal |
| dfs.namenode.delegation.key.update-interval | internal |
| dfs.namenode.delegation.token.max-lifetime | internal |
| dfs.namenode.delegation.token.renew-interval | internal |
| dfs.namenode.edit.log.autoroll.check.interval.ms | internal |
| dfs.namenode.edit.log.autoroll.multiplier.threshold | internal |
| dfs.namenode.edits.dir | internal |
| dfs.namenode.edits.dir.minimum | internal |
| dfs.namenode.edits.dir.required | internal |
| dfs.namenode.edits.journal-plugin | internal |
| dfs.namenode.edits.journal-plugin.qjournal | internal |
| dfs.namenode.edits.noeditlogchannelflush | internal |
| dfs.namenode.enable.retrycache | internal |
| dfs.namenode.handler.count | internal |
| dfs.namenode.heartbeat.recheck-interval | internal |
| dfs.namenode.inode.attributes.provider.class | internal |
| dfs.namenode.inotify.max.events.per.rpc | internal |
| dfs.namenode.invalidate.work.pct.per.iteration | internal |
| dfs.namenode.keytab.file | internal |
| dfs.namenode.lazypersist.file.scrub.interval.sec | internal |
| dfs.namenode.legacy-oiv-image.dir | internal |
| dfs.namenode.list.cache.directives.num.responses | internal |
| dfs.namenode.list.cache.pools.num.responses | internal |
| dfs.namenode.list.encryption.zones.num.responses | internal |
| dfs.namenode.max-num-blocks-to-log | internal |
| dfs.namenode.max.extra.edits.segments.retained | internal |
| dfs.namenode.max.objects | internal |
| dfs.namenode.name.cache.threshold | internal |
| dfs.namenode.name.dir | internal |
| dfs.namenode.name.dir.restore | internal |
| dfs.namenode.num.checkpoints.retained | internal |
| dfs.namenode.num.extra.edits.retained | internal |
| dfs.namenode.path.based.cache.block.map.allocation.percent | internal |
| dfs.namenode.path.based.cache.refresh.interval.ms | internal |
| dfs.namenode.path.based.cache.retry.interval.ms | internal |
| dfs.namenode.reject-unresolved-dn-topology-mapping | internal |
| dfs.namenode.replication.considerLoad | internal |
| dfs.namenode.replication.interval | internal |
| dfs.namenode.replication.max-streams | internal |
| dfs.namenode.replication.max-streams-hard-limit | internal |
| dfs.namenode.replication.min | internal |
| dfs.namenode.replication.pending.timeout-sec | internal |
| dfs.namenode.replication.work.multiplier.per.iteration | internal |
| dfs.namenode.replqueue.threshold-pct | internal |
| dfs.namenode.resource.check.interval | internal |
| dfs.namenode.resource.checked.volumes | internal |
| dfs.namenode.resource.checked.volumes.minimum | internal |
| dfs.namenode.resource.du.reserved | internal |
| dfs.namenode.retrycache.expirytime.millis | internal |
| dfs.namenode.retrycache.heap.percent | internal |
| dfs.namenode.rpc-bind-host | internal |
| dfs.namenode.safemode.extension | internal |
| dfs.namenode.service.handler.count | internal |
| dfs.namenode.servicerpc-bind-host | internal |
| dfs.namenode.shared.edits.dir | internal |
| dfs.namenode.stale.datanode.interval | internal |
| dfs.namenode.stale.datanode.minimum.interval | internal |
| dfs.namenode.startup | internal |
| dfs.namenode.startup.delay.block.deletion.sec | internal |
| dfs.namenode.support.allow.format | internal |
| dfs.namenode.tolerate.heartbeat.multiplier | internal |
| dfs.namenode.top.enabled | internal |
| dfs.namenode.top.num.users | internal |
| dfs.namenode.top.window.num.buckets | internal |
| dfs.namenode.top.windows.minutes | internal |
| dfs.namenode.write.stale.datanode.ratio | internal |
| dfs.namenode.xattrs.enabled | internal |
| dfs.permissions.superusergroup | internal |
| dfs.pipeline.ecn | internal |
| dfs.qjournal.accept-recovery.timeout.ms | internal |
| dfs.qjournal.finalize-segment.timeout.ms | internal |
| dfs.qjournal.get-journal-state.timeout.ms | internal |
| dfs.qjournal.new-epoch.timeout.ms | internal |
| dfs.qjournal.prepare-recovery.timeout.ms | internal |
| dfs.qjournal.queued-edits.limit.mb | internal |
| dfs.qjournal.select-input-streams.timeout.ms | internal |
| dfs.qjournal.start-segment.timeout.ms | internal |
| dfs.qjournal.write-txns.timeout.ms | internal |
| dfs.quota.by.storage.type.enabled | internal |
| dfs.secondary.namenode.keytab.file | security |
| dfs.web.authentication.kerberos.keytab | security |
| fs.df.interval | internal |
| fs.du.interval | internal |
| ha.failover-controller.active-standby-elector.zk.op.retries | internal |
| ha.health-monitor.check-interval.ms | internal |
| ha.health-monitor.connect-retry-interval.ms | internal |
| ha.health-monitor.rpc-timeout.ms | internal |
| ha.health-monitor.sleep-after-disconnect.ms | internal |
| ha.zookeeper.acl | security |
| ha.zookeeper.auth | security |
| ha.zookeeper.parent-znode | internal |
| ha.zookeeper.quorum | internal |
| ha.zookeeper.session-timeout.ms | internal |
| hadoop.htrace.spanreceiver.classes | internal |
| hadoop.http.authentication.cookie.domain | internal |
| hadoop.http.authentication.kerberos.keytab | security |
| hadoop.http.authentication.signature.secret.file | security |
| hadoop.http.authentication.token.validity | internal |
| hadoop.http.authentication.type | internal |
| hadoop.http.cross-origin.allowed-headers | internal |
| hadoop.http.cross-origin.allowed-methods | internal |
| hadoop.http.cross-origin.allowed-origins | internal |
| hadoop.http.filter.initializers | internal |
| hadoop.http.staticuser.user | internal |
| hadoop.jetty.logs.serve.aliases | internal |
| hadoop.security.group.mapping | internal |
| hadoop.security.group.mapping.ldap.base | security |
| hadoop.security.group.mapping.ldap.bind.password.file | security |
| hadoop.security.group.mapping.ldap.bind.user | security |
| hadoop.security.group.mapping.ldap.directory.search.timeout | security |
| hadoop.security.group.mapping.ldap.search.attr.group.name | security |
| hadoop.security.group.mapping.ldap.search.attr.member | security |
| hadoop.security.group.mapping.ldap.search.filter.group | security |
| hadoop.security.group.mapping.ldap.search.filter.user | security |
| hadoop.security.group.mapping.ldap.ssl | security |
| hadoop.security.group.mapping.ldap.ssl.keystore | security |
| hadoop.security.group.mapping.ldap.ssl.keystore.password.file | security |
| hadoop.security.group.mapping.ldap.url | security |
| hadoop.security.group.mapping.provider.* | security |
| hadoop.security.group.mapping.providers | security |
| hadoop.security.groups.cache.secs | security |
| hadoop.security.groups.cache.warn.after.ms | security |
| hadoop.security.groups.negative-cache.secs | security |
| hadoop.security.impersonation.provider.class | security |
| hadoop.security.instrumentation.requires.admin | security |
| ipc*.backoff.enable | internal |
| ipc.*.callqueue.impl | internal |
| ipc.*.identity-provider.impl | internal |
| ipc.maximum.data.length | internal |
| mapreduce.jobhistory.admin.acl | security |
| mapreduce.jobhistory.client.thread-count | internal |
| mapreduce.jobhistory.datestring.cache.size | internal |
| mapreduce.jobhistory.joblist.cache.size | internal |
| mapreduce.jobhistory.keytab | security |
| mapreduce.jobhistory.loadedjobs.cache.size | internal |
| mapreduce.jobhistory.move.interval-ms | internal |
| mapreduce.jobhistory.move.thread-count | internal |
| mapreduce.jobhistory.recovery.enable | internal |
| mapreduce.jobhistory.recovery.store.class | internal |
| mapreduce.jobhistory.recovery.store.leveldb.path | internal |
| mapreduce.jobhistory.store.class | internal |
| net.topology.configured.node.mapping | internal |
| net.topology.dependency.script.file.name | internal |
| net.topology.impl | internal |
| net.topology.node.switch.mapping.impl | internal |
| net.topology.script.file.name | internal |
| net.topology.script.number.args | internal |
| net.topology.table.file.name | internal |
| nfs.keytab.file | security |
| rpc.metrics.percentiles.intervals | internal |
| rpc.metrics.quantile.enable | internal |
| security.applicationhistory.protocol.acl | security |
| security.client.datanode.protocol.acl | security |
| security.client.protocol.acl | security |
| security.datanode.protocol.acl | security |
| security.get.user.mappings.protocol.acl | security |
| security.ha.service.protocol.acl | security |
| security.inter.datanode.protocol.acl | security |
| security.namenode.protocol.acl | security |
| security.qjournal.service.protocol.acl | security |
| security.refresh.callqueue.protocol.acl | security |
| security.refresh.generic.protocol.acl | security |
| security.refresh.policy.protocol.acl | security |
| security.refresh.user.mappings.protocol.acl | security |
| security.service.authorization.default.acl | security |
| security.service.authorization.default.acl.blocked | security |
| security.trace.protocol.acl | security |
| security.zkfc.protocol.acl | security |
| ssl.server.keystore.* | security |
| ssl.server.truststore.* | security |
| yarn.admin.acl | security |
| yarn.am.blacklisting.disable-failure-threshold | internal |
| yarn.am.blacklisting.enabled | internal |
| yarn.authorization-provider | internal |
| yarn.client.nodemanager-client-async.thread-pool-max-size | internal |
| yarn.nodemanager.admin-env | internal |
| yarn.nodemanager.amrmproxy.client.thread-count | internal |
| yarn.nodemanager.amrmproxy.interceptor-class.pipeline | internal |
| yarn.nodemanager.container-executor.class | internal |
| yarn.nodemanager.container-manager.thread-count | internal |
| yarn.nodemanager.container-monitor.process-tree.class | internal |
| yarn.nodemanager.container-monitor.procfs-tree.smaps-based-rss.enabled | internal |
| yarn.nodemanager.container-monitor.resource-calculator.class | internal |
| yarn.nodemanager.delete.thread-count | internal |
| yarn.nodemanager.health-checker.script.opts | internal |
| yarn.nodemanager.health-checker.script.path | internal |
| yarn.nodemanager.keytab | security |
| yarn.nodemanager.linux-container-executor.resources-handler.class | internal |
| yarn.nodemanager.localizer.cache.cleanup.interval-ms | internal |
| yarn.nodemanager.localizer.cache.target-size-mb | internal |
| yarn.nodemanager.localizer.client.thread-count | internal |
| yarn.nodemanager.localizer.fetch.thread-count | internal |
| yarn.nodemanager.log-aggregation.policy.class | internal |
| yarn.nodemanager.log.deletion-threads-count | internal |
| yarn.nodemanager.node-labels.provider.script.opts | internal |
| yarn.nodemanager.node-labels.provider.script.path | internal |
| yarn.nodemanager.recovery.dir | internal |
| yarn.nodemanager.resource-calculator.class | internal |
| yarn.nodemanager.runtime.linux.docker.privileged-containers.acl | security |
| yarn.nodemanager.webapp.spnego-keytab-file | security |
| yarn.resourcemanager.admin.client.thread-count | internal |
| yarn.resourcemanager.amlauncher.thread-count | internal |
| yarn.resourcemanager.client.thread-count | internal |
| yarn.resourcemanager.delegation-token-renewer.thread-count | internal |
| yarn.resourcemanager.fs.state-store.retry-policy-spec | internal |
| yarn.resourcemanager.fs.state-store.uri | internal |
| yarn.resourcemanager.history-writer.multi-threaded-dispatcher.pool-size | internal |
| yarn.resourcemanager.keytab | security |
| yarn.resourcemanager.leveldb-state-store.path | internal |
| yarn.resourcemanager.max-log-aggregation-diagnostics-in-memory | internal |
| yarn.resourcemanager.nodemanager-connect-retries | internal |
| yarn.resourcemanager.nodes.exclude-path | internal |
| yarn.resourcemanager.nodes.include-path | internal |
| yarn.resourcemanager.reservation-system.class | internal |
| yarn.resourcemanager.reservation-system.plan.follower | internal |
| yarn.resourcemanager.reservation-system.planfollower.time-step | internal |
| yarn.resourcemanager.resource-tracker.client.thread-count | internal |
| yarn.resourcemanager.rm.container-allocation.expiry-interval-ms | internal |
| yarn.resourcemanager.scheduler.class | internal |
| yarn.resourcemanager.scheduler.client.thread-count | internal |
| yarn.resourcemanager.scheduler.monitor.enable | internal |
| yarn.resourcemanager.store.class | internal |
| yarn.resourcemanager.system-metrics-publisher.dispatcher.pool-size | internal |
| yarn.resourcemanager.system-metrics-publisher.enabled | internal |
| yarn.resourcemanager.webapp.delegation-token-auth-filter.enabled | internal |
| yarn.resourcemanager.webapp.spnego-keytab-file | security |
| yarn.resourcemanager.zk-acl | security |
| yarn.resourcemanager.zk-state-store.root-node.acl | internal |
| yarn.scheduler.maximum-allocation-mb | internal |
| yarn.scheduler.maximum-allocation-vcores | internal |
| yarn.scheduler.minimum-allocation-mb | internal |
| yarn.scheduler.minimum-allocation-vcores | internal |
| yarn.sharedcache.admin.thread-count | internal |
| yarn.sharedcache.app-checker.class | internal |
| yarn.sharedcache.client-server.address | internal |
| yarn.sharedcache.enabled | internal |
| yarn.sharedcache.nested-level | internal |
| yarn.sharedcache.nm.uploader.replication.factor | internal |
| yarn.sharedcache.nm.uploader.thread-count | internal |
| yarn.sharedcache.root-dir | internal |
| yarn.sharedcache.store.class | internal |
| yarn.sharedcache.uploader.server.thread-count | internal |
| yarn.timeline-service.handler-thread-count | internal |
| yarn.timeline-service.keytab | security |
| yarn.timeline-service.leveldb-state-store.path | internal |
| yarn.timeline-service.leveldb-timeline-store.path | internal |
| yarn.timeline-service.leveldb-timeline-store.read-cache-size | internal |
| yarn.timeline-service.leveldb-timeline-store.start-time-read-cache-size | internal |
| yarn.timeline-service.leveldb-timeline-store.start-time-write-cache-size | internal |
| yarn.timeline-service.leveldb-timeline-store.ttl-interval-ms | internal |
| yarn.timeline-service.state-store-class | internal |
| yarn.timeline-service.store-class | internal |
| yarn.web-proxy.keytab | internal |

Appendix B, Hive Configuration Values to be White Listed
========================================================
The following values should be settable by Hive users in their Hive sessions.  Values ending in .* indicate all configuration values with the matching prefix.  This list is taken from the
default set of white listed values in Apache Hive 1.2.

| Configuration Value |
|:--------------------|
| hive.analyze.stmt.collect.partlevel.stats |
| hive.auto.* |
| hive.autogen.columnalias.prefix.includefuncname |
| hive.autogen.columnalias.prefix.label |
| hive.cache.expr.evaluation |
| hive.cbo.* |
| hive.client.stats.counters |
| hive.compat |
| hive.compute.query.using.stats |
| hive.convert.* |
| hive.counters.group.name |
| hive.default.fileformat.managed |
| hive.display.partition.cols.separately |
| hive.enforce.bucketing |
| hive.enforce.bucketmapjoin |
| hive.enforce.sorting |
| hive.enforce.sortmergebucketmapjoin |
| hive.error.on.empty.partition |
| hive.exec.*.dynamic.partitions.* |
| hive.exec.check.crossproducts |
| hive.exec.compress.* |
| hive.exec.concatenate.check.index |
| hive.exec.default.partition.name |
| hive.exec.drop.ignorenonexistent |
| hive.exec.dynamic.partition* |
| hive.exec.infer.* |
| hive.exec.job.debug.capture.stacktraces |
| hive.exec.job.debug.timeout |
| hive.exec.max.created.files |
| hive.exec.mode.local.* |
| hive.exec.orc.* |
| hive.exec.reducers.bytes.per.reducer |
| hive.exec.reducers.max |
| hive.exec.rowoffset |
| hive.exec.show.job.failure.debug.info |
| hive.exec.tasklog.debug.timeout |
| hive.execution.engine |
| hive.exim.uri.scheme.whitelist |
| hive.explain.* |
| hive.fetch.task.* |
| hive.file.max.footer |
| hive.groupby.skewindata |
| hive.hashtable.initialCapacity |
| hive.hashtable.loadfactor |
| hive.hbase.* |
| hive.ignore.mapjoin.hint |
| hive.index.* |
| hive.index.* |
| hive.insert.into.multilevel.dirs |
| hive.intermediate.* |
| hive.join.* |
| hive.limit.* |
| hive.limit.row.max.size |
| hive.localize.resource.num.wait.attempts |
| hive.log.* |
| hive.map.aggr |
| hive.mapjoin.* |
| hive.mapred.mode |
| hive.mapred.supports.subdirectories |
| hive.merge.* |
| hive.multi.insert.move.tasks.share.dependencies |
| hive.optimize.* |
| hive.orc.* |
| hive.outerjoin.* |
| hive.output.file.extension |
| hive.parquet.* |
| hive.ppd.* |
| hive.prewarm.* |
| hive.reorder.nway.joins |
| hive.resultset.use.unique.column.names |
| hive.server2.logging.operation.level |
| hive.server2.proxy.user |
| hive.skewjoin.* |
| hive.smbjoin.* |
| hive.stats.* |
| hive.support.quoted.identifiers |
| hive.support.sql11.reserved.keywords |
| hive.tez.* |
| hive.variable.substitute |
| hive.variable.substitute.depth |
| hive.vectorized.* |
| mapred.map.* |
| mapred.output.compression.codec |
| mapred.reduce.* |
| mapreduce.input.fileinputformat.split.minsize |
| mapreduce.job.queuename |
| mapreduce.job.reduce.slowstart.completedmaps |
| mapreduce.map.* |
| mapreduce.reduce.* |
| tez.am.* |
| tez.queue.name |
| tez.runtime.* |
| tez.task.* |

This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/legalcode)
