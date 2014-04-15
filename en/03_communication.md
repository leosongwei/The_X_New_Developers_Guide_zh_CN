Communication Between Client and Server
=======================================

*Alan Coopersmith*

The X client libraries and server infrastructure handle most of the details of encoding, decoding and transmitting the X protocol. However, a basic understanding of the communication model is often needed to be able to develop and debug X clients, server code and driver code.

The communications channel between an X client and server is full-duplex: either side can send a message to the other at any time. This is canonically implemented over a TCP/IP socket interface, though other communications channels are often used, including Unix domain sockets, named pipes and shared memory. The channel must provide a reliable, ordered byte stream---the X protocol provides no mechanism for reordering or resending packets. For TCP/IP connections, for example, TCP is used rather than UDP.

The messages sent over the X communications channel are defined by the X Protocol, Version 11: X11 for short. This protocol was originally published in 1987. While it has been extended greatly over the past 25 years, the core protocol is still compatible with the original definition. The protocol specification documents can be read at http://www.x.org/releases/current/doc/ .

One of the key features of the X11 protocol is its extension mechanism. While the core protocol has seen some minor backward-compatible changes over the years, most evolution happens via optional extensions to the protocol. The core protocol provides a mechanism to list and query the extensions supported by the X server, and for those extensions to add their messages to the set supported in communications between client and server. Server builders can choose which extensions to support, and developers can add or remove extensions over time as use cases evolve, without having to worry about breaking compatibility with the core protocol. Clients need to check for extensions they require. If an extension is not present (at an appropriate version level), the client may either fall back to providing a less functional interface, or inform the user that the extension is missing and fail.

When an X client first connects to the X server, a handshaking procedure occurs to establish the channel, to verify that the client is authenticated to connect, and to set some communication parameters, such as the byte-endianness to be used. In the early days of X, it was assumed that the server was a highly capable machine and the client was not. Thus, interestingly, the client may ask the server to provide whichever endinanness the client prefers: the server will byte-swap requests and responses as needed.

After that, the client sends request messages to the X server, asking the server to perform an operation or to provide some information to the client. Clients typically accumulate requests in a buffer, sending them in batches for more efficient communication handling and context switching. This buffer is normally provided and managed by Xlib or XCB for the client; the client application often needs to flush buffered requests to the X server to ensure timely processing.

The X server processes the requests from each client in the order received from that client. However, the server is typically multiplexing requests from multiple clients at any given time, and does not guarantee any ordering between clients unless special requests are made to ensure that. Both server and client keep track of the number of requests sent on their connection so far, and use that number as a sequence number for each request to identify it later.

The server sends responses to the client. Many requests from clients are simply handled by the X server, and result in nothing being returned to the client if all goes well. If a request is defined to return information, the server sends a response called a reply to the client. The sequence number of the reply identifies the request to which the reply data is a response. Clients may have sent many requests at once to the X server, and be waiting for the responses asynchronously. The replies will come back in order, but the client needs to remember the request sequence number to map the reply back to the request that was made.

If there is a problem handling a request, then the X server will send an error response to the client. The error response includes the sequence number of the problematic packet, an error code and some additional details about what went wrong. Debugging a failure often involves determining the origin of the errant request. Error responses are usually received well after the client has moved on to later calls, due to the asynchronicity of the protocol; thus the sequence number is invaluable information here.

Clients may also register to be notified of a variety of events from the X server. Events are sent as a specific response type from the server to interested clients as they occur. Events may be caused by:

* User input, such as moving a mouse or typing on the keyboard
* Another client, such as a window manager moving a window
* External events, such as devices being connected or disconnected from the system.

Toolkits and X protocol libraries have functions for checking if any events have been received from the X server. Most X applications are driven from a main "event loop" which captures event responses and handles them appropriately.

In the X11 protocol, each request and response type has an 8-bit identifier code to determine which operation, event, or error is being signaled. These codes are unique in each communication direction. For instance, any message from a client to the server starting with byte value 3 is a GetWindowAttributes Request. Errors and events have separate numbering spaces: an error code of 3 indicates a BadWindow error, while an event code of 3 indicates a KeyRelease event.

Many of the features of the communication protocol are tuned to support the computing environment of many years ago. Request sequence numbers, for example, are not transmitted at all by the client: the client and server can presumably keep count. Response sequence numbers are only partially transmitted, using a clever protocol plus counting to keep track of the rest. Similarly, resources on the server are identified in protocol requests using small integers known as XID's. The XIDs are actually allocated by the client, from an XID space given to them by the server. Both of these optimizations, as well as the 8-bit request / response ID space, are inspired by the desire to have very small packets on the wire. Thus, a line can be drawn on the screen using very few bytes moving in only one direction.

Extensions can add requests, events and errors to the core protocol as needed. The X server dynamically assigns codes to extension messages at server startup. Thus, the same extension may have different values on differently configured servers. The values in use on a currently running X server can be displayed via the command "xdpyinfo -queryExtensions". There are only 256 8-bit numbers, so extensions normally try to conserve this space: the extension will add a single request or event code and then use a second byte as a "minor" code to identify sub-messages specific to that extension.

X Protocol Security model
-------------------------

The core X protocol has a fairly simple security model. The address of a client is checked at connection time to see if it is allowed to connect to the server: if not, the connection is refused. If the client is allowed to connect, then it is granted full access to the server. This includes being able to send messages to all clients, to monitor the state of all devices (including all keystrokes, mouse movements, etc.). The client may even request that the X server kill another client's connection.

Various extensions have added finer-grained optional security models. The "SECURITY" extension offers a simple "Trusted" vs. "Untrusted" client distinction. More complex multi-level security models are provided by the SELinux and Solaris Xtsol extensions.

Authentication methods
----------------------

The mechanisms by which X clients can be authenticated as being allowed to connect to the X server have evolved over time. Most servers support a variety of choices.

### xhost

The initial connection security method was host-based authentication, in which any user or program on a given host is given permission to connect. By default only the local host can connect, but "xhost +hostname" can be used to add additional hosts. This mechanism is somewhat naive: it assumes that each machine has a single user, or that all users are equally trustworthy.

The xhost mechanism has been extended in later versions to add additional access control methods controlled via the same mechanisms. These methods include an extensible "server interpreted" scheme which just passes the string to the X server and allows the X server to define new access control types as needed. The most common server interpreted scheme is "localuser", which uses interfaces provided in modern operating systems to get the user id of the connecting program for authentication. The xhost and Xsecurity man pages provide more details on this topic.

### xauth

The authentication method most commonly used today is based on a shared secret between the client and server. This is a later addition to the X11 protocol, and addresses many of the problems of xhost authentication. Several forms of secret sharing are supported. The most basic is MIT-MAGIC-COOKIE-1. The program that starts the X server and session (xinit, gdm, xdm, etc.) simply chooses a random 128 bit number and records it in a file passed to the X server via the "-auth" command line option and to the clients in an Xauthority file (found in either "$HOME/.Xauthority" or "$XAUTHORITY"). The client is allowed to connect if it can read the file and present the correct value to the server. When using the SECURITY extension, the cookie may also identify a client as Trusted or Untrusted, and grant privileges appropriately. The xauth and Xsecurity man pages provide more details on these.

Access control within the X server
----------------------------------

*Martin Peres (mupuf)*

Applications used to be limited in size and complexity and fairly trusted. This doesn't hold true anymore since users have to deal with proprietary software or large pieces of software such as web browser that may be malicious or may be turned to into one (eg. buffer overflow exploit). Unfortunately, once a X client is authenticated on the X server, it is mostly free to interact with other X clients in several ways:

* Redirect input
* Screen capture/transparent rendering
* Read/write from/to the clipboard
* Alter other X clients' properties

Thus, many X applications cannot be trusted and pose both a confidentiality and an integrity threat to the user.

X was ported to SELinux to alleviate this problem by extending the SELinux model to the graphic server. The key idea is to get a fine-grained access control to enforce security policies that effectively isolate GUIs. To operate, the SELinux model requires that:

* Each ressource inside the X server must be labelled (class/domain)
* Each operation on a ressource must be checked (hooks)
* An engine separated from the hooks should take the decision to accept/deny a request from a X client onto a ressource

The hooks inside the X server are provided by the X Access Control Extension (XACE) which are in turn used by xauth, xhost and "XSELinux". The label on each ressource is either set during the allocation of said ressource or generated later on when performing an access control. The decision engine is provided by the libselinux and mostly runs in the kernel.

Most notably, XSELinux can control accesses to:

* X extensions: "use" and "query" (is it available?)
* X properties: "listprop" and "chprop" (change property)
* Clipboard: "read" and "write"
* Screen/pixels: "copy" (get pixels from the screen) and "transparency" (is transparency allowed?)
* Input: "setfocus", "grab" and "ungrab"

XSELinux provides a fine-grained access control that can be used to effectively isolate X clients from the X server point of view.

For more information, please visit http://selinuxproject.org/page/NBXWIN and read https://www.nsa.gov/research/files/selinux/papers/xorg07-paper.pdf.
