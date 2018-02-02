.. _caching-purge:

第5章 Caching無効
******************

この章では、Cachingされたコンテンツを無効にする方法について説明する。 業界用語でPurgeで通称が、様々な状況や環境により、細分化されたAPIが必要である。

ソースからキャッシュされたコンテンツは、 :ref:`caching-policy-ttl` に基づい更新周期を持つ。 しかし、明らかに内容が変更され、管理者がこれをすぐに反映したい場合 :ref:`caching-policy-ttl` が期限切れになるまで待つ必要はない。
`Purge`_ / `Expire`_ / `HardPurge`_ などを使用すると、すぐにコンテンツを無効化させることができる。

無効化APIは、単にブラウザによって呼び出される場合もあるが、自動化されている場合が多い。 例えばFTP経由でファイルのアップロードが完了したら、すぐに `Purge`_ を呼び出す式である。 管理者は、次のようにいくつかの動作に設定することができる。 ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <Purge2Expire>NONE</Purge2Expire>
   <RootPurgeExpire>ON</RootPurgeExpire>
   <ResCodeNoCtrlTarget>200</ResCodeNoCtrlTarget>

-  ``<Purge2Expire> (基本: NONE)``

   `Purge`_ 要求を設定に応じて、 `Expire`_ で処理する。 たとえば、特定のパターン(*.jpg)を `Purge`_ する場合、意図せず、多くのコンテンツが削除され、ソースに過度な負荷を発生させることができる。 このような場合、 `Expire`_ に処理するように設定すると、過剰なソース負荷を防止することができる。
   
   - ``NONE`` `Expire`_ で処理しない。
   - ``ROOT`` ドメイン全体（/ * ）の `Purge`_ を `Expire`_ で処理する。
   - ``PATTERN`` すべてのパターン `Purge`_ を `Expire`_ で処理する。
   - ``ALL`` すべての `Purge`_ を `Expire`_ で処理する。

-  ``<RootPurgeExpire> (基本: ON)``
   
   全体のコンテンツへの意図しない `Purge`_ / `Expire`_ は、過度の元のサーバーの負荷を発生させることができる。 この設定を介してすべてのコンテンツの `Purge`_ / `Expire`_ を遮断することができる。 この設定は、 ``<Purge2Expire>`` よりも優先します。
   
   - ``ON`` `Purge`_ / `Expire`_ を可能にする。
   - ``PURGE`` `Purge`_ のみ可能にする。
   - ``EXPIRE`` `Expire`_ のみ可能にする。
   - ``OFF`` 모든 `Purge`_ / `Expire`_ を禁止する。

-  ``<ResCodeNoCtrlTarget> (基本: 200)``

   `Purge`_ , `Expire`_ , `HardPurge`_ , `ExpireAfter`_ の対象オブジェクトが存在しないときのHTTP応答コードを設定する。
   

ターゲット設定は、URLは、パターン2つに表現する。 ::

   example.com/logo.jpg      // URL
   example.com/img/          // URL
   example.com/img/*.jpg     // パターン
   example.com/img/*         // パターン
   
明確なURLのほかのパターン(*.jpg)で無効化が可能である。 しかし、作業を実行するまで、ターゲット数を明確に知ることができない。 これは、ややもすると、管理者の意図とは異なるので、多くの対象を指定することができる。 これは、実際にCPUリソースをあまり消費することになり、システム全体に負担を与えることができる。
   
したがって、実サービスの中には明確なURLだけを使用することを強く推奨する。 パターン表現はサービスから排除された状態で、管理目的で使用するためである。


.. note::

   セキュリティ上の理由からexample.com/files/などの特定のディレクトリへのアクセスは、403 FORBIDDENなどで遮断される。 しかし、ルートディレクトリは、例外を持つ。 たとえば、ユーザーがexample.comにアクセスすると、ブラウザは、ルートディレクトリ（/）を要請する。 ::
   
      GET / HTTP/1.1
      Host: example.com
   
   これに対して、Webサーバーは、管理者が設定したデフォルトのページ（おそらくindex.htmlまたはindex.htm）で応答する。 明らかに、Webサービスの構成では、ルートディレクトリ（/）は、ディレクトリではなくページで動作する。
   
   しかし、Cacheサーバーは、ルートディレクトリ（/）にアクセスしたところ、200 OKのページが来た理解する。 さらに、元のサーバーがどのページを応答した知らない。 簡単に整理すると、Cacheサーバーの観点では、ディレクトリ表現もURLの一種に過ぎない。 ::
   
      example.com/img/          // example.com 仮想ホストの / img / に アクセスした 結果 のページ
      example.com/              // example.com の仮想ホストの 基本 ページ （/）
      example.com/img/*         // example.com 仮想ホストの / img ディレクトリと その サブ ページ
      example.com/*             // example.com 仮想ホストの すべての コンテンツ
         


.. toctree::
   :maxdepth: 2


.. _api-cmd-purge:

Purge
====================================

ターゲットコンテンツを無効にさせて、元のサーバーからコンテンツを再ダウンロード受けるようにする。 Purge後最初のアクセス時に、元のサーバーからコンテンツを再キャッシュする。 もし元のサーバーに障害が発生してコンテンツを取得することができない場合は無効化され、コンテンツを復元させてサービスに障害がないように処理する。 このように復元されたコンテンツは、その時点からConnectTimeout設定だけ後ろに更新する。 ::

    http://127.0.0.1:10040/command/purge?url=...
    
ターゲットコンテンツは、URLは、パターンとして指定することができるだけでなく、 "|"（Vertical Bar）を区切り文字を使用して、複数のドメインに複数のターゲットを指定することができる。 もしドメイン名が省略された場合、最近使用されたドメインを使用する。 ::

    http://127.0.0.1:10040/command/purge?url=http://www.site1.com/image.jpg
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image.jpg
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image/bmp/
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image/*.bmp
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image1.jpg|/css/style.css|/script.js
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image1.jpg|www.site2.com/page/*.html
    
結果は、JSON形式で提供される。 ターゲットコンテンツ数/容量と処理時間（単位：ms）が明示される。 すでにPurgeされたコンテンツは、再びPurgeされない。 ::

    {
        "version": "2.0.0",
        "method": "purge",
        "status": "OK",
        "result": { "Count": 24, "Size": 3747491, "Time": 12 }
    }
    
``<Purge2Expire>`` を介して特定の条件のPurgeをExpireで動作するように設定することができる。 結果のない応答には、 ``<ResCodeNoCtrlTarget>`` でHTTP応答コードを設定することができる。


.. note::
   
   ソースサーバーが障害のためにすべて排除された場合、コンテンツを更新することができないため、Purgeが動作しない。
   

.. _api-cmd-expire:
   
Expire
====================================

ターゲットコンテンツのTTLをすぐに期限切れにさせる。 Expire後最初のアクセス時に、元のサーバーから変更するかどうかを確認する。 変更されていない場合TTL延長があるだけコンテンツのダウンロードは発生しない ::

    http://127.0.0.1:10040/command/expire?url=...
    
その他のすべての動作は、 `Purge`_ と同じである。


.. _api-cmd-expireafter:
   
ExpireAfter
====================================

ターゲットコンテンツのTTL有効期限を現在の（API呼び出し時点）から入力された時間（秒）だけ後ろに設定する。 ExpireAfterに有効期限を繰り上げて、コンテンツをより迅速に更新したり、逆に有効期限を増やしソースサーバーの負荷を軽減することができる。 ::

   http://127.0.0.1:10040/command/expireafter?sec=86400&url=...

関数呼び出しの仕様は、 `Purge`_ / `Expire`_ と似ているがsecパラメータ（単位：秒）を使用してTTLの有効期限を指定することができる。 secが省略場合、デフォルト値は1日（86400秒）に設定され、0を入力した場合に失敗する。 結果は、 `Purge`_ / `Expire`_ と同じですが、元のサーバーの障害の有無にかかわらず、動作します。 結果のない応答には、 ``<ResCodeNoCtrlTarget>`` でHTTP応答コードを設定することができる。

.. note::
   ExpireAfterはキャッシュされているコンテンツの現在の有効期限だけを設定するだけでカスタムTTLや設定されたデフォルトのTTLを変更させるAPIはない。 ExpireAfter呼び出しの後、キャッシュされたコンテンツは、影響を受けない。
   
   
   urlパラメータを先に入力した場合、secパラメータがurlパラメータのQueryStringとして認識されることができる。 したがって、secパラメータが最初に入力されていることが安全である。
   
   

.. _api-cmd-hardpurge:
   
HardPurge
====================================

`Purge`_ / `Expire`_ / `ExpireAfter`_ 以上のAPIは、元のサーバーの障害状況でもコンテンツが消えずに正常に動作します。 しかし、HardPurgeは、コンテンツの完全な削除を意味する。 HardPurgeは最も強力な削除方法が、削除されたコンテンツは、元のサーバーに障害が発生しても復帰できない。 結果のない応答には、 ``<ResCodeNoCtrlTarget>`` でHTTP応答コードを設定することができる。 ::

    http://127.0.0.1:10040/command/hardpurge?url=...


Purgeのデフォルトの動作
====================================

Purge APIが呼び出されると、コンテンツの回復するかどうかを選択する。 ::

   # server.xml - <Server><Cache>
   
   <Purge>Normal</Purge>
      
-  ``<Purge>`` 
   
   - ``Normal (基本)`` `Purge`_ で動作する。 (元の障害時復旧する)
   
   - ``Hard`` `HardPurge`_ で動作する。 (元の障害時復旧していない)


HTTP Method
====================================

無効化APIを拡張HTTP Methodで呼び出すことができます。 ::

    PURGE /sample.dat HTTP/1.1
    host: ston.winesoft.co.kr
    
HTTP Methodは、基本的にManagerのポートとサービス（80）ポートで動作します。 サービスポートに要求されるHTTP Methodの :ref:`env-host` で設定する。


.. _api-etc-post:

POST規格
====================================

無効化APIを次のようにPOSTで呼び出すことができます。 ::

   POST /command/purge HTTP/1.1
   Content-Length: 37
 
   url=http://ston.winesoft.co.kr/sample.dat
    

