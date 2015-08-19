### Status: draft

Objective
=========

Objectives of the ODP TWG is to achieve the following:

1.  **For consumers:** ability to run any “ODP-compatible” software on any “ODP-compliant” platform and have it work.

2.  **For ISVs:** compatibility guidelines that allow them to “test once, run everywhere.”

3.  **For Hadoop platform providers:** compliance guidelines that enable ODP-compatible software to run successfully on their solutions. But the guidelines must allow providers to patch their customers in an expeditious manner, to deal with emergencies.

The goal of this document is to define the interface between ODP-compliant Apache Hadoop Runtime Services (such as HDFS) and ODP-compatible applications that achieves the above goal. This interface in turn can be used by ISVs to properly build their software, and will be used as the basis of a compliance test suite that can be used by ODP-compliant platform providers to test compliance.

Technical Context
=================

At this time, the ODP specification is a source-code specification: compliance is specified as shipping a platform built from a specific set of source artifacts. The exact source artifacts change with each ODP version, and thus are specified outside the scope of this document. That said, this document was written in the context of Apache Hadoop 2.x and may have to evolve as Hadoop itself evolves.

While the ODP spec is source-based, the Hadoop implementation leaves many degrees of freedom in how Hadoop is deployed and configured--and also how it is used (e.g., nothing stops applications from calling private interfaces). These degrees of freedom that interfere with the goal of “test once, run everywhere” (TONE) The goal of this spec is to close enough of those freedoms to achieve TONE.

The following component diagram describes the components of an Apache Hadoop cluster compliant to this specification:

The blue components in this diagram are the containers that host Application Code (the “Client Host” of a Hadoop cluster is sometimes called the “gateway” or “edge node”). The primary goal of this document is to specify the runtime environment of those containers, i.e., the interface between the platform and the software running on the platform.

The green boxes represent components of the Hadoop cluster that Application Code interacts with (directly). However, the term “interact” here needs to be defined very carefully:

> Application Code may directly access the REST interfaces of the green boxes, but they may not directly access the RPC or other non-REST interfaces. Rather, compatible Application Code must use the Hadoop client libraries or command-line tools to access the non-REST interfaces. Given this definition of “interact,” Application Code should not access the gray components, through RPC interfaces, REST interfaces, or even through client libraries or command-line tools. These components are internal to Hadoop.

Hadoop Build Specifications
===========================

As mentioned above, the ODP spec is a source-based specification. It was also mentioned above that degrees of freedom in the deployment and configuration of Hadoop interfere with the objective of TONE. It turns out that this troublesome configuration issues start all the way back to the build process. To help achieve TONE, ODP-compliant Hadoop platforms MUST conform to the following build specifications.

Hadoop Version Specifications
-----------------------------

For this version of the specification, an ODP Platform Apache Hadoop distribution MUST be a descendent of the Apache Hadoop 2.7 branch. Future versions of this specification MAY change the base version.

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

As mentioned above, the primary goal of this specification is to specify the runtime environments of the blue-boxes of Figure 1 -- the containers of Application Code.

Minimum Versions
----------------

Applications on Unix platforms need to understand the base specification of some key components of which they write software. Two of those components are the Java runtime environment and the shell environment.

-   **Java:** ODP Platforms SHOULD support both JRE 7 and JRE 8 runtime environments (64-bit only). ODP Applications SHOULD work in at least one of these, and SHOULD be clear when they don’t support both.

<!-- -->

-   **Shell scripts:** ODP Platforms and Applications SHOULD use either POSIX sh or GNU bash with the appropriate bang path configured for that operating system. GNU bash usage SHOULD NOT require any version of GNU bash later than 3.2.

Compliance
----------

In order to location common resources, some commonly set configuration details are necessary:

-   An ODP Platform MUST set the JAVA\_HOME.

-   ODP Platforms MUST have all of the base Apache Hadoop components installed.

-   An ODP Platform MUST keep the same basic directory layout and content as the equivalent Apache component. Changes to that directory layout MUST be enabled by the component itself with the appropriate configurations for that layout configured.

-   An ODP Platform MUST set the $\*\_HOME and $\*\_LIB\_JAR\_DIR environment variables for Apache Hadoop as defined in [*this document*](https://raw.githubusercontent.com/apache/hadoop/trunk/hadoop-common-project/hadoop-common/src/main/bin/hadoop-layout.sh.example). This enables applications the capability to locate where the various Apache Hadoop components are located (binaries and Java JAR files).

-   “$\*\_HOME/bin” MUST contain the same binaries and executables that they contain in the ODP Reference Implementation and the Apache Hadoop distribution. They MAY be modified to be either fix bugs or have enhanced features.

-   An ODP Platform MUST set the HADOOP\_CONF\_DIR environment variable to point to Apache Hadoop’s configuration directory if config files aren’t being stored in \*\_HOME/etc/hadoop.

-   It MUST be possible to determine key Hadoop configuration values by using “${HADOOP\_HDFS\_HOME}/bin/hdfs getconf” so that directly reading the XML via Hadoop’s Configuration object SHOULD NOT be required.

-   A common application-architecture is one where there’s a fair bit of stuff running on the “Client Host” -- a Web server, all kinds of app logic, maybe even a database. They interact with Hadoop using client-libraries and cluster-config files installed locally on the client host. These apps tend to have a lot of requirements in terms of the packages installed locally. A good ODP Platform implementation SHOULD NOT get in the way: at most, they SHOULD care about the version of Java and and Bash and nothing else.

-   TODO: say something about default configuration values: Nodemanager environment, YARN app class path

Requirements we’d like to push upstream from a compatibility perspective:

-   Don’t assume GNU userland -- POSIX please -- to increase cross-platform compatibility.

Best practices for ODP Platforms:

-   ODP Platforms SHOULD avoid using randomized ports when possible. For example, the nodemanager RPC port SHOULD NOT use the default ‘0’ (or random) value. Using randomized ports makes security extremely difficult as well as makes some parts of Apache Hadoop function incorrectly.

Compatibility
-------------

OPD Compatible Applications must follow these guidelines:

-   Applications needed a different version of Java other than the one pointed to by JAVA\_HOME, MUST NOT change the ODP Platform’s JAVA\_HOME setting. Instead, they SHOULD set it appropriately for their specific code as appropriate.

-   Applications SHOULD get the Java version via ${JAVA\_HOME}/bin/java -version or via Java system property detection.

-   Applications SHOULD use ${HADOOP\_CONF\_DIR} or ${\*\_HOME}/etc/hadoop as the location of the configuration directory.

-   Applications SHOULD use the REST interfaces in lieu of direct RPC calls. See note above about using REST and Client Libraries vs direct access to RPC.

-   YARN applications SHOULD use the Web App proxy to surface their UIs to users, rather than asking users to connect directly to the Application Manager.

-   Applications SHOULD use the Java client libraries or “$HADOOP\_HDFS\_HOME/bin/hdfs getconf” to obtain configuration information, rather than reading config files directly. This includes getting the YARN Resource Manager address and port information.

-   Applications SHOULD only use the HADOOP\_CLASSPATH environment variable hook (2.x) or the shellprofile.d infrastructure (3.x) to manipulate the content of the Java classpath. Applications SHOULD NOT inject themselves into the classpath other than manipulation of this environment variable.

-   Applications SHOULD obtain the Java classpath via the “${HADOOP\_COMMON\_HOME}/bin/hadoop classpath” command.

-   Applications SHOULD obtain the version of a specific Apache Hadoop component via the appropriate “$\*\_HOME/bin/cmd version” command.

-   Applications SHOULD NOT assume that HDFS is the currently configured distributed file system. They SHOULD use “hadoop fs” commands instead of “hdfs dfs” commands. Future specifications MAY include the ability to use any file system that is compatible with the Hadoop Compatible File System (HCFS) specification.

-   Applications SHOULD either launch via the Apache Hadoop YARN ResourceManager REST API or via “${HADOOP\_YARN\_HOME}/bin/yarn jar”

-   -   TODO: say something about where apps should put their own configuration settings (best practice?)

-   TODO: log4j

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
