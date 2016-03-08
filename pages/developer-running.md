---
layout: default
---

### [Developer Guide](developer-guide.html)

# How to spin up a portal with Maven

## 1. Use a Maven settings.xml like this:

    <?xml version="1.0" encoding="UTF-8"?>
    <settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                              http://maven.apache.org/xsd/settings-1.0.0.xsd">
      <servers>
        <server>
          <id>cbioportal_test</id>
          <username>xxxx</username>
          <password>xxxx</password>
        </server>
      </servers>

      <profiles>
        <profile>
        <id>env-test</id>
          <properties>
            <db.test.serverkey>settingsKey</db.test.serverkey>
            <db.test.driver>com.mysql.jdbc.Driver</db.test.driver>
            <db.test.url>jdbc:mysql://xxxx:3306/xxxx?sessionVariables=sql_mode=ansi</db.test.url>
            <db.test.username>xxxx</db.test.username>
            <db.test.password>xxxx</db.test.password>
          </properties>
        </profile>
      </profiles>

      <activeProfiles>
        <activeProfile>env-test</activeProfile>
      </activeProfiles>
    </settings>

But use your database server server host and credentials instead of the `xxxx`s.

## 2. Check out and run the portal

    git clone https://github.com/pughlab/cbioportal.git
    cd cbioportal
    cp src/main/resources/portal.properties.EXAMPLE portal.properties
    cp src/main/resources/log4j.properties.CONSOLE log4j.properties
    mvn verify -P test-portal


# How to spin up a portal in Eclipse

First do everything above, checking out the server and setting up the Maven settings, as Maven writes these into a few files in a way that Eclipse doesn't. Then:

## 3. From a command line

    mvn process-test-resources eclipse:eclipse

## 4. Within Eclipse

* Select `server/src/main/java/org/mskcc/cbio/server/httpd/CBioPortalJettyServer`
* Run as application...

The following settings are needed too. You can set these under `Run Configurations...`

* In Program Arguments, set: `-Dorg.eclipse.jetty.webapp.LEVEL=DEBUG`
* In Environment, add a variable `PORTAL_HOME` with the value: `${git_dir}/..`

Now when you run the application, it should spin up a complete portal, with the mini data set, on the URL: `http://localhost:8080`
