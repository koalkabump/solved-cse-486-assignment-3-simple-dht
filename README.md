Download Link: https://assignmentchef.com/product/solved-cse-486-assignment-3-simple-dht
<br>
In this assignment, you will design a simple DHT based on Chord. Although the design is based on Chord, it is a simplified version of Chord; you do not need to implement finger tables and finger-based routing; you also do not need to handle node leaves/failures.Therefore, there are three things you need to implement: 1) ID space partitioning/re-partitioning, 2) Ring-based routing, and 3) Node joins.

Just like the previous assignment, your app should have an activity and a content provider. However, the main activity should be used for testing only and should not implement any DHT functionality. The content provider should implement all DHT functionalities and support insert and query operations. Thus, if you run multiple instances of your app, all content provider instances should form a Chord ring and serve insert/query requests in a distributed fashion according to the Chord protocol.

<h1>References</h1>

Before we discuss the requirements of this assignment, here are two references for the Chord design:

<ol>

 <li><u><a href="http://www.cse.buffalo.edu/~stevko/courses/cse486/spring19/lectures/14-dht.pdf">Lecture slides on Chord</a></u></li>

 <li><u><a href="http://www.cse.buffalo.edu/~stevko/courses/cse486/spring19/files/chord_sigcomm.pdf">Chord paper</a></u></li>

</ol>

The lecture slides give an overview, but do not discuss Chord in detail, so it should be a good reference to get an overall idea. The paper presents pseudo code for implementing Chord, so it should be a good reference for actual implementation.

<h1>Note</h1>

<em>It is important to remember that this assignment does not require you to implement everything about Chord. </em>​Mainly, there are three things you ​<strong><em>do not</em></strong>​ ​need to consider from the Chord paper.

<ol>

 <li>Fingers and finger-based routing (i.e., Section 4.3 &amp; any discussion about fingers in Section 4.4)</li>

 <li>Concurrent node joins (i.e., Section 5)</li>

 <li>Node leaves/failures (i.e., Section 5)</li>

</ol>

We will discuss this more in “Step 2: Writing a Content Provider” below.

<h1>Step 0: Importing the project template</h1>

Just like the previous assignment, we have a project template you can import to Android Studio.

<ol>

 <li>Download<u>​</u><u><a href="http://www.cse.buffalo.edu/~stevko/courses/cse486/spring19/files/SimpleDht.zip"> the project template zip file</a></u><u>​</u> to a directory.</li>

 <li>Extract the template zip file and copy it to your Android Studio projects directory.</li>

 <li>Make sure that you copy the correct directory. After unzipping, the directory name should be “SimpleDht”, and right underneath, it should contain a number of directories and files such as “app”, “build”, “gradle”, “build.gradle”, “gradlew”, etc.</li>

 <li>After copying, delete the downloaded template zip file and unzipped directories and files. This is to make sure that you do not submit the template you just downloaded. (There were many people who did this before.)</li>

 <li>Open the project template you just copied in Android Studio.</li>

 <li>Use the project template for implementing all the components for this assignment.</li>

 <li>The template has the package name of “edu.buffalo.cse.cse486586.simpledht“. Please do not change this.</li>

 <li>The template also defines a content provider authority and class. Please use it to implement your Chord functionalities.</li>

 <li>We will use SHA-1 as our hash function to generate keys. The following code snippet takes a string and generates a SHA-1 hash as a hexadecimal string. Please use it to generate your keys. The template already has the code, so you just need to use it at appropriate places. Given two keys, you can use the standard lexicographical string comparison to determine which one is greater.</li>

</ol>




<table width="0">

 <tbody>

  <tr>

   <td width="624">import java.security.MessageDigest;import java.security.NoSuchAlgorithmException; import java.util.Formatter; private String genHash(String input) throws NoSuchAlgorithmException { MessageDigest sha1 = MessageDigest.getInstance(“SHA-1”);byte[] sha1Hash = sha1.digest(input.getBytes());Formatter formatter = new Formatter(); for (byte b : sha1Hash) { formatter.format(“%02x”, b);}return formatter.toString();}</td>

  </tr>

 </tbody>

</table>

<h1>Step 1: Writing the Content Provider</h1>

First of all, your app should have a content provider. This content provider should implement all DHT functionalities. For example, it should create server and client threads (if this is what you decide to implement), open sockets, and respond to incoming requests; it should also implement a simplified version of the Chord routing protocol; lastly, it should also handle node joins. The following are the requirements for your content provider:

<ol>

 <li>We will test your app with any number of instances up to 5 instances.</li>

 <li>The content provider should implement all DHT functionalities. This includes all communication as well as mechanisms to handle insert/query requests and node joins.</li>

 <li>Each content provider instance should have a node id derived from its emulator port.</li>

</ol>

<em>This node id should be obtained by applying the above hash function (i.e., genHash()) to the emulator port.</em>​ For example, the node id of the content provider instance running on emulator-5554 should be, ​<em>node_id = genHash(“5554”)</em>​. This is necessary to find the correct position of each node in the Chord ring.

<ol start="4">

 <li>Your content provider should implement insert(), query(), and delete(). The basic interface definition is the same as the previous assignment, which allows a client app to insert arbitrary &lt;”key”, “value”&gt; pairs where both the key and the value are strings.

  <ol>

   <li>For delete(URI uri, String selection, String[] selectionArgs), you only need to use use the first two parameters, uri &amp; selection. This is similar to what you need to do with query().</li>

   <li><em>However, please keep in mind that this “key” should be hashed by the above genHash() before getting inserted to your DHT in order to find the correct position in the Chord ring. </em></li>

  </ol></li>

 <li>For your query() and delete(), you need to recognize two special strings for the ​<em>selection</em>

  <ol>

   <li>If * (​not​ including quotes, i.e., “*” should be the string in your code) is given as the <em>selection</em>​ parameter to query(), then you need to return all &lt;key, value&gt; pairs stored in your entire DHT.</li>

   <li>Similarly, if * is given as the ​<em>selection</em>​ parameter to delete(), then you need to delete all &lt;key, value&gt; pairs stored in your entire DHT.</li>

   <li>If @ (not​ including quotes, i.e., “@” should be the string in your code) is given as​ the <em>selection</em>​ ​ parameter to query() on an AVD, then you need to return all &lt;key, value&gt; pairs ​<em>stored in your local partition of the node</em>​, i.e., all &lt;key, value&gt; pairs stored locally in the AVD on which you run query().</li>

   <li>Similarly, if @ is given as the ​<em>selection</em>​ parameter to delete() on an AVD, then you need to delete all &lt;key, value&gt; pairs ​<em>stored in your local partition of the node</em>​, i.e., all &lt;key, value&gt; pairs stored locally in the AVD on which you run delete().</li>

  </ol></li>

 <li>An app that uses your content provider can give arbitrary &lt;key, value&gt; pairs, e.g., &lt;”I want to”, “store this”&gt;; then your content provider should hash the key via genHash(),</li>

</ol>

e.g., genHash(“I want to”), get the correct position in the Chord ring based on the hash value, and store &lt;”I want to”, “store this”&gt; in the appropriate node.

<ol start="7">

 <li>Your content provider should implement ring-based routing. Following the design of Chord, your content provider should maintain predecessor and successor pointers and forward each request to its successor until the request arrives at the correct node. Once the correct node receives the request, it should process it and return the result (directly or recursively) to the original content provider instance that first received the request.

  <ol>

   <li>Your content provider do not need to maintain finger tables and implement finger-based routing. This is not required.</li>

   <li>As with the previous assignment, we will fix all the port numbers (see below). This means that you can use the port numbers (11108, 11112, 11116, 11120, &amp; 11124) as your successor and predecessor pointers.</li>

  </ol></li>

 <li>Your content provider should handle new node joins. ​For this, you need to have the first emulator instance (i.e., emulator-5554) receive all new node join requests.​ Your implementation should not choose a random node to do that. Upon completing a new node join request, affected nodes should have updated their predecessor and successor pointers correctly.

  <ol>

   <li>Your content provider do not need to handle concurrent node joins. You can assume that a node join will only happen once the system completely processes the previous join.</li>

   <li>Your content provider do not need to handle insert/query requests while a node is joining. You can assume that insert/query requests will be issued only with a stable system.</li>

   <li>Your content provider do not need to handle node leaves/failures. This is not required.</li>

  </ol></li>

 <li>We have fixed the ports &amp; sockets.

  <ol start="10000">

   <li>Your app should open one server socket that listens on 10000.</li>

   <li>You need to use run_avd.py and set_redir.py to set up the testing environment.</li>

   <li>The grading will use 5 AVDs. The redirection ports are 11108, 11112, 11116, 11120, and 11124.</li>

   <li>You should just hard-code the above 5 ports and use them to set up connections.</li>

   <li>Please use the code snippet provided in PA1 on how to determine your local AVD.</li>

   <li>emulator-5554: “5554” ii. emulator-5556: “5556” iii. emulator-5558: “5558” iv. emulator-5560: “5560” v. emulator-5562: “5562”</li>

  </ol></li>

 <li>Your content provider’s URI should be:</li>

</ol>

“content://edu.buffalo.cse.cse486586.simpledht.provider”, which means that any app should be able to access your content provider using that URI. Your content provider does not need to match/support any other URI pattern.

<ol start="11">

 <li>As with the previous assignment, Your provider should have two columns.

  <ol>

   <li>The first column should be named as “key” (an all lowercase string without the quotation marks). This column is used to store all keys.</li>

   <li>The second column should be named as “value” (an all lowercase string without the quotation marks). This column is used to store all values.</li>

   <li>All keys and values that your provider stores should use the string data type.</li>

  </ol></li>

 <li>Note that your content provider should only store the &lt;key, value&gt; pairs local to its own partition.</li>

 <li>Any app (not just your app) should be able to access (read and write) your content provider. As with the previous assignment, please do not include any permission to access your content provider.</li>

 <li>Please read the notes at the end of this document. You might run into certain problems, and the notes might give you some ideas about a couple of potential problems.</li>

</ol>

<h1>Step 2: Writing the Main Activity</h1>

The template has an activity used for ​<em>your own testing and debugging.</em>​ It has three buttons, one button that displays “Test”, one button that displays “LDump” and another button that displays “GDump.” As with the previous assignment, “Test” button is already implemented (it’s the same as “PTest” from the last assignment). You can implement the other two buttons to further test your DHT.

<ol>

 <li>LDump

  <ol>

   <li>When touched, this button should dump and display all the &lt;key, value&gt; pairs <em>stored in your local partition of the node</em>​.</li>

   <li>This means that this button can give @ as the selection parameter to query().</li>

  </ol></li>

 <li>GDump

  <ol>

   <li>When touched, this button should dump and display all the &lt;key, value&gt; pairs stored in your ​<strong><em>whole</em></strong>​ Thus, LDump button is for local dump, and this button (GDump) is for global dump of the entire &lt;key, value&gt; pairs.</li>

   <li>This means that this button can give * as the selection parameter to query().</li>

  </ol></li>

</ol>

<h1>Testing</h1>




We have testing programs to help you see how your code does with our grading criteria. If you find any rough edge with the testing programs, please report it on Piazza so the teaching staff can fix it. The instructions are the following:

<ol>

 <li>Download a testing program for your platform. If your platform does not run it, please report it on Piazza.

  <ol start="8">

   <li><u><a href="http://www.cse.buffalo.edu/~stevko/courses/cse486/spring19/files/simpledht-grading.exe">Windows</a></u>​: We’ve tested it on 32- and 64-bit Windows 8.</li>

   <li><u><a href="http://www.cse.buffalo.edu/~stevko/courses/cse486/spring19/files/simpledht-grading.linux">Linux</a></u>​: We’ve tested it on 32- and 64-bit Ubuntu 12.04.</li>

   <li><u><a href="http://www.cse.buffalo.edu/~stevko/courses/cse486/spring19/files/simpledht-grading.osx">OS X</a></u>​: We’ve tested it on 32- and 64-bit OS X 10.9 Mavericks.</li>

  </ol></li>

 <li>Before you run the program, please make sure that you are running five AVDs. ​python py 5​ will do it.</li>

 <li>Run the testing program from the command line.</li>

 <li>On your terminal, it will give you your partial and final score, and in some cases, problems that the testing program finds.</li>

</ol>