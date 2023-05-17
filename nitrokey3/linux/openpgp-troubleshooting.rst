OpenPGP Troubleshooting
=======================

.. contents:: :local:

GnuPG: OpenPGP Card Not Available
---------------------------------

**Problem:**
GnuPG cannot access the Nitrokey 3 and shows an error message like this::

    $ gpg --card-status 
    gpg: selecting openpgp failed: No such device
    gpg: OpenPGP card not available: No such device

**Solution:**
There are two common smartcard services on Linux systems: ``scdaemon``, GnuPG’s smartcard daemon, and ``pcscd``, a generic smartcard daemon.
``scdaemon`` has two drivers for accessing smartcards:
Its integrated ``ccid`` driver tries to directly access the smartcard.
The ``pcsc`` drivers uses the ``pcscd`` daemon instead.

A smartcard can only be accessed directly by one daemon.
This means that depending on the startup order, ``pcscd`` might lock the card before ``scdaemon`` tries to access it using the internal ``ccid`` driver.
Therefore we recommend to use the ``pcscd`` driver for ``scdaemon``.
You can activate it by adding ``disable-ccid`` to the ``~/.gnupg/scdaemon.conf`` config file and restarting ``scdaemon``, for example with ``gpg-connect-agent "SCD KILLSCD" /bye``.
If this does not fix the problem, see the next section for more information.

Alternatively, you can deactivate or uninstall ``pcscd`` to avoid this conflict.

pcscd: Card Not Found
---------------------

**Problem:**
An application using ``pcscd`` does not show the Nitrokey 3.

**Solution:**
First, make sure that ``scdaemon`` is not running (see the previous section)::

    $ gpg-connect-agent "SCD KILLSCD" /bye

Now list the smartcards recognized by ``pcscd`` with ``pcsc_scan -r``.
You should see an entry like this one::

    $ pcsc_scan -r
    Using reader plug'n play mechanism
    Scanning present readers..
    0: Nitrokey 3 [CCID/ICCD Interface] 00 00

If the Nitrokey 3 shows up, it is recognized correctly by ``pcscd`` and there might be an issue with the application that tries to access it.
If it does not show up, make sure that your ``libccid`` version is up to date.
Support for the Nitrokey 3 was added in ``libccid`` 1.5.0.

Updating The Device Database
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you cannot update ``libccid`` to a supported version, you have to manually update the device database.
The path of the database depends on your distribution:

- Arch, Debian, Ubuntu: ``/etc/libccid_Info.plist``

Make sure to backup the file before overwriting it.
You can download an `updated device database file <https://github.com/Nitrokey/nitrokey-3-firmware/blob/main/Info.plist>`__ from the ``nitrokey-3-firmware`` repository.
After updating the file, restart ``pcscd`` and run ``pcsc_scan -r`` again.
The Nitrokey 3 should now show up.
