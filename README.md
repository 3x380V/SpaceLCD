# SpaceLCD

## Subtitle

![Photo showing SpaceLCD logo](/img/lcd_photo.jpg)

**SpaceLCD** displays SVG or PNG images on the SpaceExplorer Enterprise 3D mouse. It can also contol the LCD brightness.

## Dependencies

SpaceLCD requires libusb to control the 3D mouse, and also zlib to compress the images. Basic SVG support is provided by [NanoSVG](https://github.com/memononen/nanosvg), which is included. If you wish to additionally display PNG images, libpng is also needed.

## Usage

splcd [-b] [-c] [-v] [-l brightness] [-r] (<file.svg>|<file.png>|<file.raw>)
-r reset
-l set lcd brightness

The -w flag prevents parsing of image data, and instead transfers the raw file as a bulk USB message. The -W flag performs the same function but additionally disables some sanity checks on the data. 

The -r flag sends the reset signal to the mouse, which will revert the display to the boot logo. This can sometimes prevent the need to power-cycle the device if it receives corrupted data.

The -l flag sets the brightness of the LCD backlight. Valid values are in the range 0-100, although the actual 

## Background

The SpaceMouse Enterprise has a 640x150 pixel display with 16-bit R5G6B5 colour support. Images are sent to the display with a 512-byte header, followed by the bitmap, which is compressed using raw deflate.

SpaceLCD is only used to control the display; [spacenavd](http://spacenav.sourceforge.net) and [libspnav](http://spacenav.sourceforge.net) are required in order to read the motion data from the mouse.

## Limitations

The official 3DConnexion driver has some support for animation, which is not currently understood and therefore not implemented here. The proprietary driver can also use partial screen updates, whereas SpaceLCD uploads the full screen each time.

SpaceLCD has only been tested using one SpaceExplorer Enterprise, with USB ID 256f:c633, bought as part of the SpaceMouse Enterprise Kit 3DX-700058.

Operating system support is currently GNU/Linux only.

## Endianness Alert

Because this code has only been developed on an x86 system, the endianness of 16-bit words has probably been taken for granted in several places, so it may work incorrectly on another architecture. Something for a future fix.

## Notes

USB (brightness)
	bmrequest: 0x21
	brequest: 9
	wvalue: 0x0311
	windex: 1
	wlength: 2
	data fragment: 1100 (off) 1107 (dim) 1160 (very bright) 1164 (max)

USB (change lcd graphics)
	URB_CONTROL
		bmrequest: 0x21
		brequest: 9
		wvalue: 0x0314
		windex: 1
		wlength: 5
		data fragment: 14fcff071c
	URB_BULK
		words are little-endian
		180f start of bulk header (512 bytes)
		next two bytes are size of payload following first 512 bytes (also byte 29/30)
		3e1a 2dff common
		110f complete redraw
		180f partial redraw of titles?
		1805 a360
USB_CONTROL
	0x0f,0x01 perhaps flips the display to show the alternate bitmap buffer

Main body after first 512 bytes:
	eb2b ee1b
		change ee1b to ee1c changes the grey colour
		then pixel data?

		28 1c 85 a3 70 14 8e c2 51 38 0a 47 e1
Splash screen:
	reset
	make (block_whole_screen_no_scroll)
	shutdown

To enable libusb debugging
	: libusb_set_option(NULL, LIBUSB_OPTION_LOG_LEVEL, * LIBUSB_LOG_LEVEL_DEBUG); 

The data may be compressed using gzip compression.

Create a bitmap, save as 565 rgb, dd skip=70 if=<file.bmp> bs=1 status=none | gzip -c

{ printf "\x1f\x8b\x08\x00\x00\x00\x00\x00\x00\x00"; dd skip=512 if=/tmp/<file> bs=1 status=none count=128; } > /tmp/file

{ printf "\x78\x9c"; dd if=bytes/wholechrome.bulk bs=1 count=8 status=none skip=512; } | ./zpipe -d | xxd

lcd image converter

binwalk -X <file>

## Unbinding from USB HID

dmesg to get the hid-generic number XXXX:XXXX:XXXX.XXXX

Then

sudo sh -c 'echo -n "XXXX:XXXX:XXXX.XXXX" > /sys/bus/hid/drivers/hid-generic/unbind

## Screen Resolution

640 x 144, 16-bit bgr

## To swap little/big endian

dd conv=swab

## Byte Meanings

0x00: Screen change type. 0x11 = instant, 0x14 = down, 0x15 = up, 0x16 = left, 0x17 = right, 0x18 is unknown but related to partial update of screen
0x01: 0x0f
0x02-0x03(little-endian): The length of the data stream starting from byte 512 (mirrored in bytes 0x1c/0x1d)
0x04-0x0a: zero?
0x0e-0x0f: 0x000f
0x28-0x29:The only bytes that differ between two partial screen changes

pre-0x8e73: 0x02 0x0100 0x0000
0x40-0x200: 0x8e73 repeated

## Bugs

## Example

~~~{.py}
import sys
~~~

## Copyright and Licencing

spacelcd is released under [zlib license](LICENSE.txt) 
