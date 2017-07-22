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

.. configfile:: cache.config

============
cache.config
============

:file:`cache.config` 文件 ( 默认位于 ``/usr/local/etc/trafficserver/`` ) 定义 Traffic Server 如何缓存 web 对象. 你可以添加缓存规则去指定以下内容: 

    - Not to cache objects from specific IP addresses 
    - How long to pin particular objects in the cache
    - How long to consider cached objects as fresh 
    - Whether to ignore no-cache directives from the server
    
.. important::

   After you modify the :file:`cache.config` file, navigate to
   the Traffic Server bin directory; then run the :option:`traffic_ctl config reload`
   command to apply changes. When you apply the changes to a node in a
   cluster, Traffic Server automatically applies the changes to all other
   nodes in the cluster.

Format
======

Each line in the :file:`cache.config` file contains a caching rule. Traffic
Server recognizes three space-delimited tags::

   primary_destination=value secondary_specifier=value action=value

You can use more than one secondary specifier in a rule. However, you
cannot repeat a secondary specifier. The following list shows the
possible primary destinations with allowed values.

.. _cache-config-format-dest-domain:

``dest_domain``
   请求的域名. Traffic Server 从请求的 URL 中匹配目标的域名.

.. _cache-config-format-dest-host:

``dest_host``
   请求的主机名. Traffic Server 从请求的 URL 中匹配目标的主机名.

.. _cache-config-format-dest-ip:

``dest_ip``
   请求的 IP 地址. Traffic Server 匹配请求的目标的 IP 地址.

.. _cache-config-format-url-regex:

``url_regex``
   A regular expression (regex) to be found in a URL.

The secondary specifiers are optional in the :file:`cache.config` file. The
following list shows possible secondary specifiers with allowed values.

.. _cache-config-format-port:

``port``
   请求的 URL 端口.

.. _cache-config-format-scheme:

``scheme``
   请求的 URL 协议: http 或 https.

.. _cache-config-format-prefix:

``prefix``
   在 URL 里部分路径的前缀.

.. _cache-config-format-suffix:

``suffix``
   在 URL 里的文件后缀.

.. _cache-config-format-method:

``method``
   请求的 URL 方法: get, put, post, trace.

.. _cache-config-format-time:

``time``
   时间范围, 如 08:00-14:00.

.. _cache-config-format-src-ip:

``src_ip``
   客户端 IP 地址.

.. _cache-config-format-internal:

``internal``
    A boolean value, ``true`` or ``false``, specifying if the rule should
    match (or not match) a transaction originating from an internal API. This
    is useful to differentiate transaction originating from an ATS plugin.

The following list shows possible actions and their allowed values.


.. _cache-config-format-action:

``action``
   One of the following values:

   -  ``never-cache`` 配置 Traffic Server 从不缓存指定的对象.
   -  ``ignore-no-cache`` 配置 Traffic Server 忽略所有 ``Cache-Control: no-cache`` 标头.
   -  ``ignore-client-no-cache`` 配置 Traffic Server 忽略来自客户端请求的 `Cache-Control: no-cache`` 标头.
   -  ``ignore-server-no-cache`` 配置 Traffic Server 忽略来自源服务器响应的 ``Cache-Control: no-cache`` 标头.
   -  ``cluster-cache-local`` 配置集群缓存, 以允许在每个集群节点上存储这个内容.

.. _cache-responses-to-cookies:

``cache-responses-to-cookies``
   Change the style of caching with regard to cookies. This effectively
   overrides the configuration parameter
   :ts:cv:`proxy.config.http.cache.cache_responses_to_cookies`
   and uses the same values with the same semantics. The override happens
   only for requests that match.
    

.. _cache-config-format-pin-in-cache:

``pin-in-cache``
   保留缓存中的对象, 防止它们被逐出. 
   不会影响那些已经确定不可缓存的对象. 
   这个设置可能会产生性能问题, 并严重影响缓存.  
   在实例中, 如果主要目标匹配所有对象, 一旦缓存存满, 因为没有对象被逐出, 因此任何新的对象都不会写入. 
   同样, 对于每个 cache-miss , 每个对象都会进行额外的检查, 以确保它所替换的对象是否可以被覆盖. 

   这个值是设置对象在缓存中保留的时间. 允许使用下列时间格式:

   -  ``d`` 天; 例如: 2d
   -  ``h`` 小时; 例如: 10h
   -  ``m`` 分钟; 例如: 5m
   -  ``s`` 秒; 例如: 20s
   -  混合单位; 例如: 1h15m20s

.. _cache-config-format-revalidate:

``revalidate``
   For objects that are in cache, overrides the the amount of time the object(s) 
   are to be considered fresh. 使用与 ``pin-in-cache`` 相同的时间格式.

.. _cache-config-format-ttl-in-cache:

``ttl-in-cache``
   Forces object(s) to become cached, as if they had a Cache-Control: max-age:<time>
   header. Can be overruled by requests with cookies. The value is the amount of 
   time object(s) are to be kept in the cache, regardless of Cache-Control response 
   headers. Use the same time formats as pin-in-cache and revalidate.

例子
========

下面 Traffic Server 的示例 每隔6个小时重新验证 ``mydomain.com`` 域的 ``gif`` 和 ``jpeg`` 对象, 以及每隔1小时重新验证所有其他对象. 规则按列出的顺序应用. ::

   dest_domain=mydomain.com suffix=gif revalidate=6h
   dest_domain=mydomain.com suffix=jpeg revalidate=6h
   dest_domain=mydomain.com revalidate=1h

强制一个指定的正则, 应用于服务器时间的 7-11pm 之间, 缓存26个小时. ::

   url_regex=example.com/articles/popular.* time=19:00-23:00 ttl-in-cache=1d2h

防止对象在缓存中被逐出: 

   url_regex=example.com/game/.* pin-in-cache=1h

