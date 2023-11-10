<div align="center">

[**_``Go Back``_**](../README.md)

# RMI

</div>

``Java RMI (Remote Method Invocation)`` is a powerful technology in Java that allows you to create distributed applications by enabling objects to invoke methods on remote objects, as if they were local objects. This technology facilitates communication between different Java Virtual Machines (JVMs) over a network. RMI is based on object-oriented principles and is a key part of Java's network programming capabilities.It is provided in the package ``java.rmi``.

> ``RMI`` is a mechanism that allows an object residing in one system (JVM) to **access/invoke** an object running on another JVM.

## RMI Architecture
------------------------
In an RMI application, we write two programs, a **server program** (***resides on the server***) and a **client program** (***resides on the client***).

- Inside the server program, a remote object is created and reference of that object is made available for the client (using the registry).

- The client program requests the remote objects on the server and tries to invoke its methods.

The following diagram shows the architecture of an RMI application:

<div align="center">

![RMI Architecture](img/RMI-Architecture.jpg)

</div>

Let us now discuss the components of this architecture.

- **``Transport Layer``** : This layer connects the **client** and the **server**. It manages the existing connection and also sets up new connections.

- **``Stub``** : A stub is a representation (proxy) of the remote object at client. It resides in the client system; it acts as a gateway for the client program.

- **``Skeleton``** : This is the object which resides on the server side. **stub** communicates with this **skeleton** to pass request to the remote object.

- **``RRL(Remote Reference Layer)``** : It is the layer which manages the references made by the client to the remote object.

## **Role of Client and Server in RMI**

In ``RMI (Remote Method Invocation)``, a distributed computing technology in Java, there are typically two key components: the client and the server. Each plays a distinct role in facilitating communication between distributed objects or services. Here's an overview of the roles of the client and server in RMI:

### Server:

- ``Service Provider``: The server is responsible for providing one or more services or objects that the client can invoke remotely. These services or objects are typically implemented as Java classes with methods that can be called remotely.

- ``Object Exporter``: The server exports its remote objects, making them available to remote clients. This involves binding the objects to a registry (such as RMI Registry) or using a more advanced method like RMI Activation.

- ``Listening for Client Requests``: The server listens for incoming requests from remote clients. It waits for client requests to invoke methods on its exported objects.

- ``Processing Client Requests``: When a client request arrives, the server processes the request by invoking the appropriate method on the remote object. The server performs the requested computation and sends the result back to the client.

- ``Security and Access Control``: The server may implement security measures to control access to its services. It can enforce authentication, authorization, and other security mechanisms to protect sensitive operations.

### Client:

- ``Service Consumer``: The client is responsible for consuming the services provided by the server. It typically initiates the communication by looking up remote objects from a registry or other discovery mechanisms.

- ``Remote Method Invocation``: The client invokes methods on remote objects provided by the server as if they were local objects. It uses the same method calls and parameter passing.

- ``Serialization`` and ``Deserialization``: The client serializes method calls and their parameters into a format suitable for transmission over the network and deserializes the results received from the server.

- ``Handling Remote Exceptions``: The client handles exceptions that may occur during remote method invocation. These exceptions can include network-related issues, remote server failures, and application-specific exceptions thrown by the server.

- ``Callback Mechanism (Optional)``: In some cases, the client may also act as a server, allowing the server to make callback invocations to it. This is known as a callback or two-way RMI, where the client and server can exchange messages bidirectionally.

- ``Resource Management``: The client is responsible for managing the resources associated with remote objects, such as closing remote connections and releasing resources when they are no longer needed.

## Working of RMI
------------------
The communication between **client** and **server** is handled by using two intermediate objects: ``Stub object (on client side)`` and ``Skeleton object (on server-side)`` as also can be depicted from below media as follows:

<div align="center">

![RMI Working](img/Working-Of-RMI.jpg)

</div>

## Remote Method Calls
-----------------------
In Java's ``Remote Method Invocation (RMI)`` technology, ``Remote Method Call`` allow you to invoke methods on objects that reside on remote servers as if they were local objects. RMI simplifies the process of building distributed applications by abstracting away many of the complexities of network communication. 

## Stub
---------
In Java's ``Remote Method Invocation (RMI)`` framework, a ``stub`` is a critical component that facilitates communication between a client and a remote object located on a server. Stubs act as proxies for remote objects, allowing clients to invoke methods on remote objects as if they were local objects, even though the actual object resides on a different JVM (Java Virtual Machine).

## Parameter Marshalling and Unmarshalling
---------------------------------
Whenever a client invokes a method that accepts parameters on a remote object, the parameters are bundled into a message before being sent over the network. These parameters may be of primitive type or objects. In case of primitive type, the parameters are put together and a header is attached to it. In case the parameters are objects, then they are serialized. This process is known as **marshalling**.

At the server side, the packed parameters are unbundled and then the required method is invoked. This process is known as **unmarshalling**.

## RMI Registry
----------------
``RMI`` registry is a namespace on which all server objects are placed. Each time the server creates an object, it registers this object with the ``RMIregistry`` (using ``bind()`` or ``reBind()`` methods). These are registered using a unique name known as bind name.

To invoke a remote object, the client needs a reference of that object. At that time, the client fetches the object from the registry using its bind name (using ``lookup()`` method).

The following illustration explains the entire process:

<div align="center">

![RMI Registry](img/RMI-Registry.jpg)

</div>

## Defining & Implementating RMI Service Interface
---------------------------------------------------
An ``RMI`` application begins with defining a remote interface that declares the methods that can be invoked remotely by clients. This interface should extend the ``java.rmi.Remote`` interface and declare the methods that the server will provide.

```Java
import java.rmi.Remote;
import java.rmi.RemoteException;

public interface MessageInterface extends Remote {
    String HelloMessage() throws RemoteException;
    // Add more remote methods here
}
```

The next step is to implement the remote interface ( ``MessageInterface`` ). To implement the remote interface, the class should extend to ``UnicastRemoteObject`` class of ``java.rmi`` package. Also, a default constructor needs to be created to throw the ``java.rmi.RemoteException`` from its parent constructor in class.

```Java
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;

public class MessageService extends UnicastRemoteObject implements MessageInterface {
    public MessageService() throws RemoteException {
        // Constructor must declare RemoteException
    }

    @Override
    public String HelloMessage() throws RemoteException {
        return "Hello from the RMI Message server!";
    }
}

```

## Creating an RMI Server & Client
-----------------------------------

### **Creating an RMI Server**

The next step is to create the server application program. The RMI server is responsible for hosting the remote objects and providing their implementations. 

```Java
import java.rmi.Naming;

public class RMIMessageServer{
    public static void main(String[] args) {
        try {
            MessageInterface server = new MessageService();
            Naming.rebind("RMIMessageService", server);
            System.out.println("RMI message server is running...");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### **Creating an RMI Client**

The RMI Client is responsible for locating the remote object on the server and invoking its methods. It does so using the ``java.rmi.Naming`` class to look up the remote object by its registered name.

```Java
import java.rmi.Naming;

public class RMIMessageClient {
    public static void main(String[] args) {
        try {
            MessageInterface remoteService = (MessageInterface) Naming.lookup("rmi://localhost/RMIMessageService");
            String response = remoteService.HelloMessage();
            System.out.println("Response from server: " + response);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## Running the RMI System
--------------------------
To run an ``RMI`` application, follow these steps:

- **Compile all Java files: ``javac`` :**

    **Command:**

    ```bash
    javac MessageInterface.java
    javac MessageService.java
    javac RMIMessageServer.java
    javac RMIMessageClient.java
    ```

- **Start the RMI registry: ``rmiregistry`` in the command line :**
    
    **Command:**
    
    ```
    rmiregistry
    ```

    **Output:**
    
    ```
    WARNING: A terminally deprecated method in java.lang.System has been called
    WARNING: System::setSecurityManager has been called by sun.rmi.registry.RegistryImpl
    WARNING: Please consider reporting this to the maintainers of sun.rmi.registry.RegistryImpl
    WARNING: System::setSecurityManager will be removed in a future release
    ```

- **Run the RMI Server :** 

    **Command:**
    
    ```
    java RMIMessageServer
    ```

    **Output:**
    
    ```
    RMI message server is running...
    ```

- **Run the RMI Client :**

    **Command:**
    
    ```
    java RMIMessageClient
    ```

    **Output:**
    
    ```
    Response from server: Hello from the RMI Message server!
    ```