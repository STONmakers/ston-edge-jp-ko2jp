.. _bandwidth_control:

第16章 Bandwidth
******************

この章では仮想ホストごとに多様な方式のBandwidth制限（調節）する方法について説明します。 過去にはBandwidthが一定レベルを超えないように制限することが目的でした。 今は効果的にBandwidthを調節することに変わりました。 さらにコンテンツをリアルタイムで分析しそれぞれに最適化されたBandwidthを使用するように設定することができます。


.. toctree::
   :maxdepth: 2

仮想ホストBandwidth制限
====================================

仮想ホストの最大Bandwidthを制限します。 これは最優先する物理的な方法です。 ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <TrafficCap Session="0">0</TrafficCap>

-  ``<TrafficCap> (デフォルト: 0 Mbps)``
   の仮想ホストの最大BandwidthをMbps単位で設定します。 0に設定するとBandwidthを制限しません。
   ``Session (デフォルト: 0 Kbps)`` 属性はクライアントセッションごとに送信することができる最大のBandwidthを設定します。

例えば ``<TrafficCap>`` を50（Mbps）に設定した場合は50Mbps NICをインストールしたのと同じ効果を出す。 仮想ホストにアクセスするすべてのクライアントBandwidthの合計は50Mbpsを超えることができない。

``Session`` は次のように動作する

1. ``Session`` が設定されていてもすべてのクライアントBandwidthの合計は ``<TrafficCap>`` を超えることができない。
2. `Bandwidth Throttling`_ を設定してもクライアントセッションの最大速度は ``Session`` を超えることができない。


.. _bandwidth-control-bt:

Bandwidth Throttling
====================================

BT（Bandwidth Throttling）は（各セッションごとに）クライアントの転送帯域幅を動的に調整する機能です。 一般的なメディアファイルの内部には次のようにヘッダ、V（Video）、A（Audio）で構成されている。

.. figure:: img/conf_media_av.png
   :align: center

   ヘッダはBTの対象ではない。

ヘッダは再生時間が長いかKey Frame周期が短いほど大きくなる。 したがって認識することができるメディアファイルであれば円滑な再生のためにヘッダーは帯域幅制限なしで送信します。 次の図のようにヘッダが完全に転送された後BTが開始されます。

.. figure:: img/conf_bandwidththrottling2.png
   :align: center

   動作シナリオ

::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <BandwidthThrottling>
      <Settings>
         <Bandwidth Unit="kbps">1000</Bandwidth>
         <Ratio>100</Ratio>
         <Boost>5</Boost>
      </Settings>
      <Throttling>OFF</Throttling>
   </BandwidthThrottling>

``<BandwidthThrottling>`` タグの下位にデフォルトの動作を設定します

-  ``<Settings>``

   デフォルトの動作を設定します。

   -  ``<Bandwidth> (デフォルト: 1000 Kbps)``
      クライアントの転送帯域幅を設定します。
      ``Unit`` プロパティを介してデフォルト単位( ``kbps`` 、 ``mbps`` 、 ``bytes`` 、 ``kb`` 、 ``mb`` )を設定します。

   -  ``<Ratio> (デフォルト: 100 %)``
      ``<Bandwidth>`` 設定の率を反映して帯域幅を設定します。

   -  ``<Boost> (デフォルト: 5 秒)``
      一定時間だけのデータを速度制限なしのクライアントに送信します。 データの量は ``<Boost>`` X ``<Bandwidth>`` X ``<Ratio>`` 公式で計算します。

-  ``<Throttling>``

   -  ``OFF (デフォルト)`` BTを適用しません。
   -  ``ON`` 条件リストと一致するとBTを適用します。


Bandwidth Throttling条件リスト
--------------------------

BTの条件のリストを設定します。 条件のリストと一致するとBTが適用されます。 設定された順序で条件との一致をチェックします。 トランスポートポリシーは/ svc / {仮想ホスト名} /throttling.txtに設定します。 ::

   # /svc/www.example.com/throttling.txt
   # 区切り文字はカンマ（、）であり、{条件}、{Bandwidth},{Ratio},{Boost} 順に表記します。
   # {条件}を除くすべてのフィールドは省略可能です。
   # 省略されたフィールドは ``<Settings>`` に設定されたデフォルト値が使用されます。
   # すべての条件式はacl.txt設定と同じです。
   # {Bandwidth} 単位は ``<Settings>`` ``<Bandwidth>`` の ``Unit`` 属性を使用します。

   # 3秒のデータを速度制限なしで送信した後3Mbps（3000Kbps = 2000Kbps X 150％）でクライアントに送信します。
   $IP[192.168.1.1], 2000, 150, 3

   # bandwidthのみを定義します。 5（デフォルト）秒のデータを速度制限なしで送信した後800 Kbpsでクライアントに送信します。
   !HEADER[referer], 800

   # boostのみを定義します。  10秒のデータを速度制限なしで送信した後1000 Kbpsにクライアントに送信します。
   HEADER[cookie], , , 10

   # 拡張子がm4aの場合BTを適用しません。
   $URL[*.m4a], no

メディアファイル（MP4M4AMP3）を分析するとEncoding RateからBandwidthを得ることができます。 アクセスされるコンテンツの拡張子は必ず.mp4.m4a.mp3のいずれかでなければならない。 動的にBandwidthを抽出するには次のようにBandwidth後ろ **x** を付ける。 ::

   # /vod/*.mp4 ファイルへのアクセスであればbandwidthデーターを取得します。取得できない場合は1000をbandwidthに使用します。
   $URL[/vod/*.mp4], 1000x, 120, 5

   # user-agentヘッダがない場合は bandwidthデーターを取得します。所得できない場合は 500をbandwidthに使用します。
   !HEADER[user-agent], 500x

   # /low_quality/* ファイルへのアクセスであればbandwidthデーターを取得します。取得できない場合はデフォルト値をbandwidthに使用します。
   $URL[/low_quality/*], x, 200


QueryString優先条件
--------------------------

規定のQueryStringを使用して ``<Bandwidth>`` , ``<Ratio>`` , ``<Boost>`` を動的に設定します。 この設定はBTの条件よりも優先されます。

::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <BandwidthThrottling>
      <Settings>
         <Bandwidth Param="mybandwidth" Unit="mbps">2</Bandwidth>
         <Ratio Param="myratio">100</Ratio>
         <Boost Param="myboost">3</Boost>
      </Settings>
      <Throttling QueryString="ON">ON</Throttling>
   </BandwidthThrottling>

-  ``<Bandwidth>`` , ``<Ratio>`` , ``<Boost>`` の ``Param``

    それぞれの意味に合わせてQueryStringキーを設定します。

-  ``<Throttling>`` の ``QueryString``

   - ``OFF (デフォルト)`` QueryStringに条件を再定義しない。

   - ``ON`` QueryStringに条件を再定義する。

上記のように設定されている場合は次のようにクライアントが要求されたURLに基づいてBTが動的に設定されます。 ::

    # 10秒のデータを速度制限なしで送信した後1.3Mbps（1mbps X 130％）でクライアントに送信します。
    http://www.winesoft.co.kr/video/sample.wmv?myboost=10&mybandwidth=1&myratio=130

必ずすべてのパラメータを指定する必要はありません。 ::

    http://www.winesoft.co.kr/video/sample.wmv?myratio=150

上記のように一部の条件が省略された場合は残りの条件（ここではbandwidthboost）を決定するために条件のリストをチェックします。 ここでも適切な条件が見つからない場合 ``<Settings>`` に設定されたデフォルト値が使用されます。 QueryStringが一部存在しても条件リストで無視オプション（no）が設定されている場合はBTの適用されない。

QueryStringを使用するのでややもすると :ref:`caching-policy-applyquerystring` と混沌する可能性があります。
:ref:`caching-policy-applyquerystring` が ``ON`` の場合クライアントが要求されたURLのQueryStringがすべて認識されますが ``BoostParam`` , ``BandwidthParam`` , ``RatioParam`` は除外されます。 ::

   GET /video.mp4?mybandwidth=2000&myratio=130&myboost=10
   GET /video.mp4?tag=3277&myboost=10&date=20130726

例えば上記のような入力はBTを決定するためだけに使用されます。Caching-Keyを生成したりオリジンサーバーに要求を送信する場合は削除されます。 つまりそれぞれ次のように認識されます。 ::

    GET /video.mp4
    GET /video.mp4?tag=3277&date=20130726
