### RabbitMQ Training with C\#
This training will cover the rabbitmq tutorial at url [RabbitMQ Tutorial](https://www.rabbitmq.com/tutorials/tutorial-one-dotnet.html).
Code for this tutorial is located at [https://github.com/ldillon-perforce/rabbitmq-training]
### Installing and configuring RabbitMQ on Windows
* ``https://www.rabbitmq.com/install-windows.html``
* ``https://erlang.org/download/otp_versions_tree.html``

If using Internet Explorer, you might need to configure security:

* Gear icon in upper left corner
* Internet Options
* Security tab
* Highlight Internet icon
* Click Custom level...
* Scroll down to File Download under Downloads section
* Click OK

* Download `OTP 23.1.2 win64`.  This is the erlang installer.  Erlang is the language rabbitmq is written in.  This must be installed first before rabbitmq.
* Right click, run as administrator
* Download `rabbitmq-server-3.8.9.exe` from the rabbitmq page
* Run installer as administrator
* May say it is unknown and can't run, but there is `More info`.  Click that
* Click `Run anyway`

Will install and create some start menu entries.  Cilck on `RabbitMQ Command Prompt`

It will place you in a directory.  You can type `rabbitmqctl.bat status` to see current status of rabbitmq.

### Creating Projects
Create a directory to hold your main source code.  I am using `Example01`, `Example02`, etc. as the names.
```
mkdir Example01
cd Example01
```
Change into each directory and run the following commands to create individual projects:
```
dotnet new console --name Send
mv Send/Program.cs Send/Send.cs
dotnet new console --name Receive
mv Receive/Program.cs Receive/Receive.cs
```
Now we are going to add the RabbitMQ client library information to each project:
```
cd Send
dotnet add package RabbitMQ.Client
dotnet restore
cd ../Receive
dotnet add package RabbitMQ.Client
dotnet restore
```

Everytime we have an example, we will create the main directory, create the individual projects, then add the RabbitMQ.Client package and restore.  We will provide source for each project that can be copied over the file, rather than having to type it in manually each time.

## Examples in C\#
### Simple example sending and receiving
Tutorial link: [Tutorial One](https://www.rabbitmq.com/tutorials/tutorial-one-dotnet.html)
Code used:

* ``Send``
* ``Receive``

This is a simple example that has two parts. The `Send` that puts messages on a queue, and the `Receive` that reads messages from the queue.
This creates a simple queue with the default exchange.  The consumer waits for a message, the sender sends the message to the queue.  The consumer then gets the message and prints it.

### Work queues
Tutorial Link: [Tutorial Two](https://www.rabbitmq.com/tutorials/tutorial-two-dotnet.html)
Code used:

* Part 1
    * ``NewTask1``
    * ``Worker1``
* Part 2
    * ``NewTask``
    * ``Worker``

#### Part 1:
This uses a simple round-robin policy to distribute messages to workers.  If worker 1 gets a message, then worker 2, then worker 2 finishes before worker 1, the next message will still go to worker 1.
Create three shells.  Change to `NewTask` in one shell, and `Worker1` in the other two.
Run the following in the `Worker1` directories:
```
dotnet run
```
Then run in `NewTask1`
```
dotnet run Message
```
One of the workers will get the message.  Press ENTER and run again.  The other worker will get the message now.
Now we will test adding delays to show that this is strict round-robin.
```
dotnet run Message1...........................
dotnet run Test...
dotnet run Test2...
dotnet run Test3.
dotnet run Test4.
dotnet run Test5
```
You will see that `Test`, `Test3`, and `Test5` go to the second worker, and after the first worker finishes with `Message1`, it will get the other messages and process them.  This means messages could be blocked until others are processed, even if there are available workers.  That's because we are auto-acknowledging, even if a message isn't finished yet.
In the next part we will change that.
####Part 2:
#####Temporary queues
Queue names are important when sharing queues between consumers (Work queue).  For this type, every consumer wants its own queue.  We could create a random queue, but we can let rabbitmq do this for us.
```
var queueName = channel.QueueDeclare().QueueName;
```
This creates a non-durable, exclusive, autodelete queue with a generated name.

Current code marks a message for deletion immediately after delivering, even if the consumer dies before it finishes processing.  To prevent this, we can use acknowledgments.  An ack is sent back by the consumer to tell rabbitmq that a message.
If a consumer dies, rabbitmq won't get an ack and will requeue the message.
Switch to `NewTask` and `Worker`
```
dotnet run Message1............................
dotnet run Test...
dotnet run Test2...
dotnet run Test3.
dotnet run Test4.
dotnet run Test5
```
You will see that while one of the workers is processing `Message1`, the other messages are going to the other worker.  That's because the first worker hasn't acknowledged the message yet, so the other messages get passed to other workers.
Try this again, but abort the first worker while it is in the middle of processing the first message, `Message1`.  Notice that it is immediately delivered to the other worker.  This is because of the acknowledgements.  Also, if the first test is aborted in the middle, the message is lost because the message was acked immediately, even though it wasn't processed yet.
We did this by setting prefetch to 1, rather than leaving at the default.

### Routing
Routing uses a routing key to send traffic to different queues.
We were using a fanout exchange, which sends the traffic to every queue bound to that exchange. 

* ``EmitLogs``
* ``ReceiveLogs``

Now we will change the exchange to a direct exchange.

* ``EmitLogsDirect``
* ``ReceiveLogsDirect``

### Topics
Topics are list of words, separated by dots. Examples:

* ``stock.usd.nyse``
* ``nyse.vmw``
* ``quick.orange.rabbit``

Wildcards:

* ``*.orange.*``
* ``*.*.rabbit``
* ``lazy.#``

Keys that don't match any bound patterns are lost.

`#` matches every key, like a fanout exchange.
When `*` and `#` aren't used, it is like a direct exchange.

### RPC

### Publisher Confirms
RabbitMQ Extension, so not part of the official amqp 0-9-1 standard.

#### commands used
``rabbitmqctl list_queues``
``rabbitmqctl delete_queue task_queue``

### Other topics
* Planning delayed messages (to be further processed as retries) on queues using C#
Clarification?
Set manual ack, 
* Use acknowledgments

