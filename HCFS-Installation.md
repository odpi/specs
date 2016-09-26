# HCFS (Hadoop Compatible File System) Implementation and Installation in Apache Hadoop, and modifying Apache Ambari to install an HCFS
*Copyright © 2014-2016, Hortonworks, Inc.*

*Contributed to ODPi under ________ license*

*Draft v.0.5, 2016.09.21*

### Introduction
This documents pulls together information about HCFS (Hadoop Compatible File System) from multiple sources, regarding implementing, 
installing, and integrating with Ambari.  
This information may not be complete, but is believed currently correct.

*NOTE:  All non-HDFS file systems are implemented by their vendors, or in some cases third parties, and may or may not be placed 
in opensource.  The HCFS interface defined in Apache Hadoop allows for such third-party file systems to be integrated as independent 
object-code modules, a.k.a., “plug-ins”.*

### Terminology
* HDFS – Hadoop’s built-in distributed file system
* HCFS – a Hadoop Compatible File System, ie, a file system other than HDFS which can be “plugged in” to the Hadoop stack via the HCFS interface, then used for storage like any compatible file system
* HCFS implementation JAR – a compiled Java JAR that provides the interface, or “glue layer” between the HCFS and the Hadoop stack.  Placing this JAR in Hadoop’s classpath is how one “plugs in” the HCFS so Hadoop can use it.  There may be multiple HCFS installed, in which case each will have its own separate HCFS implementation JAR.
* Scheme name – A unique, one-word name for each file system must be specified, which is used in the URI “scheme:” prefix, and the related configuration parameter names.  Examples in use today are: `hdfs` (Hadoop built-in), `s3a` (Amazon), `wasb` (Windows Azure), `swift` (OpenStack), etc.

### Implementation
A good overview of the HCFS (Hadoop Compatible File System) plug-in interface is provided in the Apache wiki at this page: https://wiki.apache.org/hadoop/HCFS

The HCFS implementation JAR shall provide an implementation of these two abstract classes:
* `org.apache.hadoop.fs.FileSystem`
* `org.apache.hadoop.fs.AbstractFileSystem`

The FileSystem implementation class must be named:
* `org.apache.hadoop.fs.<hcfsname>.<HCFSFileSystemImplName>`

where `<hcfsname>` is substituted with the scheme name, and `<HCFSFileSystemImplName>` is substituted with an appropriate class name.

The JAR shall also declare the FileSystem implementation class as a service, by including a file 
* `META-INF/services/org.apache.hadoop.fs.FileSystem`

in which the HCFS FileSystem implementation class is named.  (For an example, see https://github.com/apache/hadoop/blob/trunk/hadoop-tools/hadoop-aws/src/main/resources/META-INF/services/org.apache.hadoop.fs.FileSystem )

There is no constraint on the naming of the AbstractFileSystem implementation class, but by convention it also should be in the 
package `org.apache.hadoop.fs.<hcfsname>`.  It should not be named in the `META-INF/services` file.

All behaviors of the HDFS implementation should be imitated insofar as the file system underlying the HCFS is compatible with 
such behavior.  In particular, if the HCFS is capable of providing locality information, it should support the API for 
telling applications such as Apache Tez and Apache Spark which TaskTracker is closest to the 
data (`FileSystem:getFileBlockLocations()`).  Where HDFS-native APIs cannot be implemented, then the HCFS should return values 
which are compatible with the rest of the Hadoop stack. As an example, while getBlockSize() is not a value which a blockless 
filesystem can actually return, a value suitable for block-partitioning algorithms should be returned, such as 256M, 512M or 
similar.

A detailed discussion of the API contract that must be supported by an HCFS is provided 
here: http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/filesystem/index.html

An important distinction is **whether or not a file system is capable of replacing HDFS in its role as primary storage and working 
space for MapReduce, Tez, and HBase.**
Almost any file system can be interfaced to the Hadoop stack via the HCFS interface, and can therefore be used for basic i/o and 
mass storage, alongside HDFS.  They are still HCFS implementations, but additional features (consistency, atomicity, and durability) 
must be provided in order for the file system to serve as primary storage replacing HDFS.  The difference is discussed in detail 
in the section titled *Object Stores vs Filesystems,* 
at: http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/filesystem/introduction.html#Object_Stores_vs._Filesystems 

As explained there, the key properties of consistency, atomicity, and durability, are missing from many filesystem-like storage 
platforms, including Amazon S3 and OpenStack Swift.  These are often referred to as Object Stores or Blob Stores, but be aware 
that this distinction by name is not necessarily correct.  In particular, some Blob Store-based file systems *do* support the 
necessary features and can be substituted for HDFS.  For example, on the Microsoft Azure HDInsights platform, the WASB file 
system (“Windows Azure Storage - Blob”) supports all features necessary to act as primary storage and working space for 
MapReduce, Tez, and HBase, and is successfully substituted for HDFS on the HDInsights platform.

The HCFS implementation interface also provides for Unit Testing.  Instructions for making an HCFS implementation JAR compatible 
with the built-in unit test capability are provided here: https://wiki.apache.org/hadoop/HCFS/Progress 

### Installation

Installing an HCFS in Hadoop consists of providing an HCFS implementation JAR, adding that JAR and any dependent JARs to the Hadoop classpath, and setting various configuration parameters.  Of course the underlying file system itself must also be available.  Some underlying file systems require installation on the cluster.  Others are off-cluster services, perhaps Cloud-based, and available without an installation process.  If the file system underlying the HCFS requires installation, we assume it is installed prior to and independent of installing the HCFS into Hadoop.  

The HCFS implementation JAR must be added to the class path defined by the HADOOP_CLASSPATH environment variable.  (This 
environment variable is a required part of any Hadoop installation.)  This may be done by:
* placing the JAR in the default location of `HADOOP_CLASSPATH`, at `$HADOOP_HOME/lib/`
* or by extending the `HADOOP_CLASSPATH` environment variable value to include the path to some other location where the HCFS 
implementation JAR has been deployed.  This may be defined in the `hadoop-env.sh` file, or by other usual means for managing 
environment variables.

To enable the HCFS, the following two configuration parameters must be set in core-site.xml:
```
<property>
  <name>fs.hcfsname.impl</name>
  <value>fully qualified class name of FileSystem concrete 
         implementation class</value>
  <description>optional description</description>
</property>
<property>
  <name>fs.AbstractFileSystem.hcfsname.impl</name>
  <value>fully qualified class name of AbstractFileSystem 
         concrete implementation class</value>
  <description>optional description</description>
</property>
```
where `hcfsname` is replaced with the HCFS scheme name, and the actual fully qualified class name is provided for the 
values.  [Note: the `fs.hcfsname.impl` parameter is not strictly required since `META-INF/services` declarations started 
being used in Hadoop-2.0.  However, it is still documented and therefore may be expected by some applications, so we 
recommend defining it.  The `fs.AbstractFileSystem.hcfsname.impl` parameter, on the other hand, is required.]

Example (using opensource specifications from AWS “S3” file system, which uses `s3a:` as its HCFS scheme name): 
```
<property>
  <name>fs.s3a.impl</name>
  <value>org.apache.hadoop.fs.s3a.S3AFileSystem</value>
</property>
<property>
  <name>fs.AbstractFileSystem.s3a.impl</name>
  <value>org.apache.hadoop.fs.s3a.S3A</value>
</property>
```

If the HCFS is intended to replace HDFS as primary storage and workspace for MapReduce, Tez, and HBase, the following additional 
config parameter must also be set in `core-site.xml`:
```
<property>
  <name>fs.defaultFS</name>
  <value>hcfsname://localservice</value>
</property>
```
where again `hcfsname` is replaced with the HCFS scheme name, and `localservice` is replaced with the URI authority element 
appropriate to the HCFS, at the server level. The value of `localservice` may be just `localhost:port#` for many HCFS 
implementations.  [Note: `fs.defaultFS` used to be `fs.default.name`, but the older parameter name is now deprecated.]

Since the Hadoop configuration parameter set is extensible, arbitrary additional parameters, specific to the HCFS, may be defined.  
These can then be accessed from the HCFS implementation JAR using the general Hadoop configuration APIs.  In this manner, the HCFS 
can be configured via parameters like any other Hadoop facility.  By convention, any such HCFS-specific configuration parameters 
should include the HCFS scheme name as an element of the parameter name.

### Integration with Ambari
Apache Ambari is a highly extensible management utility for Hadoop deployments.  A developer can customize Ambari to provide both 
installation and monitoring/management of an HCFS.

The canonical way to add an HCFS installation experience in Ambari is to use the Stacks feature of Ambari to create a child stack 
inheriting from the underlying Stack version (without the HCFS), and make modifications to:
* add the repo containing the HCFS implementation JAR, 
* adjust the HADOOP_CLASSPATH definition, 
* and change or add configuration parameters as needed for the particular HCFS.  

How to add a new Ambari Stack is documented here: https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=38571133
In the current Apache Ambari source code tree, you can see examples of HCFS integrations under
https://github.com/apache/ambari/tree/trunk/ambari-server/src/main/resources/stacks
For example, under the HDP product in the above directory, you can see examples for both GlusterFS and ECS third-party file 
system HCFS integrations.  

Comparing and contrasting the HCFS integrated Stacks with the vanilla Stack of the same version number, may assist in providing 
concrete understanding of the documentation about Ambari Stacks.

Once a stack extension is defined and the various files are added to the “stacks” path in Ambari, the “active” flag in the 
`metainfo.xml` file must be set to true to enable the extension. The extension developer may also change the build scripts to 
produce an Ambari build with the new stack extension built in and enabled by default.

### Testing

The easiest way to verify that an HCFS has been correctly installed and configured is to use it from the command line:
* `hadoop fs -ls hcfsname://localservice/`

The URI string (`hcfsname://localservice/`) should be adapted to whatever is appropriate for the scheme name and root URI of 
the particular HCFS.  It is important to include the trailing “/” at the end of the URI; this is to guarantee that the root 
of the filesystem is listed.  If omitted, then the user's home directory is listed, which may be absent.

After verifying that the contents of the filesystem can be listed, a simple series of `hadoop fs` operations to manipulate 
the filesystem can verify write and read access, as well as basic behavior: (For CLI syntax 
see https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/FileSystemShell.html )
* `-mkdir`
* `-rm`
* `-copyFromLocal`
* `-copyToLocal`

The `hadoop distcp` command should also be exercised.

Once command line functionality has been verified, the next step is to perform queries in Hive, Pig and Spark which use the 
filesystem as a source and destination of data.

### Troubleshooting

*The error message: "No FileSystem for scheme"*

1. The URI Scheme is incorrect (i.e it is is misspelled or simply wrong).
2. The implementation JAR is not on the classpath.
3. The implementation JAR is on the classpath, but a dependency is missing.  Examine the log of the application to see if this 
problem was detected.
4. The `META-INF/services/org.apache.hadoop.fs.FileSystem` file is absent from the HCFS implementation JAR
5. The `fs.AbstractFileSystem.hcfsname.impl` configuration parameter declaration is absent or 
incorrect.  (Occurs if client uses the FileContext APIs) 

*“ClassNotFoundException” when trying to use an HCFS scheme*

A class is registered as being the implementation of a filesystem scheme, but its implementation JAR or a dependency is not on 
the classpath.

*Basic command line file system operations failing with permission errors*

This usually means that any security setup, credentials, etc needed to authenticate with the HCFS are not supplied. Unless 
Kerberos is used for authentication, the specific configuration options to authenticate with an HCFS must be set in the 
environment/Hadoop configuration. If Kerberos is used, the user must be logged in.

*Submitted jobs failing with permission errors, even though command line operations work*

This can mean that the security credentials are not being set in, or propagated to, the worker nodes in an application.

*Client-side Split calculation during job submission takes a very long time or runs out of memory.*

This can be caused by the returned block size of the file system (as returned in `FileSystem.getStatus()`) being 0 or another 
low value. A minimum block size of a few tens of MB is necessary for work to be be efficiently divided up, even if the file 
system has no notion of "block size". We currently recommend a default value of 64 MB.  File systems which simulate a block 
size usually make this configurable to other values.

*Jobs using the HCFS pause at the end, in Hive and Spark queries, DistCp and elsewhere*

This arises if the HCFS is an object store which only supports directory rename through explicit copy and delete of individual 
files under the specific path.  The implementors of the HCFS client must try to produce as fast a rename() operation as possible, 
because it is required to atomically commit operations.  It may also be possible to provide a "direct" committer to write output 
without renaming.  Any HCFS implementor providing such a committer must ensure that their implementation handles race conditions 
from speculative execution and failure recovery from network partitioned 
executors (the “consistency, atomicity, and durability” requirements).

## Appendix – Apache jiras related to HCFS

For developers interested in the implementation discussions and issues related to HCFS, especially in Ambari, here is a selection of jiras relating to the early work:

`HADOOP-9361` - Strictly define the expected behavior of filesystem APIs and write tests to verify compliance 

`AMBARI-1817` - Create ability to add alternate storage system that uses Ambari management tools

`AMBARI-2118` - ambari-web modifications to allow for Hadoop Compatible Filesystems (HCFS)

`AMBARI-2119` - ambari-agent modifications to allow for Hadoop Compatible Filesystems (HCFS)

`AMBARI-2128` - ambari-server modifications to allow for Hadoop Compatible File System (HCFS)

`AMBARI-2130` - Use modified dependencies if a stack contains an HCFS service (Hadoop Compatible File System)


