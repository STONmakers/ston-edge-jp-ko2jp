.. _monitoring_stats:

第10章 監視＆統計
******************

この章では、モニタリングと統計について説明する。 監視と統計は用途に応じて異なる理解されている場合が多い。 しかし、サービスは数字で話するという観点から、二人は同じである。

ここで最も重要な要素は、リアルタイム性である。 5分も長い。 リアルタイムでサービスの状態の変化を見ることができなければならない。 多くのポリシーが適用さと同時に効果を出すのか、すぐに知ることができなければならない。 すべての統計情報は、1秒単位で収集され、最小単位となる。

すべての統計情報は、仮想ホストごとに別々に収集されるだけでなく、リアルタイム（1秒）、5分平均で提供される。 顧客が統計をより簡単に分析、加工することができるようにJSONとXMLフォーマットで提供する。 ::

    http://127.0.0.1:10040/monitoring/realtime?type=[JSON または XML]
    http://127.0.0.1:10040/monitoring/average?type=[JSON または XML]

-  ``realtime``
   1秒前のサービスの状態を提供する。

-  ``average``
   5分単位の統計情報を提供する。


.. toctree::
   :maxdepth: 2



収集範囲
====================================

統計情報の収集範囲を設定する。 ::

   # server.xml - <Server><VHostDefault>
   # vhosts.xml - <Vhosts><Vhost>

   <Stats>
      <DirDepth>0</DirDepth>
      <DirDepthAccum>OFF</DirDepthAccum>
      <HttpsTraffic>OFF</HttpsTraffic>
      <ClientLocal>OFF</ClientLocal>
      <OriginLocal>OFF</OriginLocal>
   </Stats>

-  ``<DirDepth> (基本: 0)``

   ディレクトリごとの統計情報を収集している。 0に設定された場合、すべての統計情報を、ルート（/）ディレクトリに収集する。 1に設定すると、統計情報は、最初のDepthディレクトリごとに収集される。

   .. note:

      値の制限はありませんが数万個以上のディレクトリの統計情報を収集する場合、ややもするメモリの問題を引き起こすことができる。

-  ``<DirDepthAccum>``

   ディレクトリごとの統計情報を収集する際に、親ディレクトリの統計情報合算するかどうかを設定する。
   ``<DirDepth>`` が0であれば、この設定は無視される。

   - ``OFF (基本)`` 上位ディレクトリに統計を合算していない。

   - ``ON`` 上位ディレクトリに統計を合算する。

   たとえば、 ``<DirDepth>`` が2であり、すべてのディレクトリに同じように10ほどのトラフィックが発生していると仮定する。
   ``<DirDepthAccum>`` が ``OFF`` であれば、左図のように、トラフィックが発生しているディレクトリ別に統計が収集される。
   ``ON`` であれば、右図のようにサブディレクトリのすべての統計情報が親ディレクトリに累積される。

   .. figure:: img/stats_dirdepth.jpg
      :align: center

      親ディレクトリの累積統計

   たとえば、/ imgディレクトリはサブディレクトリのトラフィックと自分のトラフィックを加えた30の統計値に持ち、このトラフィックは、親ディレクトリに含まれる。

-  ``<HttpsTraffic>``

   - ``OFF (基本)`` HTTPSトラフィックをSSL統計のみを収集する。

   - ``ON`` HTTPSトラフィックをSSLとHTTPの両方の統計にように収集する。

   基本的にはSSL層を通過すると、別のSSL統計に収集する。 HTTPSの場合、上位プロトコルで、HTTPで処理されるため、より細かい統計情報の収集が可能である。 しかし、SSL統計およびHTTP統計の両方に重複して統計情報の収集となりますのでHTTP統計だけ信頼することを推奨する。

-  ``<ClientLocal>``

   LoopbackクライアントとSTON区間のトラフィックを統計に集計する。

   - ``OFF (基本)`` 集計していない。

   - ``ON`` 集計する。

-  ``<OriginLocal>``

   STON区間とLoopbackソースサーバー区間のトラフィックを統計に集計する。

   - ``OFF (基本)`` 集計していない。

   - ``ON`` 集計する。


ホスト総合統計
====================================

ホスト統計情報は、最も上位概念の統計にサービスするすべての仮想ホストの統計を総合する。 などの統計をJSONとXML形式で提供する。 ::

   {                                            <Host
     "Host":                                      Version="2.0.0"
     {                                            Name="localhost"
       "Version":"2.0.0",                         State="Healthy"
       "Name":"localhost",                        Uptime="155986"
       "State":"Healthy",                         OriginSession="32"
       "Uptime":155996,                           OriginActiveSession="20"
       "OriginSession":33,                        OriginInbound="1140741"
       "OriginActiveSession":20,                  OriginOutbound="10059"
       "OriginInbound":688177,                    OriginReqCount="42"
       "OriginOutbound":14184,                    OriginResTotalCount="42"
       "OriginReqCount":62,                       OriginResTotalTimeRes="5071"
       "OriginResTotalCount":62,                  OriginResTotalTimeComplete="10288"
       "OriginResTotalTimeRes":2375,              OriginRes2xxCount="19"
       "OriginResTotalTimeComplete":2509,         OriginRes2xxTimeRes="9989"
       "OriginRes2xxCount":54,                    OriginRes2xxTimeComplete="21521"
       "OriginRes2xxTimeRes":2327,                OriginRes3xxCount="23"
       "OriginRes2xxTimeComplete":2481,           OriginRes3xxTimeRes="1008"
       "OriginRes3xxCount":8,                     OriginRes3xxTimeComplete="1008"
       "OriginRes3xxTimeRes":2700,                OriginRes4xxCount="0"
       "OriginRes3xxTimeComplete":2700,           OriginRes4xxTimeRes="0"
       "OriginRes4xxCount":0,                     OriginRes4xxTimeComplete="0"
       "OriginRes4xxTimeRes":0,                   OriginRes5xxCount="0"
       "OriginRes4xxTimeComplete":0,              OriginRes5xxTimeRes="0"
       "OriginRes5xxCount":0,                     OriginRes5xxTimeComplete="0"
       "OriginRes5xxTimeRes":0,                   ClientSession="165"
       "OriginRes5xxTimeComplete":0,              ClientActiveSession="80"
       "ClientSession":155,                       ClientInbound="14792"
       "ClientActiveSession":80                   ClientOutbound="1981700"
       "ClientInbound":35748,                     ClientReqCount="64"
       "ClientOutbound":972906,                   ClientResTotalCount="64"
       "ClientReqCount":152,                      ClientResTotalTimeRes="5535"
       "ClientResTotalCount":152,                 ClientResTotalTimeComplete="6840"
       "ClientResTotalTimeRes":1411,              ClientRes2xxCount="44"
       "ClientResTotalTimeComplete":1479,         ClientRes2xxTimeRes="8050"
       "ClientRes2xxCount":93,                    ClientRes2xxTimeComplete="9943"
       "ClientRes2xxTimeRes":2305,                ClientRes3xxCount="20"
       "ClientRes2xxTimeComplete":2409,           ClientRes3xxTimeRes="5"
       "ClientRes3xxCount":59,                    ClientRes3xxTimeComplete="15"
       "ClientRes3xxTimeRes":3,                   ClientRes4xxCount="0"
       "ClientRes3xxTimeComplete":13,             ClientRes4xxTimeRes="0"
       "ClientRes4xxCount":0,                     ClientRes4xxTimeComplete="0"
       "ClientRes4xxTimeRes":0,                   ClientRes5xxCount="0"
       "ClientRes4xxTimeComplete":0,              ClientRes5xxTimeRes="0"
       "ClientRes5xxCount":0,                     ClientRes5xxTimeComplete="0"
       "ClientRes5xxTimeRes":0,                   RequestHitRatio="6923"
       "ClientRes5xxTimeComplete":0,              ByteHitRatio="4243">
       "RequestHitRatio":6387,                    <HttpCountSum
       "ByteHitRatio":2926,                         OriginReqCount="0"
       "HttpCountSum" :                             OriginResTotalCount="0"
       {                                            OriginRes2xxCount="0"
         "OriginReqCount" : 0,                      OriginRes3xxCount="0"
         "OriginResTotalCount" : 0,                 OriginRes4xxCount="0"
         "OriginRes2xxCount" : 0,                   OriginRes5xxCount="0"
         "OriginRes3xxCount" : 0,                   ClientReqCount="0"
         "OriginRes4xxCount" : 0,                   ClientResTotalCount="0"
         "OriginRes5xxCount" : 0,                   ClientRes2xxCount="0"
         "ClientReqCount" : 0,                      ClientRes3xxCount="0"
         "ClientResTotalCount" : 0,                 ClientRes4xxCount="0"
         "ClientRes2xxCount" : 0,                   ClientRes5xxCount="0"/>
         "ClientRes3xxCount" : 0,                 <HttpRequestHitSum
         "ClientRes4xxCount" : 0,                   TCP_NONE="0"
         "ClientRes5xxCount" : 0                    TCP_HIT="0"
       },                                           TCP_IMS_HIT="0"
       "HttpRequestHitSum" :                        TCP_REFRESH_HIT="0"
       {                                            TCP_REF_FAIL_HIT="0"
         "TCP_NONE" : 0,                            TCP_NEGATIVE_HIT="0"
         "TCP_HIT" : 0,                             TCP_REDIRECT_HIT="0"
         "TCP_IMS_HIT" : 0,                         TCP_MISS="0"
         "TCP_REFRESH_HIT" : 0,                     TCP_REFRESH_MISS="0"
         "TCP_REF_FAIL_HIT" : 0,                    TCP_CLIENT_REFRESH_MISS="0"
         "TCP_NEGATIVE_HIT" : 0,                    TCP_DENIED="0"
         "TCP_REDIRECT_HIT" : 0,                    TCP_ERROR="0"/>
         "TCP_MISS" : 0,                          <FileSystem>
         "TCP_REFRESH_MISS" : 0,                    <RequestHitRatio>0</RequestHitRatio>
         "TCP_CLIENT_REFRESH_MISS" : 0,             <ByteHitRatio>0</ByteHitRatio>
         "TCP_DENIED" : 0,                          <Outbound>0</Outbound>
         "TCP_ERROR" : 0                            <Session>0</Session>
       },                                         </FileSystem>
       "FileSystem":                              <System> ... </System>
       {                                          <VirtualHost> ... </VirtualHost>
         "RequestHitRatio":0,                     <VirtualHost> ... </VirtualHost>
         "ByteHitRatio":0,                        <VirtualHost> ... </VirtualHost>
         "Outbound":0,                            <View> ... </View>
         "Session":0                              <View> ... </View>
       },                                       </Host>
       "System":{ ... },
       "VirtualHost": [ ... ]
       "View": [ ... ]
     }
   }

-  ``Version`` STONバージョン
-  ``Name`` ホスト名。 設定しなかった場合、システムの名前を示している。
-  ``State`` サービスの状態。 （Healthy =通常のサービス、Inactive =ライセンス無効化、Emergency）
-  ``Uptime (単位: 秒)`` サービスの実行時間
-  ``OriginSession`` 元セッション数
-  ``OriginActiveSession`` 転送中のソースセッション数
-  ``OriginInbound (単位: Bytes, 平均)`` 元のサーバーから受信した量
-  ``OriginReqCount (平均)`` 元のサーバーに送信される要求の数
-  ``OriginOutbound (単位: Bytes, 平均)`` 元のサーバーに送信さ量
-  ``OriginResTotalCount (平均)`` 元のサーバーの応答回数
-  ``OriginResTotalTimeRes (単位: 0.01ms, 平均)`` 元のサーバーの応答時間（HTTPリクエスト送信〜HTTP応答の最初の受信）
-  ``OriginResTotalTimeComplete (単位: 0.01ms, 平均)`` 元のサーバーHTTPトランザクションの完了時間（HTTPリクエスト送信〜HTTP応答完了）
-  ``OriginRes2xxCount (平均)`` 元のサーバー2xx応答回数
-  ``OriginRes2xxTimeRes (単位: 0.01ms, 平均)`` 元のサーバー2xx応答時間
-  ``OriginRes2xxTimeComplete (単位: 0.01ms, 平均)`` 元のサーバー2xxトランザクションの完了時間
-  ``OriginRes3xxCount (平均)`` 元のサーバー3xx応答回数
-  ``OriginRes3xxTimeRes (単位: 0.01ms, 平均)`` 元のサーバー3xx応答時間
-  ``OriginRes3xxTimeComplete (単位: 0.01ms, 平均)`` 元のサーバー3xxトランザクションの完了時間
-  ``OriginRes4xxCount (平均)`` 元のサーバー4xx応答回数
-  ``OriginRes4xxTimeRes (単位: 0.01ms, 平均)`` 元のサーバー4xx応答時間
-  ``OriginRes4xxTimeComplete (単位: 0.01ms, 平均)`` 元のサーバー4xxトランザクションの完了時間
-  ``OriginRes5xxCount (平均)`` 元のサーバー5xx応答回数
-  ``OriginRes5xxTimeRes (単位: 0.01ms, 平均)`` 元のサーバー5xx応答時間
-  ``OriginRes5xxTimeComplete (単位: 0.01ms, 平均)`` 元のサーバー5xxトランザクションの完了時間
-  ``ClientSession`` クライアントセッション数
-  ``ClientActiveSession`` 送信しているクライアントセッションの数
-  ``ClientInbound (単位: Bytes, 平均)`` クライアントから受信した量
-  ``ClientOutbound (単位: Bytes, 平均)`` クライアントに送信量
-  ``ClientReqCount (平均)`` クライアントが送信した要求数
-  ``ClientResTotalCount (平均)`` クライアントの応答回数
-  ``ClientResTotalTimeRes (単位: 0.01ms, 平均)`` クライアントの応答時間（HTTPリクエストを受信〜HTTP応答送信）
-  ``ClientResTotalTimeComplete (単位: 0.01ms, 平均)`` クライアントHTTPトランザクションの完了時間（HTTPリクエストを受信〜HTTP応答完了）
-  ``ClientRes2xxCount (平均)`` クライアント2xx応答回数
-  ``ClientRes2xxTimeRes (単位: 0.01ms, 平均)`` クライアント2xx応答時間
-  ``ClientRes2xxTimeComplete (単位: 0.01ms, 平均)`` クライアント2xxトランザクションの完了時間
-  ``ClientRes3xxCount (平均)`` クライアント3xx応答回数
-  ``ClientRes3xxTimeRes (単位: 0.01ms, 平均)`` クライアント3xx応答時間
-  ``ClientRes3xxTimeComplete (単位: 0.01ms, 平均)`` クライアント3xxトランザクションの完了時間
-  ``ClientRes4xxCount (平均)`` クライアント4xx応答回数
-  ``ClientRes4xxTimeRes (単位: 0.01ms, 平均)`` クライアント4xx応答時間
-  ``ClientRes4xxTimeComplete (単位: 0.01ms, 平均)`` クライアント4xxトランザクションの完了時間
-  ``ClientRes5xxCount (平均)`` クライアント5xx応答回数
-  ``ClientRes5xxTimeRes (単位: 0.01ms, 平均)`` クライアント5xx応答時間
-  ``ClientRes5xxTimeComplete (単位: 0.01ms, 平均)`` クライアント5xxトランザクションの完了時間
-  ``RequestHitRatio (単位: 0.01%, 平均)`` Hit率。 キャッシュオブジェクトが生成されており、そのオブジェクトが初期化されている場合Hitある。 逆にキャッシュオブジェクトが存在しないか、そのオブジェクトが元のサーバーから初期化されていない場合Hitで打たない。 応答コードとHit率は関連がない。

   .. figure:: img/stat_filesystem1.png
      :align: center

      HTTPとFile I / Oは、仮想ホストを共有する。

   Apacheを使用してアクセスされるFile I / OのRequestHitRatioは0％になる。 しかし、HTTP Serverの場合、File I / Oによってキャッシュされたファイルがサービスされるので、100％のRequestHitRatioを持つ。 ByteHitRatioの場合、元のInbound比Http outbound、File I / O outboundにそれぞれ計算される。

-  ``ByteHitRatio (単位: 0.01%, 平均)`` 元のサーバーに比べ、クライアント転送速度。 ::

      (クライアント Outbound - ソースサーバー Inbound) / クライアント Outbound

   ソースサーバーがはるかに速い速度を持っているか、クライアントセッションがすぐに切断された場合、負になる。

-  ``FileSystem`` 独立FileSystem統計に他の統計数値に収集されない。

   - ``RequestHitRatio (単位: 0.01%, 平均)`` File I / Oを使用したHit率
   - ``ByteHitRatio (単位: 0.01%, 平均)`` 元のサーバーに比べFile I / Oレート
   - ``Outbound (単位: Bytes, 平均)`` File I / Oサービスデータサイズ
   - ``Session (平均)`` File I / O処理中のThreadができ

.. note::

   5分の統計のみ提供されているアイテムです。

   -  ``HttpCountSum`` HTTPトランザクションの合計数
   -  ``HttpRequestHitSum`` キャッシュHIT結果


System統計
====================================

システムとグローバル・リソースの統計情報をJSONとXML形式で提供する。 ::

    "System":                                   <System>
    {                                             <CPU
      "CPU":                                          Kernel="689"
      {                                               User="1316"
        "Kernel":689,                                 Idle="7993"
        "User":1316,                                  ProcKernel="570"
        "Idle":7993,                                  ProcUser="1216"
        "ProcKernel":570,                             Nice="0"
        "ProcUser":1216,                              IOWait="52"
        "Nice":0,                                     IRQ="10"
        "IOWait":52,                                  SoftIRQ="12"
        "IRQ":10,                                     Steal="0" />
        "SoftIRQ":12,                             <Mem Free="5914644" STON="9785800"/>
        "Steal":0                                 <Storage>
      },                                            <Disk
      "Mem":                                        	Path="/cache1"
      {                                             	Status="Normal"
        "Free":5914644,                             	Read="23"
        "STON":9785800                              	ReadMerged="0"
      },                                            	ReadSectors="344"
      "Storage":                                    	ReadTime="117"
      {                                             	Write="24"
        "Disk":                                     	WriteMerged="93"
        [                                           	WriteSectors="936"
          {                                         	WriteTime="256"
            "Path":"/cache1",                       	IOProgress="0"
            "Status":"Normal",                      	IOTime="173"
            "Read":23,                              	IOWeightedTime="373"/>
            "ReadMerged":0,                         <Disk
            "ReadSectors":344,                      	Path="/cache2"
            "ReadTime":117,                         	Status="Normal"
            "Write":24,                             	Read="27"
            "WriteMerged":93,                       	ReadMerged="1"
            "WriteSectors":936,                     	ReadSectors="488"
            "WriteTime":256,                        	ReadTime="144"
            "IOProgress":0,                         	Write="24"
            "IOTime":173,                           	WriteMerged="86"
            "IOWeightedTime":373                    	WriteSectors="880"
          },                                        	WriteTime="254"
          {                                         	IOProgress="0"
            "Path":"/cache2",                       	IOTime="189"
            "Status":"Normal",                      	IOWeightedTime="380"/>
            "Read":27,                            </Storage>
            "ReadMerged":1,                       <ServerSocket
            "ReadSectors":488,                    	Total="42"
            "ReadTime":144,                       	Established="2"
            "Write":24,                            	Accepted="1"
            "WriteMerged":86,                      	Closed="0"/>
            "WriteSectors":880,                   <ClientSocket
            "WriteTime":254,                       	Total="1"
            "IOProgress":0,                        	Established="0"
            "IOTime":189,                          	Connected="0"
            "IOWeightedTime":380                   	Closed="0"/>
          }                                       <TCPSocket
        ]                                          	Established="30"
      },                                           	Timewait="2"
      "ServerSocket":                              	Orphan="0"
      {                                            	Alloc="0"
        "Total":42,                                	Mem="20"/>
        "Established":1,                          <EQ>0</EQ>
        "Accepted":0,                             <RQ>1000000</RQ>
        "Closed":0                                <WaitingFiles2Write>0</WaitingFiles2Write>
      },                                          <ServiceAccess Allow="60" Deny="2"/>
      "ClientSocket":                             <SystemLoadAverage Min1="0" Min5="0" Min15="0"/>
      {                                           <URLRewrite>57</URLRewrite>
        "Total":1,                              </System>
        "Established":0,
        "Connected":0,
        "Closed":0
      },
      "TCPSocket":
      {
        "Established":30,
        "Timewait":2,
        "Orphan":0,
        "Alloc":0,
        "Mem":20
      },
      "EQ":0,
      "RQ":1000000,
      "WaitingFiles2Write":0,
      "ServiceAccess":{"Allow":60, "Deny":2}
      "SystemLoadAverage":
      {
        "Min1":0,
        "Min5":0,
        "Min15":0
      },
      "URLRewrite":57
    }

-  ``CPU (単位: 0.01%)`` CPU使用率。 全体的なCPUの使用率は、Kernel + Userで計算しなければならない。

   - ``Kernel`` CPU(Kernel) の使用量
   - ``User`` CPU(User) の使用量
   - ``Idle`` 使用されていないCPU量
   - ``ProcKernel`` STONが使用するCPU（Kernel）の使用量
   - ``ProcUser`` STONが使用するCPU（User）の使用量
   - ``Nice`` niced processes executing in user mode
   - ``IOWait`` waiting for I/O to complete
   - ``IRQ`` servicing interrupts
   - ``SoftIRQ`` servicing softirqs
   - ``Steal`` involuntary wait

-  ``Mem (単位: Bytes)`` メモリ使用量

   - ``Free`` システムFreeメモリサイズ
   - ``STON`` STONが使用するメモリサイズ

-  ``Disk`` ディスク性能指標

   - ``Path`` ディスクパス
   - ``Status`` ディスクの状態（Normal：正常動作、Invalid：障害に古い関数、Unmounted：管理者によってUnmountさ）
   - ``Read`` 読む成功回数
   - ``ReadMerged`` 読み込みがマージされた回数
   - ``ReadSectors`` 読んだセクタ数
   - ``ReadTime (単位: ms)`` を読む所要時間
   - ``Write`` 執筆成功回数
   - ``WriteMerged`` 書き込みがマージされた回数
   - ``WriteSectors`` 書かれているセクタ数
   - ``WriteTime (単位: ms)`` を送る所要時間
   - ``IOProgress`` 進行中IO数
   - ``IOTime (単位: ms)`` IO所要時間
   - ``IOWeightedTime (単位: ms)`` IO所要時間（重み付け）

-  ``ServerSocket`` サーバソケット（クライアントとSTON区間）について

   - ``Total`` 全サーバソケット数
   - ``Established`` 接続された状態のサーバソケット数
   - ``Accepted`` 新た接続されたサーバソケットができ
   - ``Closed`` 接続が終了されたサーバーソケット数

-  ``ClientSocket`` クライアントソケット（STONと元サーバー区間）について

   - ``Total`` 全クライアントソケット数
   - ``Established`` 接続された状態のクライアントソケット数
   - ``Connected`` 新た接続されたクライアントソケット数
   - ``Closed`` 接続が終了されたクライアントソケット数

-  ``TCPSocket`` システム（OS）が提供するTCPの状態情報

   - ``Established`` Established状態のTCP接続数
   - ``Timewait`` TIME_WAIT状態のTCP接続数
   - ``Orphan`` まだfile handleにattachされていないTCP接続
   - ``Alloc`` 割り当てられたTCP接続
   - ``Mem`` (undocumented)

-  ``EQ`` STON Frameworkでまだ処理されていないEvent数
-  ``RQ`` 最近サービスされたコンテンツを参照キューに格納されたEvent数
-  ``WaitingFiles2Write`` ディスクへの書き込み待機しているファイルの数
-  ``ServiceAccess`` ServiceAccessによって許可（Allow）、拒否（Deny）されたソケット数
-  ``SystemLoadAverage`` System Load Averageの1分/ 5分/ 15分の平均
-  ``URLRewrite`` URL前処理によって変換が成功した回数


仮想ホストの統計
====================================

仮想ホストごとに統計が提供される。 仮想ホストの統計は、HTTP転送（ディレクトリ別）、URL、バイパス、ポートバイパス、SSLに区分される。 ::

   "VirtualHost":                               <VirtualHost
   [                                                Name="image.11st.co.kr"
     {                                              Uptime="155956"
       "Name":"image.11st.co.kr",                   OriginSession="12"
       "Uptime":155966,                             OriginActiveSession="6"
       "OriginSession":12,                          OriginInbound="106914"
       "OriginActiveSession":6,                     OriginOutbound="3238"
       "OriginInbound":169,                         OriginReqCount="42"
       "OriginOutbound":269,                        OriginResTotalCount="13"
       "OriginReqCount":62,                         OriginResTotalTimeRes="1553"
       "OriginResTotalCount":1,                     OriginResTotalTimeComplete="6630"
       "OriginResTotalTimeRes":3300,                OriginRes2xxCount="1"
       "OriginResTotalTimeComplete":3300,           OriginRes2xxTimeRes="3300"
       "OriginRes2xxCount":0,                       OriginRes2xxTimeComplete="69300"
       "OriginRes2xxTimeRes":0,                     OriginRes3xxCount="12"
       "OriginRes2xxTimeComplete":0,                OriginRes3xxTimeRes="1408"
       "OriginRes3xxCount":1,                       OriginRes3xxTimeComplete="1408"
       "OriginRes3xxTimeRes":3300,                  OriginRes4xxCount="0"
       "OriginRes3xxTimeComplete":3300,             OriginRes4xxTimeRes="0"
       "OriginRes4xxCount":0,                       OriginRes4xxTimeComplete="0"
       "OriginRes4xxTimeRes":0,                     OriginRes5xxCount="0"
       "OriginRes4xxTimeComplete":0,                OriginRes5xxTimeRes="0"
       "OriginRes5xxCount":0,                       OriginRes5xxTimeComplete="0"
       "OriginRes5xxTimeRes":0,                     ClientSession="30"
       "OriginRes5xxTimeComplete":0,                ClientActiveSession="12"
       "ClientSession":26,                          ClientInbound="4113"
       "ClientActiveSession":16,                    ClientOutbound="895937"
       "ClientInbound":13968,                       ClientReqCount="64"
       "ClientOutbound":110398,                     ClientResTotalCount="18"
       "ClientReqCount":152,                        ClientResTotalTimeRes="666"
       "ClientResTotalCount":52,                    ClientResTotalTimeComplete="4377"
       "ClientResTotalTimeRes":94,                  ClientRes2xxCount="10"
       "ClientResTotalTimeComplete":107,            ClientRes2xxTimeRes="1200"
       "ClientRes2xxCount":1,                       ClientRes2xxTimeComplete="7870"
       "ClientRes2xxTimeRes":4700,                  ClientRes3xxCount="8"
       "ClientRes2xxTimeComplete":4800,             ClientRes3xxTimeRes="0"
       "ClientRes3xxCount":51,                      ClientRes3xxTimeComplete="12"
       "ClientRes3xxTimeRes":3,                     ClientRes4xxCount="0"
       "ClientRes3xxTimeComplete":15,               ClientRes4xxTimeRes="0"
       "ClientRes4xxCount":0,                       ClientRes4xxTimeComplete="0"
       "ClientRes4xxTimeRes":0,                     ClientRes5xxCount="0"
       "ClientRes4xxTimeComplete":0,                ClientRes5xxTimeRes="0"
       "ClientRes5xxCount":0,                       ClientRes5xxTimeComplete="0"
       "ClientRes5xxTimeRes":0,                     RequestHitRatio="10000"
       "ClientRes5xxTimeComplete":0,                ByteHitRatio="8806">
       "RequestHitRatio":10000,                   <FileSystem>
       "ByteHitRatio":9984,                         <RequestHitRatio>0</RequestHitRatio>
       "FileSystem":                                <ByteHitRatio>0</ByteHitRatio>
       {                                            <Outbound>0</Outbound>
         "RequestHitRatio":0,                       <Session>0</Session>
         "ByteHitRatio":0,                        </FileSystem>
         "Outbound":0,                            <Memory>784786700</Memory>.
         "Session":0                              <SecuredMemory>0</SecuredMemory>.
       },                                         <Disk> ... </Disk>
       "Memory":785740769,                        <Session> ... </Session>
       "SecuredMemory":0,                         <Dims> ... </Dims>
       "Disk": { ... },                           <Compression> ... </Compression>
       "Session": { ... },                        <File Total="458278" Opened="15" Instance="458292"/>
       "Dims": { ... },                           <Cached> ... </Cached>
       "Compression": { ... },                    <CacheFileEvent> ... </CacheFileEvent>
       "FileTotal":458308,                        <WaitingFiles2Delete>1087593</WaitingFiles2Delete>
       "FileOpened":15,                           <CacheFileEvent Create=\"%u\" Swap=\"%u\" Erase=\"%u\" Purge=\"%u\" Expire=\"%u\" />
       "FileInstance":458320,                     <ClientHttpReqBypass Sum="8100">27</ClientHttpReqBypass>
       "Cached": { ... },                         <ClientHttpReqDenied Sum="400">1</ClientHttpReqDenied>
       "CacheFileEvent": { ... },                 <OriginTraffic> ... </OriginTraffic>
       "WaitingFiles2Delete":1087595,             <PortBypass> ... </PortBypass>
       "ClientHttpReqBypassSum":8100,             <ClientTraffic> ... </ClientTraffic>
       "ClientHttpReqBypass":27,                  <UrlBypass> ... </UrlBypass>
       "ClientHttpReqDeniedSum":400,            </VirtualHost>
       "ClientHttpReqDenied":1,                 <VirtualHost> ... </VirtualHost>
       "OriginTraffic": { ... },                <VirtualHost> ... </VirtualHost>
       "PortBypass": { ... },                   <VirtualHost> ... </VirtualHost>
       "ClientTraffic": { ... },
       "UrlBypass": { ... }

     },
     ...
   ]

.. note:

   ※ Name부터 FileSystem까지 호스트 통계와 동일하다.

-  ``Memory (単位: Bytes)`` メモリにロードされたコンテンツの量
-  ``SecuredMemory (単位: Bytes)`` メモリから削除されたコンテンツの量
-  ``Disk`` ディスク情報
-  ``Session`` セッション情報
-  ``Dims`` DIMS変換統計
-  ``Compression`` 圧縮統計
-  ``FileTotal`` ファイル全体数
-  ``FileOpened`` 開いているローカルファイルの数
-  ``FileInstance`` キャッシュファイルの数
-  ``Cached`` キャッシュ情報
-  ``CacheFileEvent`` キャッシュファイルイベント
-  ``WaitingFiles2Delete`` 削除待機しているファイルの数
-  ``ClientHttpReqBypass`` バイパスしたクライアントのHTTP要求の数
-  ``ClientHttpReqDenied`` HTTP要求がブロックされた回数
-  ``OriginTraffic`` ソースサーバーのトラフィックの統計情報
-  ``PortBypass`` ポートバイパストラフィックの統計情報
-  ``ClientTraffic`` クライアントのトラフィックの統計情報
-  ``UrlBypass`` URLマッチングまたは ``<BypassNoCacheRequest>`` を介して、元のサーバーに変換統計されたHTTPトラフィックの統計情報

.. note::

   5分の統計のみ提供されているアイテムです。

   -  ``ClientHttpReqBypassSum`` バイパスされるHTTPリクエストの合計数
   -  ``ClientHttpReqDeniedSum`` DenyされるHTTPリクエストの合計数


ディスク統計
------------------------------

仮想ホストが使用するディスクの統計情報を提供する。 ::

   "Disk":                                      <Disk>
   {                                              <TotalSize>22003701435</TotalSize>
     "TotalSize":22004057982,                     <Create>1</Create>
     "Create":0,                                  <Open>10</Open>
     "Open":1,                                    <Delete>0</Delete>
     "Delete":0,                                  <ReadCount>9</ReadCount>
     "ReadCount":1,                               <ReadSize>735726</ReadSize>
     "ReadSize":104744,                           <WriteCount>1</WriteCount>
     "WriteCount":0,                              <WriteSize>157145</WriteSize>
     "WriteSize":0,                               <Distribution
     "Distribution":                                U1K="45725"
     {                                              U2K="192523"
       "U1K="45725,                                 U4K="137055"
       "U2K="192523,                                U8K="39740"
       "U4K="137055,                                U16K="13408"
       "U8K="39740,                                 U32K="12303"
       "U16K="13408,                                U64K="11462"
       "U32K="12303,                                U128K="2560"
       "U64K="11462,                                U256K="22"
       "U128K="2560,                                U512K="0"
       "U256K="22,                                  U1M="45725"
       "U512K="0,                                   U2M="192523"
       "U1M="45725,                                 U4M="137055"
       "U2M="192523,                                U8M="39740"
       "U4M="137055,                                U16M="13408"
       "U8M="39740,                                 U32M="12303"
       "U16M="13408,                                U64M="11462"
       "U32M="12303,                                U128M="2560"
       "U64M="11462,                                U256M="22"
       "U128M="2560,                                U512M="0"
       "U256M="22,                                  U1G="0"
       "U512M="0,                                   U2G="0"
       "U1G="0,                                     U4G="0"
       "U2G="0,                                     U8G="0"
       "U4G="0,                                     U16G="0"
       "U8G="0,                                     O16G="0" />
       "U16G":0,                                  </Disk>
       "O16G":0

     }
   }

-  ``TotalSize (単位: Bytes)`` ローカルファイルサイズがあり
-  ``Create`` ローカルファイルの作成回数
-  ``Open`` ローカルファイルOpen回数
-  ``Delete`` ローカルファイルの削除回数
-  ``ReadCount`` ローカルファイルからReadした回数
-  ``ReadSize (単位: Bytes)`` ローカルファイルからReadしたサイズ
-  ``WriteCount`` ローカルファイルからWriteした回数
-  ``WriteSize (単位: Bytes)`` ローカルファイルからWriteサイズ
-  ``Distribution`` ローカルファイル人数分布

   - ``U1K`` 1KB 未満のファイル数
   - ``U2K`` 2KB 未満のファイル数
   - ``U4K`` 4KB 未満のファイル数
   - ``U8K`` 8KB 未満のファイル数
   - ``U16K`` 16KB 未満のファイル数
   - ``U32K`` 32KB 未満のファイル数
   - ``U64K`` 64KB 未満のファイル数
   - ``U128K`` 128KB 未満のファイル数
   - ``U256K`` 256KB 未満のファイル数
   - ``U512K`` 512KB 未満のファイル数
   - ``U1M`` 1MB 未満のファイル数
   - ``U2M`` 2MB 未満のファイル数
   - ``U4M`` 4MB 未満のファイル数
   - ``U8M`` 8MB 未満のファイル数
   - ``U16M`` 16MB 未満のファイル数
   - ``U32M`` 32MB 未満のファイル数
   - ``U64M`` 64MB 未満のファイル数
   - ``U128M`` 128MB 未満のファイル数
   - ``U256M`` 256MB 未満のファイル数
   - ``U512M`` 512MB 未満のファイル数
   - ``U1G`` 1GB 未満のファイル数
   - ``U2G`` 2GB 未満のファイル数
   - ``U4G`` 4GB 未満のファイル数수
   - ``U8G`` 8GB 未満のファイル数
   - ``U16G`` 16GB 未満のファイル数
   - ``O16G`` 16GB 以上のファイル数


セッション統計
------------------------------

仮想ホストが使用するディスクの統計情報を提供する。 ::

   "Session":                                   <Session
   {                                              Client="30"
     "Client":30,                                 ActiveClient="20"
     "ActiveClient":20,                           Origin="12"
     "Origin":12,                                 ActiveOrigin="7" />
     "ActiveOrigin":7
   },

-  ``Client`` 完全なHTTPクライアントセッション数
-  ``ActiveClient`` 完全なHTTPクライアントの転送中のセッション数
-  ``Origin`` 全体元のサーバーのセッション数
-  ``ActiveOrigin`` 転送中のソースサーバーセッション数



DIMS統計
------------------------------

DIMSの性能指標を提供する。 ::

   "Dims":                                   <Dims
   {                                           Requests="30"
     "Requests": 30,                           Converted="29"
     "Converted": 29,                          Failed="1"
     "Failed": 1,                              AvgSrcSize="1457969"
     "AvgSrcSize": 1457969,                    AvgDestSize="598831"
     "AvgDestSize": 598831,                    AvgTime="34" />
     "AvgTime": 34
   },

-  ``Requests`` 変換要求数
-  ``Converted`` 変換成功回数
-  ``Failed`` 変換に失敗した回数
-  ``AvgSrcSize (単位: Bytes)`` 元の画像の平均サイズ
-  ``AvgDestSize (単位: Bytes)`` 変換された画像の平均サイズ
-  ``AvgTime (単位: ms)`` 変換所要時間


圧縮統計
------------------------------

圧縮の性能指標を提供する。 ::

   "Compression":                             <Compression
   {                                           Requests="30"
     "Requests": 30,                           Converted="29"
     "Converted": 29,                          Failed="1"
     "Failed": 1,                              AvgSrcSize="1457969"
     "AvgSrcSize": 1457969,                    AvgDestSize="598831"
     "AvgDestSize": 598831,                    AvgTime="34" />
     "AvgTime": 34
   },

-  ``Requests`` 圧縮要求数
-  ``Converted`` 圧縮成功回数
-  ``Failed`` 圧縮に失敗した回数
-  ``AvgSrcSize (単位: Bytes)`` ソースファイルの平均サイズ
-  ``AvgDestSize (単位: Bytes)`` 圧縮されたファイルの平均サイズ
-  ``AvgTime (単位: ms)`` 圧縮所要時間



ソース統計
------------------------------

STONと元のサーバーの間で発生するトラフィックの統計情報を提供する。 ::

   "OriginTraffic":                             <OriginTraffic>
   {                                              <HttpReqCount Sum="600">2</HttpReqCount>
     "HttpReqCountSum":0,                         <HttpReqHeaderSize>3238</HttpReqHeaderSize>
     "HttpReqCount":0,                            <HttpReqBodySize>0</HttpReqBodySize>
     "HttpReqHeaderSize":269,                     <HttpResHeaderSize>2020</HttpResHeaderSize>
     "HttpReqBodySize":0,                         <HttpResBodySize>104894</HttpResBodySize>
     "HttpResHeaderSize":169,                     <Response>
     "HttpResBodySize":0,                           <ResTotal>
     "Response":                                      <Count Sum="8100">13</Count>
     {                                                <Completed Sum="8100">12</Completed>
       "ResTotal":                                    <TimeRes>1553</TimeRes>
       {                                              <TimeComplete>6630</TimeComplete>
         "CountSum":0,                              </ResTotal>
         "Count":1,                                 <Res2xx>
         "CompletedSum":0,                            <Count Sum="8100">1</Count>
         "Completed":1,                               <Completed Sum="8100">1</Completed>
         "TimeRes":3300,                              <TimeRes>3300</TimeRes>
         "TimeComplete":3300                          <TimeComplete>69300</TimeComplete>
       },                                           </Res2xx>
       "Res2xx":                                    <Res3xx>
       {                                              <Count Sum="8100">12</Count>
         "CountSum":0,                                <Completed Sum="8100">11</Completed>
         "Count":0,                                   <TimeRes>1408</TimeRes>
         "CompletedSum":0,                            <TimeComplete>1408</TimeComplete>
         "Completed":0,                             </Res3xx>
         "TimeRes":0,                               <Res4xx>
         "TimeComplete":0                             <Count Sum="8100">0</Count>
       },                                             <Completed Sum="8100">0</Completed>
       "Res3xx":                                      <TimeRes>0</TimeRes>
       {                                              <TimeComplete>0</TimeComplete>
         "CountSum":0,                              </Res4xx>
         "Count":1,                                 <Res5xx>
         "CompletedSum":0,                            <Count Sum="8100">0</Count>
         "Completed":1,                               <Completed Sum="8100">0</Completed>
         "TimeRes":3300,                              <TimeRes>0</TimeRes>
         "TimeComplete":3300                          <TimeComplete>0</TimeComplete>
       },                                           </Res5xx>
       "Res4xx":                                    <ConnectTimeout Sum="8100">0</ConnectTimeout>
       {                                            <ReceiveTimeout Sum="8100">0</ReceiveTimeout>
         "CountSum":0,                              <Close Sum="8100">0</Close>
         "Count":0,                               </Response>
         "CompletedSum":0,                        <Connect>
         "Completed":0,                             <Count>0</Count>
         "TimeRes":0,                               <AvgDNSQueryTime>0</AvgDNSQueryTime>
         "TimeComplete":0                           <AvgConnTime>0</AvgConnTime>
       },                                         </Connect>
       "Res5xx":                                </OriginTraffic>
       {
         "CountSum":0,
         "Count":0,
         "CompletedSum":0,
         "Completed":0,
         "TimeRes":0,
         "TimeComplete":0
       },
       "ConnectTimeoutSum":0,
       "ConnectTimeout":0,
       "ReceiveTimeoutSum":0,
       "ReceiveTimeout":0,
       "CloseSum":0,
       "Close":0
     },
     "Connect":
     {
       "Count":0,
       "AvgDNSQueryTime":0,
       "AvgConnTime":0
     }
   },

-  ``HttpReqCount`` 元サーバーに送信されるHTTPリクエストの数
-  ``HttpReqHeaderSize (単位: Bytes)`` ソースサーバーに送信されるHTTPヘッダーのサイズ
-  ``HttpReqBodySize (単位: Bytes)`` ソースサーバーに送信されるHTTP Bodyサイズ
-  ``HttpResHeaderSize (単位: Bytes)`` ソースサーバーから受信したHTTPヘッダーのサイズ
-  ``HttpResBodySize (単位: Bytes)`` ソースサーバーから受信したHTTP Bodyサイズ
-  ``Response`` 元サーバーから送信される応答 (ResXXX)

   -  ``Count`` 応答回数
   -  ``Completed`` 正常に送信完了したHTTPトランザクションの数
   -  ``TimeRes`` HTTP応答時間
   -  ``TimeComplete`` HTTPトランザクションの完了時間

-  ``Response`` その他のフィールド

   -  ``ConnectTimeout`` 接続に失敗し
   -  ``ReceiveTimeout`` 伝送遅延
   -  ``Close`` 転送中、ソースサーバーから最初のソケットを終了

-  ``Connect`` ソースサーバー接続の統計情報

   -  ``Count`` 接続回数
   -  ``AvgDNSQueryTime (単位: 0.01ms)`` 平均DNSクエリ時間
   -  ``AvgConnTime (単位: 0.01ms)`` の平均接続時間（TCP SYN送信〜TCP SYN ACK受信）

.. note::

   5分の統計のみ提供されているアイテムです。

   -  ``HttpReqCountSum`` HTTPリクエストの合計回数
   -  ``CountSum`` HTTP応答の合計回数
   -  ``CompletedSum`` 完了したHTTPトランザクションの合計回数
   -  ``ConnectTimeoutSum`` ソースサーバー接続に失敗し、合計回収
   -  ``ReceiveTimeoutSum`` ソースサーバーの伝送遅延の合計回数
   -  ``CloseSum`` 元サーバーで最初に接続を終了した総回数



ポートバイパス統計
------------------------------

``<PortBypass>`` を介して発生したトラフィックの統計情報を提供する。 ::

   "PortBypass":                                            <PortBypass SrcPort="1935" DestPost="1935">
   [                                                          <Session>0</Session>
     {                                                        <Hit Established="0"
       "SrcPort":1935, "DestPort":1935, "Session":0,               ClientClosed="0"
       "Hit":                                                      OriginClosed="0"
       {                                                           OriginCT="0" />
         "Established":0, "ClientClosed":0,                   <ClientTraffic In="0" Out="0"/>
         "OriginClosed":0, "OriginCT":0                       <OriginTraffic In="0" Out="0"/>
       },                                                   </PortBypass>
       "ClientTraffic": { "In":0, "Out":0 },                <PortBypass SrcPort="1936" DestPost="1936">
       "OriginTraffic": { "In":0, "Out":0 }                   <Session>17</Session>
     },                                                       ...
     {                                                      </PortBypass>
       "SrcPort":1936, "DestPort":1936, "Session":17,
       ...
     }
   ],


-  ``SrcPort/DestPort`` バイパスするSTONポート/ソースサーバーのポート
-  ``Session`` 現在接続しているセッション数
-  ``Hit`` バイパス接続統計

   -  ``Established`` 成立した接続数
   -  ``ClientClosed`` クライアントが接続を終了した回数
   -  ``OriginClosed`` 元のサーバーで接続を終了した回数
   -  ``OriginCT`` ソースサーバー接続に失敗した回数

-  ``ClientTraffic (単位: Bytes)`` クライアントのトラフィック (In=Inbound, Out=Outbound)
-  ``OriginTraffic (単位: Bytes)`` ソースサーバーのトラフィック (In=Inbound, Out=Outbound)



.. _monitoring_stats_vhost_client:

クライアント統計
------------------------------

クライアントのトラフィックはディレクトリ毎の統計の設定かどうかによって、「Traffic」がマルチで表現される。ディレクトリ毎の統計情報を設定しなかった場合、すべてのトラフィックは、ルート（/）に集計される。ディレクトリの統計情報が設定されている場合は、ルート（/）と、トラフィックが発生したディレクトリのみ提供される。 ::

   "ClientTraffic":                             <ClientTraffics Depth="0" Accum="OFF" HttpsTraffic="OFF">
   {                                              <TrafficCount>1</TrafficCount>
     "Depth":0,                                   <Traffic RequestHitRatio="0">
     "Accum":"OFF",                                 <Path>/</Path>
     "HttpsTraffic":"OFF",                          <HttpReqCount Sum="0">0</HttpReqCount>
     "TrafficCount":1,                              <HttpReqHeaderSize>4113</HttpReqHeaderSize>
     "Traffic":                                     <HttpReqBodySize>0</HttpReqBodySize>
     [                                              <HttpResHeaderSize>3066</HttpResHeaderSize>
       {                                            <HttpResBodySize>892871</HttpResBodySize>
         "RequestHitRatio" : 9984,                  <Response>
         "Path":"/",                                  <ResTotal>
         "HttpReqCountSum":0,                           <Count Sum="0">18</Count>
         "HttpReqCount":100,                            <Completed Sum="0">18</Completed>
         "HttpReqHeaderSize":13968,                     <TimeRes>666</TimeRes>
         "HttpReqBodySize":0,                           <TimeComplete>4377</TimeComplete>
         "HttpResHeaderSize":5654,                    </ResTotal>
         "HttpResBodySize":104744,                    <Res2xx>
         "Response":                                    <Count Sum="0">10</Count>
         {                                              <Completed Sum="0">10</Completed>
           "ResTotal":                                  <TimeRes>1200</TimeRes>
           {                                            <TimeComplete>7870</TimeComplete>
             "CountSum":0,                            </Res2xx>
             "Count":52,                              <Res3xx>
             "CompletedSum":0,                          <Count Sum="0">8</Count>
             "Completed":52,                            <Completed Sum="0">8</Completed>
             "TimeRes":94,                              <TimeRes>0</TimeRes>
             "TimeComplete":107                         <TimeComplete>12</TimeComplete>
           },                                         </Res3xx>
           "Res2xx":                                  <Res4xx>
           {                                            <Count Sum="0">0</Count>
             "CountSum":0,                              <Completed Sum="0">0</Completed>
             "Count":1,                                 <TimeRes>0</TimeRes>
             "CompletedSum":0,                          <TimeComplete>0</TimeComplete>
             "Completed":1,                           </Res4xx>
             "TimeRes":4700,                          <Res5xx>
             "TimeComplete":4800                        <Count Sum="0">0</Count>
           },                                           <Completed Sum="0">0</Completed>
           "Res3xx":                                    <TimeRes>0</TimeRes>
           {                                            <TimeComplete>0</TimeComplete>
             "CountSum":0,                            </Res5xx>
             "Count":51,                            </Response>
             "CompletedSum":0,                      <SSL RecvSize="0" SendSize="0"/>
             "Completed":51,                        <RequestHit
             "TimeRes":3,                             TCP_NONE="0"
             "TimeComplete":15                        TCP_HIT="0"
           },                                         TCP_IMS_HIT="0"
           "Res4xx":                                  TCP_REFRESH_HIT="0"
           {                                          TCP_REF_FAIL_HIT="0"
             "CountSum":0,                            TCP_NEGATIVE_HIT="0"
             "Count":0,                               TCP_REDIRECT_HIT="0"
             "CompletedSum":0,                        TCP_MISS="0"
             "Completed":0,                           TCP_REFRESH_MISS="0"
             "TimeRes":0,                             TCP_CLIENT_REFRESH_MISS="0"
             "TimeComplete":0                         TCP_DENIED="0"
           },                                         TCP_ERROR="0"/>
           "Res5xx":                                <RequestHitSum
           {                                          TCP_NONE="0"
             "CountSum":0,                            TCP_HIT="0"
             "Count":0,                               TCP_IMS_HIT="0"
             "CompletedSum":0,                        TCP_REFRESH_HIT="0"
             "Completed":0,                           TCP_REF_FAIL_HIT="0"
             "TimeRes":0,                             TCP_NEGATIVE_HIT="0"
             "TimeComplete":0                         TCP_REDIRECT_HIT="0"
           }                                          TCP_MISS="0"
         },                                           TCP_REFRESH_MISS="0"
         "SSL":                                       TCP_CLIENT_REFRESH_MISS="0"
         {                                            TCP_DENIED="0"
           "RecvSize":0,                              TCP_ERROR="0"/>
           "SendSize":0                           </Traffic>
         },                                       <FileSystem>
         "RequestHit":                              <GetAttr
         {                                            TimeRes="0"
           "TCP_NONE":0,                              FileCount="0"
           "TCP_HIT":0,                               DirCount="0"
           "TCP_IMS_HIT":0,                           FailCount="0">0</GetAttr>
           "TCP_REFRESH_HIT":0,                     <Open TimeRes="0">0</Open>
           "TCP_REF_FAIL_HIT":0,                    <Read
           "TCP_NEGATIVE_HIT":0,                      TimeRes="0"
           "TCP_REDIRECT_HIT":0,                      BufferSize="0"
           "TCP_MISS":0,                              BufferFilled="0">0</Read>
           "TCP_REFRESH_MISS":0,                    <RequestHit
           "TCP_CLIENT_REFRESH_MISS":0,               TCP_NONE="0"
           "TCP_DENIED":0,                            TCP_HIT="0"
           "TCP_ERROR":0                              TCP_IMS_HIT="0"
         },                                           TCP_REFRESH_HIT="0"
         "RequestHitSum":                             TCP_REF_FAIL_HIT="0"
         {                                            TCP_NEGATIVE_HIT="0"
           "TCP_NONE":0,                              TCP_REDIRECT_HIT="0"
           "TCP_HIT":0,                               TCP_MISS="0"
           "TCP_IMS_HIT":0,                           TCP_REFRESH_MISS="0"
           "TCP_REFRESH_HIT":0,                       TCP_CLIENT_REFRESH_MISS="0"
           "TCP_REF_FAIL_HIT":0,                      TCP_DENIED="0"
           "TCP_NEGATIVE_HIT":0,                      TCP_ERROR="0"/>
           "TCP_REDIRECT_HIT":0,                    <RequestHitSum
           "TCP_MISS":0,                              TCP_NONE="0"
           "TCP_REFRESH_MISS":0,                      TCP_HIT="0"
           "TCP_CLIENT_REFRESH_MISS":0,               TCP_IMS_HIT="0"
           "TCP_DENIED":0,                            TCP_REFRESH_HIT="0"
           "TCP_ERROR":0                              TCP_REF_FAIL_HIT="0"
         },                                           TCP_NEGATIVE_HIT="0"
         "FileSystem":                                TCP_REDIRECT_HIT="0"
         {                                            TCP_MISS="0"
           "GetAttr" :                                TCP_REFRESH_MISS="0"
           {                                          TCP_CLIENT_REFRESH_MISS="0"
             "TimeRes" : 0,                           TCP_DENIED="0"
             "FileCount" : 0,                         TCP_ERROR="0"/>
             "DirCount" : 0,                      </FileSystem>
             "FailCount" : 0,                   </ClientTraffics>
             "TotalCount" : 0
           },
           "Open" :
           {
             "TimeRes" : 0,
             "Count" : 0
           },
           "Read" :
           {
             "TimeRes" : 0,
             "BufferSize" : 0,
             "BufferFilled" : 0,
             "Count" : 0
           },
           "RequestHit":
           {
             "TCP_NONE":0,
             "TCP_HIT":0,
             "TCP_IMS_HIT":0,
             "TCP_REFRESH_HIT":0,
             "TCP_REF_FAIL_HIT":0,
             "TCP_NEGATIVE_HIT":0,
             "TCP_REDIRECT_HIT":0,
             "TCP_MISS":0,
             "TCP_REFRESH_MISS":0,
             "TCP_CLIENT_REFRESH_MISS":0,
             "TCP_DENIED":0,
             "TCP_ERROR":0
           },
           "RequestHitSum":
           {
             "TCP_NONE":0,
             "TCP_HIT":0,
             "TCP_IMS_HIT":0,
             "TCP_REFRESH_HIT":0,
             "TCP_REF_FAIL_HIT":0,
             "TCP_NEGATIVE_HIT":0,
             "TCP_REDIRECT_HIT":0,
             "TCP_MISS":0,
             "TCP_REFRESH_MISS":0,
             "TCP_CLIENT_REFRESH_MISS":0,
             "TCP_DENIED":0,
             "TCP_ERROR":0
           }
         }
       }
     ]
   }

-  ``Depth`` 統計情報を収集するディレクトリDepth
-  ``Accum`` ディレクトリの統計情報が設定されている場合、サブディレクトリの統計情報を親ディレクトリに累積させる設定
-  ``HttpsTraffic`` HTTPSトラフィックをHTTPトラフィックに重複して集計する設定
-  ``TrafficCount`` 集計されたトラフィックのカウント
-  ``Traffic`` ディレクトリ毎の統計情報です。ルート（/）は常に存在する。

   -  ``Path`` サービスディレクトリ
   -  ``HttpReqCount(単位: Bytes)`` クライアントが送信したHTTPリクエスト数
   -  ``HttpReqHeaderSize(単位: Bytes)`` クライアントが送信したHTTPリクエストヘッダサイズ
   -  ``HttpReqBodySize(単位: Bytes)`` クライアントが送信したHTTPリクエストBodyサイズ
   -  ``HttpResHeaderSize(単位: Bytes)`` STONが送信されるHTTP応答ヘッダーのサイズ
   -  ``HttpResBodySize(単位: Bytes)`` STONが送信されるHTTPレスポンスBodyサイズ
   -  ``Response`` STONが送信応答

      -  ``Count`` 応答回数
      -  ``Completed`` 通常送信完了したHTTPトランザクションの数
      -  ``TimeRes`` HTTP応答時間
      -  ``TimeComplete`` HTTPトランザクションの完了時間

-  ``SSL(単位: Bytes)`` HTTPSトラフィック（RecvSize =受信サイズ、SendSize =送信サイズ）
-  ``RequestHit``  キャッシュHIT結
-  ``FileSystem`` FileSystemアクセス

   -  ``GetAttr`` getattr関数呼び出し回数と応答時間。（FileCount：File応答、DirCount：Dir応答、FailCount：失敗応答）
   -  ``Open`` open関数の呼び出し回数と応答時間
   -  ``Read`` read関数の呼び出し回数と応答時間、要求のサイズ（BufferSize）と応答のサイズ（BufferFilled）
   -  ``RequestHit`` （File I / Oアクセス）キャッシュHIT結果


.. note::

   5分の統計のみ提供されているアイテムです。

   -  ``HttpReqCountSum`` HTTPリクエストの合計回数
   -  ``CountSum`` HTTP応答の合計回数
   -  ``CompletedSum`` 完了したHTTPトランザクションの合計回数
   -  ``RequestHitSum`` キャッシュHIT結果



View
====================================

Viewは、仮想ホストを一つにまとめ、統計を抽出する方式である。Databaseの複数Tableをあたかも一つであるかのように扱うViewから取った概念である。構成は次のように非常に簡単である。 ::

   # vhosts.xml

   <Vhosts>
     <Vhost> ... </Vhost>
     <Vhost> ... </Vhost>
     ... (생략) ...
     <View Name="SK">
       <Vhost>...</Vhost>
       <Vhost>...</Vhost>
     </View>
     <View Name="KT">
       <Vhost>...</Vhost>
       <Vhost>...</Vhost>
       <Vhost>...</Vhost>
     </View>
     <View Name="LG">
       <Vhost>...</Vhost>
       <Vhost>...</Vhost>
     </View>
   </Vhosts>

存在しない仮想ホストでViewを構成しても構わない。Viewが提供する統計情報は以下の通りである。 ::

-  Realtime XML/JSON
-  SNMP - cache(1.3.6.1.4.1.40001.1.4).10 ~ 12

理解を助けるためにViewが必要例を挙げてみよう。リュホンジン、署長ホーン、薄紙ソングそれぞれ自分の好きなスポーツコミュニティサイトを運営している。 ::

   # vhosts.xml

   <Vhosts>
     <Vhost Name="baseball.com"> ... </Vhost>
     <Vhost Name="basketball.com"> ... </Vhost>
     <Vhost Name="football.com"> ... </Vhost>
   </Vhosts>

普段親しみがあった三人は意気投合し、スポーツの総合コミュニティサービスをオープンすることを決定した。ドメインもすべてのサービスを合わせることができるsports.comに決めた。開発/運営チームが解決しなければならミッションは以下の通りである。

- 統合サービスはsports.comとする。
- 既存のユーザーのために、既存のドメインとサービスはそのまま維持する。
- 開発チームは、統合する。運営チームは統合する。
- メイン（最初のページ）のみ、新規開発する。リンクを介して既存のサービスを利用する。
- 予算がない。人がいない。時間がない。精神がない。
- すでにすべての購入手続きが終わった。

このすべての要件を処理する現実的な方法で開発チームは、次のように1番目のディレクトリに既存のドメインを指定するルールを使用することを決定した。 ::

   # 既存のサービス
   http://baseball.com/standing/list.html
   http://basketball.com/stats/2014/view.html
   http://football.com/player/messi.php

   # 統合サービス
   http://sports.com/baseball/standing/list.html
   http://sports.com/basketball/stats/2014/view.html
   http://sports.com/football/player/messi.php

URLの前処理を使用すると、簡単に設定することができる。 ::

   # vhosts.xml

   <Vhosts>
     <Vhost Name="baseball.com"> ... </Vhost>
     <Vhost Name="basketball.com"> ... </Vhost>
     <Vhost Name="football.com"> ... </Vhost>
     <URLRewrite>
       <Pattern>sports.com/(.*)/(.*)</Pattern>
       <Replace>#1.com/#2</Replace>
     </URLRewrite>
   </Vhosts>

統合された運営チームでは、現在、それぞれのサービスだけでなく、統合されたサービス（交通、セッション、応答コードなど）にも監視する必要がある。ほとんどSNMPに慣れている管理者であり、統合された指標を得るためには、次のようにViewを構成する。

.. figure:: img/view1.png
   :align: center

::

   # vhosts.xml

   <Vhosts>
      <Vhost Name="baseball.com"> ... </Vhost>
      <Vhost Name="basketball.com"> ... </Vhost>
      <Vhost Name="football.com"> ... </Vhost>
      <URLRewrite>
         <Pattern>sports.com/(.*)/(.*)</Pattern>
         <Replace>#1.com/#2</Replace>
      </URLRewrite>
      <View Name="sports.com">
         <Vhost>baseball.com</Vhost>
         <Vhost>basketball.com</Vhost>
         <Vhost>football.com</Vhost>
      </View>
   </Vhosts>

以上の例からわかるようにURL RewriteとViewの組み合わせは、既存のサイトを一つにまとめてサービスするときに効果的である。


View統計
----------------------------

仮想ホストと同じ統計を提供する。次のようにタグ名のみ異なる。 ::

   "View":                                  <View ...>
   [                                           ...
     { ... },                               </View>
     { ... },                               <View> ... </View>
   ]                                        <View> ... </View>


.. _api-monitoring-vhostlist:

仮想ホストのリスト参照
====================================

仮想ホストのリストを照会する。 ::

    http://127.0.0.1:10040/monitoring/vhostslist

結果は、JSON形式で提供される。 ::

    {
        "version": "2.0.0",
        "method": "vhostslist",
        "status": "OK",
        "result": [ "www.example.com","www.winesoft.com", "site1.com" ]
    }






.. _api-monitoring-fileinfo:

キャッシュ情報
====================================

キャッシュしているファイルの状態を監視する。一般的に、ファイルは、URLで区切られますが、同じURLに他のオプション(i.e. Accept-Encodingなど)が存在する場合、複数のファイルが存在することができる。 ::

    http://127.0.0.1:10040/monitoring/fileinfo?url=example.com/sample.dat

結果は、JSON形式で提供される。以下は、/sample.datファイルの情報を閲覧した結果である。 ::

    {
        "version": "2.0.0",
        "method": "fileinfo",
        "status": "OK",
        "result":
        [
            {
                "URI": "/sample.dat",
                "Accept-Encoding": "N",
                "RefCount": 0,
                "Disk-Index": 0,
                "Size": 2100267,
                "FID": 24267,
                "LocalPath": "/cache1/example.com/000i/q3.bin",
                "File-Opened ": "N",
                "File-Updating": "-",
                "Downloader-Count": "0",
                "LastAccess": "[ 2012.09.03 14:29:50, -2 ]",
                "UpdateTime": "[ 2012.09.03 13:53:43, -2169 ]",
                "TTL-Left": "[ 2012.10.03 13:53:43, 2589831 ]",
                "ResponseCode": 200,
                "ContentType": "text/plain",
                "LastModifiedTime": "[ 2010.11.22 20:31:47, -56224685 ]",
                "ExpireTime": "[ 0, 0 ]",
                "CacheControl": "not-specified",
                "ETag": "502dd614:200c2b",
                "CustomTTL": 0,
                "NoMoreExist": "N",
                "LocalFileExist": "Y",
                "SmallFile": "N",
                "State": "Cached",
                "Deleted": "N",
                "AddedSize": "Y",
                "TransferEncoding": "N",
                "Compression": "-",
                "Purge": "N",
                "Ignore-IMS ": "N",
                "Redirect-Location ": "-",
                "Content-Disposition ": "-",
                "NoCache": "N"
            }
        ]
    }

-  ``URI`` ファイルURI
-  ``Accept-Encoding`` ("Y" or "N") Accept-Encodingをサポートする場合は "Y"
-  ``RefCount`` ファイルの参照カウント
-  ``Size`` (Bytes) ファイルサイズ
-  ``Disk-Index`` (0から始まる) 保存されたディスクのインデックス
-  ``FID`` ファイルID
-  ``LocalPath`` ローカルパス
-  ``File-Opened`` ("Y" or "N") ローカルファイルを開き場合、 "Y"
-  ``File-Updating`` ファイルを更新している場合、更新するオブジェクトのポインタが明示
-  ``Downloader-Count`` 元のサーバーでは、このファイルをダウンロードする現在のセッションの数
-  ``LastAccess`` (最後のアクセス時間、最後のアクセス時間 - 現在時刻) [ 2012.09.03 14:29:50, -2 ]の意味は、2012.09.03 14:29:50にアクセスされ、現在から2秒前にアクセスされたという意味である。
-  ``UpdateTime`` (更新時間、更新時間 - 現在時刻) ファイルが最後に更新された時間。304 Not Modifiedも時間は更新される。
-  ``TTL-Left`` (有効期限、有効期限 - 現在の時刻) コンテンツの有効期限予定の時間。TTLが残ったら正であり、有効期限が切れたとすれば負の表記される。
-  ``ResponseCode`` ソースサーバーの応答コード
-  ``ContentType`` MIME Type
-  ``LastModifiedTime`` (Last Modified Time, Last Modified Time`` 現在時刻) ソースサーバーが送信Last Modified Time。ソースサーバーが、この値を送信していない場合、0に表示される。
-  ``ExpireTime`` (Expire Time, Expire Time`` 現在時刻）ソースサーバーが送信したExpire Time。ソースサーバーが、この値を送信していない場合、0に表示される。
-  ``CacheControl`` ("no-cache" or "not-specified" or (整数））元のサーバーが送信したCache-Contorl値
-  ``ETag`` STONが生成したETag
-  ``CustomTTL`` カスタムTTL。設定されていない場合0である。
-  ``NoMoreExist`` ("Y" or "N") ファイルを破棄予約されている場合は "Y"
-  ``LocalFileExist`` ("Y" or "N") ローカルにファイルが存在する場合、 "Y" (200 OK以外のファイルは、常に "Y")
-  ``SmallFile`` ("Y" or "N") ファイルを小さなファイルに判断すれば、 "Y" (開発的な理由)
-  ``State`` ("Not Init" or "Cached" or "Error") ファイルの状態
-  ``Deleted`` ("Y" or "N") を削除された場合は "Y" (開発的な理由)
-  ``AddedSize`` ("Y" or "N") のサイズは統計に反映されている場合は "Y" (開発的な理由)
-  ``TransferEncoding`` ("Y" or "N") Transfer-Encodingをサポートする場合は "Y"
-  ``Compression`` 圧縮方式
-  ``Purge`` ("Y" or "N") Purgeたなら "Y"
-  ``Ignore-IMS`` ("Y" or "N") 更新すると、If-Modified-Sinceヘッダを送信しないように設定された場合は "Y"
-  ``Redirect-Location`` Locationヘッダの値
-  ``Content-Disposition`` Content-Disposition ヘッダの値
-  ``NoCache`` ("Y" or "N") 元のサーバーでno-cache応答を与えた場合、 "Y"



.. _api-monitoring-logtrace:

Log Trace
====================================

記録されたログをリアルタイムで受けてみる。Access、Origin、Monitoingログは、仮想ホスト（vhost）を指定しなければならない。 ::

    http://127.0.0.1:10040/monitoring/logtrace/info
    http://127.0.0.1:10040/monitoring/logtrace/deny
    http://127.0.0.1:10040/monitoring/logtrace/sys
    http://127.0.0.1:10040/monitoring/logtrace/originerror
    http://127.0.0.1:10040/monitoring/logtrace/access?vhost=www.site1.com
    http://127.0.0.1:10040/monitoring/logtrace/origin?vhost=www.site1.com
    http://127.0.0.1:10040/monitoring/logtrace/monitoring?vhost=www.site1.com
