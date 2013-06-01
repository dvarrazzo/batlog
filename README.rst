Lifetime battery charge and capacity logger
===========================================

Copyright (C) 2013 Daniele Varrazzo <daniele.varrazzo@gmail.com>

Monitor lifelong battery charge and capacity. From a `wavexx idea`__, but for
Linux.

.. __: http://www.thregr.org/~wavexx/hacks/bcmon/


Usage
-----

Usage: ``batlog`` [*options*]

Options:
  -h, --help      show this help message and exit
  --logdir=DIR    where to create log files [default: .]
  --interval=SEC  how often to sample the battery level [default: 60 sec]
  --verbose       print more details on stderr

The script samples batteries ACPI info from ``/proc/acpi/battery/`` and saves
information into ``BAT?.info.log`` and ``BAT?.state.log`` file.  A new line
is saved only if different from the previous one.  Program start/stop info are
printed on stderr.

Each log file has a titles line  prefixed by ``:`` (printed when the program
starts) and a line for each sample.  Fields are separated by tab.


Installation
------------

The ``batlog`` script can be run anywhere and doesn't require special
privileges. By default it will save the batteries logs in the current
directory.

If you want to log the system continuously you can copy the ``batlog`` script
into ``/usr/local/bin/`` and ``batlog.conf`` into ``/etc/init/``: the logger
will start automatically when the system starts and log battery info into
``/var/log/batlog/``.  Use ``sudo service batlog start/stop`` to control the
program.
