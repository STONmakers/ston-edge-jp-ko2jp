.. _origin:

第7章 ソースサーバー
******************

この章では、STONと元のサーバーの関係について説明する。 ソースサーバーとは、一般的にHTTPの仕様に準拠しているWebサーバーを意味する。 管理者であれば、原稿を保護するために、今回の章のすべての内容を熟知する必要がある。 これを基に、元の障害にも耐久性を備えた柔軟なサービスを構築することができる。

ソースサーバーは、保護されるべきである。 障害の種類が多様なだけに対処案も多様である。 ソース保護ポリシーを適切に設定すると、ゆったりとした点検時間を持つことができます。


.. toctree::
   :maxdepth: 2



.. _origin_exclusion_and_recovery:

障害の検出と回復
====================================

Caching過程の中で、元のサーバーに障害が発生した場合、自動的に排除する。 再び安定化されたと判断すると、サービスに投入する。 ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <ConnectTimeout>3</ConnectTimeout>
   <ReceiveTimeout>10</ReceiveTimeout>
   <Exclusion>3</Exclusion>
   <Recovery Cycle="10" Uri="/" ResCode="0" Log="ON">5</Recovery>

-  ``<ConnectTimeout> (基本: 3秒)``

   n秒以内に元のサーバーとの接続が行われていない場合、接続に失敗とみなす。

-  ``<ReceiveTimeout> (基本: 10秒)``

   通常のHTTPリクエストにもかかわらず、元のサーバーがHTTP応答をn秒間送信しない場合は、送信が失敗であると考えている。

-  ``<Exclusion> (基本: 3回)``

   元のサーバーで連続的にn回障害状況( ``<ConnectTimeout>`` または ``<ReceiveTimeout>`` )が発生した場合、そのサーバーを有効ソースサーバーのリストから排除する。 排除前、通常の通信が行われた場合は、この値は0に初期化される。

-  ``<Recovery> (基本: 5回)``

   ``Cycle`` ごと ``Uri`` に要求して、元のサーバーが ``ResCode`` に連続的にn回応答すると、そのサーバーを回復する。 この値を0に設定すると、回復していない。

   -  ``Cycle (基本: 10秒)`` 一定時間（秒）ごとにしようとする。

   -  ``Uri (基本: /)`` 要求を送信Uri

   -  ``ResCode (基本: 0)`` 正常応答として処理する応答コード。 0の場合は、応答コードに関係なく、応答が来れば成功とみなす。 200に設定すると、応答コードが必ず200でなければなら正常応答で処理する。 コンマ（、）を使用して、有効な応答コードをマルチに設定する。 200、206、404に設定すると、応答コードがこれらのいずれかである場合、通常の応答で処理する。

   -  ``Log (基本: ON)`` 回復のために使用されたHTTP Transactionを :ref:`admin-log-origin` に記録する。



.. _origin-health-checker:

Health-Checker
====================================

`장애감지와 복구`_ は、Caching過程中に発生する障害に対応する。
``<Recovery>`` は応答コードを受け取り次第HTTP Transactionを終了する。 しかし、Health-Checkerは、HTTP Transactionが成功することを確認する。 ::

   # vhosts.xml - <Vhosts><Vhost>

   <Origin>
      <Address> ... </Address>
      <HealthChecker ResCode="0" Timeout="10" Cycle="10"
                     Exclusion="3" Recovery="5" Log="ON">/</HealthChecker>
      <HealthChecker ResCode="200, 404" Timeout="3" Cycle="5"
                     Exclusion="5" Recovery="20" Log="ON">/alive.html</HealthChecker>
   </Origin>

-  ``<HealthChecker> (基本: /)``

   Health-Checkerを構成する。 マルチで構成が可能である。 値としてUriを指定し、XML例外文字の場合CDATAを使用する。

   -  ``ResCode (基本: 0)`` 正しい応答コード（コンマでマルチ構成可能）

   -  ``Timeout (基本: 10秒)`` ソケット接続からHTTP Transactionが完了するまで有効時間

   -  ``Cycle (基本: 10秒)`` 実行サイクル

   -  ``Exclusion (基本: 3回)`` 連続n回失敗した場合、そのサーバー排除

   -  ``Recovery (基本: 5回)`` 連続n回成功した場合、そのサーバーの再投入

   -  ``Log (基本: ON)`` HTTP Transactionを :ref:`admin-log-origin` に記録する。

Health-Checkerは、マルチで構成することができ、クライアントの要求に関係なく独立して実行される。
`障害の検出と回復`_ や他のHealth-Checkerとも情報を共有せずに、自分だけの基準で排除および責任を決定する。


.. _origin-use-policy:

送信元アドレスを使用ポリシー
====================================

送信元アドレス（IP）は、次の要素によってどのように使用されるか決定される。

-  :ref:`env-vhost-activeorigin` アドレスの形式（IPアドレスまたはDomain）とセカンダリアドレス
-  `障害の検出と回復`_
-  `Health-Checker`_

サービスを運営していると、元のアドレスが排除/回復されることは頻繁である。 STONは、IPテーブルに基づいて、元のアドレスを使用して `origin-status`_ APIを介して情報を提供する。

送信元アドレスをIPに設定した場合、非常に簡単である。

-  設定の変更に加えて、IPリストを変化させる要因はない。
-  TTLにより、IPアドレスが期限切れにならない。
-  障害/復旧の両方の設定（IPアドレス）に基づいて動作する。

送信元アドレスをDomainに設定すると、ResolvingてIPアドレスを得なければならない。
( :ref:`admin-log-dns` に記録される。)
IPリストは動的に変更されることができ、すべてのIPは、TTL（Time To Live）の間だけ有効である。

-  Domainは定期的に（1〜10秒）Resolvingする。
-  Resolvingを介して使用するIPテーブルを構成する。
-  すべてのIPは、TTL分だけ有効であり、TTLが期限切れになると使用しない。
-  同じIPアドレスが再びResolvingされると、TTLを更新する。
-  IPテーブルは空白ではない。 （TTLが期限切れにされても）最後のIPは削除されない。

IPのTTLが長すぎる場合、過度に多くのIPアドレスを使用するようになって意図しない結果を作成することができる。 これを防止するために、IPの最大TTLを制限することができる。 ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <DnsMaxTTL>60</DnsMaxTTL>

-  ``<DnsMaxTTL> (基本: 60秒)`` ResolvingされたIPアドレスの最大使用時間（秒）を設定する。 この値が0の場合は、DNSから提供されたTTLをそのまま使用する。


.. note::

   送信元アドレスをDomainに設定しても、障害/復旧は、IPベースで動作する。 Domainアドレス障害/復旧ポリシーは、次のとおりである。

   -  （Domainについて）知っているすべてのIPアドレスが排除（Inactive）と、そのDomainアドレスが排除される。
   -  新規IPがResolvingもDomainが排除されている場合は、IPアドレスは、最初から排除される。
   -  すべてのIPアドレスはTTL有効期限が切れても排除されたDomain状態は解けない。
   -  排除されたDomainに属するIPアドレスのいずれかが回復されるべきは、Domainは再び有効にされる。

   やや複雑な内容であるため、 `origin-status`_ APIを使用してサービスの動作状態について理解を高めるのが良い。



.. _origin-status:

元の状態の監視
====================================

APIを介して、仮想ホストのソースの状態を監視します。 ::

   http://127.0.0.1:10040/monitoring/origin       // すべての仮想ホスト
   http://127.0.0.1:10040/monitoring/origin?vhost=www.example.com

結果は、JSON形式で提供される。 ::

   {
       "origin" :
       [
           {
               "VirtualHost" : "example.com",
               "Address" :
               [
                   { "1.1.1.1" : "Active" },
                   { "1.1.1.2" : "Active" }
               ],
               "Address2" : [  ],
               "ActiveIP" :
               [
                   { "1.1.1.1" : 0 },
                   { "1.1.1.2" : 0 }
               ] ,
               "InactiveIP" : [ ]
           },
           {
               "VirtualHost" : "foobar.com",
               "Address" :
               [
                   { "origin.foobar.com" : "Active" }
               ],
               "Address2" : [  ],
               "ActiveIP" :
               [
                   { "5.5.5.5" : 21 },
                   { "5.5.5.6" : 60 },
                   { "5.5.5.7" : 37 }
               ],
               "InactiveIP" :
               [
                   { "5.5.5.8" : 10 },
                   { "5.5.5.9" : 184 }
               ]
           }
       ]
   }

-  ``VirtualHost`` 仮想ホスト名

-  ``Address`` :ref:`env-vhost-activeorigin` 。
   設定アドレスが使用中であれば ``Active`` , （障害が発生して）使用していない場合 ``Inactive`` として表示される。

-  ``Address2`` :ref:`env-vhost-standbyorigin` 。
   設定アドレスを使用中であれば ``Active`` 、使用していない場合 ``Inactive`` として表示される。

-  ``ActiveIP`` 使用中のIPリストとTTL。 ソースサーバーをIPに設定すると、 ``Address`` と同じIPにTTLは、0と表示される。 Domainに設定すると、Resolving結果に従う。 様々なIPとTTLを使用する。

-  ``InactiveIP`` 使用しないIPリストとTTL。 使用していなくても、回復しているかHealthCheckerによって管理することができる。 そのアドレスは、TTLの間回復されなければ削除される。



.. _origin-status-reset:

元の状態の初期化
====================================

APIを介して、仮想ホストのソースサーバー排除/回復を初期化する。 また、現在使用中のセッションを再利用せずに、新たに接続を作成します。 ::

   http://127.0.0.1:10040/command/resetorigin       // すべての仮想ホスト
   http://127.0.0.1:10040/command/resetorigin?vhost=www.example.com



.. _origin-busysessioncount:

過負荷判断
====================================

最初に要求されているコンテンツは、常に元のサーバーに要求しなければならない。 しかし、すでにCachingされたコンテンツであれば、より柔軟に対応することができる。 ソースサーバーが過負荷状態と判断されると、更新をずらして、元の負荷を高くない。 ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <BusySessionCount>100</BusySessionCount>

-  ``<BusySessionCount> (基本: 100個)``
   ソースサーバーとHTTPトランザクションを進行中のセッションの数が一定数を超えると、過負荷状態と判断する。 過負荷状態で有効期限が切れたコンテンツを更新するために、元のサーバーに接続しないように、TTLを :ref:`caching-policy-ttl` の ``<OriginBusy>`` だけ延長する。 無条件元サーバーに要求が行くようにするには、この値を非常に大きく設定すればよい。


.. _origin-balancemode:

ソースの選択
====================================

元サーバーのアドレスがマルチ（2個以上）で構成されているときに、元のサーバーの選択ポリシーを設定する。 ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <BalanceMode>RoundRobin</BalanceMode>

-  ``<BalanceMode> (基本: RoundRobin)``

   -  ``RoundRobin (基本)``
      すべてのソースサーバーが均等に要求を受信するようにRoundRobinで動作する。 接続されたIdleセッションは、そのサーバーに要求が必要な場合にのみ使用する。

   -  ``Session``
      再使用可能なセッションがある場合、まず使用する。 新規セッションが必要な場合Round-Robinに割り当てる。

   -  ``Hash``
      コンテンツを `Consistent Hashing <http://en.wikipedia.org/wiki/Consistent_hashing>`_ アルゴリズムに基づいて、元のサーバーに分散して要請する。 サーバーが選択されると、既に接続されたセッションを再利用しない場合は、新規に接続する。


=========== =================================================================== =====================================================
/           RoundRobin                                                          Session
=========== =================================================================== =====================================================
負荷（要求）	すべてのサーバーが負荷を均等に分配	                                反応性と再利用性が良いサーバーに負荷が加重される
接続コスト	高（そのサーバーの順序がされると、接続されたセッションを探していない場合、接続しようと）   低（再使用可能なセッションがない場合のみ接続）
再利用性	低（サーバー分配優先）	                                            高（常に接続されたセッションを優先使用）
セッション数	    沢山（各サーバーごとに同時に進行されるHTTPトランザクションの合計）          少ない（同時に進行されるHTTPトランザクションだけセッションが存在）
=========== =================================================================== =====================================================


セッションの再利用
====================================

ソースサーバーがKeep-Aliveをサポートする場合接続されたセッションは、常に再使用される。 しかし、セッションを再利用して送信要求に対して、元のサーバーが一方的に接続を終了することができる。 ので、接続を回復するのに、ユーザーの反応が遅くなる可能性がある。 特に長い間再使用していないセッションの場合、これらの可能性はさらに高い。 これを防止するためにn秒の間再利用されていないセッションに対して自動的に接続を終了するように設定する。 ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <ReuseTimeout>60</ReuseTimeout>

-  ``<ReuseTimeout> (基本: 60秒)``
   一定時間使用されていない元のセッションは終了する。 0に設定すると、元のサーバーのセッションを再利用していない。


.. _origin_partsize:

Rangeリクエスト
====================================

一度ダウンロードされたコンテンツのサイズを設定する。 動画のように前の部分だけが、主に消費されるコンテンツの場合、ダウンロードサイズを制限すると、不要なソーストラフィックを減らすことができる。 ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <PartSize>0</PartSize>

-  ``<PartSize> (基本: 0 MB)``
   0よりも大きい場合、クライアントが要求した時点から設定サイズ（MB）だけRangeリクエストにダウンロードする。


``<PartSize>`` を使用しているもう一つの理由は、ディスクスペースを節約するためである。 デフォルトの設定でSTONは元のサイズのファイルをディスク上に作成する。 しかし、 ``<PartSize>`` が0でない場合はダウンロードされている分だけファイルを分割して保存する。

たとえば、1時間の映像（600MB）を1分（10MB）のみ視聴した場合に、ディスク領域を10MBだけを使用する。 スペースを節約する利点はあるが、ファイルが分割されて保存されるため、ディスク負荷が少し高くなる。

.. note::

   最初のコンテンツをダウンロードする際にContent-Lengthを知ることができないので、Range要求をすることができない。 ため ``<PartSize>`` が設定されている場合は設定サイズ分だけダウンロードして接続を終了する。




全体Range初期化
====================================

一般的に、元のサーバーから、最初のファイルをダウンロードするときや更新確認するときは、次のように単純な形のGETリクエストを送信します。 ::

    GET /file.dat HTTP/1.1

しかし、元のサーバーが一般的なGETリクエストに対して常にファイルを改ざんするように設定されている場合は、元のファイルをそのままCachingできなく問題になることがあります。

最も代表的な例は、Apache Webサーバがmod_h.264_streamingなどの外部モジュールのように駆動される場合である。 Apache Webサーバーは、GETリクエストに対して常にmod_h.264_streamingモジュールを介して応答する。 クライアント（この場合には、STON）は、元のファイルのままではなく、モジュールによって変調されたファイルをサービス受ける。

   .. figure:: img/conf_origin_fullrangeinit1.png
      :align: center

      mod_h.264_streamingモジュールは常にソースを変調する。

Range要求を使用すると、モジュールをバイパスして、元のをダウンロードすることができる。 ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <FullRangeInit>OFF</FullRangeInit>

-  ``<FullRangeInit>``

   - ``OFF (基本)`` の一般的なHTTPリクエストを送る。

   - ``ON`` 0から始まるRangeリクエストを送る。 Apacheの場合Rangeヘッダが明示されると、モジュールをバイパスする。 ::

        GET /file.dat HTTP/1.1
        Range: bytes=0-

     最初にファイルCachingするときは、コンテンツのRangeを知らないので、Full-Range（= 0から始まる）を要請する。 ソースサーバーがRange要求に対して正常に応答（206 OK）かどうかを必ず確認しなければならない。

コンテンツを更新するときは、次のように **If-Modified-Since** ヘッダが一緒に指定される。 ソースサーバーが正しく **304 Not Modified** に応答しなければならない。 ::

   GET /file.dat HTTP/1.1
   Range: bytes=0-
   If-Modified-Since: Sat, 29 Oct 1994 19:43:31 GMT

.. note::

   ``<FullRangeInit>`` が正常に動作することを確認し、Webサーバーのリスト。

   - Microsoft-IIS/7.5
   - nginx/1.4.2
   - lighttpd/1.4.32
   - Apache/2.2.22


.. _origin-wholeclientrequest:

クライアントの要求を維持
====================================

元に要求するとき、クライアントが送信した要求を維持するように設定する。 ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <WholeClientRequest>OFF</WholeClientRequest>

-  ``<WholeClientRequest>``

   - ``OFF (基本)`` Caching-Keyを元に要求するURLに使用する。

   - ``ON`` クライアントが要求されたURLに元に要請する。

Hit Ratioを高めるために、次の設定を使用してCaching-Keyを決定する。

- :ref:`caching-policy-casesensitive`
- :ref:`caching-policy-applyquerystring`
- :ref:`caching-policy-post-method-caching`

これにより、元のサーバーに要求したURLとCaching-Keyは、次のように決定される。

============================================== ======================= ============================
設定                                           クライアント要求URL       元の要求URL / Caching-Key
============================================== ======================= ============================
:ref:`caching-policy-casesensitive` ``OFF``    /Image/LOGO.png         /image/logo.png
:ref:`caching-policy-casesensitive` ``ON``     /Image/LOGO.png         /Image/LOGO.png
:ref:`caching-policy-applyquerystring` ``OFF`` /view/list.php?type=A   /view/list.php
:ref:`caching-policy-applyquerystring` ``ON``  /view/list.php?type=A   /view/list.php?type=A
============================================== ======================= ============================

``<WholeClientRequest>`` を ``ON`` に設定すると、次のようにCaching-Keyとは関係なく、クライアントが送信したURLをそのまま元に送る。

============================================== =================================== ============================
設定                                            クライアント/元の要求URL           Caching-Key
============================================== =================================== ============================
:ref:`caching-policy-casesensitive` ``OFF``    /Image/LOGO.png                     /image/logo.png
:ref:`caching-policy-casesensitive` ``ON``     /Image/LOGO.png                     /Image/LOGO.png
:ref:`caching-policy-applyquerystring` ``OFF`` /view/list.php?type=A               /view/list.php
:ref:`caching-policy-applyquerystring` ``ON``  /view/list.php?type=A               /view/list.php?type=A
============================================== =================================== ============================

POSTリクエストをキャッシュする場合、元のサーバーに要求したとき、クライアントが送信したPOSTリクエストのBodyデータが変更なしで送信される。

.. note::

   クライアントが送信したURLをそのまま送信するため :ref:`media-trimming` ようアドオンのためにつけられたQueryStringもそのまま元のサーバーに転送される。


.. _origin-httprequest:

元の要求の基本Header
====================================

Hostヘッダ
---------------------

ソースサーバーに送信HTTPリクエストのHostヘッダを設定する。 別に設定していない場合は、仮想ホスト名が明示される。 ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <Host />

-  ``<Host>``
   元のサーバーに送信Hostヘッダを設定する。 元のサーバーで80ポート以外のポートでサービスしている場合、必ずポート番号を明示しなければならない。 ::

      # server.xml - <Server><VHostDefault><OriginOptions>
      # vhosts.xml - <Vhosts><Vhost><OriginOptions>

      <Host>www.example2.com:8080</Host>


クライアントが送信したHostヘッダを元に送りたい場合*に設定する。


User-Agentヘッダ
---------------------

ソースサーバーに送信HTTPリクエストのUser-Agentヘッダを設定する。 ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <UserAgent>STON</UserAgent>

-  ``<UserAgent> (基本: STON)``
   元のサーバーに送信UserAgentヘッダーを設定する。


クライアントが送信したUser-Agentヘッダを元に送りたい場合*に設定する。


XFF(X-Forwarded-For) ヘッダ
---------------------

클クライアントと元のサーバーとの間にSTONが位置すると、元のサーバーは、クライアントのIPアドレスを取得することができない。 ためSTONは、元のサーバーに送信されるすべてのHTTP要求にX-Forwarded-Forヘッダを明示する ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <XFFClientIPOnly>OFF</XFFClientIPOnly>

-  ``<XFFClientIPOnly>``

   - ``OFF (基本)`` クライアント（IP：128.134.9.1）が送信XFFヘッダにクライアントのIPアドレスを追加します。 クライアントがXFFヘッダを送信していない場合、クライアントのIPのみ明示される。 ::

        X-Forwarded-For: 220.61.7.150, 61.1.9.100, 128.134.9.1

   - ``ON`` XFFヘッダの最初のアドレスだけを元のサーバーに転送する。 ::

        X-Forwarded-For: 220.61.7.150


ETagヘッダ認識
---------------------

ソースサーバーからの応答するETag認識するかどうかを設定する。 ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <OriginalETag>OFF</OriginalETag>

-  ``<OriginalETag>``

   - ``OFF (基本)``  ETagヘッダを無視する。

   - ``ON`` ETagを認識し、コンテンツ更新時If-None-Matchヘッダを追加します。



.. _origin_url_rewrite:

元の要求URLの変更
====================================

キャッシュを目的として、元に送信するHTTPリクエストのURLを変更する。 ::

   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <URLRewrite>
      <Pattern>/download/(.*)</Pattern>
      <Replace>/#1</Replace>
   </URLRewrite>
   // Pattern : /download/1.3.4
   // Replace : /1.3.4

   <URLRewrite>
      <Pattern>/img/(.*\.(jpg|png).*)</Pattern>
      <Replace>/#1/STON/composite/watermark1</Replace>
   </URLRewrite>
   // Pattern : /img/image.jpg?date=20140326
   // Replace : /image.jpg?date=20140326/STON/composite/watermark1

:ref:`handling_http_requests_url_rewrite` のような表現を使用しますが、仮想ホストごとに独立して設定するため、仮想ホスト名を入力していない。

.. note::

   バイパスされるHTTP要求のURLは変更できない。
   ``<WholeClientRequest>`` よりも優先します。



.. _origin_modify_client:

元のリクエストヘッダの変更
====================================

元にHTTPリクエストを送信するときの条件に応じて、HTTPヘッダーを変更する。 ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <ModifyHeader FirstOnly="OFF">OFF</ModifyHeader>

-  ``<ModifyHeader>``

   -  ``OFF (基本)`` 変更しない。

   -  ``ON`` ヘッダ変更条件に応じて、ヘッダーを変更する。

ヘッダ変更時点は、HTTP要求パケットが完成されて、元のサーバーに転送する直前に実行される。 ただし、Rangeヘッダは変調することができない。

この機能は、 :ref:`handling_http_requests_modify_client` のサブ機能である。 ヘッダ変更には$ ORGREQキーワードを使用する。 ::

   # /svc/www.example.com/headers.txt

   $URL[/*.mp4], $ORGREQ[x-media-type: video/mp4], set
   $IP[1.1.1.1], $ORGREQ[user-agent: media_probe], put
   *, $ORGREQ[If-Modified-Since], unset
   *, $ORGREQ[If-None-Match], unset

   # #PROTOCOLキーワードを介してクライアントが要求したプロトコルをヘッダに追加します。
   $URL[*], $ORGREQ[X-Forwarded-Proto: #PROTOCOL], set


.. note::

   If-Modified-SinceヘッダとIf-None-Matchヘッダを ``unset`` するTTLが期限切れコンテンツは常に再度ダウンロードする。
