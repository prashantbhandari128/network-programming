<div align="center">

[**_``Go Back``_**](../README.md)

# HTTP

</div>

## The Protocol : Keep Alive
-----------------------------
The **``Hypertext Transfer Protocol (HTTP)``** is an ``application layer`` protocol in the ``Internet protocol suite`` model for distributed, collaborative, hypermedia information systems.It is invented by ``Tim Berner Lee``. **``HTTP``** is the foundation of data communication for the World Wide Web, where hypertext documents include ``hyperlinks`` to other resources that the user can easily access. A typical flow over **``HTTP``** involves a client machine making a request to a server, which then sends a response message.

**``HTTP``** is the standard protocol for communication between web browsers and web servers. **``HTTP``** specifies how a ``client`` and ``server`` establish a connection, how the client requests data from the server, how the server responds to that request, and finally, how the connection is closed. **``HTTP``** connections use the ``TCP/IP`` protocol for data transfer. For each request from ``client`` to ``server``, there is a sequence of four steps:

- The client opens a ``TCP`` connection to the server on port ``80``, by default; other ports may be specified in the URL.

- The client sends a message to the server requesting the resource at a specified path. The request includes a header, and optionally (depending on the nature of the request) a blank line followed by data for the request.

- The server sends a response to the client. The response begins with a response code, followed by a header full of metadata, a blank line, and the requested document or an error message.

- The server closes the connection.

This is the basic ``HTTP 1.0`` procedure. In ``HTTP 1.1`` and later, multiple requests and responses can be sent in series over a single ``TCP`` connection. That is, ``steps 2`` and ``3`` can repeat multiple times in between ``steps 1`` and ``4``. Furthermore, in ``HTTP 1.1``, requests and responses can be sent in multiple chunks. This is more scalable.

Each ``request`` and ``response`` has the same basic form: a header line, an ``HTTP`` header containing ``metadata``, a blank line, and then a message body.

### **Keep Alive**

``HTTP`` persistent connection, also called ``HTTP keep-alive``, or ``HTTP connection reuse``, is the idea of using a single ``TCP`` connection to ``send`` and ``receive`` multiple ``HTTP requests/responses``, as opposed to opening a new connection for every single ``request/response`` pair. The newer ``HTTP/2`` protocol uses the same idea and takes it further to allow multiple concurrent ``requests/responses`` to be multiplexed over a single connection.

A client indicates that it’s willing to reuse a socket by including a ``Connection`` field in the ``HTTP`` request header with the value ``Keep-Alive``:

```
Connection: Keep-Alive
```
This will continue until either the client or the server decides that the conversation is over and in this case they omit the ``Connection:`` header from the last message sent or, better, they add the keyword ``close`` to it:

```
Connection: close
```

The ``URL`` class transparently supports ``HTTP Keep-Alive`` unless explicitly turned ``off``. That is, it will reuse a socket if you connect to the same server again before the server has closed the connection. You can control ``Java’s`` use of ``HTTP Keep-Alive`` with several system properties:

- Set **``http.keepAlive``** to ``true`` or ``false`` to enable/disable HTTP Keep-Alive. (It is enabled by default.)

- Set **``http.maxConnections``** to the number of sockets you’re willing to hold open at one time. The default is ``5``.

- Set **``http.keepAlive.remainingData``** to ``true`` to let Java clean up after abandoned connections (Java 6 or later). It is ``false`` by default.

- Set **``sun.net.http.errorstream.enableBuffering``** to ``true`` to attempt to buffer the relatively short error streams from ``400- and 500-level`` responses, so the connection can be freed up for reuse sooner. It is ``false`` by default.

- Set **``sun.net.http.errorstream.bufferSize``** to the number of ``bytes`` to use for buffering error streams. The default is ``4,096`` bytes.

- Set **``sun.net.http.errorstream.timeout``** to the number of ``milliseconds`` before timing out a read from the error stream. It is ``300`` milliseconds by default.

The defaults are reasonable, except that you probably do want to set **``sun.net.http.errorstream.enableBuffering``** to ``true`` unless you want to read the error streams from failed requests.

## HTTP Methods
----------------
The ``HTTP`` specification includes a collection of methods that are used to interact with server-side resources. There are commonly referred to as ``HTTP`` request methods or ``HTTP`` verbs and are intended to cover all possible types of interaction with resources.

While ``HTTP`` request methods typically perform different operations, there is an overlap in functionality, and depending on the task, several ``HTTP`` requests will have to be made before it is complete. There are also ``HTTP`` method properties to consider, including whether a ``HTTP`` request is safe, idempotent, or cacheable.

Some ``HTTP`` methods are:

- ``GET``
- ``POST``
- ``PUT``
- ``HEAD``
- ``DELETE``
- ``PATCH``
- ``OPTIONS``
- ``CONNECT``
- ``TRACE``

The two most common ``HTTP`` methods are: ``GET`` and ``POST``.

### ``GET`` 

``GET`` is used to request data from a specified resource. Note that the query string (name/value pairs) is sent in the URL of a ``GET`` request:

```
/test/example.php?name1=value1&name2=value2
```

- ``GET`` requests can be cached
- ``GET`` requests remain in the browser history
- ``GET`` requests can be bookmarked
- ``GET`` requests should never be used when dealing with sensitive data
- ``GET`` requests have length restrictions
- ``GET`` requests are only used to request data (not modify)

### ``POST``

``POST`` is used to send data to a server to ``create/update`` a resource.

The data sent to the server with ``POST`` is stored in the request body of the ``HTTP`` request:

```
POST /test/example.php HTTP/1.1
Host: example.com

name1=value1&name2=value2
```

- ``POST`` requests are never cached
- ``POST`` requests do not remain in the browser history
- ``POST`` requests cannot be bookmarked
- ``POST`` requests have no restrictions on data length

## The Request Body
--------------------
The ``GET`` method retrieves a representation of a resource identified by a ``URL``. The exact location of the resource you want to ``GET`` from a server is specified by the various parts of the path and query string. How different paths and query strings map to different resources is determined by the server. The ``URL`` class doesn’t really care about that. As long as it knows the ``URL``, it can download from it.

``POST`` and ``PUT`` are more complex. In these cases, the client supplies the representation of the resource, in addition to the path and the query string. The representation of the resource is sent in the body of the request, after the header. That is, it sends these four items in order:

- A starter line including the method, path and query string, and HTTP version

- An HTTP header

- A blank line (two successive carriage return/linefeed pairs)

- The body

For example, this ``POST`` request sends form data to a server:

```
POST /example/text.php HTTP 1.0
Date: Sun, 27 Apr 2013 12:32:36
Host: www.example.org
Content-type: application/x-www-form-urlencoded
Content-length: 54
username=ram+bahadur&email=ram%40example.org
```
In this example, the body contains an application/x-www-form-urlencoded data, but that’s just one possibility. In general, the body can contain arbitrary bytes. However, the ``HTTP`` header should include two fields that specify the nature of the body:

- A ``Content-length`` field that specifies how many bytes are in the body (54 in the preceding example)

- A ``Content-type`` field that specifies the MIME media type of the bytes (**``application/x-www-form-urlencoded``** in the preceeding example).

The ``application/x-www-form-urlencoded`` MIME type used in the preceding example is common because it’s how web browsers encode most form submissions. Thus it’s used by a lot of server-side programs that talk to browsers. However, it’s hardly the only possible type you can send in the body. For example, a camera uploading a picture to a photo sharing site can send ``image/jpeg``. A text editor might send ``text/html``. It’s all just bytes in the end. 

## Cookies : CookieManager and CookieStore
-------------------------------------------
Many websites use small strings of text known as cookies to store persistent client-side state between connections. Cookies are passed from server to client and back again in the HTTP headers of requests and responses. Cookies can be used by a server to indicate session IDs, shopping cart contents, login credentials, user preferences, and more. For instance, a cookie set by an online bookstore might have the value ``ISBN=0802099912&price=$34.95`` to specify a book that I’ve put in my shopping cart.

However, more likely, the value is a meaningless string such as **``ATVPDKIKX0DER``**, which identifies a particular record in a database of some kind where the real information is kept. Usually the cookie values do not contain the data but merely point to it on the server.

Cookies are limited to nonwhitespace ASCII text, and may not contain commas or semicolons.

To set a cookie in a browser, the server includes a Set-Cookie header line in the HTTP header. For example, this HTTP header sets the cookie ``“cart”`` to the value **``“ATVPDKIKX0DER”``**:

```
HTTP/1.1 200 OK
Content-type: text/html
Set-Cookie: cart=ATVPDKIKX0DER
```

If a browser makes a second request to the same server, it will send the cookie back in a Cookie line in the HTTP request header like so:

```
GET /index.html HTTP/1.1
Host: www.example.org
Cookie: cart=ATVPDKIKX0DER
Accept: text/html
```
As long as the server doesn’t reuse cookies, this enables it to track individual users and sessions across multiple, otherwise stateless, HTTP connections.

### **CookieManager**

The ``CookieManager`` class provides a precise implementation of ``CookieHandler``. This separates the storage of cookies from the policy surrounding accepting and rejecting cookies. A ``CookieManager`` is initialized with a ``CookieStore`` and a ``CookiePolicy``. The ``CookieStore`` manages storage, and the ``CookiePolicy`` object makes policy decisions on cookie acceptance/rejection.

Java 5 includes an abstract ``java.net.CookieHandler`` class that defines an API for storing and retrieving cookies. However, it does not include an implementation of that abstract class, so it requires a lot of grunt work. Java 6 fleshes this out by adding a concrete ``java.net.CookieManager`` subclass of ``CookieHandler`` that you can use. However, it is not turned on by default. Before Java will store and return cookies, you need to enable it:

```Java
CookieManager manager = new CookieManager();
CookieHandler.setDefault(manager);
```

If all you want is to receive cookies from sites and send them back to those sites, you’re done. That’s all there is to it. After installing a ``CookieManager`` with those two lines of code, Java will store any cookies sent by HTTP servers you connect to with the URL class, and will send the stored cookies back to those same servers in subsequent requests. However, you may wish to be a bit more careful about whose cookies you accept. You can do this by specifying a ``CookiePolicy``. Three policies are predefined:

- **``CookiePolicy.ACCEPT_ALL``**: All cookies allowed.
- **``CookiePolicy.ACCEPT_NONE``**: No cookies allowed.
- **``CookiePolicy.ACCEPT_ORIGINAL_SERVER``**: Only first party cookies allowed.

For example, this code fragment tells Java to block third-party cookies but accept firstparty cookies:

```Java
CookieManager manager = new CookieManager();
manager.setCookiePolicy(CookiePolicy.ACCEPT_ORIGINAL_SERVER);
CookieHandler.setDefault(manager);
```
That is, it will only accept cookies for the server that you’re talking to, not for any server on the Internet.

If you want more fine-grained control, for instance to allow cookies from some known domains but not others, you can implement the ``CookiePolicy`` interface yourself and override the **``shouldAccept()``** method:

```Java
public boolean shouldAccept(URI uri, HttpCookie cookie)
```
The methods inside the ``CookieManager`` class:

- **``getCookieStore()``** : This method will retrieve the current cookie store.
- **``setCookiePolicy(CookiePolicy cookiePolicy)``** : This method will set the cookie policy of this cookie manager.
- **``get(URI uri, Map<String, List<String>> requestHeaders)``** : This method will get all the applicable cookies from a cookie cache for the specified URI in the request header.
- **``put(URI uri, Map<String, List<String>> responseHeaders)``** : This method will set all the applicable cookies.

Example:

```Java
import java.io.IOException;
import java.net.*;
import java.util.List;
public class CookieManagerExample {
    public static void main(String[] args) throws IOException {
        String uri = "https://www.example.com/";
        CookieManager cookieManager = new CookieManager();
        CookieHandler.setDefault(cookieManager);
        CookiePolicy cookiePolicy = CookiePolicy.ACCEPT_ORIGINAL_SERVER;
        cookieManager.setCookiePolicy(cookiePolicy);
        URL url = new URL(uri);
        URLConnection connection = url.openConnection();
        connection.getContent();
        CookieStore cookieStore = cookieManager.getCookieStore();
        List<HttpCookie> cookieList = cookieStore.getCookies();
        for (HttpCookie cookie : cookieList) {
            System.out.println("Domain name is: " + cookie.getDomain());
            System.out.println("Cookie name is: " + cookie.getName());
        }
    }
}
```

### **CookieStore**

A ``CookieStore`` is an interface in Java that is a storage area for cookies. It is used to store and retrieve cookies. A ``CookieStore`` is responsible for removing ``HTTPCookie`` instances that have expired.

The ``CookieManager`` adds the cookies to the ``CookieStore`` for every incoming HTTP response by calling ``CookieStore.add()`` and retrieves the cookies from the ``CookieStore`` for every outgoing ``HTTP`` request by calling ``CookieStore.get()``.

It is sometimes necessary to put and get cookies locally. For instance, when an application quits, it can save the cookie store to disk and load those cookies again when it next starts up. You can retrieve the store in which the CookieManager saves its cookies with the ``getCookieStore()`` method:

```Java
CookieStore store = manager.getCookieStore();
```

The ``CookieStore`` class allows you to add, remove, and list cookies so you can control the cookies that are sent outside the normal flow of HTTP requests and responses:

- **``public void add(URI uri, HttpCookie cookie)``**: Adds one HTTP cookie to the store.
- **``public List<HttpCookie> get(URI uri)``**: Retrieves cookies whose domain matches the ``URI``.
- **``public List<HttpCookie> getCookies()``**: Get all cookies in ``CookieStore`` which are not expired.
- **``public List<URI> getURIs()``**: Get all ``URIs`` that identify cookies in ``CookieStore``.
- **``public boolean remove(URI uri, HttpCookie cookie)``**: Removes a cookie from ``CookieStore``.
- **``public boolean removeAll()``**: Removes all cookies in the ``CookieStore``.

Example:

```Java
import java.net.*;
import java.util.*;
public class CookieStoreExample {
    public static void main(String[] args) {
        // CookieManager and CookieStore
        CookieManager cookieManager = new CookieManager();
        CookieStore cookieStore = cookieManager.getCookieStore();
        // creating cookies and URI
        HttpCookie cookieA = new HttpCookie("First", "1");
        HttpCookie cookieB = new HttpCookie("Second", "2");
        URI uri = URI.create("https://www.example.com/");
        // Method 1 - add(URI uri, HttpCookie cookie)
        cookieStore.add(uri, cookieA);
        cookieStore.add(null, cookieB);
        System.out.println("Cookies successfully added\n");
        // Method 2 - get(URI uri)
        List cookiesWithURI = cookieStore.get(uri);
        System.out.println("Cookies associated with URI in CookieStore : " + cookiesWithURI + "\n");
        // Method 3 - getCookies()
        List cookieList = cookieStore.getCookies();
        System.out.println("Cookies in CookieStore : " + cookieList + "\n");
        // Method 4 - getURIs()
        List uriList = cookieStore.getURIs();
        System.out.println("URIs in CookieStore" + uriList + "\n");
        // Method 5 - remove(URI uri, HttpCookie cookie)
        System.out.println("Removal of Cookie : " + cookieStore.remove(uri, cookieA));
        List remainingCookieList = cookieStore.getCookies();
        System.out.println("Remaining Cookies : " + cookieList + "\n");
        // Method 6 - removeAll()
        System.out.println("Removal of all Cookies : " + cookieStore.removeAll());
        List EmptyCookieList = cookieStore.getCookies();
        System.out.println("Empty CookieStore : " + cookieList);
    }
}
```

Example:

```
Cookies successfully added

Cookies associated with URI in CookieStore : [First="1"]

Cookies in CookieStore : [First="1", Second="2"]

URIs in CookieStore[http://www.example.com]

Removal of Cookie : true
Remaining Cookies : [Second="2"]

Removal of all Cookies : true
Empty CookieStore : []
```