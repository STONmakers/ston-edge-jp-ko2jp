.. _bypass:

第8章 バイパス
******************

この章ではクライアント要求の処理をオリジンサーバーに委任するバイパスについて説明します。 バイパスは条件と動作的に区分されます。


.. note::

   - `[Q&A] リアルタイムで変更された情報は、キャッシングを適用することができないのではありませんか？ <https://www.youtube.com/watch?v=f28LL5Sm_rY&index=7&list=PLqvIfHb2IlKfRKtHg7vEZtrP7web63sW8>`_


バイパスはCachingポリシーよりも優先します。 設計段階でEdge導入が検討されていないサービスであれば静的リソースと動的リソースを精巧に区分こなせない場合が多い。 このような場合すべてのクライアントの要求をバイパスするように構成した後ログに基づいて要求の多いコンテンツのみCachingように設定します。 ほとんどの場合数時間のログだけでオリジンサーバーの負荷を劇的に下げることができます。
:ref:`monitoring_stats` でリアルタイムの情報を提供している理由もサービスをリアルタイムチューニングできるようにするためです。

バイパスは非常に速いだけでなくHTTPトランザクション単位で動作します。 いくらパーソナライズされたサイトといってもほとんどはメインページ（.html）のみ動的に変更されるだけで残りの99％は静的なリソースとして構成されている。 オリジンサーバーの動作に合わせることができるようオリジン :ref:`origin-httprequest` のバイパスバージョンが別々に存在します。


.. toctree::
   :maxdepth: 2


No-Cacheリクエストバイパス
====================================

クライアントがno-cacheリクエストを送信ばバイパスする。 ::

   GET / HTTP/1.1
   cache-control: no-cache または cache-control:max-age=0
   pragma: no-cache

::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <BypassNoCacheRequest>OFF</BypassNoCacheRequest>

-  ``<BypassNoCacheRequest>``

   - ``OFF (デフォルト)`` Cacheモジュールが処理します。

   - ``ON`` オリジンサーバーにバイパスします。

.. note::

    この設定はクライアントの動作（おそらく ``ctrl`` + ``F5`` )によって判断されます。 したがって大量のバイパスがオリジンサーバーに負担を与える可能性があります。


.. _bypass-getpost:

GET / POSTバイパス
====================================

バイパスがGET / POSTリクエストのデフォルトの動作になるように設定することができます。 GETとPOSTの用途が異なるだけデフォルトの動作が異なることに注意する必要があります。 ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <BypassPostRequest>ON</BypassPostRequest>
   <BypassGetRequest>OFF</BypassGetRequest>

-  ``<BypassPostRequest>``

   - ``ON (デフォルト)`` POSTリクエストをオリジンサーバーサーバーにバイパスします。

   - ``OFF`` POSTリクエストをSTONが処理します。

-  ``<BypassGetRequest>``

   - ``OFF (デフォルト)`` GETリクエストをSTONが処理します。

   - ``ON`` GETリクエストをオリジンサーバーにバイパスします。

:ref:`access-control-vhost` と同じ条件の両方をサポートします。 バイパス例外条件は/ svc / {仮想ホスト名} /bypass.txtに設定します。。 ::

   # /svc/www.example.com/bypass.txt
   $IP[192.168.2.1-255]
   /index.html

cacheやbypass条件を明確にしていない場合はデフォルトの設定と反対に動作します。 例えば ``<BypassGetRequest>`` が ``ON`` であれば例外条件はCachingリストになる。 混乱余地が多い場合2番目のパラメータを使用してより明確に条件を設定することができます。 ::

   # /svc/www.winesoft.co.kr/bypass.txt

   $HEADER[cookie: *ILLEGAL*], cache               // 常にCaching処理
   !HEADER[referer:]                               // デフォルトの設定に応じて
   !HEADER[referer] & !HEADER[user-agent], bypass  // 常にバイパス
   $URL[/source/public.zip]                        // デフォルトの設定に応じて

整理すると優先順位は次の通りです。

1. No-Cacheバイパス
2. bypass.txtにbypassと明記されて
3. bypass.txtのデフォルト設定



オリジンサーバーの固定
====================================

ログイン状態のようにオリジンサーバーとクライアントが必ず1：1で通信する必要があります。
`GET/POST バイパス`_ の属性にオリジンサーバーを固定させることができます。 ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <BypassPostRequest OriginAffinity="ON">...</BypassPostRequest>
   <BypassGetRequest OriginAffinity="ON">...</BypassGetRequest>

-  ``OriginAffinity``

   - ``ON (デフォルト)`` クライアント要求が常に同じサーバーにバイパスされることを保証します。 ただし同じソケットであることを保証するものではない。

     バイパスしなければならオリジンサーバーと接続しているすべてのソケットが切れる状況が発生することもあります。 しかしこのような場合でもサーバーに新しいソケット接続を要求します。

     .. figure:: img/private_bypass3.jpg
        :align: center

        常に同じサーバーにバイパスされます。

     バイパスたオリジンサーバーが障害を排除したりDNSに陥った場合新しいサーバーにバイパスされます。

   - ``OFF`` クライアントからの要求がどのサーバーにバイパスされることを保証することはできない。

     .. figure:: img/private_bypass1.jpg
        :align: center

        :ref:`origin-balancemode` によって従う。



オリジンセッション固定
====================================

クライアントソケットごとにオリジンサーバーと1：1でバイパスセッションを使用します。

.. figure:: img/private_bypass2.jpg
   :align: center

   クライアントがオリジンセッションを所有します。

`GET/POST バイパス`_ の属性にオリジンセッションを固定することができます。 ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <BypassPostRequest Private="OFF">...</BypassPostRequest>
   <BypassGetRequest Private="OFF">...</BypassGetRequest>

-  ``Private``

   - ``ON`` クライアントセッションがオリジンサーバー専用のセッションを使用するように動作します。 常に同じサーバーに要求がバイパスされます。 クライアントとオリジンサーバーのいずれか一方のセッションが終了された瞬間相手のセッションも終了されます。

   - ``OFF (デフォルト)`` 専用のセッションを使用しません。

オリジンサーバーがユーザーのログイン情報をセッションに基づいて維持する場合のようにクライアントの要求が必ずしも同じソケットに処理されるべき場合に便利です。

.. note::

   ややもするとあまりにも多くの要求を ``Private`` にバイパスする場合はクライアントの数だけオリジンサーバーに接続されて多大な負荷を与えることができます。 またこのように接続されたオリジンサーバーのセッションはクライアントが所有することになるので悪意のある攻撃の状況でのリスクをもたらすことができます。


Timeout
-----------------------

バイパスはオリジンサーバーから動的に処理した結果を応答する場合が多い。 これにより処理速度が静的なコンテンツよりも遅い場合が多い。 バイパス専用Timeoutを設定して不器用な障害の判断がされないようにします。 ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <BypassConnectTimeout>5</BypassConnectTimeout>
   <BypassReceiveTimeout>300</BypassReceiveTimeout>

-  ``<BypassConnectTimeout> (デフォルト: 5秒)``
   バイパスのためにn秒以内にオリジンサーバーとの接続が行われていない場合接続に失敗として処理します。


-  ``<BypassReceiveTimeout> (デフォルト: 5秒)``
   バイパス中オリジンサーバーからの応答がn秒間ない場合は送信失敗と処理します。



バイパスヘッダ
====================================

:ref:`origin-httprequest` 設定のバイパスを許可するを設定します。 ::

   # server.xml - <Server><VHostDefault><OriginOptions>
   # vhosts.xml - <Vhosts><Vhost><OriginOptions>

   <UserAgent Bypass="OFF">...</UserAgent>
   <Host Bypass="ON"/>
   <XFFClientIPOnly Bypass="ON">...</XFFClientIPOnly>

-  ``Bypass`` 属性

   - ``ON`` に設定されヘッダーを指定します。

   - ``OFF`` クライアントが送信した関連ヘッダを設定します。


.. _bypass-port:

Portバイパス
====================================

特定のTCPポートのすべてのパケットを送信元サーバーとしてバイパスします。 仮想ホストのみ設定だ。 ::

   # vhosts.xml - <Vhosts>

   <Vhost Name="www.example">
      <PortBypass>443</PortBypass>
      <PortBypass Dest=”1935”>1935</PortBypass>
   </Vhost>

-  ``<PortBypass>``
   指定されたポートで入力されたすべてのパケットをオリジンサーバーの同じポートにバイパスします。
   ``Dest`` 属性にオリジンサーバーのポートを設定します。

たとえばポート443をバイパスする場合クライアントはオリジンサーバーと直接SSL通信をする効果を持つ。 バイパスされるポートは絶対に重複することができない。

.. note::

   構造的にPortバイパスはHTTPより下位LayerであるTCPで行われる。 特定の仮想ホストの下位にPortバイパスを設定する理由は統計情報を収集する主体が必要だからだ。
