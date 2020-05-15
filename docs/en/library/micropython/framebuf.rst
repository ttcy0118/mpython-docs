:mod:`framebuf` --- FrameBuffer Operation
=============================================

.. module:: framebuf
   :synopsis: FrameBuffer operation

This module provides a general frame buffer that can be used to create bitmap images and then send to display.

FrameBuffer Class
-----------------

The FrameBuffer class provides a pixel buffer that can be drawn using pixels, lines, rectangles, text, and even other FrameBuffers. Useful when generating output for display.

Example::

    import framebuf

    # FrameBuffer needs 2 bytes for every RGB565 pixel
    buffer=bytearray(10 * 100 * 2)
    fbuf = framebuf.FrameBuffer(buffer, 10, 100, framebuf.RGB565)

    fbuf.fill(0)
    fbuf.text('MicroPython!', 0, 0, 0xffff)

Build function
------------

.. class:: FrameBuffer(buffer, width, height, format, stride=width)

    Build a FrameBuffer object. parameter is:

        - *buffer* Is an object with a buffering protocol that must be large enough to contain each pixel defined by the width, height, and format of the framebuffer.
        - *width*  width of FrameBuffer in pixels
        - *height* height of FrameBuffer in pixels
        - *format* Specifies the pixel type used in framebuffer; The allowed values are listed under the following constants. These settings are used to encode the number of bits in the color value and the layout of those bits in the buffer. In the case of passing the color value C to the method, C is a small integer whose encoding depends on the format of framebuffer.
        - *stride*  is the number of pixels between each horizontal pixel line in framebuffer. The default is width, but you may need to adjust when implementing framebuffer in another larger framebuffer or screen.
        The buffer size must accommodate the increased step size. 
    Valid *buffer* , *width*, *height*, *format*  and optional *stride* must be specified. Invalid buffer size or size may cause unexpected errors.

Draw original shape
------------------------

Methods to draws shapes on framebuffer as follows.

.. method:: FrameBuffer.fill(c)

    Fills the entire framebuffer with the specified color.

.. method:: FrameBuffer.pixel(x, y[, c])

   If C is not given, the color value of the specified pixel is obtained. If C is given, the specified pixel is set to the given color. 

.. method:: FrameBuffer.hline(x, y, w, c)
.. method:: FrameBuffer.vline(x, y, h, c)
.. method:: FrameBuffer.line(x1, y1, x2, y2, c)

    Draws a line from a set of coordinates using the given color and 1 pixel thickness. The ``line`` method draws lines to the second set of coordinates, while the ``hline`` and ``vline``  methods draw horizontal and vertical lines respectively until a given length.

.. method:: FrameBuffer.rect(x, y, w, h, c)
.. method:: FrameBuffer.fill_rect(x, y, w, h, c)

    Draws a rectangle at a given location, size, and color. The ``rect`` method only draws 1 pixel outline, while the ``fill_rect`` method draws outline and interior.

Draw Text
------------

.. method:: FrameBuffer.text(s, x, y[, c])

    Use coordinates as top left corner of text to write text to `FrameBuffer` . The color of the text can be defined by optional parameters, but the default value is 1. The size of all characters is 8x8 pixels, Font cannot be changed at this time.


Other Methods
-------------

.. method:: FrameBuffer.scroll(xstep, ystep)

   Move the contents of `FrameBuffer` according to the given vector. This may leave footprints of previous colors in the `FrameBuffer` .

.. method:: FrameBuffer.blit(fbuf, x, y[, key])

 

    Draw another `FrameBuffer` on the current one at the given coordinate. If *key* is specified, it should be a color integer, and the corresponding color will be treated as transparent: all pixels with that color value will not be drawn.

    This method works between instances of `FrameBuffer` with different formats. However, due to color format mismatch, the resulting color may be unexpected.

Constant
---------

.. data:: framebuf.MONO_VLSB

    Monochrome (1bit) color format this defines a mapping in which bits in bytes are mapped vertically and bits 0 are closest to the top of the screen.
    Therefore, each byte occupies 8 vertical pixels. Subsequent bytes appear in successive horizontal positions until they reach the extreme right.
    Render the other bytes starting from the far left, 8 pixels lower.

.. data:: framebuf.MONO_HLSB

    Monochrome (1-bit) color format this defines the mapping of bits in a byte to be mapped horizontally. Each byte occupies 8 horizontal pixels, of which bit 0 is the extreme left.
    Subsequent bytes appear in successive horizontal positions until they reach the far right. Render more bytes on the next line, one pixel lower.

.. data:: framebuf.MONO_HMSB

    Monochrome (1 bit) color format this defines the mapping of bits in a byte to be mapped horizontally. Each byte occupies 8 horizontal pixels, of which bit 0 is the rxtreme left. 
    Subsequent bytes appear in successive horizontal positions until they reach the extreme right. Render more bytes on the next line, one pixel lower.

.. data:: framebuf.RGB565

    RGB（16bit，5 + 6 + 5）color format式

.. data:: framebuf.GS2_HMSB

    Grey scale（2bit）color format

.. data:: framebuf.GS4_HMSB

    Grey scale（4bit）color format


.. data:: framebuf.GS8

    Grey scale（8bit）color format
