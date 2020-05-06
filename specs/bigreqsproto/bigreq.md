<center>
<h1> Big Requests Extension </h1>
<h2> X Consortium Standard </h2>
<h3> Bob Scheifler </h3>
X Consortium
<h3> X Version 11, Release 7.7 </h3>
<h4> Version 2.0 </h4>
</center>
 <p style="font-size: 16px;"> Copyright © 1993, 1994 X Consortium </p>

<p style="font-size: 14px;"><em>
Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the “Software”), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:
</br></br>
The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.
</br></br>
THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE X CONSORTIUM BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
</br></br>
Except as contained in this notice, the name of the X Consortium shall not be used in advertising or otherwise to promote the sale, use or other dealings in this Software without prior written authorization from the X Consortium.
X Window System is a trademark of The Open Group.
</em>
</p>

---
## Table of Contents  
[1. Overview](#overview)<br>
[2. Requests](#requests)<br>
[3. Events and Errors](#events-and-errors)<br>
[4. Encoding](#encoding)<br>
[5. C language binding](#c-language-binding)<br>
[6. Acknowledgements](#acknowledgements)<br>


### 1. Overview

This extension enables the use of protocol requests that exceed 262140 bytes in length.

The core protocol restricts the maximum length of a protocol request to 262140 bytes, in that it uses a 16-bit length field specifying the number of 4-byte units in the request. This is a problem in the core protocol when joining large numbers of lines (PolyLine) or arcs (PolyArc), since these requests cannot be broken up into smaller requests without disturbing the rendering of the join points. It is also much more of a problem for protocol extensions, such as the PEX extension for 3D graphics and the XIE extension for imaging, that need to send long data lists in output commands.

This extension defines a mechanism for extending the length field beyond 16 bits. If the normal 16-bit length field of the protocol request is zero, then an additional 32-bit field containing the actual length (in 4-byte units) is inserted into the request, immediately following the 16-bit length field.

For example, a normal PolyLine encoding is:

**PolyLine**

| bytes | value       | enum       | field           |
| :----:|:-----------:|:----------:|:---------------:|
| 1     | 65          | X_PolyLine | opcode          |
| 1     |             |            | coordinate-mode |
|       | 0           | Origin     |                 |
|       | 1           | Previous   |                 |
| 2     | 3+n         |            |                 |
| 4     | DRAWABLE    |            | drawable        |
| 4     | GCONTEXT    |            | gc              |
| 4n    | LISTofPOINT |            | points          |

An extended-length PolyLine encoding is:

**PolyLine**

| bytes | value       | enum      | field                |
| :---:|:-----------:|:----------:|:--------------------:|
| 1    | 65          | X_PolyLine | opcode               |
| 1    |             |            | coordinate-mode      |
|      | 0           | Origin     |                      |
|      | 1           | Previous   |                      |
| 2    | 0           |            | extended length flag |
| 4    | 4+n         |            | request length       |
| 4    | DRAWABLE    |            | drawable             |
| 4    | GCONTEXT    |            | gc                   |
| 4n   | LISTofPOINT |            | points               |



Extended-length protocol encodings, once enabled, can be used on all protocol requests, including all extensions.

---

### 2. Requests

BigReqEnable

=>

maximum-request-length: CARD32

This request enables extended-length protocol requests for the requesting client. It also returns the maximum length of a request, in 4-byte units, that can be used in extended-length protocol requests. This value will always be greater than the maximum-request-length returned in the connection setup information.

---

### 3. Events and Errors

No new events or errors are defined by this extension.

---

### 4. Encoding

Please refer to the X11 Protocol Encoding document as this document uses conventions established there.

The name of this extension is “BIG-REQUESTS”.

**BigReqEnable**

| bytes | value  |         field          |
| :---: | :----: | :--------------------: |
|   1   | Card8  |         opcode         |
|   1   |   0    |     bigreq opcode      |
|   2   |   1    |     request length     |
|  =>   |        |                        |
|   1   |   1    |         Replay         |
|   1   |        |         unused         |
|   2   | CARD16 |    sequence number     |
|   4   |   0    |         length         |
|   4   | CARD32 | maximum-request-length |
|   2   |   0    |         unused         |

---

### 5. C language binding

It is desirable for core Xlib, and other extensions, to use this extension internally when necessary. It is also desirable to make the use of this extension as transparent as possible to the X client. For example, if enabling of the extension were delayed until the first time it was needed, an application that used [XNextRequest](https://www.x.org/releases/X11R7.7/doc/libX11/libX11/libX11.html#XNextRequest "XNextRequest") to determine the sequence number of a request would no longer get the correct sequence number. As such, [XOpenDisplay](https://www.x.org/releases/X11R7.7/doc/libX11/libX11/libX11.html#XOpenDisplay "XOpenDisplay") will determine if the extension is supported by the server and, if it is, enable extended-length encodings.

The core Xlib functions [XDrawLines](https://www.x.org/releases/X11R7.7/doc/libX11/libX11/libX11.html#XDrawLines "XDrawLines"), [XDrawArcs](https://www.x.org/releases/X11R7.7/doc/libX11/libX11/libX11.html#XDrawArcs "XDrawArcs"), [XFillPolygon](https://www.x.org/releases/X11R7.7/doc/libX11/libX11/libX11.html#XFillPolygon "XFillPolygon"), [XChangeProperty](https://www.x.org/releases/X11R7.7/doc/libX11/libX11/libX11.html#XChangeProperty "XChangeProperty"), [XSetClipRectangles](https://www.x.org/releases/X11R7.7/doc/libX11/libX11/libX11.html#XSetClipRectangles "XSetClipRectangles"), and [XSetRegion](https://www.x.org/releases/X11R7.7/doc/libX11/libX11/libX11.html#XSetRegion "XSetRegion"). are required to use extended-length encodings when necessary, if supported by the server. Use of extended-length encodings in other core Xlib functions ([XDrawPoints](https://www.x.org/releases/X11R7.7/doc/libX11/libX11/libX11.html#XDrawPoints "XDrawPoints"), [XDrawRectangles](https://www.x.org/releases/X11R7.7/doc/libX11/libX11/libX11.html#XDrawRectangles "XDrawRectangles"), [XDrawSegments](https://www.x.org/releases/X11R7.7/doc/libX11/libX11/libX11.html#XDrawSegments "XDrawSegments"). [XFillArcs](https://www.x.org/releases/X11R7.7/doc/libX11/libX11/libX11.html#XFillArcs "XFillArcs"), [XFillRectangles](https://www.x.org/releases/X11R7.7/doc/libX11/libX11/libX11.html#XFillRectangles "XFillRectangles"), [XPutImage](https://www.x.org/releases/X11R7.7/doc/libX11/libX11/libX11.html#XPutImage "XPutImage") is permitted but not required; an Xlib implementation may choose to split the data across multiple smaller requests instead.

To permit clients to know what the maximum-request-length for extended-length encodings is, the following function is added to Xlib:

`long XExtendedMaxRequestSize(Display *display);`

Returns zero (0) if the specified display does not support this extension, otherwise returns the maximum-request-length (in 4-byte units) supported by the server through the extended-length encoding.

---

### 6. Acknowledgements

Clive Feather (IXI) originated the extended-length encoding used in this extension proposal.
