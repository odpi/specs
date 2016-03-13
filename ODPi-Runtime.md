ODPi Technical Working Group

ODPi Runtime Specification: 1.0

Date of Publication: 2016-01-22

Status: Draft


---

Abstract
========

Specifications covering ODPi Platforms based upon Apache Hadoop 2.7 and related branches. Compatibility guidelines for applications running on ODPi Platforms.

Objective
=========

Objectives of the ODPi TWG is to achieve the following:

1.  **For consumers:** ability to run any “ODPi-compatible” software on any “ODPi-compliant” platform and have it work.

2.  **For ISVs:** compatibility guidelines that allow them to “test once, run everywhere.”

3.  **For Hadoop platform providers:** compliance guidelines that enable ODPi-compatible software to run successfully on their solutions. But the guidelines must allow providers to patch their customers in an expeditious manner, to deal with emergencies.

The goal of this document is to define the interface between ODPi-compliant Apache Hadoop Runtime Services (such as HDFS) and ODPi-compatible applications that achieves the above goal. This interface in turn can be used by ISVs to properly build their software, and will be used as the basis of a compliance test suite that can be used by ODPi-compliant platform providers to test compliance.

Technical Context
=================

At this time, the ODPi specification is a source-code specification: compliance is specified as shipping a platform built from a specific set of source artifacts. The exact source artifacts change with each ODPi version, and thus are specified outside the scope of this document. That said, this document was written in the context of Apache Hadoop 2.7 with an eye towards future versions. It may and likely will have to evolve as Hadoop itself evolves.

While the ODPi spec is source-based, the Hadoop implementation leaves many degrees of freedom in how Hadoop is deployed and configured--and also how it is used (e.g., nothing stops applications from calling private interfaces). These degrees of freedom interfere with the goal of “test once, run everywhere” (TONE). The goal of this spec is to close enough of those freedoms to achieve TONE.

Specification Format
====================

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

Each entry will having a designation in order to pinpoint which parts of the specification are in violation during certification.

Hadoop Build Specifications
===========================

To help achieve TONE, ODPi-compliant Hadoop platforms MUST conform to the following build specifications.

Hadoop Version Specifications
-----------------------------

-   **[HADOOP_VERSION]** For this version of the specification, ODPi Platforms MUST be a descendent of the Apache Hadoop 2.7 branch.  Future versions MAY increase the base Apache Hadoop version.

-   The Apache components in an ODPi reference release MUST have their source be 100% committed to an Apache source tree.

Hadoop Patch Specifications
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

-   **[ODPi_JRE]** **Java:** ODPi Platforms SHOULD support both JRE 7 and JRE 8 runtime environments (64-bit only). ODPi Applications SHOULD work in at least one of these, and SHOULD be clear when they don’t support both.

-   **[ODPi_SCRIPT]**  **Shell scripts:** On Unix and Unix-like systems, ODPi Platforms and Applications SHOULD use either POSIX sh or GNU bash with the appropriate bang path configured for that operating system. GNU bash usage SHOULD NOT require any version of GNU bash later than 3.2.  On Windows, OPDi platforms and Applications SHOULD use Microsoft batch or PowerShell.


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

-   **[HADOOP_SUBPROJS]** ODPi Platforms MUST have all of the base Apache Hadoop components installed.

-   **[HADOOP_BIGTOP]** ODPi Platforms MUST pass the Apache Big Top 1.0.0 Hadoop smoke tests.

-   **[HADOOP_API]** ODPi Platforms MUST NOT change public APIs, where an API is defined as either a Java API (aka "Apache Hadoop ABI") or a REST API. See the [Apache Hadoop Compatibility guidelines](http://hadoop.apache.org/docs/r2.7.1/hadoop-project-dist/hadoop-common/Compatibility.html#Java_Binary_compatibility_for_end-user_applications_i.e._Apache_Hadoop_ABI) for more information.

-   **[HADOOP_PLATVER]** ODPi Platforms MUST modify the version string output by Hadoop components, such as those displayed in log files, or returned via public API's such that they contain `-(vendor string)` where `(vendor string)` matches the regular expression [A-Za-z_0-9]+ and appropriately identifies the ODPi Platform vendor in the output.

-   An ODPi Platform MUST keep the same basic directory layout with regards to directory and filenames as the equivalent Apache component. Changes to that directory layout MUST be enabled by the component itself with the appropriate configurations for that layout configured.  For example, if Apache Hadoop YARN's package distribution contains a libexec directory with content, then that libexec directory with the equivalent content must be preset.  Additionally:

    -   **[HADOOP_DIRSTRUCT_COMMON]** Contents of HADOOP_COMMON_HOME should match the reference implementation. 
    -   **[HADOOP_DIRSTRUCT_HDFS]** Contents of HADOOP_HDFS_HOME should match the reference implementation.
    -   **[HADOOP_DIRSTRUCT_MAPREDUCE]** Contents of HADOOP_MAPRED_HOME should match the reference implementation.
    -   **[HADOOP_DIRSTRUCT_YARN]** Contents of HADOOP_YARN_HOME should match the reference implementation.

    -   **[HADOOP_BINCONTENT]**`HADOOP_COMMON_HOME/bin`, `HADOOP_HDFS_HOME/bin`, `HADOOP_MAPRED_HOME/bin`, and `HADOOP_YARN_HOME/bin` SHOULD contain the same binaries and executables that they contain in the ODPi Reference Implementation and the Apache Hadoop distribution of the appropriate platform, with exceptions granted for bug fixes.  In future versions of this spec, this will become a MUST. Therefore, there MUST NOT be any additional content in order to avoid potential future conflicts.

    -   **[HADOOP_LIBJARSCONTENT]** `HADOOP_COMMON_LIB_JARS_DIR`, `HDFS_LIB_JARS_DIR`, `MAPRED_LIB_JARS_DIR`, and `YARN_LIB_JARS_DIR` MUST contain the same binaries and executables that they contain in the ODPi Reference Implementation and the Apache Hadoop distribution. They MAY be modified to be either fix bugs or have enhanced features.  There MUST NOT be any additional content in order to avoid potential future conflicts.

-   **[HADOOP_GETCONF]** It MUST be possible to determine key Hadoop configuration values by using `${HADOOP_HDFS_HOME}/bin/hdfs getconf` so that directly reading the XML via Hadoop’s Configuration object SHOULD NOT be required.

-   **[HADOOP_COMPRESSION]** The native compression codecs for gzip and snappy MUST be available.

-   A common application-architecture is one where there’s a fair bit of stuff running on the “Client Host” -- a Web server, all kinds of app logic, maybe even a database. They interact with Hadoop using client-libraries and cluster-config files installed locally on the client host. These apps tend to have a lot of requirements in terms of the packages installed locally. A good ODPi Platform implementation SHOULD NOT get in the way: at most, the implementation SHOULD only care about the version of Java and Bash, and nothing else.

-   **[HADOOP_DISTCONF]**  ODPi Platforms SHOULD publish all modified (i.e., not-default) Apache Hadoop configuration entries, regardless of client, server, etc applicability to all nodes unless it is known to be node hardware specific, private to a service, security-sensitive, or otherwise problematic.  The list of variables that SHOULD NOT be shared are defined as:

[**TODO: blacklist**]

Requirements we’d like to push upstream from a compatibility perspective:

-   Don’t assume GNU userland -- POSIX please -- to increase cross-platform compatibility.

Best practices for ODPi Platforms:

-   ODPi Platforms SHOULD avoid using randomized ports when possible. For example, the NodeManager RPC port SHOULD NOT use the default ‘0’ (or random) value. Using randomized ports may make firewall setup extremely difficult as well as makes some parts of Apache Hadoop function incorrectly.  Be aware that users MAY change these port numbers, including back to randomization.

-   Future versions of this specification MAY require other components to set the environment variable *component*_HOME to the location in which the component is installed and *component*_CONF_DIR to the directory in which the component's configuration can be found, unless the configuration directory is located in *component*_HOME/conf.

Compatibility
-------------

OPDi Compatible Applications must follow these guidelines:

-   Applications that need a different version of Java MUST NOT change the ODPi Platform’s `JAVA_HOME` setting. Instead, they SHOULD set it appropriately for their specific code in an appropriate way (either own startup scripts,
custom-to-the-application configuration file, etc) that does not impact the ODPi Platform.

-   Applications SHOULD get the Java version via `${JAVA_HOME}/bin/java` -version or via Java system property detection.

-   Applications SHOULD use `${HADOOP_CONF_DIR}` or `${*_HOME}/etc/hadoop` as the location of the configuration directory.

-   Applications SHOULD use the REST interfaces in lieu of direct RPC calls. Applications MUST use the Hadoop client libraries or command-line tools to access the non-REST interfaces.  Applications SHOULD use stable interfaces when possible. Interfaces marked as evolving interfaces MAY be used however they are not preferred.

-   Applications SHOULD NOT use traditionally human consumable interfaces such as log file output or shell command output.

-   YARN applications SHOULD use the Web App proxy to surface their UIs to users, rather than asking users to connect directly to the Application Manager.

-   Applications SHOULD use the Java client libraries or `${HADOOP_HDFS_HOME}/bin/hdfs getconf` to obtain configuration information, rather than reading config files directly. This includes getting the YARN Resource Manager address and port information.

-   Applications SHOULD NOT depend upon the following configuration entries, as they are known to be node specific, private to a service, security-sensitive, or otherwise problematic:

**TODO: blacklist**

-   Applications SHOULD only use the `HADOOP_CLASSPATH` environment variable hook (2.x) or the shellprofile.d infrastructure (3.x) to manipulate the runtime content of the Java classpath. Applications SHOULD NOT inject themselves into the classpath other than manipulation of this environment variable.

-   Applications SHOULD obtain the Java classpath via the `${HADOOP_COMMON_HOME}/bin/hadoop classpath` command with the understanding that users and platforms may add or upgrade objects in that classpath.

-   Applications SHOULD obtain the version of a specific Apache Hadoop component via the appropriate `$_HOME/bin/cmd version` command.

-   Applications SHOULD NOT assume that HDFS is the currently configured distributed file system. They SHOULD use `hadoop fs` commands instead of `hdfs dfs` commands. Future specifications MAY include the ability to use any file system that is compatible with the Hadoop Compatible File System (HCFS) specification.

-   Applications SHOULD either launch via the Apache Hadoop YARN ResourceManager REST API or via `${HADOOP_YARN_HOME}/bin/yarn jar`

Best practices for compatible apps to be portable and operator friendly:

-   Applications SHOULD NOT assume that an Intel processor is being used.

-   Applications SHOULD NOT assume that Linux is being used.

-   Applications SHOULD NOT assume that an Oracle JRE is being used.

-   Applciations SHOULD include both Microsoft batch or PowerShell as well as Unix-compatible shell scripts.

-   Applications SHOULD NOT install their own dependent packages (e.g., Ruby, Python, Apache Web Server) unless absolutely necessary. They SHOULD list them as system requirements and let the operator install them.

-   Similarly, applications SHOULD NOT ship with “fat jars” that include Hadoop Java libraries. They SHOULD pick them up from their runtime environment.

-   In order to avoid conflicting with other services, applications SHOULD use distributed cache as much as possible to distribute execution objects to compute nodes. Pre-installation SHOULD be avoided as much as possible.

Glossary
========

-   **Service** - A *service* refers to a software package that is installable within the Hadoop stack. A service can be comprised of one or more *components* (such as a NameNode, DataNode, etc.) or may be as simple as a single library.

-   **Component** - A *service* is comprised of one or more components. For example, HDFS has three components: NameNode, Secondary NameNode, and DataNode. A single component may be installed across multiple nodes in the cluster, such as in the case of the HDFS data node.

-   **Distribution** - A collection of components.

-   **ISV vendor** - Individual or company that created an ISV application.

-   **ISV application** - Non-ODPi application or process that runs on top of or beside an ODPi platform.

-   **ODPi Runtime** - ODPi specification and platforms geared towards holistic management.

-   **ODPi Core** - ODPi specification and platforms geared towards components outside of any management requirements.
