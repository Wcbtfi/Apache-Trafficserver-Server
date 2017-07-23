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

=============
volume.config
=============

.. configfile:: volume.config

使用 :file:`volume.config` 文件能够更有效地管理缓存空间, 并通过为特定协议创建不同大小的缓存卷来限制磁盘使用率. 您可以进一步配置这些卷以存储 :file:`hosting.config` 文件中某些原始服务器和/或域的数据.

.. important::

    The volume configuration must be the same on all nodes in
    a cluster. You must stop Traffic Server before you change the cache
    volume size and protocol assignment. For step-by-step instructions about
    partitioning the cache, refer to :ref:`partitioning-the-cache`.

格式
======

对于每个要创建的卷, 请输出以下格式的行: ::

    volume=volume_number  scheme=protocol_type  size=volume_size

其中 ``volume_number`` 是一个 1 到 255 之间的数字 (最大卷数是255) , ``protocol_type`` 是 ``http``. Traffic
Server 对于 HTTP 卷类型支持 ``http`` ; ``volume_size`` is the
amount of cache space allocated to the volume. This value can be either
a percentage of the total cache space or an absolute value. The absolute
value must be a multiple of 128 MB, where 128 MB is the smallest value.
If you specify a percentage, then the size is rounded down to the
closest multiple of 128 MB.

Each volume is striped across several disks to achieve parallel I/O. For
example: if there are four disks, then a 1-GB volume will have 256 MB on
each disk (assuming each disk has enough free space available). If you
do not allocate all the disk space in the cache, then the extra disk
space is not used. You can use the extra space later to create new
volumes without deleting and clearing the existing volumes.

例子
========

以下例子在 HTTP 和 HTTPS 请求之间均匀分配缓存大小::

    volume=1 scheme=http size=50%
    volume=2 scheme=https size=50%

