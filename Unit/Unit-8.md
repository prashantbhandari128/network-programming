<div align="center">

[**_``Go Back``_**](../README.md)

# Secure Sockets

</div>

Secure Sockets are essential for ensuring secure communication between clients and servers over a network. In this study note, we'll cover the basics of creating secure client and server sockets using the Secure Socket Layer (SSL) or Transport Layer Security (TLS) protocols.

## Secure Communications
--------------------------
Secure communications involve encrypting data transmission to prevent eavesdropping and unauthorized access. SSL/TLS provides encryption and authentication, ensuring data integrity and confidentiality.

The Java Secure Socket Extension is divided into four packages:

- ``javax.net.ssl`` : The abstract classes that define Java’s API for secure network communication.

- ``javax.net`` : The abstract socket factory classes used instead of constructors to create secure sockets.

- ``java.security.cert`` : The classes for handling the public-key certificates needed for SSL. 

- ``com.sun.net.ssl`` : The concrete classes that implement the encryption algorithms and protocols in Sun’s reference implementation of the JSSE. Technically, these are not part of the JSSE standard. Other implementers may replace this package with one of their own; for instance, one that uses native code to speed up the CPU-intensive key generation and encryption process.

## Creating Secure Client Sockets
----------------------------------

If you don’t care very much about the underlying details, using an encrypted SSL socket to talk to an existing secure server is truly straightforward. Rather than constructing a ``java.net.Socket`` object with a constructor, you get one from a ``javax.net.ssl.SSLSocketFactory`` using its **``createSocket()``** method. ``SSLSocketFactory`` is an abstract class that follows the abstract factory design pattern.You get an instance of it by invoking the **``static SSLSocketFactory.getDefault()``** method:

```Java
SocketFactory factory = SSLSocketFactory.getDefault();
Socket socket = factory.createSocket("www.example.org", 7000);
```
This either returns an instance of ``SSLSocketFactory`` or throws an ``InstantiationException`` if no concrete subclass can be found. Once you have a reference to the factory, use one of these five overloaded ``createSocket()`` methods to build an ``SSLSocket``:

```Java
public abstract Socket createSocket(String host, int port) throws IOException, UnknownHostException
public abstract Socket createSocket(InetAddress host, int port) throws IOException
public abstract Socket createSocket(String host, int port, InetAddress interface, int localPort) throws IOException, UnknownHostException
public abstract Socket createSocket(InetAddress host, int port, InetAddress interface, int localPort) throws IOException , UnknownHostException
public abstract Socket createSocket(Socket proxy, String host, int port, boolean autoClose) throws IOException
```

The first two methods create and return a socket that’s connected to the specified ``host`` and ``port`` or throw an ``IOException`` if they can’t connect. The third and fourth methods connect and return a socket that’s connected to the specified ``host`` and ``port`` from the specified local network ``interface`` and ``port``.The last **``createSocket()``** method, however,is a little different. It begins with an existing Socket object that’s connected to a proxy server. It returns a ``Socket`` that tunnels through this proxy server to the specified ``host`` and ``port``. The ``autoClose`` argument determines whether the underlying proxy socket should be closed when this socket is closed. If ``autoClose`` is ``true``, the underlying socket will be closed; if ``false``, it won’t be.

The ``Socket`` that all these methods return will really be a ``javax.net.ssl.SSLSocket``, a subclass of ``java.net.Socket``. 

**Program :**
```Java
import javax.net.ssl.SSLSocket;
import javax.net.ssl.SSLSocketFactory;
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintWriter;

public class SecureClientSocketDemo {
    public final static String HOST = "www.example.com";
    public final static int PORT = 443;

    public static void main(String[] args) {
        try {

            // Create SSL socket factory
            SSLSocketFactory sslSocketFactory =(SSLSocketFactory) SSLSocketFactory.getDefault();

            // Create SSL socket
            SSLSocket sslSocket = (SSLSocket) sslSocketFactory.createSocket(HOST, PORT);

            // Get input and output streams
            PrintWriter out = new PrintWriter(sslSocket.getOutputStream(), true);
            BufferedReader in = new BufferedReader(new InputStreamReader(sslSocket.getInputStream()));

            // Send an HTTP GET request
            out.println("GET / HTTP/1.1");
            out.println("Host: " + HOST);
            out.println("Connection: close");
            out.println();

            // Read and print the response
            String line;
            while ((line = in.readLine()) != null) {
                System.out.println(line);
            }

            // Close the socket
            sslSocket.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```

**Output :**
```
HTTP/1.1 200 OK
Age: 309192
Cache-Control: max-age=604800
Content-Type: text/html; charset=UTF-8
Date: Fri, 04 Aug 2023 14:15:29 GMT
Etag: "3147526947+gzip+ident"
Expires: Fri, 11 Aug 2023 14:15:29 GMT
Last-Modified: Thu, 17 Oct 2019 07:18:26 GMT
Server: ECS (dcb/7F3C)
Vary: Accept-Encoding
X-Cache: HIT
Content-Length: 1256
Connection: close

<!doctype html>
<html>
<head>
    <title>Example Domain</title>

    <meta charset="utf-8" />
    <meta http-equiv="Content-type" content="text/html; charset=utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <style type="text/css">
    body {
        background-color: #f0f0f2;
        margin: 0;
        padding: 0;
        font-family: -apple-system, system-ui, BlinkMacSystemFont, "Segoe UI", "Open Sans", "Helvetica Neue", Helvetica, Arial, sans-serif;
        
    }
    div {
        width: 600px;
        margin: 5em auto;
        padding: 2em;
        background-color: #fdfdff;
        border-radius: 0.5em;
        box-shadow: 2px 3px 7px 2px rgba(0,0,0,0.02);
    }
    a:link, a:visited {
        color: #38488f;
        text-decoration: none;
    }
    @media (max-width: 700px) {
        div {
            margin: 0 auto;
            width: auto;
        }
    }
    </style>    
</head>

<body>
<div>
    <h1>Example Domain</h1>
    <p>This domain is for use in illustrative examples in documents. You may use this
    domain in literature without prior coordination or asking for permission.</p>
    <p><a href="https://www.iana.org/domains/example">More information...</a></p>
</div>
</body>
</html>
```

### **Choosing the Cipher Suites**
When establishing a secure socket connection using ``SSL/TLS`` in Java, the choice of cipher suites determines the encryption algorithms and security protocols that will be used for communication between the client and the server. Cipher suites are sets of ``cryptographic algorithms`` that include key exchange, encryption, and message authentication algorithms.

It's important to choose appropriate cipher suites to ensure a balance between security and compatibility with the systems you're interacting with. Some cipher suites may provide stronger security but could be incompatible with older or certain systems.

The **``getSupportedCipherSuites()``** method in ``SSLSocketFactory`` tells you which combination of algorithms is available on a given socket:

```Java
public abstract String[] getSupportedCipherSuites()
```

**Program :**

```Java
import javax.net.ssl.SSLSocket;
import javax.net.ssl.SSLSocketFactory;
public class SupportedCipherSuitesExample {
    public static void main(String[] args) {
        try {
            // Create SSL socket factory
            SSLSocketFactory sslSocketFactory = (SSLSocketFactory) SSLSocketFactory.getDefault();

            // Create a dummy socket to retrieve the supported cipher suites
            SSLSocket dummySocket = (SSLSocket) sslSocketFactory.createSocket();
            String[] supportedCipherSuites = dummySocket.getSupportedCipherSuites();

            // Print the list of supported cipher suites
            System.out.println("Supported Cipher Suites:");
            for (String cipherSuite : supportedCipherSuites) {
                System.out.println("- " + cipherSuite);
            }

            // Close the dummy socket
            dummySocket.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

**Example :**

```
Supported Cipher Suites:
- TLS_AES_256_GCM_SHA384
- TLS_AES_128_GCM_SHA256
- TLS_CHACHA20_POLY1305_SHA256
- TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
- TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
- TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256
- TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
- TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256
- TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
- TLS_DHE_RSA_WITH_AES_256_GCM_SHA384
- TLS_DHE_RSA_WITH_CHACHA20_POLY1305_SHA256
- TLS_DHE_DSS_WITH_AES_256_GCM_SHA384
- TLS_DHE_RSA_WITH_AES_128_GCM_SHA256
- TLS_DHE_DSS_WITH_AES_128_GCM_SHA256
- TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384
- TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384
- TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
- TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256
- TLS_DHE_RSA_WITH_AES_256_CBC_SHA256
- TLS_DHE_DSS_WITH_AES_256_CBC_SHA256
- TLS_DHE_RSA_WITH_AES_128_CBC_SHA256
- TLS_DHE_DSS_WITH_AES_128_CBC_SHA256
- TLS_ECDH_ECDSA_WITH_AES_256_GCM_SHA384
- TLS_ECDH_RSA_WITH_AES_256_GCM_SHA384
- TLS_ECDH_ECDSA_WITH_AES_128_GCM_SHA256
- TLS_ECDH_RSA_WITH_AES_128_GCM_SHA256
- TLS_ECDH_ECDSA_WITH_AES_256_CBC_SHA384
- TLS_ECDH_RSA_WITH_AES_256_CBC_SHA384
- TLS_ECDH_ECDSA_WITH_AES_128_CBC_SHA256
- TLS_ECDH_RSA_WITH_AES_128_CBC_SHA256
- TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA
- TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA
- TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA
- TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA
- TLS_DHE_RSA_WITH_AES_256_CBC_SHA
- TLS_DHE_DSS_WITH_AES_256_CBC_SHA
- TLS_DHE_RSA_WITH_AES_128_CBC_SHA
- TLS_DHE_DSS_WITH_AES_128_CBC_SHA
- TLS_ECDH_ECDSA_WITH_AES_256_CBC_SHA
- TLS_ECDH_RSA_WITH_AES_256_CBC_SHA
- TLS_ECDH_ECDSA_WITH_AES_128_CBC_SHA
- TLS_ECDH_RSA_WITH_AES_128_CBC_SHA
- TLS_RSA_WITH_AES_256_GCM_SHA384
- TLS_RSA_WITH_AES_128_GCM_SHA256
- TLS_RSA_WITH_AES_256_CBC_SHA256
- TLS_RSA_WITH_AES_128_CBC_SHA256
- TLS_RSA_WITH_AES_256_CBC_SHA
- TLS_RSA_WITH_AES_128_CBC_SHA
- TLS_EMPTY_RENEGOTIATION_INFO_SCSV
```

However, not all cipher suites that are understood are necessarily allowed on the connection. Some may be too weak and consequently disabled. The **``getEnabledCipherSuites()``** method of ``SSLSocketFactory`` tells you which suites this socket is willing to use:

The actual suite used is negotiated between the client and server at connection time. It’s possible that the client and the server won’t agree on any suite. It’s also possible that although a suite is enabled on both client and server, one or the other or both won’t have the keys and certificates needed to use the suite. In either case, the **``createSocket()``** method will throw an ``SSLException``, a subclass of ``IOException``. You can change the suites the client attempts to use via the **``setEnabledCipherSuites()``** method:

```Java
public abstract void setEnabledCipherSuites(String[] suites)
```

The argument to this method should be a list of the suites you want to use. Each name must be one of the suites listed by **``getSupportedCipherSuites()``**. Otherwise, an ``IllegalArgumentException`` will be thrown. 

**Program :**

```Java
import javax.net.ssl.SSLSocket;
import javax.net.ssl.SSLSocketFactory;
import java.io.IOException;

public class EnabledCipherSuitesExample {
    public final static String HOST = "www.example.com";
    public final static int PORT = 443;

    public static void main(String[] args) {
        try {

            // Create SSL socket factory
            SSLSocketFactory sslSocketFactory = (SSLSocketFactory) SSLSocketFactory.getDefault();

            // Create SSL socket
            SSLSocket sslSocket = (SSLSocket) sslSocketFactory.createSocket(HOST, PORT);

            // Set the list of enabled cipher suites
            String[] cipherSuites = {
                    "TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256",
                    "TLS_RSA_WITH_AES_128_CBC_SHA",
                    "TLS_RSA_WITH_AES_256_CBC_SHA"
            };
            sslSocket.setEnabledCipherSuites(cipherSuites);

            // Get the list of enabled cipher suites
            String[] enabledCipherSuites = sslSocket.getEnabledCipherSuites();

            // Print the list of enabled cipher suites
            System.out.println("Enabled Cipher Suites:");
            for (String cipherSuite : enabledCipherSuites) {
                System.out.println("- " + cipherSuite);
            }

            // Close the socket
            sslSocket.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

**Output :**
```
Enabled Cipher Suites:
- TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
- TLS_RSA_WITH_AES_128_CBC_SHA
- TLS_RSA_WITH_AES_256_CBC_SHA
```

## Event Handlers
------------------
Network communications are slow compared to the speed of most computers. Authenticated network communications are even slower. The necessary key generation and setup for a secure connection can easily take several seconds. Consequently, you may want to deal with the connection asynchronously. JSSE uses the standard Java event model to notify programs when the handshaking between client and server is complete. The pattern is a familiar one. In order to get notifications of handshake-complete events, simply implement the ``HandshakeCompletedListener`` interface:

```Java
public interface HandshakeCompletedListener extends java.util.EventListener
```
This interface declares the **``handshakeCompleted()``** method:

```Java
public void handshakeCompleted(HandshakeCompletedEvent event)
```
This method receives as an argument a ``HandshakeCompletedEvent``:

```Java
public class HandshakeCompletedEvent extends java.util.EventObject
```
The ``HandshakeCompletedEvent`` class provides four methods for getting information about the event:

```Java
public SSLSession getSession()
public String getCipherSuite()
public X509Certificate[] getPeerCertificateChain() throws SSLPeerUnverifiedException
public SSLSocket getSocket()
```
Particular ``HandshakeCompletedListener`` objects register their interest in handshakecompleted events from a particular SSLSocket via its **``addHandshakeCompletedListener()``** and **``removeHandshakeCompletedListener()``** methods:

```Java
public abstract void addHandshakeCompletedListener(HandshakeCompletedListener listener)
public abstract void removeHandshakeCompletedListener(HandshakeCompletedListener listener) throws IllegalArgumentException
```

## Session Management
----------------------
Session management in secure sockets, such as those implemented using the Java Secure Socket Extension (JSSE), involves handling and maintaining the state of SSL/TLS sessions between clients and servers. Sessions are essential for optimizing performance, reducing the overhead of repeated handshakes, and maintaining security during ongoing communication.

In the JSSE, sessions are represented by instances of the ``SSLSession`` interface; you can use the methods of this interface to check the times the session was created and last accessed, invalidate the session, and get various information about the session:

```Java
public byte[] getId()
public SSLSessionContext getSessionContext()
public long getCreationTime()
public long getLastAccessedTime()
public void invalidate()
public void putValue(String name, Object value)
public Object getValue(String name)
public void removeValue(String name)
public String[] getValueNames()
public X509Certificate[] getPeerCertificateChain() throws SSLPeerUnverifiedException
public String getCipherSuite()
public String getPeerHost()
```
The **``getSession()``** method of ``SSLSocket`` returns the ``Session`` this socket belongs to:

```Java
public abstract SSLSession getSession()
```
To prevent a socket from creating a session that passes ``false`` to **``setEnableSessionCreation()``**, use:

```Java
public abstract void setEnableSessionCreation(boolean allowSessions)
```
The **``getEnableSessionCreation()``** method returns true if multisocket sessions are allowed, ``false`` if they’re not:

On rare occasions, you may even want to reauthenticate a connection (i.e., throw away all the certificates and keys that have previously been agreed to and start over with a new session). The **startHandshake()** method does this:

```Java
public abstract void startHandshake() throws IOException
```

**Pragram :**
```Java
import javax.net.ssl.*;
import java.io.IOException;

public class SessionManagementExample {
    public final static String HOST = "www.example.com";
    public final static int PORT = 443;
    public static void main(String[] args) throws IOException {
        try {
            SSLContext sslContext = SSLContext.getInstance("TLS");
            sslContext.init(null, null, null);
            SSLSocketFactory sslSocketFactory = sslContext.getSocketFactory();

            // Create an SSL socket with a secure connection
            SSLSocket sslSocket = (SSLSocket) sslSocketFactory.createSocket(HOST, PORT);

            // Start the SSL handshake
            sslSocket.startHandshake();

            // Get the SSL session associated with the socket
            SSLSession sslSession = sslSocket.getSession();

            // Access session information
            System.out.println("Session ID: " + sslSession.getId());
            System.out.println("Creation Time: " + sslSession.getCreationTime());
            System.out.println("Last Accessed Time: " + sslSession.getLastAccessedTime());
            System.out.println("Cipher Suite: " + sslSession.getCipherSuite());
            System.out.println("Peer Host: " + sslSession.getPeerHost());

            // Store and retrieve session values
            sslSession.putValue("Name", "Prashant");
            System.out.println("Session Value: " + sslSession.getValue("Name"));

            // Remove session value
            sslSession.removeValue("Name");

            // Invalidate the session
            sslSession.invalidate();

            // Close the SSL socket
            sslSocket.close();

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

**Output :**
```
Session ID: [B@15b3e5b
Creation Time: 1691168876739
Last Accessed Time: 1691168876739
Cipher Suite: TLS_AES_256_GCM_SHA384
Peer Host: www.example.com
Session Value: Prashant
```

## Client Mode 
---------------
Client Mode in secure sockets refers to the operational state of a network communication entity, typically a software application, that initiates a secure connection with a remote server using encryption protocols like ``SSL (Secure Sockets Layer)`` or its successor, ``TLS (Transport Layer Security)``. This mode is fundamental to establishing secure and encrypted channels for data transmission over untrusted networks such as the internet. 

The **``setUseClientMode()``** method determines whether the socket needs to use authentication in its first handshake. The name of the method is a little misleading. It can be used for both client- and server-side sockets. However, when ``true`` is passed in, it means the socket is in client mode (whether it’s on the client side or not) and will not offer to authenticate itself. When ``false`` is passed, it will try to authenticate itself:

```Java
public abstract void setUseClientMode(boolean mode) throws IllegalArgumentException
```
This property can be set only once for any given socket. Attempting to set it a second time throws an ``IllegalArgumentException``.

The **``getUseClientMode()``** method simply tells you whether this socket will use authentication in its first handshake:

```Java
public abstract boolean getUseClientMode()
```
A secure socket on the server side (i.e., one returned by the **``accept()``** method of an ``SSLServerSocket``) uses the **``setNeedClientAuth()``** method to require that all clients connecting to it authenticate themselves (or not):

```Java
public abstract void setNeedClientAuth(boolean needsAuthentication) throws IllegalArgumentException
```

This method throws an ``IllegalArgumentException`` if the socket is not on the server side.

The **``getNeedClientAuth()``** method returns ``true`` if the socket requires authentication from the client side, ``false`` otherwise:

```Java
public abstract boolean getNeedClientAuth()
```
**Program :**
```Java
import javax.net.ssl.*;
import java.io.*;
import java.security.*;

public class SSLClientServerDemo {

    public static void main(String[] args) {
        int port = 12345;

        // Start the server in a separate thread
        Thread serverThread = new Thread(() -> startServer(port));
        serverThread.start();

        // Allow some time for the server to start
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        // Start the client
        startClient("localhost", port);
    }

    private static void startServer(int port) {
        try {
            SSLContext sslContext = SSLContext.getInstance("TLS");
            sslContext.init(null, null, new SecureRandom());

            SSLServerSocketFactory serverSocketFactory = sslContext.getServerSocketFactory();
            SSLServerSocket serverSocket = (SSLServerSocket) serverSocketFactory.createServerSocket(port);

            // Set client authentication mode
            serverSocket.setNeedClientAuth(true);

            System.out.println("Server listening on port " + port);

            while (true) {
                try (SSLSocket clientSocket = (SSLSocket) serverSocket.accept()) {
                    boolean needClientAuth = clientSocket.getNeedClientAuth();
                    System.out.println("Server: Need Client Auth: " + needClientAuth);
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private static void startClient(String host, int port) {
        try {
            SSLContext sslContext = SSLContext.getInstance("TLS");
            sslContext.init(null, null, new SecureRandom());

            SSLSocketFactory socketFactory = sslContext.getSocketFactory();
            SSLSocket clientSocket = (SSLSocket) socketFactory.createSocket(host, port);

            // Set client mode
            clientSocket.setUseClientMode(true);

            System.out.println("Client connected to " + host + " on port " + port);

            if(clientSocket.getUseClientMode())
            {
                System.out.println("Socket will use authentication in its first handshake.");
            }

            boolean needClientAuth = clientSocket.getNeedClientAuth();
            System.out.println("Client: Need Client Auth: " + needClientAuth);

            clientSocket.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

**Output :**
```
Server listening on port 12345
Server: Need Client Auth: true
Client connected to localhost on port 12345
Socket will use authentication in its first handshake.
Client: Need Client Auth: false
```

## Creating Secure Server Sockets
-----------------------------------
Creating secure server sockets involves setting up a server-side application that can accept incoming connections from clients over a secure channel using ``SSL/TLS`` encryption. 

Secure client sockets are only half of the equation. The other half is SSL-enabled server sockets. These are instances of the ``javax.net.SSLServerSocket`` class:

```Java
public abstract class SSLServerSocket extends ServerSocket
```
Like ``SSLSocket``, all the constructors in this class are protected and instances are created by an abstract factory class, ``javax.net.SSLServerSocketFactory``

```Java
public abstract class SSLServerSocketFactory extends ServerSocketFactory
```

Also like ``SSLSocketFactory``, an instance of ``SSLServerSocketFactory`` is returned by a static **``SSLServerSocketFactory.getDefault()``** method:

```Java
public static ServerSocketFactory getDefault()
```

And like ``SSLSocketFactory``, ``SSLServerSocketFactory`` has three overloaded create **``ServerSocket()``** methods that return instances of ``SSLServerSocket`` and are easily understood by analogy with the ``java.net.ServerSocket`` constructors:

```Java
public abstract ServerSocket createServerSocket(int port) throws IOException
public abstract ServerSocket createServerSocket(int port, int queueLength) throws IOException
public abstract ServerSocket createServerSocket(int port, int queueLength, InetAddress interface) throws IOException
```
**Program :**
```Java
import javax.net.ssl.*;
import java.io.*;

public class SecureServerSocketExample {
    public final static int PORT = 8080;

    public static void main(String[] args) {

        try {
            // Create an SSLServerSocketFactory
            SSLServerSocketFactory sslServerSocketFactory = (SSLServerSocketFactory) SSLServerSocketFactory.getDefault();

            // Create a secure server socket
            SSLServerSocket serverSocket = (SSLServerSocket) sslServerSocketFactory.createServerSocket(PORT);

            System.out.println("Secure server started on port : " + PORT);

            // Wait for incoming connections
            while (true) {
                SSLSocket clientSocket = (SSLSocket) serverSocket.accept();
                System.out.println("Client connected: " + clientSocket.getInetAddress());
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

If that were all there was to creating secure server sockets, they would be quite straight forward and simple to use. Unfortunately, that’s not all there is to it. The factory that ``SSLServerSocketFactory.getDefault()`` returns generally only supports server authentication. To get encryption as well, server-side secure sockets require more initialization and setup.

- Generate public keys and certificates using ``keytool``.
- Pay money to have your certificates authenticated by a trusted third party such as ``Comodo``.
- Create an ``SSLContext`` for the algorithm you’ll use.
- Create a ``TrustManagerFactory`` for the source of certificate material you’ll be using.
- Create a ``KeyManagerFactory`` for the type of key material you’ll be using
- Create a ``KeyStore`` object for the key and certificate database. (Oracle’s default is ``JKS``.)
- Fill the ``KeyStore`` object with keys and certificates; for instance, by loading them from the filesystem using the passphrase they’re encrypted with.
- Initialize the ``KeyManagerFactory`` with the ``KeyStore`` and its ``passphrase``.
- Initialize the context with the necessary key managers from the ``KeyManagerFactory``, trust managers from the ``TrustManagerFactory``, and a source of randomness.(The last two can be ``null`` if you’re willing to accept the defaults.)

## Configuring SSLServerSockets
-------------------------------
Once you’ve successfully created and initialized an ``SSLServerSocket``, there are a lot of applications you can write using nothing more than the methods inherited from ``java.net.ServerSocket``. However, there are times when you need to adjust its behavior a little. Like ``SSLSocket``, ``SSLServerSocket`` provides methods to choose cipher suites, manage sessions, and establish whether clients are required to authenticate themselves. Most of these methods are similar to the methods of the same name in ``SSLSocket``. The difference is that they work on the server side and set the defaults for sockets accepted by an ``SSLServerSocket``. In some cases, once an ``SSLSocket`` has been accepted, you can still use the methods of ``SSLSocket`` to configure that one socket rather than all sockets accepted by this ``SSLServerSocket``.

### **Choosing the Cipher Suites**

The ``SSLServerSocket`` class has the same three methods for determining which cipher suites are supported and enabled as ``SSLSocket`` does:

```Java
public abstract String[] getSupportedCipherSuites()
public abstract String[] getEnabledCipherSuites()
public abstract void setEnabledCipherSuites(String[] suites)
```

These use the same suite names as the similarly named methods in ``SSLSocket``. The difference is that these methods apply to all sockets accepted by the ``SSLServerSocket`` rather than to just one ``SSLSocket``. For example, the following code fragment has the effect of enabling anonymous, unauthenticated connections on the ``SSLServerSocket`` server. It relies on the names of these suites containing the string ``anon``. This is ``true`` for Oracle’s reference implementations, though there’s no guarantee that other implementers will follow this convention:

```Java
String[] supported = server.getSupportedCipherSuites();
String[] anonCipherSuitesSupported = new String[supported.length];
int numAnonCipherSuitesSupported = 0;
for (int i = 0; i < supported.length; i++) {
    if (supported[i].indexOf("_anon_") > 0) {
        anonCipherSuitesSupported[numAnonCipherSuitesSupported++]= supported[i];
    }
}
String[] oldEnabled = server.getEnabledCipherSuites();
String[] newEnabled = new String[oldEnabled.length+ numAnonCipherSuitesSupported];
System.arraycopy(oldEnabled, 0, newEnabled, 0, oldEnabled.length);
System.arraycopy(anonCipherSuitesSupported, 0, newEnabled, oldEnabled.length, numAnonCipherSuitesSupported);
server.setEnabledCipherSuites(newEnabled);
```
This fragment retrieves the list of both supported and enabled cipher suites using **``getSupportedCipherSuites()``** and **``getEnabledCipherSuites()``**. It looks at the name of every supported suite to see whether it contains the substring ``“anon”``. If the suite name does contain this substring, the suite is added to a list of anonymous cipher suites. Once the list of anonymous cipher suites is built, it’s combined in a new array with the previous list of enabled cipher suites. The new array is then passed to ``setEnabledCipherSuites()`` so that both the previously enabled and the anonymous cipher suites can now be used.

### **Session Management**

Both client and server must agree to establish a session. The server side uses the **``setEnableSessionCreation()``** method to specify whether this will be allowed and the **``getEnableSessionCreation()``** method to determine whether this is currently allowed:

```Java
public abstract void setEnableSessionCreation(boolean allowSessions)
public abstract boolean getEnableSessionCreation()
```
``Session`` creation is enabled by default. If the server disallows session creation, then a client that wants a session will still be able to connect. It just won’t get a session and will have to handshake again for every socket. Similarly, if the client refuses sessions but the server allows them, they’ll still be able to talk to each other but without sessions.

### **Client Mode**

The ``SSLServerSocket`` class has two methods for determining and specifying whether client sockets are required to authenticate themselves to the server. By passing ``true`` to the **``setNeedClientAuth()``** method, you specify that only connections in which the client is able to authenticate itself will be accepted. By passing ``false``, you specify that authentication is not required of clients. The default is ``false``. If, for some reason, you need to know what the current state of this property is, the ``getNeedClientAuth()`` method will tell you:

```Java
public abstract void setNeedClientAuth(boolean flag)
public abstract boolean getNeedClientAuth()
```
The **``setUseClientMode()``** method allows a program to indicate that even though it has created an ``SSLServerSocket``, it is and should be treated as a client in the communication with respect to authentication and other negotiations. For example, in an ``FTP`` session, the client program opens a server socket to receive data from the server, but that doesn’t make it less of a client. The **``getUseClientMode()``** method returns ``true`` if the ``SSLServerSocket`` is in client mode, ``false`` otherwise:

```Java
public abstract void setUseClientMode(boolean flag)
public abstract boolean getUseClientMode()
```