.. _access-control:

第15章 アクセス制御
******************

この章では不要なクライアントアクセスをブロックする方法について説明します。 アクセスブロックは通常ACL（Access Control List）にブロックリスト（Black-list）を作成しますが許可リスト（White-list）を作成することもあります。

アクセス制御はアクセスの段階で実行するサーバーへのアクセス制御と仮想ホストごとに設定する仮想ホストのアクセス制御に分けられる。 レベルごとに視点と判断基準が異なりますので効果的なブロック時点を決定する必要があります。 アクセス制御の動作はすべてログに記録されます。

.. toctree::
   :maxdepth: 2


.. _access-control-serviceaccess:

サーバーアクセス制御
====================================

クライアントがサーバーに接続した瞬間IP情報を使用してブロック有無を決定します。 接続の段階で処理されるため最も確実で速い。 グローバル設定（server.xml）に設定し最も高い優先順位を持つ。 ::

   # server.xml - <Server><Host>

   <ServiceAccess Default="Allow">
      <Deny>192.168.7.9-255</Deny>
      <Deny>192.168.8.10/255.255.255.0</Deny>
   </ServiceAccess>

-  ``<ServiceAccess>``
   IPベースのACLを設定します。 IP、IP Range、Bitmask、Subnetの四つの形式をサポートします。 順序を認識し上に設定された表現が優先します。
   ``Default (デフォルト: Allow)`` 属性は一致する条件がない場合の処理方法です。 この属性を ``Deny`` に設定すると下位に ``<Allow>`` で許可する条件を明記する必要があります。

ブロックされたIPは :ref:`admin-log-deny` に記録されます。


.. _access-control-geoip:

GeoIP
====================================

GeoIPを使用して国別のアクセスを遮断することができます。
`GeoIP Databases <http://dev.maxmind.com/geoip/legacy/downloadable/>`_ 中Binary Databasesを `GEOIP_MEMORY_CACHE and GEOIP_CHECK_CACHE <http://dev.maxmind.com/geoip/legacy/benchmarks/>`_ へのリンクしてリアルタイムで変更を反映します。 ::

   # server.xml - <Server><Host>

   <ServiceAccess GeoIP="/var/ston/geoip/">
      <Deny>AP</Deny>
      <Deny>GIN</Deny>
   </ServiceAccess>

``<ServiceAccess>`` の ``GeoIP`` 属性にGeoIP Databasesパスを設定します。 国コードは `ISO 3166-1 alpha-2 <http://en.wikipedia.org/wiki/ISO_3166-1_alpha-2>`_ と
`ISO 3166-1 alpha-3 <http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3>`_ をサポートします。

.. note::

   GeoIPはファイル名が予約されているので必ず保存されたローカルパスを設定します。 変更は自動的に反映されるます。


GeoIPが設定されている場合はそのディレクトリに保存されたファイルの一覧を表示します。 設定されていない場合404 NOT FOUNDに応答します。 ::

   http://127.0.0.1:10040/monitoring/geoiplist

結果はJSON形式で提供されます。 ::

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

仮想ホストへのアクセス制御
====================================

仮想ホストごとにサービスを許可/ブロック/ redirectを設定します。 ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <AccessControl Default="Allow" DenialCode="401">OFF</AccessControl>

-  ``<AccessControl>``

   - ``OFF (デフォルト)`` ACLが有効になっていない。 すべてのクライアントの要求を許可します。

   - ``ON`` ACLが有効になる。 ブロックされた要求には ``DenialCode`` 属性に設定された応答コードで応答します。
     ``Default (デフォルト: Allow)`` 属性が ``Allow`` であればACLは拒否リストになる。 反対に ``Deny`` ならACLは許可リストになる。

アクセス制御リストは/ svc / {仮想ホスト名} /acl.txtに設定します。


.. _access-control-vhost_allow_deny:

許可/拒否
---------------------

すべてのクライアントのHTTP要求に対して許可/拒否有無を設定します。 Denyされたリクエストは :ref:`admin-log-access` にTCP_DENYに記録されます。

各条件ごとに個別に応答コードを設定することもできます。 ::

   # /svc/www.example.com/acl.txt
   # 区切り文字はカンマ（、）であり{条件}、{allowまたはdeny}順に表記します。
   # denyの場合キーワードの後に応答コードを指定することができます。
   # 指定しなければ ``<AccessControl>`` の ``DenialCode`` を使用します。
   # n個の条件を組み合わせて（AND）するためには&を使用します。

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

許可/遮断条件はIP、GeoIP、Header、URLの4つの設定が可能です。

-  **IP**

   $IP[...]で表記し IP, IP Range, Bitmask, Subnet 4種類をサポートします。

-  **GEOIP**

   $IP[...]で表記し必ずGeoIP設定が必須です。

-  **HEADER**

   $HEADER[Key : Value]と表記します。 
   Valueは明確な表現とパターンを認識します。 
   $HEADER [Key：]のように区切り文字はありますがValueが空の文字列であればリクエストヘッダの値が空の場合を意味します。 $HEADER [Key]のように区切り文字なしにKeyのみ指定されている場合はKeyに対応するヘッダの存在の有無を条件で判断します。

-  **URL**

   $URL[...]で表記し省略が可能です。 明確な表現とパターンを認識します。

$は "条件に合うなら〜する"を意味するが！は "条件に合わない場合は〜する"を意味します。 次のように否定条件で支援します。 ::

   # 国がJPNではない場合はdenyします。
   !IP[JPN], deny

   # refererヘッダが存在しない場合denyします。
   !HEADER[referer], deny

   # /secure/ 経路の以下ではない場合はallowします。
   !URL[/secure/*], allow



.. _access-control-vhost_redirect:

Redirect
---------------------

すべてのクライアントのHTTP要求に対してRedirect有無を設定します。 Redirectされた要求には **302 Moved temporarily** で応答します。 ::

   # /svc/www.example.com/acl.txt
   # 区切り文字はカンマ（、）であり{条件}、{redirect}順に表記します。
   # redirectの場合キーワードの後に移動させるURLを指定します。 (Locationヘッダの値に明示)

   $IP[GIN], redirect, /page/illegal_access.html
   $HEADER[referer:], redirect, http://another-site.com
   !HEADER[referer], redirect, http://example.com#URL

Redirectする場合にクライアントが要求されたURLが必要になることができます。 このような場合 ``#URL`` キーワードを使用します。

HTTPSのみをサポートするサービスの場合HTTPリクエストには次のように ``$PROTOCOL[HTTP]`` の条件でHTTPSを強制するようにredirectさせることができます。 ::

   $PROTOCOL[HTTP], redirect, https://example.com#URL

