# Enterprise Connector Proposal

Based on my java web application development experience, I had some idea about the Enterprise Connector(as known as extension sources). I hope it can be more convenient and flexible.

## summary
Basic idea is that:
 * Develope a java web application for Enterprise Connector, using spring boot, or VertX.
 * Make every extension sources as a plugin, let Enterprise Connector can recognize it, and display as a list in web page.
 * For extension sources we provided like jdbc, opc, package them as jar, and include them in the java web app.
 * For every extension sources, we can run an instance, using some configuration. And provide a form for the configuration, for that common setting.
 * Using H2 as default DB, saving instance above, and running configuration. Embedded H2 DB will make the Enterprise Connector server can be used out-of-box. It is important in POC environment. And later can change to use MySQL or else.
 * Make every extension sources can be developed separately. For now, even I just want to package opcua module, I still need to setup opencv and jdbc driver.
 


## detail
### Java Web App
Main reasons to use a web app server are, We can have multiple connectors, checking running status, configure in UI, and even check the running log(by storing error/warn log in DB).

We can use java reflection or other dynamic technic to discover Enterprise Connector plugins.

We can also run/stop the instance in UI, since every connector has a web socket client.

Using spring boot, we can just package the code as a runnable jar file. And run with default H2 DB. With provided several connector(JDBC, opcua, UDP etc...), we can create an instance, input some configuration, and run. It will be very useful for a quick demo and poc.

### configuration
For now, I find that the configuration is too complicated, even for a simple opcua connector, we took several hours to figure out how to configure and and to select from that source in Vantiq procedure.

It will be more convenient to make it simple, provide a UI to input common configuration, and an extra json object for specific of that connector.

If we want to make it better, we can use java annotation on the configuration class, to get those properties and create a dynamic input form in UI. 

### JDBC connector
For JDBC, for every different type of databases, we need to add its driver jar in classpath. So we can package some jars for mysql, oracle mssql in the package. But if customer has different db, still need to repackage. I am not sure whether there is some common JDBC driver, which is independent on spefici DB. 

Maybe another solution is, we can upload specific jar, and load the jar in java code.

### running and status
We can run multiple instance for a connector, like get several databases. And save error/warn log in DB, or using log appender to write logs in servlet response, to display them in UI.



