<div align="center">

[**_``Go Back``_**](../README.md)

# URLs and URIs

</div>

## URIs : URLs and Relative URLs
---------------------------------

``URI`` stands for ``Uniform Resource Identifier``. A ``Uniform Resource Identifier`` is a sequence of characters used for identification of a particular resource. It enables for the interaction of the representation of the resource over the network using specific protocols.

### **URLs and Relative URLs**

#### URLs
A ``URL``, or ``Uniform Resource Locator``, is a reference or address used to access resources on the internet. It consists of several components that specify how to locate the resource. The basic structure of a ``URL`` includes the following parts:

- **``Scheme``**: The scheme indicates the protocol or method used to access the resource. Common schemes include "http", ``"https"``, ``"ftp"``, ``"mailto"``, ``"file"`` and more.

- **``Host``**: The host is the domain name or IP address of the server where the resource is located. For example, in the URL ``"https://www.example.com"``, ``"www.example.com"`` is the host.

- **``Port``**: The port is an optional component that specifies the port number to use when connecting to the server. If not specified, the default port for the scheme is used (e.g., ``80`` for ``HTTP``, ``443`` for ``HTTPS``).

- **``Path``**: The path identifies the specific location or file on the server's file system. It represents the hierarchy of directories and subdirectories leading to the resource. For example, ``"/images/logo.png"`` is a path.

- **``Query``**: The query is an optional component that can include parameters used to send data to the server. It is typically used in dynamic web applications to pass information to a script or application.

- **``Fragment``**: The fragment is also optional and represents a specific section or anchor within the resource. It is often used in web pages to scroll to a specific part of a page.

#### Relative URLs
A ``Relative URL`` is a ``URL`` that specifies the location of a resource relative to the current URL or the base ``URL`` of a web page. ``Relative URLs`` are often used within the context of a website to link to other pages or resources within the same site. They do not include the scheme or domain, relying on the browser to determine the absolute URL based on the current context.

``Relative URLs`` can take various forms:

- **``Relative Path``**: Specifies the path to a resource relative to the current page. For example, if you are on ``"https://www.example.com/products/index.html"`` and you link to ``"images/logo.png"``, the browser resolves it as ``"https://www.example.com/products/images/logo.png"``.

- **``Parent Directory``**: You can use ``".."`` to represent the parent directory. For example, if you are on ``"https://www.example.com/products/index.html"`` and you link to ``"../about.html"``, the browser resolves it as ``"https://www.example.com/about.html"``.

- **``Root-relative Path``**: Specifies the path from the root directory of the website. It starts with a leading slash (``"/"``). For example, ``"/contact.html"`` refers to ``"https://www.example.com/contact.html"`` regardless of the current page.


## The URL Class : Creating New URLs, Retrieving Data From URL, Splitting URL into pieces, Equality & Comparison and Conversion
--------------------------------------------------------------------------------------------------------------------------------
The **``java.net.URL``** class is an abstraction of a ``Uniform Resource Locator`` such as ``http://www.mywebsite.com/`` or ``ftp://ftp.myftp.com/pub/``. It extends **``java.lang.Object``**, and it is a final class that cannot be subclassed. Rather than relying on inheritance to configure instances for different kinds of ``URLs``, it uses the strategy design pattern. Protocol handlers are the strategies, and the ``URL`` class itself forms the context through which the different strategies are selected.

Although storing a ``URL`` as a string would be trivial, it is helpful to think of ``URLs`` as objects with fields that include the scheme (a.k.a. the protocol), hostname, port, path, query string, and fragment identifier (a.k.a. the ref), each of which may be set independently. Indeed, this is almost exactly how the **``java.net.URL``** class is organized, though the details vary a little between different versions of Java.

``URLs`` are immutable. After a ``URL`` object has been constructed, its fields do not change. This has the side effect of making them thread safe.

### **Creating New URLs**

You can construct instances of **``java.net.URL``**. The constructors differ in the information they require:

```Java
public URL(String url) throws MalformedURLException
```
```Java
public URL(String protocol, String hostname, String file) throws MalformedURLException
```
```Java
public URL(String protocol, String host, int port, String file) throws MalformedURLException
```
```Java
public URL(URL base, String relative) throws MalformedURLException
```

Which constructor you use depends on the information you have and the form it’s in. All these constructors throw a ``MalformedURLException`` if you try to create a ``URL`` for an unsupported protocol or if the ``URL`` is syntactically incorrect.

Exactly which protocols are supported is implementation dependent. The only protocols that have been available in all virtual machines are http and file, and the latter is notoriously flaky. Today, Java also supports the https, jar, and ftp protocols. Some virtual machines support mailto and gopher as well as some custom protocols like doc, netdoc, systemresource, and verbatim used internally by Java.

Other than verifying that it recognizes the ``URL`` scheme, Java does not check the correctness of the ``URLs`` it constructs. The programmer is responsible for making sure that ``URLs`` created are valid. For instance, Java does not check that the hostname in an ``HTTP``
``URL`` does not contain spaces or that the query string is ``x-www-form-URL-encoded``. It does not check that a mailto ``URL`` actually contains an email address. You can create ``URLs`` for hosts that don’t exist and for hosts that do exist but that you won’t be allowed to connect to.

**Constructing a URL from a string**

The simplest ``URL`` constructor just takes an absolute ``URL`` in string form as its single argument:

```java
public URL(String url) throws MalformedURLException
```

Like all constructors, this may only be called after the new operator, and like all ``URL`` constructors, it can throw a ``MalformedURLException``. The following code constructs a ``URL`` object from a String, catching the exception that might be thrown:

```java
try {
    URL u = new URL("http://www.mywebsite.org/");
} catch (MalformedURLException ex) {
    System.err.println(ex);
}
```

**Constructing a URL from its component parts**

You can also build a ``URL`` by specifying the protocol, the hostname, and the file:

```java
public URL(String protocol, String hostname, String file) throws MalformedURLException
```

This constructor sets the port to ``-1`` so the default port for the protocol will be used. The ``file`` argument should begin with a slash and include a path, a filename, and optionally a fragment identifier. Forgetting the initial slash is a common mistake, and one that is not easy to spot. Like all ``URL`` constructors, it can throw a ``MalformedURLException``. For example:

```java
try {
    URL u = new URL("http", "www.mywebsite.org", "/home.html#intro");
} catch (MalformedURLException ex) {
    System.err.println(ex);
}
```

This creates a ``URL`` object that points to ``http://www.mywebsite.org/home.html#intro``, using the default port for the HTTP protocol (port 80). The file specification includes a reference to a named anchor. The code catches the exception that would be thrown if the virtual machine did not support the ``HTTP`` protocol. However, this shouldn’t happen in practice.

For the rare occasions when the default ``port`` isn’t correct, the next constructor lets you specify the ``port`` explicitly as an ``int``. The other arguments are the same. For example,this code fragment creates a ``URL`` object that points to ``http://www.mywebsite.org/home.html#intro``, specifying port ``8000`` explicitly:

```java
try {
    URL u = new URL("http", "www.mywebsite.org", 8000, "/home.html#intro");
} catch (MalformedURLException ex) {
    System.err.println(ex);
}
```

**Constructing relative URLs**

This constructor builds an ``absolute URL`` from a ``relative URL`` and a ``base URL``:

```java
public URL(URL base, String relative) throws MalformedURLException
```

For instance, you may be parsing an ``HTML`` document at ``http://www.mywebsite.org/home.html`` and encounter a link to a file called ``aboutus.html`` with no further qualifying information. In this case, you use the ``URL`` to the document that contains the link to provide the missing information. The constructor computes the new URL as ``http://www.mywebsite.org/aboutus.html``.

For example:
```java
try {
    URL u1 = new URL("http://www.mywebsite.org/home.html");
    URL u2 = new URL (u1, "aboutus.html");
} catch (MalformedURLException ex) {
    System.err.println(ex);
}
```
The filename is removed from the path of ``u1`` and the new filename ``aboutus.html`` is appended to make ``u2``. This constructor is particularly useful when you want to loop through a list of files that are all in the same directory. You can create a ``URL`` for the first file and then use this ``initial URL`` to create ``URL`` objects for the other files by substituting their filenames.

### **Retrieving Data From URL**

Naked ``URLs`` aren’t very exciting. What’s interesting is the data contained in the documents they point to. The ``URL`` class has several methods that retrieve data from a ``URL``:

- ``public InputStream openStream() throws IOException``: This method opens a connection to this ``URL`` and returns an ``InputStream`` for reading from that connection.
- ``public URLConnection openConnection() throws IOException``: This method returns a URLConnection instance that represents a connection to the remote object referred to by the URL.
- ``public URLConnection openConnection(Proxy proxy) throws IOException``: Same as ``openConnection()``, except that the connection will be made through the specified proxy; Protocol handlers that do not support proxing will ignore the proxy parameter and make a normal connection.
- ``public Object getContent() throws IOException``: This method gets the contents of this ``URL``.
- ``public Object getContent(Class[] classes) throws IOException``: This method gets the contents of this ``URL``.

The most basic and most commonly used of these methods is ``openStream()``, which returns an ``InputStream`` from which you can read the data. If you need more control over the download process, call ``openConnection()`` instead, which gives you a ``URLConnection`` which you can configure, and then get an ``InputStream`` from it.

Finally, you can ask the ``URL`` for its content with ``getContent()`` which may give you a more complete object such as ``String`` or an ``Image``. Then again, it may just give you an ``InputStream`` anyway.

Example: Download a web page

```java
import java.io.*;
import java.net.*;

public class SourceViewer {
    public static void main(String[] args) {
        if (args.length > 0) {
            InputStream in = null;
            try {
                // Open the URL for reading
                URL u = new URL(args[0]);
                in = u.openStream();
                // buffer the input to increase performance
                in = new BufferedInputStream(in);
                // chain the InputStream to a Reader
                Reader r = new InputStreamReader(in);
                int c;
                while ((c = r.read()) != -1) {
                    System.out.print((char) c);
                }
            } catch (MalformedURLException ex) {
                System.err.println(args[0] + " is not a parseable URL");
            } catch (IOException ex) {
                System.err.println(ex);
            } finally {
                if (in != null) {
                    try {
                        in.close();
                    } catch (IOException e) {
                        System.err.println(e);
                    }
                }
            }
        }
    }
}
```

### **Splitting URL into pieces**

``URL`` is an acronym of ``Uniform Resource Locator``. It is a pointer to locate resource in www (World Wide Web). A resource can be anything from a simple text file to any other like images, file directory etc. 

The typical ``URL`` may look like

```
http://www.example.com:80/index.html
```

The ``URL`` has the following parts:

- ``Protocol``: In this case the protocol is ``HTTP``, It can be ``HTTPS`` in some cases
- ``Hostname``: Hostname represent the address of the machine on which resource is located, in this case, ``www.example.com``
- ``Port Number``: It is an optional attribute. If not specified then it returns ``-1``. In the above case, the port number is ``80``.
- ``Resource name``: It is the name of a resource located on the given server that we want to see.

Read-only access to these parts of a ``URL`` is provided by these public methods:

- ``getAuthority()``: Returns the authority part of ``URL`` or ``null`` if empty.
- ``getDefaultPort()``: Returns the default port used.
- ``getFile()``: Returns the file name.
- ``getHost()``: Return the hostname of the URL in IPv6 format.
- ``getPath()``: Returns the path of the ``URL``, or ``null`` if empty.
- ``getPort()``: Returns the port associated with the protocol specified by the ``URL``.
- ``getDefaultPort()``: Returns the default port used for this URL’s protocol when none is specified in the ``URL``. If no default port is defined for the protocol, then ``getDefaultPort()`` returns ``-1``.
- ``getProtocol()``: Returns the protocol used by the ``URL``.
- ``getQuery()``: the Returns the query part of ``URL``. A query is a part after the ``‘?’`` in the ``URL``. Whenever logic is used to display the result, there would be a query field in the ``URL``. It is - similar to querying a database.
- ``getRef()``: Returns the reference of the ``URL`` object. Usually, the reference is the part marked by a ``‘#’`` in the URL. You can see the working example by querying anything on Google and seeing the part after ``‘#’``.

Example:
```java
import java.net.*;

public class URLSplitter {
    public static void main(String[] args) {
        String url = "https://admin@www.example.com:8080/student.html?id=788#top";
        try {
            URL u = new URL(url);
            System.out.println("The URL is " + u);
            System.out.println("The scheme is " + u.getProtocol());
            System.out.println("The user info is " + u.getUserInfo());
            String host = u.getHost();
            if (host != null) {
                int atSign = host.indexOf('@');
                if (atSign != -1) host = host.substring(atSign + 1);
                System.out.println("The host is " + host);
            } else {
                System.out.println("The host is null.");
            }
            System.out.println("The port is " + u.getPort());
            System.out.println("The path is " + u.getPath());
            System.out.println("The ref is " + u.getRef());
            System.out.println("The query string is " + u.getQuery());
        } catch (MalformedURLException ex) {
            System.err.println( url + " is not a URL I understand.");
        }
    }
}

```

Output:

```
The URL is https://admin@www.example.com:8080/student.html?id=788#top
The scheme is https
The user info is admin
The host is www.example.com
The port is 8080
The path is /student.html
The ref is top
The query string is id=788
```

### **Equality & Comparison**

The ``URL`` class contains the usual ``equals()`` and ``hashCode()`` methods. These behave almost as you’d expect. Two ``URLs`` are considered equal if and only if both ``URLs`` point to the same resource on the same ``host``, ``port``, and ``path``, with the same fragment identifier and query string. However there is one surprise here. The ``equals()`` method actually tries to resolve the host with ``DNS`` so that, for example, it can tell that ``http://www.example.org/`` and ``http://example.org/`` are the same.

On the other hand, ``equals()`` does not go so far as to actually compare the resources identified by two ``URLs``. For example, ``http://www.example.com/`` is not equal to ``http://www.example.com/index.html``; and ``http://www.example.com:80`` is not equal to ``http://www.example.com/``.

Example: creates URL objects for ``http://www.example.org/`` and ``http://example.org/`` and tells you if they’re the same using the ``equals()`` method.

```Java
import java.net.*;

public class URLequality {
    public static void main(String[] args) {
        try {
            URL u1 = new URL("http://www.example.org/");
            URL u2 = new URL("http://example.org/");
            if (u1.equals(u2)) {
                System.out.println(u1 + " is the same as " + u2);
            } else {
                System.out.println(u1 + " is not the same as " + u2);
            }
        } catch (MalformedURLException ex) {
            System.err.println(ex);
        }
    }
}
```
Output:
```
http://www.example.org/ is the same as http://example.org/
```

``URL`` does not implement Comparable.

The ``URL`` class also has a ``sameFile()`` method that checks whether two ``URLs`` point to the
same resource:

```Java
public boolean sameFile(URL other)
```
The comparison is essentially the same as with ``equals()``, ``DNS`` queries included, except that ``sameFile()`` does not consider the fragment identifier. This ``sameFile()`` returns ``true`` when comparing ``http://www.example.com/index.html#p1`` and ``http://www.example.com/index.html#q2`` while ``equals()`` would return ``false``.

Example:

```Java
import java.net.*;

public class URLsamefile {
    public static void main(String[] args) {
        try {
            URL u1 = new URL("http://www.example.com/index.html#p1");
            URL u2 = new URL("http://www.example.com/index.html#q2");
            if (u1.sameFile(u2)) {
                System.out.println(u1 + " is the same file as " + u2);
            } else {
                System.out.println(u1 + " is not the same file as " + u2);
            }
        } catch (MalformedURLException ex) {
            System.err.println(ex);
        }
    }
}
```
Output:

```
http://www.example.com/index.html#p1 is the same file as http://www.example.com/index.html#q2
```

### **Conversion**

``URL`` has three methods that convert an instance to another form: ``toString()``, ``toExternalForm()``, and ``toURI()``.

Like all good classes, ``java.net.URL`` has a ``toString()`` method. The String produced by ``toString()`` is always an absolute ``URL``, such as ``http://www.example.org/report.html``. It’s uncommon to call ``toString()`` explicitly. Print statements call to ``String()`` implicitly. Outside of print statements, it’s more proper to use ``toExternalForm()`` instead:

```java
public String toExternalForm()
```

The ``toExternalForm()`` method converts a ``URL`` object to a string that can be used in an HTML link or a web browser’s Open ``URL`` dialog.

The ``toExternalForm()`` method returns a human-readable String representing the ``URL``. It is identical to the ``toString()`` method. In fact, all the ``toString()`` method does is return ``toExternalForm()``.

Finally, the ``toURI()`` method converts a ``URL`` object to an equivalent ``URI`` object:

```Java
public URI toURI() throws URISyntaxException
```

## The URI Class : Constructing a URI, The Parts of the URI, Resolving the URIs, Equality & Comparison and String Representetion
---------------------------------------------------------------------------------------------------------------------------------

A ``URI`` is a generalization of a ``URL`` that includes not only ``Uniform Resource Locators`` but also ``Uniform Resource Names (URNs)``. Most ``URIs`` used in practice are ``URLs``, butmost specifications and standards such as ``XML`` are defined in terms of ``URIs``. In Java, ``URIs`` are represented by the ``java.net.URI`` class. This class differs from the ``java.net.URL`` class in three important way :

- The ``URI`` class is purely about identification of resources and parsing of ``URIs``. It provides no methods to retrieve a representation of the resource identified by its ``URI``.
- The ``URI`` class is more conformant to the relevant specifications than the ``URL`` class.
- A ``URI`` object can represent a relative ``URI``. The ``URL`` class absolutizes all ``URIs`` before storing them.

In brief, a ``URL`` object is a representation of an application layer protocol for network retrieval, whereas a ``URI`` object is purely for string parsing and manipulation. The ``URI`` class has no network retrieval capabilities. The ``URL`` class has some string parsing methods, such as ``getFile()`` and ``getRef()``, but many of these are broken and don’t always behave exactly as the relevant specifications say they should. Normally, you should use the ``URL`` class when you want to download the content at a ``URL`` and the ``URI`` class when you want to use the ``URL`` for identification rather than retrieval, for instance, to represent an ``XML`` namespace. When you need to do both, you may convert from a ``URI`` to a ``URL`` with the ``toURL()`` method, and from a ``URL`` to a ``URI`` using the ``toURI()`` method.

### **Constructing a URI**

``URIs`` are built from strings. You can either pass the entire ``URI`` to the constructor in a single string, or the individual pieces:

```Java
public URI(String uri) throws URISyntaxException
```
```Java
public URI(String scheme, String schemeSpecificPart, String fragment) throws URISyntaxException
```
```Java
public URI(String scheme, String host, String path, String fragment) throws URISyntaxException
```
```Java
public URI(String scheme, String authority, String path, String query, String fragment) throws URISyntaxException
```
```Java
public URI(String scheme, String userInfo, String host, int port, String path, String query, String fragment) throws URISyntaxException
```

Unlike the ``URL`` class, the ``URI`` class does not depend on an underlying protocol handler. As long as the ``URI`` is syntactically correct, ``Java`` does not need to understand its protocol in order to create a representative ``URI`` object. Thus, unlike the ``URL`` class, the ``URI`` class can be used for new and experimental ``URI`` schemes.

The first constructor creates a new ``URI`` object from any convenient string. For example:

```Java
URI voice = new URI("tel:+1-800-9988-9938");
URI web = new URI("http://www.xml.com/pub/a/2003/09/17/stax.html#id=hbc");
URI book = new URI("urn:isbn:1-565-92870-9");
```

If the string argument does not follow ``URI`` syntax rules—for example, if the URI begins with a colon—this constructor throws a ``URISyntaxException``. This is a ``checked exception``, so either catch it or declare that the method where the constructor is invoked can throw it.

The second constructor that takes a scheme specific part is mostly used for nonhierarchical ``URIs``. The scheme is the ``URI’s`` protocol, such as ``http``, ``urn``, ``tel``, and so forth. It must be composed exclusively of ``ASCII`` letters and digits and the three punctuation characters ``+``, ``-``, and ``.``. It must begin with a letter. Passing null for this argument omits the scheme, thus creating a relative ``URI``. For example:

```Java
URI absolute = new URI("http", "//www.example.org" , null);
URI relative = new URI(null, "/student/index.shtml", "today");
```

The scheme-specific part depends on the syntax of the URI scheme; it’s one thing for an ``http URL``, another for a ``mailto URL``, and something else again for a ``tel URI``. Because the ``URI`` class encodes illegal characters with percent escapes, there’s effectively no syntax error you can make in this part.

Finally, the third argument contains the ``fragment identifier``, if any. Again, characters that are forbidden in a fragment identifier are escaped automatically. Passing ``null`` for this argument simply omits the fragment identifier.

The third constructor is used for ``hierarchical URIs`` such as ``http`` and ``ftp`` URLs. The ``host`` and ``path`` together (separated by a ``/``) form the ``scheme-specific`` part for this ``URI``. For example:

```Java
URI today= new URI("http", "www.example.org", "/student/index.html", "today");
```
This produces the ``URI``:
```
http://www.example.org/student/index.html#today
```

If the constructor cannot form a legal ``hierarchical URI`` from the supplied pieces—for instance, if there is a scheme so the ``URI`` has to be absolute but the path doesn’t start with ``/``—then it throws a ``URISyntaxException``.

The fourth constructor is basically the same as the third, with the addition of a query string. For example:

```Java
URI today = new URI("http", "www.student.org", "/student/index.html", "referrer=cnet&date=2014-02-23", "today");
```

As usual, any unescapable syntax errors cause a ``URISyntaxException`` to be thrown and ``null`` can be passed to omit any of the arguments.

The fifth constructor is the master hierarchical URI constructor that the previous two invoke. It divides the authority into separate ``user info``, ``host``, and ``port`` parts, each of which has its own syntax rules. For example:

```Java
URI styles = new URI("ftp", "anonymous:prashant@example.org","ftp.example.com", 21, "/data/pdf", null, null);
```
However, the resulting ``URI`` still has to follow all the usual rules for ``URIs``; and again ``null`` can be passed for any argument to omit it from the result.

### **The Parts of the URI**

A ``URI`` reference has up to three parts: a ``scheme``, a ``scheme-specific`` part, and a ``fragment identifier``. The general format is:

```
scheme:[//[user:password@]host[:port]][/]path[?query][#fragment]
```

- **scheme**: This component lays out the specific protocols that are linked with the ``URI``. Even though ``“//”`` are required for some schemes, it is not used in others.

- **authority**: This component is made up of different components such as ``authentication part``, ``host``, and a ``port`` number preceded by a colon ``‘:’``. In the first section, the Authentication section consists of a username and password. At the same time, the second section Host can be any of the ip address. The third section port number is an optional one.

- **path**: The component which has a string that consists of the address present within the server to the particular resource.

- **query**: This component is nonhierarchical data in which the query used for finding a specific resource by a ``‘?’`` question mark from the preceding part.

- **fragment**: The component that identifies the secondary resources. It can be headings as well as subheadings present on a page etc.

Some Methods:

- ``isAbsolute()``: Returns ``true`` if this ``URI`` is absolute, otherwise ``false``. A ``URI`` is absolute if, and only if, it has a scheme component.
- ``isOpaque()`` : Returns ``true`` if this ``URI`` is opaque, otherwise false. A ``URI`` is opaque if, and only if, it is ``absolute`` and its ``scheme-specific`` part does not begin with a slash character ``‘/’``
- ``getAuthority()``: Returns the URI’s authority component, which is decoded.
- ``getFragment()``: Returns the URI’s fragment component, which is decoded.
- ``getHost()``: Returns the URI’s host component, which is decoded.
- ``getPath()``: Returns the URI’s path component, which is decoded.
- ``getPort()``: URI port number will be returned.
- ``getQuery()``: Returns the URI’s query component, which is decoded.
- ``getRawAuthority()``: Returns the URI’s raw authority component.
- ``getRawFragment()``: Returns the URI’s raw fragment component.
- ``getRawPath()``: Returns the URI’s raw path component.
- ``getRawQuery()``: Returns the URI’s raw query component.
- ``getRawSchemeSpecificPart()``: Returns the URI’s raw scheme-specific part.
- ``getRawUserInfo()``: Returns the URI’s raw user information component.
- ``getScheme()``: Returns the URI’s scheme component.
- ``getSchemeSpecificPart()``: Returns the URI’s scheme-specific part which is decoded.
- ``getUserInfo()``: Returns the URI’s user information component which is decoded.

Example:

```Java
import java.net.*;
public class URISplitter {
    public static void main(String[] args) throws URISyntaxException {
        URI u = new URI("http://admin@www.example.org:80/student/index.html?id=90#today");
        System.out.println("The URI is " + u);
        if (u.isOpaque()) {
            System.out.println("This is an opaque URI.");
            System.out.println("The scheme is " + u.getScheme());
            System.out.println("The scheme specific part is "
                    + u.getSchemeSpecificPart());
            System.out.println("The fragment ID is " + u.getFragment());
        } else {
            System.out.println("This is a hierarchical URI.");
            System.out.println("The scheme is " + u.getScheme());
            try {
                u = u.parseServerAuthority();
                System.out.println("The host is " + u.getHost());
                System.out.println("The user info is " + u.getUserInfo());
                System.out.println("The port is " + u.getPort());
            } catch (URISyntaxException ex) {
                // Must be a registry based authority
                System.out.println("The authority is " + u.getAuthority());
            }
            System.out.println("The path is " + u.getPath());
            System.out.println("The query string is " + u.getQuery());
            System.out.println("The fragment ID is " + u.getFragment());
        }
    }
}
```

Output:

```
The URI is http://admin@www.example.org:80/student/index.html?id=90#today
This is a hierarchical URI.
The scheme is http
The host is www.example.org
The user info is admin
The port is 80
The path is /student/index.html
The query string is id=90
The fragment ID is today
```

### **Resolving the URIs**

The URI class has three methods for converting back and forth between relative and absolute URIs:

- ``public URI resolve(URI uri)`` : The given ``URI`` is resolved against the ``URI``.
- ``public URI resolve(String uri)`` : A new ``URI`` is constructed by parsing the string mentioned and resolving the same against the ``URI``.
- ``public URI relativize(URI uri)`` : Given ``URI`` gets relativized to the ``URI`` specified.

The ``resolve()`` methods compare the ``uri`` argument to this ``URI`` and use it to construct a new ``URI`` object that wraps an absolute ``URI``. For example, consider these three lines of code:

```Java
URI absolute = new URI("http://www.example.com/");
URI relative = new URI("images/logo.png");
URI resolved = absolute.resolve(relative);
```

After they’ve executed, resolved contains the absolute ``URI`` ``http://www.example.com/images/logo.png``.

If the invoking ``URI`` does not contain an ``absolute URI`` itself, the ``resolve()`` method resolves as much of the ``URI`` as it can and returns a new ``relative URI`` object as a result. For example, take these three statements:

```Java
URI top = new URI("javafaq/books/");
URI resolved = top.resolve("jnp3/examples/07/index.html");
```

After they’ve executed, resolved now contains the ``relative URI`` ``javafaq/books/jnp3/examples/07/index.html`` with no scheme or authority.

It’s also possible to reverse this procedure; that is, to go from an ``absolute URI`` to a relative one. The ``relativize()`` method creates a new ``URI`` object from the uri argument that is ``relative`` to the invoking ``URI``. The argument is not changed. For example:

```Java
URI absolute = new URI("http://www.example.com/images/logo.png");
URI top = new URI("http://www.example.com/");
URI relative = top.relativize(absolute);
```
The ``URI`` object relative now contains the ``relative URI`` ``images/logo.png``.

### **Equality & Comparison**

``URIs`` are tested for equality pretty much as you’d expect. It’s not quite direct string comparison. Equal ``URIs`` must both either be ``hierarchical`` or ``opaque``. The ``scheme`` and ``authority`` parts are compared without considering case. That is, ``http`` and ``HTTP`` are the same ``scheme``, and ``www.example.com`` is the same authority as ``www.EXAMPLE.com``. The rest of the ``URI`` is case sensitive, except for hexadecimal digits used to escape illegal characters. Escapes are not decoded before comparing. ``http://www.example.com/A`` and ``http://www.example.com/%41`` are unequal ``URIs``.

The ``hashCode()`` method is consistent with equals. Equal ``URIs`` do have the same hash code and unequal ``URIs`` are fairly unlikely to share the same hash code. 

``URI`` implements ``Comparable``, and thus ``URIs`` can be ordered. The ordering is based on string comparison of the individual parts, in this sequence:

- If the schemes are different, the schemes are compared, without considering case.

- Otherwise, if the schemes are the same, a hierarchical URI is considered to be less than an opaque URI with the same scheme.

- If both URIs are opaque URIs, they’re ordered according to their scheme-specific parts.

- If both the scheme and the opaque scheme-specific parts are equal, the URIs are compared by their fragments.

- If both URIs are hierarchical, they’re ordered according to their authority components, which are themselves ordered according to user info, host, and port, in that order. Hosts are case insensitive.

- If the schemes and the authorities are equal, the path is used to distinguish them.

- If the paths are also equal, the query strings are compared.

- If the query strings are equal, the fragments are compared.

``URIs`` are not comparable to any type except themselves. Comparing a ``URI`` to anything except another ``URI`` causes a ``ClassCastException``.

### **String Representetion**

Two methods convert ``URI`` objects to strings, ``toString()`` and ``toASCIIString()``:

- ``public String toString()``: Content of the URI mentioned is returned as a string.
- ``public String toASCIIString()``: Content of the URI mentioned is returned as a ``US-ASCII`` string.

The ``toString()`` method returns an unencoded string form of the ``URI`` (i.e., characters like ``é`` and ``\`` are not percent escaped). Therefore, the result of calling this method is not guaranteed to be a syntactically correct URI, though it is in fact a syntactically correct ``IRI``. This form is sometimes useful for display to human beings, but usually not for retrieval.

The ``toASCIIString()`` method returns an encoded string form of the ``URI``. Characters like ``é`` and ``\`` are always percent escaped whether or not they were originally escaped. This is the string form of the URI you should use most of the time. Even if the form returned by ``toString()`` is more legible for humans,they may still copy and paste it into areas that are not expecting an illegal ``URI``. ``toASCIIString()`` always returns a syntactically correct ``URI``.

## x-www-form-urlencoded : URLEncoder & URLDecoder
-----------------------------------------------------
One of the challenges faced by the designers of the Web was dealing with the differences between ``operating systems``. These differences can cause problems with ``URLs``: for example, some operating systems allow spaces in filenames; some don’t. Most operating systems won’t complain about a ``#`` sign in a filename; but in a ``URL``, a ``#`` sign indicates that the filename has ended, and a fragment identifier follows. Other special characters, nonalphanumeric characters, and so on, all of which may have a special meaning inside a ``URL`` or on another operating system, present similar problems. Furthermore, Unicode was not yet ubiquitous when the Web was invented, so not all systems could handle characters such as ``é`` and ``㦻``. To solve these problems, characters used in URLs must come from a fixed subset of ``ASCII``, specifically:

- The capital letters ``A`` to ``Z``
- The lowercase letters ``a`` to ``z``
- The digits ``0`` to ``9``
- The punctuation characters ``-`` ``_`` ``.`` ``!`` ``~`` ``*`` ``'`` (and ``,``)

The characters : ``/`` ``&`` ``?`` ``@`` ``#`` ``;`` ``$`` ``+`` ``=`` and ``%`` may also be used, but only for their specified purposes. If these characters occur as part of a path or query string, they and all other characters should be encoded.

The encoding is very simple. Any characters that are not ``ASCII`` numerals, letters, or the punctuation marks specified earlier are converted into bytes and each byte is written as a percent sign followed by two hexadecimal digits. Spaces are a special case because they’re so common. Besides being encoded as ``%20``, they can be encoded as a plus sign ``(+)``. The plus sign itself is encoded as ``%2B``. The ``/`` ``#`` ``=`` ``&`` and ``?`` characters should be encoded when they are used as part of a name, and not as a separator between parts of the ``URL``.

The ``URL`` class does not ``encode`` or ``decode`` automatically. You can construct ``URL`` objects that use illegal ``ASCII`` and ``non-ASCII`` characters and/or percent escapes. Such characters and escapes are not automatically ``encoded`` or ``decoded`` when output by methods such as ``getPath()`` and ``toExternalForm()``. You are responsible for making sure all such characters are properly encoded in the strings used to construct a ``URL`` object.

Luckily, ``Java`` provides ``URLEncoder`` and ``URLDecoder`` classes to cipher strings in this format.

###  **URLEncoder**
To ``URL`` encode a string, pass the string and the character set name to the ``URLEncoder.encode()`` method. 

Syntax:

```Java
public static String encode(String s, String encoding) throws UnsupportedEncodingException
```

For example:
```Java
String encoded = URLEncoder.encode("This*string*has*asterisks", "UTF-8");
```

``URLEncoder.encode()`` returns a copy of the input string with a few changes. Any non‐alphanumeric characters are converted into ``%`` sequences (except the space, underscore, hyphen, period, and asterisk characters). It also encodes all ``non-ASCII`` characters. The space is converted into a plus sign. This method is a little overaggressive; it also converts tildes, single quotes, exclamation points, and parentheses to percent escapes, even though they don’t absolutely have to be. However, this change isn’t forbidden by the URL specification, so web browsers deal reasonably with these excessively encoded ``URLs``.

Although this method allows you to specify the character set, the only such character set you should ever pick is ``UTF-8``. ``UTF-8`` is compatible with the ``IRI`` specification, the ``URI`` class, modern web browsers, and more additional software than any other encoding you could choose.

Example:

```Java
import java.io.*;
import java.net.*;

public class URLEncodeTest {
    public static void main(String[] args) {
        try {
            System.out.println(URLEncoder.encode("This string has spaces", "UTF-8"));
            System.out.println(URLEncoder.encode("This*string*has*asterisks", "UTF-8"));
            System.out.println(URLEncoder.encode("This%string%has%percent%signs", "UTF-8"));
            System.out.println(URLEncoder.encode("This+string+has+pluses", "UTF-8"));
            System.out.println(URLEncoder.encode("This/string/has/slashes", "UTF-8"));
            System.out.println(URLEncoder.encode("This\"string\"has\"quote\"marks", "UTF-8"));
            System.out.println(URLEncoder.encode("This:string:has:colons", "UTF-8"));
            System.out.println(URLEncoder.encode("This~string~has~tildes", "UTF-8"));
            System.out.println(URLEncoder.encode("This(string)has(parentheses)", "UTF-8"));
            System.out.println(URLEncoder.encode("This.string.has.periods", "UTF-8"));
            System.out.println(URLEncoder.encode("This=string=has=equals=signs", "UTF-8"));
            System.out.println(URLEncoder.encode("This&string&has&ampersands", "UTF-8"));
            System.out.println(URLEncoder.encode("Thiséstringéhasé non - ASCII characters", "UTF-8"));
        } catch (UnsupportedEncodingException ex) {
            ex.printStackTrace();
        }
    }
}
```

Output:
```
This+string+has+spaces
This*string*has*asterisks
This%25string%25has%25percent%25signs
This%2Bstring%2Bhas%2Bpluses
This%2Fstring%2Fhas%2Fslashes
This%22string%22has%22quote%22marks
This%3Astring%3Ahas%3Acolons
This%7Estring%7Ehas%7Etildes
This%28string%29has%28parentheses%29
This.string.has.periods
This%3Dstring%3Dhas%3Dequals%3Dsigns
This%26string%26has%26ampersands
This%C3%A9string%C3%A9has%C3%A9+non+-+ASCII+characters
```

###  **URLDecoder**
The corresponding ``URLDecoder`` class has a static ``decode()`` method that decodes strings encoded in ``x-www-form-url-encoded`` format. That is, it converts all plus signs to spaces and all percent escapes to their corresponding character:

```Java
public static String decode(String s, String encoding) throws UnsupportedEncodingException
```

Example:
```Java
String decoded = URLDecoder.decode("https://www.google.com/search?hl=en&as_q=Java&as_epq=I%2FO", "UTF-8");
```

If you have any doubt about which encoding to use, pick ``UTF-8``. It’s more likely to be correct than anything else.

An ``UnsupportedEncodingException`` should be thrown if the string contains a percent sign that isn’t followed by two hexadecimal digits or decodes into an illegal sequence.

## Proxies : System Properties, The Proxy Class and The ProxySelector Class 
-----------------------------------------------------------------------------
Many systems access the Web and sometimes other ``non-HTTP`` parts of the Internet through proxy servers. A proxy server receives a request for a remote server from a local client. The proxy server makes the request to the remote server and forwards the result back to the local client. Sometimes this is done for security reasons, such as to prevent remote hosts from learning private details about the local network configuration. Other times it’s done to prevent users from accessing forbidden sites by filtering outgoing requests and limiting which sites can be viewed. For instance, an elementary school might want to block access to ``http://www.playboy.com``. And still other times it’s done purely for performance, to allow multiple users to retrieve the same popular documents from a local cache rather than making repeated downloads from the remote server.

Java programs based on the ``URL`` class can work through most common proxy servers and protocols. Indeed, this is one reason you might want to choose to use the ``URL`` class rather than rolling your own ``HTTP`` or other client on top of raw sockets.

> The ``proxy server`` is like an intermediate system between the ``client-side application`` and other servers. In an enterprise application, which is used to provide control over the user's content across network boundaries.

**Advantages of Using Proxy Servers**

The proxy servers are useful in the following cases:

- To capture the traffic between a client and server.
- To control and limit the uploading/downloading the bandwidth to discover the loading of the website with slow connections.
- To analyze the system reaction when there is trouble in your network.
- To update the content of a client/server.
- To create statistics about traffic.

### **System Properties**

Java supports proxy handlers for different protocols such as ``FTP``, ``HTTP``, ``HTTPS``, and ``SOCKs``. We can define an individual proxy for an individual handler as the hostname and port number. The following system properties are available in Java proxy configuration:

- **``proxyHost``**: It defines the hostname for the ``HTTP`` proxy server.
- **``proxyPort``**: It defines the port number for the ``HTTP`` proxy server. The port property is an optional property it will be set to defaults to ``80`` if not provided.
- **``nonProxyHosts``**: It defines a pipe-delimited ("|") for the available host patterns for which we want to bypass the proxy. It can be applied to both the ``HTTP`` and ``HTTPS`` handlers.
- **``SocksProxyHost``**: It defines the ``SOCKS`` proxy server's hostname.
- **``SocksProxyPort``**: It defines the ``SOCKS`` proxy server's port number.

**Using a Global Setting**

Java provides several system properties that we have discussed above to configure the ``JVM-wide`` behavior. These properties are easy to implement for a particular use case.

We can also set the necessary properties using the ``command line`` while invoking the ``JVM``. There is an alternative way to do so, and they can be set by calling the ``System.setProperty()`` method at runtime.

Let's understand how to set them using the command line:

#### **Set Proxy Using the Command Line**

We can also set the proxy properties using the command line arguments. To define the proxies using the command line, pass the settings as system properties as follows:

```ps
java -Dhttp.proxyHost=127.0.0.1 -Dhttp.proxyPort=3020 com.example.CommandLineProxyDemo  
```

By starting process in this way, we can use openConnection() method on the URL without doing any further effort as follows:

```java
URL url = new URL(RESOURCE_URL);  
URLConnection con = url.openConnection();  
```

#### **Set Proxy Using the ``System.setProperty()`` Method**

If we face difficulty while using the command line, there is an alternative way to do so by using the ``System.setProperty()`` method. 

```Java
System.setProperty("http.proxyHost", "127.0.0.1");  
System.setProperty("http.proxyPort", "3020");  
URL url = new URL(RESOURCE_URL);  
URLConnection con = url.openConnection();
```

Later, we can unset the system properties, and if we want, then they will be removed from our application. To unset the system property, make it ``null`` using the ``System.setProperty()`` method by defining it within our program as follows:

```Java
System.setProperty("http.proxyHost", null);
```
or
```Java
System.clearProperty("http.proxyHost");
```

### **The Proxy Class**

The Proxy class allows more fine-grained control of proxy servers from within a Java program. Specifically, it allows you to choose different proxy servers for different remote hosts. The proxies themselves are represented by instances of the ``java.net.Proxy`` class.

There are still only three kinds of proxies, HTTP, SOCKS, and DIRECT connections (no proxy at all), represented by three constants in the ``Proxy.Type`` enum:

- ``Proxy.Type.DIRECT``
- ``Proxy.Type.HTTP``
- ``Proxy.Type.SOCKS``

Besides its type, the other important piece of information about a proxy is its address and port, given as a ``SocketAddress`` object. For example, this code fragment creates a Proxy object representing an ``HTTP`` proxy server on port ``80`` of ``proxy.example.com``:

```Java
SocketAddress address = new InetSocketAddress("proxy.example.com", 80);
Proxy proxy = new Proxy(Proxy.Type.HTTP, address);
```

Although there are only three kinds of proxy objects, there can be many proxies of the same type for different proxy servers on different hosts.

### **The ProxySelector Class**

**Methods of ``ProxySelector`` class** 

- **``connectFailed()``**: This method is invoked when failed to establish a connection
- **``getDefault()``**: This method is used for retrieving the system-wide ``ProxySelector``
- **``select()``**: This method returns Proxy to access resource
- **``setDefault()``**: This method is used to set or unset the system-wide ``ProxySelector``

Each running virtual machine has a single ``java.net.ProxySelector`` object it uses to locate the proxy server for different connections. The default ``ProxySelector`` merely inspects the various system properties and the URL’s protocol to decide how to connect to different hosts. However, you can install your own subclass of ``ProxySelector`` in place of the default selector and use it to choose different proxies based on protocol, host, path, time of day, or other criteria.

The key to this class is the abstract ``select()`` method:

```Java
public abstract List<Proxy> select(URI uri)
```
Java passes this method a ``URI`` object (not a ``URL`` object) representing the host to which a connection is needed. For a connection made with the ``URL`` class, this object typically has the form ``http://www.example.com/`` or ``ftp://ftp.example.com/pub/files/``, for example. For a pure ``TCP`` connection made with the ``Socket`` class, this ``URI`` will have the form ``socket://host:port:``, for instance, ``socket://www.example.com:80``. The ``ProxySelector`` object then chooses the right proxies for this type of object and returns them in a ``List<Proxy>``.

The second abstract method in this class you must implement is ``connectFailed()``:

```Java
public abstract void connectFailed(URI uri, SocketAddress address, IOException ex)
```
This is a callback method used to warn a program that the proxy server isn’t actually making the connection.

**Example**: A ``ProxySelector`` that remembers what it can connect to

```Java
import java.io.*;
import java.net.*;
import java.util.*;
public class LocalProxySelector extends ProxySelector {
    private List<URI> failed = new ArrayList<>();
    public List<Proxy> select(URI uri) {
        List<Proxy> result = new ArrayList<Proxy>();
        if (failed.contains(uri) || !"http".equalsIgnoreCase(uri.getScheme())) {
            result.add(Proxy.NO_PROXY);
        }
        else {
            SocketAddress proxyAddress = new InetSocketAddress( "proxy.example.com", 8000);
            Proxy proxy = new Proxy(Proxy.Type.HTTP, proxyAddress);
            result.add(proxy);
        }
        return result;
    }
    public void connectFailed(URI uri, SocketAddress address, IOException ex) {
        failed.add(uri);
    }
}
```

As I said, each virtual machine has exactly one ``ProxySelector``. To change the ``ProxySelector``, pass the new selector to the ``static ProxySelector.setDefault()`` method, like so:

```Java
ProxySelector selector = new LocalProxySelector():
ProxySelector.setDefault(selector);
```
From this point forward, all connections opened by that virtual machine will ask the ``ProxySelector`` for the right proxy to use. 

Example:

``PrivateDataProxy.java``

```Java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.net.Proxy;
import java.net.ProxySelector;
import java.net.SocketAddress;
import java.net.URI;
import java.util.ArrayList;
import java.util.List;

public class PrivateDataProxy extends ProxySelector {
    private List<URI> failed = new ArrayList<>();
    private final List<Proxy> noProxy = new ArrayList<>();
    private final List<Proxy> proxies = new ArrayList<>();

    public PrivateDataProxy() {
        noProxy.add(Proxy.NO_PROXY);
        InetSocketAddress inetSocketAddress = new InetSocketAddress("secure.connection.com", 443);
        Proxy proxy = new Proxy(Proxy.Type.HTTP, inetSocketAddress);
        proxies.add(proxy);
    }

    @Override
    public List<Proxy> select(URI uri) {
        if (uri.getPath().startsWith("/confidential")) {
            return proxies;
        }
        return noProxy;
    }

    @Override
    public void connectFailed(URI uri, SocketAddress address, IOException ex) {
        failed.add(uri);
    }
}

```

``Main.java``

```Java
import java.io.IOException;
import java.net.Proxy;
import java.net.ProxySelector;
import java.net.URISyntaxException;
import java.net.URL;
import java.util.List;

public class Main {
    public static void main(String[] args) throws URISyntaxException, IOException {
        PrivateDataProxy privateDataProxy = new PrivateDataProxy();
        // The setting the system-wide proxy selector
        ProxySelector.setDefault(privateDataProxy);
        // Print the default value
        // using getDefault() method
        System.out.println("Default value: " + ProxySelector.getDefault());
        // Display message only
        System.out.println("Getting proxy for /confidential");
        // Passing the string URL
        String confidentialUrl = "https://www.download.com/confidential";
        // Now, calling the constructor of the URL class
        URL confidential = new URL(confidentialUrl);
        // Requiring an proxy for url
        List<Proxy> confidentialProxies = privateDataProxy.select(confidential.toURI());
        // Show the proxy that was selected
        System.out.println("Proxy to use : " + confidentialProxies.get(0));
        // Display message only
        System.out.println("Getting proxy for /non-confidential");
        // passing the string URL
        // Custom URL as input
        String nonConfidentialURL = "https://www.download.com/non-confidential";
        // Now, calling the constructor of the URL class
        URL nonConfidential = new URL(nonConfidentialURL);
        // Requiring an proxy for URL
        List<Proxy> nonConfidentialProxies = privateDataProxy.select(nonConfidential.toURI());
        // Display the proxy that was selected
        System.out.println("Proxy to use : " + nonConfidentialProxies.get(0));
    }
}
```
Output:

```
Default value: PrivateDataProxy@5674cd4d
Getting proxy for /confidential
Proxy to use : HTTP @ secure.connection.com/<unresolved>:443
Getting proxy for /non-confidential
Proxy to use : DIRECT
```

## Communicating the Server-Side Programs through GET 
-----------------------------------------------------
The ``URL`` class makes it easy for Java applets and applications to communicate with serverside programs such as CGIs, servlets, PHP pages, and others that use the ``GET`` method. (Server-side programs that use the ``POST`` method require the ``URLConnection`` class).All you need to know is what combination of names and values the program expects to receive.

```Java
import java.io.*;
import java.net.*;
public class Google {
    public static void main(String[] args) {
        String query = "Prashant";
        try {
            URL u = new URL("https://www.google.com/search?q=" + query);
            try (InputStream in = new BufferedInputStream(u.openStream())) {
                InputStreamReader theHTML = new InputStreamReader(in);
                int c;
                while ((c = theHTML.read()) != -1) {
                    System.out.print((char) c);
                }
            }
        } catch (MalformedURLException ex) {
            System.err.println(ex);
        } catch (IOException ex) {
            System.err.println(ex);
        }
    }
}
```

## Accessing Password-Protected Sites: The ``Authenticator`` Class, The ``PasswordAuthentication`` Class and The ``JPasswordField`` Class
------------------------------------------------------------------------------------------------------------------------------------------
Many popular sites require a ``username`` and ``password`` for access. Some sites, implement this through ``HTTP`` authentication. Others, implement it through cookies and HTML forms. Java’s ``URL`` class can access sites that use ``HTTP`` authentication, although you’ll of course need to tell it which ``username`` and ``password`` to use.

### **The ``Authenticator`` Class**

The ``java.net`` package includes an ``Authenticator`` class you can use to provide a username and password for sites that protect themselves using ``HTTP`` authentication:

```Java
public abstract class Authenticator extends Object
```

Since ``Authenticator`` is an abstract class, you must subclass it.

To make the ``URL`` class use the subclass, install it as the default authenticator by passing
it to the static ``Authenticator.setDefault()`` method:

```Java
public static void setDefault(Authenticator a)
```

For example, if you’ve written an ``Authenticator`` subclass named ``UserAuthenticator``, you’d install it like this:

```Java
Authenticator.setDefault(new UserAuthenticator());
```

You only need to do this once. From this point forward, when the ``URL`` class needs a username and password, it will ask the ``UserAuthenticator`` using the static ``Authenticator.requestPasswordAuthentication()`` method:

```Java
public static PasswordAuthentication requestPasswordAuthentication(InetAddress address, int port, String protocol, String prompt, String scheme) throws SecurityException
```

**Parameter:**

- **``address``** : ``Inetaddress`` of the site asking for authentication.
- **``port``** : ``port`` of requesting site.
- **``protocol``** : ``protocol`` used for connection.
- **``prompt``** : message for the user.
- **``scheme``** : authentication scheme.
- **Throws** :
    - **``SecurityException``** : if security manager doesn't allow setting password authentication.

The address argument is the host for which authentication is required. The port argument is the port on that host, and the protocol argument is the application layer
protocol by which the site is being accessed. The HTTP server provides the prompt. It’s typically the name of the realm for which authentication is required. (Some large web servers have multiple realms, each of which requires different usernames and passwords.) The scheme is the authentication scheme being used. (Here the word scheme is not being used as a synonym for protocol. Rather, it is an HTTP authentication scheme, typically basic.)

Untrusted applets are not allowed to ask the user for a name and password. Trusted applets can do so, but only if they possess the ``requestPasswordAuthentication`` ``NetPermission``. Otherwise, ``Authenticator.requestPasswordAuthentication()`` throws a ``SecurityException``.

The ``Authenticator`` subclass must override the ``getPasswordAuthentication()`` method. Inside this method, you collect the username and password from the user or some other source and return it as an instance of the ``java.net.PasswordAuthentication`` class:

```Java
protected PasswordAuthentication getPasswordAuthentication()
```

If you don’t want to authenticate this request, return ``null``, and Java will tell the server it doesn’t know how to authenticate the connection. If you submit an incorrect username or password, Java will call ``getPasswordAuthentication()`` again to give you another chance to provide the right data. You normally have five tries to get the username and password correct; after that, ``openStream()`` throws a ``ProtocolException``.

Usernames and passwords are cached within the same virtual machine session. Once you set the correct password for a realm, you shouldn’t be asked for it again unless you’ve explicitly deleted the password by zeroing out the char array that contains it.

You can get more details about the request by invoking any of these methods inherited from the ``Authenticator`` superclass:

- **``protected final InetAddress getRequestingSite()``**: returns the inetaddress of the site requesting authentication. 
- **``protected final int getRequestingPort()``**: returns the port of connection. 
- **``protected final String getRequestingProtocol()``**: returns the protocol requesting the connection. 
- **``protected final String getRequestingPrompt()``**: returns the message prompted by requester. 
- **``protected final String getRequestingScheme()``**: returns the scheme of the of requesting site. 
- **``protected final String getRequestingHost()``**: returns the hostname of the site requesting authentication. 
- **``protected final String getRequestingURL()``**:  returns the url of the requester. 
- **``protected Authenticator.RequestorType getRequestorType()``**: returns one of the two named constants (i.e., ``Authenticator.RequestorType.PROXY ``or ``Authenticator.RequestorType.SERVER``) to indicate whether the server or the proxy server is requesting the authentication.

Example:
```Java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.*;

class MyAuthenticator extends Authenticator {
    String username = "Admin";
    String password = "Admin@123";
    protected PasswordAuthentication getPasswordAuthentication() {
        return new PasswordAuthentication(username, password.toCharArray());
    }
}

public class AuthenticatorExample {
    public static void main(String[] args) {
        MyAuthenticator auth = new MyAuthenticator();
        String line;
        try {
            Authenticator.setDefault(new MyAuthenticator());
            URL url = new URL("www.example.com");
            HttpURLConnection connection = (HttpURLConnection) url.openConnection();
            BufferedReader in = new BufferedReader(new InputStreamReader(connection.getInputStream()));
            auth.getPasswordAuthentication();
            while ((line = in.readLine()) != null) {
                System.out.println(line);
            }
            in.close();
        } catch (MalformedURLException e) {
            System.out.println("Malformed URL : " + e.getMessage());
        } catch (IOException e) {
            System.out.println("I/O Error : " + e.getMessage());
        }
    }
}

```

### **The ``PasswordAuthenticator`` Class**

``PasswordAuthentication`` is a very simple ``final`` class that supports two read-only properties: ``username`` and ``password``. The username is a ``String``. The password is a ``char`` array so that the password can be erased when it’s no longer needed. A String would have to wait to be garbage collected before it could be erased, and even then it might still exist somewhere in memory on the local system, possibly even on disk if the block of memory that contained it had been swapped out to ``virtual memory`` at one point. Both username and password are set in the constructor:

```Java
public PasswordAuthentication(String userName, char[] password)
```
Each is accessed via a getter method:

- **``public String getUserName()``**: Returns the username.
- **``public char[] getPassword()``**: Returns the user password.

Example:
```Java
import java.net.PasswordAuthentication;

public class PasswordAuthenticationTest {
    public static void main(String[] args) {
        String username = "admin";
        char[] password = { 'a','d','m','i','n','1','2','3' };
        PasswordAuthentication adminAuthentication = new PasswordAuthentication(username, password);
        System.out.println("Username: " + adminAuthentication.getUserName());
        System.out.println("Password: " + adminAuthentication.getPassword());
        // You can get the password in normal string
        System.out.println("Password: " + String.copyValueOf(adminAuthentication.getPassword()));
    }
}
```
Output:
```
Username: admin
Password: [C@372f7a8d
Password: admin123
```

### The ``JPasswordField`` Class

One useful tool for asking users for their passwords in a more or less secure fashion is the ``JPasswordField`` component from ``Swing``:

```Java
public class JPasswordField extends JTextField
```

This lightweight component behaves almost exactly like a text field. However, anything the user types into it is echoed as an asterisk. This way, the password is safe from anyone looking over the user’s shoulder at what’s being typed on the screen.

``JPasswordField`` also stores the passwords as a ``char`` array so that when you’re done with the password you can overwrite it with zeros. It provides the ``getPassword()`` method to return this:

```Java
public char[] getPassword()
```

Example:
```Java
import javax.swing.*;

public class PasswordFieldExample extends JFrame {
    public PasswordFieldExample() {
        super("Password Field Example");
        JPanel panel = new JPanel();
        JPasswordField passwordField = new JPasswordField(10);
        panel.add(new JLabel("Enter password:"));
        panel.add(passwordField);
        JButton button = new JButton("Get Password");
        button.addActionListener(e -> {
            char[] password = passwordField.getPassword();
            System.out.println("Entered password: " + new String(password));
        });
        panel.add(button);
        getContentPane().add(panel);
        pack();
        setVisible(true);
    }

    public static void main(String[] args) {
        new PasswordFieldExample();
    }
}
```