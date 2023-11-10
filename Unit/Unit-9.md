<div align="center">

[**_``Go Back``_**](../README.md)

# Nonblocking I/O

</div>

``Nonblocking I/O``, also known as ``Asynchronous I/O`` or ``Non-blocking sockets``, is an advanced concept in Java network programming that allows you to handle multiple network connections without blocking the execution of your program. Unlike traditional ``blocking I/O``, where each ``I/O`` operation waits until it completes, ``Nonblocking I/O`` enables your program to continue executing while waiting for ``I/O`` operations to finish.

``Nonblocking I/O`` is particularly useful in scenarios where you need to handle multiple clients simultaneously, such as in a server that serves multiple clients concurrently. By using ``Nonblocking I/O``, you can efficiently manage multiple network connections in a single thread, reducing the overhead of creating and managing multiple threads for each connection.

## An Example Server and Client
--------------------------------

### **An Example Server**
To demonstrate the basics, this example implements a simple server for the character generator protocol. When implementing a server that takes advantage of the new ``I/O APIs``, begin by calling the static factory method **``ServerSocketChannel.open()``** method to create a new ``ServerSocketChannel`` object:

```Java
ServerSocketChannel serverChannel = ServerSocketChannel.open();
```
Initially, this channel is not actually listening on any port. To bind it to a port, retrieve its ``ServerSocket`` peer object with the **``socket()``** method and then use the ``bind()`` method on that peer. For example, this code fragment binds the channel to a server socket on port ``8080``:

```Java
ServerSocket ss = serverChannel.socket();
ss.bind(new InetSocketAddress(8080));
```
In ``Java 7`` and later, you can bind directly without retrieving the underlying ``java.net.ServerSocket``:

```Java
serverChannel.bind(new InetSocketAddress(8080));
```
The server socket channel is now listening for incoming connections on port ``8080``. To accept one, call the ``accept()`` method, which returns a ``SocketChannel`` object:

```Java
SocketChannel clientChannel = serverChannel.accept();
```
On the server side, you’ll definitely want to make the client channel nonblocking to allow the server to process multiple simultaneous connections:
```Java
clientChannel.configureBlocking(false);
```
You may also want to make the ``ServerSocketChannel`` nonblocking. By default, this **``accept()``** method blocks until there’s an incoming connection, like the **``accept()``** method of ``ServerSocket``. To change this, simply call **``configureBlocking(false)``** before calling **``accept()``**:

```Java
serverChannel.configureBlocking(false);
```

A nonblocking **``accept()``** returns ``null`` almost immediately if there are no incoming connections. Be sure to check for that or you’ll get a nasty ``NullPointerException`` when trying to use the socket.

There are now two open channels: a server channel and a client channel. Both need to be processed. Both can run indefinitely. Furthermore, processing the server channel will create more open client channels. In the traditional approach, you assign each connection a thread, and the number of threads climbs rapidly as clients connect. Instead, in the new ``I/O API``, you create a ``Selector`` that enables the program to iterate over all the connections that are ready to be processed. To construct a new ``Selector``, just call the static **``Selector.open()``** factory method:

```Java
Selector selector = Selector.open();
```

Next, you need to register each channel with the selector that monitors it using the channel’s **``register()``** method. When registering, specify the operation you’re interested in using a named constant from the ``SelectionKey`` class. For the server socket, the only operation of interest is ``OP_ACCEPT``; that is, is the server socket channel ready to accept a new connection?

```Java
serverChannel.register(selector, SelectionKey.OP_ACCEPT);
```
For the client channels, you want to know something a little different—specifically, whether they’re ready to have data written onto them. For this, use the ``OP_WRITE`` key:

```Java
SelectionKey key = clientChannel.register(selector, SelectionKey.OP_WRITE);
```
Both **``register()``** methods return a SelectionKey object. However, you’re only going to need to use that key for the client channels, because there can be more than one of them. Each ``SelectionKey`` has an attachment of arbitrary Object type. This is normally used to hold an object that indicates the current state of the connection. In this case, you can store the buffer that the channel writes onto the network. Once the buffer is fully drained, you’ll refill it. Fill an array with the data that will be copied into each buffer. Rather than writing to the end of the buffer, and then rewinding to the beginning of the buffer and writing again, it’s easier just to start with two sequential copies of the data so every line is available as a contiguous sequence in the array.

```Java
byte[] rotation = new byte[95*2];
for (byte i = ' '; i <= '~'; i++) {
    rotation[i - ' '] = i;
    rotation[i + 95 - ' '] = i;
}
```
Because this array will only be read from after it’s been initialized, you can reuse it for multiple channels. However, each channel will get its own buffer filled with the contents of this array. You’ll stuff the buffer with the first ``72`` bytes of the rotation array, then add a carriage return/linefeed pair to break the line. Then you’ll flip the buffer so it’s ready for draining, and attach it to the channel’s key:

```Java
ByteBuffer buffer = ByteBuffer.allocate(74);
buffer.put(rotation, 0, 72);
buffer.put((byte) '\r');
buffer.put((byte) '\n');
buffer.flip();
key2.attach(buffer);
```
To check whether anything is ready to be acted on, call the selector’s **``select()``** method. For a long-running server, this normally goes in an infinite loop:

```Java
while (true) {
    selector.select ();
    // process selected keys...
}
```
Assuming the selector does find a ready channel, its **``selectedKeys()``** method returns a ``java.util.Set`` containing one ``SelectionKey`` object for each ready channel. Otherwise, it returns an empty set. In either case, you can loop through this with a ``java.util.Iterator``:

```Java
Set<SelectionKey> readyKeys = selector.selectedKeys();
Iterator iterator = readyKeys.iterator();
while (iterator.hasNext()) {
    SelectionKey key = iterator.next();
    // Remove key from set so we don't process it twice
    iterator.remove();
    // operate on the channel...
}
```
Removing the key from the set tells the Selector that you’ve dealt with it, and the ``Selector`` doesn’t need to keep giving it back every time you call **``select()``**. The ``Selector`` will add the channel back into the ready set when **``select()``** is called again if the channel becomes ready again. It’s really important to remove the key from the ready set here, though.

If the ready channel is the server channel, the program accepts a new socket channel and adds it to the selector. If the ready channel is a socket channel, the program writes as much of the buffer as it can onto the channel. If no channels are ready, the selector waits for one. One thread, the main thread, processes multiple simultaneous connections.

In this case, it’s easy to tell whether a client or a server channel has been selected because the server channel will only be ready for accepting and the client channels will only be ready for writing. Both of these are I/O operations, and both can throw ``IOExceptions`` for a variety of reasons, so you’ll want to wrap this all in a ``try`` block:

```Java
try {
    if (key.isAcceptable()) {
        ServerSocketChannel server = (ServerSocketChannel) key.channel();
        SocketChannel connection = server.accept();
        connection.configureBlocking(false);
        connection.register(selector, SelectionKey.OP_WRITE);
        // set up the buffer for the client...
    } else if (key.isWritable()) {
    SocketChannel client = (SocketChannel) key.channel();
    // write data to client...
    }
}
```
Writing the data onto the channel is easy. Retrieve the key’s attachment, cast it to ``ByteBuffer``, and call **``hasRemaining()``** to check whether there’s any unwritten data left in the buffer. If there is, write it. Otherwise, refill the buffer with the next line of data from the rotation array and write that.

```Java
ByteBuffer buffer = (ByteBuffer) key.attachment();
if (!buffer.hasRemaining()) {
    // Refill the buffer with the next line
    // Figure out where the last line started
    buffer.rewind();
    int first = buffer.get();
    // Increment to the next character
    buffer.rewind();
    int position = first - ' ' + 1;
    buffer.put(rotation, position, 72);
    buffer.put((byte) '\r');
    buffer.put((byte) '\n');
    buffer.flip();
}
client.write(buffer);
```

The algorithm that figures out where to grab the next line of data relies on the characters being stored in the rotation array in ASCII order. **``buffer.get()``** reads the first byte of data from the buffer. From this number you subtract the space character (32) because that’s the first character in the rotation array. This tells you which index in the array the buffer currently starts at. You add 1 to find the start of the next line and refill the buffer.

In the chargen protocol, the server never closes the connection. It waits for the client to break the socket. When this happens, an exception will be thrown. Cancel the key and close the corresponding channel:

```Java
catch (IOException ex) {
    key.cancel();
    try {
        key.channel().close();
    } catch (IOException cex) {
        // ignore
    }
}
```

**Program:**

```Java
import java.nio.*;
import java.nio.channels.*;
import java.net.*;
import java.util.*;
import java.io.IOException;
public class ChargenServer {
    public static int DEFAULT_PORT = 8080;
    public static void main(String[] args) {
        int port;
        try {
            port = Integer.parseInt(args[0]);
        } catch (RuntimeException ex) {
            port = DEFAULT_PORT;
        }
        System.out.println("Listening for connections on port " + port);
        byte[] rotation = new byte[95*2];
        for (byte i = ' '; i <= '~'; i++) {
            rotation[i -' '] = i;
            rotation[i + 95 - ' '] = i;
        }
        ServerSocketChannel serverChannel;
        Selector selector;
        try {
            serverChannel = ServerSocketChannel.open();
            ServerSocket ss = serverChannel.socket();
            InetSocketAddress address = new InetSocketAddress(port);
            ss.bind(address);
            serverChannel.configureBlocking(false);
            selector = Selector.open();
            serverChannel.register(selector, SelectionKey.OP_ACCEPT);
        } catch (IOException ex) {
            ex.printStackTrace();
            return;
        }
        while (true) {
            try {
                selector.select();
            } catch (IOException ex) {
                ex.printStackTrace();
                break;
            }
            Set<SelectionKey> readyKeys = selector.selectedKeys();
            Iterator<SelectionKey> iterator = readyKeys.iterator();
            while (iterator.hasNext()) {
                SelectionKey key = iterator.next();
                iterator.remove();
                try {
                    if (key.isAcceptable()) {
                        ServerSocketChannel server = (ServerSocketChannel) key.channel();
                        SocketChannel client = server.accept();
                        System.out.println("Accepted connection from " + client);
                        client.configureBlocking(false);
                        SelectionKey key2 = client.register(selector, SelectionKey.
                                OP_WRITE);
                        ByteBuffer buffer = ByteBuffer.allocate(74);
                        buffer.put(rotation, 0, 72);
                        buffer.put((byte) '\r');
                        buffer.put((byte) '\n');
                        buffer.flip();
                        key2.attach(buffer);
                    } else if (key.isWritable()) {
                        SocketChannel client = (SocketChannel) key.channel();
                        ByteBuffer buffer = (ByteBuffer) key.attachment();
                        if (!buffer.hasRemaining()) {
                            // Refill the buffer with the next line
                            buffer.rewind();
                            // Get the old first character
                            int first = buffer.get();
                            // Get ready to change the data in the buffer
                            buffer.rewind();
                            // Find the new first characters position in rotation
                            int position = first - ' ' + 1;
                            // copy the data from rotation into the buffer
                            buffer.put(rotation, position, 72);
                            // Store a line break at the end of the buffer
                            buffer.put((byte) '\r');
                            buffer.put((byte) '\n');
                            // Prepare the buffer for writing
                            buffer.flip();
                        }
                        client.write(buffer);
                    }
                } catch (IOException ex) {
                    key.cancel();
                    try {
                        key.channel().close();
                    }
                    catch (IOException cex) {}
                }
            }
        }
    }
}
```

### **An Example Client**

When implementing a client that takes advantage of the new ``I/O APIs``, begin by invoking the static factory method **``SocketChannel.open()``** to create a new ``java.nio.channels.SocketChannel`` object. The argument to this method is a ``java.net.SocketAddress`` object indicating the host and port to connect to. For example, this fragment connects the channel to ``localhost`` on port ``8080``:

```Java
SocketAddress address = new InetSocketAddress("localhost", 8080);
SocketChannel client = SocketChannel.open(address);
```
The channel is opened in blocking mode, so the next line of code won’t execute until the connection is established. If the connection can’t be established, an ``IOException`` is thrown.

If this were a traditional client, you’d now ask for the socket’s input and/or output streams. However, it’s not. With a channel you write directly to the channel itself. Rather than writing byte arrays, you write ``ByteBuffer`` objects. You’ve got a pretty good idea that the lines of text are ``74`` ``ASCII characters`` long (``72`` printable characters followed by a carriage return/linefeed pair) so you’ll create a ``ByteBuffer`` that has a 74-byte capacity using the static **``allocate()``** method:

```Java
ByteBuffer buffer = ByteBuffer.allocate(74);
```
Pass this ``ByteBuffer`` object to the channel’s read() method. The channel fills this buffer with the data it reads from the socket. It returns the number of bytes it successfully read and stored in the buffer:

```Java
int bytesRead = client.read(buffer);
```
By default, this will read at least one byte or return –1 to indicate the end of the data, exactly as an ``InputStream`` does. It will often read more bytes if more bytes are available to be read. Shortly you’ll see how to put this client in nonblocking mode where it will return ``0`` immediately if no bytes are available, but for the moment this code blocks just like an ``InputStream``. As you could probably guess, this method can also throw an ``IOException`` if anything goes wrong with the read.

Assuming there is some data in the buffer—that is, ``n > 0`` this data can be copied to ``System.out``. There are ways to extract a byte array from a ``ByteBuffer`` that can then be written on a traditional ``OutputStream`` such as ``System.out``. However, it’s more informative to stick with a pure, channel-based solution. Such a solution requires wrapping the ``OutputStream`` ``System.out`` in a channel using the Channels utility class, specifically, its **``newChannel()``** method:

```Java
WritableByteChannel output = Channels.newChannel(System.out);
```
You can then write the data that was read onto this output channel connected to ``System.out``. However, before you do that, you have to flip the buffer so that the output channel starts from the beginning of the data that was read rather than the end:

```Java
buffer.flip();
output.write(buffer);
```
You don’t have to tell the output channel how many bytes to write. Buffers keep track of how many bytes they contain. However, in general, the output channel is not guaranteed to write all the bytes in the buffer. In this specific case, though, it’s a blocking channel and it will either do so or throw an ``IOException``.

You shouldn’t create a new buffer for each read and write. That would kill the performance. Instead, reuse the existing buffer. You’ll need to clear the buffer before reading into it again:

```Java
buffer.clear();
```
This is a little different than flipping. Flipping leaves the data in the buffer intact, but prepares it for writing rather than reading. Clearing resets the buffer to a pristine state.

**Program:**

```Java
import java.nio.*;
import java.nio.channels.*;
import java.net.*;
import java.io.IOException;
public class ChargenClient {
    public static String HOST = "localhost";
    public static int PORT = 8080;
    public static void main(String[] args) {
        try {
            SocketAddress address = new InetSocketAddress(HOST,PORT);
            SocketChannel client = SocketChannel.open(address);
            ByteBuffer buffer = ByteBuffer.allocate(74);
            WritableByteChannel out = Channels.newChannel(System.out);
            while (client.read(buffer) != -1) {
                buffer.flip();
                out.write(buffer);
                buffer.clear();
            }
        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }
}
```

**Output:**
```
 !"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ[\]^_`abcdefg
!"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ[\]^_`abcdefgh
"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ[\]^_`abcdefghi
#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ[\]^_`abcdefghij
$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ[\]^_`abcdefghijk
%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ[\]^_`abcdefghijkl
&'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ[\]^_`abcdefghijklm
```

## Buffers
-----------
Buffers play a crucial role in non-blocking I/O (input/output). Non-blocking I/O is designed to efficiently handle multiple I/O operations concurrently without blocking the program's execution. Buffers provide a mechanism for temporarily storing data during these I/O operations.

Streams are byte-based, while channels are block-based in contrast. Channels handle blocks of data stored in buffers. This buffer-based approach simplifies data manipulation and enables reading and writing on the same object in most network programs. However, some channels might support only reading or writing due to specific limitations, throwing ``UnsupportedOperationException`` when used otherwise.

Buffers, despite their underlying implementation details varying across different systems, can be perceived as fixed-size lists of elements, typically of primitive data types. There are specific subclasses of ``Buffer`` for all of Java’s primitive data types except boolean: ``ByteBuffer``, ``CharBuffer``, ``ShortBuffer``, ``IntBuffer``, ``LongBuffer``, ``FloatBuffer``, and ``DoubleBuffer``. r. The methods in each subclass have appropriately typed return values and argument lists. For example, the ``DoubleBuffer`` class has methods to put and get doubles. The ``IntBuffer`` class has methods to put and get ints. The common Buffer superclass only provides methods that don’t need to know the type of the data the buffer contains.(The lack of primitive-aware generics really hurts here.) Network programs use ``ByteBuffer`` almost exclusively, although occasionally one program might use a view that overlays the ``ByteBuffer`` with one of the other types.

Besides its list of data, each buffer tracks four key pieces of information. All buffers have the same methods to set and get these values, regardless of the buffer’s type:

- **position**:The next location in the buffer that will be read from or written to. This starts counting at 0 and has a maximum value equal to the size of the buffer. It can be set or gotten with these two methods:

    ```Java
    public final int position()
    public final Buffer position(int newPosition)
    ``

- **capacity**:The maximum number of elements the buffer can hold. This is set when the buffer
is created and cannot be changed thereafter. It can be read with this method:

    ```Java
    public final int capacity()
    ```

- **limit**:The end of accessible data in the buffer. You cannot write or read at or past this point without changing the limit, even if the buffer has more capacity. It is set and gotten with these two methods:

    ```Java
    public final int limit()
    public final Buffer limit(int newLimit)
    ```

- **mark**:A client-specified index in the buffer. It is set at the current position by invoking the **``mark()``** method. The current position is set to the marked position by invoking **``reset()``**:

    ```Java
    public final Buffer mark()
    public final Buffer reset()
    ```
    If the position is set below an existing mark, the mark is discarded.

Unlike reading from an ``InputStream``, reading from a buffer does not actually change the buffer’s data in any way. It’s possible to set the position either forward or backward so you can start reading from a particular place in the buffer. Similarly, a program can adjust the limit to control the end of the data that will be read. Only the capacity is fixed.

The common ``Buffer`` superclass also provides a few other methods that operate by reference to these common properties.

The **``clear()``** method ``“empties”`` the buffer by setting the position to ``zero`` and the limit
to the capacity. This allows the buffer to be completely refilled:

```Java
public final Buffer clear()
```
However, the **``clear()``** method does not remove the old data from the buffer. It’s still present and could be read using absolute get methods or changing the limit and position again.

The **``rewind()``** method sets the position to ``zero``, but does not change the limit:

```Java
public final Buffer rewind()
```
This allows the buffer to be reread.

The **``flip()``** method sets the limit to the current position and the position to ``zero``:

```Java
public final Buffer flip()
```
It is called when you want to drain a buffer you’ve just filled.

Finally, there are two methods that return information about the buffer but don’t change it. The **``remaining()``** method returns the number of elements in the buffer between the current position and the limit. The **``hasRemaining()``** method returns ``true`` if the number of remaining elements is greater than ``zero``:

```Java
public final int remaining()
public final boolean hasRemaining()
```

### **Creating Buffers**

The buffer class hierarchy is based on inheritance but not really on polymorphism, at least not at the top level. You normally need to know whether you’re dealing with an ``IntBuffer`` or a ``ByteBuffer`` or a ``CharBuffer`` or something else. You write code to one of these subclasses, not to the common ``Buffer`` superclass.

Each typed buffer class has several factory methods that create implementation-specific subclasses of that type in various ways. Empty buffers are normally created by ``allocate()`` methods. Buffers that are prefilled with data are created by wrap methods. The allocate methods are often useful for input, and the wrap methods are normally used for output.

#### Allocation

The basic **``allocate()``** method simply returns a new, empty buffer with a specified fixed capacity. 

For example, these lines create ``byte`` and ``int`` buffers, each with a size of ``100``:

```Java
ByteBuffer buffer1 = ByteBuffer.allocate(100);
IntBuffer buffer2 = IntBuffer.allocate(100);
```

The cursor is positioned at the beginning of the buffer (i.e., the position is ``0``). A buffer created by **``allocate()``** will be implemented on top of a Java array, which can be accessed by the **``array()``** and **``arrayOffset()``** methods. For example, you could read a large chunk of data into a buffer using a channel and then retrieve the array from the buffer to pass to other methods:

```Java
byte[] data1 = buffer1.array();
int[] data2 = buffer2.array();
```

The **``array()``** method does expose the buffer’s private data, so use it with caution. Changes to the backing array are reflected in the buffer and vice versa. The normal pattern here is to fill the buffer with data, retrieve its backing array, and then operate on the array. This isn’t a problem as long as you don’t write to the buffer after you’ve started working with the array.

#### Direct allocation

The ByteBuffer class (but not the other buffer classes) has an additional **``allocateDirect()``** method that may not create a backing array for the buffer. The ``VM`` may implement a directly allocated ByteBuffer using direct memory access to the buffer on an Ethernet card, kernel memory, or something else. It’s not required, but it’s allowed, and this can improve performance for ``I/O`` operations. From an ``API`` perspective, **``allocateDirect()``** is used exactly like **``allocate()``**:

```Java
ByteBuffer buffer = ByteBuffer.allocateDirect(100);
```
Invoking ``array()`` and ``arrayOffset()`` on a direct buffer will throw an ``UnsupportedOperationException``. Direct buffers may be faster on some virtual machines, especially if the buffer is large (roughly a megabyte or more). However, direct buffers are more expensive to create than indirect buffers, so they should only be allocated when the buffer is expected to be around for a while. The details are highly ``VM`` dependent. As is generally true for most performance advice, you probably shouldn’t even consider using direct buffers until measurements prove performance is an issue.

#### Wrapping

If you already have an array of data that you want to output, you’ll normally wrap a buffer around it, rather than allocating a new buffer and copying its components into the buffer one at a time. For example:

```Java
byte[] data = "Some data".getBytes("UTF-8");
ByteBuffer buffer1 = ByteBuffer.wrap(data);
char[] text = "Some text".toCharArray();
CharBuffer buffer2 = CharBuffer.wrap(text);
```

Here, the buffer contains a reference to the array, which serves as its backing array. Buffers created by wrapping are never direct. Again, changes to the array are reflected in the buffer and vice versa, so don’t wrap the array until you’re finished with it.

### **Filling and Draining**

Buffers are designed for sequential access. Recall that each buffer has a curent position identified by the **``position()``** method that is somewhere between ``zero`` and the number of elements in the buffer, inclusive. The buffer’s position is incremented by one when an element is read from or written to the buffer. For example, suppose you allocate a ``CharBuffer`` with capacity 12, and fill it by putting five characters into it:

```Java
CharBuffer buffer = CharBuffer.allocate(12);
buffer.put('H');
buffer.put('e');
buffer.put('l');
buffer.put('l');
buffer.put('o');
```

The position of the buffer is now ``5``. This is called filling the buffer.

You can only fill the buffer up to its capacity. If you tried to fill it past its initially set capacity, the **``put()``** method would throw a ``BufferOverflowException``.

If you now tried to **``get()``** from the buffer, you’d get the ``null`` character (\u0000) that Java initializes ``char`` buffers with that’s found at position ``5``. Before you can read the data you wrote in out again, you need to flip the buffer:

```Java
buffer.flip();
```
This sets the limit to the position (``5`` in this example) and resets the position to ``0``, the
start of the buffer. Now you can drain it into a new string:

```Java
String result = "";
while (buffer.hasRemaining()) {
    result += buffer.get();
}
```

Each call to **``get()``** moves the position forward one. When the position reaches the limit, **``hasRemaining()``** returns ``false``. This is called draining the buffer.

Buffer classes also have absolute methods that fill and drain at specific positions within
the buffer without updating the position. For example, ``ByteBuffer`` has these two:

```Java
public abstract byte get(int index)
public abstract ByteBuffer put(int index, byte b)
```

These both throw an ``IndexOutOfBoundsException`` if you try to access a position at or past the limit of the buffer. For example, using absolute methods, you can put the same text into a buffer like this:

```Java
CharBuffer buffer = CharBuffer.allocate(12);
buffer.put(0, 'H');
buffer.put(1, 'e');
buffer.put(2, 'l');
buffer.put(3, 'l');
buffer.put(4, 'o');
```

However, you no longer need to flip before reading it out, because the absolute methods don’t change the position. Furthermore, order no longer matters. This produces the same end result:

```Java
CharBuffer buffer = CharBuffer.allocate(12);
buffer.put(1, 'e');
buffer.put(4, 'o');
buffer.put(0, 'H');
buffer.put(3, 'l');
buffer.put(2, 'l');
```

### **Bulk Methods**

Even with buffers, it’s often faster to work with blocks of data rather than filling and draining one element at a time. The different buffer classes have bulk methods that fill and drain an array of their element type.

For example, ``ByteBuffer`` has **``put()``** and **``get()``** methods that fill and drain a ``ByteBuffer`` from a preexisting byte array or subarray:

```Java
public ByteBuffer get(byte[] dst, int offset, int length)
public ByteBuffer get(byte[] dst)
public ByteBuffer put(byte[] array, int offset, int length)
public ByteBuffer put(byte[] array)
```
These **``put``** methods insert the data from the specified array or subarray, beginning at the current position. The **``get``** methods read the data into the argument array or subarray beginning at the current position. Both **``put``** and **``get``** increment the position by the length of the array or subarray. The put methods throw a ``BufferOverflowException`` if the buffer does not have sufficient space for the array or subarray. The get methods throw a ``BufferUnderflowException`` if the buffer does not have enough data remaining to fill the array or subarrray. These are runtime exceptions.

### **Data Conversion**

All data in Java ultimately resolves to ``bytes``. Any primitive data type—``int``, ``double``, ``float``, etc.—can be written as bytes. Any sequence of bytes of the right length can be interpreted as a primitive datum. For example, any sequence of four ``bytes`` corresponds to an ``int`` or a ``float`` (actually both, depending on how you want to read it). A sequence of eight bytes corresponds to a ``long`` or a ``double``. The ``ByteBuffer`` class (and only the ``ByteBuffer`` class) provides relative and absolute **``put``** methods that fill a buffer with the bytes corresponding to an argument of primitive type (except ``boolean``) and relative and absolute **``get``** methods that read the appropriate number of bytes to form a new primitive datum:

```Java
public abstract char getChar()
public abstract ByteBuffer putChar(char value)
public abstract char getChar(int index)
public abstract ByteBuffer putChar(int index, char value)
public abstract short getShort()
public abstract ByteBuffer putShort(short value)
public abstract short getShort(int index)
public abstract ByteBuffer putShort(int index, short value)
public abstract int getInt()
public abstract ByteBuffer putInt(int value)
public abstract int getInt(int index)
public abstract ByteBuffer putInt(int index, int value)
public abstract long getLong()
public abstract ByteBuffer putLong(long value)
public abstract long getLong(int index)
public abstract ByteBuffer putLong(int index, long value)
public abstract float getFloat()
public abstract ByteBuffer putFloat(float value)
public abstract float getFloat(int index)
public abstract ByteBuffer putFloat(int index, float value)
public abstract double getDouble()
public abstract ByteBuffer putDouble(double value)
public abstract double getDouble(int index)
public abstract ByteBuffer putDouble(int index, double value)
```

**Program:**
```Java
import java.nio.ByteBuffer;

public class ByteBufferExample {
    public static void main(String[] args) {
        // Create a ByteBuffer with a capacity of 16 bytes
        ByteBuffer buffer = ByteBuffer.allocate(16);

        // Put a char value into the buffer
        buffer.putChar('A');

        // Put a short value into the buffer
        buffer.putShort((short) 123);

        // Put an int value into the buffer
        buffer.putInt(456789);

        // Put a long value into the buffer
        buffer.putLong(9876543210L);

        // Rewind the buffer to prepare for reading
        buffer.rewind();

        // Get and print the char value from the buffer
        char charValue = buffer.getChar();
        System.out.println("Char value: " + charValue);

        // Get and print the short value from the buffer
        short shortValue = buffer.getShort();
        System.out.println("Short value: " + shortValue);

        // Get and print the int value from the buffer
        int intValue = buffer.getInt();
        System.out.println("Int value: " + intValue);

        // Get and print the long value from the buffer
        long longValue = buffer.getLong();
        System.out.println("Long value: " + longValue);
    }
}
```
**Output:**

```
Char value: A
Short value: 123
Int value: 456789
Long value: 9876543210
```

In the world of new I/O, these methods do the job performed by ``DataOutputStream`` and ``DataInputStream`` in traditional I/O. These methods do have an additional ability not present in ``DataOutputStream`` and ``DataInputStream``. You can choose whether to interpret the byte sequences as ``big-endian`` or ``little-endian`` ``ints``, ``floats``, ``doubles``, and so on. By default, all values are read and written as ``big endian`` (i.e., most significant byte first). The two **``order()``** methods inspect and set the buffer’s byte order using the named constants in the ``ByteOrder`` class. For example, you can change the buffer to ``little-endian`` interpretation like so:

```Java
if (buffer.order().equals(ByteOrder.BIG_ENDIAN)) {
    buffer.order(ByteOrder.LITTLE_ENDIAN);
}
```

**Example:**
Implementation of a simple integer generator server using Java's NIO (``Non-blocking I/O``) classes. This server listens for incoming connections on a specified port and responds with a sequence of increasing integers. It's a basic example of a server using non-blocking I/O to handle multiple connections efficiently.

**Program:**
```Java
import java.nio.*;
import java.nio.channels.*;
import java.net.*;
import java.util.*;
import java.io.IOException;

public class IntgenServer {
    public static int PORT = 8080;

    public static void main(String[] args) {
        System.out.println("Listening for connections on port " + PORT);
        ServerSocketChannel serverChannel;
        Selector selector;
        try {
            serverChannel = ServerSocketChannel.open();
            ServerSocket ss = serverChannel.socket();
            InetSocketAddress address = new InetSocketAddress(PORT);
            ss.bind(address);
            serverChannel.configureBlocking(false);
            selector = Selector.open();
            serverChannel.register(selector, SelectionKey.OP_ACCEPT);
        } catch (IOException ex) {
            ex.printStackTrace();
            return;
        }
        while (true) {
            try {
                selector.select();
            } catch (IOException ex) {
                ex.printStackTrace();
                break;
            }
            Set<SelectionKey> readyKeys = selector.selectedKeys();
            Iterator<SelectionKey> iterator = readyKeys.iterator();
            while (iterator.hasNext()) {
                SelectionKey key = iterator.next();
                iterator.remove();
                try {
                    if (key.isAcceptable()) {
                        ServerSocketChannel server = (ServerSocketChannel) key.channel();
                        SocketChannel client = server.accept();
                        System.out.println("Accepted connection from " + client);
                        client.configureBlocking(false);
                        SelectionKey key2 = client.register(selector, SelectionKey.OP_WRITE);
                        ByteBuffer output = ByteBuffer.allocate(4);
                        output.putInt(0);
                        output.flip();
                        key2.attach(output);
                    } else if (key.isWritable()) {
                        SocketChannel client = (SocketChannel) key.channel();
                        ByteBuffer output = (ByteBuffer) key.attachment();
                        if (!output.hasRemaining()) {
                            output.rewind();
                            int value = output.getInt();
                            output.clear();
                            output.putInt(value + 1);
                            output.flip();
                        }
                        client.write(output);
                    }
                } catch (IOException ex) {
                    key.cancel();
                    try {
                        key.channel().close();
                    } catch (IOException cex) {
                    }
                }
            }
        }
    }
}
```

Here's a breakdown of how the code works:

- The server sets up the ``ServerSocketChannel``, binds it to a specific port, configures it as non-blocking, and registers it with a Selector for the ``OP_ACCEPT`` event.

- In an infinite loop, the server enters the main event loop by calling **``selector.select()``** to wait for events (such as new connections or write availability).

- When a connection request (``OP_ACCEPT``) is received, the server accepts the connection, configures the new ``SocketChannel`` as non-blocking, and registers it with the selector for the ``OP_WRITE`` event. It also attaches a ``ByteBuffer`` initialized with an integer value of ``0``.

- When a socket becomes writable (``OP_WRITE``), the server checks if the attached ``ByteBuffer`` has remaining data. If not, it means that it has already sent an integer, so it updates the ByteBuffer with the next integer, increments it, and flips it for writing.

- The server then writes data from the ``ByteBuffer`` to the client's ``SocketChannel``.

- If there's an ``I/O exception`` while processing a client, the server cancels the associated ``SelectionKey``, closes the channel, and handles the exception gracefully.

**Output:**
```
Listening for connections on port 8080
Accepted connection from java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:33428]
Accepted connection from java.nio.channels.SocketChannel[connected local=/127.0.0.1:8080 remote=/127.0.0.1:51162]
```

### **View Buffers**

If you know the ``ByteBuffer`` read from a ``SocketChannel`` contains nothing but elements of one particular primitive data type, it may be worthwhile to create a view buffer. This is a new ``Buffer`` object of appropriate type (e.g., ``DoubleBuffer``, ``IntBuffer``, etc.), that draws its data from an underlying ``ByteBuffer`` beginning with the current position. Changes to the view buffer are reflected in the underlying buffer and vice versa. However, each buffer has its own independent limit, capacity, mark, and position. View buffers are created with one of these six methods in ``ByteBuffer``:

```Java
public abstract ShortBuffer asShortBuffer()
public abstract CharBuffer asCharBuffer()
public abstract IntBuffer asIntBuffer()
public abstract LongBuffer asLongBuffer()
public abstract FloatBuffer asFloatBuffer()
public abstract DoubleBuffer asDoubleBuffer()
```

**Example:**
This client connects to the server and reads a sequence of increasing integers sent by the server.

**Program:**
```Java
import java.nio.*;
import java.nio.channels.*;
import java.net.*;
import java.io.IOException;

public class IntgenClient {
    public static String HOST = "localhost";
    public static int PORT = 8080;

    public static void main(String[] args) {
        try {
            SocketAddress address = new InetSocketAddress(HOST, PORT);
            SocketChannel client = SocketChannel.open(address);
            ByteBuffer buffer = ByteBuffer.allocate(4);
            IntBuffer view = buffer.asIntBuffer();
            for (int expected = 0; ; expected++) {
                client.read(buffer);
                int actual = view.get();
                buffer.clear();
                view.rewind();
                if (actual != expected) {
                    System.err.println("Expected " + expected + "; was " + actual);
                    break;
                }
                System.out.println(actual);
            }
        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }
}
```

Let's break down how the code works:

- The client specifies the host and port to connect to (``HOST`` and ``PORT`` variables).

- In the main method, the client creates a ``SocketAddress`` using the specified ``host`` and ``port``.

- The client then opens a ``SocketChannel`` and connects to the server using the provided address.

- It creates a ``ByteBuffer`` with a capacity of ``4`` bytes, which will be used to read the integer data.

- It creates an ``IntBuffer`` view of the same buffer, allowing easier reading of integers from the buffer.

- The client enters an infinite loop to read integers from the server. Inside the loop:

    - It reads data from the server into the buffer using **``client.read(buffer)``**.
    - It extracts the integer value from the ``IntBuffer`` view using **``view.get()``**.
    - It clears the buffer for the next read using **``buffer.clear()``**.
    - It rewinds the ``IntBuffer`` view to the beginning using **``view.rewind()``**.
    - The client compares the received ``integer`` with the expected value. If they match, it prints the received ``integer``; otherwise, it prints an error message and breaks the loop.

**Output:**

```
2575683
2575684
2575685
2575686
2575687
2575688
2575689
```

### **Compacting Buffers**
In Java, a compact operation on a ``ByteBuffer`` is used to reorganize the buffer's data by discarding any processed data and shifting the remaining data to the beginning of the buffer. This is particularly useful when you've read some data from a buffer and want to make space for new incoming data.

The **``compact()``** method is a part of the ``ByteBuffer`` class in the ``java.nio`` package. It works as follows:

- It copies any unread data from the current position to the end of the buffer to the beginning of the buffer.

- It updates the position to the point after the last copied byte.

- It sets the limit to the capacity of the buffer, effectively making the buffer ready for new data.

Most writable buffers support a **``compact()``** method :

```Java
public abstract ByteBuffer compact()
public abstract IntBuffer compact()
public abstract ShortBuffer compact()
public abstract FloatBuffer compact()
public abstract CharBuffer compact()
public abstract DoubleBuffer compact()
```

**Example:**
Implementation of a simple echo server using Java's NIO (``Non-blocking I/O``) classes. This server listens for incoming connections, reads data from connected clients, and echoes the received data back to the clients. Let's break down how the code works:

**Program:**
```Java
import java.nio.*;
import java.nio.channels.*;
import java.net.*;
import java.util.*;
import java.io.IOException;

public class EchoServer {
    public static int PORT = 8080;

    public static void main(String[] args) {
        System.out.println("Listening for connections on port " + PORT);
        ServerSocketChannel serverChannel;
        Selector selector;
        try {
            serverChannel = ServerSocketChannel.open();
            ServerSocket ss = serverChannel.socket();
            InetSocketAddress address = new InetSocketAddress(PORT);
            ss.bind(address);
            serverChannel.configureBlocking(false);
            selector = Selector.open();
            serverChannel.register(selector, SelectionKey.OP_ACCEPT);
        } catch (IOException ex) {
            ex.printStackTrace();
            return;
        }
        while (true) {
            try {
                selector.select();
            } catch (IOException ex) {
                ex.printStackTrace();
                break;
            }
            Set<SelectionKey> readyKeys = selector.selectedKeys();
            Iterator<SelectionKey> iterator = readyKeys.iterator();
            while (iterator.hasNext()) {
                SelectionKey key = iterator.next();
                iterator.remove();
                try {
                    if (key.isAcceptable()) {
                        ServerSocketChannel server = (ServerSocketChannel) key.channel();
                        SocketChannel client = server.accept();
                        System.out.println("Accepted connection from " + client);
                        client.configureBlocking(false);
                        SelectionKey clientKey = client.register(selector, SelectionKey.OP_WRITE | SelectionKey.OP_READ);
                        ByteBuffer buffer = ByteBuffer.allocate(100);
                        clientKey.attach(buffer);
                    }
                    if (key.isReadable()) {
                        SocketChannel client = (SocketChannel) key.channel();
                        ByteBuffer output = (ByteBuffer) key.attachment();
                        client.read(output);
                    }
                    if (key.isWritable()) {
                        SocketChannel client = (SocketChannel) key.channel();
                        ByteBuffer output = (ByteBuffer) key.attachment();
                        output.flip();
                        client.write(output);
                        output.compact();
                    }
                } catch (IOException ex) {
                    key.cancel();
                    try {
                        key.channel().close();
                    } catch (IOException cex) {
                    }
                }
            }
        }
    }
}
```

Let's break down how the code works:

- The server sets up a ``ServerSocketChannel``, binds it to a specified port, configures it as non-blocking, and registers it with a Selector for the ``OP_ACCEPT`` event.

- In an infinite loop, the server enters the main event loop by calling **``selector.select()``** to wait for events (such as new connections, read availability, or write availability).

- Inside the loop, the server processes the selected keys and events. If the key is acceptable, the server accepts the connection, configures the new ``SocketChannel`` as non-blocking, registers it for both read and write events, and attaches a ``ByteBuffer`` to hold the client's data.

- If the key is readable, the server reads data from the client's ``SocketChannel`` into the attached ``ByteBuffer``.

- If the key is writable, the server writes data from the attached ``ByteBuffer`` back to the client's ``SocketChannel``. The ``ByteBuffer`` is flipped before writing and compacted after writing to manage the data properly.

- If there's an ``I/O`` exception while processing a client, the server cancels the associated ``SelectionKey``, closes the channel, and handles the exception gracefully.

### **Duplicating Buffers**

It’s often desirable to make a copy of a buffer to deliver the same information to two or more channels. The **``duplicate()``** methods in each of the six typed buffer classes do this:

```Java
public abstract ByteBuffer duplicate()
public abstract IntBuffer duplicate()
public abstract ShortBuffer duplicate()
public abstract FloatBuffer duplicate()
public abstract CharBuffer duplicate()
public abstract DoubleBuffer duplicate()
```

The return values are not clones. The duplicated buffers share the same data, including the same backing array if the buffer is indirect. Changes to the data in one buffer are reflected in the other buffer. Thus, you should mostly use this method when you’re only going to read from the buffers. Otherwise, it can be tricky to keep track of where the data is being modified.

The original and duplicated buffers do have independent marks, limits, and positions even though they share the same data. One buffer can be ahead of or behind the other buffer.

Duplication is useful when you want to transmit the same data over multiple channels, roughly in parallel. You can make duplicates of the main buffer for each channel and allow each channel to run at its own speed.

### **Slicing Buffers**

Slicing a buffer is a slight variant of duplicating. Slicing also creates a new buffer that shares data with the old buffer. However, the slice’s zero position is the current position of the original buffer, and its capacity only goes up to the source buffer’s limit. That is, the slice is a subsequence of the original buffer that only contains the elements from the current position to the limit. Rewinding the slice only moves it back to the position of the original buffer when the slice was created. The slice can’t see anything in the original buffer before that point. Again, there are separate **``slice()``** methods in each of the six typed buffer classes:

```Java
public abstract ByteBuffer slice()
public abstract IntBuffer slice()
public abstract ShortBuffer slice()
public abstract FloatBuffer slice()
public abstract CharBuffer slice()
public abstract DoubleBuffer slice()
```

This is useful when you have a long buffer of data that is easily divided into multiple parts such as a protocol header followed by the data. You can read out the header, then slice the buffer and pass the new buffer containing only the data to a separate method or class.

### **Marking and Resetting**

Like input streams, buffers can be marked and reset if you want to reread some data. Unlike input streams, this can be done to all buffers, not just some of them. For a change, the relevant methods are declared once in the ``Buffer`` superclass and inherited by all the various subclasses:

```Java
public final Buffer mark()
public final Buffer reset()
```
The **``reset()``** method throws an ``InvalidMarkException``, a runtime exception, if the mark is not set. The mark is also unset when the position is set to a point before the mark.

### **Object Methods**

The buffer classes all provide the usual **``equals()``**, **``hashCode()``**, and **``toString()``** methods. They also implement Comparable, and therefore provide **``compareTo()``** methods.

However, buffers are not ``Serializable`` or ``Cloneable``. Two buffers are considered to be equal if:

- They have the same type (e.g., a ``ByteBuffer`` is never equal to an ``IntBuffer`` but
may be equal to another ``ByteBuffer``).
- They have the same number of elements remaining in the buffer.
- The remaining elements at the same relative positions are equal to each other.

Note that equality does not consider the buffers’ elements that precede the position, the buffers’ capacity, limits, or marks. For example, this code fragment prints ``true`` even though the first buffer is twice the size of the second:

```Java
CharBuffer buffer1 = CharBuffer.wrap("12345678");
CharBuffer buffer2 = CharBuffer.wrap("5678");
buffer1.get();
buffer1.get();
buffer1.get();
buffer1.get();
System.out.println(buffer1.equals(buffer2));
```

The ``hashCode()`` method is implemented in accordance with the contract for equality. That is, two equal buffers will have equal hash codes and two unequal buffers are very unlikely to have equal hash codes. However, because the buffer’s hash code changes every time an element is added to or removed from the buffer, buffers do not make good hash table keys.

Comparison is implemented by comparing the remaining elements in each buffer, one by one. If all the corresponding elements are equal, the buffers are equal. Otherwise, the result is the outcome of comparing the first pair of unequal elements. If one buffer runs out of elements before an unequal element is found and the other buffer still has elements, the shorter buffer is considered to be less than the longer buffer.

The ``toString()`` method returns strings that look something like this:

```
java.nio.HeapByteBuffer[pos=0 lim=62 cap=62]
```

These are primarily useful for debugging. The notable exception is ``CharBuffer``, which returns a string containing the remaining chars in the buffer.

## Channels
-------------

Channels move blocks of data into and out of buffers to and from various I/O sources such as files, sockets, datagrams, and so forth. The channel class hierarchy is rather convoluted, with multiple interfaces and many optional operations. However, for purposes of network programming there are only three really important channel classes, ``SocketChannel``, ``ServerSocketChannel``, and ``DatagramChannel``; and for the ``TCP`` connections we’ve talked about so far you only need the first two.

### **SocketChannel**

The ``SocketChannel`` class reads from and writes to TCP sockets. The data must be encoded in ``ByteBuffer`` objects for reading and writing. Each ``SocketChannel`` is associated with a peer ``Socket`` object that can be used for advanced configuration, but this requirement can be ignored for applications where the default options are fine.

#### Connecting

The ``SocketChannel`` class does not have any public constructors. Instead, you create a new ``SocketChannel`` object using one of the two static **``open()``** methods:

```Java
public static SocketChannel open(SocketAddress remote) throws IOException
public static SocketChannel open() throws IOException
```

The first variant makes the connection. This method blocks (i.e., the method will not return until the connection is made or an exception is thrown). For example:

```Java
SocketAddress address = new InetSocketAddress("www.example.com", 80);
SocketChannel channel = SocketChannel.open(address);
```
The ``noargs`` version does not immediately connect. It creates an initially unconnected socket that must be connected later using the connect() method. For example:

```Java
SocketChannel channel = SocketChannel.open();
SocketAddress address = new InetSocketAddress("www.example.com", 80);
channel.connect(address);
```
You might choose this more roundabout approach in order to configure various options on the channel and/or the socket before connecting. Specifically, use this approach if you want to open the channel without blocking:

```Java
SocketChannel channel = SocketChannel.open();
SocketAddress address = new InetSocketAddress("www.example.com", 80);
channel.configureBlocking(false);
channel.connect();
```
With a nonblocking channel, the **``connect()``** method returns immediately, even before the connection is established. The program can do other things while it waits for the operating system to finish the connection. However, before it can actually use the connection, the program must call **``finishConnect()``**:

```Java
public abstract boolean finishConnect() throws IOException
```

(This is only necessary in nonblocking mode. For a blocking channel, this method returns ``true`` immediately.) If the connection is now ready for use, **``finishConnect()``** returns ``true``. If the connection has not been established yet, **``finishConnect()``** returns ``false``. Finally, if the connection could not be established, for instance because the network is down, this method throws an exception.

If the program wants to check whether the connection is complete, it can call these two
methods:

```Java
public abstract boolean isConnected()
public abstract boolean isConnectionPending()
```

The **``isConnected()``** method returns ``true`` if the connection is open. The **``isConnectionPending()``** method returns ``true`` if the connection is still being set up but is not yet open.


#### Reading

To read from a ``SocketChannel``, first create a ByteBuffer the channel can store data in. Then pass it to the **``read()``** method:

```Java
public abstract int read(ByteBuffer dst) throws IOException
```

The channel fills the buffer with as much data as it can, then returns the number of bytes it put there. When it encounters the end of stream, the channel fills the buffer with any remaining bytes and then returns ``–1`` on the next call to **``read()``**. If the channel is blocking, this method will read at least one byte or return ``–1`` or throw an exception. If the channel is nonblocking, however, this method may return ``0``.

Because the data is stored into the buffer at the current position, which is updated automatically as more data is added, you can keep passing the same buffer to the **``read()``** method until the buffer is filled. For example, this loop will read until the buffer is filled or the end of stream is detected:

```Java
while (buffer.hasRemaining() && channel.read(buffer) != -1);
```
It is sometimes useful to be able to fill several buffers from one source. This is called a **scatter**. These two methods accept an array of ``ByteBuffer`` objects as arguments and fill each one in turn:

```Java
public final long read(ByteBuffer[] dsts) throws IOException
public final long read(ByteBuffer[] dsts, int offset, int length) throws IOException
```

The first variant fills all the buffers. The second method fills length buffers, starting with the one at offset. To fill an array of buffers, just loop while the last buffer in the list has space remaining.

For example:

```Java
ByteBuffer[] buffers = new ByteBuffer[2];
buffers[0] = ByteBuffer.allocate(1000);
buffers[1] = ByteBuffer.allocate(1000);
while (buffers[1].hasRemaining() && channel.read(buffers) != -1) ;
```

#### Writing

Socket channels have both read and write methods. In general, they are full duplex. In order to write, simply fill a ``ByteBuffer``, flip it, and pass it to one of the write methods, which drains it while copying the data onto the output—pretty much the reverse of the reading process.

The basic **``write()``** method takes a single buffer as an argument:

```Java
public abstract int write(ByteBuffer src) throws IOException
```
As with reads (and unlike ``OutputStreams``), this method is not guaranteed to write the complete contents of the buffer if the channel is nonblocking. Again, however, the cursor-based nature of buffers enables you to easily call this method again and again until the buffer is fully drained and the data has been completely written:

```Java
while (buffer.hasRemaining() && channel.write(buffer) != -1) ;
```

It is often useful to be able to write data from several buffers onto one socket. This is called a **gather**. For example, you might want to store the HTTP header in one buffer and the HTTP body in another buffer. The implementation might even fill the two buffers simultaneously using two threads or overlapped I/O. These two methods accept an array of ``ByteBuffer`` objects as arguments, and drain each one in turn:

```Java
public final long write(ByteBuffer[] dsts) throws IOException
public final long write(ByteBuffer[] dsts, int offset, int length) throws IOException
```

The first variant drains all the buffers. The second method drains length buffers, starting with the one at ``offset``.

#### Closing

Just as with regular sockets, you should close a channel when you’re done with it to free up the port and any other resources it may be using:

```Java
public void close() throws IOException
```

Closing an already closed channel has no effect. Attempting to write data to or read data from a closed channel throws an exception. If you’re uncertain whether a channel has been closed, check with **``isOpen()``**:

```Java
public boolean isOpen()
```
Naturally, this returns ``false`` if the channel is closed, ``true`` if it’s open (**``close()``** and
**``isOpen()``** are the only two methods declared in the ``Channel`` interface and shared by all channel classes).

Starting in Java 7, ``SocketChannel`` implements ``AutoCloseable``, so you can use it in ``trywith-resources``.

### **ServerSocketChannel**

The ``ServerSocketChannel`` class has one purpose: to accept incoming connections. You cannot read from, write to, or connect a ``ServerSocketChannel``. The only operation it supports is accepting a new incoming connection. The class itself only declares four methods, of which **``accept()``** is the most important. ``ServerSocketChannel`` also inherits
several methods from its superclasses, mostly related to registering with a Selector for notification of incoming connections. And finally, like all channels, it has a **``close()``** method that shuts down the server socket

#### Creating server socket channels

The static factory method **``ServerSocketChannel.open()``** creates a new ``ServerSocketChannel`` object. However, the name is a little deceptive. This method does not actually open a new server socket. Instead, it just creates the object. Before you can use it, you need to call the **``socket()``** method to get the corresponding peer ``ServerSocket``. At this point, you can configure any server options you like, such as the receive buffer size or the socket timeout, using the various setter methods in ``ServerSocket``. Then connect this ``ServerSocket`` to a ``SocketAddress`` for the port you want to bind to. For example, this code fragment opens a ``ServerSocketChannel`` on port ``80``:

```Java
try {
    ServerSocketChannel server = ServerSocketChannel.open();
    ServerSocket socket = serverChannel.socket();
    SocketAddress address = new InetSocketAddress(80);
    socket.bind(address);
} catch (IOException ex) {
    System.err.println("Could not bind to port 80 because " + ex.getMessage());
}
```

In Java 7, this gets a little simpler because ``ServerSocketChannel`` now has a **``bind()``** method of its own:

```Java
try {
    ServerSocketChannel server = ServerSocketChannel.open();
    SocketAddress address = new InetSocketAddress(80);
    server.bind(address);
} catch (IOException ex) {
    System.err.println("Could not bind to port 80 because " + ex.getMessage());
}
```

A factory method is used here rather than a constructor so that different virtual machines can provide different implementations of this class, more closely tuned to the local hardware and OS. However, this factory is not user configurable. The **``open()``** method always returns an instance of the same class when running in the same virtual machine.

#### Accepting connections

Once you’ve opened and bound a ``ServerSocketChannel`` object, the **``accept()``** method can listen for incoming connections:

```Java
public abstract SocketChannel accept() throws IOException
```

**``accept()``** can operate in either blocking or nonblocking mode. In blocking mode, the **``accept()``** method waits for an incoming connection. It then accepts that connection and returns a ``SocketChannel`` object connected to the remote client. The thread cannot do anything until a connection is made. This strategy might be appropriate for simple servers that can respond to each request immediately. Blocking mode is the default.

A ``ServerSocketChannel`` can also operate in nonblocking mode. In this case, the **``accept()``** method returns ``null`` if there are no incoming connections. Nonblocking mode is more appropriate for servers that need to do a lot of work for each connection and thus may want to process multiple requests in parallel. Nonblocking mode is normally used in conjunction with a Selector. To make a ``ServerSocketChannel`` nonblocking, pass ``false`` to its **``configureBlocking()``** method.

The **``accept()``** method is declared to throw an ``IOException`` if anything goes wrong. There are several subclasses of ``IOException`` that indicate more detailed problems, as well as a couple of runtime exceptions:

- ``ClosedChannelException`` : You cannot reopen a ``ServerSocketChannel`` after closing it.

- ``AsynchronousCloseException`` : Another thread closed this ``ServerSocketChannel`` while **``accept()``** was executing.

- ``ClosedByInterruptException`` : Another thread interrupted this thread while a blocking ``ServerSocketChannel`` was waiting.

- ``NotYetBoundException`` : You called ``open()`` but did not bind the ``ServerSocketChannel``’s peer ``ServerSocket`` to an address before calling **``accept()``**. This is a runtime exception, not an ``IOException``.

- ``SecurityException`` : The security manager refused to allow this application to bind to the requested port.

### **The Channels Class**

``Channels`` is a simple utility class for wrapping channels around traditional ``I/O-based`` streams, readers, and writers, and vice versa. It’s useful when you want to use the new ``I/O`` model in one part of a program for performance, but still interoperate with legacy ``API``s that expect streams. It has methods that convert from streams to channels and methods that convert from channels to streams, readers, and writers:

```Java
public static InputStream newInputStream(ReadableByteChannel ch)
public static OutputStream newOutputStream(WritableByteChannel ch)
public static ReadableByteChannel newChannel(InputStream in)
public static WritableByteChannel newChannel(OutputStream out)
public static Reader newReader (ReadableByteChannel channel, CharsetDecoder decoder, int minimumBufferCapacity)
public static Reader newReader (ReadableByteChannel ch, String encoding)
public static Writer newWriter (WritableByteChannel ch, String encoding)
```
The ``SocketChannel`` class implements both the ``ReadableByteChannel`` and ``WritableByteChannel`` interfaces seen in these signatures. ``ServerSocketChannel`` implements neither of these because you can’t read from or write to it.

For example, all current XML APIs use streams, files, readers, and other traditional ``I/O APIs`` to read the ``XML`` document. If you’re writing an ``HTTP`` server designed to process ``SOAP`` requests, you may want to read the ``HTTP`` request bodies using channels and parse the ``XML`` using ``SAX`` for performance. In this case, you’d need to convert these channels into streams before passing them to ``XMLReader``’s **``parse()``** method:

```Java
SocketChannel channel = server.accept();
processHTTPHeader(channel);
XMLReader parser = XMLReaderFactory.createXMLReader();
parser.setContentHandler(someContentHandlerObject);
InputStream in = Channels.newInputStream(channel);
parser.parse(in);
```

### **Asynchronous Channels (Java 7)**

Java 7 introduces the ``AsynchronousSocketChannel`` and ``AsynchronousServerSocketChannel`` classes. These behave like and have almost the same interface as ``SocketChannel`` and ``ServerSocketChannel`` (though they are not subclasses of those classes). However, unlike ``SocketChannel`` and ``ServerSocketChannel``, reads from and writes to asynchronous channels return immediately, even before the ``I/O`` is complete. The data read or written is further processed by a Future or a ``CompletionHandler``. The **``connect()``** and **``accept()``** methods also execute asynchronously and return Futures. Selectors are not used.

For example, suppose a program needs to perform a lot of initialization at startup. Some of that involves network connections that are going to take several seconds each. You can start several asynchronous operations in parallel, then perform your local initializations, and then request the results of the network operations:

```Java
SocketAddress address = new InetSocketAddress(args[0], port);
AsynchronousSocketChannel client = AsynchronousSocketChannel.open();
Future<Void> connected = client.connect(address);
ByteBuffer buffer = ByteBuffer.allocate(74);
// wait for the connection to finish
connected.get();
// read from the connection
Future<Integer> future = client.read(buffer);
// do other things...
// wait for the read to finish...
future.get();
// flip and drain the buffer
buffer.flip();
WritableByteChannel out = Channels.newChannel(System.out);
out.write(buffer);
```
The advantage of this approach is that the network connections run in parallel while the program does other things. When you’re ready to process the data from the network, but not before, you stop and wait for it by calling **``Future.get()``**. You could achieve the same effect with thread pools and callables, but this is perhaps a little simpler, especially if buffers are a natural fit for your application.

This approach fits the situation where you want to get results back in a very particular order. However, if you don’t care about order, if you can process each network read independently of the others, then you may be better off using a ``CompletionHandler`` instead. For example, imagine you’re writing a search engine web spider that feeds pages into some backend. Because you don’t care about the order of the responses returned,you can spawn a large number of ``AsynchronousSocketChannel`` requests and give each
one a ``CompletionHandler`` that stores the results in the backend.

The generic ``CompletionHandler`` interface declares two methods: **``completed()``**, which is invoked if the read finishes successfully; and **``failed()``**, which is invoked on an I/O error. For example, here’s a simple ``CompletionHandler`` that prints whatever it received on ``System.out``:

```Java
class LineHandler implements CompletionHandler<Integer, ByteBuffer> {
    @Override
    public void completed(Integer result, ByteBuffer buffer) {
        buffer.flip();
        WritableByteChannel out = Channels.newChannel(System.out);
        try {
            out.write(buffer);
        } catch (IOException ex) {
            System.err.println(ex);
        }
    }
    @Override
    public void failed(Throwable ex, ByteBuffer attachment) {
        System.err.println(ex.getMessage());
    }
}
```

When you read from the channel you pass a buffer, an attachment, and a ``CompletionHandler`` to the **``read()``** method:

```Java
ByteBuffer buffer = ByteBuffer.allocate(74);
CompletionHandler<Integer, ByteBuffer> handler = new LineHandler();
channel.read(buffer, buffer, handler);
```

Here I’ve made the attachment the buffer itself. This is one way to push the data read from the network into the ``CompletionHandler`` where it can handle it. Another common pattern is to make the ``CompletionHandler`` an anonymous inner class and the buffer a final local variable so it’s in scope inside the completion handler.

Although you can safely share an ``AsynchronousSocketChannel`` or ``AsynchronousServerSocketChannel`` between multiple threads, no more than one thread can read from this channel at a time and no more than one thread can write to the channel at a time. (One thread can read and another thread can write simultaneously, though.) If a thread attempts to read while another thread has a pending read, the **``read()``** method throws a ``ReadPendingException``. Similarly, if a thread attempts to write while another thread has a pending write, the **``write()``** method throws a ``WritePendingException``.

### **Socket Options (Java 7)**

Beginning in Java 7, ``SocketChannel``, ``ServerSocketChannel``, ``AsynchronousServer``, ``SocketChannel``, ``AsynchronousSocketChannel``, and ``DatagramChannel`` all implement the new ``NetworkChannel`` interface. The primary purpose of this interface is to support the various TCP options such as ``TCP_NODELAY``, ``SO_TIMEOUT``, ``SO_LINGER``, ``SO_SNDBUF``, ``SO_RCVBUF``, and ``SO_KEEPALIVE``. The options have the same meaning in the underlying TCP stack whether set on a socket or a channel. However, the interface to these options is a little different. Rather than individual methods for each supported option, the channel classes each have just three methods to get, set, and list the supported options:

```Java
<T> T getOption(SocketOption<T> name) throws IOException
<T> NetworkChannel setOption(SocketOption<T> name, T value) throws IOException
Set<SocketOption<?>> supportedOptions()
```

The ``SocketOption`` class is a generic class specifying the name and type of each option. The type ``parameter <T>`` determines whether the option is a ``boolean``, ``Integer``, or ``NetworkInterface``. The ``StandardSocketOptions`` class provides constants for each of the 11 options Java recognizes:

```Java
SocketOption<NetworkInterface> StandardSocketOptions.IP_MULTICAST_IF
SocketOption<Boolean> StandardSocketOptions.IP_MULTICAST_LOOP
SocketOption<Integer> StandardSocketOptions.IP_MULTICAST_TTL
SocketOption<Integer> StandardSocketOptions.IP_TOS
SocketOption<Boolean> StandardSocketOptions.SO_BROADCAST
SocketOption<Boolean> StandardSocketOptions.SO_KEEPALIVE
SocketOption<Integer> StandardSocketOptions.SO_LINGER
SocketOption<Integer> StandardSocketOptions.SO_RCVBUF
SocketOption<Boolean> StandardSocketOptions.SO_REUSEADDR
SocketOption<Integer> StandardSocketOptions.SO_SNDBUF
SocketOption<Boolean> StandardSocketOptions.TCP_NODELAY
```

For example, this code fragment opens a client network channel and sets ``SO_LINGER`` to 240 seconds:

```Java
NetworkChannel channel = SocketChannel.open();
channel.setOption(StandardSocketOptions.SO_LINGER, 240);
```

Different channels and sockets support different options. For instance, ``ServerSocketChannel`` supports ``SO_REUSEADDR`` and ``SO_RCVBUF`` but not ``SO_SNDBUF`` Trying to set an option the channel doesn’t support throws an ``UnsupportedOperationException``.

**Example:** This is a simple program to list all supported socket options for the different types of network channels.

```Java
import java.io.*;
import java.net.*;
import java.nio.channels.*;
public class OptionSupport {
    public static void main(String[] args) throws IOException {
        printOptions(SocketChannel.open());
        printOptions(ServerSocketChannel.open());
        printOptions(AsynchronousSocketChannel.open());
        printOptions(AsynchronousServerSocketChannel.open());
        printOptions(DatagramChannel.open());
    }
    private static void printOptions(NetworkChannel channel) throws IOException {
        System.out.println(channel.getClass().getSimpleName() + " supports:");
        for (SocketOption<?> option : channel.supportedOptions()) {
            System.out.println(option.name() + ": " + channel.getOption(option));
        }
        System.out.println();
        channel.close();
    }
}
```
**Output:**

```
SocketChannelImpl supports:
TCP_KEEPINTERVAL: 75
SO_OOBINLINE: false
TCP_QUICKACK: false
IP_TOS: 0
TCP_KEEPIDLE: 7200
TCP_KEEPCOUNT: 9
SO_SNDBUF: 8192
TCP_NODELAY: false
SO_LINGER: -1
SO_KEEPALIVE: false
SO_RCVBUF: 65536
SO_INCOMING_NAPI_ID: 0
SO_REUSEADDR: false
SO_REUSEPORT: false

ServerSocketChannelImpl supports:
TCP_KEEPINTERVAL: 75
TCP_QUICKACK: false
SO_RCVBUF: 65536
TCP_KEEPIDLE: 7200
TCP_KEEPCOUNT: 9
SO_INCOMING_NAPI_ID: 0
SO_REUSEADDR: true
SO_REUSEPORT: false

UnixAsynchronousSocketChannelImpl supports:
SO_KEEPALIVE: false
TCP_KEEPINTERVAL: 75
TCP_QUICKACK: false
SO_RCVBUF: 65536
TCP_KEEPIDLE: 7200
TCP_KEEPCOUNT: 9
SO_INCOMING_NAPI_ID: 0
SO_SNDBUF: 8192
TCP_NODELAY: false
SO_REUSEADDR: false
SO_REUSEPORT: false

UnixAsynchronousServerSocketChannelImpl supports:
TCP_KEEPINTERVAL: 75
TCP_QUICKACK: false
SO_RCVBUF: 65536
TCP_KEEPIDLE: 7200
TCP_KEEPCOUNT: 9
SO_INCOMING_NAPI_ID: 0
SO_REUSEADDR: true
SO_REUSEPORT: false

DatagramChannelImpl supports:
IP_DONTFRAGMENT: false
SO_RCVBUF: 106496
IP_TOS: 0
IP_MULTICAST_TTL: 1
SO_INCOMING_NAPI_ID: 0
SO_SNDBUF: 106496
IP_MULTICAST_LOOP: true
SO_BROADCAST: false
SO_REUSEADDR: false
SO_REUSEPORT: false
IP_MULTICAST_IF: null

```
## Readiness Selection
-------------------------
For ``network programming``, the second part of the new ``I/O`` APIs is readiness selection, the ability to choose a socket that will not block when read or written. This is primarily of interest to servers, although clients running multiple simultaneous connections with several windows open such as a web spider or a browser can take advantage of it as well.

In order to perform readiness selection, different channels are registered with a ``Selector`` object. Each channel is assigned a ``SelectionKey``. The program can then ask the ``Selector`` object for the set of keys to the channels that are ready to perform the operation you want to perform without blocking.

### **The Selector Class**

The only constructor in Selector is protected. Normally, a new selector is created by invoking the static factory method **``Selector.open()``**:

```Java
public static Selector open() throws IOException
```
The next step is to add channels to the selector. There are no methods in the ``Selector`` class to add a channel. The  **``register()``** method is declared in the ``SelectableChannel`` class. Not all channels are selectable in particular, FileChannels aren’t selectable but all network channels are. Thus, the channel is registered with a selector by passing the selector to one of the channel’s register methods:

```Java
public final SelectionKey register(Selector sel, int ops) throws ClosedChannelException
```
```Java
public final SelectionKey register(Selector sel, int ops, Object att) throws ClosedChannelException
```
The first argument is the selector the channel is registering with. The second argument is a named constant from the ``SelectionKey`` class identifying the operation the channel is registering for. The ``SelectionKey`` class defines four named bit constants used to select the type of the operation:

- ``SelectionKey.OP_ACCEPT``
- ``SelectionKey.OP_CONNECT``
- ``SelectionKey.OP_READ``
- ``SelectionKey.OP_WRITE``

These are bit-flag int constants (1, 2, 4, etc.). Therefore, if a channel needs to register for multiple operations in the same selector (e.g., for both reading and writing on a socket), combine the constants with the bitwise or operator (``|``) when registering:

```Java
channel.register(selector, SelectionKey.OP_READ | SelectionKey.OP_WRITE);
```

The optional third argument is an attachment for the key. This object is often used to store state for the connection. For example, if you were implementing a web server, you might attach a ``FileInputStream`` or ``FileChannel`` connected to the local file the server streams to the client.

After the different channels have been registered with the selector, you can query the selector at any time to find out which channels are ready to be processed. Channels may be ready for some operations and not others. For instance, a channel could be ready for reading but not writing.

There are three methods that select the ready channels. They differ in how long they wait to find a ready channel. The first, **``selectNow()``**, performs a nonblocking select. It returns immediately if no connections are ready to be processed now:

```Java
public abstract int selectNow() throws IOException
```

The other two select methods are blocking:

```Java
public abstract int select() throws IOException
```
```Java
public abstract int select(long timeout) throws IOException
```

The first method waits until at least one registered channel is ready to be processed before returning. The second waits no longer than ``timeout`` milliseconds for a channel to be ready before returning ``0``. These methods are useful if your program doesn’t have anything to do when no channels are ready to be processed.

When you know the channels are ready to be processed, retrieve the ready channels using **``selectedKeys()``**:

```Java
public abstract Set<SelectionKey> selectedKeys()
```
You iterate through the returned set, processing each ``SelectionKey`` in turn. You’ll also want to remove the key from the iterator to tell the selector that you’ve handled it.Otherwise, the selector will keep telling you about it on future passes through the loop.

Finally, when you’re ready to shut down the server or when you no longer need the selector, you should close it:

```Java
public abstract void close() throws IOException
```

This step releases any resources associated with the selector. More importantly, it cancels all keys registered with the selector and interrupts up any threads blocked by one of this selector’s select methods.

### **The SelectionKey Class**

``SelectionKey`` objects serve as pointers to channels. They can also hold an object attachment, which is how you normally store the state for the connection on that channel.

``SelectionKey`` objects are returned by the **``register()``** method when registering a channel with a selector. However, you don’t usually need to retain this reference. The **``selectedKeys()``** method returns the same objects again inside a Set. A single channel can be registered with multiple selectors.

When retrieving a ``SelectionKey`` from the set of selected keys, you often first test what that key is ready to do. There are four possibilities:

```Java
public final boolean isAcceptable()
```
```Java
public final boolean isConnectable()
```
```Java
public final boolean isReadable()
```
```Java
public final boolean isWritable()
```
This test isn’t always necessary. In some cases, the selector is only testing for one possibility and will only return keys to do that one thing. But if the selector does test for multiple readiness states, you’ll want to test which one kicked the channel into the ready state before operating on it. It’s also possible that a channel is ready to do more than one thing.

Once you know what the channel associated with the key is ready to do, retrieve the channel with the **``channel()``** method:

```Java
public abstract SelectableChannel channel()
```

If you’ve stored an object in the ``SelectionKey`` to hold state information, you can retrieve it with the **``attachment()``** method:

```Java
public final Object attachment()
```

Finally, when you’re finished with a connection, deregister its ``SelectionKey`` object so the selector doesn’t waste any resources querying it for readiness. I don’t know that this is absolutely essential in all cases, but it doesn’t hurt. You do this by invoking the key’s **``cancel()``** method:

```Java
public abstract void cancel()
```
However, this step is only necessary if you haven’t closed the channel. Closing a channel automatically deregisters all keys for that channel in all selectors. Similarly, closing a selector invalidates all keys in that selector.

**Example:**

```Java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.util.Iterator;
import java.util.Set;

public class ReadlinessSelectionExample {
    public final static String HOST = "localhost";
    public final static int PORT = 12345;

    public static void main(String[] args) {
        try {
            ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
            serverSocketChannel.bind(new InetSocketAddress(HOST, PORT));
            serverSocketChannel.configureBlocking(false);

            Selector selector = Selector.open();
            serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

            System.out.println("Server started...");

            while (true) {
                int readyChannels = selector.select();

                if (readyChannels == 0) {
                    continue;
                }

                Set<SelectionKey> selectedKeys = selector.selectedKeys();
                Iterator<SelectionKey> keyIterator = selectedKeys.iterator();

                while (keyIterator.hasNext()) {
                    SelectionKey key = keyIterator.next();

                    if (key.isAcceptable()) {
                        ServerSocketChannel serverChannel = (ServerSocketChannel) key.channel();
                        SocketChannel clientChannel = serverChannel.accept();
                        clientChannel.configureBlocking(false);
                        clientChannel.register(selector, SelectionKey.OP_READ);
                        System.out.println("Client connected: " + clientChannel.getRemoteAddress());
                    }

                    if (key.isReadable()) {
                        SocketChannel clientChannel = (SocketChannel) key.channel();
                        ByteBuffer buffer = ByteBuffer.allocate(1024);
                        int bytesRead = clientChannel.read(buffer);

                        if (bytesRead == -1) {
                            key.cancel();
                            clientChannel.close();
                            System.out.println("Client disconnected: " + clientChannel.getRemoteAddress());
                        } else if (bytesRead > 0) {
                            buffer.flip();
                            byte[] data = new byte[buffer.remaining()];
                            buffer.get(data);
                            String message = new String(data);
                            System.out.println("Received from client: " + message);

                            // Echo the message back to the client
                            ByteBuffer responseBuffer = ByteBuffer.wrap(data);
                            clientChannel.write(responseBuffer);
                        }
                    }

                    keyIterator.remove();
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```