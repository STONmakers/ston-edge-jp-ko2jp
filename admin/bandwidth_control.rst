.. _bandwidth_control:

第16章 Bandwidth
******************

この章では、仮想ホストごとに、さまざまな方式のBandwidth制限（調節）する方法について説明する。 かつてはBandwidthが一定レベルを超えないように制限することが目的であった。 今効果的にBandwidthを調節することで、その概念が移っていった。 さらに、コンテンツをリアルタイムで分析し、それぞれに最適化されたBandwidthを使用するように設定することができる。


.. toctree::
   :maxdepth: 2

仮想ホストBandwidth制限
====================================

仮想ホストの最大Bandwidthを制限する。 これは最も優先する物理的な方法である。 ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <TrafficCap Session="0">0</TrafficCap>

-  ``<TrafficCap> (基本: 0 Mbps)``
   の仮想ホストの最大BandwidthをMbps単位で設定する。 0に設定すると、Bandwidthを制限しない。
   ``Session (基本: 0 Kbps)`` 属性は、クライアントセッションごとに送信することができる最大のBandwidthを設定する。

例えば、 ``<TrafficCap>`` を50（Mbps）に設定した場合は50Mbps NICをインストールしたのと同じ効果を出す。 仮想ホストにアクセスするすべてのクライアントBandwidthの合計は50Mbpsを超えることができない。

``Session`` は、次のように動作する

1. ``Session`` が設定されていても、すべてのクライアントBandwidthの合計は ``<TrafficCap>`` を超えることができない。
2. `Bandwidth Throttling`_ を設定しても、クライアントセッションの最大速度は、 ``Session`` を超えることができない。


.. _bandwidth-control-bt:

Bandwidth Throttling
====================================

BT（Bandwidth Throttling）と（各セッションごとに）クライアントの転送帯域幅を動的に調整する機能である。 一般的なメディアファイルの内部には、次のようにヘッダ、V（Video）、A（Audio）で構成されている。

.. figure:: img/conf_media_av.png
   :align: center

   ヘッダは、BTの対象ではない。

ヘッダは、再生時間が長いKey Frame周期が短いほど大きくなる。 したがって、認識することができるメディアファイルであれば、円滑な再生のために、ヘッダーは、帯域幅制限なしで送信する。 次の図のようにヘッダが完全に転送された後、BTが開始される。

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

``<BandwidthThrottling>`` タグの下位に、デフォルトの動作を設定する。

-  ``<Settings>``

   デフォルトの動作を設定する。

   -  ``<Bandwidth> (基本: 1000 Kbps)``
      クライアントの転送帯域幅を設定する。
      ``Unit`` プロパティを介して基本単位( ``kbps`` , ``mbps`` , ``bytes`` , ``kb`` , ``mb`` )を設定する。

   -  ``<Ratio> (基本: 100 %)``
      ``<Bandwidth>`` 設定の率を反映して、帯域幅を設定する。

   -  ``<Boost> (基本: 5 초)``
      一定時間だけのデータを速度制限なしのクライアントに送信する。 データの量は、 ``<Boost>`` X ``<Bandwidth>`` X ``<Ratio>`` 公式で計算する。

-  ``<Throttling>``

   -  ``OFF (基本)`` BTを適用しない。
   -  ``ON`` 条件リストと一致するとBTを適用する。


Bandwidth Throttling条件リスト
--------------------------

BTの条件のリストを設定する。 条件のリストと一致する必要がBTが適用される。 設定された順序で条件と一致する検査である。 トランスポートポリシーは、/ svc / {仮想ホスト名} /throttling.txtに設定する。 ::

   # /svc/www.example.com/throttling.txt
   # 区切り文字はカンマ（、）であり、{条件}、{Bandwidth},{Ratio},{Boost} 順に表記する。
   # {条件}を除くすべてのフィールドは省略可能である。
   # 省略されたフィールドは、 ``<Settings>`` に設定されたデフォルト値が使用される。
   # すべての条件式はacl.txt設定と同じである。
   # {Bandwidth} 単位は ``<Settings>`` ``<Bandwidth>`` の  ``Unit`` 属性を使用する。

   # 3秒のデータを速度制限なしで送信した後、3Mbps（3000Kbps = 2000Kbps X 150％）で、クライアントに送信する。
   $IP[192.168.1.1], 2000, 150, 3

   # bandwidthのみを定義する。  5（基本）秒のデータを速度制限なしで送信した後、800 Kbpsで、クライアントに送信する。
   !HEADER[referer], 800

   # boostのみを定義する。  10秒のデータを速度制限なしで送信した後、1000 Kbpsに、クライアントに送信する。
   HEADER[cookie], , , 10

   # 拡張子がm4aの場合BTを適用しない。
   $URL[*.m4a], no

メディアファイル（MP4、M4A、MP3）を分析すると、Encoding RateからBandwidthを得ることができる。 アクセスされるコンテンツの拡張子は必ず.mp4、.m4a、.mp3のいずれかでなければならない。 動的にBandwidthを抽出するには、次のようにBandwidth後ろ **x** を付ける。 ::

   # /vod/*.mp4 ファイルへのアクセスであれば、bandwidthを求める。 入手できない場合は、 1000を bandwidthbandwidthに使用する。
   $URL[/vod/*.mp4], 1000x, 120, 5

   # user-agentヘッダがない場合は、 bandwidthを求める。 入手できない場合は、 500をbandwidthに使用する。
   !HEADER[user-agent], 500x

   # /low_quality/* ファイルへのアクセスであれば、bandwidthを求める。 入手できない場合は、デフォルト値をbandwidthに使用する。
   $URL[/low_quality/*], x, 200


QueryString優先条件
--------------------------

約束されたQueryStringを使用して ``<Bandwidth>`` , ``<Ratio>`` , ``<Boost>`` を動的に設定する。 この設定は、BTの条件よりも優先される。

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

    それぞれの意味に合わせてQueryStringキーを設定する。

-  ``<Throttling>`` の ``QueryString``

   - ``OFF (基本)`` QueryStringに条件を再定義していない。

   - ``ON`` QueryStringに条件を再定義する。

上記のように設定されている場合は、次のように、クライアントが要求されたURLに基づいてBTが動的に設定される。 ::

    # 10秒のデータを速度制限なしで送信した後、1.3Mbps（1mbps X 130％）で、クライアントに送信する。
    http://www.winesoft.co.kr/video/sample.wmv?myboost=10&mybandwidth=1&myratio=130

必ずしもすべてのパラメータを指定する必要はない。::

    http://www.winesoft.co.kr/video/sample.wmv?myratio=150

上記のように、いくつかの条件が省略された場合、残りの条件（ここではbandwidth、boost）を決定するために条件のリストを検索する。 ここでも、適切な条件が見つからない場合 ``<Settings>`` に設定されたデフォルト値が使用される。 QueryStringが、いくつかの存在も条件の一覧で、配達オプション（no）が設定されている場合は、BTの適用されない。

QueryStringを使用するので、ややもすると :ref:`caching-policy-applyquerystring` と混同を引き起こす恐れがある。
:ref:`caching-policy-applyquerystring` が ``ON`` の場合、クライアントが要求されたURLのQueryStringがすべて認識されますが ``BoostParam`` , ``BandwidthParam`` , ``RatioParam`` は除外される。 ::

   GET /video.mp4?mybandwidth=2000&myratio=130&myboost=10
   GET /video.mp4?tag=3277&myboost=10&date=20130726

例えば、上記のような入力は、BTを決定する使わだけCaching-Keyを生成したり、元のサーバーに要求を送信する場合は削除される。 つまり、それぞれ次のように認識される。 ::

    GET /video.mp4
    GET /video.mp4?tag=3277&date=20130726
