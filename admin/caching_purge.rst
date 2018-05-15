.. _caching-purge:

第5章 Caching削除
******************

この章ではCachingされたコンテンツを削除する方法について説明します。 業界用語でPurgeで通称が様々な状況や環境により細分化されたAPIが必要です。

オリジンサーバーからキャッシュされたコンテンツは :ref:`caching-policy-ttl` に基づい更新周期を持つ。 しかし明らかに内容が変更され管理者がこれをすぐに反映したい場合 :ref:`caching-policy-ttl` が期限切れになるまで待つ必要はありません。
`Purge`_ / `Expire`_ / `HardPurge`_ などを使用するとすぐにコンテンツを削除させることができます。

削除APIは単にブラウザによって呼び出される場合もあるが自動化されている場合が多い。 例えばFTP経由でファイルのアップロードが完了したらすぐに `Purge`_ を呼び出す方法もあります。 管理者は次のようにいくつかの動作を設定することができます。 ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>
   
   <Purge2Expire>NONE</Purge2Expire>
   <RootPurgeExpire>ON</RootPurgeExpire>
   <ResCodeNoCtrlTarget>200</ResCodeNoCtrlTarget>

-  ``<Purge2Expire> (デフォルト: NONE)``

   `Purge`_ 要求を設定に応じて `Expire`_ で処理します。 たとえば特定のパターン(*.jpg)を  `Purge`_ する場合意図せず多くのコンテンツが削除されソースに過度な負荷を発生する場合があります。 このような場合 `Expire`_ に処理するように設定すると過剰なオリジンサーバーの負荷を防止することができます。
   
   - ``NONE`` `Expire`_ で処理しません。
   - ``ROOT`` ドメイン全体（/ * ）の `Purge`_ を `Expire`_ で処理します。
   - ``PATTERN`` すべてのパターン `Purge`_ を `Expire`_ で処理します。
   - ``ALL`` すべての `Purge`_ を `Expire`_ で処理します。

-  ``<RootPurgeExpire> (デフォルト: ON)``
   
   全体のコンテンツへの意図しない `Purge`_ / `Expire`_ は過度のオリジンサーバーの負荷を発生させることができます。 この設定を介してすべてのコンテンツの `Purge`_ / `Expire`_ を遮断することができます。 この設定は ``<Purge2Expire>`` よりも優先します。
   
   - ``ON`` `Purge`_ / `Expire`_ を可能にします。
   - ``PURGE`` `Purge`_ のみ可能にします。
   - ``EXPIRE`` `Expire`_ のみ可能にします。
   - ``OFF`` すべての `Purge`_ / `Expire`_ を禁止します。

-  ``<ResCodeNoCtrlTarget> (デフォルト: 200)``

   `Purge`_ , `Expire`_ , `HardPurge`_ , `ExpireAfter`_ の対象オブジェクトが存在しないときのHTTP応答コードを設定します。
   

ターゲット設定はURLとパターンの2つの方法で表現します。 ::

   example.com/logo.jpg      // URL
   example.com/img/          // URL
   example.com/img/*.jpg     // パターン
   example.com/img/*         // パターン
   
明確なURLのほかのパターン(*.jpg)でも削除が可能です。 しかし作業を実行するまでターゲットコンテンツの数を明確に把握できない。 これは管理者の意図とは違い多くの対象が指定される可能性があり実際にCPUリソースをあまり消費することになりシステム全体に負担を与えることがなる可能性があります。
   
したがって実サービスの中には明確なURLだけを設定することを強く推奨します。 パターン表現はサービスから排除された状態で管理目的で使用するためです。


.. note::

   セキュリティ上の理由からexample.com/files/などの特定のディレクトリへのアクセスは403 FORBIDDENなどで遮断されます。 しかしルートディレクトリは例外を持つ。 たとえばユーザーがexample.comにアクセスするとブラウザはルートディレクトリ（/）を要請します。 ::
   
      GET / HTTP/1.1
      Host: example.com
   
   これに対してWebサーバーは管理者が設定したデフォルトのページ（おそらくindex.htmlまたはindex.htm）で応答します。 明らかにWebサービスの構成ではルートディレクトリ（/）はディレクトリではなくページで動作します。
   
   しかしCacheサーバーはルートディレクトリ（/）にアクセスしたところ200 OKのページが来た理解します。 さらにオリジンサーバーがどのページを応答した知らない。 簡単に整理するとCacheサーバーの観点ではディレクトリ表現もURLの一種に過ぎない。 ::
   
      example.com/img/          // example.com 仮想ホストの / img / に アクセスした 結果 のページ
      example.com/              // example.com の仮想ホストの デフォルト ページ （/）
      example.com/img/*         // example.com 仮想ホストの / img ディレクトリと その サブ ページ
      example.com/*             // example.com 仮想ホストの すべての コンテンツ
         


.. toctree::
   :maxdepth: 2


.. _api-cmd-purge:

Purge
====================================

ターゲットコンテンツを無効にさせてオリジンサーバーからコンテンツを再ダウンロード受けるようにします。 Purge後最初のアクセス時にオリジンサーバーからコンテンツを再キャッシュします。 もしオリジンサーバーに障害が発生してコンテンツを取得することができない場合は削除されコンテンツを復元させてサービスに障害がないように処理します。 このように復元されたコンテンツはその時点からConnectTimeout設定だけ後ろに更新します。 ::

    http://127.0.0.1:10040/command/purge?url=...
    
ターゲットコンテンツはURLはパターンとして指定することができるだけでなく "|"（Vertical Bar）を区切り文字を使用して複数のドメインに複数のターゲットを指定することができます。 もしドメイン名が省略された場合最近使用されたドメインを使用します。 ::

    http://127.0.0.1:10040/command/purge?url=http://www.site1.com/image.jpg
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image.jpg
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image/bmp/
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image/*.bmp
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image1.jpg|/css/style.css|/script.js
    http://127.0.0.1:10040/command/purge?url=www.site1.com/image1.jpg|www.site2.com/page/*.html
    
結果はJSON形式で提供されます。 ターゲットコンテンツ数/容量と処理時間（単位：ms）が指定されます。 すでにPurgeされたコンテンツは再びPurgeされない。 ::

    {
        "version": "2.0.0",
        "method": "purge",
        "status": "OK",
        "result": { "Count": 24, "Size": 3747491, "Time": 12 }
    }
    
``<Purge2Expire>`` を介して特定の条件のPurgeをExpireで動作するように設定することができます。 結果のない応答には ``<ResCodeNoCtrlTarget>`` でHTTP応答コードを設定することができます。


.. note::
   
   オリジンサーバーが障害のためにすべて排除された場合コンテンツを更新することができないためPurgeが動作しません。
   

.. _api-cmd-expire:
   
Expire
====================================

ターゲットコンテンツのTTLをすぐに期限切れにせる。 Expire後最初のアクセス時にオリジンサーバーから変更有無を確認します。 変更されていない場合TTL延長があるだけコンテンツのダウンロードは発生しない。 ::

    http://127.0.0.1:10040/command/expire?url=...
    
その他のすべての動作は `Purge`_ とと同じです。


.. _api-cmd-expireafter:
   
ExpireAfter
====================================

ターゲットコンテンツのTTL有効期限を現在の（API呼び出し時点）から入力された時間（秒）だけ後ろに設定します。 ExpireAfterに有効期限を繰り上げてコンテンツをより迅速に更新したり反対に有効期限を増やしオリジンサーバーの負荷を軽減することができます。 ::

   http://127.0.0.1:10040/command/expireafter?sec=86400&url=...

関数呼び出しの仕様は `Purge`_ / `Expire`_ と似ているがsecパラメータ（単位：秒）を使用してTTLの有効期限を指定することができます。 secが省略場合デフォルト値は1日（86400秒）に設定され0を入力した場合に失敗します。 結果は `Purge`_ / `Expire`_ と同じですがオリジンサーバーの障害の有無にかかわらず動作します。 結果のない応答には ``<ResCodeNoCtrlTarget>`` でHTTP応答コードを設定することができます。

.. note::
   ExpireAfterはキャッシュされているコンテンツの現在の有効期限だけを設定するだけでカスタムTTLや設定されたデフォルトのTTLを変更させるAPIではない。 ExpireAfter呼び出しの後キャッシュされたコンテンツは影響を受けない。
   
   
   urlパラメータを先に入力した場合secパラメータがurlパラメータのQueryStringとして認識されることができます。 したがってsecパラメータが最初に入力されていることが安全です。
   
   

.. _api-cmd-hardpurge:
   
HardPurge
====================================

`Purge`_ / `Expire`_ / `ExpireAfter`_ 以上のAPIはオリジンサーバーの障害状況でもコンテンツが消えずに正常に動作します。 しかしHardPurgeはコンテンツの完全な削除を意味します。 HardPurgeは最も強力な削除方法が削除されたコンテンツはオリジンサーバーに障害が発生しても復帰できない。 結果のない応答には ``<ResCodeNoCtrlTarget>`` でHTTP応答コードを設定することができます。 ::

    http://127.0.0.1:10040/command/hardpurge?url=...


Purgeのデフォルトの動作
====================================

Purge APIが呼び出されるとコンテンツの復旧有無を選択します。 ::

   # server.xml - <Server><Cache>
   
   <Purge>Normal</Purge>
      
-  ``<Purge>`` 
   
   - ``Normal (デフォルト)`` `Purge`_ で動作します。 (元の障害時復旧する)
   
   - ``Hard`` `HardPurge`_ で動作します。 (元の障害時復旧していない)


HTTP Method
====================================

削除APIを拡張HTTP Methodで呼び出すことができます。 ::

    PURGE /sample.dat HTTP/1.1
    host: ston.winesoft.co.kr
    
HTTP Methodはデフォルト的にManagerのポートとサービス（80）ポートで動作します。 サービスポートに要求されるHTTP Methodの :ref:`env-host` で設定します。


.. _api-etc-post:

POST規格
====================================

削除APIを次のようにPOSTで呼び出すことができます。 ::

   POST /command/purge HTTP/1.1
   Content-Length: 37
 
   url=http://ston.winesoft.co.kr/sample.dat
    

