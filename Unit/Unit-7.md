<div align="center">

[**_``Go Back``_**](../README.md)

# Sockets for Servers

</div>

## Using ServerSockets
--------------------------
The ``ServerSocket`` class contains everything needed to write servers in Java. It has constructors that create new ``ServerSocket`` objects, methods that listen for connections on a specified port, methods that configure the various server socket options, and the usual miscellaneous methods such as **``toString()``**.

In Java, the basic life cycle of a server program is this:

1. A new ``ServerSocket`` is created on a particular port using a **``ServerSocket()``** constructor.

2. The ``ServerSocket`` listens for incoming connection attempts on that port using its **``accept()``** method. **``accept()``** blocks until a client attempts to make a connection, at which point **``accept()``** returns a ``Socket`` object connecting the client and the server.

3. Depending on the type of server, either the Socket’s **``getInputStream()``** method, **``getOutputStream()``** method, or both are called to get input and output streams that communicate with the client.

4. The server and the client interact according to an agreed-upon protocol until it is time to close the connection.

5. The server, the client, or both close the connection.

6. The server returns to step ``2`` and waits for the next connection.

 **Implementing daytime server :**

Implementing your own daytime server is easy. First, create a server socket that listens on port ``8080``:

```Java
ServerSocket server = new ServerSocket(8080);
```

Next, accept a connection:

```Java
Socket connection = server.accept();
```

The **``accept()``** call blocks. That is, the program stops here and waits, possibly for hours or days, until a client connects on port ``8080``. When a client does connect, the **``accept()``** method returns a ``Socket`` object.

Note that the connection is returned a ``java.net.Socket`` object. The daytime protocol requires the server (and only the server) to talk, so get an ``OutputStream`` from the socket. Because the daytime protocol requires text, chain this to an ``OutputStreamWriter``:

```Java
OutputStream out = connection.getOutputStream();
Writer writer = new OutputStreamWriter(writer, "ASCII");
```

Now get the current time and write it onto the stream. The daytime protocol doesn’t require any particular format other than that it be human readable, so let Java pick for you:

```Java
Date now = new Date();
out.write(now.toString() +"\r\n");
```

Finally, flush the connection and close it:

```Java
out.flush();
connection.close();
```

Of course, you’ll want to do all this repeatedly, so you’ll put this all inside a loop.

```Java
while (true) {
    try (Socket connection = server.accept()) {
        Writer out = new OutputStreamWriter(connection.getOutputStream());
        Date now = new Date();
        out.write(now.toString() +"\r\n");
        out.flush();
    } catch (IOException ex) {
        System.err.println(ex);
    }
}
```

**Program:**

```Java
import java.net.*;
import java.io.*;
import java.util.Date;
public class DayTimeServer {
    public final static int PORT = 8080;
    public static void main(String[] args) {
        try (ServerSocket server = new ServerSocket(PORT)) {
            while (true) {
                try (Socket connection = server.accept()) {
                    Writer out = new OutputStreamWriter(connection.getOutputStream());
                    Date now = new Date();
                    out.write(now.toString() +"\r\n");
                    out.flush();
                } catch (IOException ex) {
                    System.err.println(ex);
                }
            }
        } catch (IOException ex) {
            System.err.println(ex);
        }
    }
}
```

Connecting from Telnet, you should see something like this:

**Output:**

```
> telnet localhost 8080
Trying ::1...
Connected to localhost.
Escape character is '^]'.
Sun Jul 30 21:21:16 NPT 2023
Connection closed by foreign host.
```

### **Serving Binary Data**

Sending binary, nontext data is not significantly harder. You just use an ``OutputStream`` that writes a byte array rather than a ``Writer`` that writes a ``String``. 

```Java
import java.net.*;
import java.io.*;
import java.util.Date;

public class TimeServer {
    public final static int PORT = 8080;

    public static void main(String[] args) {
        long differenceBetweenEpochs = 2208988800L;
        try (ServerSocket server = new ServerSocket(PORT)) {
            while (true) {
                try (Socket connection = server.accept()) {
                    OutputStream out = connection.getOutputStream();
                    Date now = new Date();
                    long msSince1970 = now.getTime();
                    long secondsSince1970 = msSince1970 / 1000;
                    long secondsSince1900 = secondsSince1970 + differenceBetweenEpochs;
                    byte[] time = new byte[4];
                    time[0] = (byte) ((secondsSince1900 & 0x00000000FF000000L) >> 24);
                    time[1] = (byte) ((secondsSince1900 & 0x0000000000FF0000L) >> 16);
                    time[2] = (byte) ((secondsSince1900 & 0x000000000000FF00L) >> 8);
                    time[3] = (byte) (secondsSince1900 & 0x00000000000000FFL);
                    out.write(time);
                    out.flush();
                } catch (IOException ex) {
                    System.err.println(ex.getMessage());
                }
            }
        } catch (IOException ex) {
            System.err.println(ex);
        }
    }
}
```

### **Multithreaded Servers**

The server sends a few dozen bytes at most and then closes the connection. It’s plausible here to process each connection fully before moving on to the next one. Even in that case, though, it is possible that a slow or crashed client might hang the server for a few seconds until it notices the socket is broken. If the sending of data can take a significant amount of time even when client and server are behaving, you really don’t want each connection to wait for the next.

The solution here is to give each connection its own thread :

```Java
import java.net.*;
import java.io.*;
import java.util.Date;
public class MultithreadedDaytimeServer {
    public final static int PORT = 8080;

    public static void main(String[] args) {
        try (ServerSocket server = new ServerSocket(PORT)) {
            System.out.println("Server started. Listening on port: " + PORT);
            while (true) {
                try {
                    Socket connection = server.accept();
                    System.out.println("New client connected: " + connection.getInetAddress());
                    Thread task = new DaytimeThread(connection);
                    task.start();
                } catch (IOException ex) {
                    System.err.println(ex);
                }
            }
        } catch (IOException ex) {
            System.err.println("Couldn't start server");
        }
    }
}
class DaytimeThread extends Thread {
    private Socket connection;
    DaytimeThread(Socket connection) {
        this.connection = connection;
    }
    @Override
    public void run() {
        try {
            Writer out = new OutputStreamWriter(connection.getOutputStream());
            Date now = new Date();
            out.write(now.toString() + "\r\n");
            out.flush();
        } catch (IOException ex) {
            System.err.println(ex);
        } finally {
            try {
                connection.close();
            } catch (IOException e) {
                System.out.println(e);
            }
        }
    }
}
```

### **Writing to Servers with Sockets**

In the examples so far, the server has only written to client sockets. It hasn’t read from them. Most protocols, however, require the server to do both. This isn’t hard. You’ll accept a connection as before, but this time ask for both an ``InputStream`` and an ``OutputStream``. Read from the client using the ``InputStream`` and write to it using the ``OutputStream``. The main trick is understanding the protocol: when to write and when to read.

```Java
import java.io.*;
import java.net.*;

public class ServerWithReadWrite {
    public static void main(String[] args) {
        int port = 12345;

        try (ServerSocket serverSocket = new ServerSocket(port)) {
            System.out.println("Server started. Listening on port: " + port);

            while (true) {
                try {
                    Socket clientSocket = serverSocket.accept();
                    System.out.println("New client connected: " + clientSocket.getInetAddress());

                    // Create input and output streams for the client socket
                    InputStream inputStream = clientSocket.getInputStream();
                    OutputStream outputStream = clientSocket.getOutputStream();

                    // Create readers and writers for client communication
                    BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
                    PrintWriter writer = new PrintWriter(outputStream, true);

                    // Read from the client and write back
                    String clientMessage = reader.readLine();
                    System.out.println("Received from client: " + clientMessage);

                    // Echo the message back to the client
                    writer.println("Server: " + clientMessage);

                    // Close the client socket
                    clientSocket.close();
                    System.out.println("Client disconnected.");
                } catch (IOException ex) {
                    System.err.println(ex);
                }
            }
        } catch (IOException ex) {
            System.err.println("Couldn't start server");
        }
    }
}
```

### **Closing Server Sockets**

If you’re finished with a server socket, you should close it, especially if the program is going to continue to run for some time. This frees up the port for other programs that may wish to use it. Closing a ``ServerSocket`` should not be confused with closing a ``Socket``. Closing a ``ServerSocket`` frees a port on the local host, allowing another server to bind to the port; it also breaks all currently open sockets that the ``ServerSocket`` has accepted.

Server sockets are closed automatically when a program dies, so it’s not absolutely necessary to close them in programs that terminate shortly after the ``ServerSocket`` is no longer needed. Nonetheless, it doesn’t hurt. Programmers often follow the same closeif-not-null pattern in a ``try-finally`` block that you’re already familiar with from streams and client-side sockets:

```Java
ServerSocket server = null;
try {
    server = new ServerSocket(port);
    // ... work with the server socket
} finally {
    if (server != null) {
        try {
            server.close();
        } catch (IOException ex) {
            // ignore
        }
    }
}
```

You can improve this slightly by using the noargs **``ServerSocket()``** constructor, which does not throw any exceptions and does not bind to a port. Instead, you call the **``bind()``** method to bind to a socket address after the **``ServerSocket()``** object has been constructed:

```Java
ServerSocket server = new ServerSocket();
try {
    SocketAddress address = new InetSocketAddress(port);
    server.bind(address);
    // ... work with the server socket
} finally {
    try {
        server.close();
    } catch (IOException ex) {
        // ignore
    }
}
```

In Java 7, ``ServerSocket`` implements ``AutoCloseable`` so you can take advantage of ``try-with-resources`` instead:

```Java
try (ServerSocket server = new ServerSocket(port)) {
    // ... work with the server socket
}
```

After a server socket has been closed, it cannot be reconnected, even to the same port.

The **``isClosed()``** method returns ``true`` if the ``ServerSocket`` has been closed, ``false`` if it hasn’t:

```Java
public boolean isClosed()
```

``ServerSocket`` objects that were created with the noargs ServerSocket() constructor and not yet bound to a port are not considered to be closed. Invoking **``isClosed()``** on these objects returns ``false``. The **``isBound()``** method tells you whether the ``ServerSocket`` has been bound to a port:

```Java
public boolean isBound()
```

As with the ``isBound()`` method of the ``Socket`` class , the name is a little misleading. **``isBound()``** returns ``true`` if the **``ServerSocket``** has ever been bound to a port, even if it’s currently closed. If you need to test whether a ``ServerSocket`` is open, you must check both that **``isBound()``** returns ``true`` and that **``isClosed()``** returns ``false``.

For example:

```Java
public static boolean isOpen(ServerSocket ss) {
    return ss.isBound() && !ss.isClosed();
}
```

## Logging
-----------
``Servers`` run unattended for long periods of time. It’s often important to ``debug`` what happened when in a server long after the fact. For this reason, it’s advisable to store server logs for at least some period of time.

### **What to Log**

In a Java server application, logging is an essential practice to track and record important events, errors, and information about the application's behavior. It helps in monitoring the application's performance, diagnosing issues, and understanding user interactions.There are two primary things you want to store in your logs:


- ``Relevant Information``: Logs should contain relevant information about the application's behavior, events, and interactions. This includes details such as incoming requests, outgoing responses, exceptions, errors, warnings, and any other significant events that occur during the application's execution. These logs help developers and system administrators understand what the application is doing, diagnose issues, and monitor its performance.

- ``Timestamps``: Each log entry should include a timestamp indicating when the event occurred. Timestamps are crucial for understanding the chronological order of events and for analyzing the sequence of actions in the application. They are also essential for identifying the duration between events, which helps in performance monitoring and debugging.

Additionally, logs should be formatted in a consistent and structured manner to make them easily parseable and filterable. Common log formats include ``JSON``, ``XML``, or plain text with predefined fields to capture different types of information, such as ``severity level``, ``timestamp``, ``message``, ``source IP``, ``request path``, etc.

By storing relevant information with proper ``timestamps``, logs become a valuable tool for troubleshooting, performance analysis, and auditing in Java server applications or any other software system.

### **How to Log**

To implement logging in a Java server application, consider using a logging framework like ``Log4j``, ``Logback``, or Java's built-in ``java.util.logging``. These frameworks provide various logging levels (e.g., ``INFO``, ``DEBUG``, ``ERROR``) to control the verbosity of logs and allow you to configure log outputs (e.g., console, file) and log formats. But we will use Java `` java.util.logging``.

Although you can load a logger on demand, it’s usually easiest to just create one per class like so:

```Java
private final static Logger auditLogger = Logger.getLogger("requests");
```

Loggers are thread safe, so there’s no problem storing them in a shared static field. Indeed, they almost have to be because even if the ``Logger`` object were not shared between threads, the logfile or database would be. This is important in highly multithreaded servers.

This example outputs to a log named ``“requests”``. Multiple ``Logger`` objects can output to the same log, but each logger always logs to exactly one log. What and where the log is depends on external configuration. Most commonly it’s a file, which may or may not be named ``“requests”``; but it can be a database, a ``SOAP`` service running on a different server, another Java program on the same host, or something else.

Once you have a logger, you can write to it using any of several methods. The most basic is **``log()``**. For example, this catch block logs an unexpected runtime exception at the highest level:

```Java
catch (RuntimeException ex) {
    logger.log(Level.SEVERE, "unexpected error " + ex.getMessage(), ex);
}
```

Including the exception instead of just a message is optional but customary when logging from a ``catch`` block.

There are seven levels defined as named constants in ``java.util.logging.Level`` in descending order of seriousness:

- ``Level.SEVERE`` (highest value)
- ``Level.WARNING``
- ``Level.INFO``
- ``Level.CONFIG``
- ``Level.FINE``
- ``Level.FINER``
- ``Level.FINEST`` (lowest value)

I use info for audit logs and warning or severe for error logs. Lower levels are for debugging only and should not be used in production systems. Info, severe, and warning all have convenience helper methods that log at that level. For example, this statement logs a hit including the date and the remote address:

```Java
logger.info(new Date() + " : " + connection.getRemoteSocketAddress());
```

You can use any format that’s convenient for the individual log records. Generally, each record should contain a timestamp, the client address, and any information specific to the request that was being processed. If the log message represents an error, include the specific exception that was thrown. Java fills in the location in the code where the message was logged automatically, so you don’t need to worry about that.

**Program:**:

```Java
import java.util.Date;
import java.util.logging.Level;
import java.util.logging.Logger;

public class LoggingDemo {
    private final static Logger errorLogger = Logger.getLogger("errors");

    public static void main(String[] args) {
        try {
            int result = 10/ 0; // Attempting to divide by zero
            System.out.println("Result: " + result);
        } catch (ArithmeticException e) {
            errorLogger.log(Level.WARNING, "Error", e);
            errorLogger.info(new Date() + " : " + e);
        }
    }
}
```

**Output:**

```
Jul 31, 2023 10:19:00 PM LoggingDemo main
WARNING: Error :
java.lang.ArithmeticException: / by zero
	at LoggingDemo.main(LoggingDemo.java:10)

Jul 31, 2023 10:19:00 PM LoggingDemo main
INFO: Mon Jul 31 22:19:00 NPT 2023 : java.lang.ArithmeticException: / by zero
```

## Constructing Server Sockets
-------------------------------

There are four public ``ServerSocket`` constructors:

```Java
public ServerSocket(int port) throws BindException, IOException
public ServerSocket(int port, int queueLength) throws BindException, IOException
public ServerSocket(int port, int queueLength, InetAddress bindAddress)throws IOException
public ServerSocket() throws IOException
```

These constructors specify the port, the length of the queue used to hold incoming connection requests, and the local network interface to bind to. They pretty much all do the same thing, though some use default values for the queue length and the address to bind to.

For example, to create a server socket that would be used by an ``HTTP`` server on port ``80``, you would write:

```Java
ServerSocket httpd = new ServerSocket(80);
```
To create a server socket that would be used by an HTTP server on port ``80`` and queues up to ``50`` unaccepted connections at a time:

```Java
ServerSocket httpd = new ServerSocket(80, 50);
```

If you try to expand the queue past the operating system’s maximum queue length, the maximum queue length is used instead.

By default, if a host has multiple network interfaces or IP addresses, the server socket listens on the specified port on all the interfaces and IP addresses. However, you can add a third argument to bind only to one particular local IP address. That is, the server socket only listens for incoming connections on the specified address; it won’t listen for connections that come in through the host’s other addresses.

```Java
InetAddress local = InetAddress.getByName("192.168.210.122");
ServerSocket httpd = new ServerSocket(5776, 10, local);
```

In all three constructors, you can pass 0 for the port number so the system will select an available port for you. A port chosen by the system like this is sometimes called an ``anonymous port`` because you don’t know its number in advance (though you can find out after the port has been chosen). 

All these constructors throw an ``IOException``, specifically, a ``BindException``, if the socket cannot be created and bound to the requested port. An ``IOException`` when creating a ``ServerSocket`` almost always means one of two things.

### **Constructing Without Binding**

The noargs constructor creates a ServerSocket object but does not actually bind it to a port, so it cannot initially accept any connections. It can be bound later using the **``bind()``** methods:

```Java
public void bind(SocketAddress endpoint) throws IOException
public void bind(SocketAddress endpoint, int queueLength) throws IOException
```

The primary use for this feature is to allow programs to set server socket options before binding to a port. Some options are fixed after the server socket has been bound. The general pattern looks like this:

```Java
ServerSocket s = new ServerSocket();
// set socket options...
SocketAddress http = new InetSocketAddress(80);
s.bind(http);
```
You can also pass ``null`` for the SocketAddress to select an arbitrary port. This is like passing ``0`` for the port number in the other constructors.

## Getting Information About a Server Socket
--------------------------------------------
The ``ServerSocket`` class provides two getter methods that tell you the local address and port occupied by the server socket. These are useful if you’ve opened a server socket on an anonymous port and/or an unspecified network interface.

```Java
public InetAddress getInetAddress()
```

This method returns the ``address`` being used by the server (the local host). If the localhost has a single IP address (as most do), this is the address returned by ``InetAddress.getLocalHost()``. If the local host has more than one IP address, the specific address returned is one of the host’s IP addresses. You can’t predict which address you will get. For example:

```Java
ServerSocket httpd = new ServerSocket(80);
InetAddress ia = httpd.getInetAddress();
```

If the ``ServerSocket`` has not yet bound to a network interface, this method returns ``null``.

```Java
public int getLocalPort()
```

The ``ServerSocket`` constructors allow you to listen on an unspecified port by passing ``0`` for the port number. This method lets you find out what port you’re listening on.

**Program:**

```Java
import java.io.IOException;
import java.net.ServerSocket;

public class RandomPortServer {
    public static void main(String[] args) {
        try {
            ServerSocket server = new ServerSocket(0);
            System.out.println("This server runs on port : " + server.getLocalPort());
        } catch (IOException ex) {
            System.err.println(ex);
        }
    }
}
```

**Output:**

```
This server runs on port : 45039
This server runs on port : 43639
```

At least on this system, the ports aren’t truly ``random``, but they are indeterminate until runtime.

If the ``ServerSocket`` has not yet bound to a port, **``getLocalPort()``** returns ``–1``.

As with most ``Java`` objects, you can also just print out a ``ServerSocket`` using its **``toString()``** method. A String returned by a ``ServerSocket``’s **``toString()``** method looks like this:

```
ServerSocket[addr=0.0.0.0,port=0,localport=5776]
```
``addr`` is the address of the local network interface to which the server socket is bound. This will be ``0.0.0.0`` if it’s bound to all interfaces, as is commonly the case. port is always ``0``. The ``localport`` is the local port on which the server is listening for connections. This method is sometimes useful for debugging, but not much more. Don’t rely on it.

## Socket Options
-----------------
``Socket`` options specify how the native sockets on which the ``ServerSocket`` class relies send and receive data. For server sockets, ``Java`` supports three options:

- ``SO_TIMEOUT``
- ``SO_REUSEADDR``
- ``SO_RCVBUF``

It also allows you to set performance preferences for the socket’s packets.

### **SO_TIMEOUT**

``SO_TIMEOUT`` is the amount of time, in milliseconds, that **``accept()``** waits for an incoming connection before throwing a ``java.io.InterruptedIOException``. If ``SO_TIMEOUT`` is ``0``, **``accept()``** will never time out. The default is to never time out.

Setting ``SO_TIMEOUT`` is uncommon. You might need it if you were implementing a complicated and secure protocol that required multiple connections between the client and the server where responses needed to occur within a fixed amount of time. However, most servers are designed to run for indefinite periods of time and therefore just use the default timeout value, ``0`` (never time out). If you want to change this, the **``setSoTimeout()``** method sets the ``SO_TIMEOUT`` field for this server socket object:

```Java
public void setSoTimeout(int timeout) throws SocketException
public int getSoTimeout() throws IOException
```

The countdown starts when **``accept()``** is invoked. When the timeout expires, **``accept()``** throws a ``SocketTimeoutException``, a subclass of ``IOException``. You need to set this option before calling **``accept()``**; you cannot change the ``timeout`` value while **``accept()``** is waiting for a connection. The ``timeout`` argument must be greater than or equal to ``0``; if it isn’t, the method throws an ``IllegalArgumentException``. For example:

```Java
try (ServerSocket server = new ServerSocket(port)) {
    server.setSoTimeout(30000); // block for no more than 30 seconds
    try {
        Socket s = server.accept();
        // handle the connection
        // ...
    } catch (SocketTimeoutException ex) {
        System.err.println("No connection within 30 seconds");
    }
} catch (IOException ex) {
    System.err.println("Unexpected IOException: " + e);
}
```
The **``getSoTimeout()``** method returns this server socket’s current ``SO_TIMEOUT`` value. For example:

```Java
public void printSoTimeout(ServerSocket server) {
    int timeout = server.getSoTimeOut();
    if (timeout > 0) {
        System.out.println(server + " will time out after " + timeout + "milliseconds.");
    } else if (timeout == 0) {
        System.out.println(server + " will never time out.");
    } else {
        System.out.println("Impossible condition occurred in " + server);
        System.out.println("Timeout cannot be less than zero." );
    }
}
```

### **SO_REUSEADDR**

The ``SO_REUSEADDR`` option for server sockets is very similar to the same option for client sockets. It determines whether a new socket will be allowed to bind to a previously used port while there might still be data traversing the network addressed to the old socket. As you probably expect, there are two methods to get and set this option:

```Java
public boolean getReuseAddress() throws SocketException
public void setReuseAddress(boolean on) throws SocketException
```

The default value is platform dependent. This code fragment determines the default value by creating a new ``ServerSocket`` and then calling **``getReuseAddress()``**:

```Java
ServerSocket s = new ServerSocket(10240);
System.out.println("Reusable: " + s.getReuseAddress());
```

### **SO_RCVBUF**

The ``SO_RCVBUF`` option sets the default receive buffer size for client sockets accepted by the server socket. It’s read and written by these two methods:

```Java
public int getReceiveBufferSize() throws SocketException
public void setReceiveBufferSize(int size) throws SocketException
```
Setting ``SO_RCVBUF`` on a server socket is like calling **``setReceiveBufferSize()``** on each individual socket returned by **``accept()``**. 

You can set this option before or after the server socket is bound, unless you want to set a receive buffer size larger than ``64K``. In that case, you must set the option on an unbound ``ServerSocket`` before binding it. For example:

```Java
ServerSocket s = new ServerSocket();
int receiveBufferSize = s.getReceiveBufferSize();
if (receiveBufferSize < 131072) {
 s.setReceiveBufferSize(131072);
}
s.bind(new InetSocketAddress(8000));
```

### **Class of Service**

Different types of Internet services have different performance needs. For instance, live streaming video of sports needs relatively high
bandwidth. On the other hand, a movie might still need high bandwidth but be able to tolerate more delay and latency. Email can be passed over low-bandwidth connections and even held up for several hours without major harm.

Four general traffic classes are defined for TCP:
- Low cost
- High reliability
- Maximum throughput
- Minimum delay

These traffic classes can be requested for a given ``Socket``. For instance, you can request the minimum delay available at low cost. These measures are all fuzzy and relative, not guarantees of service. Not all routers and native ``TCP`` stacks support these classes.

```Java
public void setPerformancePreferences(int connectionTime, int latency, int bandwidth)
```

For instance, by setting ``connectionTime`` to ``2``, ``latency`` to ``1``, and ``bandwidth`` to ``3``, you indicate that maximum bandwidth is the most important characteristic, minimum latency is the least important, and connection time is in the middle:

```Java
s.setPerformancePreferences(2, 1, 3);
```

Exactly how any given VM implements this is implementation dependent. The underlying socket implementation is not required to respect any of these requests. They only provide a hint to the TCP stack about the desired policy. Many implementations including Android ignore these values completely.

## HTTP Server
--------------
Java Sockets provide a low-level API for network communication, and HTTP is a protocol built on top of TCP/IP. 

### **A Single-File Server**
A Single-File Server serves a single HTML file to the client. It listens for incoming client connections on a specified port. When a client connects, it reads the contents of the file and sends it as an HTTP response.

**Program:**
```Java
import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;

public class SingleFileServer {
    public final static int PORT = 8080;

    public static void main(String[] args) {
        try {
            ServerSocket serverSocket = new ServerSocket(PORT);
            System.out.println("Single-File Server listening on port : " + PORT);
            while (true) {
                Socket clientSocket = serverSocket.accept();

                BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
                PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true);

                // Read the request from the client (we ignore it for this example)
                String requestLine = in.readLine();
                System.out.println("Received request: " + requestLine);

                // Send the HTTP response with the contents of the file
                out.println("HTTP/1.1 200 OK");
                out.println("Content-Type: text/html");
                out.println();
                out.println("<html><body>");
                out.println("<h1>Hello from Single-File Server!</h1>");
                out.println("</body></html>");

                // Close the streams and the socket
                in.close();
                out.close();
                clientSocket.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```

### **A Redirector**
A ``Redirector`` Server listens for incoming requests and redirects the client to a different URL using an HTTP status code ``301``. This is typically used when a resource has moved permanently to a new location, and the server informs the client about the new URL to access the resource.

**Program:**

```Java
import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;

public class RedirectorServer {
    public final static int PORT = 8080;

    public static void main(String[] args) {
        try {
            ServerSocket serverSocket = new ServerSocket(PORT);
            System.out.println("Redirector Server listening on port : " + PORT);
            while (true) {
                Socket clientSocket = serverSocket.accept();

                // Send the HTTP response with a redirect header
                String response = "HTTP/1.1 301 Moved Permanently\r\nLocation: http://www.example.com\r\n\r\n";
                clientSocket.getOutputStream().write(response.getBytes());

                // Close the socket
                clientSocket.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### **A Full-Fledged HTTP Server**

Creating a Full-Fledged server in Java involves significant complexity and typically requires implementing various functionalities and handling different types of requests. Here, I'll provide you with a simple **multi-threaded** ``HTTP`` server that can handle basic ``GET`` requests for static files. 

```Java
import java.io.*;
import java.net.*;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class SimpleHttpServer {
    private static final int PORT = 8080;
    private static final String ROOT_DIRECTORY = "www"; // Change this to your web content directory

    public static void main(String[] args) {
        ExecutorService pool = Executors.newFixedThreadPool(10); // Adjust the number of threads as needed

        try (ServerSocket serverSocket = new ServerSocket(PORT)) {
            System.out.println("Server started. Listening on port " + PORT);

            while (true) {
                Socket clientSocket = serverSocket.accept();
                System.out.println("New client connected: " + clientSocket.getInetAddress());

                // Create a worker thread to handle the client request
                Runnable worker = new ClientHandler(clientSocket);
                pool.execute(worker);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static class ClientHandler implements Runnable {
        private final Socket clientSocket;

        ClientHandler(Socket socket) {
            this.clientSocket = socket;
        }

        @Override
        public void run() {
            try (InputStream inputStream = clientSocket.getInputStream();
                 OutputStream outputStream = clientSocket.getOutputStream();
                 BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
                 PrintWriter writer = new PrintWriter(outputStream, true)) {

                String request = reader.readLine();
                if (request != null) {
                    System.out.println("Received request: " + request);

                    // Extract the requested file path from the request
                    String[] requestParts = request.split(" ");
                    if (requestParts.length >= 2 && requestParts[0].equalsIgnoreCase("GET")) {
                        String filePath = requestParts[1];

                        // Handle the request by serving static files
                        serveStaticFile(filePath, writer);
                    } else {
                        // Handle other HTTP methods (e.g., POST) or invalid requests here
                        writer.println("HTTP/1.1 501 Not Implemented");
                    }
                }
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                try {
                    clientSocket.close();
                    System.out.println("Client disconnected.");
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

        private void serveStaticFile(String filePath, PrintWriter writer) {
            try {
                File file = new File(ROOT_DIRECTORY, filePath);

                if (file.exists() && file.isFile()) {
                    writer.println("HTTP/1.1 200 OK");
                    writer.println("Content-Type: " + getContentType(file.getName()));
                    writer.println("Content-Length: " + file.length());
                    writer.println();
                    writer.flush();

                    try (FileInputStream fileInputStream = new FileInputStream(file)) {
                        byte[] buffer = new byte[1024];
                        int bytesRead;
                        OutputStream outputStream = clientSocket.getOutputStream();
                        while ((bytesRead = fileInputStream.read(buffer)) != -1) {
                            outputStream.write(buffer, 0, bytesRead);
                        }
                    }
                } else {
                    // File not found
                    writer.println("HTTP/1.1 404 Not Found");
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

        private String getContentType(String fileName) {
            if (fileName.endsWith(".html") || fileName.endsWith(".htm")) {
                return "text/html";
            } else if (fileName.endsWith(".jpg") || fileName.endsWith(".jpeg")) {
                return "image/jpeg";
            } else if (fileName.endsWith(".gif")) {
                return "image/gif";
            } else if (fileName.endsWith(".png")) {
                return "image/png";
            } else {
                return "application/octet-stream";
            }
        }
    }
}
```