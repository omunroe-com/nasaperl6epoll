NAME
====

epoll

SYNOPSIS
========

    use epoll;

    my $epoll = epoll.new(maxevents => 1); # 1 is default

    $epoll.add($file-descriptor, :in, :out, :priority, :edge-triggered);

    # timeout in milliseconds, default -1 = block forever
    for $epoll.wait(:2000timeout)
    {
        say "{.fd} is ready for reading" if .in;
        say "{.fd} is ready for writing" if .out;
    }

    # Or use chained calls:

    for epoll.new.add(0, :in).wait
    {
        say "ready to read on {.fd}" if .in;
    }

DESCRIPTION
===========

Simple low level interface around the epoll(7) I/O event notification facility. It can monitor multiple file descriptors to see if I/O is possible on any of them. Mainly useful for interfacing with other NativeCall modules, since Perl itself has a rich I/O system. If you really want to use this with Perl `IO::Handle`s, you can use `native-descriptor()` to get a suitable descriptor.

class **epoll**
---------------

  * method **new**(:$maxevents = 1)

Create a new epoll object. Maxevents is the maximum number of events that can be returned from a single call to wait.

  * method **add**(int32 $file-descriptor, ...event flags...)

    Flags:
      :in             = EPOLLIN       = ready for read
      :out            = EPOLLOUT      = ready for write
      :priority       = EPOLLPRI      = urgent data available for read
      :edge-triggered = EPOLLET       = Edge Triggered
      :one-shot       = EPOLLONESHOT  = Disables after 1 event
      :mod            = EPOLL_CTL_MOD = Modify an existing file descriptor

:mod is equivalent to EPOLL_CTL_MOD to change the events for a file descriptor already added. It will also re-enable a file descriptor disabled by :one-shot mode.

For convenience, always returns the object itself, so you can chain calls.

  * method **remove**(int32 $file-descriptor)

Remove a file descriptor previously added.

  * method **wait**(int32 :$timeout = -1)

Wait for 1 or more events to occur on the add()ed file descriptors. You can specify an optional timeout in milliseconds.

Returns a List of up up to $maxevents **epoll-event**s.

class **epoll-event**
---------------------

  * method int32 **fd**()

The file descriptor for the event.

  * method uint32 **events**()

A bitmask of the events that occurred. You can check them like this:

if $event.events +& EPOLLIN {...}

or use the much easier:

  * method Bool **in**()

Ready to read

  * method Bool **out**()

Ready to write

Exceptions
----------

Throws Ad-hoc exceptions for any errors.

(Should save errno, and make real Exceptions -- patches welcome!)

LICENSE
=======

Copyright © 2017 United States Government as represented by the Administrator of the National Aeronautics and Space Administration. No copyright is claimed in the United States under Title 17, U.S.Code. All Other Rights Reserved.