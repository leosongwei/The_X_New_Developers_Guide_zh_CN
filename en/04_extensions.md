Modern Extensions To X
======================

*Alan Coopersmith*

As described in the "Communication Between Client and Server" chapter, the X Window System has evolved over time via the X11 protocol's extension mechanism, which allows defining new protocol requests, events and errors that servers may support. Most extensions include a versioning mechanism that allows clients to discover which features the extension supports.

Many features required in a modern window system had to be invented when the X protocol was designed, many years ago. Thus, most applications require extensions, to get the benefits of decades of experience in the form of better infrastructure. At the least, application developers need to know about the effects the extensions will have on their applications.

Some extensions are used by many clients, but are hidden under the covers of the X libraries (Xlib or XCB) so that clients never call them directly. For example, the Big-Requests extension allows libraries to send larger request messages than the original X11 protocol allowed; the XC-MISC extension allows libraries to request XID's when their original allocation is exhausted.

There are several extensions that modern application authors should be familiar with, since they are widely used and important for interoperation in the modern world. These include XKB, Xinput2, Composite, RandR, Xinerama and Sync.

XKB
---

Keyboard input and keyboard layout management today involves international keyboard layouts, vendor-specific keyboard models, and complex user preferences. This is a situation far more complex than the original keyboard support in the X11 protocol anticipated. Programmers writing against the Xlib API will find some Xlib keyboard functions now call XKB support under the hood. Other Xlib keyboard calls are deprecated and should be replaced in applications by calls to replacement XKB functions in Xlib.

[Much more information is needed here. --po8]

Xinput 2
--------

In the core X protocol, the most significant distinction between input devices is the number of buttons they have. This input model quickly became too limited to support the wide array of input devices that users wanted to attach to an Xserver. A variety of extensions were proposed to fill the gap. The one that won out and became standardized in the early 1990's was Xinput, which provided for a variety of input devices and input types. Xinput stayed static for about 15 years. When the growth of laptops and mobile devices demanded even more changes, Xinput version 2 was developed.

As of this writing, the latest version of Xinput is Xinput 2.2, included in the Xorg 1.12 release. Xinput 2.2 adds multitouch support. Clients can, for example, now distinguish between a one-finger slide and a two-finger swipe. Multitouch support adds on to enhancements in previous versions. These included input device properties to pass additional metadata and configuration information between devices and clients; also the MultiPointer X (MPX) feature. MPX allows for multiple X users to share a single X desktop. Each user may have their own on-screen cursor image and input focus, driven by their own mouse and keyboard devices.

Much more information about modern Xinput, including code and configuration examples, can be found on Peter Hutterer's blog at http://who-t.blogspot.com/ .

Composite
---------

X was originally designed to run on computers with just a couple of megabytes of RAM. In these environments, memory could not be spared to keep a complete copy of the image of every window drawn by every application. Instead, the hardware frame buffer was used as the only pixel store, storing the current screen appearance for display. When windows were moved, raised or uncovered, clients received Expose events telling them to redraw the newly visible portions of the window. Unfortunately, this redraw often caused flickering or similar effects when windows moved. Redraw also led to increased application complexity. Every application had to be prepared to efficiently redraw any portion of its windows at any time. This tended to cause applications to build internal display lists, keep internal rasters, etc to meet this obligation---activities arguably better performed in a unified fashion by the server.

Modern computers have plenty of RAM for rendering. X can now afford to trade off RAM for greater responsiveness, less redraw noise on screen and simpler applications. The Composite extension enables full buffering of every pixel of every window to off-screen memory. Composite combines these off-screen buffers into the frame buffer as needed, producing the image seen on screen. Because of its position in the stack, Composite can conveniently invoke an external "compositing manager" client to apply any desired "special effects"---for example, translucency, blurring or drop shadows---along the way.

Unfortunately, the Composite extension is not yet universally deployed, and in any case is incompatible with certain other X configuration options. Application authors must ensure they still handle Expose events properly, and test that functionality on systems with Composite disabled. However, Expose event processing in modern X applications tends to be greatly simplified; it is assumed that application redraw performance is no longer as serious an issue as it once was.

RandR and Xinerama
------------------

An X Screen was originally a direct mapping to a single monitor. Users with multiple monitors had a separate logical screen corresponding to each monitor: there was no opportunity to have a window straddle monitors or even move from one to another. Users demanded more functionality. Thus, in the mid-nineties, X11R6.4 added the Xinerama extension to combine multiple output devices into a single logical screen across which windows could cross freely. Xinerama actually provided two closely-related functions in one extension: multiplexing across devices in the X server, and protocol permitting a client to query the X server to discover device boundaries underlying each logical screen.

Later, hardware then advanced to the point it was feasible, then common, for a single graphics adapter to drive multiple output devices. The Xinerama protocol was reused for these devices: this enabled retrieving monitor information from multi-head cards, even though the multiplexing feature was no longer required for this use case.

Users also demanded the ability to change their monitor configuration or to add another output (such as plugging a projector into a laptop) "on the fly", without restarting the X server. The X Resize, Rotate and Reflect Extension (abbreviated to XRandR before reflection was added---the name stuck) responded to this need. XRandR 1.2 and later allow querying and setting monitor layouts for multiple screens per device. Xrandr 1.2 also added device properties, providing additional device-specific metadata and configuration options. Xrandr 1.3 added output transformations and panning support.

Unfortunately, if your applications desire knowledge of the screen layout, it needs to be aware of both extensions. Not all systems support Xrandr 1.2 yet: proprietary drivers or servers tend to be particularly problematic. The Xinerama multiplexer may be configured still for combining screens across multiple discrete graphics adaptors, but Xinerama provides much less information about the outputs than XRandR does. Only XRandR allows your application to register to receive events when the layout or device information changes.

SYNC
----

The core X protocol only guarantees that it processes the requests on each connection in order of receipt on that connection. It makes no guarantees about ordering with respect to requests on different connections, or with respect to system events or clocks. The X Synchronization Extension (SYNC) provides a constraint system to handle these cases. An application can instruct the X server to hold off on processing further requests on a connection until a given synchronization point; the application can also instruct the server to send an event to a client at a given time, for instance when a idle timeout is reached after a period of user inactivity.

For example, a client wishing to avoid tearing effects in their drawing may send the server all the requests needed to update an off-screen pixmap, followed by a request to wait until the screen's vertical refresh interval begins. The client can then copy from the pixmap to the on-screen window during the refresh period, rather than in the middle of a display controller update of that window.

If the standard synchronization points provided by SYNC are not sufficient, clients can also define their own sychronization points. For example, this enables a client to make updates synchronized with actions from another cooperating client.
