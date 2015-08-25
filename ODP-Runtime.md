### Status: Draft

Objective
=========

Objectives of the ODP TWG is to achieve the following:

1.  **For consumers:** ability to run any “ODP-compatible” software on any “ODP-compliant” platform and have it work.

2.  **For ISVs:** compatibility guidelines that allow them to “test once, run everywhere.”

3.  **For Hadoop platform providers:** compliance guidelines that enable ODP-compatible software to run successfully on their solutions. But the guidelines must allow providers to patch their customers in an expeditious manner, to deal with emergencies.

The goal of this document is to define the interface between ODP-compliant Apache Hadoop Runtime Services (such as HDFS) and ODP-compatible applications that achieves the above goal. This interface in turn can be used by ISVs to properly build their software, and will be used as the basis of a compliance test suite that can be used by ODP-compliant platform providers to test compliance.

Technical Context
=================

At this time, the ODP specification is a source-code specification: compliance is specified as shipping a platform built from a specific set of source artifacts. The exact source artifacts change with each ODP version, and thus are specified outside the scope of this document. That said, this document was written in the context of Apache Hadoop 2.7 with an eye towards future versions. It may and likely will have to evolve as Hadoop itself evolves.

While the ODP spec is source-based, the Hadoop implementation leaves many degrees of freedom in how Hadoop is deployed and configured--and also how it is used (e.g., nothing stops applications from calling private interfaces). These degrees of freedom interfere with the goal of “test once, run everywhere” (TONE). The goal of this spec is to close enough of those freedoms to achieve TONE.

Hadoop Build Specifications
===========================

To help achieve TONE, ODP-compliant Hadoop platforms MUST conform to the following build specifications.

Hadoop Version Specifications
-----------------------------

-   ODP Platforms MUST be a descendent of the Apache Hadoop 2.7 branch.

Hadoop Patch Specifications
---------------------------

While ODP can be more prescriptive when it comes to the source-code and release-timing of major and minor releases, platform providers need more flexibility in dealing with patch releases.  In particular, to deal with urgent security or availability problems for their customers, providers need to be able to do just about anything to triage an emergency situation.  Even after an emergency is dealt with, some customers and/or vendors are very conservative about change-management and providers need flexibility to work with such customers.

-   ODP platform providers have full flexibility to release fixes to customers who are facing urgent security or availability issues.  Once operations are restored to normal, however, these emergency fixes MUST eventually be replaced with more permanent patches.

-   ODP platform providers MAY release patched versions of ODP major or minor releases, but in doing so MUST follow strict guidelines:

    -   The source of all patches MUST be available publicly (see below).  The spirit of any patch MUST be to deal with major security, availability, compatibility, or correctness issues.  Patches MUST be 100% backward compatible (as defined by Hadoop’s compatibility guidelines) and MUST NOT be used to add features of any kind.
  
    -   ODP MUST itself issue official patch releases to the reference specification to deal with (very) major security, availability, or correctness issues.


Minimum Native build specifications
-----------------------------------

The native libraries of Hadoop have historically been a particular point of pain for ISVs. The specifications in this subsection should reduce that pain. These options guarantee a minimum set of basic functionalities that MUST be available for each of these components, including Apache Hadoop native operating system resources required for enabling Kerberos, many Java/OS performance and functionality enhancements, and the GZip and Snappy codec compression libraries. ODP Platforms MAY enable other features such as file system encryption, however they are considered optional and not part of the base specification.

### Common

-   hadoop-common-project MUST be built with:

    -   -Pnative = build libhadoop.so, which also enables ZLib/gzip compression codec

    -   -Drequire.snappy = enables Snappy compression

### HDFS

-   hadoop-hdfs-project MUST be built with:

    -   -Pnative = enable libhdfs.so

    -   -Drequire.libwebhdfs = builds the native WebHDFS shared library (libwebhdfs.so)

### YARN

-   hadoop-yarn-project MUST be built with:

    -   -Pnative = build and include container-executor

### MapReduce

-   hadoop-mapreduce-project MUST be built with:

    -   -Pnative = MapReduce client native task support

    -   -Drequire.snappy = enable Snappy support in the MapReduce native client

### Tools

-   hadoop-tools-poject MUST be built with:

    -   -Pnative = enable pipes support

Runtime Environment for Application Code
========================================


Minimum Versions
----------------

Applications on Unix platforms need to understand the base specification of some key components of which they write software. Two of those components are the Java runtime environment and the shell environment.

-   **Java:** ODP Platforms SHOULD support both JRE 7 and JRE 8 runtime environments (64-bit only). ODP Applications SHOULD work in at least one of these, and SHOULD be clear when they don’t support both.

-   **Shell scripts:** ODP Platforms and Applications SHOULD use either POSIX sh or GNU bash with the appropriate bang path configured for that operating system. GNU bash usage SHOULD NOT require any version of GNU bash later than 3.2.

Compliance
----------

In order to location common resources, some commonly set configuration details are necessary. 

-   All environment variables discussed in this section MUST be set on all nodes.
> Comment (SCG): We need to discuss the scope of these variables. Are they automatically available to every user that logs into the machine, or is there a standard location in which an "-env.sh" script can be found to set the environment variables?

-   An ODP Platform MUST set the JAVA\_HOME.
> Comment (SCG): We had discussed this being available via the "hadoop" script, is this requiring it to be set globally? 

-   ODP Platforms MUST have all of the base Apache Hadoop components installed.

-   ODP Platforms MUST pass the Big Top smoke tests.

-   ODP Platforms MUST NOT change public APIs, where an API is defined as either a Java API (aka "Apache Hadoop ABI") or a REST API. See the [Apache Hadoop Compatibility guidelines](http://hadoop.apache.org/docs/r2.7.1/hadoop-project-dist/hadoop-common/Compatibility.html#Java_Binary_compatibility_for_end-user_applications_i.e._Apache_Hadoop_ABI) for more information.

-   An ODP Platform MUST keep the same basic directory layout and content as the equivalent Apache component. Changes to that directory layout MUST be enabled by the component itself with the appropriate configurations for that layout configured.
> Comment (SCG): A clarifying example here would be useful

-   An ODP Platform MUST set the HADOOP\_COMMON\_HOME, HADOOP\_HDFS\_HOME, HADOOP\_MAPRED\_HOME, and HADOOP\_YARN\_HOME to an absolute directory of that component's location.  HADOOP\_COMMON\_LIB\_JARS\_DIR, HDFS\_LIB\_JARS\_DIR, MAPRED\_LIB\_JARS\_DIR, and YARN\_LIB\_JARS\_DIR MUST be set the relative directory to the jars of that associated component's \_HOME variable. (See [*this document*](https://github.com/apache/hadoop/blob/0bc15cb6e60dc60885234e01dec1c7cb4557a926/hadoop-common-project/hadoop-common/src/main/bin/hadoop-layout.sh.example) for related Apache Hadoop documentation.) For example:

```bash
HADOOP_COMMON_HOME="/usr/lib/hadoop-common"
HADOOP_COMMON_LIB_JARS_DIR="share/hadoop/common/lib"
```

This enables applications the capability to locate where the various Apache Hadoop components are located (user-level binaries and Java JAR files).  The content of these LIB\_JARS\_DIR directories MUST be the same as the ODP Reference Implementation and the Apache Hadoop distribution.

> Comment (SCG): Are there exceptions to be added here? I don't know enough about how extensions are plugged into hadoop, but would a vendor be allowed to have additional jars in some cases?  New compression algorithms? New scheduling algorithms? 

-   The location of the tools jar and other miscalleneous should be set to the HADOOP\_TOOLS\_PATH environment variable.  Unlike the other  LIB\_JARS\_DIR environment variables, this MUST be an absolute path and MAY contain additional content. The entire directory SHOULD NOT, by default, be included in the default hadoop class path.  Individual jars MAY be specified, however. [TODO: Update HADOOP-10787.]

-   HADOOP\_COMMON\_HOME/bin, HADOOP\_HDFS\_HOME/bin, HADOOP\_MAPRED\_HOME/bin, and HADOOP\_YARN\_HOME/bin MUST contain the same binaries and executables that they contain in the ODP Reference Implementation and the Apache Hadoop distribution. They MAY be modified to be either fix bugs or have enhanced features.  There MUST NOT be any additional content in order to avoid potential future conflicts.

-   HADOOP\_COMMON\_LIB\_JARS\_DIR, HDFS\_LIB\_JARS\_DIR, MAPRED\_LIB\_JARS\_DIR, and YARN\_LIB\_JARS\_DIR MUST contain the same binaries and executables that they contain in the ODP Reference Implementation and the Apache Hadoop distribution. They MAY be modified to be either fix bugs or have enhanced features.  There MUST NOT be any additional content in order to avoid potential future conflicts.

-   An ODP Platform MUST set the HADOOP\_CONF\_DIR environment variable to point to Apache Hadoop’s configuration directory if config files aren’t being stored in \*\_HOME/etc/hadoop.

-   It MUST be possible to determine key Hadoop configuration values by using “${HADOOP\_HDFS\_HOME}/bin/hdfs getconf” so that directly reading the XML via Hadoop’s Configuration object SHOULD NOT be required.

-   The native compression codecs for gzip and snappy MUST be available and enabled by default.

-   A common application-architecture is one where there’s a fair bit of stuff running on the “Client Host” -- a Web server, all kinds of app logic, maybe even a database. They interact with Hadoop using client-libraries and cluster-config files installed locally on the client host. These apps tend to have a lot of requirements in terms of the packages installed locally. A good ODP Platform implementation SHOULD NOT get in the way: at most, they SHOULD care about the version of Java and and Bash and nothing else.

-   ODP Platforms MUST define the APPS log4j appender to allow for ISV and user applications a common definition to log output. The actual definition, location of output, cycling requirements, etc of this appender is not defined by this specification and is ODP Platform or user- defined. [TODO: File a JIRA.]

Requirements we’d like to push upstream from a compatibility perspective:

-   Don’t assume GNU userland -- POSIX please -- to increase cross-platform compatibility.

Best practices for ODP Platforms:

-   ODP Platforms SHOULD avoid using randomized ports when possible. For example, the NodeManager RPC port SHOULD NOT use the default ‘0’ (or random) value. Using randomized ports may make firewall setup extremely difficult as well as makes some parts of Apache Hadoop function incorrectly.  Be aware that users MAY change these port numbers, including back to randomization.

-   For other components not covered by this specification, ODP Platforms SHOULD set the environment variable *component*_HOME to specify the location in which the component is installed and *component*_CONF_DIR to indicate the directory in which the component's configuration can be found, unless the configuration directory is located in *component*_HOME/conf.  

Compatibility
-------------

OPD Compatible Applications must follow these guidelines:

-   Applications that need a different version of Java other than the one pointed to by JAVA\_HOME, MUST NOT change the ODP Platform’s JAVA\_HOME setting. Instead, they SHOULD set it appropriately for their specific code as appropriate.

-   Applications SHOULD get the Java version via ${JAVA\_HOME}/bin/java -version or via Java system property detection.

-   Applications SHOULD use ${HADOOP\_CONF\_DIR} or ${\*\_HOME}/etc/hadoop as the location of the configuration directory.

-   Applications SHOULD use the REST interfaces in lieu of direct RPC calls. Applications MUST use the Hadoop client libraries or command-line tools to access the non-REST interfaces.  Applications SHOULD use stable interfaces when possible. Interfaces marked as evolving interfaces MAY be used however they are not preferred.

-   Applications SHOULD NOT use traditionally human consumable interfaces such as log file output or shell command output.

-   YARN applications SHOULD use the Web App proxy to surface their UIs to users, rather than asking users to connect directly to the Application Manager.

-   Applications SHOULD use the Java client libraries or “$HADOOP\_HDFS\_HOME/bin/hdfs getconf” to obtain configuration information, rather than reading config files directly. This includes getting the YARN Resource Manager address and port information.

-   Applications SHOULD only use the HADOOP\_CLASSPATH environment variable hook (2.x) or the shellprofile.d infrastructure (3.x) to manipulate the runtime content of the Java classpath. Applications SHOULD NOT inject themselves into the classpath other than manipulation of this environment variable.

-   Applications SHOULD obtain the Java classpath via the “${HADOOP\_COMMON\_HOME}/bin/hadoop classpath” command with the understanding that users and platforms may add or upgrade objects in that classpath.

-   Applications SHOULD obtain the version of a specific Apache Hadoop component via the appropriate “$\*\_HOME/bin/cmd version” command.

-   Applications SHOULD NOT assume that HDFS is the currently configured distributed file system. They SHOULD use “hadoop fs” commands instead of “hdfs dfs” commands. Future specifications MAY include the ability to use any file system that is compatible with the Hadoop Compatible File System (HCFS) specification.

-   Applications SHOULD either launch via the Apache Hadoop YARN ResourceManager REST API or via “${HADOOP\_YARN\_HOME}/bin/yarn jar”

Best practices for compatible apps to be portable and operator friendly:

-   Applications SHOULD NOT assume that an Intel processor is being used.

-   Applications SHOULD NOT assume that Linux is being used.

-   Applications SHOULD NOT assume that an Oracle JRE is being used.

-   Applications SHOULD NOT install their own dependent packages (e.g., Ruby, Python, Apache Web Server) unless absolutely necessary. They SHOULD list them as system requirements and let the operator install them.

-   Similarly, applications SHOULD NOT ship with “fat jars” that include Hadoop Java libraries. They SHOULD pick them up from their runtime environment.

-   In order to avoid conflicting with other services, applications SHOULD use distributed cache as much as possible to distribute execution objects to compute nodes. Pre-installation SHOULD be avoided as much as possible.

Glossary
========

-   **Service** - A *service* refers to a software package that is installable within the Hadoop stack. A service can be a be comprised of one or more *components* (such as a NameNode, DataNode, etc.) or may be as simple as a single library.

-   **Component** - A *service* is comprised of one or more components. For example, HDFS has three components: NameNode, Secondary NameNode, and DataNode. A single component may be installed across multiple nodes in the cluster, such as in the case of the HDFS data node.

-   **Distribution** -

-   **ISV vendor**

-   **ISV application**

-   **ODP Runtime**

-   **ODP Core**

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.
