---
permalink: /guides/nexus-repository-integration-with-appsody/
layout: guide-markdown
title: Nexus Repository Integration with Appsody
duration: 40 minutes
releasedate: 2019-12-6
description: TODO
tags: ['Appsody', 'Codewind', 'Java', 'Nodejs']
guide-category: none
---

<!---
>	Copyright 2019 IBM Corporation and others.
>
>	Licensed under the Apache License, Version 2.0 (the "License");
>	you may not use this file except in compliance with the License.
>	You may obtain a copy of the License at
>
>	http://www.apache.org/licenses/LICENSE-2.0
>
>	Unless required by applicable law or agreed to in writing, software
>	distributed under the License is distributed on an "AS IS" BASIS,
>	WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
>	See the License for the specific language governing permissions and
>	limitations under the License.
--->

<!---
> NOTE: This repository contains the guide documentation source. To view
> the guide in published form, view it on the https:kabanero.io/guides/{projectid}.html[website].
--->

## What you will learn

Nexus Repository is a popular repository manager that provides a single source of truth for all of the software components used by the applications in an enterprise. It provides a single access and control point for Maven (Java), npm (Node JS) and other software dependencies. It can also be used to manage or govern dependencies, or reduce the build dependencies on multiple external repositories and internet access.


With Appsody the owner of each stack can provide a default repository manager, force the use of a specific repository manager instance, or block the use of a repository manager. The current Java Microprofile and Node JS Express stacks restrict repository access, so if you want to use Nexus Repository you'll need to create your own stack. You can start by copying and customizing an existing stack rather than starting from scratch.


The remainder of this guide will walk you through the steps required to configure a Nexus Repository for both a Java and Node JS stack. If you are not familiar with customizing stacks and collections, there is a guide at Kabanero.io that provides some background and walks you through all of the required steps: [https://kabanero.io/guides/working-with-collections](https://kabanero.io/guides/working-with-collections). If you're already familiar with customizing stacks and have your own, you may want to skip to steps 4 & 5 (for Maven) and steps 2-5 (for NPM) to focus just on the Nexus Repository configuration. Following these steps will create an Appsody stack that uses Nexus Repository management for both development and build time of your applications.

## Creating an Appsody stack that pulls from a Maven proxy


**Prerequisites:**

- Have Nexus Repository Manager 3 installed ([https://help.sonatype.com/repomanager3/installation](https://help.sonatype.com/repomanager3/installation))
- Your Maven proxy has been setup on Nexus Repository Manager 3 ([https://help.sonatype.com/repomanager3/formats/maven-repositories](https://help.sonatype.com/repomanager3/formats/maven-repositories))
- Appsody has been installed ([https://appsody.dev/docs/getting-started/installation](https://appsody.dev/docs/getting-started/installation))
- Codewind has been installed on either VS Code or Eclipse ([https://www.eclipse.org/codewind/gettingstarted.html](https://www.eclipse.org/codewind/gettingstarted.html))
- Docker is installed
- Maven is installed
- The settings.xml with the server credentials is passed to the various Maven calls. However, on the final Appsody image, the settings.xml is removed after all the Maven calls are made to ensure the credentials are not shared outside of development


**In this example...**

- We will assume the IP address of your Nexus Repository Manager is at http://9.108.127.66:8081 and your Maven public proxy URL is http://9.108.127.66:8081/repository/maven-public/. Do not use localhost as some Maven commands are run in a Docker container and localhost will not resolve correctly
- Your Nexus Repository Manager is using the default credentials with user name `admin` and password `admin123`
- You are creating a Maven project based on Appsody's java-microprofile stack
- The complete example can be found via [my-java-microprofile](code/my-java-microprofile)


**Steps:**

1. Create a new Appsody Maven stack by running `appsody stack create my-java-microprofile --copy incubator/java-microprofile` on your operating system terminal
1. Ensure you have your settings.xml file in your Maven home directory (e.g. ~/.m2). An example settings.xml can be found via [settings.xml](code/my-java-microprofile/image/project/settings.xml)
1. Go to the generated my-java-microprofile directory
1. For every pom.xml, the Maven proxy needs to be added as a `<repository>`. The files image/project/pom.xml, image/project/pom-dev.xml, and /image/templates/default/pom.xml need to be modified. See the following files for all the changes:
    - [project's pom.xml](code/my-java-microprofile/image/project/pom.xml)
    - [project's pom-dev.xml](code/my-java-microprofile/image/project/pom-dev.xml)
    - [template's pom.xml](code/my-java-microprofile/templates/default/pom.xml)
1. For any mention of `mvn` or `mvnw`, the settings.xml needs to be passed in to ensure the proper settings.xml configuration needs to be used.  Note: .appsody-init.bat and .appsody-init-sh calls from your operating system, so it references ~/.m2/settings.xml, not /.m2/settings.xml. See the following files for all the changes:
    - [.appsody-init.bat](code/my-java-microprofile/image/project/.appsody-init.bat)
    - [.appsody-init.sh](code/my-java-microprofile/image/project/.appsody-init.sh)
    - [Dockerfile](code/my-java-microprofile/image/project/Dockerfile)
    - [install-dev-deps.sh](code/my-java-microprofile/image/project/install-dev-deps.sh)
    - [validate.sh](code/my-java-microprofile/image/project/validate.sh)
    - [Dockerfile-stack](code/my-java-microprofile/image/Dockerfile-stack)
        - `io.takari:maven:wrapper` is changed to `io.takari:maven:0.6.1:wrapper` due to an authentication issue with the latest wrapper ([issue](https://github.com/takari/maven-wrapper/issues/142))
1. Go to the root directory of your stack and run `appsody stack package`.  For the purpose of this example, the publishing is done into the local dev.local and the images are not pushed to Docker hub. To push to Docker hub, follow the [publishing instructions in the Appsody documentation](https://appsody.dev/docs/stacks/publish)
1. The Appsody command line will output logs. Ensure you only see your proxy being used and not the official Maven repository (https://repo1.maven.org/maven2). For example, we want to see logs like: `[Docker] [INFO] Downloading from nexus: http://9.108.127.66:8081/repository/maven-public/org/apache/maven/maven-model/3.2.3/maven-model-3.2.3.jar`
1. Run `appsody stack add-to-repo sghung --release-url https://github.com/sghung/appsodystacks/releases/latest/download/` in your Appsody stack root directory. Replace the repository with your own Github repository
1. Go to ~/.appsody/stacks/dev.local. By default, Appsody home is ~/.appsody. Your generated files will be located here. The important files are:
    - my-java-microprofile.v0.2.21.templates.default.tar.gz
    - sghung-index.yaml
1. For Codewind to pickup the stacks, a sghung-index.json needs to be generated. The format of the JSON format is of the form: [sghung-index.json](code/sghung-index.json)
    - Copy your displayName and description from sghung-index.yaml
    - Update the language to Java
    - Ensure the location will be where you upload the stack on Github
    - The link can be left as is. It is required by Codewind but will not be used
1. Upload your files as a release onto Github. For example: https://github.com/sghung/appsodystacks/releases/tag/0.1.0
1. Open Codewind and go to "Manage Template Sources"
1. Add your JSON file. For example: https://github.com/sghung/appsodystacks/releases/download/0.1.0/sghung-index.json
1. Create a new Codewind project and you should see your repo and my-java-microprofile
1. Choose it and a directory of your choosing to install the files into
1. View the project logs to ensure it is downloading from your Maven proxy
1. The application should go into a running state and can be used for development


## Creating an Appsody stack that pulls from a NPM proxy


**Prerequisites:**

- Have Nexus Repository Manager 3 installed ([https://help.sonatype.com/repomanager3/installation](https://help.sonatype.com/repomanager3/installation))
- Your NPM proxy has been setup on Nexus Repository Manager 3 ([https://help.sonatype.com/repomanager3/formats/npm-registry](https://help.sonatype.com/repomanager3/formats/npm-registry))
- Appsody has been installed ([https://appsody.dev/docs/getting-started/installation](https://appsody.dev/docs/getting-started/installation))
- Codewind has been installed on either VS Code or Eclipse ([https://www.eclipse.org/codewind/gettingstarted.html](https://www.eclipse.org/codewind/gettingstarted.html))
- Docker is installed
- NPM is installed


**In this example...**

- We will assume the IP address of your Nexus Repository Manager is at http://9.108.127.66:8081 and your NPM public proxy URL is http://9.108.127.66:8081/repository/npm-all/. Do not use localhost as some Maven commands are run in a Docker container and localhost will not resolve correctly
- Your Nexus Repository Manager is using the default credentials with user name `admin` and password `admin123`
- You are creating a Maven project based on Appsody's nodejs-express stack
- The complete example can be found via [my-nodejs-express](code/my-nodejs-express)
- For the NPM proxy, the logs do not show logs that it is pulling from the NPM proxy. Instead, browse the NPN proxy to ensure it is being populated
- The sampleCredential file included in this example should not be checked into a repository. This file is just for this guide to show the format


**Steps:**

1. Create a new Appsody Maven stack by running `appsody stack create my-nodejs-express --copy incubator/nodejs-express` on your operating system terminal
1. From following the NPM proxy information in the prerequisites, you should have an encrypted authentication. For the default password of `admin123`, the value is `_auth=YWRtaW46YWRtaW4xMjM=`. Create a credentials file in image/project and add the server credentials for the registry. An example of the final file can be found via [sampleCredentials](code/my-nodejs-express/image/project/sampleCredentials). Make sure this file does not get checked into your repository to avoid the credentials being stored inappropriately
1. Search for "npm audit" and remove all mentions of it. The removal is required due to this [issue](https://issues.sonatype.org/browse/NEXUS-16954)
1. Modify the Dockerfile-stack and Dockerfile to use .npmrc before calling any `npm install` commands
    - [Dockerfile-stack](code/my-nodejs-express/image/Dockerfile-stack)
    - [Dockerfile](code/my-nodejs-express/image/project/Dockerfile)
    - For both these files, the .npmrc is removed after `npm install` is called to avoid having the credentials show up in the Docker image
1. Go to the root directory of your stack and run `appsody stack package`.  For the purpose of this example, the publishing is done into the local dev.local and the images are not pushed to Docker hub. To push to Docker hub, follow the [publishing instructions in the Appsody documentation](https://appsody.dev/docs/stacks/publish)
1. Run `appsody stack add-to-repo sghung2 --release-url https://github.com/sghung/appsodystacks/releases/latest/download/` in your Appsody stack root directory. Replace the repository with your own Github repository
1. Upload your files as a release onto Github. For example: https://github.com/sghung/appsodystacks/releases/tag/0.1.1
1. Open Codewind and go to "Manage Template Sources"
1. Add your JSON file. For example: https://github.com/sghung/appsodystacks/releases/download/0.1.1/sghung2-index.json
1. Create a new Codewind project and you should see your repo and my-nodejs-express
1. Choose it and a directory of your choosing to install the files into
1. The application developer must put the .npmrc file into the root directory of the project. It should not be packaged up into the stack's template or be checked into the repository. The stack architect needs to inform the application developer that credentials are needed and securely share the credentials with the application developer. The contents of the .npmrc file will be the same as [sampleCredentials](code/my-nodejs-express/image/project/sampleCredentials)
1. The application should go into a running state and can be used for development