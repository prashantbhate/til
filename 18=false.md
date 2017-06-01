
What a strange file name you are thinking..

Soon you will know why..

Ok, Here is how it all started

I was tasked to improve performance of our jBehave automated test tool

>Hmm, From where should I start ? 

First let me spun up my docker based dev environment

````
➜  devops git:(docker) ✗ ./gradlew --no-daemon up
Picked up _JAVA_OPTIONS:
:docker-db-box:dockerUp
Recreating r14box_db_1
:docker-dev-box:dockerUp
Recreating r14box_elk_1
Recreating r14box_simi_1
Recreating r14box_xxx_app_server_1
Recreating r14box_yyy_app_server_1
:wait_for_db
http://localhost:8888/apex failed -- Unexpected end of file from server -- re-try after 5 seconds
http://localhost:8888/apex failed -- Unexpected end of file from server -- re-try after 5 seconds
http://localhost:8888/apex failed -- Unexpected end of file from server -- re-try after 5 seconds
http://localhost:8888/apex failed -- Unexpected end of file from server -- re-try after 5 seconds
:wait_for_appserver_xxx
:wait_for_appserver_yyy
:wait_for_appserver_zzz
:glassfish-deployer:deploy_xxx_xxx-xxxx-jca
Application deployed with name xxx-xxxx-jca.
Command deploy executed successfully.
:glassfish-deployer:deploy_yyy_yyyyy-yyyy-jca
Application deployed with name zzzzz-zzzz-jca.
Command deploy executed successfully.
:glassfish-deployer:deploy_mmm_mmmm-ear
Application deployed with name mmmm-ear.
Command deploy executed successfully.
:glassfish-deployer:deploy_mmmm_lifecycle
Command create-lifecycle-module executed successfully.
:glassfish-deployer:deploy_qqqqqq_qqqqq
Application deployed with name qqqqqq.
Command deploy executed successfully.
:glassfish-deployer:deploy_simulator_zzz-zzz-simulator
:glassfish-deployer:deploy_simulator_simulator-web
Application deployed with name simulator-web.
Command deploy executed successfully.
:glassfish-deployer:deployAll
deployAll
:waitAndDeployAll
wait and deploy
:up
startup

BUILD SUCCESSFUL

Total time: 3 mins 59.284 secs
````
4 minutes appears too much but for the amount of things it does it is actually fast. I have masked few stuff to skip the details , but basically what above gradle script does is 

* spins up an oracle xe server which has pre built with required schema and data
* spins up 3 glassfish servers with pre configured domains
* wait for DB and 3 glassfish servers to come up and ready
* deploy  applications (couple of jca ear & war) and simulators

Once the applications are running,
Let me run the test suites to measure time it take to complete

```shell
➜  test-tool git:(master) ✗ jb '**/*.story' 
```
What's `jb` u ask ? Its just one of my zsh shortcut to cutshort my typing

```
➜  test-tool git:(master) ✗ whence -f jb
jb () {
	mvn clean install -e -Dspring.jdbc.getParameterType.ignore=true -PdefaultModules -DrunJBehave -PenvDOCKER "-Dlogback.configurationFile=file:///$CODE_BASE/config/logback.xml" "-Dlogback.log.file.location=$CODE_BASE/logs/logback.log" "-Dstory.include=$1"
}
```
and I kept it running..
FOR A WHILE actually..
After the lunch hour and a meeting Later, here is what I could see


```
...lots of maven junk...
...lots of maven junk...
...lots of maven junk...
...
[INFO] integration-tests ................................. SUCCESS [1.32.195s]
[INFO] ------------------------------------------------------------------------
...lots of maven junk...
...lots of maven junk...
```

Ouch, That's too much time

A profiling this through jvisualvm should help to identify any bottlenecks 

> ➜  test-tool git:(master) ✗ jvisualvm

Now I have 2 options to profile Sampler vs Profiler. A sampler basically tracks All stacktraces at fixed intervals but will be very fast. However Profiler goes one level deeper and dynamically adds profiling code to loaded classes so it will be more accurate but slow.

Let me run it with Sampler, as I would get result very fast first I will leave default settings for now.

```shell
➜  test-tool git:(master) ✗ jb '**/*.story' 
```
and then open up jvisualvm and attach to running application and Click on "CPU" , yes for now my aim is to see whats taking too much time, I will look at memory allocations may be later. For now I am giving more than enough memory so that application dont cry for memory. I will keep an eye on garbage collection though...

>export MAVEN_OPTS="-Xms512m -Xmx2g -XX:MaxPermSize=1g "

Results did not surprise me
From result I could identify top 3 culprits

* DB calls that were used by the tests to data setup and assert data in the tables
* REST APIs that were core of the tests (Improving this is out of control as it is external to test-tool )
* JBehave step matchers

Ok lets start one by one

I see lot of database queries being executed but its hard to go scan through the code to identify, If only there is a way to print SQLs when they actually run would be useful.

Oracle JDBC provides (a very less known) jdbc driver with the debugging log. Its very clunky and bit difficult to configure, but hey, how hard could it be?

Let me download debug version of the ojdbc jar replace my current jdbc driver with a debug version in my local repository

>mvn install:install-file -Dfile=ojdbc6_g.jar -DgroupId=oracle -DartifactId=ojdbc -Dversion=11.2.0.4 -Dpackaging=jar

Enable oracle tracing with below sytem property

>-Doracle.jdbc.Trace=true

Let me update logback.xml with 

><logger name="oracle" level="INFO"/>

Run the test suite with single story

```shell
➜  test-tool git:(master) ✗ jb '**/a-xxx-xx-masked.story' 
```

I cant see any logs ? what could be wrong . Oracle I know uses java util logging (JUL)  I have configured jul-to-slf4j-xxx.jar bridge..

Oh I know, there is a SLF4JBridgeHandler that needs to be configured to be used to route all jul logging to slf4j , as per documentation, I need to add below snippet in a static block. I know this way of log forwarding is very inefficient to let me wrap it in a conditional.

````
static{
if (Boolean.getBoolean("enable.jul.slf4j.bridge")) {
			SLF4JBridgeHandler.removeHandlersForRootLogger(); //Remove all existing JUL handlers
			SLF4JBridgeHandler.install(); // Install 
			LogManager.getLogManager()
				.getLogger("")
				.setLevel(Level.ALL);
   }
}
````

Let me run it again to see If I can see some oracle logs

```shell
➜  test-tool git:(master) ✗ jb '**/a-xxx-xx-masked.story' 
```
What I see instead is below stack trace

```
....application specific stacktrace....
....application specific stacktrace....
....application specific stacktrace....
Caused by: java.lang.IllegalArgumentException: can't parse argument number: 18=false
	at java.text.MessageFormat.makeFormat(MessageFormat.java:1420) ~[na:1.7.0_80]
	at java.text.MessageFormat.applyPattern(MessageFormat.java:479) ~[na:1.7.0_80]
	at java.text.MessageFormat.<init>(MessageFormat.java:363) ~[na:1.7.0_80]
	at java.text.MessageFormat.format(MessageFormat.java:835) ~[na:1.7.0_80]
	at org.slf4j.bridge.SLF4JBridgeHandler.getMessageI18N(SLF4JBridgeHandler.java:264) ~[jul-to-slf4j-1.7.12.jar:1.7.12]
	at org.slf4j.bridge.SLF4JBridgeHandler.callLocationAwareLogger(SLF4JBridgeHandler.java:220) ~[jul-to-slf4j-1.7.12.jar:1.7.12]
	at org.slf4j.bridge.SLF4JBridgeHandler.publish(SLF4JBridgeHandler.java:297) ~[jul-to-slf4j-1.7.12.jar:1.7.12]
	at java.util.logging.Logger.log(Logger.java:616) ~[na:1.7.0_80]
	at java.util.logging.Logger.doLog(Logger.java:641) ~[na:1.7.0_80]
	at java.util.logging.Logger.log(Logger.java:685) ~[na:1.7.0_80]
	at oracle.net.ns.NSProtocol.establishConnection(NSProtocol.java:934) ~[ojdbc-11.2.0.4.jar:11.2.0.4.0]
	at oracle.net.ns.NSProtocol.connect(NSProtocol.java:271) ~[ojdbc-11.2.0.4.jar:11.2.0.4.0]
	at oracle.jdbc.driver.T4CConnection.connect(T4CConnection.java:1663) ~[ojdbc-11.2.0.4.jar:11.2.0.4.0]
	at oracle.jdbc.driver.T4CConnection.logon(T4CConnection.java:385) ~[ojdbc-11.2.0.4.jar:11.2.0.4.0]
	at oracle.jdbc.driver.PhysicalConnection.<init>(PhysicalConnection.java:564) ~[ojdbc-11.2.0.4.jar:11.2.0.4.0]
	at oracle.jdbc.driver.T4CConnection.<init>(T4CConnection.java:251) ~[ojdbc-11.2.0.4.jar:11.2.0.4.0]
	at oracle.jdbc.driver.T4CDriverExtension.getConnection(T4CDriverExtension.java:29) ~[ojdbc-11.2.0.4.jar:11.2.0.4.0]
	at oracle.jdbc.driver.OracleDriver.connect(OracleDriver.java:563) ~[ojdbc-11.2.0.4.jar:11.2.0.4.0]
	at org.apache.tomcat.jdbc.pool.PooledConnection.connectUsingDriver(PooledConnection.java:310) ~[tomcat-jdbc-8.5.15.jar:na]
	... 77 common frames omitted
Caused by: java.lang.NumberFormatException: For input string: "18=false"
	at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65) ~[na:1.7.0_80]
	at java.lang.Integer.parseInt(Integer.java:492) ~[na:1.7.0_80]
	at java.lang.Integer.parseInt(Integer.java:527) ~[na:1.7.0_80]
	at java.text.MessageFormat.makeFormat(MessageFormat.java:1418) ~[na:1.7.0_80]
	... 95 common frames omitted
```

I can see from stack trace SLF4JBridgeHandler is trying to something with Oracle Logged message.

Let me run the application in debug mode to catch this exception to see full me.

How to attach debugger to console application ? here is the trick

Create a new intellij Remote debug configuration.  There are 2 Debugger modes Attach vs Listen . I normally choose "Attach" option to connect to Running application server which is pre configured with debugger settings. For console run application I prefer  "Listen" mode , where IntelliJ would listen to incomming debug requests, so that I can use same shared setting and port for different console applications.

>export MAVEN_OPTS="-Xms512m -Xmx2g -XX:MaxPermSize=1g -agentlib:jdwp=transport=dt_socket,server=n,address=localhost:5005,suspend=n"

Before I start my testlet me add a Java Exception Breakpoint for 'java.lang.NumberFormatException' with below condition

>getMessage().contains("18=false")

so that it halts when above exception is thrown. Its a pretty neat trick as it will not stop when NumberFormatException is thrown for other reasons.

Let me trigger
```shell
➜  test-tool git:(master) ✗ jb '**/a-xxx-xx-masked.story' 
```
and wait for debugger to hit this point (I would disable all other breakpoints for now)


