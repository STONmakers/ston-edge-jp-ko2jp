.. _env:

第3章 設定構造
******************

.. note::

   - `[動画講座]みよう！ STON Edge Server - Chapter 2.を設定する <https://youtu.be/BROcSuFyHOQ?list=PLqvIfHb2IlKeZ-Eym_UPsp6hbpeF-a2gE>`_
   - `[動画講座]みよう！ STON Edge Server - Chapter 3.仮想ホスト作成 <https://youtu.be/AvxxSWgXcqA?list=PLqvIfHb2IlKeZ-Eym_UPsp6hbpeF-a2gE>`_

この章では、設定の構造と変更された設定を適用する方法について説明する。 構造を正確に理解する必要が迅速にサーバーを配置することができるだけでなく、障害の状況を柔軟に克服することができる。

設定は大きく全域（server.xml）と仮想ホスト（vhosts.xml）に分けられる。

   .. figure:: img/conf_files.png
      :align: center

      2つの.xmlファイルがすべてです。

2つのXMLファイルで、ほとんどのサービスを構成する。 複数TXTファイルには、仮想ホスト固有の例外条件を設定するが、特定の機能のリストを作成するのに使用される。 機能説明のために、以下のように完全な形のXMLを例示することとても面倒である。 ::

   <Server>
       <VHostDefault>
           <Options>
               <CaseSensitive>ON</CaseSensitive>
           </Options>
       </VHostDefault>
   </Server>

ため、次のように省略して説明する。 ::

   # server.xml - <Server><VHostDefault><Options>

   <CaseSensitive>ON</CaseSensitive>


.. note::

   ライセンス（license.xml）は設定がありません。


.. _api-conf-reload:

設定Reload
====================================

設定の変更後、管理者が明確にAPIを呼び出す必要があります。 システムパフォーマンス関連の設定を除き、ほとんどの設定は、サービスを中断せず、すぐに適用される。 ::

   http://127.0.0.1:10040/conf/reload

設定が変更されるたびに :ref:`admin-log-info` に変更が記録される。


server.xmlグローバル設定
====================================

実行可能ファイルと同じパスに存在するserver.xmlこのグローバル設定ファイルである。 XML形式のテキストファイルである。 ::

    # server.xml

    <Server>
        <Host> ... </Host>
        <Cache> ... </Cache>
        <VHostDefault> ... </VHostDefault>
    </Server>

まずグローバル設定の構造と簡単な機能を中心に説明する。
:ref:`access-control` や :ref:`snmp` などのグローバル設定に位置が図体の大きい機能については、各トピックをカバーする章で説明する。

.. toctree::
   :maxdepth: 2


.. _env-host:

管理者の設定
------------------------------------

管理目的の機能を設定する。 ::

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
    サーバー名を設定する。 名前が入力されていない場合、システム名を使用する。

-  ``<Admin>``
    管理者情報（メールや名前）を設定する。 この項目は、SNMP照会目的でのみ使用される。

-  ``<Manager>``
    管理の目的で使用マネージャーポートとACL（Access Control List）を設定する。 ACLは、IP、IPアドレスの範囲、BitMask、Subnet以上四つの形式をサポートします。 接続したセッションがAllowにアクセスが許可されたIPアドレスがなければ接続を遮断する。 APIを呼び出すIPが ``<Allow>`` リストに必ず設定する必要がある。

    アクセス条件に応じて、アクセス権限（Role）を設定することができる。 アクセス権がない要求については、 **401 Unauthorized** で応答する。
    ``<Allow>`` の条件に ``Role`` 属性を明示的に宣言していない場合は ``<Manager>`` の ``Role`` 属性が適用される。

    - ``Admin`` すべてのAPI呼び出しが可能である。
    - ``User`` api_monitoring、 :ref:`api-graph` APIのみを呼び出すことができる。
    - ``Looker`` :ref:`api-graph` APIのみを呼び出すことができる。

    その他、次のような細かい管理の目的の属性を持つ。

    - ``HttpMethod``

      - ``ON (基本)`` api-etc-httpmethod呼び出し時ACLを点検する。

      - ``OFF`` api-etc-httpmethod呼び出し時ACLを検査していない。

    - ``UploadMultipartName`` :ref:`api-conf-upload` の変数名を設定する。


.. _env-cache-storage:

Storage構成
------------------------------------

Cachingされたコンテンツを格納するStorageを構成する。 ::

    # server.xml - <Server>

    <Cache>
        <Storage DiskFailSec="60" DiskFailCount="10" OnCrash="hang">
            <Disk>/user/cache1</Disk>
            <Disk>/user/cache2</Disk>
            <Disk Quota="100">/user/cache3</Disk>
        </Storage>
    </Cache>

-  ``<Storage>``
    コンテンツを格納するディスクを設定する。 サブ ``<Disk>`` 数の制限はない。

    ディスクは、障害が最も多く発生する機器であるため、明確な障害条件を設定することをお勧めします。
    ``DiskFailSec (基本: 60秒)`` の間 ``DiskFailCount (基本: 10)`` だけディスクの操作が失敗した場合、そのディスクは自動的に排除される。 排除されたディスクの状態は "Invalid"に指定される。

    すべてのディスクが排除されることもあり、このときの動作は、 ``OnCrash`` 属性に設定する。.

    - ``hang (基本)`` 障害ディスクの両方を再投入する。 通常のサービスを期待しているというよりは、元のを保護する目的が強い。

    - ``bypass`` すべての要求を元のサーバーにバイパスする。 ディスクが回復されると、すぐにSTONこのサービスを処理する。

    - ``selfkill`` STONを終了させる。

各ディスクごとに最大キャッシュ容量を ``Quota (単位: GB)`` 属性に設定することができる。 あえて設定しなくても、常にディスクがいっぱいになってないようにLRU（Least Recently Used）アルゴリズムによって、古いコンテンツを自動的に削除する。 特に互換性に問題があるファイルシステムはない。 したがって、管理者は、使い慣れたファイルシステムを使用しても、パフォーマンスに大きな影響はない。

.. note::

   v2.5.0からディスクなしで動作する :ref:`adv_topics_memory_only` がサポートされる。



.. _env-cache-resource:

メモリの制限
------------------------------------

使用最大メモリとコンテンツ積載率を設定する。 ::

    # server.xml - <Server>

    <Cache>
        <SystemMemoryRatio>100</SystemMemoryRatio>
        <ContentMemoryRatio>50</ContentMemoryRatio>
    </Cache>

-  ``<SystemMemoryRatio> (基本: 100%)``

   システムメモリからSTONが使用最大メモリを割合に設定する。 たとえば16GB装置では、数値を50（％）に設定すると、システムメモリが8GBであるかのように動作する。 特に :ref:`filesystem` などを介して他のプロセスと連動するときに便利である。

-  ``<ContentMemoryRatio> (基本: 50%)``

   STONは、ディスクからロードされたBodyデータをメモリに最大限Cachingし、サービスの品質を向上させる。 サービス形態に応じて、この割合を調節して、品質を最適化する。

      .. figure:: img/bodyratio1.png
         :align: center

         ContentMemoryRatioを通じてメモリの割合を設定する。

   例えば、ゲームのダウンロードのようなファイルの数は多くないが、Contentsサイズが大きいサービスの場合File I / O負荷が負担になる。 このような場合、 ``<ContentMemoryRatio>`` を高め、より多くのContentsデータがメモリに常駐するように設定すると、サービスの品質を向上させることができる。

      .. figure:: img/bodyratio2.png
         :align: center

         ContentMemoryRatioを上げると、I / Oが減少する。



.. _env-etc:

その他のCaching設定
------------------------------------

その他のCachingサービスの基盤動作を設定する。 ::

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
    一日に一回システムの最適化を実行する。 最適化のほとんどは、ディスクのクリーンアップ操作で、I / O負荷が発生する。 サービス品質の低下を防止するために最適化は、少しずつ段階的に実行される。

    - ``<Time> (基本: AM 2)`` Cleanup実行時間を設定する。 午後11時10分を設定したい場合は23:10に設定する。

    - ``<Age> (基本: 0, 単位: 日)`` 0よりも大きい場合には、一定の期間一度もアクセスされていないコンテンツを削除する。 ディスクを事前に確保してサービス時間中のディスク不足が発生する確率を低減するためである。

    - ``<EmptyFolder> (基本: delete)`` Cleanup時点で空のフォルダ（キャッシュ保存用に使用）の削除するかどうかを決定する。
      ``delete`` の場合は、削除し、 ``keep`` の場合は、削除していない。

-  ``<Listen>``
    すべての仮想ホストがListenするIPのリストを指定する。 すべての仮想ホストの基本Listen設定の* ：80は、0.0.0.0:80を意味する。 指定されたIPだけ開きたい場合は、次のように明確に設定する。 ::

       # server.xml - <Server>

       <Cache>
         <Listen>10.10.10.10</Listen>
         <Listen>10.10.10.11</Listen>
         <Listen>127.0.0.2</Listen>
       </Cache>

-  ``<ConfigHistory> (基本: 30日)``
    STONは設定が変更されるたびに、すべての設定をバックアップする。 解凍後./conf/に1つのファイルとして保存される。 ファイル名は、「日付_時刻_HASH.tgz "に生成される。 ::

       20130910_174843_D62CA26F16FE7C66F81D215D8C52266AB70AA5C8.tgz

    すべての設定が完全に同じであれば同じHASH値を持つ。
    :ref:`api-conf-restore` が呼び出されるよう、新しい設定として保存されます。 バックアップされた設定は、Cleanup時間を基準に設定された日分だけ保存される。 設定ファイルの保存の日の制限はない。



強制Cleanup
------------------------------------

API呼び出しにCleanupする。 ``<Age>`` をパラメータとして入力することができる。 ::

   http://127.0.0.1:10040/command/cleanup
   http://127.0.0.1:10040/command/cleanup?age=10

``<Age>`` が0であれば、ディスクの空き容量が足りないと判断した場合にのみ、Cleanupを実行する。
``<Age>`` パラメータが0よりも大きい場合は、その「仕事」の間一度もアクセスされていないコンテンツを削除する。


.. _env-vhostdefault:

仮想ホストの設定
------------------------------------

管理者は、それぞれの仮想ホストを個別に設定することができる。 しかし、仮想ホストを作成するたびに同じ設定を繰り返すことは非常に消耗である。 すべての仮想ホストは ``<VHostDefault>`` を継承します。

   .. figure:: img/vhostdefault.png
      :align: center

      単一継承である。

www.example.comの場合は、別途上書き（Overriding）した値がないため、A = 1、B = 2となる。 一方、img.example.comはB = 3で上書きしたのでA = 1、B = 3となる。 管理者は、通常、同じサービスの特性を持つサービスをしたサーバーにように構成する。 したがって、継承は非常に効果的な方法である。

``<VHostDefault>`` は機能別で囲まれた5つのサブタグを持つ。 ::

    # server.xml - <Server>

    <VHostDefault>
        <Options> ... </Options>
        <OriginOptions> ... </OriginOptions>
        <Media> ... </Media>
        <Stats> ... </Stats>
        <Log> ... </Log>
    </VHostDefault>

例えば :ref:`media` 機能は、 ``<Media>`` サブに設定する式である。


.. _env-vhost:

vhosts.xml仮想ホストの設定
====================================

実行可能ファイルと同じパスに存在するvhosts.xmlファイルを仮想ホストの設定ファイルとして認識します。 仮想ホストの数に制限はない。 ::

    # vhosts.xml

    <Vhosts>
        <Vhost Status="Active" Name="www.example.com"> ... </Vhost>
        <Vhost Status="Active" Name="img.example.com"> ... </Vhost>
        <Vhost Status="Active" Name="vod.example.com"> ... </Vhost>
    </Vhosts>


.. _env-vhost-create-destroy:

生成/破壊
------------------------------------

``<Vhosts>`` サブ ``<Vhost>`` で仮想ホストを設定する。 ::

    # vhosts.xml - <Vhosts>

    <Vhost Status="Active" Name="www.example.com">
        <Origin>
            <Address>10.10.10.10</Address>
        </Origin>
    </Vhost>

-  ``<Vhost>`` バーチャルホストを設定する。

   - ``Status (基本: Active)`` nactiveである場合は、その仮想ホストをサービスしていない。 キャッシュされたコンテンツは、維持される。
   - ``Name`` 仮想ホスト名。 重複することがない。

``<Vhost>`` を削除すると、その仮想ホストが削除される。 削除された仮想ホストのすべてのコンテンツは、削除対象となる。 再度追加しても、コンテンツは甦るない。


.. _env-vhost-find:

検索
------------------------------------

以下は、最も単純な形式のHTTPリクエストである。 ::

    GET / HTTP/1.1
    Host: www.example.com

一般的なWebサーバは、Hostヘッダに仮想ホストを探す。 一つの仮想ホストを複数の名前でサービスしたい場合は ``<Alias>`` を使用する。 ::

    # vhosts.xml - <Vhosts>

    <Vhost Name="example.com">
        <Alias>another.com</Alias>
        <Alias>*.sub.example.com</Alias>
    </Vhost>

-  ``<Alias>``

   仮想ホストの別名を設定する。 数は制限がない。 明確な表現（another.com）とパターン表現(*.sub.example.com)をサポートする。 パターンは、複雑な正規表現ではなく、prefixに*表現を1つだけ付けることができる簡単な形式のみをサポートしている。


仮想ホストの検索順序は次のとおりである。

1. ``<Vhost>`` の ``Name`` と一致するか？
2. 明示的な ``<Alias>`` と一致するか？
3. パターン ``<Alias>`` を満足するか？


.. _env-vhost-defaultvhost:

Default仮想ホスト
------------------------------------

要求を処理する仮想ホストが見つからなかった場合は、選択される仮想ホストを指定することができる。 リクエストを処理したくない場合は設定していてもよい。 ::

    # vhosts.xml

    <Vhosts>
        <Vhost Status="Active" Name="www.example.com"> ... </Vhost>
        <Vhost Status="Active" Name="img.example.com"> ... </Vhost>
        <Default>www.example.com</Default>
    </Vhosts>

-  ``<Default>``

   デフォルトの仮想ホスト名を設定する。 必ず ``<Vhost>`` の ``Name`` 属性と同じ文字列に設定する必要がある。


.. _env-vhost-listen:

サービスアドレス
------------------------------------
サービスのアドレスを設定する。 ::

    # vhosts.xml - <Vhosts>

    <Vhost Name="www.example.com">
        <Listen>*:80</Listen>
    </Vhost>

-  ``<Listen> (基本: *:80)``

   {IP}：{Port}の形式でサービスのアドレスを設定する。
   *:80 表現はすべてのNICからの80ポートに来る要求を処理するという意味だ。 たとえば、特定のIP（1.1.1.1）の90ポートにサービスしたい場合は、次のように設定する。 ::

       # vhosts.xml - <Vhosts>

       <Vhost Name="www.example.com">
           <Listen>1.1.1.1:90</Listen>
       </Vhost>

.. note:

   서비스 포트를 열지 않으려면 ``OFF`` 로 설정한다. ::

      # vhosts.xml - <Vhosts>

      <Vhost Name="www.example.com">
         <Listen>OFF</Listen>
      </Vhost>


.. _env-vhost-txt:

仮想ホスト-例外条件 (.txt)
---------------------------------------

サービスの次のように例外的な状況が必要な時がある。

- すべてのPOSTリクエストは許可していませんが、特定のURLへのPOSTリクエストは許可する。
- すべてのGETリクエストはSTONが応答するが、特定のIP帯域については、元のサーバーにバイパスする。
- 特定の国には、伝送速度を制限する。

このような例外条件は、XMLに設定していない。 すべての仮想ホストは、独立した例外条件を有する。 例外条件は./svc/仮想ホスト/ディレクトリの下位にTXTに存在する。 関連の機能について説明すると、例外条件も一緒に扱う。


仮想ホストのリストを確認
====================================

仮想ホストのリストを照会する。 ::

   http://127.0.0.1:10040/monitoring/vhostslist

結果は、JSON形式で提供される。 ::

   {
      "version": "1.1.9",
      "method": "vhostslist",
      "status": "OK",
      "result": [ "www.example.com","www.foobar.com", "site1.com" ]
   }


.. _api-conf-show:

設定を確認する
====================================

サービス中の設定ファイルを確認する。 txtファイルは、仮想ホスト（vhost）を明確に指定する必要がある。 ::

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

バックアップされた設定のリストを閲覧する。 ::

    http://127.0.0.1:10040/conf/latest
    http://127.0.0.1:10040/conf/history

結果は、JSON形式で提供される。 高速最後の設定状態のみを確認したい場合は、/ conf / latestを使用することを推奨する。 ::

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
   - ``restored`` 設定ファイルがAPIを介して回復されたとき
-  ``size`` 設定ファイルサイズ
-  ``hash`` 設定ファイルをSHA-1でhash値


.. _api-conf-restore:

設定の復元
====================================

hash値またはidに基づいて任意の時点の設定に戻す。 hashとidの両方が記載されている場合hash値が優先される。 正常Rollbackされた場合、200 OK、失敗した場合、500 Internal Errorで応答する。 ::

    http://127.0.0.1:10040/conf/restore?hash=...
    http://127.0.0.1:10040/conf/restore?id=...


.. _api-conf-download:

設定のダウンロード
====================================

hash値またはidに基づいて任意の時点の設定をダウンロードする。 Content-Typeは "application / x-compressed"に指定される。 hashとidの両方が記載されている場合hash値が優先され、その時点の設定が存在しない場合、404 NOT FOUNDに応答する。 ::

    http://127.0.0.1:10040/conf/download?hash=...
    http://127.0.0.1:10040/conf/download?id=...



.. _api-conf-upload:

設定アップロード
====================================

APIを利用して設定を変更する。 正常反映場合、200 OKで応答するが、失敗した場合、500 Internal Server Errorで応答する。 ::

   {
      "version": "2.5.10",
      "method": "uploadconfig",
      "status": "Fail",
      "result": "E0001"
    }

"result" エラーコードは、次のとおりである。

======= ===============================
result  説明
======= ===============================
E0000   設定を適用完了
E0001   アップロードしたファイルが存在しない。
E0002   tgzファイルを圧縮解除に失敗しました。
E0003   アップロードされたxmlファイルがロードされない。
E0004   アップロードされたxmlファイルの内容が間違っていた。
======= ===============================


.. _api-conf-upload-tgz:

全体の設定アップロード
------------------------------------

全体の設定圧縮ファイルをHTTP Post方式（Multipartサポート）にアップロードする。 ::

    http://127.0.0.1:10040/conf/upload

次のように住所、 Content-Length, Content-Type(="multipart/form-data")が明確に宣言されてなければならない。 ::

    POST /conf/upload
    Content-Length: 16455
    Content-Type: multipart/form-data; boundary=......

アップロードが完了すると、圧縮を解除した後、全体の設定を更新する。

Multipart方式では、 "confile"を基本名として使用する。 この値は、 ``<Manager>`` の ``UploadMultipartName`` 属性で設定することができる。 ::

    <form enctype="multipart/form-data" action="http://127.0.0.1:10040/conf/upload" method="POST">
        <input name="confile" type="file" />
        <input type="submit" value="Upload" />
    </form>


.. _api-conf-upload-xml:

XML設定アップロード
------------------------------------

XML個々の設定圧縮ファイルをHTTP Post方式（MultipartとSOAP方式の両方をサポート）にアップロードする。 ::

    http://127.0.0.1:10040/conf/upload/server.xml
    http://127.0.0.1:10040/conf/upload/vhosts.xml


.. note::
   
   server.xmlをアップロードする場合は、全体の設定を更新しますが、vhosts.xmlのみアップロードする場合は、仮想ホストにのみ設定を更新する。

