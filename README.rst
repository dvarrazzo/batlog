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

The script can use batteries information from a few sources. In order of
preference:

1. ``/sys/class/power_supply/`` contains directories such as ``BAT0`` and
   ``AC``.  The ``uevent`` file contains all the details. Values seem usually
   expressed in micro-units but I'm not sure.

2. ``/proc/acpi/battery/`` seems `to be phased out`__.  Each directory contains
   an ``info`` file with static information (maker, design capacity...) and
   a ``state`` file with the current charge and discharging rate level.  The
   unit is expressed near each value.

.. __: http://askubuntu.com/questions/214379/where-did-proc-acpi-battery-bat0-xxx-go-in-12-10

The script samples data from the first source found and saves a log file after
each power supply available.  A new line is saved only if different from the
previous one.  Program start/stop info are printed on stderr.

Each log file has a titles line prefixed by ``:`` (printed when the program
starts) and a line for each sample.  Fields are separated by tab.  No unit
conversion is performed: data is just read from the source and dumped in
tabular form.


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
