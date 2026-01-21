## **What Is RPC (Remote Procedure Call)?**

A **Remote Procedure Call (RPC)** is a communication protocol that allows a program (the *client*) to execute a function or procedure on a different machine or process (*server*) as if it were a **local function call** in the code. It hides the details of network communication and enables developers to work with distributed systems without manually handling network logic. ([Wikipedia][1])

In essence, RPC lets you write:

```js
result = remoteFunction(arg1, arg2);
```

and have that function run on a server somewhere, not locally.

---

## **Core Idea**

In a local function call, the function runs in the same program:

```
call function X locally → return result
```

In an RPC:

```
call function X remotely → send request over network → server executes → return result
```

The key is that **it feels like a normal function call** to the developer, but under the hood it becomes a **networked request/response**. ([Wikipedia][1])

---

## **How RPC Works Under the Hood (Step-by-Step)**

Even though it *looks* like a normal call, RPC involves several important mechanisms:

### **1. Client Calls a Stub (Proxy)**

* The client calls what appears to be a local function.
* In reality, this is a *client stub*, a small piece of code that mimics the real function.
* The client stub is responsible for preparing the RPC. ([TechTarget][2])

---

### **2. Marshalling (Serializing Parameters)**

* The stub collects the function parameters and converts them into a format that can be sent over the network.
* This process is called *marshalling* (or serialization).
* Common formats include JSON, Protocol Buffers, or other binary/text protocols. ([Wikipedia][1])

---

### **3. Network Transmission**

* The serialized request packet is sent through the network using a transport protocol (like HTTP, TCP/IP, or UDP).
* The system sends the packet to the remote server where the target function lives. ([TechTarget][2])

---

### **4. Server Receives the Request and Unmarshals**

* On the server side, the *server stub* receives the incoming message.
* The server stub deserializes (unmarshals) the parameters back into usable data. ([Wikipedia][1])

---

### **5. Execution of the Procedure**

* The server then invokes the actual function using the unpacked parameters.
* The function runs on the remote machine. ([TechTarget][2])

---

### **6. Marshalling the Result**

* After execution, the server stub serializes the result value(s). ([Wikipedia][1])

---

### **7. Response Sent Back**

* The result packet is sent back over the network to the client machine.
* This completes the round trip. ([TechTarget][2])

---

### **8. Client Unmarshals and Continues**

* The client stub receives the response and deserializes it.
* The program resumes as if the function had been called locally, with the returned result. ([Wikipedia][1])

---

## **Important Concepts in RPC**

### **Client Stub**

A proxy that represents the remote function in client code. It handles marshalling and sending the request.

### **Server Stub**

The component on the server that receives requests, unmarshals parameters, runs the actual function, and sends back results.

### **Marshalling / Unmarshalling**

Processes of converting function arguments/results into a transferable format and back.

### **Transport Layer**

RPC uses a lower-level protocol like TCP/IP or HTTP to actually send and receive messages. ([IBM][3])

---

## **Why RPC Is Useful**

* **Abstraction:** Developers call remote services like local functions.
* **Simplicity:** Network details are hidden.
* **Modularity:** Services and clients can evolve independently.
* **Reusability:** Remotes services can be used across multiple clients. ([Lenovo][4])

---

## **Things to Be Aware Of**

Even though RPC looks like a local call, there are essential differences:

### **Network Failures**

* RPC can fail due to network issues.
* Unlike local calls, remote calls can timeout, disconnect, or partially execute. ([Wikipedia][1])

### **Latency**

* Remote calls involve network delays, so they are usually slower than local calls.

### **Error Handling**

* Client must handle network errors and response issues explicitly.

---

## **Types of RPC**

There are various RPC implementations and protocols:

* **JSON-RPC:** Uses JSON as the message format. ([Wikipedia][5])
* **gRPC:** Modern RPC framework using HTTP/2 and Protocol Buffers (binary encoding). ([Wikipedia][6])
* **XML-RPC:** Uses XML for message format. ([Wikipedia][7])

Each has different performance and tooling characteristics.

---

## **Summary**

* RPC is a protocol for executing functions on a remote server as if they were local calls. ([Wikipedia][1])
* It abstracts networking so developers don’t write raw socket code. ([Lenovo][4])
* Under the hood, RPC involves:

  1. client stub
  2. marshalling
  3. network transmission
  4. server execution
  5. response back to client
  6. client continues as if local function was invoked. ([TechTarget][2])
[6]: https://ru.wikipedia.org/wiki/GRPC?utm_source=chatgpt.com "GRPC"
[7]: https://de.wikipedia.org/wiki/Remote_Procedure_Call?utm_source=chatgpt.com "Remote Procedure Call"
