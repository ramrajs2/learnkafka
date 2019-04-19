# Create a Message Daemon Using Maven Archetype

## Introduction

The correct way to create a message daemon project is to use Altus.
However, each time you do, you'll allocate a set of resources that may take some time to clean up (Git Repo, CI Jobs, Integration Servers, etc.).

In this lab, we'll show you have to create a message daemon using a maven archetype.

It is good enough for testing and experimenting locally, but this method should not be used for real development.

## Lab Instructions

### Create the archetype

Creating a project using the message daemon archetype requires two steps.

We will have to run the maven archetype twice.
There is a a generic maven archetype that we use to create artifacts that are common for all message daemon.
After we've run that archetype, we'll run another partial archetype that expands artifacts that are unique for a particular messaging technology.

You should be able to use the instructions in this document to create a messaging daemon on any technology.

To be able to separate the various stacks, we have documented the variations using the following form:

  ${OPTION_A|OPTION_B|OPTION_C}

You will have to replace the complete statement with one of options in the curly braces.

#### 0. Prerequisite

All of these steps are performed in a terminal window. This terminal window must have all the required tools available:
- Java
- Maven

Makes sure you're in a directory where you want to create the project.

#### 1. Run the maven archetype command

The first step is to run the generic message daemon archetype.
This archetype sets up the common files across the various technologies.

You can copy the command below (but notice that you have to pick one of the options):

```bash
mvn -U org.apache.maven.plugins:maven-archetype-plugin:RELEASE:generate \
    -DarchetypeGroupId=com.paypal.platform \
    -DarchetypeArtifactId=raptor-message-daemon \
    -DarchetypeVersion=3.3.0
```

You'll see some output (perhaps more than shown below) and finally maven will prompt you to enter some parameters:

```
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building Maven Stub Project (No POM) 1
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] >>> maven-archetype-plugin:2.4:generate (default-cli) > generate-sources @ standalone-pom >>>
[INFO]
[INFO] <<< maven-archetype-plugin:2.4:generate (default-cli) < generate-sources @ standalone-pom <<<
[INFO]
[INFO] --- maven-archetype-plugin:2.4:generate (default-cli) @ standalone-pom ---
[INFO] Generating project in Interactive mode
[INFO] Archetype repository not defined. Using the one from [com.paypal.platform:raptor-message-daemon:3.1.0 -> https://engineering.paypalcorp.com/paypalcentral/content/groups/public] found in catalog https://paypalcentral.es.paypalcorp.com/nexus/content/groups/public/archetype-catalog.xml
Define value for property 'groupId': : com.paypal.raptor.samples
Define value for property 'artifactId': : sampled
Define value for property 'version':  1.0-SNAPSHOT: :
Define value for property 'package':  com.paypal.raptor.samples: :
[INFO] Using property: appName = Default value is lowercase artifactId.
[INFO] Using property: projectDescription = Required for application registration and deploy to QA/Prod
[INFO] Using property: raptorPlatformVersion = 3.3.0
[INFO] Using property: starterAMQ = false
[INFO] Using property: starterKafka = false
[INFO] Using property: starterYAM = false
Confirm properties configuration:
```

Use the following values:

| Option | Suggested value |
|--------|-----------------|
| groupId | com.paypal.raptor.samples |
| artifactId | sampled |
| version | 1.0-SNAPSHOT |
| package | com.paypal.raptor.samples |

#### 4. Patch the archetype with the technology to use

Our next step is to patch the existing archetype with the technology you want to use.
The technology should of must match the starter that you used above.

Your options are one of:
- `raptor-message-daemon-yam`
- `raptor-message-daemon-amq`
- `raptor-message-daemon-kafka`

You can copy the command below (but notice that you have to pick one of the options):
```bash
mvn -U org.apache.maven.plugins:maven-archetype-plugin:RELEASE:generate \
    -DarchetypeVersion=3.3.0 \
    -DarchetypeGroupId=com.paypal.platform \
    -DarchetypeArtifactId=raptor-message-daemon-${yam|amq|kafka}
```

You will see the archetype expand and then be promted for some properties again:

```
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building Maven Stub Project (No POM) 1
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] >>> maven-archetype-plugin:2.4:generate (default-cli) > generate-sources @ standalone-pom >>>
[INFO]
[INFO] <<< maven-archetype-plugin:2.4:generate (default-cli) < generate-sources @ standalone-pom <<<
[INFO]
[INFO] --- maven-archetype-plugin:2.4:generate (default-cli) @ standalone-pom ---
[INFO] Generating project in Interactive mode
[INFO] Archetype repository not defined. Using the one from [com.paypal.platform:raptor-message-daemon-yam:3.3.0 -> https://engineering.paypalcorp.com/paypalcentral/content/groups/public] found in catalog https://paypalcentral.es.paypalcorp.com/nexus/content/groups/public/archetype-catalog.xml
Downloading: https://paypalcentral.es.paypalcorp.com/nexus/content/repositories/central/com/paypal/platform/raptor-message-daemon-yam/3.3.0/raptor-message-daemon-yam-3.3.0.jar
Downloading: https://paypalcentral.es.paypalcorp.com/nexus/content/groups/public/com/paypal/platform/raptor-message-daemon-yam/3.3.0/raptor-message-daemon-yam-3.3.0.jar
Downloaded: https://paypalcentral.es.paypalcorp.com/nexus/content/groups/public/com/paypal/platform/raptor-message-daemon-yam/3.3.0/raptor-message-daemon-yam-3.3.0.jar (8 KB at 12.5 KB/sec)
Downloading: https://paypalcentral.es.paypalcorp.com/nexus/content/repositories/central/com/paypal/platform/raptor-message-daemon-yam/3.3.0/raptor-message-daemon-yam-3.3.0.pom
Downloading: https://paypalcentral.es.paypalcorp.com/nexus/content/groups/public/com/paypal/platform/raptor-message-daemon-yam/3.3.0/raptor-message-daemon-yam-3.3.0.pom
Downloaded: https://paypalcentral.es.paypalcorp.com/nexus/content/groups/public/com/paypal/platform/raptor-message-daemon-yam/3.3.0/raptor-message-daemon-yam-3.3.0.pom (812 B at 1.5 KB/sec)
Define value for property 'groupId': : com.paypal.raptor.samples
Define value for property 'artifactId': : sampled
Define value for property 'version':  1.0-SNAPSHOT: :
Define value for property 'package':  com.paypal.raptor.samples: :
[INFO] Using property: appName = Default value is lowercase artifactId.
[INFO] Using property: raptorPlatformVersion = 3.3.0
Confirm properties configuration:
```

Make sure you enter the same properties as when you expanded the initial archetype.

If you followed the instruction, those values should be:

| Option | Suggested value |
|--------|-----------------|
| groupId | com.paypal.raptor.samples |
| artifactId | sampled |
| version | 1.0-SNAPSHOT |
| package | com.paypal.raptor.samples |

#### 3. CD into the project

```bash
cd sampled
```

#### 4. Build the project

Use maven to build the project to ensure correctness.

```bash
mvn package
```

You should see some output ending with:

```
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 51.705 s
[INFO] Finished at: 2016-04-21T11:31:30-05:00
[INFO] Final Memory: 58M/581M
[INFO] ------------------------------------------------------------------------
```

#### 5. Import the project into your IDE

Import the project into your IDE using the same wizards as we used before.
