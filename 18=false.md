
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
	mvn clean install -e -Dspring.jdbc.getParameterType.ignore=true -Doracle.jdbc.Trace=true -PdefaultModules -DrunJBehave -PenvDOCKER "-Dlogback.configurationFile=file:///$CODE_BASE/config/logback.xml" "-Dlogback.log.file.location=$CODE_BASE/logs/logback.log" "-Dstory.include=$1"
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
