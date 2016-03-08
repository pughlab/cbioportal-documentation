---
layout: default
---

### [Import Guide](import-guide.html)

# The import runner

Unlike the upstream cBioPortal, UHN uses a simple Java-based runner that allows an entire import process to be run in a single transaction. This means that any errors in the import process leave the application in a consistent state. This reduces the burden on those who import data, as they don't need to worry about repairing data if they import data that is broken.

This is a departure from the public site, that uses a large set of Python wrapper scripts. Because each uses a separate database connection, these are not transactional, so a break in one leaves the database at risk of inconsistency. Since most of these Python only invoke a `main` method in a given Java class, the import runner generates a single script that runs all these `main` methods within a single transaction. 

## Using the import runner

The cBioPortal at UHN import runner is a simple Java application, which runs from the command line as follows:

```
java -cp scripts-1.0-SNAPSHOT.jar org.mskcc.cbio.portal.ImportWrapper <dataDir> <dao.xml>
```

The `dataDir` points to a directory of files in standard cBioPortal import format, i.e., there should be a `meta-study.txt` file, and all its friends. `dataDir` will normally be different for each study's data.

The `dao.xml` is a fixed Spring context file, which is mainly used to point to the right database and provide any other generic settings, like the URL of the server. `dao.xml` should normally the same for all imports into the same server instance.

For example, the command:

```
java -cp scripts-1.0-SNAPSHOT.jar org.mskcc.cbio.portal.ImportWrapper /tmp/study dao.xml
```

results in output that'll look something like this:

```
Shut Down Hook Attached.
Reading data files to be imported...
Script file created at /tmp/import-script3138823484168248197.xml
Import script created successfully: /tmp/import-script3138823484168248197.xml
Loaded the following cancer study:
ID:  51
Name:  STUDY
Description:  STUDY
Done.
Reading data from:  /tmp/study/data_clinical.txt
 --> total number of lines:  7
6 records inserted into clinical_sample
Total number of patient specific clinical attributes added:  0
Total number of sample specific clinical attributes added:  6
Success!
Reading data from:  /tmp/study/data_mutations_extended.txt
 --> total number of lines:  4367
C12ORF9	93669 in the cbio cancer gene list is not a HUGO gene symbol.
Extracting OMA Scores from Column Number:  -1
..........
Percentage Complete:  23%
Mem Allocated:  1,963 MB, Mem used:  380.626 MB, Mem free:  1,582.374 MB
..........
Percentage Complete:  46%
Mem Allocated:  1,963 MB, Mem used:  452.449 MB, Mem free:  1,510.551 MB
....Ambiguous alias LST3: corresponding entrez ids of 28234,338821
Ambiguous alias LST3: corresponding entrez ids of 28234,338821
......
Percentage Complete:  69%
Mem Allocated:  1,963 MB, Mem used:  513 MB, Mem free:  1,450 MB
..........
Percentage Complete:  92%
Mem Allocated:  1,963 MB, Mem used:  194.681 MB, Mem free:  1,768.319 MB
...1252 records inserted into mutation
1077 records inserted into mutation_event
Mutation filter decisions: 4226
Rejects: 2974
Mutation Status 'None' Rejects:  0
Silent or Intron Rejects:  1735
UTR Rejects:  1238
IGR Rejects:  1
LOH or Wild Type Rejects:  0
Empty Annotation Rejects:  0
Missense Germline Rejects:  0
Success!
Read data from:  /tmp/study/case_lists/cases_targeted.txt
 --> stable ID:  study_targeted
 --> patient list name:  Targeted
 --> number of patients:  3
Read data from:  /tmp/study/case_lists/cases_all.txt
 --> stable ID:  study_all
 --> patient list name:  All
 --> number of patients:  6
Read data from:  /tmp/study/case_lists/cases_exome.txt
 --> stable ID:  study_exome
 --> patient list name:  Exome
 --> number of patients:  3
Sending 'POST' request to URL : http://clintracker.uhnresearch.ca:8089/reload_server_cache.do
Response Code : 200
```

You'll likely need to restart the server for the study to show up (this is a caching issue that we are working on).

## The Spring context file for importing

As we mentioned, the import runner uses a fixed Spring context file to set the database settings. It looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xmlns:tx="http://www.springframework.org/schema/tx"
     xmlns:context="http://www.springframework.org/schema/context"
     xsi:schemaLocation="
     http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
     http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
     http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">

	<bean class="org.springframework.beans.factory.config.MethodInvokingFactoryBean">
	    <property name="staticMethod" value="org.mskcc.cbio.portal.dao.JdbcUtil.setDataSource"/>
	    <property name="arguments">
	        <list>
	            <ref bean="businessDataSource"/>
	        </list>
	   </property>
	</bean>

	<bean id="businessDataSource" class="org.springframework.jdbc.datasource.TransactionAwareDataSourceProxy">
         <constructor-arg ref="dbcpDataSource"/>
    </bean>

	<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    	<property name="dataSource" ref="businessDataSource"/>
	</bean>

	<bean id="scriptTransactionTemplate" class="org.springframework.transaction.support.TransactionTemplate">
		<property name="transactionManager" ref="transactionManager" />
	    <property name="isolationLevelName" value="ISOLATION_DEFAULT"/>
	</bean>

	<bean id="dbcpDataSource" destroy-method="close"
		class="org.apache.commons.dbcp.BasicDataSource">
		<property name="driverClassName" value="com.mysql.jdbc.Driver" />
		<property name="url" value="jdbc:mysql://localhost:3306/xxx?sessionVariables=sql_mode=ansi" />
		<property name="username" value="xxx" />
		<property name="password" value="xxx" />
		<property name="minIdle" value="0" />
		<property name="maxIdle" value="10" />
		<property name="maxActive" value="100" />
		<property name="poolPreparedStatements" value="true" />
	</bean>

	<bean id="url" class="org.mskcc.cbio.portal.ImportWrapper">
    		<property name="url" value="http://cbioportal.example.com/reload_server_cache.do" />
	</bean>

</beans>
```
