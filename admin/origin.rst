.. _origin:

第7章 オリジンサーバー
******************

この章ではSTONとオリジンサーバーの関係について説明します。 オリジンサーバーは一般的にHTTPの仕様に準拠しているWebサーバーを意味します。 管理者はサービスを保護するために今回の章のすべての内容を熟知する必要があります。 これを基にオリジンサーバーの障害にも耐久性を備えた柔軟なサービスを構築することができます。

オリジンサーバーは保護されるべきです。 障害の種類が多様なほど対処案も多様です。 オリジン保護ポリシーを適切に設定するとゆったりとした点検時間を持つことができます。


.. toctree::
   :maxdepth: 2



.. _origin_exclusion_and_recovery:

障害の検知と復旧
====================================

Caching過程の中でオリジンサーバーに障害が発生した場合は自動的に排除します。 復元されたと判断するとサービスに投入します。 ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <ConnectTimeout>3</ConnectTimeout>
   <ReceiveTimeout>10</ReceiveTimeout>
   <Exclusion>3</Exclusion>
   <Recovery Cycle="10" Uri="/" ResCode="0" Log="ON">5</Recovery>

-  ``<ConnectTimeout> (デフォルト: 3秒)``

   n秒以内にオリジンサーバーとの接続が行われていない場合接続に失敗とみなす。

-  ``<ReceiveTimeout> (デフォルト: 10秒)``

   通常のHTTPリクエストにもかかわらずオリジンサーバーがHTTP応答をn秒間送信しない場合は送信が失敗であると考えている。

-  ``<Exclusion> (デフォルト: 3回)``

   オリジンサーバーで連続的にn回障害状況( ``<ConnectTimeout>`` または ``<ReceiveTimeout>`` )が発生した場合そのサーバーを有効オリジンサーバーのリストから排除します。 排除前通常の通信が行われた場合はこの値は0に初期化されます。

-  ``<Recovery> (デフォルト: 5回)``

   ``Cycle`` ごと ``Uri`` に要求してオリジンサーバーが ``ResCode`` に連続的にn回応答するとそのサーバーを復旧します。 この値を0に設定すると復旧しません。

   -  ``Cycle (デフォルト: 10秒)`` 一定時間（秒）ごとにしようとします。

   -  ``Uri (デフォルト: /)`` 要求を送信Uri

   -  ``ResCode (デフォルト: 0)`` 正常応答として処理する応答コード。 0の場合は応答コードに関係なく応答が来れば成功とみなす。 200に設定すると応答コードが必ず200でなければなら正常応答で処理します。 コンマ（）を使用して有効な応答コードをマルチに設定します。 200、206、404に設定すると応答コードがこれらのいずれかである場合通常の応答で処理します。

   -  ``Log (デフォルト: ON)`` 復旧のために使用されたHTTP Transactionを :ref:`admin-log-origin` に記録します。



.. _origin-health-checker:

Health-Checker
====================================

`障害の検出と回復`_ はCaching過程中に発生する障害に対応します。
``<Recovery>`` は応答コードを受け取り次第HTTP Transactionを終了します。 しかしHealth-CheckerはHTTP Transactionが成功することを確認します。 ::

   # vhosts.xml - <Vhosts><Vhost>

   <Origin>
      <Address> ... </Address>
      <HealthChecker ResCode="0" Timeout="10" Cycle="10"
                     Exclusion="3" Recovery="5" Log="ON">/</HealthChecker>
      <HealthChecker ResCode="200, 404" Timeout="3" Cycle="5"
                     Exclusion="5" Recovery="20" Log="ON">/alive.html</HealthChecker>
   </Origin>

-  ``<HealthChecker> (デフォルト: /)``

   Health-Checkerを構成します。 マルチで構成が可能です。 値としてUriを指定しXML例外文字の場合CDATAを使用します。

   -  ``ResCode (デフォルト: 0)`` 正しい応答コード（コンマでマルチ構成可能）

   -  ``Timeout (デフォルト: 10秒)`` ソケット接続からHTTP Transactionが完了するまで有効時間

   -  ``Cycle (デフォルト: 10秒)`` 実行サイクル

   -  ``Exclusion (デフォルト: 3回)`` 連続n回失敗した場合そのサーバー排除

   -  ``Recovery (デフォルト: 5回)`` 連続n回成功した場合そのサーバーの再投入

   -  ``Log (デフォルト: ON)`` HTTP Transactionを :ref:`admin-log-origin` に記録します。

Health-Checkerはマルチで構成することができクライアントの要求に関係なく独立して実行されます。
`障害の検出と回復`_ や他のHealth-Checkerとも情報を共有せずに自分だけの基準で排除および責任を決定します。


.. _origin-use-policy:

送信元アドレスを使用ポリシー
====================================

送信元アドレス（IP）は次の要素によってどのように使用されるか決定されます。

-  :ref:`env-vhost-activeorigin` アドレスの形式（IPアドレスまたはDomain）とセカンダリアドレス
-  `障害の検出と回復`_
-  `Health-Checker`_

サービスを運営しているとオリジンのアドレスが排除/復旧されることは頻繁です。 STONはIPテーブルに基づいてオリジンアドレスを使用して `origin-status`_ APIを介して情報を提供します。

送信元アドレスをIPに設定した場合非常に簡単です。

-  設定の変更に加えてIPリストを変化させる要因はない。
-  TTLによりIPアドレスが期限切れにならない。
-  障害/復旧の両方の設定（IPアドレス）に基づいて動作します。

送信元アドレスをDomainに設定するとResolvingてIPアドレスを得なければならない。
( :ref:`admin-log-dns` に記録されます。)
IPリストは動的に変更されることができすべてのIPはTTL（Time To Live）の間だけ有効です。

-  Domainは定期的に（1〜10秒）Resolvingします。
-  Resolvingを介して使用するIPテーブルを構成します。
-  すべてのIPはTTL分だけ有効でありTTLが期限切れになると使用しません。
-  同じIPアドレスが再びResolvingされるとTTLを更新します。
-  IPテーブルは空白ではない。 （TTLが期限切れにされても）最後のIPは削除されない。

IPのTTLが長すぎる場合過度に多くのIPアドレスを使用するようになって意図しない結果を作成することができます。 これを防止するためにIPの最大TTLを制限することができます。 ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <DnsMaxTTL>60</DnsMaxTTL>

-  ``<DnsMaxTTL> (デフォルト: 60秒)`` ResolvingされたIPアドレスの最大使用時間（秒）を設定します。 この値が0の場合はDNSから提供されたTTLをそのまま使用します。


.. note::

   送信元アドレスをDomainに設定しても障害/復旧はIPベースで動作します。 Domainアドレス障害/復旧ポリシーは次のとおりです。

   -  （Domainについて）知っているすべてのIPアドレスが排除（Inactive）とそのDomainアドレスが排除されます。
   -  新規IPがResolvingもDomainが排除されている場合はIPアドレスは最初から排除されます。
   -  すべてのIPアドレスはTTL有効期限が切れても排除されたDomain状態は解けない。
   -  排除されたDomainに属するIPアドレスのいずれかが復旧されるべきはDomainは再び有効にされます。

   やや複雑な内容であるため `origin-status`_ APIを使用してサービスの動作状態について理解を高めるのが良い。



.. _origin-status:

オリジンの状態のモニタリング
====================================

APIを介して仮想ホストのオリジンの状態をモニタリングします。 ::

   http://127.0.0.1:10040/monitoring/origin       // すべての仮想ホスト
   http://127.0.0.1:10040/monitoring/origin?vhost=www.example.com

結果はJSON形式で提供されます。 ::

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
   設定アドレスが使用中であれば ``Active`` 、 （障害が発生して）使用していない場合 ``Inactive`` として表示されます。

-  ``Address2`` :ref:`env-vhost-standbyorigin` 。
   設定アドレスを使用中であれば ``Active`` 、使用していない場合 ``Inactive`` として表示されます。

-  ``ActiveIP`` 使用中のIPリストとTTL。 オリジンサーバーをIPに設定すると ``Address`` と同じIPにTTLは0と表示されます。 Domainに設定するとResolving結果に従う。 様々なIPとTTLを使用します。

-  ``InactiveIP`` 使用しないIPリストとTTL。 使用していなくても復旧しているかHealthCheckerによって管理することができます。 そのアドレスはTTLの間復旧されなければ削除されます。



.. _origin-status-reset:

オリジンの状態の初期化
====================================

APIを介して仮想ホストのオリジンサーバー排除/復旧を初期化します。 また現在使用中のセッションを再利用せずに新たに接続を作成します。 ::

   http://127.0.0.1:10040/command/resetorigin       // すべての仮想ホスト
   http://127.0.0.1:10040/command/resetorigin?vhost=www.example.com



.. _origin-busysessioncount:

過負荷判断
====================================

最初に要求されているコンテンツは常にオリジンサーバーに要求する必要があります。 しかしすでにCachingされたコンテンツであればより柔軟に対応することができます。 オリジンサーバーが過負荷状態と判断されると更新をずらしてオリジンサーバーの負荷を高くない。 ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <BusySessionCount>100</BusySessionCount>

-  ``<BusySessionCount> (デフォルト: 100個)``
   オリジンサーバーとHTTPトランザクションを進行中のセッションの数が一定数を超えると過負荷状態と判断します。 過負荷状態で有効期限が切れたコンテンツを更新するためにオリジンサーバーに接続しないようにTTLを :ref:`caching-policy-ttl` の ``<OriginBusy>`` だけ延長します。 無条件元サーバーに要求が行くようにするにはこの値を非常に大きく設定すればよい。


.. _origin-balancemode:

オリジンサーバーの選択
====================================

元サーバーのアドレスがマルチ（2個以上）で構成されているときにオリジンサーバーの選択ポリシーを設定します。 ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <BalanceMode>RoundRobin</BalanceMode>

-  ``<BalanceMode> (デフォルト: RoundRobin)``

   -  ``RoundRobin (デフォルト)``
      すべてのオリジンサーバーが均等に要求を受信するようにRoundRobinで動作します。 接続されたIdleセッションはそのサーバーに要求が必要な場合にのみ使用します。

   -  ``Session``
      再使用可能なセッションがある場合まず使用します。 新規セッションが必要な場合Round-Robinに割り当てる。

   -  ``Hash``
      コンテンツを `Consistent Hashing <http://en.wikipedia.org/wiki/Consistent_hashing>`_ アルゴリズムに基づいてオリジンサーバーに分散して要請します。 サーバーが選択されると既に接続されたセッションを再利用しない場合は新規に接続します。


=========== =================================================================== =====================================================
/           RoundRobin                                                          Session
=========== =================================================================== =====================================================
負荷（要求）	  すべてのサーバーが負荷を均等に分配	                                反応性と再利用性が良いサーバーに負荷が加重される
接続コスト	   高（そのサーバーの順序がされると、接続されたセッションを探していない場合、接続しようと）   低（再使用可能なセッションがない場合のみ接続）
再利用性	    低（サーバー分配優先）	                                            高（常に接続されたセッションを優先使用）
セッション数	    沢山（各サーバーごとに同時に進行されるHTTPトランザクションの合計）          少ない（同時に進行されるHTTPトランザクションだけセッションが存在）
=========== =================================================================== =====================================================


セッションの再利用
====================================

オリジンサーバーがKeep-Aliveをサポートする場合接続されたセッションは常に再使用されます。 しかしセッションを再利用して送信要求に対してオリジンサーバーが一方的に接続を終了することができます。 ので接続を復旧するのにユーザーの反応が遅くなる可能性があります。 特に長い間再使用していないセッションの場合これらの可能性はさらに高い。 これを防止するためにn秒の間再利用されていないセッションに対して自動的に接続を終了するように設定します。 ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <ReuseTimeout>60</ReuseTimeout>

-  ``<ReuseTimeout> (デフォルト: 60秒)``
   一定時間使用されていないオリジンサーバーのセッションは終了します。 0に設定するとオリジンサーバーのセッションを再利用しません。


.. _origin_partsize:

Rangeリクエスト
====================================

一度ダウンロードされたコンテンツのサイズを設定します。 動画のように前の部分だけが主に消費されるコンテンツの場合ダウンロードサイズを制限すると不要なオリジントラフィックを減らすことができます。 ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <PartSize>0</PartSize>

-  ``<PartSize> (デフォルト: 0 MB)``
   0よりも大きい場合クライアントが要求した時点から設定サイズ（MB）だけRangeリクエストにダウンロードします。


``<PartSize>`` を使用しているもう一つの理由はディスクスペースを節約するためです。 デフォルトの設定でSTONは元のサイズのファイルをディスク上に作成します。 しかし ``<PartSize>`` が0でない場合はダウンロードされている分だけファイルを分割して保存します。

たとえば1時間の映像（600MB）を1分（10MB）のみ視聴した場合にディスク領域を10MBだけを使用します。 スペースを節約する利点はあるがファイルが分割されて保存されるためディスク負荷が少し高くなる。

.. note::

   最初のコンテンツをダウンロードする際にContent-Lengthを知ることができないのでRange要求をすることができない。 ため ``<PartSize>`` が設定されている場合は設定サイズ分だけダウンロードして接続を終了します。




全体Range初期化
====================================

一般的にオリジンサーバーから最初のファイルをダウンロードするときや更新確認するときは次のように単純な形のGETリクエストを送信します。 ::

    GET /file.dat HTTP/1.1

しかしオリジンサーバーが一般的なGETリクエストに対して常にファイルを改ざんするように設定されている場合は元のファイルをそのままCachingできなく問題になることがあります。

最も代表的な例はApache Webサーバがmod_h.264_streamingなどの外部モジュールのように駆動される場合です。 Apache WebサーバーはGETリクエストに対して常にmod_h.264_streamingモジュールを介して応答します。 クライアント（この場合にはSTON）は元のファイルのままではなくモジュールによって変調されたファイルをサービス受ける。

   .. figure:: img/conf_origin_fullrangeinit1.png
      :align: center

      mod_h.264_streamingモジュールは常にオリジンサーバーを変調します。

Range要求を使用するとモジュールをバイパスして元のをダウンロードすることができます。 ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <FullRangeInit>OFF</FullRangeInit>

-  ``<FullRangeInit>``

   - ``OFF (デフォルト)`` の一般的なHTTPリクエストを送る。

   - ``ON`` 0から始まるRangeリクエストを送る。 Apacheの場合Rangeヘッダが指定されるとモジュールをバイパスします。 ::

        GET /file.dat HTTP/1.1
        Range: bytes=0-

     最初にファイルCachingするときはコンテンツのRangeを知らないのでFull-Range（= 0から始まる）を要請します。 オリジンサーバーがRange要求に対して正常に応答（206 OK）かどうかを必ず確認する必要があります。

コンテンツを更新するときは次のように **If-Modified-Since** ヘッダが一緒に指定されます。 オリジンサーバーが正しく **304 Not Modified** に応答する必要があります。 ::

   GET /file.dat HTTP/1.1
   Range: bytes=0-
   If-Modified-Since: Sat, 29 Oct 1994 19:43:31 GMT

.. note::

   ``<FullRangeInit>`` が正常に動作することを確認しWebサーバーのリスト。

   - Microsoft-IIS/7.5
   - nginx/1.4.2
   - lighttpd/1.4.32
   - Apache/2.2.22


.. _origin-wholeclientrequest:

クライアントの要求を維持
====================================

元に要求するときクライアントが送信した要求を維持するように設定します。 ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <WholeClientRequest>OFF</WholeClientRequest>

-  ``<WholeClientRequest>``

   - ``OFF (デフォルト)`` Caching-Keyを元に要求するURLに使用します。

   - ``ON`` クライアントが要求されたURLに元に要請します。

Hit Ratioを高めるために次の設定を使用してCaching-Keyを決定します。

- :ref:`caching-policy-casesensitive`
- :ref:`caching-policy-applyquerystring`
- :ref:`caching-policy-post-method-caching`

これによりオリジンサーバーに要求したURLとCaching-Keyは次のように決定されます。

============================================== ======================= ============================
設定                                           クライアント要求URL       オリジン要求URL / Caching-Key
============================================== ======================= ============================
:ref:`caching-policy-casesensitive` ``OFF``    /Image/LOGO.png         /image/logo.png
:ref:`caching-policy-casesensitive` ``ON``     /Image/LOGO.png         /Image/LOGO.png
:ref:`caching-policy-applyquerystring` ``OFF`` /view/list.php?type=A   /view/list.php
:ref:`caching-policy-applyquerystring` ``ON``  /view/list.php?type=A   /view/list.php?type=A
============================================== ======================= ============================

``<WholeClientRequest>`` を ``ON`` に設定すると次のようにCaching-Keyとは関係なくクライアントが送信したURLをそのまま元に送る。

============================================== =================================== ============================
設定                                            クライアント/オリジン要求URL           Caching-Key
============================================== =================================== ============================
:ref:`caching-policy-casesensitive` ``OFF``    /Image/LOGO.png                     /image/logo.png
:ref:`caching-policy-casesensitive` ``ON``     /Image/LOGO.png                     /Image/LOGO.png
:ref:`caching-policy-applyquerystring` ``OFF`` /view/list.php?type=A               /view/list.php
:ref:`caching-policy-applyquerystring` ``ON``  /view/list.php?type=A               /view/list.php?type=A
============================================== =================================== ============================

POSTリクエストをキャッシュする場合オリジンサーバーに要求したときクライアントが送信したPOSTリクエストのBodyデータが変更なしで送信されます。

.. note::

   クライアントが送信したURLをそのまま送信するため :ref:`media-trimming` ようアドオンのためにつけられたQueryStringもそのままオリジンサーバーに転送されます。


.. _origin-httprequest:

オリジン要求のデフォルトHeader
====================================

Hostヘッダ
---------------------

オリジンサーバーに送信するHTTPリクエストのHostヘッダを設定します。 別に設定していない場合は仮想ホスト名が指定されます。 ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <Host />

-  ``<Host>``
   オリジンサーバーに送信するHostヘッダを設定します。 オリジンサーバーで80ポート以外のポートでサービスしている場合必ずポート番号を設定する必要があります。 ::

      # server.xml - <Server><VHostDefault><OriginOptions>
      # vhosts.xml - <Vhosts><Vhost><OriginOptions>

      <Host>www.example2.com:8080</Host>


クライアントが送信したHostヘッダを元に送りたい場合*に設定します。


User-Agentヘッダ
---------------------

オリジンサーバーに送信HTTPリクエストのUser-Agentヘッダを設定します。 ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <UserAgent>STON</UserAgent>

-  ``<UserAgent> (デフォルト: STON)``
   オリジンサーバーに送信UserAgentヘッダーを設定します。


クライアントが送信したUser-Agentヘッダを元に送りたい場合*に設定します。


XFF(X-Forwarded-For) ヘッダ
---------------------

クライアントとオリジンサーバーとの間にSTONが位置するとオリジンサーバーはクライアントのIPアドレスを取得することができない。そのためSTONはオリジンサーバーに送信されるすべてのHTTP要求にX-Forwarded-Forヘッダを設定します。 ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <XFFClientIPOnly>OFF</XFFClientIPOnly>

-  ``<XFFClientIPOnly>``

   - ``OFF (デフォルト)`` クライアント（IP：128.134.9.1）が送信XFFヘッダにクライアントのIPアドレスを追加します。 クライアントがXFFヘッダを送信していない場合クライアントのIPのみ指定されます。 ::

        X-Forwarded-For: 220.61.7.150, 61.1.9.100, 128.134.9.1

   - ``ON`` XFFヘッダの最初のアドレスだけをオリジンサーバーに転送します。 ::

        X-Forwarded-For: 220.61.7.150


ETagヘッダ認識
---------------------

オリジンサーバーからの応答するETag認識有無を設定します。 ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <OriginalETag>OFF</OriginalETag>

-  ``<OriginalETag>``

   - ``OFF (デフォルト)`` ETagヘッダを無視します。

   - ``ON`` ETagを認識しコンテンツ更新時If-None-Matchヘッダを追加します。



.. _origin_url_rewrite:

オリジン要求URLの変更
====================================

キャッシュを目的として元に送信するHTTPリクエストのURLを変更します ::

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

:ref:`handling_http_requests_url_rewrite` のような表現を使用しますが仮想ホストごとに独立して設定するため仮想ホスト名を入力しません。

.. note::

   バイパスされるHTTP要求のURLは変更できない。
   ``<WholeClientRequest>`` よりも優先します。



.. _origin_modify_client:

オリジン要求ヘッダの変更
====================================

オリジンサーバーにHTTPリクエストを送信するときの条件に応じてHTTPヘッダーを変更します。 ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <ModifyHeader FirstOnly="OFF">OFF</ModifyHeader>

-  ``<ModifyHeader>``

   -  ``OFF (デフォルト)`` 変更しません。

   -  ``ON`` ヘッダ変更条件に応じてヘッダーを変更します。

ヘッダ変更時点はHTTP要求パケットが完成されてオリジンサーバーに転送する直前に実行されます。 ただしRangeヘッダは変調することができない。

この機能は :ref:`handling_http_requests_modify_client` のサブ機能です。 ヘッダ変更には$ ORGREQキーワードを使用します。 ::

   # /svc/www.example.com/headers.txt

   $URL[/*.mp4], $ORGREQ[x-media-type: video/mp4], set
   $IP[1.1.1.1], $ORGREQ[user-agent: media_probe], put
   *, $ORGREQ[If-Modified-Since], unset
   *, $ORGREQ[If-None-Match], unset

   # #PROTOCOLキーワードを介してクライアントが要求したプロトコルをヘッダに追加します。
   $URL[*], $ORGREQ[X-Forwarded-Proto: #PROTOCOL], set


.. note::

   If-Modified-SinceヘッダとIf-None-Matchヘッダを ``unset`` するTTLが期限切れコンテンツはいつも再度ダウンロードします。
