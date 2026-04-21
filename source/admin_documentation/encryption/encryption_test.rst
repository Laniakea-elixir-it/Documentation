File System Encryption Test
===========================
Test executed to ensure LUKS volume encryption.

#. Create two volumes, here named vol1, vol2.

#. Attach each one to the instance (here listed as ``/dev/vdd`` and ``/dev/vde``) and mount them respectively to ``/export`` and ``/export1``.

   ::

     $ df -h
     Filesystem      Size  Used Avail Use% Mounted on
     ...
     /dev/vdd        976M  2.6M  907M   1% /export
     /dev/vde        976M  2.6M  907M   1% /export1

#. Encrypt ``/export``, i.e. ``/dev/vdd`` using fast_luks (``/export`` is the default value).

   ::

     $ df -h
     Filesystem            Size  Used Avail Use% Mounted on
     ...
     /dev/vde              976M  2.6M  907M   1% /export1
     /dev/mapper/jtedehex  990M  2.6M  921M   1% /export

   Ensure that ``/export`` has the same permissions of the other two volumes.

   ::

     drwxr-xr-x.   3 centos centos 4096 Nov  9 10:27 export
     drwxr-xr-x.   3 centos centos 4096 Nov  9 10:27 export1


#. Put the same file on both volumes:

   ::

     $ echo "encryption test" > /export/test.txt
     $ echo "encryption test" > /export1/test.txt

#. Umount all the volumes and luksClose the encrypted one:

   ::

     $ sudo cryptsetup luksClose /dev/mapper/jtedehex

#. Create the volume binary image using ``dd``:

   ::

     sudo dd if=/dev/vdd of=/home/centos/vdd_out
     2097152+0 records in
     2097152+0 records out
     1073741824 bytes (1.1 GB) copied, 21.809 s, 49.2 MB/s

     $ sudo dd if=/dev/vde of=/home/centos/vde_out
     2097152+0 records in
     2097152+0 records out
     1073741824 bytes (1.1 GB) copied, 21.3385 s, 50.3 MB/s

#. HexDump the binary image with ``xdd``:

   ::

     $ xxd vdd_out > vdd.txt

     $ xxd vde_out > vde.txt

   As output you should have:

   ::

     $ ls -ltrh
     -rw-r--r--.  1 root   root   1.0G Nov  9 11:19 vdd_out
     -rw-r--r--.  1 root   root   1.0G Nov  9 11:22 vde_out
     -rw-rw-r--.  1 centos centos 4.2G Nov  9 11:32 vdd.txt
     -rw-rw-r--.  1 centos centos 4.2G Nov  9 11:36 vde.txt

#. Grep non-zero bytes and search for the test.txt file content ``encryption test``: 

   ::

     $ grep -v "0000 0000 0000 0000 0000 0000 0000 0000" vde.txt > grep_vde.txt
     $ grep "encryption test" grep_vde.txt 
     8081000: 656e 6372 7970 7469 6f6e 2074 6573 740a  encryption test. 

     $ grep -v "0000 0000 0000 0000 0000 0000 0000 0000" vdd.txt > grep_vdd.txt
     $ grep "encryption test" grep_vdd.txt 
     $ 

   .. Note::

      It is possible to see the test.txt file content only on the un-encrypted volume.


   Moreover, the output file grep_vde.txt is 73 kb while the encrypted one, grep_vdd.txt (138 MB), is very large:

   ::

     -rw-rw-r--.  1 centos centos  73K Nov  9 11:46 grep_vde.txt
     -rw-rw-r--.  1 centos centos 138M Nov  9 11:58 grep_vdd.txt

We also tried to open the volume when active (LUKS volume opened and mounted, Galaxy running) in the Virtual Machine, using the cloud controller (as administrator).

Test executed on the cloud controller:

::

  # rbd map volume-3bedc7bc-eaed-466f-9d55-f2c29b44a7b2 --pool volumes
  /dev/rbd0
  
  # lsblk -f
  NAME          FSTYPE      LABEL UUID                                   MOUNTPOINT
  sda                                                                    
  |-sda1        ext4              db06fc46-7231-4189-ba2b-0b0117049680   /boot
  |-sda2                                                                 
  |-sda5        swap              e5b98538-8337-4e25-8f82-f97f04258716   [SWAP]
  `-sda6        LVM2_member       n4SAgY-GRNy-4Fl2-ROoQ-rRIf-bdBP-QC1B6s
    `-vg00-root ext4              1e3f1ff1-8677-4236-8cb4-07d5cad32441   /
  rbd0          crypto_LUKS       c4bee3b9-e0dc-438e-87ae-2a3e491081c0   
  
  # mount /dev/rbd0 /mnt/
  mount: unknown filesystem type ‘crypto_LUKS’

It is not possible to mount the volume without the user password.
