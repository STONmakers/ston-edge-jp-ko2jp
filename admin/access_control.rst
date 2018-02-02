.. _access-control:

第15章 アクセス制御
******************

この章では、不要なクライアントへのアクセスをブロックする方法について説明する。 アクセスブロックは、通常、ACL（Access Control List）にブロックリスト（Black-list）を作成しますが設定便宜上許可リスト（White-list）を作成することもある。

アクセス制御は、アクセスの段階で実行するサーバーへのアクセス制御と仮想ホストごとに設定する仮想ホストのアクセス制御に分けられる。 レベルごとに視点と判断基準が異なりますので、効果的なブロック時点を決定しなければならない。 アクセス制御の動作は、すべてログに記録される。

.. toctree::
   :maxdepth: 2


.. _access-control-serviceaccess:

サーバーアクセス制御
====================================

クライアントがサーバーに接続した瞬間IP情報を使用してブロックするかどうかを決定する。 接続の段階で処理されるため、最も確実で速い。 グローバル設定（server.xml）に設定し、最も高い優先順位を持つ。 ::

   # server.xml - <Server><Host>

   <ServiceAccess Default="Allow">
      <Deny>192.168.7.9-255</Deny>
      <Deny>192.168.8.10/255.255.255.0</Deny>
   </ServiceAccess>

-  ``<ServiceAccess>``
   IPベースのACLを設定する。 IP、IP Range、Bitmask、Subnet以上四つの形式をサポートします。 順序を認識し、親に設定された表現が優先する。
   ``Default (基本: Allow)`` 属性は、一致する条件がない場合の処理方法である。 この属性を ``Deny`` に設定すると、下位に ``<Allow>`` で許可する条件を指定ヘジュオヤする。

ブロックされたIPは :ref:`admin-log-deny` に記録される。


.. _access-control-geoip:

GeoIP
====================================

GeoIPを使用して国別のアクセスを遮断することができる。
`GeoIP Databases <http://dev.maxmind.com/geoip/legacy/downloadable/>`_ 中Binary Databasesを `GEOIP_MEMORY_CACHE and GEOIP_CHECK_CACHE <http://dev.maxmind.com/geoip/legacy/benchmarks/>`_ へのリンクしてリアルタイムで変更を反映する。 ::

   # server.xml - <Server><Host>

   <ServiceAccess GeoIP="/var/ston/geoip/">
      <Deny>AP</Deny>
      <Deny>GIN</Deny>
   </ServiceAccess>

``<ServiceAccess>`` の ``GeoIP`` 属性にGeoIP Databasesパスを設定する。 国コードは、 `ISO 3166-1 alpha-2 <http://en.wikipedia.org/wiki/ISO_3166-1_alpha-2>`_ と
`ISO 3166-1 alpha-3 <http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3>`_ をサポートする。

.. note::

   GeoIPはファイル名が予約されているので、必ず保存されたローカルパスを入力するように設定する。 また、自動的に変更が反映されるため、別途設定をReloadしていてもよい。


GeoIPが設定されている場合は、そのディレクトリに保存されたファイルの一覧を表示する。 設定されていない場合、404 NOT FOUNDに応答する。 ::

   http://127.0.0.1:10040/monitoring/geoiplist

結果は、JSON形式で提供される。 ::

   {
       "version": "2.0.0",
       "method": "geoiplist",
       "status": "OK",
       "result":
       {
           "path" : "/usr/ston/geoip/",
           "files" :
           [
               {
                   "file" : "GeoIP.dat",
                   "size" : 766255
               },
               {
                   "file" : "GeoLiteCity.dat",
                   "size" : 12826936
               }
           ]
       }
   }


.. _access-control-vhost:

仮想ホストのアクセス制御
====================================

仮想ホストごとにサービスを許可/ブロック/ redirectを設定する。 ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <AccessControl Default="Allow" DenialCode="401">OFF</AccessControl>

-  ``<AccessControl>``

   - ``OFF (基本)`` ACLが有効になっていない。 すべてのクライアントの要求を許可する。

   - ``ON`` ACLが有効になる。 ブロックされた要求には、 ``DenialCode`` 属性に設定された応答コードで応答する。
     ``Default (基本: Allow)`` 属性が ``Allow`` であれば、ACLは拒否リストになる。 逆に ``Deny`` ならACLは許可リストになる。

具体的なアクセス制御リストは、/ svc / {仮想ホスト名} /acl.txtに設定する。


.. _access-control-vhost_allow_deny:

許可/拒否
---------------------

すべてのクライアントのHTTP要求に対して許可/拒否するかどうかを設定する。 Denyされたリクエストは、 :ref:`admin-log-access` にTCP_DENYに記録される。

各条件ごとに個別に応答コードを設定することもできる。 ::

   # /svc/www.example.com/acl.txt
   # 区切り文字はカンマ（、）であり、{条件}、{allowまたはdeny}順に表記する。
   # denyの場合、キーワードの後に応答コードを指定することができる。
   # 指定しなければ ``<AccessControl>`` の ``DenialCode`` を使用する。
   # n個の条件を組み合わせて（AND）するためには及びを使用する。

   $IP[192.168.1.1], allow
   $IP[192.168.2.1-255]
   $IP[192.168.3.0/24], deny
   $IP[192.168.4.0/255.255.255.0]
   $IP[AP] & !HEADER[referer], allow
   $HEADER[cookie: *ILLEGAL*], deny, 404
   $HEADER[via: Apache]
   $HEADER[x-custom-header]
   !HEADER[referer] & !HEADER[user-agent] & !HEADER[host], deny
   $URL[/source/public.zip], allow
   $URL[/source/*]
   /profile.zip, deny, 500
   /secure/*.dat

許可/遮断条件は、IP、GeoIP、Header、URL、4つに設定が可能である。

-  **IP**

   $IP[...]で表記し IP, IP Range, Bitmask, Subnet 4種類をサポートします。

-  **GEOIP**

   $IP[...]で表記し、必ずGeoIP設定がされている動作する。

-  **HEADER**

   $HEADER[Key : Value]と表記する。 Valueは明確な表現とパターンを認識する。 $ HEADER [Key：]のように区切り文字が、Valueが空の文字列であれば、リクエストヘッダの値が空の場合を意味する。 $ HEADER [Key]のように区切り文字なしにKeyのみ明示されている場合はKeyに対応するヘッダの存在の有無を条件に判断する。

-  **URL**

   $URL[...]で表記し省略が可能である。 明確な表現とパターンを認識する。

$は "条件に合うなら〜する"を意味するが、！は "条件に合わない場合は〜する"を意味する。
次のように否定条件で支援する。 ::

   # 国がJPNがない場合はdenyする。
   !IP[JPN], deny

   # refererヘッダが存在しない場合denyする。
   !HEADER[referer], deny

   # /secure/ パスサブがない場合はallowする。
   !URL[/secure/*], allow



.. _access-control-vhost_redirect:

Redirect
---------------------

すべてのクライアントのHTTP要求に対してRedirectかどうかを設定する。 Redirectされた要求には、 **302 Moved temporarily** で応答する。 ::

   # /svc/www.example.com/acl.txt
   # 区切り文字はカンマ（、）であり、{条件}、{redirect}順に表記する。
   # redirectの場合、キーワードの後に​​移動させるURLを指定する。 (Locationヘッダの値に明示)

   $IP[GIN], redirect, /page/illegal_access.html
   $HEADER[referer:], redirect, http://another-site.com
   !HEADER[referer], redirect, http://example.com#URL

Redirectするときに、クライアントが要求されたURLが必要になることができる。 このような場合 ``#URL`` キーワードを使用する。

HTTPSのみをサポートするサービスの場合、HTTPリクエストには、次のように ``$PROTOCOL[HTTP]`` の条件でHTTPSを強制するようにredirectさせることができる。 ::

   $PROTOCOL[HTTP], redirect, https://example.com#URL

