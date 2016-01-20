# How to contribute to the ODPi

Thank you for your interest in contributing to the ODPi!

To start off, please ensure you have completed a Individual CLA for yourself. If you are contributing on behalf of an organization, you will also need to have the organization complete a Corporate CLA as well. See the links below for the process:

[Contribute as an individual](https://identity.linuxfoundation.org/node/142/individual-signup)

[Contribute as an organization](https://identity.linuxfoundation.org/node/142/organization-signup)

## Ways to contribute

There are two key areas you can contribute to the ODPi efforts.

1. Provide feedback on the specs
2. Write tests to help validate an app or distro is compliant with the specs

### Contributing tests

The ODPi testing toolset is based on a fork of Apache BigTop 1.0.0, and leverages the existing BigTop smoke tests along with additional tests that verify ODPi compliance.

Here's just a quick outline on the existing examples of the integration tests. These examples could be used to start writing spec tests. The following explains how smoke tests are developed right now. All smoke tests related activities should be carried on from odpi-master branch of odpi/bigtop repo. Same `repo/branch` should be used if you're standing up a cluster with Bigtop provisioner, e.g. `./gradlew -Pnum_instances=3 -Prun_smoke_tests=true docker-provisioner`

#### Framework location
Smoke tests are ran as gradle integration tests. All existing smoke tests (similar to future spec tests are located under `bigtop-tests/smoke-tests` directory.

#### Tests location

Smoke tests for different components could be found under `bigtop-tests/smoke-tests` in the directories named after particular modules. Source code of a test suite might be co-located with the `build` directrory, or borrowed from another project. In this case, all hdfs tests source code is actually located in the maven project, yet executed as smokes.
Source code directories are controlled by component's `build.gradle` file via sourceSets element.

#### Tests

Tests could be written in Java, or Groovy. Tests can directly use JUnit APIs and will be correctly executed by the test runner. Optionally, tests could use facilities provided by iTest - Bigtop integration test framework. Among other things, iTest provide very conveninet org.apache.bigtop.itest.shell.Shell API, which adds bash scripting to the set of available languages. Shell class also has the capabilities to change the effective user owning exec'ed shell process. That requires passworless sudo to be configured.

#### Executing the smoke tests

Tests could be executed from the top-level directory as the following command (in this case for hdfs smokes)
````
./gradlew bigtop-tests:smoke-tests:hdfs:test -Psmoke.tests
````
Optional `--info` could be added for higher verbosity.
If tests log level needs to be changes, it could be done in `bigtop-tests/smoke-tests/logger-test-config/src/main/resources/log4j.properties` file

#### Examples of Spec tests

The examples of how to write spec tests could be found under
https://github.com/odpi/bigtop/tree/spec-tests/bigtop-tests/spec-tests
Adding new tests is simple and is done via adding new test descriptions to 
`runtime/src/test/resources/testRuntimeSpecConf.groovy` file. For now spec-tests are under development and aren't available from the main odpi-master branch
To run spec tests you can follow the already familiar
````
./gradlew bigtop-tests:spec-tests:runtime:test -Pspec.tests
````

## Additional resources

You can join the community conversation by subscribing to the mailing list below.

https://lists.odpi.org/mailman3/lists/odpi-technical.lists.odpi.org/
