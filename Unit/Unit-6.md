<div align="center">

[**_``Go Back``_**](../README.md)

# Socket for Clients

</div>

## Introduction to Socket
--------------------------
A ``socket`` is a mechanism for allowing communication between processes, be it programs running on the same machine or different computers connected on a network.

<div align="center">

![Socket](img/Socket.png)

</div>

``Data`` is transmitted across the Internet in packets of finite size called ``datagrams``. Each ``datagram`` contains a ``header`` and a ``payload``. The ``header`` contains the ``address`` and ``port`` to which the ``packet`` is going, the ``address`` and ``port`` from which the ``packet`` came, a ``checksum`` to detect data corruption, and various other housekeeping information used to ensure reliable transmission. The ``payload`` contains the ``data`` itself. However, because ``datagrams`` have a finite length, it’s often necessary to split the data across multiple ``packets`` and reassemble it at the destination. It’s also possible that one or more packets may be lost or corrupted in transit and need to be retransmitted or that packets arrive out of order and need to be reordered. Keeping track of this—splitting the data into packets, generating headers, parsing the headers of incoming packets, keeping track of what packets have and haven’t been received, and so on—is a lot of work and requires a lot of intricate code.

Fortunately, you don’t have to do the work yourself. Sockets allow the programmer to treat a network connection as just another stream onto which bytes can be written and from which bytes can be read. Sockets shield the programmer from low-level details of the network, such as error detection, packet sizes, packet splitting, packet retransmission, network addresses, and more.

## Using Socket
----------------
<div align="center">

![Socket API](img/socket-api.png)

</div>

A ``socket`` is a connection between two hosts. It can perform seven basic operations:

- Connect to a remote machine
- Send data
- Receive data
- Close a connection
- Bind to a port
- Listen for incoming data
- Accept connections from remote machines on the bound port

Java’s ``Socket`` class, which is used by both ``clients`` and ``servers``, has methods that correspond to the first four of these operations. The last three operations are needed only by ``servers``, which wait for ``clients`` to connect to them. They are implemented by the ``ServerSocket`` class. Java programs normally use client sockets in the following fashion:

- The program creates a new socket with a constructor.
- The socket attempts to connect to the remote host.

Once the connection is established, the ``local`` and ``remote hosts`` get ``input`` and ``output`` streams from the socket and use those streams to send data to each other. This connection is _``full-duplex``_. Both ``hosts`` can ``send`` and ``receive`` data simultaneously. What the data means depends on the protocol; different commands are sent to an ``FTP`` server than to an ``HTTP`` server. There will normally be some agreed-upon handshaking followed by the transmission of data from one to the other.

When the transmission of data is complete, one or both sides ``close`` the connection. Some protocols, such as ``HTTP 1.0``, require the connection to be closed after each request is serviced. Others, such as ``FTP`` and ``HTTP 1.1``, allow multiple requests to be processed in a single connection.

### **Investigating Protocols with Telnet** 

To get a feel for how a protocol operates, you can use ``Telnet`` to connect to a server, type different commands to it, and watch its responses. By default, Telnet attempts to connect to port ``23``. To connect to servers on different ports, specify the port you want to connect to like this:

```bash
telnet localhost 25
```

### **Reading from Servers with Sockets**

Let’s begin with a simple example. You’re going to connect to the daytime server at the ``National Institute for Standards and Technology (NIST)`` and ask it for the current time. This protocol is defined in ``RFC 867``. Reading that, you see that the daytime server listens on port ``13``, and that the server sends the time in a human-readable format and closes the connection. You can test the daytime server with ``Telnet`` like this:

**Command:**

```bash
telnet time.nist.gov 13
```

**Output:**
```
Trying 129.6.15.28...
Connected to time.nist.gov.
Escape character is '^]'.
56375 13-03-24 13:37:50 50 0 0 888.8 UTC(NIST) *
Connection closed by foreign host.
```

Now let’s see how to retrieve this same data programmatically using sockets. First, open a socket to ``time.nist.gov`` on port ``13``:

```Java
Socket socket = new Socket("time.nist.gov", 13);
```

This doesn’t just create the object. It actually makes the connection across the network. If the connection times out or fails because the server isn’t listening on port ``13``, then the constructor throws an ``IOException``, so you’ll usually wrap this in a ``try`` block. In Java 7, ``Socket`` implements ``Autocloseable`` so you can use try-with-resources:

```Java
try (Socket socket = new Socket("time.nist.gov", 13)) {
    // read from the socket...
} catch (IOException ex) {
    System.err.println("Could not connect to time.nist.gov");
}
```

In ``Java 6`` and earlier, you’ll want to explicitly close the socket in a ``finally`` block to release resources the socket holds:

```Java
Socket socket = null;
try {
    socket = new Socket(hostname, 13);
    // read from the socket...
} catch (IOException ex) {
    System.err.println(ex);
} finally {
    if (socket != null) {
        try {
            socket.close();
        } catch (IOException ex) {
            // ignore
        }
    }
}
```

The next step is optional but highly recommended. Set a timeout on the connection using the **``setSoTimeout()``** method. Timeouts are measured in milliseconds, so this statement sets the socket to time out after ``15`` seconds of nonresponsiveness:

```Java
socket.setSoTimeout(15000);
```

Once you’ve opened the socket and set its timeout, call **``getInputStream()``** to return an ``InputStream`` you can use to read bytes from the socket. In general, a server can send any bytes at all; but in this specific case, the protocol specifies that those bytes must be ``ASCII``:

```Java
InputStream in = socket.getInputStream();
StringBuilder time = new StringBuilder();
InputStreamReader reader = new InputStreamReader(in, "ASCII");
for (int c = reader.read(); c != -1; c = reader.read()) {
    time.append((char) c);
}
System.out.println(time);
```

**Program:**

```Java
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.net.Socket;

public class DayTimeClient {
    public static final String SERVER = "time.nist.gov";
    public static final int PORT = 13;
    public static final int TIMEOUT = 15000;

    public static void main(String[] args) {
        try (Socket socket = new Socket(SERVER, PORT)) {
            socket.setSoTimeout(TIMEOUT);
            InputStream in = socket.getInputStream();
            StringBuilder time = new StringBuilder();
            InputStreamReader reader = new InputStreamReader(in, "ASCII");
            for (int c = reader.read(); c != -1; c = reader.read()) {
                time.append((char) c);
            }
            System.out.println(time);
        } catch (IOException ex) {
            System.err.println("Could not connect to : "+ SERVER);
        }
    }
}
```

**Output:**

```
60151 23-07-26 15:04:10 50 0 0 821.8 UTC(NIST) * 
```

### **Writing to Servers with Sockets**

Writing to a server is not noticeably harder than reading from one. You simply ask the socket for an output stream as well as an input stream. Although it’s possible to send data over the socket using the output stream at the same time you’re reading data over the input stream, most protocols are designed so that the client is either reading or writing over a socket, not both at the same time. In the most common pattern, the client sends a request. Then the server responds. The client may send another request, and the server responds again. This continues until one side or the other is done, and closes the connection.

One simple bidirectional TCP protocol is dict, defined in ``RFC 2229``. In this protocol, the client opens a socket to port ``2628`` on the dict server and sends commands such as **``DEFINE fd-eng-fin gold``**. This tells the server to send a definition of the word ``gold`` using its ``English-to-Finnish`` dictionary. (Different servers have different dictionaries installed.) After the first definition is received, the client can ask for another. When it’s done it sends the command **``quit``**. You can explore dict with Telnet like this:

**Command:**

```bash
telnet dict.org 2628
```

**Output:**

```
Trying 199.48.130.6...
Connected to dict.org.
Escape character is '^]'.
220 dict.dict.org dictd 1.12.1/rf on Linux 4.19.0-10-amd64 <auth.mime> <203192412.22470.1690386999@dict.dict.org>
DEFINE fd-eng-fin gold
150 1 definitions retrieved
151 "gold" fd-eng-fin "English-suomi FreeDict+WikDict dictionary ver. 2018.09.13"
gold //ɡoʊld// //ɡəʊld// //ɡʊld// <adj>
 [having the colour of gold] kullankeltainen, kullanvärinen
.
250 ok [d/m/c = 1/0/14; 0.000r 0.000u 0.000s]
```

You can see that control response lines begin with a three-digit code. The actual definition is plain text, terminated with a period on a line by itself.

It’s not hard to implement this protocol in Java. First, open a socket to a dict server ``dict.org`` on port ``2628``:

```Java
Socket socket = new Socket("dict.org", 2628);
```

Once again you’ll want to set a timeout in case the server hangs while you’re connected to it:

```Java
socket.setSoTimeout(15000);
```

In the dict protocol, the client speaks first, so ask for the output stream using **getOutputStream()**:

```Java
OutputStream out = socket.getOutputStream();
```

The **``getOutputStream()``** method returns a raw ``OutputStream`` for writing data from your application to the other end of the socket. You usually chain this stream to a more convenient class like ``DataOutputStream`` or ``OutputStreamWriter`` before using it. For performance reasons, it’s a good idea to buffer it as well. Because the ``dict`` protocol is text based, more specifically ``UTF-8`` based, it’s convenient to wrap this in a ``Writer``:

```Java
Writer writer = new OutputStreamWriter(out, "UTF-8");
```

Now write the command over the socket:

```Java
writer.write("DEFINE fd-eng-fin gold\r\n");
```
Finally, flush the output so you’ll be sure the command is sent over the network:

```Java
writer.flush();
```

The server should now respond with a definition. You can read that using the socket’s input stream:

```Java
InputStream in = socket.getInputStream();
BufferedReader reader = new BufferedReader(new InputStreamReader(in, "UTF-8"));
for (String line = reader.readLine(); !line.equals("."); line = reader.readLine()) {
    System.out.println(line);
}
```

When you see a period on a line by itself, you know the definition is complete. You can then send the quit over the output stream:

```Java
writer.write("quit\r\n");
writer.flush();
```

**Program:**

```Java
import java.io.*;
import java.net.Socket;

public class DictClient {
    public static final String SERVER = "dict.org";
    public static final int PORT = 2628;
    public static final int TIMEOUT = 15000;

    public static void main(String[] args) {
        try (Socket socket = new Socket(SERVER, PORT)) {
            String[] words = {
                "gold",
                "stone",
                "addasdasd"
            };
            socket.setSoTimeout(TIMEOUT);
            OutputStream out = socket.getOutputStream();
            Writer writer = new OutputStreamWriter(out, "UTF-8");
            writer = new BufferedWriter(writer);
            InputStream in = socket.getInputStream();
            BufferedReader reader = new BufferedReader(new InputStreamReader(in, "UTF-8"));
            for (String word : words) {
                define(word, writer, reader);
            }
            writer.write("quit\r\n");
            writer.flush();
        } catch (IOException ex) {
            System.err.println(ex);
        }
    }

    static void define(String word, Writer writer, BufferedReader reader) throws IOException {
        writer.write("DEFINE fd-eng-fin " + word + "\r\n");
        writer.flush();
        for (String line = reader.readLine(); line != null; line = reader.readLine()) {
            if (line.startsWith("250")) {
                // OK
                return;
            } else if (line.startsWith("552")) {
                // no match
                System.err.println("No definition found for " + word);
                return; 
            } else if (line.matches("\\d\\d\\d .*")) ;
            else if (line.trim().equals(".")) ;
            else System.out.println(line);
        }
    }
}
```

**Output:**

```
gold //ɡoʊld// //ɡəʊld// //ɡʊld// <adj>
 [having the colour of gold] kullankeltainen, kullanvärinen
stone //stoʊn// //stəʊn// <adj>
1.  [constructed of stone] kivi, kivinen
2.  [having the appearance of stone] kivijäljitelmä
stone //stoʊn// //stəʊn// <adv>
 [absolutely, completely] täysin
stone //stoʊn// //stəʊn// <n>
1.  [colour] kivenharmaa
2.  [gem stone] kivi, jalokivi
3.  [piece of hard material used in board games] nappula
4.  [substance] kivi
5.  [small piece of stone] kivi
6.  [medical: hard, stone-like deposit] kivi
7.  [centre of some fruits] kivi
8.  [curling stone] kivi
stone //stoʊn// //stəʊn// <v>
1.  [to form a stone] kasvattaa kivi
2.  [to intoxicate, especially with narcotics] huumata, juovuttaa, öykätä
3.  [to kill] kivittää
4.  [to remove stone from] poistaa kivi
No definition found for addasdasd
```

#### Half-closed sockets

The **``close()``** method shuts down both input and output from the socket. On occasion, you may want to shut down only half of the connection, either input or output. The **``shutdownInput()``** and **``shutdownOutput()``** methods close only half the connection:

```Java
public void shutdownInput() throws IOException
public void shutdownOutput() throws IOException
```

Neither actually closes the socket. Instead, they adjust the stream connected to the socket so that it thinks it’s at the end of the stream. Further reads from the input stream after shutting down input return ``–1``. Further writes to the socket after shutting down output throw an ``IOException``.

```Java
try (Socket connection = new Socket("www.example.com", 80)) {
    Writer out = new OutputStreamWriter(connection.getOutputStream(), "8859_1");
    out.write("GET / HTTP 1.0\r\n\r\n");
    out.flush();
    connection.shutdownOutput();
    // read the response...
} catch (IOException ex) {
    ex.printStackTrace();
}
```

The **``isInputShutdown()``** and **``isOutputShutdown()``** methods tell you whether the input and output streams are open or closed, respectively. You can use these (rather than **``isConnected()``** and **``isClosed()``**) to more specifically ascertain whether you can read from or write to a ``socket``:

```Java
public boolean isInputShutdown()
public boolean isOutputShutdown()
```

## Constructing and Connecting Sockets
---------------------------------------
The ``java.net.Socket`` class is Java’s fundamental class for performing client-side TCP operations. Other client-oriented classes that make TCP network connections such as ``URL``, ``URLConnection``, ``Applet``, and ``JEditorPane`` all ultimately end up invoking the methods of this class. This class itself uses native code to communicate with the local ``TCP`` stack of the host operating system.

### **Basic Constructors**

Each ``Socket`` constructor specifies the ``host`` and the ``port`` to connect to. Hosts may be specified as an ``InetAddress`` or a ``String``. Remote ports are specified as int values from ``1`` to ``65535``:

```Java
public Socket(String host, int port) throws UnknownHostException, IOException
public Socket(InetAddress host, int port) throws IOException
```

These constructors connect the socket (i.e., before the constructor returns, an active network connection is established to the remote host). If the connection can’t be opened for some reason, the constructor throws an ``IOException`` or an ``UnknownHostException``. For example:

**Example :**
```Java
try {
    Socket socket = new Socket("www.example.com", 80);
    // send and receive data...
    socket.close();
} catch (UnknownHostException ex) {
    System.err.println(ex);
} catch (IOException ex) {
    System.err.println(ex);
}
```
In this constructor, the ``host`` argument is just a hostname expressed as a ``String``. If the domain name server cannot resolve the hostname or is not functioning, the constructor throws an ``UnknownHostException``. If the socket cannot be opened for some other reason, the constructor throws an ``IOException``.

Three constructors create unconnected sockets. These provide more control over exactly how the underlying socket behaves, for instance by choosing a different proxy server or an encryption scheme:

```Java
public Socket()
public Socket(Proxy proxy)
protected Socket(SocketImpl impl)
```

### **Picking a Local Interface to Connect From**

Two constructors specify both the ``host`` and ``port`` to connect to and the interface and ``port`` to connect from:

```Java
public Socket(String host, int port, InetAddress interface, int localPort) throws IOException, UnknownHostException
public Socket(InetAddress host, int port, InetAddress interface, int localPort) throws IOException
```

This socket connects to the ``host`` and ``port`` specified in the first two arguments. It connects from the ``local network interface`` and ``port`` specified by the last two arguments. The network interface may be either physical (e.g., an Ethernet card) or virtual (a multihomed host with more than one IP address). If ``0`` is passed for the ``localPort`` argument, Java chooses a random available port between ``1024`` and ``65535``.

**Example :**
```Java
try {
    InetAddress inward = InetAddress.getByName("router");
    Socket socket = new Socket("mail", 25, inward, 0);
    // work with the sockets...
    socket.close();
} catch (IOException ex) {
    System.err.println(ex);
}
```

By passing ``0`` for the local port number, I say that I don’t care which port is used but I do want to use the network interface bound to the local hostname ``router``.

### **Constructing Without Connecting**

All the constructors we’ve talked about so far both create the socket object and open a network connection to a remote host. Sometimes you want to split those operations. If you give no arguments to the ``Socket`` constructor, it has nowhere to connect to:

```Java
public Socket()
```

You can connect later by passing a ``SocketAddress`` to one of the **``connect()``** methods

```Java
try {
    Socket socket = new Socket();
    // fill in socket options
    SocketAddress address = new InetSocketAddress("time.nist.gov", 13);
    socket.connect(address);
    // work with the sockets...
    socket.close();
} catch (IOException ex) {
    System.err.println(ex);
}
```

You can pass an ``int`` as the second argument to specify the number of ``milliseconds`` to wait before the connection times out:

```Java
public void connect(SocketAddress endpoint, int timeout) throws IOException
```

The default, ``0``, means wait forever.

### **Socket Addresses**

The ``SocketAddress`` class represents a connection endpoint. It is an empty abstract class with no methods aside from a default constructor. At least theoretically, the ``SocketAddress`` class can be used for both ``TCP`` and ``non-TCP`` sockets. In practice, only ``TCP/IP`` sockets are currently supported and the socket addresses you actually use are all instances of ``InetSocketAddress``.

The primary purpose of the ``SocketAddress`` class is to provide a convenient store for transient socket connection information such as the IP address and port that can be reused to create new sockets, even after the original socket is disconnected and garbage collected. To this end, the Socket class offers two methods that return ``SocketAddress`` objects (**``getRemoteSocketAddress()``** returns the address of the system being connected to and **``getLocalSocketAddress()``** returns the address from which the connection is made):

```Java
public SocketAddress getRemoteSocketAddress()
public SocketAddress getLocalSocketAddress()
```

Both of these methods return ``null`` if the socket is not yet connected. For example, first you might connect to ``Google`` then store its address:

```Java
Socket socket = new Socket("www.google.com", 80);
SocketAddress google = socket.getRemoteSocketAddress();
socket.close();
```
Later, you could reconnect to ``Google`` using this address:

```Java
Socket socket2 = new Socket();
socket2.connect(yahoo);
```

The ``InetSocketAddress`` class is usually created with a ``host`` and a ``port`` (for clients) or just a ``port`` (for servers):

```Java
public InetSocketAddress(InetAddress address, int port)
public InetSocketAddress(String host, int port)
public InetSocketAddress(int port)
```

You can also use the static factory method ``InetSocketAddress.createUnresolved()`` to skip looking up the host in ``DNS``:

```Java
public static InetSocketAddress createUnresolved(String host, int port)
```

``InetSocketAddress`` has a few getter methods you can use to inspect the object:

```Java
public final InetAddress getAddress()
public final int getPort()
public final String getHostName()
```

### **Proxy Servers**

The last constructor creates an unconnected socket that connects through a specified proxy server:

```Java
public Socket(Proxy proxy)
```
Normally, the proxy server a socket uses is controlled by the ``socksProxyHost`` and ``socksProxyPort`` system properties, and these properties apply to all sockets in the system. However, a socket created by this constructor will use the specified proxy server instead. Most notably, you can pass ``Proxy.NO_PROXY`` for the argument to bypass all proxy servers completely and connect directly to the remote host. Of course, if a firewall prevents direct connections, there’s nothing Java can do about it; and the connection will fail.

To use a particular proxy server, specify it by address. For example, this code fragment uses the ``SOCKS`` proxy server at ``myproxy.example.com`` to connect to the host ``login.ibiblio.org``:

```Java
SocketAddress proxyAddress = new InetSocketAddress("myproxy.example.com", 1080);
Proxy proxy = new Proxy(Proxy.Type.SOCKS, proxyAddress)
Socket s = new Socket(proxy);
SocketAddress remote = new InetSocketAddress("login.ibiblio.org", 25);
s.connect(remote);
```
``SOCKS`` is the only low-level proxy type Java understands. There’s also a high-level ``Proxy.Type.HTTP`` that works in the application layer rather than the transport layer and a ``Proxy.Type.DIRECT`` that represents proxyless connections.

## Getting Information About a Socket
-------------------------------------
``Socket`` objects have several properties that are accessible through ``getter`` methods:

- Remote address
- Remote port
- Local address
- Local port

Here are the ``getter`` methods for accessing these properties:

```Java
public InetAddress getInetAddress()
public int getPort()
public InetAddress getLocalAddress()
public int getLocalPort()
```

There are no ``setter`` methods. These properties are set as soon as the socket connects, and are fixed from there on.

The **``getInetAddress()``** and **``getPort()``** methods tell you the remote host and port the ``Socket`` is connected to; or, if the connection is now closed, which host and port the ``Socket`` was connected to when it was connected. The **``getLocalAddress()``** and **``getLocalPort()``** methods tell you the network interface and port the ``Socket`` is connected from.

**Program:**
```Java
import java.io.IOException;
import java.net.Socket;

public class SocketInfo {
    public static final String SERVER = "time.nist.gov";
    public static final int PORT = 13;

    public static void main(String[] args) {
        try (Socket socket = new Socket(SERVER, PORT)) {
            System.out.println("Socket Info :");
            System.out.println("==============================");
            System.out.println("Connected to : " + socket.getInetAddress());
            System.out.println("Port : "+socket.getPort());
            System.out.println("------------------------------");
            System.out.println("From Port : "+socket.getLocalPort());
            System.out.println("Of : "+socket.getLocalAddress());
            System.out.println("==============================");

        } catch (IOException ex) {
            System.err.println("Could not connect to : "+ SERVER);
        }
    }
}
```

**Output:**
```
Socket Info :
==============================
Connected to : time.nist.gov/132.163.97.4
Port : 13
------------------------------
From Port : 38712
Of : /192.168.1.19
==============================
```

### **Closed or Connected?**

The **``isClosed()``** method returns ``true`` if the socket is closed, ``false`` if it isn’t. If you’re uncertain about a socket’s state, you can check it with this method rather than risking an ``IOException``. For example:

```Java
if (socket.isClosed()) {
    // do something...
} else {
    // do something else...
}
```
However, this is not a perfect test. If the socket has never been connected in the first place, **``isClosed()``** returns ``false``, even though the socket isn’t exactly open.

The ``Socket`` class also has an **``isConnected()``** method. The name is a little misleading. It does not tell you if the socket is currently connected to a remote host (like if it is unclosed). Instead, it tells you whether the socket has ever been connected to a remote host. If the socket was able to connect to the remote host at all, this method returns ``true``, even after that socket has been closed. To tell if a socket is currently open, you need to check that **``isConnected()``** returns ``true`` and **``isClosed()``** returns ``false``. For example:

```Java
boolean connected = socket.isConnected() && ! socket.isClosed();
```

Finally, the **``isBound()``** method tells you whether the socket successfully bound to the outgoing port on the local system. Whereas **``isConnected()``** refers to the remote end of the socket, **``isBound()``** refers to the local end. 

### **toString()**

The ``Socket`` class overrides only one of the standard methods from ``java.lang.Object:toString()``. The **``toString()``** method produces a string that looks like this:

```
Socket[addr=www.example.com/198.112.208.11,port=80,localport=50055]
```

This is useful primarily for debugging. 

## Setting Socket Options
-------------------------
``Socket`` options specify how the native sockets on which the Java Socket class relies send and receive data. Java supports nine options for client-side sockets:

- ``TCP_NODELAY``
- ``SO_BINDADDR``
- ``SO_TIMEOUT``
- ``SO_LINGER``
- ``SO_SNDBUF``
- ``SO_RCVBUF``
- ``SO_KEEPALIVE``
- ``OOBINLINE``
- ``IP_TOS``

### **``TCP_NODELAY``**

Setting ``TCP_NODELAY`` to ``true`` ensures that packets are sent as quickly as possible regardless of their size. Normally, small (one-byte) packets are combined into larger packets before being sent. Before sending another packet, the local host waits to receive acknowledgment of the previous packet from the remote system. This is known as **Nagle’s algorithm**. The problem with **Nagle’s algorithm** is that if the remote system doesn’t send acknowledgments back to the local system fast enough, applications that depend on the steady transfer of small parcels of information may slow down. On a really slow network, even simple typing can be too slow because of the constant buffering. Setting ``TCP_NODELAY`` to ``true`` defeats this buffering scheme, so that all packets are sent as soon as they’re ready.

```Java
public void setTcpNoDelay(boolean on) throws SocketException
public boolean getTcpNoDelay() throws SocketException
```

**``setTcpNoDelay(true)``** turns off buffering for the socket. **``setTcpNoDelay(false)``** turns it back on. **``getTcpNoDelay()``** returns ``true`` if buffering is ``off`` and ``false`` if buffering is ``on``. For example, the following fragment turns off buffering (that is, it turns on ``TCP_NODELAY``) for the socket ``s`` if it isn’t already off:

```Java
if (!s.getTcpNoDelay()){
    s.setTcpNoDelay(true);
}
```

These two methods are each declared to throw a ``SocketException``, which will happen if the underlying socket implementation doesn’t support the ``TCP_ NODELAY`` option.

### **``SO_LINGER``**

The ``SO_LINGER`` option specifies what to do with datagrams that have not yet been sent when a socket is closed. By default, the **``close()``** method returns immediately; but the system still tries to send any remaining data. If the linger time is set to ``zero``, any unsent packets are thrown away when the socket is closed. If ``SO_LINGER`` is turned ``on`` and the linger time is any positive value, the **``close()``** method blocks while waiting the specified number of seconds for the data to be sent and the acknowledgments to be received. When that number of seconds has passed, the socket is closed and any remaining data is not sent, acknowledgment or no.

```Java
public void setSoLinger(boolean on, int seconds) throws SocketException
public int getSoLinger() throws SocketException
```

These two methods each throw a ``SocketException`` if the underlying socket implementation does not support the ``SO_LINGER`` option. The **``setSoLinger()``** method can also throw an ``IllegalArgumentException`` if you try to set the linger time to a ``negative`` value. However, the **``getSoLinger()``** method may return ``–1`` to indicate that this option is ``disabled``, and as much time as is needed is taken to deliver the remaining data; for example, to set the linger timeout for the Socket ``s`` to four minutes, if it’s not already set to some other value:

```Java
if (s.getTcpSoLinger() == -1){
    s.setSoLinger(true, 240);
}
```

The maximum linger time is ``65,535`` seconds, and may be smaller on some platforms.Times larger than that will be reduced to the maximum linger time. Frankly, ``65,535`` seconds (more than ``18`` hours) is much longer than you actually want to wait. Generally, the platform default value is more appropriate.

### **``SO_TIMEOUT``**

```Java
public void setSoTimeout(int milliseconds) throws SocketException
public int getSoTimeout() throws SocketException
```

Normally when you try to read data from a socket, the **``read()``** call blocks as long as necessary to get enough bytes. By setting ``SO_TIMEOUT``, you ensure that the call will not block for more than a fixed number of milliseconds. When the timeout expires, an ``InterruptedIOException`` is thrown, and you should be prepared to catch it. However, the socket is still connected. Although this **``read()``** call failed, you can try to read from the socket again. The next call may succeed.

Timeouts are given in milliseconds. ``Zero`` is interpreted as an infinite timeout; it is the default value. For example, to set the timeout value of the Socket object ``s`` to ``3`` minutes if it isn’t already set, specify ``180,000`` milliseconds:

```Java
if (s.getSoTimeout() == 0){
    s.setSoTimeout(180000);
}
```

These two methods each throw a ``SocketException`` if the underlying socket implementation does not support the ``SO_TIMEOUT`` option. The ``setSoTimeout()`` method also throws an ``IllegalArgumentException`` if the specified timeout value is ``negative``.

### **SO_RCVBUF and SO_SNDBUF**

``TCP`` uses buffers to improve network performance. Larger buffers tend to improve performance for reasonably fast (say, 10Mbps and up) connections whereas slower, dialup connections do better with smaller buffers. Generally, transfers of large, continuous blocks of data, which are common in file transfer protocols such as ``FTP`` and ``HTTP``, benefit from large buffers, whereas the smaller transfers of interactive sessions, such as Telnet and many games, do not. Relatively old operating systems designed in the age of small files and slow networks, such as ``BSD 4.2``, use two-kilobyte buffers. Windows XP used 17,520 byte buffers. These days, 128 kilobytes is a common default.

The ``SO_RCVBUF`` option controls the suggested send buffer size used for network input. The ``SO_SNDBUF`` option controls the suggested send buffer size used for network output:

```Java
public void setReceiveBufferSize(int size) throws SocketException, IllegalArgumentException
public int getReceiveBufferSize() throws SocketException
public void setSendBufferSize(int size) throws SocketException, IllegalArgumentException
public int getSendBufferSize() throws SocketException
```

Although it looks like you should be able to set the send and receive buffers independently, the buffer is usually set to the smaller of these two. For instance, if you set the send buffer to ``64K`` and the receive buffer to ``128K``, you’ll have ``64K`` as both the send and receive buffer size. Java will report that the receive buffer is ``128K``, but the underlying TCP stack will really be using ``64K``.

The **``setReceiveBufferSize()``/``setSendBufferSize``** methods suggest a number of bytes to use for buffering output on this socket. However, the underlying implementation is free to ignore or adjust this suggestion. In particular, Unix and Linux systems often specify a maximum buffer size, typically ``64K`` or ``256K``, and do not allow any socket to have a larger one. If you attempt to set a larger value, Java will just pin it to the maximum possible buffer size. On Linux, it’s not unheard of for the underlying implementation to double the requested size. For example, if you ask for a ``64K`` buffer, you may get a ``128K`` buffer instead.

These methods throw an ``IllegalArgumentException`` if the argument is less than or equal to ``zero``. Although they’re also declared to throw ``SocketException``, they probably won’t in practice, because a ``SocketException`` is thrown for the same reason as ``IllegalArgumentException`` and the check for the ``IllegalArgumentException`` is made first.

### **SO_KEEPALIVE**

If ``SO_KEEPALIVE`` is turned ``on``, the client occasionally sends a data packet over an idle connection (most commonly once every two hours), just to make sure the server hasn’t crashed. If the server fails to respond to this packet, the client keeps trying for a little more than 11 minutes until it receives a response. If it doesn’t receive a response within 12 minutes, the client closes the socket. Without ``SO_KEEPALIVE``, an inactive client could live more or less forever without noticing that the server had crashed. These methods turn ``SO_KEEPALIVE`` on and off and determine its current state:

```Java
public void setKeepAlive(boolean on) throws SocketException
public boolean getKeepAlive() throws SocketException
```

The default for ``SO_KEEPALIVE`` is ``false``. This code fragment turns ``SO_KEEPALIVE`` off, if it’s turned ``on``:

```Java
if (s.getKeepAlive()){
    s.setKeepAlive(false);
}
```

### **OOBINLINE**

``TCP`` includes a feature that sends a single byte of “urgent” data out of band. This data is sent immediately. Furthermore, the receiver is notified when the urgent data is received and may elect to process the urgent data before it processes any other data that has already been received. Java supports both sending and receiving such urgent data.
The sending method is named, obviously enough, **``sendUrgentData()``**:

```Java
public void sendUrgentData(int data) throws IOException
```

This method sends the ``lowest-order byte`` of its argument almost immediately. If necessary, any currently cached data is flushed first.

How the receiving end responds to urgent data is a little confused, and varies from one platform and ``API`` to the next. Some systems receive the urgent data separately from the regular data. However, the more common, more modern approach is to place the urgent data in the regular received data queue in its proper order, tell the application that urgent data is available, and let it hunt through the queue to find it.

By default, Java ignores urgent data received from a socket. However, if you want to receive urgent data inline with regular data, you need to set the ``OOBINLINE`` option to ``true`` using these methods:

```Java
public void setOOBInline(boolean on) throws SocketException
public boolean getOOBInline() throws SocketException
```

The default for ``OOBINLINE`` is ``false``. This code fragment turns ``OOBINLINE`` on, if it’s turned off:

```Java
if (!s.getOOBInline()){
    s.setOOBInline(true);
}
```

### **SO_REUSEADDR**

When a socket is closed, it may not immediately release the ``local port``, especially if a connection was open when the socket was closed. It can sometimes wait for a small amount of time to make sure it receives any lingering packets that were addressed to the port that were still crossing the network when the socket was closed. The system won’t do anything with any of the late packets it receives. It just wants to make sure they don’t accidentally get fed into a new process that has bound to the same port.

This isn’t a big problem on a random port, but it can be an issue if the socket has bound to a well-known port because it prevents any other socket from using that port in the meantime. If the ``SO_REUSEADDR`` is turned ``on`` (it’s turned off by default), another socket is allowed to bind to the port even while data may be outstanding for the previous socket.

In Java this option is controlled by these two methods:

```Java
public void setReuseAddress(boolean on) throws SocketException
public boolean getReuseAddress() throws SocketException
```
For this to work, **``setReuseAddress()``** must be called before the new socket binds to the port. This means the socket must be created in an unconnected state using the noargs constructor; then **``setReuseAddress(true)``** is called, and the socket is connected using the **``connect()``** method. Both the socket that was previously connected and the new socket reusing the old address must set ``SO_REUSEADDR`` to ``true`` for it to take effect.

### **IP_TOS Class of Service**

Different types of Internet service have different performance needs. For instance, video chat needs relatively high bandwidth and low latency for good performance, whereas email can be passed over low-bandwidth connections and even held up for several hours without major harm. ``VOIP`` needs less bandwidth than video but minimum jitter. It might be wise to price the different classes of service differentially so that people won’t ask for the highest class of service automatically. After all, if sending an overnight letter cost the same as sending a package via media mail, we’d all just use FedEx overnight, which would quickly become congested and overwhelmed. The Internet is no different.

The class of service is stored in an eight-bit field called ``IP_TOS`` in the IP header. Java lets you inspect and set the value a socket places in this field using these two methods:

```Java
public int getTrafficClass() throws SocketException
public void setTrafficClass(int trafficClass) throws SocketException
```
The traffic class is given as an int between ``0`` and ``255``. Because this value is copied to an eight-bit field in the ``TCP`` header, only the low order byte of this int is used; and values outside this range cause ``IllegalArgumentExceptions``.

## Socket Exceptions
--------------------
Most methods of the ``Socket`` class are declared to throw ``IOException`` or its subclass, ``java.net.SocketException``:

```Java
public class SocketException extends IOException
```

However, knowing that a problem occurred is often not sufficient to deal with the problem. Did the remote host refuse the connection because it was busy? Did the remote host refuse the connection because no service was listening on the port? Did the connection attempt timeout because of network congestion or because the host was down? There are several subclasses of ``SocketException`` that provide more information about what went wrong and why:

```Java
public class BindException extends SocketException
```
```Java
public class ConnectException extends SocketException
```
```Java
public class NoRouteToHostException extends SocketException
```

A ``BindException`` is thrown if you try to construct a ``Socket`` or ``ServerSocket`` object on a local port that is in use or that you do not have sufficient privileges to use. A ``ConnectException`` is thrown when a connection is refused at the remote host, which usually happens because the host is busy or no process is listening on that port. Finally, a ``NoRouteToHostException`` indicates that the connection has timed out.

The ``java.net`` package also includes ProtocolException, which is a direct subclass of ``IOException``:

```Java
public class ProtocolException extends IOException
```

This is thrown when data is received from the network that somehow violates the ``TCP/IP`` specification.

None of these exception classes have any special methods you wouldn’t find in any other exception class, but you can take advantage of these subclasses to provide more informative error messages or to decide whether retrying the offending operation is likely to be successful.