# cics-java-liberty-springboot-emp-jpa

This project demonstrates a Spring Boot application, which uses Spring Data JPA, integrated with IBM CICS that can be deployed to a CICS Liberty JVM server. The application makes use of the employee sample table supplied with Db2 for z/OS. The application allows you to add, update, delete or display employee information from the table EMP.

## Prerequisites

    CICS TS V5.3 or later
    A configured Liberty JVM server
    Java SE 1.8 or later on the z/OS system
    Java SE 1.8 or later on the workstation
    Either Gradle or Apache Maven on the workstation
    IBM Db2 V11 or later on z/OS
 
## Building

You can choose to build the project using Gradle or Maven. The project includes both Gradle and Maven wrappers, these wrappers will automatically download required components from your chosen build tool; if not already present on your workstation.

You can also build the sample project through plug-in tooling of your chosen IDE. Both Gradle buildship and Maven m2e will integrate with Eclipse's "Run As..." capability allowing you to specify the required build-tasks. There are typically clean bootWar for Gradle and clean package for Maven, as reflected in the command line approach shown later.

Note: When building a WAR file for deployment to Liberty it is good practice to exclude Tomcat from the final runtime artifact. We demonstrate this in the pom.xml with the provided scope, and in build.gradle with the providedRuntime() dependency.

Note: If you import the project to an IDE of your choice, you might experience local project compile errors. To resolve these errors you should refresh your IDEs configuration.   
For example, if you are using Eclipse:
* for Gradle, right-click on "Project", select "Gradle -> Refresh Gradle Project", 
* for Maven, right-click on "Project", select "Maven -> Update Project...".

### Gradle

#### Run the following in a local command prompt:

On Linux or Mac:

./gradlew clean bootWar

On Windows:

gradlew.bat clean bootWar

This creates a WAR file inside the build/libs directory.

### Maven

#### Run the following in a local command prompt:

On Linux or Mac:

./mvnw clean package

On Windows:

mvnw.cmd clean package

This creates a WAR file inside the target directory.

## Deploying

### update features in server.xml
Ensure you have the following features in server.xml:
* servlet-3.1 or servlet-4.0
* jsp-2.3
* springBoot-2.0
* jdbc-4.0

Note: servlet-4.0 will only work for CICS TS V5.5 or later. If you choose to use servlet-4.0 then you must specify `-Dcom.ibm.cics.jvmserver.wlp.wab=false` in your jvmprofile

### add a datasource definition to server.xml
Add a datasource definition to your server.xml. this sample uses a type 4 connection. The application connects to this datasource by using a @Bean datasource which connects using the jndiName value **jdbc/jdbcDataSource-bean**

E.g. as follows:

```
<dataSource id="t4" jndiName="jdbc/jdbcDataSource-bean" type="javax.sql.DataSource">
        <jdbcDriver>
            <library name="DB2LIB">
                <fileset dir="/usr/lpp/db2v11/jdbc/classes" includes="db2jcc4.jar db2jcc_license_cisuz.jar"/>
                <fileset dir="/usr/lpp/db2v11/jdbc/lib"/>
            </library>
        </jdbcDriver>
        <properties.db2.jcc currentSchema="DSN81110" databaseName="DSNV11P2" driverType="4" 
	     password="<your password>" portNumber="<your port number>" serverName="<your server name>" user="<your userid>"/>
</dataSource> 
```

### create CICS bundle
Copy and paste the WAR from your target or build/libs directory into a CICS bundle project and create a new WARbundlepart for that WAR file.

Deploy the CICS bundle project as normal. For example in Eclipse, select "Export Bundle Project to z/OS UNIX File System".

### create application definition in server.xml
**Alternatively**, manually upload the WAR file to zFS and add an <application> configuration element to server.xml.

For example:
```
   <application id="com.ibm.cicsdev.springboot.emp.jpa-0.1.0"  
     location="${server.config.dir}/springapps/com.ibm.cicsdev.springboot.emp.jpa-0.1.0.war"  
     name="com.ibm.cicsdev.springboot.jpa-0.1.0" type="war">
     <application-bnd>
        <security-role name="cicsAllAuthenticated">
            <special-subject type="ALL_AUTHENTICATED_USERS"/>
        </security-role>
     </application-bnd>  
   </application>
```
## application.properties file

this file contains the following entries 

```
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.DB2390Dialect
spring.jpa.show-sql=true
```

*spring-jpa.show-sql* is not required, but is useful to display the sql which the JPA function is using to satisy the requests being made.

*spring.jpa.properties.hibernate.dialect* is required to ensure that the application generates SQL which can be run by DB2 


## Trying out the sample

Find the base URL for the application in the Liberty messages.log 
    e.g. http://myzos.mycompany.com:httpPort/com.ibm.cicsdev.springboot.emp.jpa-0.1.0.

Paste the base URL along with the REST service suffix 'allRows' into the browser 
    e.g. http://myzos.mycompany.com:httpPort/com.ibm.cicsdev.springboot.emp.jpa-0.1.0/allRows

The browser will prompt for basic authentication. Enter a valid userid and password - according to the configured registry for your target Liberty JVM server.

All the rows in table EMP should be returned.

    
## Summary of all available interfaces     

http://myzos.mycompany.com:httpPort/com.ibm.cicsdev.springboot.emp.jpa-0.1.0/allRows
    
  >All rows in table EMP will be returned - the datasource is obtained from the application.properties file
    
http://myzos.mycompany.com:httpPort/com.ibm.cicsdev.springboot.emp.jpa-0.1.0/allRows2
  
  >All rows in table EMP will be returned - the datasource is obtained from an @Bean method
    
http://myzos.mycompany.com:httpPort/com.ibm.cicsdev.springboot.emp.jpa-0.1.0/addEmployee/{firstName}/{lastName}
  
  >A new employee record will be created using the first name and last name supplied. All other fields in
  the table will be set by the application to the same values by this demo application.
  If successful the employee number created will be returned.
    
http://myzos.mycompany.com:httpPort/com.ibm.cicsdev.springboot.emp.jpa-0.1.0/oneEmployee/{empno}
  
  >A single employee record will be displayed if it exists.
    
http://myzos.mycompany.com:httpPort/com.ibm.cicsdev.springboot.emp.jpa-0.1.0/updateEmployee/{empNo}/{newSalary}
  >The employee record will be updated with the salary amount specified.
    
http://myzos.mycompany.com:httpPort/com.ibm.cicsdev.springboot.emp.jpa-0.1.0/deleteEmployee/{empNo}
  
  >The employee record with the empNo specified will be deleted if it exists

### Notes:
{firstName} and {lastName} should be replaced by names of your choosing.
>>the definition of FIRSTNME in table EMP is VARCHAR(12)
>>the definition of LASTNAME in table EMP is VARCHAR(15)

{empno} would be replaced by a 6 character employee number. 
>>the definition of EMPNO in the EMP table is char(6)

{newSalary} should be replaced by a numeric amount 
>>the definition of SALARY in the EMP table is DECIMAL(9, 2)

License

This project is licensed under Apache License Version 2.0.
