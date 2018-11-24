Docs 
Forums
Training
Try Gradle Enterprise
User Manual
Tutorials
DSL Reference
Release Notes
Getting Started
Creating New Gradle Builds
Creating Build Scans
Migrating From Maven
Project Tutorials
Android
C++
Groovy
Java
JavaScript
Kotlin
Scala
Integrating Gradle
Continuous Integration
Extending Gradle
Plugin Development Guides
Developing Parallel Tasks
Consuming JVM Libraries
Contents
What you’ll build
What you’ll need
Create project
Create the application
Licensing
Summary
Next Steps
Help improve this guide
This guide demonstrates how to create a Java application which consumes an external library.

What you’ll build
You’ll generate a Java application with the standard layout. You will then add a third-party library to your application, build it and package it. The final product will be a greeter application that produces an ASCII-art type greeting.

What you’ll need
About 12 minutes

A text editor

A command prompt

The Java Development Kit (JDK), version 1.7 or higher

A Gradle distribution, version 4.10-rc-2 or better

Create project
The first step is to create a folder for the new project and add a Gradle Wrapper to the project.

$ mkdir consuming-jvm-libraries
$ cd consuming-jvm-libraries
$ gradle wrapper 

:wrapper

BUILD SUCCESSFUL
This allows a version of Gradle to be locked to a project and henceforth you can use ./gradlew instead of gradle.
Instead of using the built-in wrapper task, you may wish to use the init task from the Build Init plugin instead. This will create the initial project structure for you. See the Building Java Applications guide for a detailed explanation of how to do this.
Create a settings.gradle file and set the name of the project. Doing this will ensure that the project builds with the correct name regardless of the name of the project root folder.

settings.gradle
rootProject.name='greeterApp'
Now create a build.gradle file and apply the Java plugin.

build.gradle
apply plugin : 'java' 
That is enough to build a self-contained Java project, but you will be creating a greeter application, which will print the greeting in Ascii-art. For that an external library is required. Continue editing build.gradle and add a repositories block.

build.gradle
repositories {
    jcenter() 
}
Use JCenter as the repository.
In order to find artifacts you have to tell Gradle where to look. Gradle supports two specialized repository types - Maven & Ivy, among others. In addition, Gradle supports simplified configuration for the most popular centralized repositories - JCenter, Maven Central, and Google’s Android repository. In this guide you are using JCenter as it has access to all of the repositories hosted on Maven Central as well as many more that are published to Bintray.

The next step is to tell Gradle which external artifacts you need for your projects. As you are doing an Ascii-art application you will be using JFiglet. Add a dependencies block to build.gradle for the JFiglet library.

build.gradle
dependencies {
    implementation 'com.github.lalyos:jfiglet:0.0.8' 
}
Add the JFiglet dependency to the implementation configuration.
Gradle supports a variety of notations. The one used above is probably the most popular, which uses what is most commonly known as Maven coordinates.

Adding a dependency has two parts: one is the dependency itself and the other is the configuration to which it is added. The latter term is used in Gradle to effectively group dependencies together by context. The current build uses the implementation configuration, which is provided by the Java Plugin.

You can learn more about using configurations for your own custom purposes by studying the ConfigurationContainer.
The purpose of the implementation configuration is to collect dependencies that are used by a library or application and add them to the compilation classpath, but not export them via any of its APIs. As this is a stand-alone application, all dependencies can be placed in this configuration for purposes of application construction. This makes the use of the JFiglet library an implementation detail that can be changed a later date without affecting any clients.

You can inspect all of the dependencies you have added on a per-configuration basis, by using the dependencies task that is built into Gradle.

./gradlew dependencies --configuration implementation 


> Task :dependencies

------------------------------------------------------------
Root project
------------------------------------------------------------

implementation - Implementation only dependencies for source set 'main'. (n)
\--- com.github.lalyos:jfiglet:0.0.8 (n)

(n) - Not resolved (configuration is not meant to be resolved)

A web-based, searchable dependency report is available by adding the --scan option.

BUILD SUCCESSFUL in 5s
1 actionable task: 1 executed
The --configuration parameter restricts inspection to a single configuration.
If you are using Gradle 4.0 or later you may see less output from the console that you might see in this guide. In this guide, output is shown using the --console-plain flag on the command-line. This is done to show the tasks that Gradle is executing.
Create the application
Proceed to creating a src/main/java folder and place a GreeterApp.java file within that folder containing the following source code

src/main/java/GreeterApp.java
import java.io.IOException;
import com.github.lalyos.jfiglet.FigletFont;

public class GreeterApp {
    public static void main(String[] args) throws IOException {
        String asciiArt = FigletFont.convertOneLine("Hello, " + args[0]);
        System.out.println(asciiArt);
    }
}
Due to simplicity of this two line application, testing will not be included in this example.
Build your application by using the jar task.

$ ./gradlew jar

> Task :compileJava
> Task :processResources NO-SOURCE
> Task :classes
> Task :jar

BUILD SUCCESSFUL in 2s
2 actionable tasks: 2 executed
As this is an application, it will be useful to distribute it. Edit build.gradle again and add the Application plugin.

build.gradle
apply plugin : 'application' 
mainClassName = 'GreeterApp' 
The Application plugin is a very useful plugin that allows you to bundle your application along with all of its dependencies.
This configures the entry point into the application, which needs to be a class with a class (static) method called main.
Finish building your application by using the build task.

$ ./gradlew build

> Task :compileJava UP-TO-DATE
> Task :processResources NO-SOURCE
> Task :classes UP-TO-DATE
> Task :jar UP-TO-DATE
> Task :startScripts
> Task :distTar
> Task :distZip
> Task :assemble
> Task :compileTestJava NO-SOURCE
> Task :processTestResources NO-SOURCE
> Task :testClasses UP-TO-DATE
> Task :test NO-SOURCE
> Task :check UP-TO-DATE
> Task :build

BUILD SUCCESSFUL in 0s
5 actionable tasks: 3 executed, 2 up-to-date
If you inspect the build/distributions folder you will notice both .zip and .tar archives. This is the application ready for distribution. Now it is time to test your application manually. The Application plugin provides a useful installDist task to install your application into the build/install folder for validation purposes.

$ ./gradlew installDist

> Task :compileJava UP-TO-DATE
> Task :processResources NO-SOURCE
> Task :classes UP-TO-DATE
> Task :jar UP-TO-DATE
> Task :startScripts UP-TO-DATE
> Task :installDist

BUILD SUCCESSFUL in 0s
4 actionable tasks: 1 executed, 3 up-to-date
Look in build/install where you should find a greeterApp folder containing lib and bin folders. If you look inthe lib folder you will see both your application and the JFiglet JARs.

Run the application by changing to the build/install/greeterApp folder and executing the application.

$ cd build/install/greeterApp
$ ./bin/greeterApp Gradle

  _   _          _   _                  ____                      _   _
 | | | |   ___  | | | |   ___          / ___|  _ __    __ _    __| | | |   ___
 | |_| |  / _ \ | | | |  / _ \        | |  _  | '__|  / _` |  / _` | | |  / _ \
 |  _  | |  __/ | | | | | (_) |  _    | |_| | | |    | (_| | | (_| | | | |  __/
 |_| |_|  \___| |_| |_|  \___/  ( )    \____| |_|     \__,_|  \__,_| |_|  \___|
                                |/
Congratulations! You have just created an application which consumes a third-party library from an external repository.

Licensing
Third-party JVM libraries are released under various licenses. It is important to always check the licenses of the libraries you use, including all of the libraries that are introduced via transitive dependencies, for compatibility.

For example the JFiglet library is released under GPL 2.0. This means that should you want to release the application you have just built, it will also need to be under GPL 2.0.

You can utilise Jeroen van Erp’s excellent License plugin to create a report on the licenses of all of the dependencies of your project.

Summary
You now have JVM application that consumes an external library. In this process you saw:

How to configure repositories and dependencies.

Build an application consuming external dependencies.

Distribute an application with all external dependencies.

Next Steps
Read about using your own Maven repository installations

Read about configuring Ivy repositories.

Read about finding artifacts without metadata in local folders.

Read about advanced artifact resolution and how to deal with version & dependency conflicts.

Help improve this guide
Have feedback or a question? Found a typo? Like all Gradle guides, help is just a GitHub issue away. Please add an issue or pull request to gradle-guides/consuming-jvm-libraries and we’ll get back to you.

Docs
User Manual
DSL Reference
Release Notes
Javadoc
News
Blog
Newsletter
Twitter
Products
Build Scans
Build Cache
Enterprise Docs
Get Help
Forums
GitHub
Training
Services
Subscribe for important Gradle updates and news


name@email.com
Subscribe
By entering your email, you agree to our Terms and Privacy Policy, including receipt of emails. You can unsubscribe at any time.

© Gradle Inc. 2018 All rights reserved.
Careers | Privacy | Terms of Service | Contact