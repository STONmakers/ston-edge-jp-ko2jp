.. _media-video:

第18章 動画
******************

.. note::

   - `[動画講座]みよう！ STON Edge Server - Chapter 5. 動画配信 <https://youtu.be/YjOEVamhah4?list=PLqvIfHb2IlKeZ-Eym_UPsp6hbpeF-a2gE>`_

この章ではビデオ/オーディオをスマートにサービスする方法について説明します。 クライアント側はシームレスなスムーズな再生が一貫性のある目的になりますが、サーバ側では非常に複雑です。 画像の画質向上はより大きな帯域幅とストレージ容量を必要とします。 STONは様々なOn-the-fly手法を用いて既存Back-Endの修正なし柔軟な転送機能を提供します。 


.. toctree::
   :maxdepth: 2


.. _media-hls:

MP4 HLS
====================================

MP4ファイルをHLS（HTTP Live Streaming）にサービスします。 オリジンサーバーはHLSサービスのためにファイルを分割保存する必要がない。 MP4ファイルのヘッダの位置に関係なくダウンロードと同時にリアルタイムで.m3u8/.tsファイルの変換後にサービスします。

..  note::

    MP4HLSはElementary Stream（VideoまたはAudio）を変換するトランスコーディング（Transcoding）ではない。 したがってHLSに適した形式でエンコードされたMP4ファイルに限って円滑な再生が可能です。 エンコーディングが適合しない場合は画面が割れたり音が再生されないことがあります。 現在（2014.2.20）Appleの言っているVideo / Audioエンコード規格は次のとおりです。

    What are the specifics of the video and audio formats supported?
    Although the protocol specification does not limit the video and audio formats, the current Apple implementation supports the following formats:

    [Video]
    H.264 Baseline Level 3.0, Baseline Level 3.1, Main Level 3.1, and High Profile Level 4.1.

    [Audio]
    HE-AAC or AAC-LC up to 48 kHz, stereo audio
    MP3 (MPEG-1 Audio Layer 3) 8 kHz to 48 kHz, stereo audio
    AC-3 (for Apple TV, in pass-through mode only)

    Note: iPad, iPhone 3G, and iPod touch (2nd generation and later) support H.264 Baseline 3.1. If your app runs on older versions of iPhone or iPod touch, however, you should use H.264 Baseline 3.0 for compatibility. If your content is intended solely for iPad, Apple TV, iPhone 4 and later, and Mac OS X computers, you should use Main Level 3.1.


従来の方式の場合Pseudo-StreamingとHLSのために以下のように元のファイルがそれぞれ存在する必要があります。 このような場合STONも元のファイルをそのまま複製してクライアント側にサービスします。 しかし再生時間が長いほど派生ファイルは多くなり管理の難しさは増加します。

.. figure:: img/conf_media_mp4hls1.png
   :align: center

   手間が多くHLS

``<MP4HLS>`` は元のファイルからHLSサービスに必要なファイルを動的に生成します。

.. figure:: img/conf_media_mp4hls2.png
   :align: center

   スマートHLS

すべての.m3u8/.tsファイルは元のファイルから派生し別のストレージスペースを消費しません。 サービスすぐにメモリに一時的に生成されサービスされない場合自動的に消える。 ::

   # server.xml - <Server><VHostDefault><Media>
   # vhosts.xml - <Vhosts><Vhost><Media>

   <MP4HLS Status="Inactive" Keyword="mp4hls">
      <Index Ver="3" Alternates="off">index.m3u8</Index>
      <Sequence>0</Sequence>
      <Duration>10</Duration>
      <AlternatesName>playlist.m3u8</AlternatesName>
   </MP4HLS>

-  ``<MP4HLS>``

   - ``Status (デフォルト: Inactive)`` の値が ``Active`` の場合にのみ有効になる。

   - ``Keyword (デフォルト: mp4hls)`` HLSサービスキーワード

-  ``<Index> (デフォルト: index.m3u8)`` HLSインデックス（.m3u8）ファイル名 

   - ``Ver (デフォルト 3)`` インデックスファイルのバージョン。 3である場合 ``#EXT-X-VERSION:3`` ヘッダが指定されて ``#EXTINF`` の時刻の値が小数点3桁目まで表示されます。 1の場合 ``#EXT-X-VERSION`` ヘッダがなく ``#EXTINF`` の時間値が整数（丸め）に表示されます。

   - ``Alternates (デフォルト: OFF)`` Stream Alternates使用有無。

     .. figure:: img/hls_alternates_off.png
        :align: center

        OFF. ``<Index>`` でTSリストをサービスします。

     .. figure:: img/hls_alternates_on.png
        :align: center

        ON. ``<AlternatesName>`` でTSリストをサービスします。

-  ``<Sequence> (デフォルト: 0)`` .tsファイルの開始番号。 このことに基づいて順次増加する。

-  ``<Duration> (デフォルト: 10秒)`` のMP4 HLSに分割する基準時間（秒）。 分割の基準はVideo / AudioのKeyFrameです。 KeyFrameはギザギザすることができますので正確に分割されない。 もし10秒分割しようとしてKeyFrameが9秒と12秒の場合は近い値（9秒）を選択します。

-  ``<AlternatesName> (デフォルト: playlist.m3u8)`` Stream Alternatesファイル名。 ::

      http://www.example.com/video.mp4/mp4hls/playlist.m3u8


サービスアドレスは次のとおりである場合はそのアドレスにPseudo-Streamingを行うことができます。 ::

    http://www.example.com/video.mp4

仮想ホストは ``<MP4HLS>`` に定義された ``Keyword`` 文字列を認識することによりHLSサービスを進行します。 次のURLが呼び出されると/video.mp4からindex.m3u8ファイルを生成します。 ::

   http://www.example.com/video.mp4/mp4hls/index.m3u8

``Alternates`` 属性がONであれば ``<Index>`` ファイルは ``<AlternatesName>`` ファイルをサービスします。 ::

   #EXTM3U
   #EXT-X-VERSION:3
   #EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=200000,RESOLUTION=720x480
   /video.mp4/mp4hls/playlist.m3u8

``#EXT-X-STREAM-INF`` のBandwidthとResolutionは映像を分析して動的に提供します。

.. note::

   Stream Alternatesを提供しますが現在のバージョンではindex.m3u8は常に一つのサブインデックスファイル（playlist.m3u8）だけを提供します。 キャッシュの立場ではvideo_1080.mp4とvideo_720.mp4が（エンコードオプションが他の）のような映像なのか知ることができないからです。


最終的に生成された.tsリスト（バージョン3）は次のとおりです。 ::

   #EXTM3U
   #EXT-X-TARGETDURATION:10
   #EXT-X-VERSION:3
   #EXT-X-MEDIA-SEQUENCE:0
   #EXTINF:11.637,
   /video.mp4/mp4hls/0.ts
   #EXTINF:10.092,
   /video.mp4/mp4hls/1.ts
   #EXTINF:10.112,
   /video.mp4/mp4hls/2.ts

   ... (中略)...

   #EXTINF:10.847,
   /video.mp4/mp4hls/161.ts
   #EXTINF:9.078,
   /video.mp4/mp4hls/162.ts
   #EXT-X-ENDLIST

分割には3つのポリシーがあります。

-  **KeyFrame間隔よりも** ``<Duration>`` **の設定が大きい場合**
   KeyFrameが3秒 ``<Duration>`` が20秒であれば20秒を超えないKeyFrameの倍数である18秒に分割されます。

-  **KeyFrame間隔と** ``<Duration>`` **が似ている場合**
   KeyFrameが9秒 ``<Duration>`` が10秒であれば10秒を超えないKeyFrameの倍数である9秒分けられる。

-  **KeyFrame間隔が** ``<Duration>`` **に設定よりも大きい場合**
   KeyFrame単位に分割されます。

次のクライアント要求に対してSTONがどのように動作するのかを理解しましょう。  ::

   GET /video.mp4/mp4hls/99.ts HTTP/1.1
   Range: bytes=0-512000
   Host: www.winesoft.co.kr

1.	``STON`` 最初のロード（何もキャッシュされません。）
#.	``Client`` HTTP Range要求（100番目のファイルの最初の500KBリクエスト）
#.	``STON`` /video.mp4ファイルのキャッシュオブジェクトの作成
#.	``STON`` /video.mp4ファイルの分析のために必要な部分だけをオリジンサーバーからダウンロード
#.	``STON`` 100番目（99.ts）ファイルサービスのために必要な部分だけをオリジンサーバーからダウンロード
#.	``STON`` 100番目（99.ts）ファイルを作成した後Rangeサービス
#.	``STON`` サービスが完了すると99.tsファイルpurge

.. note::

   ``MP4Trimming`` 機能が ``ON`` の場合TrimmingされたMP4をHLSに変換できる。 （HLSイメージをTrimmingすることができない。HLSのMP4ではなくMPEG2TSであることに注意しよう。）映像をTrimmingした後HLSに変換するため次のように表現するのが自然です。 ::

      /video.mp4?start=0&end=60/mp4hls/index.m3u8

   動作には問題ありませんがQueryStringを一番後ろに付けるHTTP仕様に反します。 これを補完するために次のような表現も動作は同じです。 ::

      /video.mp4/mp4hls/index.m3u8?start=0&end=60
      /video.mp4?start=0/mp4hls/index.m3u8?end=60


.. _media-mp3-hls:

MP3 HLS
====================================

MP3ファイルをHLS（HTTP Live Streaming）にサービスします。 ::

   # server.xml - <Server><VHostDefault><Media>
   # vhosts.xml - <Vhosts><Vhost><Media>

   <MP3HLS Status="Inactive" Keyword="mp3hls" SegmentType="TS">
      <Index Ver="3" Alternates="off">index.m3u8</Index>
      <Sequence>0</Sequence>
      <Duration>10</Duration>
      <AlternatesName>playlist.m3u8</AlternatesName>
   </MP3HLS>

すべての設定と動作が `MP4 HLS`_ と同じでさらにSegement形式を選択することができます。 

-  ``<MP3HLS>``

   - ``SegmentType (デフォルト: TS)`` オリジンサーバーのMP3 MPEG2-TS( ``TS`` ) または ``MP3`` に分割します。

.. note::

   `MP4 HLS`_ と `MP3 HLS`_ が同じ ``Keyword`` に設定されている場合 `MP3 HLS`_ は動作しません。




MP4/M4Aヘッダの位置を変更
====================================

通常MP4形式の場合エンコード処理中にヘッダを完成することができないため完了後にファイルの末尾に付ける。 ヘッダを前の部分に移動するには別の処理が必要です。 ヘッダが後ろにいる場合これをサポートしていないプレーヤーでPseudo-Streamingが不可能です。 ヘッダの位置の変更によりPseudo-Streamingを簡単にサポートすることができます。

ヘッダの位置の変更は送信段階でのみ発生するだけでテキストの形を変更しません。 別のストレージスペースを使用することもない。 ::

   # server.xml - <Server><VHostDefault><Media>
   # vhosts.xml - <Vhosts><Vhost><Media>

   <UpfrontMP4Header>OFF</UpfrontMP4Header>
   <UpfrontM4AHeader>OFF</UpfrontM4AHeader>

-  ``<UpfrontMP4Header>``

   - ``OFF (デフォルト)`` 何もしません。

   - ``ON`` 拡張子が.mp4でヘッダが続いている場合ヘッダーを今後移し送信します。

-  ``<UpfrontM4AHeader>``

   - ``OFF (デフォルト)`` 何もしません。

   - ``ON`` 拡張子が.m4aでヘッダが続いている場合ヘッダーを今後移し送信します。

最初に要求されているコンテンツのヘッダを前に移動する必要が場合ヘッダを移すために必要な部分を優先的にダウンロードされます。 非常にスマートなだけでなく高速に動作します。 カーテンの後ろの複雑なプロセスとは関係なくクライアントはもともとヘッダが前にある完全なファイルをサービス受ける。

.. note::

   分析することができない場合または壊れたファイルであれば元の形のままサービスされます。


.. _media-trimming:

Trimming
====================================

時間値に基づいて必要な区間を抽出します。 Trimmingは送信段階でのみ発生するだけで元のファイルの形を変更しません。 別のストレージスペースを使用しません。 ::

   # server.xml - <Server><VHostDefault><Media>
   # vhosts.xml - <Vhosts><Vhost><Media>

   <MP4Trimming StartParam="start" EndParam="end" AllTracks="off">OFF</MP4Trimming>
   <M4ATrimming StartParam="start" EndParam="end" AllTracks="off">OFF</M4ATrimming>
   <MP3Trimming StartParam="start" EndParam="end">OFF</MP3Trimming>

-  ``<MP4Trimming>`` ``<MP3Trimming>`` ``<M4ATrimming>``

   - ``OFF (デフォルト)`` 何もしません。

   - ``ON`` 拡張子(.mp4、 .mp3、 .m4a)が一致すると必要な区間だけサービスするようにTrimmingします。 Trimming区間は ``StartParam`` 属性と ``EndParam`` に設定します。

   - ``AllTracks`` 属性

     - ``OFF (デフォルト)`` Audio / VideoトラックのみTrimmingします。 （Mod-H264方式）

     - ``ON`` すべてのトラックのTrimmingします。 使用前に必ずプレーヤーの互換性を確認する必要があります。

パラメータはクライアントQueryStringを介して入力されます。 たとえば10分の動画（/video.mp4）を特定区間だけTrimmingしたい場合はQueryStringに任意の時点（単位：秒）を指定します。 ::

   http://vod.wineosoft.co.kr/video.mp4                // 10分：全ムービー
   http://vod.wineosoft.co.kr/video.mp4?end=60         // 1分：最初から60秒まで
   http://vod.wineosoft.co.kr/video.mp4?start=120      // 8分：2分（120秒）から最後まで
   http://vod.wineosoft.co.kr/video.mp4?start=3&end=13 // 10秒：3秒から13秒まで

``StartParam`` 値が ``EndParam`` 値よりも大きい場合区間が指定されていないものと判断します。 この機能はHTTP Pseudo-Streamingに実装されたビデオプレーヤーのSkip機能のために開発された。 したがってRange要求を処理するようにファイルをOffsetに基づいて切らずに正常に再生されるようにキーフレームと時間を認知して区間を抽出します。

クライアントに配信されるファイルは次の図のようにMP4ヘッダが再生成された完全な形のMP4ファイルです。

.. figure:: img/conf_media_mp4trimming.png
   :align: center

   完全な形のファイルが提供されます。

抽出された区間は別のファイルとして認識されるため200 OKで応答されます。 したがって次のようにRangeヘッダが記載されている場合抽出されたファイルからRangeを計算して **206 Particial Content** で応答します。

.. figure:: img/conf_media_mp4trimming_range.png
   :align: center

   一般的なRangeリクエストのように処理されます。

区間抽出パラメータがQueryString表現を使用するためややもすると :ref:`caching-policy-applyquerystring` と混乱することができます。
``<ApplyQueryString>`` の設定が ``ON`` の場合クライアントが要求されたURLのQueryStringがすべて認識され ``StartParam`` と ``EndParam`` は除去されます。 ::

   GET /video.mp4?start=30&end=100
   GET /video.mp4?tag=3277&start=30&end=100&date=20130726

例えば上記のように ``StartParam`` が **start** で ``EndParam`` が **end** で入力された場合この値は区間を抽出するのに使われるだけでCaching-Keyを生成したりオリジンサーバーに要求を送信する場合は削除されます。 それぞれ次のように認識されます。 ::

   GET /video.mp4
   GET /video.mp4?tag=3277&date=20130726

またQueryStringパラメータは拡張モジュールやCDNソリューションによって異なることができます。

.. figure:: img/conf_media_mp4trimming_range.png
   :align: center

   JW Playerで提供しているModule / CDN星参考資料

以外のnginxの `ngx_http_mp4_module <http://nginx.org/en/docs/http/ngx_http_mp4_module.html>`_ とlighttpdの `Mod-H264-Streaming-Testing-Version2 <http://h264.code-shop.com/trac/wiki/Mod-H264-Streaming-Testing-Version2>`_ もすべて **start** をQueryStringに使用します。



.. _media-multi-trimming:

Multi-Trimming
====================================

時間値に基づいて複数の指定された区間を一つの映像として抽出します。

.. figure:: img/conf_media_multitrimming.png
   :align: center

   /video.mp4?trimming=0-30,210-270,525-555

区間の指定方法が違うだけで動作は `Trimming`_ と同じです。 ::

   # server.xml - <Server><VHostDefault><Media>
   # vhosts.xml - <Vhosts><Vhost><Media>

   <MP4Trimming MultiParam="trimming" MaxRatio="50">OFF</MP4Trimming>
   <M4ATrimming MultiParam="trimming">OFF</M4ATrimming>

-  ``<MP4Trimming>`` ``<M4ATrimming>``

   - ``MultiParam (デフォルト: "trimming")``
     に設定され名前をQueryString Keyとして使用して抽出区間を指定します。 一つの区間は "開始時刻 - 終了時刻" と表記し各区間はコンマ（、）で接続します。

   - ``MaxRatio (デフォルト: 50%)``
     Multi-Trimmingされた映像はオリジナルよりも ``MaxRatio (最大 100%)`` の割合だけまで大きくなることができます。
     ``MaxRatio`` を移る区間は無視されます。


例えば次のように起動すると3分の映像が生成されます。 ::

   http://example.com/video.mp4?trimming=10-70,560-620,1245-1305

同じ映像を繰り返したり前の背部変わった映像を作成することもできる。 ::

   http://example.com/video.mp4?trimming=17-20,17-20,17-20,17-20
   http://example.com/video.mp4?trimming=1000-1200,500-623,1900-2000
   http://example.com/video.mp4?trimming=600-,400-600

区間値を指定しない場合先頭または最後に意味します。


.. note::

   `Multi-Trimming`_ は `Trimming`_ より優先します。 QueryStringに `Multi-Trimming`_ キーが指定されている場合は `Trimming`_ キーは無視されます。


