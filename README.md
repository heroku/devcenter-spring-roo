This guide will show you how to deploy a Spring Roo application to Heroku using the Spring Roo Heroku plugin.

Sample code is available on [github](https://github.com/heroku/devcenter-spring-roo) along with this document. Edits and enhancements are welcome. Just fork the repository, make your changes and send us a pull request.

## Prerequisites

* Java, Maven, Git, the Heroku client, and Foreman (as described in the [basic Java quickstart](/java))
* An installed version of [Postgres](http://www.postgresql.org/) to test locally
* Basic knowledge of [Spring Roo](http://www.springsource.org/spring-roo) and an installed version  [1.2.0.RC1](http://s3.amazonaws.com/dist.springframework.org/milestone/ROO/spring-roo-1.2.0.RC1.zip) or later of the framework.

## Start the Roo Shell

Create a new directory for your application and start the Roo shell from inside the directory with

    :::term
    $ roo
        ____  ____  ____  
       / __ \/ __ \/ __ \ 
      / /_/ / / / / / / / 
     / _, _/ /_/ / /_/ /  
    /_/ |_|\____/\____/    1.2.0.RC1 [rev dcaa483]


    Welcome to Spring Roo. For assistance press TAB or type "hint" then hit ENTER.
    @lillevik: Best talk at #javaone so far was Java Enterprise Applications in the Cloud: Fast, Fun, and Easier Than Ever. Spring Roo seems interesting. 
    roo>

Note that depending on how you installed roo, the command may be `roo.sh` or `roo.bat`.

## Create a New Application

Use the `clinic.roo` sample script that ships with Spring Roo to create a complete web application.

    roo> script --file clinic.roo
    project --topLevelPackage com.springsource.petclinic
    Created ROOT/pom.xml
    Created SRC_MAIN_JAVA
    Created SRC_MAIN_RESOURCES
    Created SRC_TEST_JAVA
    ...
    logging setup --level INFO
    Updated SRC_MAIN_RESOURCES/log4j.properties
    Script required 19.753 seconds to execute
    ~.web roo> 

You now have a fully functional web application project.

## Switch to Postgres

Configure your application to use Postgres as database instead of the in-memory database that was configured with the out-of-the-box script:

    ~.web roo> persistence setup --provider HIBERNATE --database POSTGRES 
    Updated SRC_MAIN_RESOURCES/META-INF/spring/database.properties
    Please update your database details in src/main/resources/META-INF/spring/database.properties.
    Updated ROOT/pom.xml [added dependency postgresql:postgresql:9.0-801.jdbc3; removed dependency org.hsqldb:hsqldb:1.8.0.10]
    Updated SRC_MAIN_RESOURCES/META-INF/spring/applicationContext.xml
    Updated SRC_MAIN_RESOURCES/META-INF/persistence.xml
    ~.web roo> 

By default Hibernate will recreate the database schema every time the application is started.  This is not the behavior we want when running in production.  To fix this edit the `src/main/resources/META-INF/persistence.xml` file and change:

    <property name="hibernate.hbm2ddl.auto" value="create"/>

To:

    <property name="hibernate.hbm2ddl.auto" value="update"/>


## Add the Heroku add-on

The Spring Roo Heroku plugin automatically configures your application for Heroku deployment. This consists of the following steps:

1. Tell the Maven build to copy the jetty-runner app server into target so Heroku can use it to run your application
1. Modify the Spring context configuration to use the Heroku `DATABASE_URL` environment variable to set up the database connection.
1. Add a Procfile containing the command used to start your application on Heroku

To install the plugin run the following from the Roo shell:

    ~.web roo> addon install bundle --bundleSymbolicName net.stsmedia.roo.addon.heroku
    Target resource(s):
    -------------------
       addon-heroku (0.1.4.RELEASE)
    
    Deploying...done.
    
    Successfully installed add-on: addon-heroku [version: 0.1.4.RELEASE]
    [Hint] Please consider rating this add-on with the following command:
    [Hint] addon feedback bundle --bundleSymbolicName net.stsmedia.roo.addon.heroku --rating ... --comment "..."
    ~.web roo> 

## Run Heroku Setup

Now execute the setup command to prepare your application for Heroku:

    ~.web roo> heroku setup
    Your project as been configured for Heroku deployment. 
    Please also add the following environment variables to your system:
    export DATABASE_URL=postgres://stewie:rupert@localhost/heroku
    
    
    Updated SPRING_CONFIG_ROOT/applicationContext.xml
    Updated ROOT/pom.xml [added plugin org.apache.maven.plugins:maven-dependency-plugin:2.3]
    Created ROOT/Procfile
    ~.web roo> 

## Test it locally

Exit the Roo shell:

    ~.web roo> exit
    $ 

Set an environment variable to point to your local Postgres database (subsituting the username, password, and database name with the appropriate values:

* On Linux/Mac

        :::term
        $ export DATABASE_URL=postgres://username:password@localhost/database

* On Windows

        :::term
        $ set DATABASE_URL=postgres://username:password@localhost/database


Build the app with Maven:

    :::term
    $ mvn package
    [INFO] Scanning for projects...
    [INFO]                                                                         
    [INFO] ------------------------------------------------------------------------
    [INFO] Building petclinic 0.1.0.BUILD-SNAPSHOT
    [INFO] ------------------------------------------------------------------------
    [INFO] 
    [INFO] --- aspectj-maven-plugin:1.2:compile (default) @ petclinic ---
    ...
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time: 34.940s
    [INFO] Finished at: Mon Oct 24 15:43:40 PDT 2011
    [INFO] Final Memory: 9M/81M
    [INFO] ------------------------------------------------------------------------
    $

Run the app with foreman:

    :::term
    $ foreman start
    15:44:36 web.1     | started with pid 70508
    15:44:36 web.1     | 2011-10-24 15:44:36.835:INFO:omjr.Runner:Runner
    15:44:36 web.1     | 2011-10-24 15:44:36.836:WARN:omjr.Runner:No tx manager found
    15:44:36 web.1     | 2011-10-24 15:44:36.872:INFO:omjr.Runner:Deploying file:/Users/scott/spring-roo/target/petclinic-0.1.0.BUILD-SNAPSHOT.war @ /
    ...
    15:44:44 web.1     | 2011-10-24 15:44:44,058 [main] INFO  org.springframework.web.servlet.DispatcherServlet - FrameworkServlet 'petclinic': initialization completed in 724 ms
    15:44:44 web.1     | 2011-10-24 15:44:44.173:INFO:oejs.AbstractConnector:Started SelectChannelConnector@0.0.0.0:5000 STARTING

Test that it works by going to <http://localhost:5000> in your browser.

## Check into Git

To prevent generated files from getting checked into Git, create a `.gitignore` file with this single line:

    target

Now create a git repo and check in your code:

    :::term
    $ git init
    $ git add .
    $ git commit -m init

## Deploy to Heroku

Create the app on the Cedar stack:

    :::term
    $ heroku create -s cedar
    Creating simple-spring-6687... done, stack is cedar
    http://simple-spring-6687.herokuapp.com/ | git@heroku.com:simple-spring-6687.git
    Git remote heroku added

Deploy your code:

    :::term
    $ git push heroku master
    Counting objects: 211, done.
    Delta compression using up to 4 threads.
    Compressing objects: 100% (199/199), done.
    Writing objects: 100% (211/211), 104.02 KiB, done.
    Total 211 (delta 97), reused 0 (delta 0)

    -----> Heroku receiving push
    -----> Java app detected
    -----> Installing Maven 3.0.3..... done
    -----> Installing settings.xml..... done
    -----> executing /app/tmp/repo.git/.cache/.maven/bin/mvn -B -Duser.home=/tmp/build_6yltu8vyj3xl -Dmaven.repo.local=/app/tmp/repo.git/.cache/.m2/repository -s /app/tmp/repo.git/.cache/.m2/settings.xml -DskipTests=true clean install
           [INFO] Scanning for projects...
           [INFO]                                                                         
           [INFO] ------------------------------------------------------------------------
           [INFO] Building petclinic 0.1.0.BUILD-SNAPSHOT
           [INFO] ------------------------------------------------------------------------
           Downloading: http://maven.springframework.org/release/org/codehaus/mojo/aspectj-maven-plugin/1.2/aspectj-maven-plugin-1.2.pom
           ...
           [INFO] BUILD SUCCESS
           [INFO] ------------------------------------------------------------------------
           [INFO] Total time: 0:35.913s
           [INFO] Finished at: Mon Oct 24 22:55:55 UTC 2011
           [INFO] Final Memory: 11M/457M
           [INFO] ------------------------------------------------------------------------
    -----> Discovering process types
           Procfile declares types -> web
    -----> Compiled slug size is 39.3MB
    -----> Launching... done, v3
           http://simple-spring-6687.herokuapp.com deployed to Heroku
           
Congratulations! Your web app should now be up and running on Heroku. Open it in your browser with:

    :::term  
    $ heroku open
