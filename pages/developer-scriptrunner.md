---
layout: default
---

### [Developer Guide](developer-guide.html)

# TransactionalScriptRunner

This class, and its associated component, provides a transactional wrapping framework that makes importing a little bit easier. It consists of the following:

 * A new sub-project and associated jar component, scripts
 * A wrapping class, `TransactionalScriptRunner`, which is the main class for the jar and can be run from the command line
 * Some fairly funky use of Spring configuration files

## A brief example

A typical command is as follows:

```
java -jar scripts-1.0-SNAPSHOT.jar demoScript.xml applicationContext-dao.xml
```

Where `demoScript.xml` looks like this:

```
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xsi:schemaLocation="http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <bean class="org.mskcc.cbio.portal.util.TransactionalScripts">
        <constructor-arg>
            <list>
                <!-- Script 1 - load a study -->
                <list>
                    <value>org.mskcc.cbio.portal.scripts.ImportCancerStudy</value>
                    <value>/data/directory/cancer_study.txt</value>
                </list>

                <!-- Script 2 - load clinical data - needed to set up patients and samples -->
                <list>
                    <value>org.mskcc.cbio.portal.scripts.ImportClinicalData</value>
                    <value>/data/directory/test-classes/clinical_data.txt</value>
                    <value>test_brca</value>
                </list>
            </list>
        </constructor-arg>
    </bean>
</beans>
```

The actual concept is a very simple one: this config file contains a list of lists. Each list is, essentially, a command line to be run. The first element is a class name. The remaining elements are used to create a string array and are passed to a static `main(String[] args)` method in that class.

While theory you could do all this with shell scripting, what you can't do is provide the transactional context to
the whole action that is set up by the script runner. That depends on the second context file: `applicationContext-dao.xml` which provides a Spring transactional data source, and injects it into the portal core data access layer.

## Writing an importer

So to write an importer, all you really need to do, is build an XML configuration file, and put it on a command line to `java -jar scripts-1.0-SNAPSHOT.jar`, along with the `applicationContext-dao.xml`, and `TransactionalScriptRunner` should take care of the rest. If anything files anywhere, you ought to get an error message and a rollback.

And as soon as you have a working static main method, you can add a new list to a script like the `demoScript.xml`, which specifies its class and any command-line arguments it expects. Then, when the entire script file is run through `TransactionalScriptRunner`, it'll be handled as part of an import-style transaction.
Always assuming, of course, that your database is transactional (i.e., in MySQL uses InnoDB and not MyISAM).

## However...

Not all the importer classes (all of which have been moved into the scripts component in the package `org.mskcc.cbio.portal.scripts`, have a static `main` method. So you can't currently use these. This probably means you need to add a method, or add a new scripting class that instantiates something and calls methods on it. In any event, most of the time this might mean parsing command line arguments to identify a study or genetic profile identifier key (from a stable identifier string) and using that to initiate the script. Anyway, this remains the task.

## An important note on samples and patients

At some recent-ish stage, cBioPortal switched from using patient identifiers to using linked patient and sample identifiers. Where, previously, code could just load a bunch of data associated to a patient, you can't any more, because **first** you need to specify which are the patients and which are the samples. This is done, at least minimally, by the `ImportClinicalData` script (used in the `demoScript.xml` example above). The columns in this are fairly flexible however, except that both `PATIENT` and `SAMPLE` are treated specially. I've created a `minimal_clinical_data.txt` sample file which shows how just patients and sample identifier records can be added alone. This is probably the required step before, for example, loading mutation data keyed by sample (and it should be keyed by sample).  
