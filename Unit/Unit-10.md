<div align="center">

[**_``Go Back``_**](../README.md)

# UDP

</div>

In computer networking, the ``User Datagram Protocol (UDP)`` is one of the core communication protocols of the Internet protocol suite used to send messages (transported as datagrams in packets) to other hosts on an ``Internet Protocol (IP)`` network. Within an IP network, UDP does not require prior communication to set up communication channels or data paths.

## UDP Protocol
----------------

``User Datagram Protocol (UDP)`` is a **Transport Layer protocol**. ``UDP`` is a part of the **Internet Protocol** suite, referred to as ``UDP/IP`` suite. Unlike ``TCP``, it is an unreliable and connectionless protocol. So, there is no need to establish a connection prior to data transfer. The ``UDP`` helps to establish **low-latency** and **loss-tolerating** connections establish over the network.The ``UDP`` enables process to process communication.

Though ``Transmission Control Protocol (TCP)`` is the dominant **transport layer protocol** used with most of the Internet services; provides assured delivery, reliability, and much more but all these services cost us additional overhead and latency. Here, ``UDP`` comes into the picture. For real-time services like computer gaming, voice or video communication, live conferences; we need ``UDP``. Since high performance is needed, ``UDP`` permits packets to be dropped instead of processing delayed packets. There is no error checking in ``UDP``, so it also saves bandwidth. ``User Datagram Protocol (UDP)`` is more efficient in terms of both latency and bandwidth. 

## UDP Servers
--------------
A ``UDP`` server is a software application or component that uses the ``User Datagram Protocol (UDP)`` to receive and process incoming **datagrams (small packets of data)** from ``UDP`` clients. 

**Example:** let’s implement a daytime server over ``UDP`` at port ``8080``:

```Java
DatagramSocket socket = new DatagramSocket(8080);
```

Next, create a packet into which to receive a request. You need to supply both a byte array in which to store incoming data, the offset into the array, and the number of bytes to store. Here you set up a packet with space for 1,024 bytes starting at ``0``:

```Java
DatagramPacket request = new DatagramPacket(new byte[1024], 0, 1024);
```

Then receive it:

```Java
socket.receive(request);
```
This call blocks indefinitely until a ``UDP`` packet arrives on port 8080. When it does, Java fills the byte array with data and the **``receive()``** method returns.

Next, create a response packet. This has four parts: 

- the **raw** data to send, 
- the **number of bytes** of the **raw** data to send,
- the **host** to send to,
- and the **port** on that host to address.

In this example, the raw data comes from a String form of the current time, and the host and the port are simply the host and port of the incoming packet:

```Java
String daytime = new Date().toString() + "\r\n";
byte[] data = daytime.getBytes("US-ASCII");
InetAddress host = request.getAddress();
int port = request.getPort();
DatagramPacket response = new DatagramPacket(data, data.length, host, port);
```

Finally, send the response back over the same socket that received it:

```Java
socket.send(response);
```
Lets puts this all together.

**Program:**
```Java
import java.net.*;
import java.util.Date;
import java.io.*;

public class DaytimeUDPServer {
    private final static int PORT = 8080;

    public static void main(String[] args) {
        try (DatagramSocket socket = new DatagramSocket(PORT)) {
            while (true) {
                try {
                    DatagramPacket request = new DatagramPacket(new byte[1024], 1024);
                    socket.receive(request);
                    String daytime = new Date().toString();
                    byte[] data = daytime.getBytes("US-ASCII");
                    DatagramPacket response = new DatagramPacket(data, data.length,
                            request.getAddress(), request.getPort());
                    socket.send(response);
                    System.out.println(daytime + " " + request.getAddress());
                } catch (IOException | RuntimeException ex) {
                    ex.printStackTrace();
                }
            }
        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }
}
```

## UDP Clients
---------------

A ``UDP`` client is a software application or component that uses the ``User Datagram Protocol (UDP)`` to establish communication with a ``UDP`` server. ``UDP`` is a **transport protocol** that operates on top of ``IP (Internet Protocol)`` and provides a simple way to send and receive datagrams (small packets of data) over a network. Unlike ``TCP (Transmission Control Protocol)``, ``UDP`` is connectionless and does not provide reliable delivery or guaranteed order of data packets. This makes ``UDP`` suitable for applications where **low latency** and **speed** are more important than guaranteed delivery.

**Example:** Lets build client for above server.

Now let’s see how to retrieve this same data programmatically using ``UDP``. First, open a socket on port ``0``:

```Java
DatagramSocket socket = new DatagramSocket(0);
```

This is very different than a ``TCP`` socket. You only specify a local port to connect to. The socket does not know the remote host or address. By specifying port ``0`` you ask Java to pick a random available port for you, much as with server sockets.

The next step is optional but highly recommended. Set a timeout on the connection using the **``setSoTimeout()``** method. Timeouts are measured in milliseconds, so this statement sets the socket to time out after ``10`` seconds of nonresponsiveness:

```Java
socket.setSoTimeout(10000);
```

**Timeouts** are even more important for ``UDP`` than ``TCP`` because many problems that would cause an ``IOException`` in ``TCP`` silently fail in ``UDP``. For example, if the remote host is not listening on the targeted port, you’ll never hear about it.

Next you need to set up the packets. You’ll need two, one to send and one to receive. For the daytime protocol it doesn’t matter what data you put in the packet, but you do need to tell it the **remote host** and **remote port** to connect to:

```Java
InetAddress host = InetAddress.getByName("localhost");
DatagramPacket request = new DatagramPacket(new byte[1], 1, host, 8080);
```
The packet that receives the server’s response just contains an empty ``byte`` array. This needs to be large enough to hold the entire response. If it’s too small, it will be silently truncated ``1k`` should be enough space:

```Java
byte[] data = new byte[1024];
DatagramPacket response = new DatagramPacket(data, data.length);
```

Now you’re ready. First send the packet over the socket and then receive the response:

```Java
socket.send(request);
socket.receive(response);
```
Finally, extract the ``bytes`` from the **response** and convert them to a ``string`` you can show to the end user:

```Java
String daytime = new String(response.getData(), 0, response.getLength(), "US-ASCII");
System.out.println(daytime);
```
The constructor and **``send()``** and **``receive()``** methods can each throw an ``IOException``, so you’ll usually wrap all this in a ``try`` block. In **Java 7**, ``DatagramSocket`` implements ``Autocloseable`` so you can use **try-with-resources**:

```Java
try (DatagramSocket socket = new DatagramSocket(0)) {
    // connect to the server...
} catch (IOException ex) {
    System.err.println("Could not connect to time.nist.gov");
}
```
In **Java 6** and earlier, you’ll want to explicitly close the socket in a ``finally`` block to release resources the socket holds:

```Java
DatagramSocket socket = null;
try {
    socket = new DatagramSocket(0);
    // connect to the server...
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
Lets puts this all together.

**Program:**

```Java
import java.io.*;
import java.net.*;

public class DaytimeUDPClient {
    private static final String HOSTNAME = "localhost";
    private final static int PORT = 8080;

    public static void main(String[] args) {
        try (DatagramSocket socket = new DatagramSocket(0)) {
            socket.setSoTimeout(10000);
            InetAddress host = InetAddress.getByName(HOSTNAME);
            DatagramPacket request = new DatagramPacket(new byte[1], 1, host, PORT);
            DatagramPacket response = new DatagramPacket(new byte[1024], 1024);
            socket.send(request);
            socket.receive(response);
            String result = new String(response.getData(), 0, response.getLength(), "US-ASCII");
            System.out.println(result);
        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }
}
```
## The DatagramPacket Class
----------------------------

A **UDP datagram**, also known as a **UDP packet**, is a basic unit of data used in the ``User Datagram Protocol (UDP)``, which is one of the transport protocols in computer networking. ``UDP`` datagrams are used to transmit information between devices over a network without establishing a formal connection. Unlike ``TCP (Transmission Control Protocol)``, which provides reliable, connection-oriented communication, ``UDP`` offers a simpler, connectionless approach that is faster but lacks certain reliability features.

> A ``Datagram`` is a basic unit of data transmission in networking, and it is used in connectionless communication protocols like UDP (User Datagram Protocol). 

The structure of a ``Datagram`` can be broken down into several components:

- **``Source Port``**: A ``16-bit`` field that represents the port number of the sender.

- **``Destination Port``**: A ``16-bit`` field that represents the port number of the receiver.

- **``Length``**: A ``16-bit`` field indicating the total length of the ``Datagram (header + data)`` in bytes.

- **``Checksum``**: A ``16-bit`` field used for error checking, which helps ensure the integrity of the Datagram during transmission.

- **``Data``**: The actual payload or data being transmitted. This can vary in size and content depending on the application.

The structure of a ``UDP Datagram`` is relatively simple compared to more complex protocols like ``TCP``. Here's a graphical representation of a UDP Datagram:

```
+-------------------+-------------------+
| Source Port       | Destination Port  |
+-------------------+-------------------+
| Length            | Checksum          |
+-------------------+-------------------+
| Data (Payload)                        |
| ...                                   |
+---------------------------------------+
```

In Java, a **UDP datagram** is represented by an instance of the ``DatagramPacket`` class:

```Java
public final class DatagramPacket extends Object
```

This class provides methods to **get** and **set** the ``source`` or ``destination address`` from the IP header, to **get** and **set** the ``source`` or ``destination port``, to **get** and **set** the ``data``, and to **get** and **set** the ``length of the data``. The remaining header fields are inaccessible from pure Java code.

### **The Constructors**

``DatagramPacket`` uses different constructors depending on whether the packet will be used to **send data** or to **receive data**.

#### Constructors for receiving datagrams

These two constructors create new **DatagramPacket** objects for **receiving data** from the network:

```Java
public DatagramPacket(byte[] buffer, int length)
```
```Java
public DatagramPacket(byte[] buffer, int offset, int length)
```

If the first constructor is used, when a socket receives a datagram, it stores the datagram’s data part in buffer beginning at ``buffer[0]`` and continuing until the packet is completely stored or until length bytes have been written into the buffer. For example, this code fragment creates a new ``DatagramPacket`` for receiving a datagram of up to ``8,192`` bytes:

```Java
byte[] buffer = new byte[8192];
DatagramPacket dp = new DatagramPacket(buffer, buffer.length);
```
If the second constructor is used, storage begins at ``buffer[offset]`` instead. Otherwise, these two constructors are identical. length must be less than or equal to ``buffer.length - offset``. If you try to construct a ``DatagramPacket`` with a length that will overflow the buffer, the constructor throws an ``IllegalArgumentException``. This is a ``RuntimeException``, so your code is not required to catch it. It is OK to construct a ``DatagramPacket`` with a length less than ``buffer.length - offset``. In this case, at most the first length bytes of buffer will be filled when the datagram is received.

#### Constructors for sending datagrams

These four constructors create new ``DatagramPacket`` objects used to send data across the network:

```Java
public DatagramPacket(byte[] data, int length, InetAddress destination, int port)
```
```Java
public DatagramPacket(byte[] data, int offset, int length, InetAddress destination, int port)
```
```Java
public DatagramPacket(byte[] data, int length, SocketAddress destination)
```
```Java
public DatagramPacket(byte[] data, int offset, int length, SocketAddress destination)
```

Each constructor creates a new ``DatagramPacket`` to be sent to another host. The packet is filled with length bytes of the data array starting at offset or 0 if offset is not used. If you try to construct a ``DatagramPacket`` with a length that is greater than **data.length** (or greater than ``data.length - offset``), the constructor throws an ``IllegalArgumentException``. It’s OK to construct a DatagramPacket object with an offset and a length that will leave extra, unused space at the end of the data array. In this case, only length bytes of data will be sent over the network. The ``InetAddress`` or ``SocketAddress`` object destination points to the host you want the packet delivered to; the int argument port is the port on that host.

For instance, this code fragment creates a new ``DatagramPacket`` filled with the data **“This is a test”** in ``UTF-8``. The packet is directed at port ``7`` of ``www.example.org``:

```Java
String s = "This is a test";
byte[] data = s.getBytes("UTF-8");
try {
    InetAddress address = InetAddress.getByName("www.example.com");
    int port = 7;
    DatagramPacket dp = new DatagramPacket(data, data.length, address, port);
    // send the packet...
} catch (IOException ex){
    ex.printStackTrace();
}
```

Most of the time, the hardest part of creating a new ``DatagramPacket`` is translating the data into a byte array. Because this code fragment wants to send a string, it uses the **``getBytes()``** method of ``java.lang.String``. The ``java.io.ByteArrayOutputStream`` class can also be very useful for preparing data for inclusion in datagrams.

### **The get Methods**

``DatagramPacket`` has six methods that retrieve different parts of a datagram: the actual data plus several fields from its header. These methods are mostly used for datagrams received from the network.

- **``getAddress()``**
- **``getPort()``**
- **``getSocketAddress()``**
- **``getData()``**
- **``getLength()``**
- **``getOffset()``**

#### getAddress()
```Java
public InetAddress getAddress()
```
The **``getAddress()``** method returns an InetAddress object containing the address of the remote host.

#### getPort()
```Java
public int getPort()
```
The **``getPort()``** method returns an integer specifying the remote port.

#### getSocketAddress()
```Java
public SocketAddress getSocketAddress()
```
The **``getSocketAddress()``** method returns a ``SocketAddress`` object containing the **IP address** and **port** of the remote host.

#### getData()
```Java
public byte[] getData()
```
The **``getData()``** method returns a **byte array** containing the data from the datagram. It’s often necessary to convert the bytes into some other form of data before they’ll be useful to your program. One way to do this is to change the byte array into a ``String``. For example, given a ``DatagramPacket dp`` received from the network, you can convert it to a ``UTF-8`` String like this:

```Java
String s = new String(dp.getData(), "UTF-8");
```
If the datagram does not contain text, converting it to Java data is more difficult. One approach is to convert the byte array returned by **``getData()``** into a ``ByteArrayInputStream``. For example:

```Java
InputStream in = new ByteArrayInputStream(packet.getData(), packet.getOffset(), packet.getLength());
```
You must specify the offset and the length when constructing the **``ByteArrayInputStream``**. Do not use the **``ByteArrayInputStream()``** constructor that takes only an array as an argument. The array returned by ``packet.getData()`` probably has extra space in it that was not filled with data from the network. This space will contain whatever random values those components of the array had when the ``DatagramPacket`` was constructed.

The ``ByteArrayInputStream`` can then be chained to a ``DataInputStream``:

```Java
DataInputStream din = new DataInputStream(in);
```
The data can then be read using the ``DataInputStream``’s **``readInt()``**, **``readLong()``**, **``readChar()``**, and other methods.

#### getLength()
```Java
public int getLength()
```
The **``getLength()``** method returns the number of bytes of data in the datagram.

#### getOffset()
```Java
public int getOffset()
```
This method simply returns the point in the array returned by **``getData()``** where the data from the datagram begins.

**Example:**
```Java
import java.io.*;
import java.net.*;
public class DatagramExample {
    public static void main(String[] args) {
        String s = "This is a test.";
        try {
            byte[] data = s.getBytes("UTF-8");
            InetAddress ia = InetAddress.getByName("www.example.com");
            int port = 7;
            DatagramPacket dp = new DatagramPacket(data, data.length, ia, port);
            System.out.println("This packet is addressed to " + dp.getAddress() + " on port " + dp.getPort());
            System.out.println("There are " + dp.getLength() + " bytes of data in the packet");
            System.out.println(new String(dp.getData(), dp.getOffset(), dp.getLength(), "UTF-8"));
        } catch (UnknownHostException | UnsupportedEncodingException ex) {
            System.err.println(ex);
        }
    }
}
```
**Output:**
```
This packet is addressed to www.example.com/93.184.216.34 on port 7
There are 15 bytes of data in the packet
This is a test.
```

### **The setter Methods**

Most of the time, the six constructors are sufficient for creating datagrams. However, Java also provides several methods for changing the data, remote address, and remote port after the datagram has been created. These methods might be important in a situation where the time to create and garbage collect new ``DatagramPacket`` objects is a significant performance hit.

- **``setData(byte[] data)``**
- **``setData(byte[] data, int offset, int length)``**
- **``setAddress(InetAddress remote)``**
- **``setPort(int port)``**
- **``setAddress(SocketAddress remote)``**
- **``setLength(int length)``**

#### setData(byte[] data)

```Java
public void setData(byte[] data)
```
The **setData()** method changes the payload of the UDP datagram.

#### setData(byte[] data, int offset, int length)

```Java
public void setData(byte[] data, int offset, int length)
```
This overloaded variant of the **``setData()``** method provides an alternative approach to sending a large quantity of data. Instead of sending lots of new arrays, you can put all the data in one array and send it a piece at a time. For instance, this loop sends a large array in ``512-byte`` chunks:

```Java
int offset = 0;
DatagramPacket dp = new DatagramPacket(bigarray, offset, 512);
int bytesSent = 0;
while (bytesSent < bigarray.length) {
    socket.send(dp);
    bytesSent += dp.getLength();
    int bytesToSend = bigarray.length - bytesSent;
    int size = (bytesToSend > 512) ? 512 : bytesToSend;
    dp.setData(bigarray, bytesSent, size);
}
```
#### setAddress(InetAddress remote)

```Java
public void setAddress(InetAddress remote)
```

The **``setAddress()``** method changes the address a datagram packet is sent to. This might allow you to send the same datagram to many different recipients. For example:

```Java
String s = "Really Important Message";
byte[] data = s.getBytes("UTF-8");
DatagramPacket dp = new DatagramPacket(data, data.length);
dp.setPort(2000);
int network = "128.238.5.";
for (int host = 1; host < 255; host++) {
    try {
        InetAddress remote = InetAddress.getByName(network + host);
        dp.setAddress(remote);
        socket.send(dp);
    } catch (IOException ex) {
        // skip it; continue with the next host
    }
}
```
#### setPort(int port)

```Java
public void setPort(int port)
```

The **``setPort()``** method changes the port a datagram is addressed to.

#### setAddress(SocketAddress remote)

```Java
public void setAddress(SocketAddress remote)
```
The **``setSocketAddress()``** method changes the **address** and **port** a datagram packet is sent to. You can use this when replying. For example, this code fragment receives a datagram packet and responds to the same address with a packet containing the string ``“Hello”``:

```Java
DatagramPacket input = new DatagramPacket(new byte[8192], 8192);
socket.receive(input);
DatagramPacket output = new DatagramPacket("Hello".getBytes("UTF-8"), 11);
SocketAddress address = input.getSocketAddress();
output.setAddress(address);
socket.send(output);
```
#### setLength(int length)

```Java
public void setLength(int length)
```
The **``setLength()``** method changes the number of bytes of data in the internal buffer that are considered to be part of the datagram’s data as opposed to merely unfilled space.

## The DatagramSocket Class
----------------------------
To **send** or **receive** a ``DatagramPacket``, you must open a **datagram socket**. In Java, a **datagram socket** is created and accessed through the ``DatagramSocket`` class:

```Java
public class DatagramSocket extends Object
```

### **The Constructors**

The ``DatagramSocket`` constructors are used in different situations, much like the ``DatagramPacket`` constructors. The first constructor opens a datagram socket on an anonymous local port. The second constructor opens a datagram socket on a well-known local port that listens to all local network interfaces. The last two constructors open a datagram socket on a well-known local port on a specific network interface. All constructors deal only with the local address and port. The remote address and port are stored in the ``DatagramPacket``, not the ``DatagramSocket``. Indeed, one ``DatagramSocket`` can send and receive datagrams from multiple remote hosts and ports.

```Java
public DatagramSocket() throws SocketException
```
This constructor creates a socket that is bound to an anonymous port. For example:

```Java
try {
    DatagramSocket client = new DatagramSocket();
    // send packets...
} catch (SocketException ex) {
    System.err.println(ex);
}
```
The constructor throws a ``SocketException`` if the socket can’t bind to a **port**. 

```Java
public DatagramSocket(int port) throws SocketException
```
This constructor creates a socket that listens for incoming datagrams on a particular port, specified by the port argument. Use this constructor to write a server that listens on a well-known port. A ``SocketException`` is thrown if the socket can’t be created.

```Java
public DatagramSocket(int port, InetAddress interface) throws SocketException
```
This constructor is primarily used on multihomed hosts; it creates a socket that listens for incoming datagrams on a specific port and network interface. The port argument is the port on which this socket listens for datagrams.

```Java
public DatagramSocket(SocketAddress interface) throws SocketException
```
This constructor is similar to the previous one except that the network interface address and port are read from a ``SocketAddress``. For example, this code fragment creates a socket that only listens on the local loopback address:

```Java
SocketAddress address = new InetSocketAddress("127.0.0.1", 9999);
DatagramSocket socket = new DatagramSocket(address);
```
```Java
protected DatagramSocket(DatagramSocketImpl impl) throws SocketException
```
This constructor enables subclasses to provide their own implementation of the UDP protocol, rather than blindly accepting the default. Unlike sockets created by the other four constructors, this socket is not initially bound to a port. Before using it, you have to bind it to a ``SocketAddress`` using the **``bind()``** method:

```Java
public void bind(SocketAddress addr) throws SocketException
```

You can pass ``null`` to this method, binding the socket to any available **address** and **port**.

### **Sending and Receiving Datagrams**

The primary task of the ``DatagramSocket`` class is to **send** and **receive** UDP datagrams. One socket can both **send** and **receive**. Indeed, it can send and receive to and from multiple hosts at the same time.

#### send(DatagramPacket dp)

```Java
public void send(DatagramPacket dp) throws IOException
```
Once a ``DatagramPacket`` is created and a ``DatagramSocket`` is constructed, send the packet by passing it to the socket’s **``send()``** method. For example, if **theSocket** is a ``DatagramSocket`` object and **theOutput** is a ``DatagramPacket`` object, send **theOutput** using **theSocket** like this:

```Java
theSocket.send(theOutput);
```
### receive(DatagramPacket dp)

```Java
public void receive(DatagramPacket dp) throws IOException
```
This method receives a single ``UDP`` datagram from the network and stores it in the preexisting ``DatagramPacket`` object **dp**. Like the **``accept()``** method in the ``ServerSocket`` class, this method blocks the calling thread until a datagram arrives. If your program does anything besides wait for datagrams, you should call **``receive()``** in a separate thread.

### close()

```Java
public void close()
```

Calling a ``DatagramSocket`` object’s **``close()``** method frees the port occupied by that socket. As with streams and ``TCP`` sockets, you’ll want to take care to close the datagram socket in a finally block:

```Java
DatagramSocket server = null
try {
    server = new DatagramSocket();
    // use the socket...
} catch (IOException ex) {
    System.err.println(ex);
} finally {
    try {
        if (server != null) server.close();
    } catch (IOException ex) {
        System.err.println(ex);
    }
}
```

In Java 7, ``DatagramSocket`` implements ``AutoCloseable`` so you can use **try-with-resources**:

```Java
try (DatagramSocket server = new DatagramSocket()) {
    // use the socket...
}
```

#### getLocalPort()

```Java
public int getLocalPort()
```
A ``DatagramSocket``’s **``getLocalPort()``** method returns an ``int`` that represents the **local port** on which the socket is listening. Use this method if you created a ``DatagramSocket`` with an **anonymous port** and want to find out what **port** the socket has been assigned. For example:

```Java
DatagramSocket ds = new DatagramSocket();
System.out.println("The socket is using port " + ds.getLocalPort());
```

#### getLocalAddress()

```Java
public InetAddress getLocalAddress()
```
A ``DatagramSocket``’s **``getLocalAddress()``** method returns an ``InetAddress`` object that represents the local address to which the socket is bound. It’s rarely needed in practice. Normally, you either already know or simply don’t care which address a socket is listening to.

#### getLocalSocketAddress()

```Java
public SocketAddress getLocalSocketAddress()
```
The **``getLocalSocketAddress()``** method returns a ``SocketAddress`` object that wraps the **local interface** and **port** to which the socket is bound. Like **``getLocalAddress()``**, it’s a little hard to imagine a realistic use case here. This method probably exists mostly for parallelism with **``setLocalSocketAddress()``**.

### **Managing Connections**

There are five methods for managing the connection.

- **``connect(InetAddress host, int port)``**
- **``disconnect()``**
- **``getPort()``**
- **``getInetAddress()``**
- **``getRemoteSocketAddress()``**

#### connect(InetAddress host, int port)

```Java
public void connect(InetAddress host, int port)
```
The **``connect()``** method doesn’t really establish a connection in the ``TCP`` sense. However, it does specify that the ``DatagramSocket`` will only send packets to and receive packets from the specified remote host on the specified remote port. Attempts to send packets to a different host or port will throw an ``IllegalArgumentException``. Packets received from a different host or a different port will be discarded without an exception or other notification.

#### disconnect()

```Java
public void disconnect()
```
The **``disconnect()``** method breaks the connection of a connected ``DatagramSocket`` so that it can once again **send packets** to and **receive packets** from any **host** and **port**.

#### getPort() 

```Java
public int getPort()
```

If and only if a ``DatagramSocket`` is connected, the **``getPort()``** method returns the **remote port** to which it is connected. Otherwise, it returns ``–1``.

#### getInetAddress()

```Java
public InetAddress getInetAddress()
```
If and only if a ``DatagramSocket`` is connected, the **``getInetAddress()``** method returns the address of the **remote host** to which it is connected. Otherwise, it returns ``null``.

#### getRemoteSocketAddress()

```Java
public InetAddress getRemoteSocketAddress()
```
If a ``DatagramSocket`` is connected, the **``getRemoteSocketAddress()``** method returns the address of the remote host to which it is connected. Otherwise, it returns ``null``.

## Socket Options
-----------------
Java supports six socket options for ``UDP``:

- ``SO_TIMEOUT``
- ``SO_RCVBUF``
- ``SO_SNDBUF``
- ``SO_REUSEADDR``
- ``SO_BROADCAST``
- ``IP_TOS``

### **``SO_TIMEOUT``**

``SO_TIMEOUT`` is the amount of time, in **milliseconds**, that **``receive()``** waits for an incoming datagram before throwing an ``InterruptedIOException``, which is a subclass of ``IOException``. Its value must be **nonnegative**. If ``SO_TIMEOUT`` is ``0``, **``receive()``** never times out. This value can be changed with the **``setSoTimeout()``** method and inspected with the **``getSoTimeout()``** method:

```Java
public void setSoTimeout(int timeout) throws SocketException
```

```Java
public int getSoTimeout() throws IOException
```

### **``SO_RCVBUF``**

The ``SO_RCVBUF`` option of ``DatagramSocket`` is closely related to the ``SO_RCVBUF`` option of Socket. It determines the size of the buffer used for network I/O.

``DatagramSocket`` has methods to set and get the suggested receive buffer size used for network input:

```Java
public void setReceiveBufferSize(int size) throws SocketException
```
```Java
public int getReceiveBufferSize() throws SocketException
```

### **``SO_SNDBUF``**

``DatagramSocket`` has methods to **get** and **set** the suggested send buffer size used for network output:

```Java
public void setSendBufferSize(int size) throws SocketException
```
```Java
public int getSendBufferSize() throws SocketException
```

### **``SO_REUSEADDR``**

The ``SO_REUSEADDR`` option does not mean the same thing for ``UDP`` sockets as it does for ``TCP`` sockets. For ``UDP``, ``SO_REUSEADDR`` controls whether multiple datagram sockets can bind to the same **port** and **address** at the same time. If multiple sockets are bound to the same port, received packets will be copied to all bound sockets. This option is controlled by these two methods:

```Java
public void setReuseAddress(boolean on) throws SocketException
```
```Java
public boolean getReuseAddress() throws SocketException
```

### **``SO_BROADCAST``**

The ``SO_BROADCAST`` option controls whether a socket is allowed to send packets to and receive packets from broadcast addresses such as ``192.168.254.255``, the local network broadcast address for the network with the local address ``192.168.254.*``. ``UDP`` broadcasting is often used for protocols such as ``DHCP`` that need to communicate with servers on the local net whose addresses are not known in advance. This option is controlled with these two methods:

```Java
public void setBroadcast(boolean on) throws SocketException
```
```Java
public boolean getBroadcast() throws SocketException
```

### **``IP_TOS``**

Because the traffic class is determined by the value of the ``IP_TOS`` field in each IP packet header, it is essentially the same for ``UDP`` as it is for ``TCP``. After all, packets are actually routed and prioritized according to ``IP``, which both ``TCP`` and ``UDP`` sit on top of. There’s really no difference between the **``setTrafficClass()``** and **``getTrafficClass()``** methods in ``DatagramSocket`` and those in ``Socket``. They just have to be repeated here because ``DatagramSocket`` and ``Socket`` don’t have a common superclass. These two methods let you inspect and set the class of service for a socket using these two methods:

```Java
public int getTrafficClass() throws SocketException
```
```Java
public void setTrafficClass(int trafficClass) throws SocketException
```
The traffic class is given as an int between ``0`` and ``255``.

## DatagramChannel
-------------------
The ``DatagramChannel`` class is used for nonblocking ``UDP`` applications, in the same way as ``SocketChannel`` and ``ServerSocketChannel`` are used for nonblocking TCP applications. Like ``SocketChannel`` and ``ServerSocketChannel``, ``DatagramChannel`` is a subclass of ``SelectableChannel`` that can be registered with a ``Selector``. This is useful in servers where one thread can manage communications with multiple clients. However, ``UDP`` is by its nature much more asynchronous than ``TCP`` so the net effect is smaller. In ``UDP``, a single datagram socket can process requests from multiple clients for both input and output. What the ``DatagramChannel`` class adds is the ability to do this in a nonblocking fashion, so methods return quickly if the network isn’t immediately ready to receive or send data.

### **Using DatagramChannel**

``DatagramChannel`` is a near-complete alternate ``API`` for ``UDP``. In Java 6 and earlier, you still need to use the ``DatagramSocket`` class to bind a channel to a port. However, you do not have to use it thereafter, and you don’t have to use it all in Java 7 and later. Nor do you ever use ``DatagramPacket``. Instead, you read and write byte buffers, just as you do with a ``SocketChannel``.

#### Opening a socket

The ``java.nio.channels.DatagramChannel`` class does not have any public constructors. Instead, you create a new ``DatagramChannel`` object using the static **``open()``** method For example:

```Java
DatagramChannel channel = DatagramChannel.open();
```
This channel is not initially bound to any port. To bind it, you access the channel’s peer ``DatagramSocket`` object using the **``socket()``** method. For example, this binds a channel to port ``3141``:

```Java
SocketAddress address = new InetSocketAddress(3141);
DatagramSocket socket = channel.socket();
socket.bind(address);
```
Java 7 adds a convenient **``bind()``** method directly to ``DatagramChannel``, so you don’t have to use a ``DatagramSocket`` at all. For example:

```Java
SocketAddress address = new InetSocketAddress(3141);
channel.bind(address);
```
#### Receiving

The **``receive()``** method reads one datagram packet from the channel into a ``ByteBuffer``. It returns the address of the host that sent the packet:

```Java
public SocketAddress receive(ByteBuffer dst) throws IOException
```

#### Sending

The **``send()``** method writes one datagram packet into the channel from a ``ByteBuffer`` to the address specified as the second argument:

```Java
public int send(ByteBuffer src, SocketAddress target) throws IOException
```

#### Connecting

Once you’ve opened a datagram channel, you connect it to a particular remote address using the **``connect()``** method:

```Java
SocketAddress remote = new InetSocketAddress("www.example.com", 37);
channel.connect(remote);
```

The channel will only send data to or receive data from this host.

There is an **``isConnected()``** method that returns ``true`` if and only if the ``DatagramSocket`` is connected:

```Java
public boolean isConnected()
```
This tells you whether the ``DatagramChannel`` is limited to one host. Unlike ``SocketChannel``, a ``DatagramChannel`` doesn’t have to be connected to transmit or receive data.

Finally, the **``disconnect()``** method breaks the connection:

```Java
public DatagramChannel disconnect() throws IOException
```

#### Reading

Besides the special-purpose **``receive()``** method, ``DatagramChannel`` has the usual three **``read()``** methods:

```Java
public int read(ByteBuffer dst) throws IOException
```
```Java
public long read(ByteBuffer[] dsts) throws IOException
```
```Java
public long read(ByteBuffer[] dsts, int offset, int length) throws IOException
```
However, these methods can only be used on connected channels. That is, before invoking one of these methods, you must invoke **``connect()``** to glue the channel to a particular remote host. This makes them more suitable for use with clients that know who they’ll be talking to than for servers that must accept input from multiple hosts at the same time that are normally not known prior to the arrival of the first packet. 

Each of these three methods only reads a single datagram packet from the network. As much data from that datagram as possible is stored in the argument ``ByteBuffer(s)``. Each method returns the number of bytes read or ``–1`` if the channel has been closed. This method may return 0 for any of several reasons, including:

- The channel is nonblocking and no packet was ready.
- A datagram packet contained no data.
- The buffer is full.

As with the **``receive()``** method, if the datagram packet has more data than the ``ByteBuffer(s)`` can hold, ***the extra data is thrown away with no notification of the problem***. You do not receive a ``BufferOverflowException`` or anything similar.

#### Writing

Naturally, ``DatagramChannel`` has the three write methods common to all writable, scattering channels, which can be used instead of the **``send()``** method:

```Java
public int write(ByteBuffer src) throws IOException
```
```Java
public long write(ByteBuffer[] dsts) throws IOException
```
```Java
public long write(ByteBuffer[] dsts, int offset, int length) throws IOException
```
These methods can only be used on connected channels; otherwise, they don’t know where to send the packet. Each of these methods sends a single datagram packet over the connection. None of these methods are guaranteed to write the complete contents of the buffer(s). Fortunately, the cursor-based nature of buffers enables you to easily call this method again and again until the buffer is fully drained and the data has been completely sent, possibly using multiple datagram packets. For example:

```Java
while (buffer.hasRemaining() && channel.write(buffer) != -1);
```

#### Closing

Just as with regular datagram sockets, a channel should be closed when you’re done with it to free up the port and any other resources it may be using:

```Java
public void close() throws IOException
```
Closing an already closed channel has no effect. Attempting to **write data** to or **read data** from a closed channel throws an exception. If you’re uncertain whether a channel has been closed, check with **``isOpen()``**:

```Java
public boolean isOpen()
```
This returns ``false`` if the channel is **closed**, ``true`` if it’s **open**.

Like all channels, in Java 7 ``DatagramChannel`` implements ``AutoCloseable`` so you can use it in **try-with-resources** statements. Prior to Java 7, close it in a ``finally`` block if you can. By now the pattern should be quite familiar. In Java 6 and earlier:

```Java
DatagramChannel channel = null;
try {
    channel = DatagramChannel.open();
    // Use the channel...
} catch (IOException ex) {
    // handle exceptions...
} finally {
    if (channel != null) {
        try {
            channel.close();
        } catch (IOException ex) {
            // handle exceptions...
        }
    }
}
```
and in Java 7 and later:

```Java
try (DatagramChannel channel = DatagramChannel.open()) {
    // Use the channel...
} catch (IOException ex) {
    // handle exceptions...
}
```

#### Socket Options (Java 7)

In Java 7 and later, ``DatagramChannel`` supports eight socket options listed :

- ``SO_SNDBUF`` : Size of the buffer used for sending datagram packets.
- ``SO_RCVBUF`` : Size of the buffer used for receiving datagram packets.
- ``SO_REUSEADDR`` : Enable/disable address reuse.
- ``SO_BROADCAST`` : Enable/disable broadcast messages.
- ``IP_TOS`` : Integer Traffic class.
- ``IP_MULTICAST_IF`` : Local network interface to use for multicast.
- ``IP_MULTICAST_TTL`` : Time-to-live value for multicast datagrams.
- ``IP_MULTICAST_LOOP`` : Enable/disable loopback of multicast datagrams.

The first **five** options have the same meanings as they do for datagram sockets as described in **Socket Options**. The last **three** are used by multicast sockets.

These are inspected and configured by just three methods:

```Java
public <T> DatagramChannel setOption(SocketOption<T> name, T value) throws IOException
```
```Java
public <T> T getOption(SocketOption<T> name) throws IOException
```
```Java
public Set<SocketOption<?>> supportedOptions()
```
The **``supportedOptions()``** method lists the available socket options. The **``getOption()``** method tells you the current value of any of these. And **``setOption()``** lets you change the value. For example, suppose you want to send a broadcast message. ``SO_BROADCAST`` is usually turned off by default, but you can switch it on like so:

```Java
try (DatagramChannel channel = DatagramChannel.open()) {
    channel.setOption(StandardSocketOptions.SO_BROADCAST, true);
    // Send the broadcast message...
} catch (IOException ex) {
    // handle exceptions...
}
```