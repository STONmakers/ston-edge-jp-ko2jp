.. _caching-policy:

第4章 Cachingポリシー
******************

この章ではサービスの中核となるTTL（Time To Live）Caching-Keyと有効期限ポリシーについて説明します。 保存されたコンテンツはTTLの期間内に有効です。 HTTPの仕様はTTLを設定できるようにCache-Controlを明示しています。 しかしこれは絶対的なものではないです。 様々な方式のTTLポリシーと :ref:`caching-purge` を介してサービスの質を向上させることができます。


.. note::

   - `[Q&A] どのようなコンテンツをキャッシングすることができますか？ <https://www.youtube.com/watch?v=i7rCZ-6026I&index=9&list=PLqvIfHb2IlKfRKtHg7vEZtrP7web63sW8>`_


HTTPはコンテンツを区別する様々な規格が存在します。 それほどCaching-Keyも多様に存在することができます。 コンテンツの変更がないほど元の負荷を軽減することができるだけでなく簡単に拡張することができます。 サービスに最適化された有効期限ポリシーを策定する多様な方法について説明します。

これから説明される設定をすべての仮想ホストのデフォルト設定に適用したい場合は ``<VHostDefault>`` タグの下位に設定します。 反対に特定の仮想ホストにのみ適用さしたい場合は<Vhost>タグの下位に設定します。

**Caching-Key** はコンテンツを分離するユニークな値です。 ファイルシステム上のファイルと区別される固有のパス（例/usr/conf.txt）を持つのと同じ概念です。 **Caching-Key** はURLと混同されやすいです。 HTTPの複数の機能に応じて同じURLであってもコンテンツが変わることがあります。


.. toctree::
   :maxdepth: 2



.. _caching-policy-ttl:

TTL (Time To Live)
====================================

TTLは保存されたコンテンツの有効時間です。 TTLを長く設定するとオリジンサーバーの負荷は減りますが変更が遅れて反映されます。 反対に短く設定するとあまりにも頻繁な変更の確認要求にオリジンサーバーの負荷が高くなります。 運営の醍醐味はTTLを適切に設定して元の負荷を減らすことにあります。 TTLは一度設定されると有効期限が切れるまで変わらない。 新しいTTLはファイルの期限が切れるときに適用されます。 管理者は :ref:`api-cmd-purge` , :ref:`api-cmd-expire` , :ref:`api-cmd-expireafter` , :ref:`api-cmd-hardpurge` などのAPIを使用してTTLを変更することができます。


デフォルトTTL
---------------------

デフォルト的にはTTLはオリジンサーバーの応答に基づいて決定されます。 TTLが期限切れになるまで保存されたコンテンツにサービスされます。 TTLが期限切れになるとオリジンサーバーにコンテンツの変更有無( **If-Modified-Since** または **If-None-Match** )を確認します。 オリジンサーバーから **304 Not Modified** 応答をくる場合TTLは延長されます。 ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>

    <TTL>
        <Res2xx Ratio="20" Max="86400">1800</Res2xx>
        <NoCache Ratio="0" Max="5" MaxAge="0">5</NoCache>
        <Res3xx>300</Res3xx>
        <Res4xx>30</Res4xx>
        <Res5xx>30</Res5xx>
        <ConnectTimeout>3</ConnectTimeout>
        <ReceiveTimeout>3</ReceiveTimeout>
        <OriginBusy>3</OriginBusy>
    </TTL>

``Ratio`` （0〜100）を除くすべての設定単位は秒（sec）です。

-  ``<Res2xx> (デフォルト: 1800秒, Ratio: 20, Max=86400)``
   オリジンサーバーが200 OKで応答したときTTLを設定します。 コンテンツを最初に保存するときに ``<Res2xx>`` 秒後のコンテンツの期限が切れ（TTL）されるように設定します。 （TTL満了後）オリジンサーバーから変更されていない場合（304 Not Modified） ``Ratio`` 比率（0〜100）だけTTLを延長します。 TTLは最大 ``Max`` まで増加します。

-  ``<NoCache> (デフォルト: 5秒, Ratio: 0, Max=5, MaxAge=0)``
   ``<Res2xx>`` と同じかオリジンサーバーがno-cacheに応答する場合にのみ適用されます。 ::

      cache-control: no-cache または private または must-revalidate

   ``MaxAge`` が0より大きければmax-ageを与えることができます。

   .. figure:: img/nocache_maxage.png
      :align: center

      Max-AgeだけクライアントにCachingされる。

-  ``<Res3xx> (デフォルト: 300秒)``
   オリジンサーバーが3xxで応答したときのTTLを設定します。 Redirect用途に使用されている場合が多い。

-  ``<Res4xx> (デフォルト: 30秒)``
   オリジンサーバーが4xxに応答したときのTTLを設定します。
   **404 Not Found** の場合が多い。

-  ``<Res5xx> (デフォルト: 30秒)``
   オリジンサーバーが5xxに応答したときのTTLを設定します。 オリジンサーバーの内部障害状況である場合が多い。

-  ``<ConnectTimeout> (デフォルト: 3秒)``
   オリジンサーバーに接続できない場合のTTLを設定します。 コンテンツが既に保存されている場合は ``<ConnectTimeout>`` 秒だけTTLを延長します。 コンテンツが保存されていない場合は ``<ConnectTimeout>`` 秒ほどの障害状況に応答します。 これは障害の状況をサービスするという意味ではなくTTL時間（おそらく障害状況日）オリジンサーバーに負担をかけないため。

-  ``<ReceiveTimeout> (デフォルト: 3秒)``
   接続はされたがデータを受信していない場合のTTLを設定します。
   ``<ConnectTimeout>`` と意味的同じ。

-  ``<OriginBusy> (デフォルト: 3秒)``
   :ref:`origin-busysessioncount` 条件を満たした場合オリジンサーバーへの要求なしで有効期限が切れたコンテンツのTTLを設定した分延長します。 これはオリジンサーバーの負荷を加重させないため。

.. note::

   TTL値を0に設定するとサービスの直後すぐに切れます。 もしすべてのリクエストに対してオリジンサーバーの応答を与える必要がある場合バイパスすることを推奨します。


.. _caching-policy-customttl:

Custom TTL
---------------------

URLごとに個別にTTLを設定します。 明確なURLまたはパターンのURLマッチングされるコンテンツごとに固定されたTTLを設定することができます。 / svc / {仮想ホスト名} /ttl.txtに設定します。 ::

    # /svc/www.example.com/ttl.txt
    # 区切り文字はカンマ（、）であり時間の単位は秒です。

    *.jsp, 10
    /,5
    /index.html, 5
    /script/*.js, 300
    /image/ad.jpg, 1800


すべてのページ（htmlphpjspなど）に別途のTTLを設定するために* .htmlを追加しても初のページ（/）は設定されない。 オリジンサーバーが最初のページをどのページ（例えばindex.phpにdefault.jspなど）に設定したのかHTTPプロトコルでは知ることができない。 したがってすべてのページに個別のTTLを設定するためには/を追加する必要があります。


TTLの優先順位
---------------------

適用するTTL設定の優先順位を設定します。 ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>

    <TTL Priority="cc_nocache, custom, cc_maxage, rescode">
        ... (省略) ...
    </TTL>

``<TTL>`` の ``Priority (デフォルト: cc_nocache, custom, cc_maxage, rescode)`` 属性に設定します。

- ``cc_nocache`` オリジンサーバーがCache-Control：no-cacheに応答した場合
- ``custom`` `caching-policy-customttl`
- ``cc_maxage`` オリジンサーバーがCache-Controlにmaxageを明示した場合
- ``rescode`` 元応答コード別デフォルトのTTL


異常TTL延長
---------------------

オリジンサーバーのシャットダウンのための応答が来ない場合には障害の判断が明確であるがたまに正常に応答し障害の状況である場合が発生します。 たとえばコンテンツを保存するStorageとのアクセスを失うか通常の処理が不可能であると判断する場合があります。 前者の場合4xx応答（主に **404 Not Found** ), 後者は5xx応答（主に  **500 Internal Error** )を受けることになる。

しかしすでに関連するコンテンツが保存されている場合はテキストの応答を信じることよりもTTLを延長させてサービス全体に障害が発生しないようにしたほうが効果的です。 ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>

    <TTLExtensionBy4xx>OFF</TTLExtensionBy4xx>
    <TTLExtensionBy5xx>ON</TTLExtensionBy5xx>

-  ``<TTLExtensionBy4xx>``

   -  ``OFF (デフォルト)`` 4xx応答としてコンテンツを更新します。

   -  ``ON`` 304 not modifiedを受けたかのように動作します。

意図された4xx応答がないか注意する必要があります。

-  ``<TTLExtensionBy5xx>``

   -  ``ON (デフォルト)`` **304 Not Modified** を受けたかのように動作します。

   -  ``OFF`` 5xx応答でコンテンツを更新します。

通常のサーバーであれば5xxで応答しません。 主にサーバーの一時的な障害からのコンテンツを削除して元の負荷を加重させないための用途に使用されます。



.. _caching-policy-unvalidatable:

更新不可時のポリシー
---------------------

コンテンツのTTLは有効期限が切れましたがオリジンサーバーがすべて排除されて正常にコンテンツを更新（Revalidate）することができないときのポリシーを設定します。 ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>

    <UnvalidatableObjectResCode>0</UnvalidatableObjectResCode>

-  ``<UnvalidatableObjectResCode>``

   -  ``0 (デフォルト)`` 有効期限が切れたコンテンツのTTLを ``<ConnectTimeout>`` だけ延長します。

   -  ``HTTP 応答コード`` 更新することができない場合は設定された応答コードで応答します。 
   
例えば次のように設定されている場合はコンテンツを更新することができないとき有効期限が切れたコンテンツを404 Not Foundで応答します。 ::

   <UnvalidatableObjectResCode>404</UnvalidatableObjectResCode>



.. _caching-policy-renew:

更新ポリシー
====================================

TTL期限切れのコンテンツの場合オリジンサーバーの更新有無を確認した後サービスが行われます。

   .. figure:: img/perf_refreshexpired.jpg
      :align: center

      変更の確認後応答

1. TTLが有効であります。 即座に応答します。

#. TTLが期限切れになってオリジンサーバーに変更を確認（If-Modified-Since）を要請します。 変更の確認がされるまでクライアントに応答しません。

#. オリジンサーバーからの応答があるばあいTTLを延長まやはコンテンツを変更（Swap）します。 オリジンサーバーで確認がされたのでクライアントに応答します。

#. 変更の確認がされたコンテンツであるため次のTTL満了時まで即座に応答します。

高画質動画やゲームのような比較的反応性よりも転送速度が重要なサービスではこの方法が大きな問題にならない。 大容量データの場合はオリジンサーバーが10秒で更新の応答をしても送信に数分かかるので比較的オリジンサーバーの反応性が大きく重要ではありません。 むしろアクセス頻度が高くないコンテンツであるため必ず更新確認を行なう必要があります。

しかしショッピングモールのようなサービスの場合、状況は異なります。 Webページはすぐにロードされていることが何よりも重要であります。 1〜2秒以内にクライアントの画面表示が完了される必要があります。 転送速度よりも反応速度がより重要になります。

この時TTLが期限切れになってオリジンサーバーに更新を確認すると非常に大きな遅延が発生します。 通常のショッピングモールが数百万個のコンテンツを同時にサービスすることを考えるといつもオリジンサーバーからの更新確認作業が発生していると考えなければならない。 もしオリジンサーバーに障害が発生したりネットワーク障害が発生した場合大きな問題になります。

サービス提供側の要望はオリジンサーバーの障害や遅延からキャッシュされたコンテンツが安全に配信されることです。

   .. figure:: img/perf_refreshexpired2.jpg
      :align: center

      障害怖くない！

そのためバックグラウンドコンテンツ更新機能が開発されました。 ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>

    <RefreshExpired>ON</RefreshExpired>

-  ``<RefreshExpired>``

   -  ``ON (デフォルト)`` 変更の確認後に応答します。

   -  ``OFF`` 変更の確認応答を待たずに応答します。 新しいコンテンツのダウンロード`後変更（Swap）します。

``OFF`` 設定のより大きな理由はコンテンツがほとんど変更頻度ないからだ。

   .. figure:: img/perf_refreshexpired5.jpg
      :align: center

      変更に敏感でない場合は待機しません。

上の図ではオリジンサーバーとの更新作業がすべてバックグラウンドで行われるためキャッシュされたコンテンツは待機せずにすぐにクライアントにサービスされます。 オリジンサーバーが **304 Not Modified** に応答する場合TTLだけ延長されます。 ファイルが更新されオリジンサーバーから200 OKを応答した場合はそのファイルが完全にダウンロードされた後にファイルがスムーズに切り替えられる。 コンテンツが変わっても以前のコンテンツ（緑）をダウンロード受けたユーザーは通常のダウンロードが行われる。 ファイル交換後のアクセスされたユーザー（からし色）は変更したファイルにサービスされます。 コンテンツの更新ネットワーク障害オリジンサーバー障害など何らかの変数もコンテンツ更新はバックグラウンドで行われるため実際のサービスには全く遅れがない。


クライアントno-cacheリクエストTTLの有効期限
---------------------

クライアントのHTTP要求にno-cacheの設定が複数記載されている場合そのコンテンツをすぐに期限切れにさせることができます。 ::

    GET /logo.jpg HTTP/1.1
    ...
    cache-control: no-cache または cache-control:max-age=0
    pragma: no-cache
    ...

::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>

    <NoCacheRequestExpire>OFF</NoCacheRequestExpire>

-  ``<NoCacheRequestExpire>``

   -  ``OFF (デフォルト)`` は無視します。

   -  ``ON`` TTLをすぐに満了します。

有効期限が切れたコンテンツは `更新ポリシー`_ に従う。


.. _caching-policy-accept-encoding:

Accept-Encodingヘッダ
====================================

同じURLへのHTTP要求であってもAccept-Encodingヘッダの存在の有無に応じて他のコンテンツがキャッシュされることができます。 オリジンサーバーに要求を送信する時に圧縮有無を知ることができない。 応答を受けたとしても圧縮有無を毎回比較することもない。

   .. figure:: img/acceptencoding.png
      :align: center

      オリジンサーバーがどんな答えを与える知ることができない。

::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>

    <AcceptEncoding>ON</AcceptEncoding>

-  ``<AcceptEncoding>``

   -  ``ON (デフォルト)`` HTTPクライアントが送信するAccept-Encodingヘッダを認識します。

   -  ``OFF`` HTTPクライアントが送信しAccept-Encodingヘッダを無視します。

オリジンサーバーで圧縮をサポートしていないかまたは圧縮が必要な大容量ファイルの場合 ``OFF`` に設定することが望ましい。


.. _caching-policy-casesensitive:

大文字と小文字の区別
====================================

オリジンサーバー上のコンテンツの大文字と小文字の識別ができない。

   .. figure:: img/casesensitive.png
      :align: center

      おそらくそのようなコンテンツであるか404が発生します。

::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>

    <CaseSensitive>ON</CaseSensitive>

-  ``<CaseSensitive>``

   -  ``ON (デフォルト)`` URL大文字と小文字を構文であります。

   -  ``OFF`` URL大文字と小文字を区別しません。 すべて小文字で処理されます。


.. _caching-policy-applyquerystring:

QueryString区分
====================================

QueryStringによって動的に生成されたコンテンツではない場合はQueryStringを認識する必要はありません。 何の意味のないRandom値や常に変化するtimestampの値が毎回付く場合オリジンサーバーに多大な負荷が発生する恐れがあります。

   .. figure:: img/querystring.png
      :align: center

      動的なコンテンツがない場合は同じコンテンツである可能性が高い。

::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>

    <ApplyQueryString Collective="OFF">ON</ApplyQueryString>

-  ``<ApplyQueryString>``

   -  ``ON (デフォルト)`` QueryStringを認識します。 例外条件に満足するQueryStringが無視されます。

   -  ``OFF`` QueryStringを無視します。 例外条件に満足するQueryStringを認識します。

QueryString-例外条件は/ svc / {仮想ホスト名} /querystring.txtに設定します。 ::

    # ./svc/www.example.com/querystring.txt

    /private/personal.jsp?login=ok*
    /image/ad.jpg

例外条件が ``<ApplyQueryString>`` の設定に応じ意味が異なることに注意する必要があります。 明確なURLまたはパターン（ *のみを可能にする）に設定が可能であります。

``Collective`` 属性は :ref:`api-cmd-purge` APIが呼び出されたときにターゲットを指定します。

-  ``Collective``

   -  ``OFF (デフォルト)`` パラメータURLのみを対象とします。

   -  ``ON`` パラメータURLだけでなくURLのQueryStringが存在するすべてのコンテンツを対象に指定します。

``Collective`` 属性がONでありファイルが多ければ多いほど :ref:`api-cmd-purge` API実行CPU負荷が高くなる。 関連ファイルを検索する時間が長くなることがあり予期しない問題が発生することができます。 なるべくQueryStringまでついた明確なURLに :ref:`api-cmd-purge` APIを呼び出すことをお勧めします。




Varyヘッダ
====================================

Varyヘッダを認識してコンテンツを区分します。 一般的にVaryヘッダはCacheサーバーのパフォーマンスを大幅に落とす原因になります。 ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>

    <VaryHeader />

-  ``<VaryHeader>``

   オリジンサーバーが応答したVaryヘッダのサポートヘッダーのリストを設定します。 区切り文字はカンマ（、）を使用します。

たとえばオリジンサーバーが次のようにVaryヘッダを送ったとしても ``<VaryHeader>`` が設定されていない場合は無視します。 ::

    Vary: Accept-Encoding, Accept, User-Agent

User-Agentを除くAccept-EncodingとAcceptヘッダのみを認識するようにするには次のように設定します。 ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>

    <VaryHeader>Accept-Encoding, Accept</VaryHeader>

オリジンサーバーが送信したすべてのVaryヘッダを認識するようにするには次のように設定します。 ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>

    <VaryHeader>*</VaryHeader>


.. _caching-policy-post-method-caching:

POSTリクエストのキャッシュ
====================================

POSTリクエストをCachingするように設定します。 POSTリクエストの特性上URLは同じですがBodyデータが異なる場合があります。 ::

    # server.xml - <Server><VHostDefault><Options>
    # vhosts.xml - <Vhosts><Vhost><Options>

    <PostRequest MaxContentLength="102400" BodySensitive="ON">OFF</PostRequest>

-  ``<PostRequest>``

   -  ``OFF (デフォルト)``  POSTリクエストが来ればセッションを終了します。

   -  ``ON`` POSTリクエストをCachingします。

実際にPOSTリクエストを処理するほとんどの場合はBodyデータをCaching-Keyとして使用します。
``BodySensitive`` 属性と例外条件を使用して洗練された設定が可能であります。

-  ``BodySensitive``

    -  ``ON (デフォルト)`` BodyデータまでCaching-Keyとして認識します。 最大の長さは ``MaxContentLength (デフォルト: 102400 Bytes)`` 属性に制限します。 例外条件に満足するBodyデータを無視します。

    -  ``OFF`` Bodyデータは無視します。 例外条件に満足するBodyデータを認識します。

POSTリクエストの例外条件は/ svc / {仮想ホスト名} /postbody.txtに設定します。 ::

    # /svc/www.example.com/postbody.txt

    /bigsale/*.php?nocache=*
    /goods/search.php

例外条件が ``BodySensitive`` 設定に応じて意味が異なることに注意する必要があります。 明確なURLまたはパターン（ *のみを許可します。）に設定が可能であります。

この設定は :ref:`bypass-getpost` とポリシー的に混乱になる可能性があります。
``<BypassPostRequest> (デフォルト: ON)`` によりPOSTリクエストがキャッシュされないことがあります。 したがってPOSTリクエストをキャッシュするためには ``<BypassPostRequest>`` を ``OFF`` または例外条件を設定する必要があります。 整理すると優先順位は次の通りであります。

* バイパス条件( :ref:`bypass-getpost` )に満足した場合オリジンサーバーにバイパスします。
* Content-Lengthヘッダがない場合は接続を終了します。
* ``PostRequest`` が ``ON`` に設定されてContent-Lengthが ``MaxContentLength`` 属性値を超えなければキャッシュモジュールによって処理されます。
* 以上のシナリオでは未処理の要求は終了します。

.. note::

    ``MaxContentLength`` 属性をも大きく設定した場合Caching-Key管理に多くのメモリが必要になります。可能な限り小さく設定するのが良い。
