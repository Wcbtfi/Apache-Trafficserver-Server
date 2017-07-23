.. Licensed to the Apache Software Foundation (ASF) under one
   or more contributor license agreements.  See the NOTICE file
   distributed with this work for additional information
   regarding copyright ownership.  The ASF licenses this file
   to you under the Apache License, Version 2.0 (the
   "License"); you may not use this file except in compliance
   with the License.  You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing,
   software distributed under the License is distributed on an
   "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
   KIND, either express or implied.  See the License for the
   specific language governing permissions and limitations
   under the License.

==============
storage.config
==============

.. configfile:: storage.config

:file:`storage.config` 文件 ( 默认位于 ``/usr/local/etc/trafficserver/`` ) , 列出组成 Traffic Server 缓存的所有文件, 目录和/或硬盘分区. 修改 :file:`storage.config` 文件后, 新的配置不会立即生效, 需要重新启动 Traffic Server .

格式
======

 :file:`storage.config` 文件格式是: 

   *pathname* *size* [ ``volume=``\ *number* ] [ ``id=``\ *string* ]

其中 :arg:`pathname` 是一个分区的名称, 目录或文件; :arg:`size` 是命名的分区, 目录或文件(bytes)的大小; :arg:`volume` 是用在 :file:`volume.config` 和 :file:`hosting.config` 里的卷号; :arg:`id` 用于:ref:`assignment-table`. 必须指定目录的大小; 在文件和 raw 分区里, 大小是可选的. :arg:`volume` 和 arg:`seed` 是可选的.

.. 注意::

   The :arg:`volume` option is independent of the :arg:`seed` option and either can be used with or without the other,
   and their ordering on the line is irrelevant.

.. note::

   If the :arg:`id` option is used every use must have a unique value for :arg:`string`.

可以使用任何大小的任何分区.  For best performance:

-  使用 raw 磁盘分区.
-  对于每个磁盘, 所有分区大小相同.
-  对于每个节点, 在所有磁盘上使用相同数量的分区.
-  将相似的存储分组到不同的卷中. 例如, 将 SSD 或 RAM 驱动器拆分成自己的卷.

Specify pathnames according to your operating system requirements. See
the following examples. In the :file:`storage.config` file, a formatted or
raw disk must be at least 128 MB.

When using raw disk or partitions, you should make sure the :ts:cv:`Traffic
Server user <proxy.config.admin.user_id>` used by the Traffic Server process
has read and write privileges on the raw disk device or partition. One good
practice is to make sure the device file is set with 'g+rw' and the Traffic
Server user is in the group which owns the device file.  However, some
operating systems have stronger requirements - see the following examples for
more information.

As with standard ``records.config`` integers, human readable prefixes are also
supported. They include

   - ``K`` Kilobytes (1024 bytes)
   - ``M`` Megabytes (1024^2 or 1,048,576 bytes)
   - ``G`` Gigabytes (1024^3 or 1,073,741,824 bytes)
   - ``T`` Terabytes (1024^4 or 1,099,511,627,776 bytes)

.. _assignment-table:

Assignment Table
----------------

Each storage element defined in :file:`storage.config` is divided in to :term:`stripes <cache stripe>`. The
assignment table maps from an object URL to a specific stripe. The table is initialized based on a
pseudo-random process which is seeded by hashing a string for each stripe. This string is composed
of a base string, an offset (the start of the stripe on the storage element), and the length of the
stripe. By default the path for the storage is used as the base string. This ensures that each
stripe has a unique string for the assignment hash. This does make the assignment table very
sensitive to the path for the storage elements and changing even one can have a cascading effect
which will effectively clear most of the cache. This can be problem when drives fail and a system
reboot causes the path names to change.

The :arg:`id` option can be used to create a fixed string that an administrator can use to keep the
assignment table consistent by maintaing the mapping from physical device to base string even in the presence of hardware changes and failures.

例子
========

下面的示例显示在 ``/big_dir`` 目录中128MB缓存存储::

   /big_dir 134217728

可以使用 ``.`` 表示当前目录. 以下是在当前目录有 64MB 缓存存储::

   . 134217728

另一种方式是使用可读后缀, 可以使用以下命令来表达 64GB 缓存文件::

   /really_big_dir 64G


.. note::
    When using on-filesystem cache disk storage, you can only have one such
    directory specified. This will be address in a future version.


Solaris Example
---------------

The following example is for the Solaris operating system::

   /dev/rdsk/c0t0d0s5
   /dev/rdsk/c0t0d1s5

.. note:: Size is optional. If not specified, the entire partition is used.

Linux 例子
-------------
.. note::
    Rather than refer to disk devices like ``/dev/sda``, ``/dev/sdb``, etc.,
    modern Linux supports `alternative symlinked names for disk devices
    <https://wiki.archlinux.org/index.php/persistent_block_device_naming#by-id_and_by-path>`_ in the ``/dev/disk``
    directory structure. As noted for the :ref:`assignment-table` the path used for the disk can effect
    the cache if it changes. This can be ameloriated in some cases by using one of the alternate paths
    in via ``/dev/disk``. Note that if the ``by-id`` or ``by-path`` style is used, replacing a failed drive will cause
    that path to change because the new drive will have a different physical ID or path. The original hash string can
    be kept by adding :arg:`id` or :arg:`path` with the original path to the storage line.

    If this is not sufficient then the :arg:`id` or :arg:`path` argument should be used to create a more permanent
    assignment table. An example would be::

       /dev/sde id=cache.disk.0
       /dev/sdg id=cache.disk.1

The following example will use an entire raw disk in the Linux operating
system::

   /dev/disk/by-id/[DiskA_ID]    volume=1
   /dev/disk/by-path/[DiskB_Path]   volume=2

In order to make sure :program:`traffic_server` will have access to this disk
you can use :manpage:`udev(7)` to persistently set the right permissions. The
following rules are targeted for an Ubuntu system, and stored in
``/etc/udev/rules.d/51-cache-disk.rules``::

   # Assign DiskA and DiskB to the tserver group
   # make the assignment final, no later changes allowed to the group!
   SUBSYSTEM=="block", KERNEL=="sd[ef]", GROUP:="tserver"

In order to apply these settings, trigger a reload with :manpage:`udevadm(8)`:::

   udevadm trigger --subsystem-match=block


FreeBSD Example
---------------

Starting with 5.1 FreeBSD dropped support for explicit raw devices. All
devices on FreeBSD can be accessed raw now.

The following example will use an entire raw disk in the FreeBSD
operating system::

   /dev/ada1
   /dev/ada2

In order to make sure :program:`traffic_server` will have access to this disk
you can use :manpage:`devfs(8)` to persistently set the right permissions. The
following rules are stored in :manpage:`devfs.conf(5)`::

   # Assign /dev/ada1 and /dev/ada2 to the tserver user
   own    ada[12]  tserver:tserver
