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
  -h, --help         show this help message and exit
  --logdir=DIR       where to create log files [default: .]
  --fields=FIELDS    fields to include; the file is not created if no field is
                     selected; comma-separated list [default: all]
  --interval=SEC     how often to sample the battery level [default: 60 sec]
  --verbose          print more details on stderr

The script can use batteries information from a few sources. In order of
preference:

1. ``/sys/class/power_supply/`` contains directories such as ``BAT0`` and
   ``AC``.  The ``uevent`` file contains all the details. Values seem usually
   expressed in micro-units, see table below.

2. ``/proc/acpi/battery/`` seems `to be phased out`__.  Each directory contains
   an ``info`` file with static information (maker, design capacity...) and
   a ``state`` file with the current charge and discharging rate level.  The
   unit is expressed near each value.

.. __: http://askubuntu.com/questions/214379/where-did-proc-acpi-battery-bat0-xxx-go-in-12-10

The script samples data from the first source found and saves a log file after
each power supply available.  Program start/stop info are printed on stderr.

Each log file has a titles line prefixed by ``:`` (printed when the program
starts) and a line for each sample.  Fields are separated by tab.  No unit
conversion is performed: data is just read from the source and dumped in
tabular form.

A new line is saved only if different from the previous one. Note that some
fields may be volatile and not really relevant to battery profiling (e.g. the
``VOLTAGE_NOW``): you may want to exclude these fields from the output (using
``--fields``) to minimize the lines saved.


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


Fields available
----------------

On Ubuntu 12.04 with kernel 3.2.0-44, this is the relation between the fields
in the ``/sys/class/power_supply/NAME/uevent`` and the ones in
``/proc/acpi/battery/NAME/state`` and ``info``. All the fields in ``uevent``
have a ``POWER_SUPPLY_`` prefix. The fields refer to batteries only, not other
types of power supplies (e.g. A.C.).

================== =========== ================== =========== ======================= =============
``uevent``         sample      ``state``          sample      ``info``                sample
================== =========== ================== =========== ======================= =============
NAME               BAT0
STATUS             Discharging charging state     discharging
PRESENT            1           present            yes         present                 yes
TECHNOLOGY         Li-ion                                     battery type            LION
CYCLE_COUNT        0                                          cycle count             0
VOLTAGE_MIN_DESIGN 11100000                                   design voltage          11100 mV
VOLTAGE_NOW        11258000    present voltage    11248 mV
CURRENT_NOW        1469000     present rate       1465 mA
CHARGE_FULL_DESIGN 7800000                                    design capacity         7800 mAh
CHARGE_FULL        6677000                                    last full capacity      6677 mAh
CHARGE_NOW         2889000     remaining capacity 2888 mAh
MODEL_NAME         42T4799                                    model number            42T4799
MANUFACTURER       SANYO                                      OEM info                SANYO
SERIAL_NUMBER      13660                                      serial number           13660
n.a.                           capacity state     ok
n.a.                                                          battery technology      rechargeable
n.a.                                                          design capacity warning 333 mAh
n.a.                                                          design capacity low     18 mAh
n.a.                                                          capacity granularity 1  1 mAh
n.a.                                                          capacity granularity 2  1 mAh
================== =========== ================== =========== ======================= =============


Simple plot example
-------------------

You can create a basic plot of the gathered data using matplotlib_::

    $ ipython --pylab

    In [1]: import matplotlib.pyplot as plt

    In [2]: list(enumerate(open('BAT0.uevent.log').readline().split('\t')))
    Out[2]:
    [(0, ':timestamp'),
     (1, 'STATUS'),
     (2, 'CHARGE_FULL'),
     (3, 'CHARGE_NOW')]

    In [3]: data = [ l.split('\t') for l in open('BAT0.uevent.log')
        if not l.startswith(':') ]

    In [4]: plt.plot(
        [datetime.datetime.strptime(r[0], '%Y-%m-%d %H:%M:%S') for r in data],
        [float(r[3]) / 1000 for r in data], '.')

    In [5]: plt.ylabel('Capacity (mAh)')

.. _matplotlib: http://matplotlib.org/
