Download Link: https://assignmentchef.com/product/solved-cs2030s-lab-8-bus
<br>
<h3 id="bus"></h3>

<h4 id="topic-coverage">Topic Coverage</h4>

<ul>

 <li>Asynchronous programming</li>

 <li>CompletableFuture</li>

 <li>Lambda functions</li>

</ul>

<h4 id="problem-description">Problem Description</h4>

We have a Web API online for querying bus services and bus stops in Singapore. You can go ahead and try:

<ul>

 <li><a class="uri" href="https://cs2030-bus-api.herokuapp.com/bus_services/96">https://cs2030-bus-api.herokuapp.com/bus_services/96</a> returns the list of bus stops (id followed by description) served by Bus 96.</li>

 <li><a class="uri" href="https://cs2030-bus-api.herokuapp.com/bus_stops/16189">https://cs2030-bus-api.herokuapp.com/bus_stops/16189</a> returns the description of the stop followed by a list of bus services that serve the stop.</li>

</ul>

(note: our database is two years old though — don’t rely on this for your daily commute!)

In this lab, we will write a program that uses the Web API to do the following:

Given the current stop <code>S</code>, and a search string <code>Q</code>, returns the list of buses serving <code>S</code> that also serves any stop with description containing <code>Q</code>. For instance, given <code>16189</code> and <code>Clementi</code>, the program will output

<pre><code>Search for: 16189 &lt;-&gt; Clementi:From 16189- Can take 96 to:  - 17171 Clementi Stn  - 17091 Aft Clementi Ave 1  - 17009 Clementi Int- Can take 151 to:  - 17091 Aft Clementi Ave 1- Can take 151e to:  - 17091 Aft Clementi Ave 1Took 11,084ms</code></pre>

The pairs of <code>S</code> and <code>Q</code> are entered through the standard input with every pair of <code>S</code> and <code>Q</code> in a separate line.

<pre><code>08031 Orchard17009 NUS17009 MRT15131 Stn08031 Int</code></pre>

A working program has been written and is available <a href="https://codecrunch.comp.nus.edu.sg/taskfile.php/4883/bus_skeleton.zip">here</a>. Download and study the program to understand what it does and how it works. Keep a copy of program around for comparison and reference later.

The given program is written synchronously. Every query to the Web API is done one-by-one, and the program has to wait until one query completes before it can continue execution of the program. As a result, the program is slower than it should be.

<h4 id="the-task">The Task</h4>

Your task, for this lab, is to change the given program so that it executes asynchronously. Doing so can significantly speed up the program.

The root of synchronous Web API access can be found in the method <code>httpGet</code> in <code>BusAPI.java</code>, in which the invocation of method <a href="https://docs.oracle.com/en/java/javase/11/docs/api/java.net.http/java/net/http/HttpClient.html#send(java.net.http.HttpRequest,java.net.http.HttpResponse.BodyHandler)"><code>send</code></a> from the class <a href="https://docs.oracle.com/en/java/javase/11/docs/api/java.net.http/java/net/http/HttpClient.html"><code>HttpClient</code></a> is done synchronously (i.e., it blocks until the response returns).<code>HttpClient</code> also provides an asynchronous version of <code>send</code> called <a href="https://docs.oracle.com/en/java/javase/11/docs/api/java.net.http/java/net/http/HttpClient.html#sendAsync(java.net.http.HttpRequest,java.net.http.HttpResponse.BodyHandler)"><code>sendAsync</code></a>, which is exactly the same as <code>send</code> except that it is asynchronous and returns a <code>CompletableFuture&lt;HttpResponse&lt;T&gt;&gt;</code> instead of <code>HttpResponse&lt;T&gt;</code>. (You do not need to get into the nitty-gritty details of the <code>HttpClient</code> and <code>HttpResponse</code> for this lab — but they are still good to know, so read up about them at your leisure).

To make the program synchronous, you should first change the invocation of <code>send</code> in <code>BusAPI</code> to <code>sendAsync</code>. All other changes will be triggered by this.

It is important to note that your should not call <code>CompletableFuture::join</code> prematurely, so that everything that has been <strong>promised</strong> so far, from the lower-level Web API calls to the higher-level logic of searching for bus services, are done asynchronously. The only place where you should <code>join</code> is in the <code>main()</code> method after waiting for all the <code>CompletableFuture</code> objects to complete using <code>allOf</code>. The final step would then be to print out the description of the bus routes found.

The speed-up you would experience for the asynchronous version depends on the complexity of the inputs. For the following test input:

<pre><code>08031 Orchard17009 NUS17009 MRT15131 Stn08031 Int12345 Dummy</code></pre>

a typical reduction in the time from around 150-180 seconds to 10-15 seconds (i.e. more than 10 times speed up) is expected. Your milage may vary, but you should see some speed up in the total query time.

This task is divided into several levels. Read through all the levels to see how the different levels are related.

Remember to: * always compile your program files first before either using <code>jshell</code> to test your program, or using <code>java</code> to run your program * run <code>checkstyle</code> on your code

For this lab, you are also required to write <code>javadoc</code> comments and make sure you can generate the <code>javadoc</code> HTML files. You are not required to submit the <code>javadoc</code> HTML files, but they will be generated later based on your code.

<table border="1" cellpadding="10">

 <tbody>

  <tr>

   <td><h4 id="level-1-busapi.java">Level 1: <code>BusAPI.java</code></h4>Start by changing the method call <code>HttpClient::send</code> to <code>HttpClient::sendAsync</code> in the method <code>httpGet</code>. In the skeleton code provided, notice that as soon as the <code>send</code> method returns with a <code>HttpResponse</code>, it proceeds to check the status. Since the <code>send</code> could take a long time to complete, we would have to wait for it to complete first.On the other hand, using <code>sendAsync</code> returns a response wrapped within a <code>CompletableFuture</code>; it is a <strong>promise</strong> that a response will be returned (not now, but eventually). The part of the code that deals with the response can now be attached as a callback to this <code>CompletableFuture</code>.Modify the <code>BusAPI.java</code> program such that sending a <code>http</code> request is now done asynchronously. You will also need to modify the <code>getBusStopsServedBy</code> and <code>getBusServicesAt</code> methods accordingly.Compile your java program as follows:<pre><code>$ javac -Xlint:rawtypes BusAPI.java</code></pre>Once compiled, you can update the given <a href="https://codecrunch.comp.nus.edu.sg/taskfile.php/4883/level1.jar">jar</a> file and test your program.<pre><code>$ jar uf level1.jar *.class$ echo "16189 Clementi" | java -jar level1.jarSearch for: 16189  &lt;-&gt; Clementi:From 16189- Can take 96 to:  - 17171 Clementi Stn  - 17091 Aft Clementi Ave 1  - 17009 Clementi Int- Can take 151 to:  - 17091 Aft Clementi Ave 1- Can take 151e to:  - 17091 Aft Clementi Ave 1Took 3,131ms</code></pre>You may now proceed to submit <code>BusAPI.java</code> and test if this level passes. CodeCrunch will provide the asynchronous versions of the other classes.Check your styling by issuing the following<pre><code>$ checkstyle *.java</code></pre></td>

  </tr>

  <tr>

   <td><h4 id="level-2-busservice.java">Level 2: <code>BusService.java</code></h4>Now compile the <code>BusService.java</code> class:<pre><code>$ javac -Xlint:rawtypes BusService.java</code></pre>and take note of the compilation errors. Specifically, the <code>getBusStops</code> method should now return a <strong>promise</strong> of a <code>Set&lt;BusStop&gt;</code>.In this case, as soon as <code>BusAPI::getBusStopsServedBy</code> method completes, a <code>Scanner object</code> can then be created to proceed with reading user input.Likewise, modify the <code>findStopsWith</code> method accordingly.Compile your java program as follows:<pre><code>$ javac -Xlint:rawtypes BusService.java</code></pre>Once compiled, you can update the given <a href="https://codecrunch.comp.nus.edu.sg/taskfile.php/4883/level2.jar">jar</a> file and test your program.<pre><code>$ jar uf level2.jar *.class$ echo "16189 Clementi" | java -jar level2.jarSearch for: 16189  &lt;-&gt; Clementi:From 16189- Can take 96 to:  - 17171 Clementi Stn  - 17091 Aft Clementi Ave 1  - 17009 Clementi Int- Can take 151 to:  - 17091 Aft Clementi Ave 1- Can take 151e to:  - 17091 Aft Clementi Ave 1Took 3,131ms</code></pre>You may now proceed to submit <code>BusAPI.java</code> and <code>BusService.java</code>, in order to test if this level passes. CodeCrunch will provide the asynchronous versions of the other classes.Check your styling by issuing the following<pre><code>$ checkstyle *.java</code></pre></td>

  </tr>

  <tr>

   <td><h4 id="level-3-bussg.java-and-busroute.java">Level 3: <code>BusSg.java</code> and <code>BusRoutes.java</code></h4>Within the <code>BusSg</code> class, modify the <code>getBusServices</code> method to return a <strong>promise</strong> of a <code>Set&lt;BusServices&gt;</code>. In addition, the <code>findBusServicesBetween</code> method needs to be modified accordingly. This will trigger a change in the constructor of the <code>BusRoutes</code> class.Within the <code>BusRoutes</code> class, the <code>Map</code> of bus services should now map a bus service to a <strong>promise</strong> of a <code>Set&lt;BusStop&gt;</code>. When generating the description of the bus route, the original method goes through the bus services with the route, and for each bus service, it obtains a set of matching bus-stops and calls <code>decribeService</code> to return a <code>String</code> representing the bus service and its matching stops. With the new implementation, the set of matching bus-stops is now a <strong>promise</strong>. You will need to be able to combine these <strong>promises</strong> together, and return the string representation of the bus route as a <strong>promise</strong>.Compile your java program as follows:<pre><code>$ javac -Xlint:rawtypes BusSg.java</code></pre>Once compiled, you can update the given <a href="https://codecrunch.comp.nus.edu.sg/taskfile.php/4883/level3.jar">jar</a> file and test your program.<pre><code>$ jar uf level3.jar *.class$ echo "16189 Clementi" | java -jar level3.jarSearch for: 16189  &lt;-&gt; Clementi:From 16189- Can take 96 to:  - 17171 Clementi Stn  - 17091 Aft Clementi Ave 1  - 17009 Clementi Int- Can take 151 to:  - 17091 Aft Clementi Ave 1- Can take 151e to:  - 17091 Aft Clementi Ave 1Took 3,131ms</code></pre>You may now proceed to submit <code>BusAPI.java</code>, <code>BusService.java</code>, <code>BusRoutes.java</code> and <code>BusSg.java</code>, in order to test if this level passes.Check your styling by issuing the following<pre><code>$ checkstyle *.java</code></pre></td>

  </tr>

  <tr>

   <td><h4 id="level-4-main.java">Level 4: <code>Main.java</code></h4>In the <code>Main</code> class, you will now need to construct all the <strong>promises</strong> as <code>CompletableFuture</code> objects and wait for all of them to complete. Finally, output the description of the bus routes.A sample run of your java program as follows:<pre><code>$ javac -Xlint:rawtypes Main.java$ echo "16189 Clementi" | java MainSearch for: 16189  &lt;-&gt; Clementi:From 16189- Can take 96 to:  - 17171 Clementi Stn  - 17091 Aft Clementi Ave 1  - 17009 Clementi Int- Can take 151 to:  - 17091 Aft Clementi Ave 1- Can take 151e to:  - 17091 Aft Clementi Ave 1Took 3,131ms$ cat test.in16189 Clementi 17009 NUS$ cat test.in | java MainSearch for: 16189  &lt;-&gt; Clementi:From 16189- Can take 96 to:  - 17171 Clementi Stn  - 17091 Aft Clementi Ave 1  - 17009 Clementi Int- Can take 151 to:  - 17091 Aft Clementi Ave 1- Can take 151e to:  - 17091 Aft Clementi Ave 1Search for: 17009  &lt;-&gt; NUS:From 17009- Can take 196 to:  - 17191 NUS High Sch- Can take 156 to:  - 41011 NUS Bt Timah Campus- Can take 96 to:  - 16169 NUS Raffles Hall  - 16199 NUS Fac Of Design &amp; Env  - 16149 NUS Fac Of Architecture  - 16159 NUS Fac Of EngrgTook 3,924ms</code></pre>You may proceed to submit your java files.Check your styling by issuing the following<pre><code>$ checkstyle *.java</code></pre></td>

  </tr>

 </tbody>

</table>