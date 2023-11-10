<div align="center">

[**_``Go Back``_**](../README.md)

# URLConnections

</div>

``URLConnection`` is an abstract class that represents an active connection to a resource specified by a ``URL``. The ``URLConnection`` class has two different but related purposes. First, it provides more control over the interaction with a server (especially an ``HTTP`` server) than the ``URL`` class. A ``URLConnection`` can inspect the header sent by the server and respond accordingly. It can set the header fields used in the client request. Finally, a ``URLConnection`` can send data back to a web server with ``POST``, ``PUT``, and other ``HTTP`` request methods. 

## Openning URLConnection
--------------------------
A program that uses the ``URLConnection`` class directly follows this basic sequence of steps:

- Construct a ``URL`` object.
- Invoke the ``URL`` object’s ``openConnection()`` method to retrieve a ``URLConnection`` object for that ``URL``.
- Configure the ``URLConnection``.
- Read the header fields.
- Get an input stream and read data.
- Get an output stream and write data.
- Close the connection.

The single constructor for the URLConnection class is protected:

```Java
protected URLConnection(URL url)
```
Consequently, unless you’re subclassing ``URLConnection`` to handle a new kind of URL(i.e., writing a protocol handler), you create one of these objects by invoking the ``openConnection()`` method of the ``URL`` class. 

```Java
public URLConnection openConnection() throws IOException {}    
```

For example:

```Java
try {
    URL u = new URL("https://www.example.com/");
    URLConnection uc = u.openConnection();
    // read from the URL...
} catch (MalformedURLException ex) {
    System.err.println(ex);
} catch (IOException ex) {
    System.err.println(ex);
}
```

## Reading Data from a Server
-----------------------------
The following is the minimal set of steps needed to retrieve data from a ``URL`` using a ``URLConnection`` object:

- Construct a ``URL`` object.
- Invoke the ``URL`` object’s **``openConnection()``** method to retrieve a ``URLConnection`` object for that ``URL``.
- Invoke the ``URLConnection’s`` **``getInputStream()``** method.
- Read from the input stream using the usual stream ``API``.

Example:

```Java
import java.io.*;
import java.net.*;

public class SourceViewer {
    public static void main(String[] args) {
        try {
            URL u = new URL("https://www.example.com/");
            URLConnection uc = u.openConnection();
            InputStream stream = uc.getInputStream();
            int i;
            while ((i = stream.read()) != -1) {
                System.out.print((char) i);
            }
        } catch (Exception e) {
            System.out.println(e);
        }
    }
}
```

The ``openStream()`` method of the ``URL`` class just returns an ``InputStream`` from its own ``URLConnection`` object. The differences between ``URL`` and ``URLConnection`` aren’t apparent with just a simple input stream as in this example. The biggest differences between the two classes are:

- ``URLConnection`` provides access to the HTTP header.
- ``URLConnection`` can configure the request parameters sent to the server.
- ``URLConnection`` can write data to the server as well as read data from the server.

## Reading the Header
----------------------
HTTP servers provide a substantial amount of information in the header that precedes each response. For example, here’s a typical HTTP header returned by an Apache web server:

```
HTTP/1.1 301 Moved Permanently
Date: Sun, 21 Apr 2023 15:12:46 GMT
Server: Apache
Location: http://www.example.com/
Content-Length: 296
Connection: close
Content-Type: text/html; charset=iso-8859-1
```

There’s a lot of information there. In general, an ``HTTP`` header may include the content type of the requested document, the length of the document in bytes, the character set in which the content is encoded, the date and time, the date the content expires, and the date the content was last modified. However, the information depends on the server; some servers send all this information for each request, others send some information, and a few don’t send anything. The methods of this section allow you to query a ``URLConnection`` to find out what metadata the server has provided.

### **Retrieving Specific Header Fields**

The first six methods request specific, particularly common fields from the header. These are:

- Content-type
- Content-length
- Content-encoding
- Date
- Last-modified
- Expires

Methods:

- **``public String getContentType()``**: Returns the value of the content-type header field.

- **``public int getContentLength()``**: The ``getContentLength()`` method tells you how many bytes there are in the content. If there is no ``Content-length`` header, ``getContentLength()`` returns ``–1``.

- **``public String getContentEncoding()``**: The ``getContentEncoding()`` method returns a String that tells you how the content is encoded. If the content is sent unencoded (as is commonly the case with HTTP servers), this method returns ``null``. It throws no exceptions.

- **``public long getDate()``**: The ``getDate()`` method returns a long that tells you when the document was sent, in ``milliseconds`` since midnight, ``Greenwich Mean Time (GMT)``, January 1, 1970. You can convert it to a ``java.util.Date``. For example:

    ```Java
    Date documentSent = new Date(uc.getDate());
    ```

- **``public long getExpiration()``**: Some documents have server-based expiration dates that indicate when the document should be deleted from the cache and reloaded from the server. ``getExpiration()`` is very similar to ``getDate()``, differing only in how the return value is interpreted. It returns a ``long`` indicating the number of milliseconds after ``12:00 A.M., GMT, January 1, 1970``, at which the document expires. If the HTTP header does not include an Expiration field, ``getExpiration()`` returns ``0``, which means that the document does not expire and can remain in the cache indefinitely.

- **``public long getLastModified()``**: getLastModified(), returns the date on which the document was last modified. Again, the date is given as the number of milliseconds since midnight, ``GMT, January 1, 1970``. If the HTTP header does not include a Last-modified field (and many don’t), this method returns ``0``.

Example:

```Java
import java.io.IOException;
import java.net.MalformedURLException;
import java.net.URL;
import java.net.URLConnection;
import java.util.Date;

public class HeaderViewer {
    public static void main(String[] args) {
        String url = "https://www.example.com/";
        try {
            URL u = new URL(url);
            URLConnection uc = u.openConnection();
            System.out.println("Content-type: " + uc.getContentType());
            if (uc.getContentEncoding() != null) {
                System.out.println("Content-encoding: " + uc.getContentEncoding());
            }
            if (uc.getDate() != 0) {
                System.out.println("Date: " + new Date(uc.getDate()));
            }
            if (uc.getLastModified() != 0) {
                System.out.println("Last modified: " + new Date(uc.getLastModified()));
            }
            if (uc.getExpiration() != 0) {
                System.out.println("Expiration date: " + new Date(uc.getExpiration()));
            }
            if (uc.getContentLength() != -1) {
                System.out.println("Content-length: " + uc.getContentLength());
            }
        } catch (MalformedURLException ex) {
            System.err.println(url + " is not a URL I understand");
        } catch (IOException ex) {
            System.err.println(ex.getMessage());
        }
    }
}

```

Output:

```
Content-type: text/html; charset=utf-8
Date: Fri May 31 18:08:09 EDT 2013
Last modified: Fri May 31 17:04:14 EDT 2013
Expiration date: Fri May 31 22:08:09 EDT 2013
Content-length: 83273
```

### **Retrieving Arbitrary Header Fields**

The last six methods requested specific fields from the header, but there’s no theoretical limit to the number of header fields a message can contain. The next five methods inspect arbitrary fields in a header. Indeed, the methods of the preceding section are just thin wrappers over the methods discussed here; you can use these methods to get header fields that Java’s designers did not plan for. If the requested header is found, it is returned. Otherwise, the method returns ``null``.

- **``public String getHeaderField(String name)``** : The ``getHeaderField()`` method returns the value of a named header field. The name of the header is not case sensitive and does not include a closing colon. For example, to get the value of the Content-type and Content-encoding header fields of a ``URLConnection`` object ``uc``, you could write:
    ```Java
    String contentType = uc.getHeaderField("content-type");
    String contentEncoding = uc.getHeaderField("content-encoding");
    ```
    To get the Date, Content-length, or Expires headers, you’d do the same:
    ```Java
    String date = uc.getHeaderField("date");
    String expires = uc.getHeaderField("expires");
    String contentLength = uc.getHeaderField("Content-length");
    ```
    These methods all return ``String``, not ``int`` or ``long`` as the ``getContentLength()``, ``getExpirationDate()``, ``getLastModified()``, and ``getDate()``.

- **``public String getHeaderFieldKey(int n)``** : This method returns the key (i.e., the field name) of the ``nth`` header field (e.g., Contentlength or Server). The request method is header zero and has a null key. The first header is one. For example, in order to get the sixth key of the header of the ``URLConnection uc``, you would write:
    ```Java
    String header6 = uc.getHeaderFieldKey(6);
    ```

- **``public String getHeaderField(int n)``** : This method returns the value of the ``nth`` header field. In HTTP, the starter line containing the request method and path is header field ``zero`` and the ``first`` actual header is ``one``.

    Example:

    ```Java
    import java.io.*;
    import java.net.*;
    public class HeaderViewer2 {
        public static void main(String[] args) {
            String url = "https://www.example.com/";
            try {
                URL u = new URL(url);
                URLConnection uc = u.openConnection();
                for (int j = 1; ; j++) {
                    String header = uc.getHeaderField(j);
                    if (header == null) break;
                    System.out.println(uc.getHeaderFieldKey(j) + ": " + header);
                }
            } catch (MalformedURLException ex) {
                System.err.println(url + " is not a URL I understand.");
            } catch (IOException ex) {
                System.err.println(ex);
            }
        }
    }
    ```

    Output:

    ```
    Date: Sat, 04 May 2013 11:28:26 GMT
    Server: Apache
    Last-Modified: Sat, 04 May 2013 07:35:04 GMT
    Accept-Ranges: bytes
    Content-Length: 80366
    Content-Type: text/html; charset=utf-8
    Cache-Control: max-age=14400
    Expires: Sat, 04 May 2013 15:28:26 GMT
    Vary: Accept-Encoding
    Keep-Alive: timeout=3, max=100
    Connection: Keep-Alive
    ```

- **``public long getHeaderFieldDate(String name, long default)``** : This method first retrieves the header field specified by the name argument and tries to convert the string to a long that specifies the milliseconds since ``midnight, January 1, 1970, GMT``. ``getHeaderFieldDate()`` can be used to retrieve a header field that represents a date (e.g., the Expires, Date, or Last-modified headers). To convert the string to an integer, ``getHeaderFieldDate()`` uses the ``parseDate()`` method of ``java.util.Date``.
    ```Java
    Date expires = new Date(uc.getHeaderFieldDate("expires", 0));
    long lastModified = uc.getHeaderFieldDate("last-modified", 0);
    Date now = new Date(uc.getHeaderFieldDate("date", 0));
    ```

- **``public int getHeaderFieldInt(String name, int default)``** : This method retrieves the value of the header field name and tries to convert it to an int. If it fails, either because it can’t find the requested header field or because that field does not contain a recognizable integer, ``getHeaderFieldInt()`` returns the default argument. This method is often used to retrieve the ``Content-length`` field. For example, to get the content length from a ``URLConnection uc``, you would write:
    ```Java
    int contentLength = uc.getHeaderFieldInt("content-length", -1);
    ```
    In this code fragment, ``getHeaderFieldInt()`` returns ``–1`` if the ``Content-length`` header isn’t present.

## Caches
-----------
Web browsers have been caching pages and images for years. If a logo is repeated on every page of a site, the browser normally loads it from the remote server only once,stores it in its cache, and reloads it from the cache whenever it’s needed rather than requesting it from the remote server every time the logo is encountered. Several HTTP headers, including Expires and Cache-control, can control caching.

A simple Java class for parsing and querying Cache-control headers:

```Java
import java.util.Date;
import java.util.Locale;

public class CacheControl {
    private Date maxAge = null;
    private Date sMaxAge = null;
    private boolean mustRevalidate = false;
    private boolean noCache = false;
    private boolean noStore = false;
    private boolean proxyRevalidate = false;
    private boolean publicCache = false;
    private boolean privateCache = false;

    public CacheControl(String s) {
        if (s == null || !s.contains(":")) {
            return; // default policy
        }
        String value = s.split(":")[1].trim();
        String[] components = value.split(",");
        Date now = new Date();
        for (String component : components) {
            try {
                component = component.trim().toLowerCase(Locale.US);
                if (component.startsWith("max-age=")) {
                    int secondsInTheFuture = Integer.parseInt(component.substring(8));
                    maxAge = new Date(now.getTime() + 1000 * secondsInTheFuture);
                } else if (component.startsWith("s-maxage=")) {
                    int secondsInTheFuture = Integer.parseInt(component.substring(8));
                    sMaxAge = new Date(now.getTime() + 1000 * secondsInTheFuture);
                } else if (component.equals("must-revalidate")) {
                    mustRevalidate = true;
                } else if (component.equals("proxy-revalidate")) {
                    proxyRevalidate = true;
                } else if (component.equals("no-cache")) {
                    noCache = true;
                } else if (component.equals("public")) {
                    publicCache = true;
                } else if (component.equals("private")) {
                    privateCache = true;
                }
            } catch (RuntimeException ex) {
                continue;
            }
        }
    }

    public Date getMaxAge() {
        return maxAge;
    }

    public Date getSharedMaxAge() {
        return sMaxAge;
    }

    public boolean mustRevalidate() {
        return mustRevalidate;
    }

    public boolean proxyRevalidate() {
        return proxyRevalidate;
    }

    public boolean noStore() {
        return noStore;
    }

    public boolean noCache() {
        return noCache;
    }

    public boolean publicCache() {
        return publicCache;
    }

    public boolean privateCache() {
        return privateCache;
    }
}
```

A client can take advantage of this information:

- If a representation of the resource is available in the local cache, and its expiry date has not arrived, just use it. Don’t even bother talking to the server.
- If a representation of the resource is available in the local cache, but the expiry date has arrived, check the server with HEAD to see if the resource has changed before performing a full GET.

### **Web Cache for Java**

By default, Java does not cache anything. To install a system-wide cache of the ``URL`` class will use, you need the following:

- A concrete subclass of ``ResponseCache``
- A concrete subclass of ``CacheRequest``
- A concrete subclass of ``CacheResponse``

You install your subclass of ``ResponseCache`` that works with your subclass of ``CacheRequest`` and ``CacheResponse`` by passing it to the static method ``ResponseCache.setDefault()``. This installs your cache object as the system default. A ``Java virtual machine`` can only support a single shared cache.

Once a cache is installed whenever the system tries to load a new ``URL``, it will first look for it in the cache. If the cache returns the desired content, the ``URLConnection`` won’t need to connect to the remote server. However, if the requested data is not in the cache, the protocol handler will download it. After it’s done so, it will put its response into the cache so the content is more quickly available the next time that URL is loaded.

Two abstract methods in the ``ResponseCache`` class store and retrieve data from the system’s single cache:

```Java
public abstract CacheResponse get(URI uri, String requestMethod, Map<String, List<String>> requestHeaders) throws IOException
```

```Java
public abstract CacheRequest put(URI uri, URLConnection connection) throws IOException
```

The **``put()``** method returns a ``CacheRequest`` object that wraps an ``OutputStream`` into which the ``URL`` will write cacheable data it reads. CacheRequest is an abstract class with two methods, as shown

```Java
package java.net;
public abstract class CacheRequest 
{
    public abstract OutputStream getBody() throws IOException;
    public abstract void abort();
}
```
The **``getOutputStream()``** method in the subclass should return an ``OutputStream`` that points into the cache’s data store for the ``URI`` passed to the **``put()``** method at the same time. For instance, if you’re storing the data in a file, you’d return a ``FileOutputStream`` connected to that file. The protocol handler will copy the data it reads onto this ``OutputStream``. If a problem arises while copying (e.g., the server unexpectedly closes the connection), the protocol handler calls the **``abort()``** method. This method should then remove any data from the cache that has been stored for this request.

**Example**: A concrete ``CacheRequest`` subclass

```Java
import java.io.*;
import java.net.*;
public class SimpleCacheRequest extends CacheRequest {
    private ByteArrayOutputStream out = new ByteArrayOutputStream();

    @Override
    public OutputStream getBody() throws IOException {
        return out;
    }

    @Override
    public void abort() {
        out.reset();
    }

    public byte[] getData() {
        if (out.size() == 0) return null;
        else return out.toByteArray();
    }
}
```

The **``get()``** method in ``ResponseCache`` retrieves the data and headers from the cache and returns them wrapped in a ``CacheResponse`` object. It returns ``null`` if the desired ``URI`` is not in the cache, in which case the protocol handler loads the ``URI`` from the remote server as normal. Again, this is an abstract class that you have to implement in a subclass.

The CacheResponse class:

```Java
public abstract class CacheResponse {
    public abstract Map<String, List<String>> getHeaders() throws IOException;
    public abstract InputStream getBody() throws IOException;
}
```
**Example:** A concrete ``CacheResponse`` subclass

```Java
import java.io.*;
import java.net.*;
import java.util.*;

public class SimpleCacheResponse extends CacheResponse {
    private final Map<String, List<String>> headers;
    private final SimpleCacheRequest request;
    private final Date expires;
    private final CacheControl control;

    public SimpleCacheResponse(SimpleCacheRequest request, URLConnection uc, CacheControl control) throws IOException {
        this.request = request;
        this.control = control;
        this.expires = new Date(uc.getExpiration());
        this.headers = Collections.unmodifiableMap(uc.getHeaderFields());
    }

    @Override
    public InputStream getBody() {
        return new ByteArrayInputStream(request.getData());
    }

    @Override
    public Map<String, List<String>> getHeaders() throws IOException {
        return headers;
    }

    public CacheControl getControl() {
        return control;
    }

    public boolean isExpired() {
        Date now = new Date();
        if (control.getMaxAge().before(now)) return true;
        else if (expires != null && control.getMaxAge() != null) {
            return expires.before(now);
        } else {
            return false;
        }
    }
}
```
Finally, you need a simple ``ResponseCache`` subclass that stores and retrieves the cached values as requested while paying attention to the original ``Cache-control`` header

An in-memory ``ResponseCache``:

```Java
import java.io.*;
import java.net.*;
import java.util.*;
import java.util.concurrent.*;

public class MemoryCache extends ResponseCache {
    private final Map<URI, SimpleCacheResponse> responses = new ConcurrentHashMap<URI, SimpleCacheResponse>();
    private final int maxEntries;

    public MemoryCache() {
        this(100);
    }

    public MemoryCache(int maxEntries) {
        this.maxEntries = maxEntries;
    }

    @Override
    public CacheRequest put(URI uri, URLConnection conn) throws IOException {
        if (responses.size() >= maxEntries) return null;
        CacheControl control = new CacheControl(conn.getHeaderField("Cache-Control"));
        if (control.noStore()) {
            return null;
        } else if (!conn.getHeaderField(0).startsWith("GET ")) {
            // only cache GET
            return null;
        }
        SimpleCacheRequest request = new SimpleCacheRequest();
        SimpleCacheResponse response = new SimpleCacheResponse(request, conn, control);
        responses.put(uri, response);
        return request;
    }

    @Override
    public CacheResponse get(URI uri, String requestMethod, Map<String, List<String>> requestHeaders) throws IOException {
        if ("GET".equals(requestMethod)) {
            SimpleCacheResponse response = responses.get(uri);
            // check expiration date
            if (response != null && response.isExpired()) {
                responses.remove(response);
                response = null;
            }
            return response;
        } else {
            return null;
        }
    }
}
```

Java only allows one ``URL`` cache at a time. To install or change the cache, use the static **``ResponseCache.setDefault()``** and **``ResponseCache.getDefault()``** methods:

These set the single cache used by all programs running within the same ``Java virtual machine``.

## Configuring the Connection
-----------------------------
The ``URLConnection`` class has seven protected instance fields that define exactly how the client makes the request to the server. These are:

```Java
protected URL url;
protected boolean doInput = true;
protected boolean doOutput = false;
protected boolean allowUserInteraction = defaultAllowUserInteraction;
protected boolean useCaches = defaultUseCaches;
protected long ifModifiedSince = 0;
protected boolean connected = false;
```

For instance, if **``doOutput``** is ``true``, you’ll be able to write data to the server over this ``URLConnection`` as well as read data from it. If **``useCaches``** is ``false``, the connection bypasses any local caching and downloads the file from the server afresh.

Because these fields are all ``protected``, their values are accessed and modified via obviously named ``setter`` and ``getter`` methods:

```Java
public URL getURL()
public void setDoInput(boolean doInput)
public boolean getDoInput()
public void setDoOutput(boolean doOutput)
public boolean getDoOutput()
public void setAllowUserInteraction(boolean allowUserInteraction)
public boolean getAllowUserInteraction()
public void setUseCaches(boolean useCaches)
public boolean getUseCaches()
public void setIfModifiedSince(long ifModifiedSince)
public long getIfModifiedSince()
```

You can modify these fields only before the ``URLConnection`` is connected (before you try to read content or headers from the connection). Most of the methods that set fields throw an **``IllegalStateException``** if they are called while the connection is open. In general, you can set the properties of a ``URLConnection`` object only before the connection is opened.

There are also some ``getter`` and ``setter`` methods that define the default behavior for all instances of ``URLConnection``. These are:

```Java
public boolean getDefaultUseCaches()
public void setDefaultUseCaches(boolean defaultUseCaches)
public static void setDefaultAllowUserInteraction(boolean defaultAllowUserInteraction)
public static boolean getDefaultAllowUserInteraction()
public static FileNameMap getFileNameMap()
public static void setFileNameMap(FileNameMap map)
```

Unlike the instance methods, these methods can be invoked at any time. The new defaults will apply only to ``URLConnection`` objects constructed after the new default values
are set.

### **protected URL url**

The ``url`` field specifies the URL that this ``URLConnection`` connects to. The constructor sets it when the ``URLConnection`` is created and it should not change thereafter. You can retrieve the value by calling the **``getURL()``** method. 

Example : opens a ``URLConnection`` to http://www.example.com/, gets the ``URL`` of that connection, and prints it.

```Java
import java.io.*;
import java.net.*;
public class URLPrinter {
    public static void main(String[] args) {
        try {
            URL u = new URL("http://www.example.com/");
            URLConnection uc = u.openConnection();
            System.out.println(uc.getURL());
        } catch (IOException ex) {
            System.err.println(ex);
        }
    }
}
```

Here’s the result, which should be no great surprise. The ``URL`` that is printed is the one used to create the ``URLConnection``.

```
http://www.example.com/
```
### **protected boolean connected**

The ``boolean`` field connected is ``true`` if the connection is open and ``false`` if it’s closed.Because the connection has not yet been opened when a new ``URLConnection`` object is created, its initial value is ``false``. This variable can be accessed only by instances of ``java.net.URLConnection`` and its subclasses.

There are no methods that directly read or change the value of connected. However,any method that causes the ``URLConnection`` to connect should set this variable to ``true``, including **``connect()``**, **``getInputStream()``**, and **``getOutputStream()``**. Any method that causes the ``URLConnection`` to disconnect should set this field to ``false``. There are no such methods in ``java.net.URLConnection``, but some of its subclasses, such as ``java.net.HttpURLConnection``, have ``disconnect()`` methods.

If you subclass ``URLConnection`` to write a protocol handler, you are responsible for setting connected to ``true`` when you are connected and resetting it to false when the connection closes. Many methods in ``java.net.URLConnection`` read this variable to determine what they can do. If it’s set incorrectly, your program will have severe bugs that are not easy to diagnose.

### **protected boolean allowUserInteraction**

Some ``URLConnections`` need to interact with a user. For example, a web browser may need to ask for a ``username`` and ``password``. However, many applications cannot assume that a user is present to interact with it. For instance, a search engine robot is probably running in the background without any user to provide a ``username`` and ``password``. As its name suggests, the ``allowUserInteraction`` field specifies whether user interaction sis allowed. It is ``false`` by default.

This variable is ``protected``, but the **``public getAllowUserInteraction()``** method can read its value and the **``public setAllowUserInteraction()``** method can change it:

```Java
public void setAllowUserInteraction(boolean allowUserInteraction)
public boolean getAllowUserInteraction()
```
The value ``true`` indicates that user interaction is allowed; ``false`` indicates that there is no user interaction. The value may be read at any time but may be set only before the ``URLConnection`` is connected. Calling ``setAllowUserInteraction()`` when the ``URLConnection`` is connected throws an ``IllegalStateException``.

For example, this code fragment opens a connection that could ask the user for authentication if it’s required:

```Java
URL u = new URL("http://www.example.com/passwordProtectedPage.html");
URLConnection uc = u.openConnection();
uc.setAllowUserInteraction(true);
InputStream in = uc.getInputStream();
```

### **protected boolean doInput**

A ``URLConnection`` can be used for reading from a server, writing to a server, or both. The protected boolean field **``doInput``** is ``true`` if the ``URLConnection`` can be used for reading, ``false`` if it cannot be. The default is ``true``. To access this protected variable, use the public **``getDoInput()``** and **``setDoInput()``** methods:

```Java
public void setDoInput(boolean doInput)
public boolean getDoInput()
```
Example:

```Java
try {
    URL u = new URL("http://www.example.com");
    URLConnection uc = u.openConnection();
    if (!uc.getDoInput()) {
        uc.setDoInput(true);
    } 
    // read from the connection...
} catch (IOException ex) {
    System.err.println(ex);
}
```

### **protected boolean doOutput**

Programs can use a ``URLConnection`` to send output back to the server. For example, a program that needs to send data to the server using the ``POST`` method could do so by getting an output stream from a ``URLConnection``. The protected boolean field **``doOutput``** is ``true`` if the ``URLConnection`` can be used for writing, ``false`` if it cannot be; it is ``false`` by default. To access this protected variable, use the **``getDoOutput()``** and **``setDoOutput()``** methods:

```Java
public void setDoOutput(boolean dooutput)
public boolean getDoOutput()
```

Example:

```Java
try {
    URL u = new URL("http://www.example.com");
    URLConnection uc = u.openConnection();
    if (!uc.getDoOutput()) {
        uc.setDoOutput(true);
    }
    // write to the connection...
} catch (IOException ex) {
    System.err.println(ex);
}
```

### **protected boolean ifModifiedSince**

The **``ifModifiedSince``** field in the ``URLConnection`` class specifies the date ``(in milliseconds since midnight, Greenwich Mean Time, January 1, 1970)``, which will be placed in the ``If-Modified-Since`` header field. Because **``ifModifiedSince``** is protected, programs should call the **``getIfModifiedSince()``** and **``setIfModifiedSince()``** methods to read or modify it:

```java
public long getIfModifiedSince()
public void setIfModifiedSince(long ifModifiedSince)
```

Example: Set **``ifModifiedSince``** to 24 hours prior to now

```Java
import java.io.*;
import java.net.*;
import java.util.*;

public class Last24 {
    public static void main(String[] args) {
        // Initialize a Date object with the current date and time
        Date today = new Date();
        long millisecondsPerDay = 24 * 60 * 60 * 1000;
        try {
            URL u = new URL("https://www.example.com/");
            URLConnection uc = u.openConnection();
            System.out.println("Original if modified since: " + new Date(uc.getIfModifiedSince()));
            uc.setIfModifiedSince((new Date(today.getTime() - millisecondsPerDay)).getTime());
            System.out.println("Will retrieve file if it's modified since " + new Date(uc.getIfModifiedSince()));
        } catch (IOException ex) {
            System.err.println(ex);
        }
    }
}
```
Here’s the result. First, you see the default value: ``midnight, January 1, 1970, GMT``, converted to ``Pacific Standard Time``. Next, you see the new time, which you set to 24 hours prior to the current time:

```
Original if modified since: Thu Jan 01 05:30:00 NPT 1970
Will retrieve file if it's modified since Tue Jul 11 22:06:38 NPT 2023
```
### **protected boolean useCaches**

Some clients, notably web browsers, can retrieve a document from a local cache, rather than retrieving it from a server. Applets may have access to the browser’s cache. Standalone applications can use the ``java.net.ResponseCache`` class. The **``useCaches``** variable determines whether a cache will be used if it’s available. The default value is ``true``, meaning that the cache will be used; ``false`` means the cache won’t be used. Because **``useCaches``** is protected, programs access it using the **``getUseCaches()``** and **``setUseCaches()``** methods:

```Java
public void setUseCaches(boolean useCaches)
public boolean getUseCaches()
```

This code fragment disables caching to ensure that the most recent version of the document is retrieved by setting **``useCaches``** to ``false``:

```Java
try {
    URL u = new URL("http://www.example.com/");
    URLConnection uc = u.openConnection();
    uc.setUseCaches(false);
    // read the document...
} catch (IOException ex) {
    System.err.println(ex);
}
```

Two methods define the initial value of the **``useCaches``** field, **``getDefaultUseCaches()``** and **``setDefaultUseCaches()``**:

```Java
public void setDefaultUseCaches(boolean useCaches)
public boolean getDefaultUseCaches()
```

Although nonstatic, these methods do ``set`` and ``get`` a static field that determines the default behavior for all instances of the ``URLConnection`` class created after the change. The next code fragment disables caching by default; after this code runs, ``URLConnections`` that want caching must enable it explicitly using **``setUseCaches(true)``**:

```Java
if(uc.getDefaultUseCaches()){
    uc.setDefaultUseCaches(false);
}
```

### **Timeouts**

Four methods query and modify the ``timeout`` values for connections; that is, how long the underlying socket will wait for a response from the remote end before throwing a **``SocketTimeoutException``**. These are:

```Java
public void setConnectTimeout(int timeout)
public int getConnectTimeout()
public void setReadTimeout(int timeout)
public int getReadTimeout()
```
The **``setConnectTimeout()/getConnectTimeout()``** methods control how long the socket waits for the initial connection. The **``setReadTimeout()/getReadTimeout()``** methods control how long the input stream waits for data to arrive. All four methods measure timeouts in ``milliseconds``. All four interpret zero as meaning never time out. Both setter methods throw an ``IllegalArgumentException`` if the timeout is negative.

For example, this code fragment requests a ``30-second`` connect timeout and a ``45-second`` read timeout:

```Java
URL u = new URL("http://www.example.org");
URLConnuction uc = u.openConnection();
uc.setConnectTimeout(30000);
uc.setReadTimeout(45000);
```

## Configuring the Client Request HTTP Header
---------------------------------------------
An HTTP client (e.g., a browser) sends the server a request line and a header. For example, here’s an ``HTTP`` header that Chrome sends:

```
Accept:text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Charset:ISO-8859-1,utf-8;q=0.7,*;q=0.3
Accept-Encoding:gzip,deflate,sdch
Accept-Language:en-US,en;q=0.8
Cache-Control:max-age=0
Connection:keep-alive
Cookie:reddit_first=%7B%22firsttime%22%3A%20%22first%22%7D
DNT:1
Host:lesswrong.com
User-Agent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_8_3) AppleWebKit/537.31 (KHTML, like Gecko) Chrome/26.0.1410.65 Safari/537.31
```

A web server can use this information to serve different pages to different clients, to get and set cookies, to authenticate users through passwords, and more. Placing different fields in the header that the client sends and the server responds with does all of this.

Each ``URLConnection`` sets a number of different ``name-value`` pairs in the header by default.

You add headers to the HTTP header using the **``setRequestProperty()``** method before you open the connection:

```Java
public void setRequestProperty(String name, String value)
```

The **``setRequestProperty()``** method adds a field to the header of this ``URLConnection`` with a specified name and value. This method can be used only before the connection is opened. It throws an ``IllegalStateException`` if the connection is already open. The **``getRequestProperty()``** method returns the value of the named field of the HTTP header used by this ``URLConnection``.

For instance, web servers and clients store some limited persistent information with cookies. A cookie is a collection of ``name-value`` pairs. The server sends a cookie to a client using the response ``HTTP`` header. From that point forward, whenever the client requests a ``URL`` from that server, it includes a ``Cookie`` field in the ``HTTP`` request header that looks like this:

```
Cookie: username=prashant; password=ACD0X9F23JJJn6G; session=100678945
```

This particular Cookie field sends three ``name-value`` pairs to the server. There’s no limit to the number of ``name-value`` pairs that can be included in any one cookie. Given a ``URLConnection`` object ``uc``, you could add this cookie to the connection, like this:

```Java
uc.setRequestProperty("Cookie","username=prashant; password=ACD0X9F23JJJn6G; session=100678945");
```

You can set the same property to a new value, but this changes the existing property value. To add an additional property value, use the **``addRequestProperty()``** method instead:

```Java
public void addRequestProperty(String name, String value)
```

If, for some reason, you need to inspect the headers in a URLConnection, there’s a standard getter method:

```Java
public String getRequestProperty(String name)
```

``Java`` also includes a method to get all the request properties for a connection as a ``Map``:

```Java
public Map<String,List<String>> getRequestProperties()
```
The keys are the header field names. The values are lists of property values. Both names and values are stored as strings.

## Security Considerations for URLConnections
---------------------------------------------
``URLConnection`` objects are subject to all the usual security restrictions about making network connections, reading or writing files, and so forth. For instance, a ``URLConnection`` can be created by an untrusted applet only if the ``URLConnection`` is pointing to the host that the applet came from. However, the details can be a little tricky because different ``URL`` schemes and their corresponding connections can have different security implications. For example, a jar URL that points into the applet’s own jar file should be fine. However, a file URL that points to a local hard drive should not be.

Before attempting to connect a ``URL``, you may want to know whether the connection will be allowed. For this purpose, the ``URLConnection`` class has a **``getPermission()``** method:

```Java
public Permission getPermission() throws IOException
```

This returns a ``java.security.Permission`` object that specifies what permission is needed to connect to the ``URL``. It returns ``null`` if no permission is needed (e.g., there’s no security manager in place). Subclasses of ``URLConnection`` return different subclasses of ``java.security.Permission``. For instance, if the underlying ``URL`` points to ``www.gwbush.com``, **``getPermission()``** returns a ``java.net.SocketPermission`` for the host ``www.gwbush.com`` with the connect and resolve actions.

## Guessing MIME Media Types
-----------------------------
The ``URLConnection`` class provides two static methods to help programs figure out the ``MIME`` type of some data; you can use these if the content type just isn’t available or if you have reason to believe that the content type you’re given isn’t correct. The first of these is **``URLConnection.guessContentTypeFromName()``**:

```Java
public static String guessContentTypeFromName(String name)
```
This method tries to guess the content type of an object based upon the extension in the filename portion of the object’s ``URL``. It returns its best guess about the content type as a ``String``. This guess is likely to be correct; people follow some fairly regular conventions when thinking up filenames.

The second ``MIME`` type guesser method is **``URLConnection.guessContentTypeFromStream()``**:

```Java
public static String guessContentTypeFromStream(InputStream in)
```
This method tries to guess the content type by looking at the first few bytes of data in the stream. For this method to work, the ``InputStream`` must support marking so that you can return to the beginning of the stream after the first bytes have been read. Java inspects the first 16 bytes of the ``InputStream``, although sometimes fewer bytes are needed to make an identification. These guesses are often not as reliable as the guesses made by **``guessContentTypeFromName()``**. For example, an ``XML`` document that begins with a comment rather than an ``XML`` declaration would be mislabeled as an ``HTML`` file. This method should be used only as a last resort.

## HttpURLConnection
---------------------
The ``java.net.HttpURLConnection`` class is an abstract subclass of ``URLConnection``; it provides some additional methods that are helpful when working specifically with http ``URLs``. In particular, it contains methods to ``get`` and ``set`` the request method, decide whether to follow ``redirects``, get the ``response code`` and ``message``, and figure out whether a proxy server is being used. It also includes several dozen mnemonic constants matching the various HTTP response codes. Finally, it overrides the **``getPermission()``** method from the ``URLConnection`` superclass, although it doesn’t change the semantics of this method at all.

Because this class is abstract and its only constructor is protected, you can’t directly create instances of ``HttpURLConnection``. However, if you construct a ``URL`` object using an _http_ ``URL`` and invoke its **``openConnection()``** method, the ``URLConnection`` object returned will be an instance of ``HttpURLConnection``. Cast that ``URLConnection`` to ``HttpURLConnection`` like this:

```Java
URL u = new URL("http://example.com/");
URLConnection uc = u.openConnection();
HttpURLConnection http = (HttpURLConnection) uc;
```

Or, skipping a step, like this:

```Java
URL u = new URL("http://example.com/");
HttpURLConnection http = (HttpURLConnection) u.openConnection();
```

### **The Request Method**

When a web client contacts a web server, the first thing it sends is a request line. Typically, this line begins with ``GET`` and is followed by the path of the resource that the client wants to retrieve and the version of the ``HTTP`` protocol that the client understands. For example:

```
GET /student/details.html HTTP/1.0
```

However, web clients can do more than simply ``GET`` files from web servers. They can ``POST`` responses to forms. They can ``PUT`` a file on a web server or ``DELETE`` a file from a server. And they can ask for just the ``HEAD`` of a document. They can ask the web server for a list of the ``OPTIONS`` supported at a given ``URL``. They can even ``TRACE`` the request itself. All of these are accomplished by changing the request method from ``GET`` to a different keyword. For example, here’s how a browser asks for just the header of a document using ``HEAD``:

```
HEAD /student/details.html HTTP/1.1
Host: www.example.com
Accept: text/html, image/gif, image/jpeg, *; q=.2, */*; q=.2
Connection: close
```

By default, ``HttpURLConnection`` uses the ``GET`` method. However, you can change this with the **``setRequestMethod()``** method:

```Java
public void setRequestMethod(String method) throws ProtocolException
```

The method argument should be one of these seven case-sensitive strings:

- ``GET``
- ``POST``
- ``HEAD``
- ``PUT``
- ``DELETE``
- ``OPTIONS``
- ``TRACE``

If it’s some other method, then a ``java.net.ProtocolException``, a subclass of ``IOException``, is thrown. 

**Example:**
```Java
import java.io.IOException;
import java.net.HttpURLConnection;
import java.net.URL;

public class HttpRequestDemo {
    public static void main(String[] args) {
        String url = "https://www.example.com/";

        try {
            // Create a URL object
            URL apiUrl = new URL(url);

            // Open a connection to the URL
            HttpURLConnection connection = (HttpURLConnection) apiUrl.openConnection();

            // Set the request method to HEAD
            connection.setRequestMethod("HEAD");

            // Set request headers (optional)
            connection.setRequestProperty("Host", "www.example.com");
            connection.setRequestProperty("Accept", "text/html, image/gif, image/jpeg, *; q=.2, */*; q=.2");
            connection.setRequestProperty("Connection", "close");

            // Get the response code & message
            int responseCode = connection.getResponseCode();
            String responseMessage = connection.getResponseMessage();

            // Print the response code & message
            System.out.println("Response Code: " + responseCode);
            System.out.println("Response Message: " + responseMessage);

            // Close the connection
            connection.disconnect();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### **Disconnecting from the Server**

HTTP 1.1 supports persistent connections that allow multiple requests and responses to be sent over a single TCP socket. However, when Keep-Alive is used, the server won’t immediately close a connection simply because it has sent the last byte of data to the client. The client may, after all, send another request. Servers will time out and close the connection in as little as 5 seconds of inactivity. However, it’s still preferred for the client to close the connection as soon as it knows it’s done.

The ``HttpURLConnection`` class transparently supports HTTP Keep-Alive unless you explicitly turn it off. That is, it will reuse sockets if you connect to the same server again before the server has closed the connection. Once you know you’re done talking to a particular host, the **``disconnect()``** method enables a client to break the connection:

```Java
public abstract void disconnect()
```

If any streams are still open on this connection, **``disconnect()``** closes them. However, the reverse is not true. Closing a stream on a persistent connection does not close the socket and disconnect.

### **Handling Server Responses**

The first line of an ``HTTP`` server’s response includes a numeric ``code`` and a ``message`` indicating what sort of response is made. For instance, the most common response is ``200 OK``, indicating that the requested document was found. For example:

```
HTTP/1.1 200 OK
Cache-Control:max-age=3, must-revalidate
Connection:Keep-Alive
Content-Type:text/html; charset=UTF-8
Date:Sat, 04 May 2013 14:01:16 GMT
Keep-Alive:timeout=5, max=200
Server:Apache
Transfer-Encoding:chunked
Vary:Accept-Encoding,Cookie
WP-Super-Cache:Served supercache file from PHP
<HTML>
<HEAD>
rest of document follows...
```

Another response that you’re undoubtedly all too familiar with is ``404 Not Found``, indicating that the URL you requested no longer points to a document. For example:

```
HTTP/1.1 404 Not Found
```
There are many other, less common responses. For instance, code ``301`` indicates that the resource has permanently moved to a new location and the browser should redirect
itself to the new location and update any bookmarks that point to the old location. For example:

```
HTTP/1.1 301 Moved Permanently
Connection: Keep-Alive
Content-Length: 299
Content-Type: text/html; charset=iso-8859-1
Date: Sat, 04 May 2013 14:20:58 GMT
Keep-Alive: timeout=5, max=200
Location: http://www.anotherlocation.org/
Server: Apache
```

Often all you need from the ``response message`` is the numeric ``response code``. ``HttpURLConnection`` also has a **``getResponseCode()``** method to return this as an ``int``:

```Java
public int getResponseCode() throws IOException
```

The text string that follows the ``response code`` is called the ``response message`` and is returned by the aptly named **``getResponseMessage()``** method:

```Java
public String getResponseMessage() throws IOException
```

#### Error conditions

On occasion, the server encounters an error but returns useful information in the message body nonetheless. For example, when a client requests a nonexistent page from the
``www.example.org`` website, rather than simply returning a 404 error code, the server sends the search page. 

The **``getErrorStream()``** method returns an ``InputStream`` containing this page or ``null`` if no error was encountered or no data returned

```Java
public InputStream getErrorStream()
```

Generally, you’ll invoke **``getErrorStream()``** inside a catch block after **``getInputStream()``** has failed.

Example:

```Java
try (InputStream raw = uc.getInputStream()) {
    printFromStream(raw);
} catch (IOException ex) {
    printFromStream(uc.getErrorStream());
}
```
#### Redirects

The ``300-level`` response codes all indicate some sort of redirect; that is, the requested resource is no longer available at the expected location but it may be found at some other location. When encountering such a response, most browsers automatically load the document from its new location. However, this can be a security risk, because it has the potential to move the user from a trusted site to an untrusted one, perhaps without the user even noticing.

By default, an ``HttpURLConnection`` follows redirects. However, the ``HttpURLConnection`` class has two static methods that let you decide whether to follow redirects:

```Java
public static boolean getFollowRedirects()
public static void setFollowRedirects(boolean follow)
```

The **``getFollowRedirects()``** method returns ``true`` if redirects are being followed, ``false`` if they aren’t. With an argument of ``true``, the **``setFollowRedirects()``** method makes ``HttpURLConnection`` objects follow redirects. With an argument of ``false``, it prevents them from following redirects. Because these are static methods, they change the behavior of all ``HttpURLConnection`` objects constructed after the method is invoked. The **``setFollowRedirects()``** method may throw a ``SecurityException`` if the security manager disallows the change. Applets especially are not allowed to change this value.

Java has two methods to configure redirection on an instance-by-instance basis. These are:

```Java
public boolean getInstanceFollowRedirects()
public void setInstanceFollowRedirects(boolean followRedirects)
```

If **``setInstanceFollowRedirects()``** is not invoked on a given ``HttpURLConnection``, that ``HttpURLConnection`` simply follows the default behavior as set by the class method **``HttpURLConnection.setFollowRedirects()``**.

### **Proxies**

Many users behind firewalls or using AOL or other high-volume ISPs access the Web through proxy servers. The **``usingProxy()``** method tells you whether the particular ``HttpURLConnection`` is going through a proxy server:

```Java
public abstract boolean usingProxy()
```

It returns ``true`` if a proxy is being used, ``false`` if not. In some contexts, the use of a proxy server may have security implications.

### **Streaming Mode**

Every request sent to an ``HTTP server`` has an ``HTTP header``. One field in this header is the ``Content-length`` (i.e., the number of bytes in the body of the request). The header comes before the body. However, to write the header you need to know the length of the body, which you may not have yet. Normally, the way Java solves this catch-22 is by caching everything you write onto the ``OutputStream`` retrieved from the ``HttpURLConnection`` until the stream is closed. At that point, it knows how many bytes are in the body so it has enough information to write the ``Content-length`` header.

This scheme is fine for small requests sent in response to typical web forms. However,it’s burdensome for responses to very long forms or some SOAP messages. It’s very wasteful and slow for medium or large documents sent with ``HTTP PUT``. It’s much more efficient if Java doesn’t have to wait for the last byte of data to be written before sending the first byte of data over the network. Java offers two solutions to this problem. If you know the size of your data—for instance, you’re uploading a file of known size using ``HTTP PUT``—you can tell the ``HttpURLConnection`` object the size of that data. If you don’t know the size of the data in advance, you can use chunked transfer encoding instead. In chunked transfer encoding, the body of the request is sent in multiple pieces, each with its own separate content length. To turn on chunked transfer encoding, just pass the size of the chunks you want to the **``setChunkedStreamingMode()``** method before you connect the ``URL``:

```Java
public void setChunkedStreamingMode(int chunkLength)
```

If you do happen to know the size of the request data in advance, you can optimize the connection by providing this information to the ``HttpURLConnection`` object. If you do this, ``Java`` can start streaming the data over the network immediately. Otherwise, it has to cache everything you write in order to determine the ``content length``, and only send it over the network after you’ve closed the stream. If you know exactly how big your data is, pass that number to the **``setFixedLengthStreamingMode()``** method:

```Java
public void setFixedLengthStreamingMode(int contentLength)
public void setFixedLengthStreamingMode(long contentLength) // Java 7
```

> **Note** : Because this number can actually be larger than the maximum size of an ``int``, in ``Java 7`` and later you can use a ``long`` instead.

Java will use this number in the Content-length HTTP header field. However, if you then try to write more or less than the number of bytes given here, Java will throw an ``IOException``. Of course, that happens later, when you’re writing data, not when you first call this method. The **``setFixedLengthStreamingMode()``** method itself will throw an ``IllegalArgumentException`` if you pass in a negative number, or an ``IllegalStateException`` if the connection is connected or has already been set to chunked transfer encoding. (You can’t use both chunked transfer encoding and fixed-length streaming mode on the same request.)”

``Fixed-length`` streaming mode is transparent on the server side. Servers neither know nor care how the ``Content-length`` was set, as long as it’s correct. However, like chunked transfer encoding, streaming mode does interfere with authentication and redirection. If either of these is required for a given ``URL``, an ``HttpRetryException`` will be thrown; you have to manually retry. Therefore, don’t use this mode unless you really need it.