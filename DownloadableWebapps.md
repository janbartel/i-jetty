# Creating Downloadable Web Applications #

Making a webapp that is dynamically downloadable by i-jetty is very much like making a normal webapp. The only tricky step is that you need to run the `dx` tool over the classes that would normally be in `WEB-INF/classes` and the jars that would normally be in `WEB-INF/lib`.

You can see examples of how we do that with maven in the [Console](ConsoleWebApplication.md) web application and the Hello World web application.

We do three things specially for Android:

  1. we unpack the classes/resources out of all of the dependency jars of the webapp
  1. we run the `dx` tool over the webapp's own classes and all of the classes from the dependency jars
  1. we `zip` up the `classes.dex` output from `dx` and put it into the webapp's `WEB-INF/lib` directory

#### Unpacking the dependencies ####
```
       <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-dependency-plugin</artifactId>
        <version>2.3</version>
        <executions>
          <execution>
            <id>unpack-dependencies</id>
            <phase>generate-sources</phase>
            <goals>
              <goal>unpack-dependencies</goal>
            </goals>
            <configuration>
              <failOnMissingClassifierArtifact>false</failOnMissingClassifierArtifact>
              <excludeArtifactIds>servlet-api,android</excludeArtifactIds>
              <excludeTransitive>true</excludeTransitive>
              <outputDirectory>${project.build.directory}/generated-classes</outputDirectory>
            </configuration>
          </execution>
        </executions>
      </plugin>
```

#### Using dx ####

We hook the dx tool into the build of the webapp at the `process-classes` phase of the maven build lifecycle:

```
<plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>exec-maven-plugin</artifactId>
        <version>1.2</version>
        <executions>
          <!-- Convert the compiled classes into a clases.dex. -->
          <execution>
            <id>generate-dex</id>
            <phase>process-classes</phase>
            <goals>
              <goal>exec</goal>
            </goals>
            <configuration>
              <executable>java</executable>
              <arguments>
                <argument>-jar</argument>
                <argument>${env.ANDROID_HOME}/platform-tools/lib/dx.jar</argument>
                <argument>--dex</argument>
                <argument>--verbose</argument>
                <argument>--core-library</argument>
                <argument>--output=${project.build.directory}/classes.dex</argument>
                <argument>--positions=lines</argument>
                <argument>${project.build.directory}/classes/</argument>
                <argument>${project.build.directory}/generated-classes/</argument>
              </arguments>
            </configuration>
          </execution>
        </executions>
      </plugin>
```

#### Zipping the classes.dex file ####
```
      <plugin>
        <artifactId>maven-antrun-plugin</artifactId>
        <version>1.6</version>
        <executions>
          <execution>
            <id>copydex</id>
            <phase>process-classes</phase>
            <goals>
              <goal>run</goal>
            </goals>
            <configuration>
              <tasks>
                <mkdir
                  dir="${project.build.directory}/${project.artifactId}-${project.version}/WEB-INF/lib" />
                <jar
                  basedir="${project.build.directory}"
                  update="true"
                  includes="classes.dex"
                  destfile="${project.build.directory}/${project.artifactId}-${project.version}/WEB-INF/lib/classes.zip" />
              </tasks>
            </configuration>
          </execution>
        </executions>
      </plugin>
```

## Using i-jetty to Download a WebApp ##
<font color='red'>**NOTE: Android SDK 1.5 and above is needed to support dynamic webapps.</font>**

After you've prepared your web app - you might like to read the section below on Sample Webapps for some hints - you need to download it to the phone.

First, you'll need to arrange for the webapp to be served from a http:// url. The easiest way to do this is to serve it from a [Jetty](http://jetty.mortbay.org) installation on your desktop machine. The Jetty site has full instructions on how to download and install it.

Once you've got Jetty installed - and lets assume you installed it to $JETTY\_HOME - make a directory called $JETTY\_HOME/webapps/downloads.  Now copy your android-ready war file to that directory. Start Jetty. Your webapp will now be available as a downloadable file from the url `http://[your host ip]/downloads/[your webapp name]`. So for example, if your host ip address is `10.10.1.10` and you build a webapp called `myfirstwebapp.war`, then it is downloadable from `http://10.10.1.10/downloads/myfirstwebapp.war`.

Now, go to your phone or emulator and start i-jetty if its not already running. Select the **Download** button and type in the url of your webapp. Then enter the context path at which you want that webapp available. For example, you might want to access it as "/howcoolami".

i-jetty will now download your webapp and install it to the phone.

At the time of writing, you will need to restart i-jetty to get it to deploy your new webapp, although we are working on providing a hot deploy capability.


## Sample WebApps ##

The i-jetty project comes with a couple of example webapps that you can build, and then dynamically download with i-jetty to get you going with the concept of dynamic webapps in the android environment.


### Hello World Example ###

The "HelloWorld" web application in `$I_JETTY_HOME/example-webapps/hello` prints out the famous "Hello World" message. The web app contains one very simple Servlet as an example of how to prepare a web app that has classes in addition to static content.

The [pom.xml](http://code.google.com/p/i-jetty/source/browse/trunk/example-webapps/hello/pom.xml) for HelloWorld does a couple of things to make the webapp android-ifiable (if I can invent that word):

  * runs the `dx` tool over all the java classes in the webapp to produce a `classes.dex` file
  * zips up the `classes.dex` output file from `dx`

That's all you need to do for a simple webapp that uses no libraries. For more complex webapps that use 3rd party libs, refer to the "Chat" webapp.

### Chat Example ###

The "Chat" web application is a much more sophisticated beast than "HelloWorld". It uses 3rd party java libraries, such as the [Jetty](http://jetty.mortbay.org) implementation of the [cometd Bayeux protocol](http://docs.codehaus.org/display/JETTY/Cometd+(aka+Bayeux)), along with the [dojox.cometd](http://cometdproject.dojotoolkit.org/documentation/cometd-dojox) javascript library to implement the now-classic Ajax chat room.

Again, this is essentially a straight-forward webapp, with the only added complication that the 3rd party jars need to be dex-ified. Looking at the [pom.xml](http://code.google.com/p/i-jetty/source/browse/trunk/modules/chat/pom.xml) for the Chat application, you'll see that prior to the 2 steps outlined above in the HelloWorld app, the build first unpacks the classes from the dependency jars. Thus, during the `dx` step, all classes - both those of the webapp and those from the libraries it uses - are all compiled into the `classes.dex` and then `classes.zip` file.

### Other Examples ###

[HybridServerPages](http://www.hybridserverpages.com) has a Sample application permanently running on http://www.hybridjava.com:8080/HJ_Sample/, which can be run under i-jetty on Android. It is downloadable via the web or via i-jetty's **Download** feature from http://www.hybridjava.com:8080/HJ.war.