.. _env:

第3章 設定構造
******************

.. note::

   - `[動画講座]みよう！ STON Edge Server - Chapter 2.を設定する <https://youtu.be/BROcSuFyHOQ?list=PLqvIfHb2IlKeZ-Eym_UPsp6hbpeF-a2gE>`_
   - `[動画講座]みよう！ STON Edge Server - Chapter 3.仮想ホスト作成 <https://youtu.be/AvxxSWgXcqA?list=PLqvIfHb2IlKeZ-Eym_UPsp6hbpeF-a2gE>`_

この章では設定の構造と変更された設定を適用する方法を説明します。 構造を正確に理解すれば迅速なサーバーデプロイ・障害状況にも円滑な対応ができます。

設定は大きく全域（server.xml）と仮想ホスト（vhosts.xml）に分けられる。

   .. figure:: img/conf_files.png
      :align: center

      2つの.xmlファイルがすべてです。

2つのXMLファイルでほとんどのサービスを構成します。 複数のTXTファイルで仮想ホスト別の例外条件を設定し特定機能のリストを作成するのに使用されます。完全な形のXMLは以下の形式になります。 ::

   <Server>
       <VHostDefault>
           <Options>
               <CaseSensitive>ON</CaseSensitive>
           </Options>
       </VHostDefault>
   </Server>

説明では次のように省略して表記します。 ::

   # server.xml - <Server><VHostDefault><Options>

   <CaseSensitive>ON</CaseSensitive>


.. note::

   ライセンス（license.xml）は設定がありません。


.. _api-conf-reload:

設定Reload
====================================

設定の変更後に管理者が明確にAPIを呼び出す必要があります。 システムパフォーマンス関連の設定以外のほとんどの設定はサービスを中断せずすぐに適用されます。 ::

   http://127.0.0.1:10040/conf/reload

設定が変更されるたびに :ref:`admin-log-info` に変更が記録されます。


server.xmlグローバル設定
====================================

実行可能ファイルと同じパスに存在するserver.xmlこのグローバル設定ファイルです。 XML形式のテキストファイルです。 ::

    # server.xml

    <Server>
        <Host> ... </Host>
        <Cache> ... </Cache>
        <VHostDefault> ... </VHostDefault>
    </Server>

まずグローバル設定の構造と簡単な機能を説明します。グローバル設定中ボリュームが大きい機能については :ref:`access-control` や :ref:`snmp` などの各トピックをカバーする章で説明します。

.. toctree::
   :maxdepth: 2


.. _env-host:

管理者の設定
------------------------------------

管理目的の機能を設定します。 ::

    # server.xml - <Server>

    <Host>
        <Name>stream_07</Name>
        <Admin>admin@example.com</Admin>
        <Manager Port="10040" HttpMethod="ON" Role="Admin" UploadMultipartName="confile">
            <Allow>192.168.1.1</Allow>
            <Allow Role="Admin">192.168.2.1-255</Allow>
            <Allow Role="User">192.168.3.0/24</Allow>
            <Allow Role="Looker">192.168.4.0/255.255.255.0</Allow>
        </Manager>
    </Host>

-  ``<Name>``
    サーバー名を設定します。 名前が入力されていない場合はシステム名を使用します。

-  ``<Admin>``
    管理者情報（メールや名前）を設定します。 この項目はSNMP照会目的のみ使用されます。

-  ``<Manager>``
    管理の目的で使用マネージャーポートとACL（Access Control List）を設定します。 ACLはIPIP rangeBitMaskSubnetの四つの形式をサポートします。 接続したセッションがAllowに設定されたIPアドレスのみ許可します。 APIを呼び出すIPは ``<Allow>`` リストに必ず設定する必要があります。

    アクセス条件に合わせてアクセス権限（Role）を設定することができます。 アクセス権がない要求については **401 Unauthorized** で応答します。
    ``<Allow>`` の条件に ``Role`` 属性を明示的に宣言していない場合は ``<Manager>`` の ``Role`` 属性が適用されます。

    - ``Admin`` すべてのAPI呼び出しが可能です。
    - ``User`` api_monitoring、 :ref:`api-graph` APIのみを呼び出すことができます。
    - ``Looker`` :ref:`api-graph` APIのみを呼び出すことができます。

    その他次のような細かい管理目的の属性を持つ。

    - ``HttpMethod``

      - ``ON (デフォルト)``  :ref:`caching-purge-http-method` 呼び出し時ACLを点検します。

      - ``OFF`` :ref:`caching-purge-http-method` 呼び出し時ACLを検査しません。

    - ``UploadMultipartName`` :ref:`api-conf-upload` の変数名を設定します。


.. _env-cache-storage:

Storage構成
------------------------------------

Cachingされたコンテンツを保存するStorageを構成します。 ::

    # server.xml - <Server>

    <Cache>
        <Storage DiskFailSec="60" DiskFailCount="10" OnCrash="hang">
            <Disk>/user/cache1</Disk>
            <Disk>/user/cache2</Disk>
            <Disk Quota="100">/user/cache3</Disk>
        </Storage>
    </Cache>

-  ``<Storage>``
    コンテンツを保存するディスクを設定します。 サブ ``<Disk>`` 数の制限はありません。

    ディスクは障害が最も頻繁に発生する機器のため明確な障害条件を設定することをお勧めします。
    ``DiskFailSec (デフォルト: 60秒)`` の間 ``DiskFailCount (デフォルト: 10)`` だけディスクの操作が失敗した場合はそのディスクは自動的に排除されます。 排除されたディスクの状態は "Invalid"に指定されます。

    すべてのディスクが排除された場合の動作は ``OnCrash`` 属性に設定します。

    - ``hang (デフォルト)`` 障害ディスクの両方を再投入します。 通常のサービスを期待しているというよりは元のを保護する目的が強い。

    - ``bypass`` すべての要求をオリジンサーバーにバイパスします。 ディスクが復旧されるとすぐにSTONこのサービスを処理します。

    - ``selfkill`` STONを終了させる。

各ディスクごとに最大キャッシュ容量を ``Quota (単位: GB)`` 属性に設定することができます。 あえて設定しなくても常にディスクがいっぱいになってないようにLRU（Least Recently Used）アルゴリズムによって古いコンテンツを自動的に削除します。 特に互換性に問題があるファイルシステムはありません。 したがって管理者は使い慣れたファイルシステムを使用してもパフォーマンスに大きな影響はありません。

.. note::

   v2.5.0からディスクなしで動作する :ref:`adv_topics_memory_only` がサポートされます。



.. _env-cache-resource:

メモリの制限
------------------------------------

使用最大メモリとコンテンツの積載率を設定します。 ::

    # server.xml - <Server>

    <Cache>
        <SystemMemoryRatio>100</SystemMemoryRatio>
        <ContentMemoryRatio>50</ContentMemoryRatio>
    </Cache>

-  ``<SystemMemoryRatio> (デフォルト: 100%)``

   システムメモリ中STONが使用可能な最大メモリを割合を設定します。 たとえば16GB装置では数値を50（％）に設定するとシステムメモリが8GBであるように動作します。 特に :ref:`filesystem` などを介して他のプロセスと連動するときに便利です。

-  ``<ContentMemoryRatio> (デフォルト: 50%)``

   STONはディスクからロードされたBodyデータをメモリに最大限Cachingしサービスの品質を向上させる。 サービス形態に応じてこの割合を調節して品質を最適化します。

      .. figure:: img/bodyratio1.png
         :align: center

         ContentMemoryRatioを通じてメモリの割合を設定します。

   例えばゲームのダウンロードのようなファイルの数は多くないがContentsサイズが大きいサービスの場合File I / O負荷が負担になる。 このような場合 ``<ContentMemoryRatio>`` を高めより多くのContentsデータがメモリに常駐するように設定するとサービスの品質を向上させることができます。

      .. figure:: img/bodyratio2.png
         :align: center

         ContentMemoryRatioを上げるとI / Oが減少します。



.. _env-etc:

その他のCaching設定
------------------------------------

その他のCachingサービスの基盤動作を設定します。 ::

    # server.xml - <Server>

    <Cache>
        <Cleanup>
            <Time>02:00</Time>
            <Age>0</Age>
            <EmptyFolder>delete</EmptyFolder>
        </Cleanup>
        <Listen>0.0.0.0</Listen>
        <ConfigHistory>30</ConfigHistory>
    </Cache>

-  ``<Cleanup>``
    一日に一回システムの最適化を実行します。 最適化のほとんどはディスクのクリーンアップ操作でI / O負荷が発生します。 サービス品質の低下を最低限にするために最適化は少しずつ段階的に実行されます。

    - ``<Time> (デフォルト: AM 2)`` Cleanup実行時間を設定します。 午後11時10分を設定したい場合は23:10に設定します。

    - ``<Age> (デフォルト: 0, 単位: 日)`` 0よりも大きい場合には一定の期間一度もアクセスされていないコンテンツを削除します。 ディスクを事前に確保してサービス時間中のディスク不足が発生する確率を低減するためです。

    - ``<EmptyFolder> (デフォルト: delete)`` Cleanup時点で空のフォルダ（キャッシュ保存用に使用）の削除有無を決定します。
      ``delete`` の場合は削除し ``keep`` の場合は削除しません。

-  ``<Listen>``
    すべての仮想ホストがListenするIPのリストを指定します。 すべての仮想ホストのデフォルトListen設定の *：80は0.0.0.0:80を意味します。 指定されたIPだけ開きたい場合は次のように明確に設定します。 ::

       # server.xml - <Server>

       <Cache>
         <Listen>10.10.10.10</Listen>
         <Listen>10.10.10.11</Listen>
         <Listen>127.0.0.2</Listen>
       </Cache>

-  ``<ConfigHistory> (デフォルト: 30日)``
    STONは設定が変更されるたびにすべての設定をバックアップします。 圧縮後./conf/に1つのファイルとして保存されます。 ファイル名は "日付_時刻_HASH.tgz" に生成されます。 ::

       20130910_174843_D62CA26F16FE7C66F81D215D8C52266AB70AA5C8.tgz

    すべての設定が完全に同じであれば同じHASH値を持つ。
    :ref:`api-conf-restore` が呼び出される場合も新しい設定として保存されます。 バックアップされた設定はCleanup時間を基準に設定された日分だけ保存されます。 設定ファイルの保存の日の制限はありません。



強制Cleanup
------------------------------------

API呼び出しにCleanupします。 <Age> をパラメータとして入力することができます。 ::

   http://127.0.0.1:10040/command/cleanup
   http://127.0.0.1:10040/command/cleanup?age=10

``<Age>`` が0であればディスクの空き容量が足りないと判断した場合にのみCleanupを実行します。
``<Age>`` パラメータが0よりも大きい場合はその「日」に一度もアクセスされていないコンテンツを削除します。


.. _env-vhostdefault:

仮想ホストの設定
------------------------------------

管理者はそれぞれの仮想ホストを個別に設定することができます。 しかし仮想ホストを作成するたびに同じ設定を繰り返すことは非常に消耗です。 すべての仮想ホストは ``<VHostDefault>`` を継承します。

   .. figure:: img/vhostdefault.png
      :align: center

      単一継承です。

www.example.comの場合は別途上書き（Overriding）した値がないためA = 1B = 2となる。 一方img.example.comはB = 3で上書きしたのでA = 1B = 3となる。 管理者は通常同じサービスの特性を持つサービスをしたサーバーにように構成します。 したがって継承は非常に効果的な方法です。

``<VHostDefault>`` は機能別で囲まれた5つのサブタグを持つ。 ::

    # server.xml - <Server>

    <VHostDefault>
        <Options> ... </Options>
        <OriginOptions> ... </OriginOptions>
        <Media> ... </Media>
        <Stats> ... </Stats>
        <Log> ... </Log>
    </VHostDefault>

例えば :ref:`media` 機能は ``<Media>`` サブに設定する式です。


.. _env-vhost:

vhosts.xml仮想ホストの設定
====================================

実行可能ファイルと同じパスに存在するvhosts.xmlファイルを仮想ホストの設定ファイルとして認識します。 仮想ホストの数に制限はありません。 ::

    # vhosts.xml

    <Vhosts>
        <Vhost Status="Active" Name="www.example.com"> ... </Vhost>
        <Vhost Status="Active" Name="img.example.com"> ... </Vhost>
        <Vhost Status="Active" Name="vod.example.com"> ... </Vhost>
    </Vhosts>


.. _env-vhost-create-destroy:

生成/破壊
------------------------------------

``<Vhosts>`` サブ ``<Vhost>`` で仮想ホストを設定します。 ::

    # vhosts.xml - <Vhosts>

    <Vhost Status="Active" Name="www.example.com">
        <Origin>
            <Address>10.10.10.10</Address>
        </Origin>
    </Vhost>

-  ``<Vhost>`` バーチャルホストを設定します。

   - ``Status (デフォルト: Active)`` Inactiveである場合はその仮想ホストはサービスしません。 キャッシュされたコンテンツは維持されます。
   - ``Name`` 仮想ホスト名。 重複することがない。

``<Vhost>`` を削除するとその仮想ホストが削除されます。 削除された仮想ホストのすべてのコンテンツは削除対象となる。 再度追加してもコンテンツは甦るない。


.. _env-vhost-find:

検索
------------------------------------

以下は最も単純な形式のHTTPリクエストです。 ::

    GET / HTTP/1.1
    Host: www.example.com

一般的なWebサーバはHostヘッダに仮想ホストを探す。 一つの仮想ホストを複数の名前でサービスしたい場合は ``<Alias>`` を使用します。 ::

    # vhosts.xml - <Vhosts>

    <Vhost Name="example.com">
        <Alias>another.com</Alias>
        <Alias>*.sub.example.com</Alias>
    </Vhost>

-  ``<Alias>``

   仮想ホストの別名を設定します。 数は制限がない。 明確な表現（another.com）とパターン表現(*.sub.example.com)をサポートします。 パターンは複雑な正規表現ではなくprefixに*表現を1つだけ付けることができる簡単な形式のみをサポートします。


仮想ホストの検索順序は次のとおりです。

1. ``<Vhost>`` の ``Name`` と一致するか？
2. 明示的な ``<Alias>`` と一致するか？
3. パターン ``<Alias>`` を満足するか？


.. _env-vhost-defaultvhost:

Default仮想ホスト
------------------------------------

要求を処理する仮想ホストが見つからなかった場合は選択される仮想ホストを指定することができます。 リクエストを処理したくない場合は設定していてもよい。 ::

    # vhosts.xml

    <Vhosts>
        <Vhost Status="Active" Name="www.example.com"> ... </Vhost>
        <Vhost Status="Active" Name="img.example.com"> ... </Vhost>
        <Default>www.example.com</Default>
    </Vhosts>

-  ``<Default>``

   デフォルトの仮想ホスト名を設定します。 必ず ``<Vhost>`` の ``Name`` 属性と同じ文字列に設定する必要があります。


.. _env-vhost-listen:

サービスアドレス
------------------------------------
サービスのアドレスを設定します。 ::

    # vhosts.xml - <Vhosts>

    <Vhost Name="www.example.com">
        <Listen>*:80</Listen>
    </Vhost>

-  ``<Listen> (デフォルト: *:80)``

   {IP}：{Port}の形式でサービスのアドレスを設定します。
   *:80 表現はすべてのNICからの80ポートに来る要求を処理するという意味だ。 たとえば特定のIP（1.1.1.1）の90ポートにサービスしたい場合は次のように設定します。 ::

       # vhosts.xml - <Vhosts>

       <Vhost Name="www.example.com">
           <Listen>1.1.1.1:90</Listen>
       </Vhost>

.. note:

   サービスポートを開かないようにするに ``OFF``に設定する。 ::

      # vhosts.xml - <Vhosts>

      <Vhost Name="www.example.com">
         <Listen>OFF</Listen>
      </Vhost>


.. _env-vhost-txt:

仮想ホスト-例外条件 (.txt)
---------------------------------------

サービスの次のように例外的な状況が必要な時があります。

- すべてのPOSTリクエストは許可していませんが特定のURLへのPOSTリクエストは許可します。
- すべてのGETリクエストはSTONが応答するが特定のIP帯域についてはオリジンサーバーにバイパスします。
- 特定の国には伝送速度を制限します。

このような例外条件はXMLに設定しません。 すべての仮想ホストは独立した例外条件を有します。 例外条件は./svc/仮想ホスト/ディレクトリの下位にTXTに存在します。 関連の機能について説明すると例外条件も一緒に扱う。


仮想ホストのリストを確認
====================================

仮想ホストのリストを照会します。 ::

   http://127.0.0.1:10040/monitoring/vhostslist

結果はJSON形式で提供されます。 ::

   {
      "version": "1.1.9",
      "method": "vhostslist",
      "status": "OK",
      "result": [ "www.example.com","www.foobar.com", "site1.com" ]
   }


.. _api-conf-show:

設定を確認する
====================================

サービス中の設定ファイルを確認します。 txtファイルは仮想ホスト（vhost）を明確に指定する必要があります。 ::

    http://127.0.0.1:10040/conf/server.xml
    http://127.0.0.1:10040/conf/vhosts.xml
    http://127.0.0.1:10040/conf/querystring.txt?vhost=www.example.com
    http://127.0.0.1:10040/conf/bypass.txt?vhost=www.example.com
    http://127.0.0.1:10040/conf/ttl.txt?vhost=www.example.com
    http://127.0.0.1:10040/conf/expires.txt?vhost=www.example.com
    http://127.0.0.1:10040/conf/acl.txt?vhost=www.example.com
    http://127.0.0.1:10040/conf/headers.txt?vhost=www.example.com
    http://127.0.0.1:10040/conf/throttling.txt?vhost=www.example.com
    http://127.0.0.1:10040/conf/postbody.txt?vhost=www.example.com


.. _api-conf-history:

設定のリスト
====================================

バックアップされた設定のリストを閲覧します。 ::

    http://127.0.0.1:10040/conf/latest
    http://127.0.0.1:10040/conf/history

結果はJSON形式で提供されます。 高速最後の設定状態のみを確認したい場合は/ conf / latestを使用することを推奨します。 ::

    {
        "history" :
        [
            {
                "id" : "5",
                "conf-date" : "2013-11-06",
                "conf-time" : "15:26:37",
                "type" : "loaded",
                "size" : "16368",
                "hash" : "D62CA26F16FE7C66F81D215D8C52266AB70AA5C8",
                "ver": "1.2.8"
            },
            {
                "id" : "6",
                "conf-date" : "2013-11-07",
                "conf-time" : "07:02:21",
                "type" : "modified",
                "size" : "27544",
                "hash" : "F81D215D8C52266AB70AA5C8D62CA26F16FE7C66",
                "ver": "1.2.8"
            }
        ]
    }

-  ``id`` 設定の一意のID（Reloadするたびに +1)
-  ``conf-date`` の設定変更日
-  ``conf-time`` の設定を変更時間
-  ``type`` の設定が反映された形
   - ``loaded`` STONが開始される
   - ``modified`` 設定が（管理者またはWMによって）変更されたとき
   - ``uploaded`` 設定ファイルAPIを介してアップロードされたとき
   - ``restored`` 設定ファイルがAPIを介して復旧されたとき
-  ``size`` 設定ファイルサイズ
-  ``hash`` 設定ファイルをSHA-1でhash値


.. _api-conf-restore:

設定の復元
====================================

hash値またはidに基づいて任意の時点の設定に戻す。 hashとidの両方が記載されている場合hash値が優先されます。 正常Rollbackされた場合200 OK失敗した場合500 Internal Errorで応答します。 ::

    http://127.0.0.1:10040/conf/restore?hash=...
    http://127.0.0.1:10040/conf/restore?id=...


.. _api-conf-download:

設定のダウンロード
====================================

hash値またはidに基づいて任意の時点の設定をダウンロードします。 Content-Typeは "application / x-compressed" に指定されます。 hashとidの両方が記載されている場合hash値が優先されその時点の設定が存在しない場合404 NOT FOUNDに応答します。 ::

    http://127.0.0.1:10040/conf/download?hash=...
    http://127.0.0.1:10040/conf/download?id=...



.. _api-conf-upload:

設定アップロード
====================================

APIを利用して設定を変更します。 正常反映場合200 OKで応答するが失敗した場合500 Internal Server Errorで応答します。 ::

   {
      "version": "2.5.10",
      "method": "uploadconfig",
      "status": "Fail",
      "result": "E0001"
    }

"result" エラーコードは次のとおりです。

======= ===============================
result  説明
======= ===============================
E0000   設定を適用完了
E0001   アップロードしたファイルが存在しません。
E0002   tgzファイルを圧縮解除に失敗しました。
E0003   アップロードされたxmlファイルがロードされない。
E0004   アップロードされたxmlファイルの内容が間違っていた。
======= ===============================


.. _api-conf-upload-tgz:

全体の設定アップロード
------------------------------------

全体の設定圧縮ファイルをHTTP Post方式（Multipartサポート）にアップロードします。 ::

    http://127.0.0.1:10040/conf/upload

次のように住所 Content-Length, Content-Type(="multipart/form-data")が明確に宣言されてなければならない。 ::

    POST /conf/upload
    Content-Length: 16455
    Content-Type: multipart/form-data; boundary=......

アップロードが完了すると圧縮を解除した後全体の設定を更新します。

Multipart方式では "confile"をデフォルト名として使用します。 この値は、 ``<Manager>`` の ``UploadMultipartName`` 属性で設定することができます。 ::

    <form enctype="multipart/form-data" action="http://127.0.0.1:10040/conf/upload" method="POST">
        <input name="confile" type="file" />
        <input type="submit" value="Upload" />
    </form>


.. _api-conf-upload-xml:

XML設定アップロード
------------------------------------

XML個々の設定圧縮ファイルをHTTP Post方式（MultipartとSOAP方式の両方をサポート）にアップロードします。 ::

    http://127.0.0.1:10040/conf/upload/server.xml
    http://127.0.0.1:10040/conf/upload/vhosts.xml


.. note::
   
   server.xmlをアップロードする場合は全体の設定を更新しますがvhosts.xmlのみアップロードする場合は仮想ホストにのみ設定を更新します。

