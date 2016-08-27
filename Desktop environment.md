# Desktop environment relalated issues

Note: this is not about any specific DE.

## Security

### Sandboxed apps
  
This is not required for all apps out there, but it should be possible to install an application
without providing it all the permissions by default, but allowing it to request more permissions.
Should be about the same from the usability point of view as is done in iOS and newer Android versions.

## Resources

### Memory

It's quite easy to get e.g. Kate, a text editor, consume 1.3 GiB on zero open files.
And no, no gc involved — it's written in C++ and Qt.
And no, that is not caused by memory leaks.
And no, that memory is actually being blocked and is not returned to the system.

This happens because of internal glibc memory malloc optimizations for some common cases,
due to which it uses `sbrk()` to allocate small memory chunks by extending the DS.
Strings are often small memory chunks,
and almost everything that is being allocated is allocated using small memory chunks.

This results in the sutuation when e.g. opening a large number of files and then closing them does not return the
memory back to the system, as something else has been placed in the DS after that freed memory (e.g. undo history).

That explains why doesn't the text editor give claimed memory back to system when all documents were closed,
but I'm still not sure why does a text editor consume a few MiBs per open file. I believe that could be optimized.

### Disk operations

Check what disk operations does your DE perform.
Does it perform read operations on `/etc/passwd` 130 times per second?
Does it perform write operations in browser profile every second, even when you are doing nothing?
Things like that shouldn't happen, but they are quite common.

## Looks

### Vector icons everywhere

Seriously, it's 2016, and there are still raster icons in random places, and those look blurry.
Vector icons could be rendered only the first time and cached, so the performance is not the issue here.
Some source images are incompatible with different renderers and are displayed correctly only in the editor,
because of subtle rendering bugs, but that could be fixed, as those icons are quire rare.
