.. _snmp:

第11章 SNMP
******************

この章では、SNMP（Simple Network Management Protocol）について取り上げる。
:ref:`monitoring_stats` のすべての数値は、SNMPにも提供される。 だけでなく、さらに細分化された時間の単位とシステムの状態情報まで提供する。 仮想ホストごとにリアルタイムの統計情報と、最大60分まで "分" 単位の平均統計を提供する。

- 別のパッケージが必要ない。
- snmpdを個別に実行していない。
- SNMP v1およびv2cをサポートします。

.. toctree::
   :maxdepth: 2


.. _snmp-var:

変数
====================================

設定やユーザーの意図によって変更されることができる値を[変数名]に指定する。 たとえば、ディスクは複数が存在することができる。 この場合、各ディスクを指す一意の番号が必要であり、入力された順に1から割り当てられる。 このような変数を ``[diskIndex]`` で指定する。

-  ``[diskIndex]``

   Storageに設定されたディスクを意味する。 ::

      # server.xml - <Server><Cache>

      <Storage>
         <Disk>/cache1</Disk>
         <Disk>/cache2</Disk>
         <Disk>/cache3</Disk>
      </Storage>

   上記のように3つのディスクが設定された環境では、/ cache1の
   ``[diskIndex]`` は1、/ cache3の ``[diskIndex]`` は3を有する。 例えば、/ cache1の全容量に対応するOIDはsystem.diskInfo.diskInfoTotalSize.1（1.3.6.1.4.1.40001.1.2.18.1.3）1となる。 最後.1は最初のディスクを意味する。

-  ``[vhostIndex]``

   仮想ホストがロードされるときに自動的に付与される。 ::

      # vhosts.xml

      <Vhosts>
         <Vhost Status="Active" Name="kim.com"> ... </Vhost>
         <Vhost Status="Active" Name="lee.com"> ... </Vhost>
         <Vhost Status="Active" Name="park.com" StaticIndex="10300"> ... </Vhost>
      </Vhosts>

   最初、上記のよう3つの仮想ホストがロードされると、1から順番に  ``[vhostIndex]`` が付与される。 以降、仮想ホストは、  ``[vhostIndex]`` を記憶し、仮想ホストが削除されても  ``[vhostIndex]`` は変わらない。 仮想ホストの削除と追加が同時に発生した場合、削除は、最初に動作し、新規追加された仮想ホストは、空の  ``[vhostIndex]`` を与えられる。

   .. figure:: img/snmp_vhostindex.png
      :align: center

      ``[vhostIndex]`` の動作

-  ``[diskMin]`` , ``[vhostMin]``

   時間（分）を意味する。 5は、5分の平均を意味し、60は60分の平均を意味する。 この値は、1（分）から60（分）までの範囲を持ち、0は、リアルタイム（1秒）のデータを意味する。

SNMPは、動的に値が変わることができる項目についてTable構造を使用する。 たとえば、「ディスク全体のサイズ」は、ディスクの数に応じて提供するデータ数が変わるため、Table構造を使用して表現しなければならない。 STONは、すべての仮想ホストに対して「分」単位の統計情報を提供する。 したがって、 ``[vhostMin]`` . ``[vhostIndex]`` という多少難解な表現を提供する。

この表現は、仮想ホストごとに必要な "分" 単位の統計情報を見ることができる利点を持っているが、変数が2つなので、Table構造で表現するのは難しいという短所がある。 このような問題を克服するために  ``[vhostMin]`` のデフォルト値を設定して、SNMPWalkが動作できるようにする。


.. _snmp-conf:

有効
====================================

グローバル設定（server.xml）を介してSNMP動作とACLを設定する。 ::

   # server.xml - <Server><Host>

   <SNMP Port="161" Status="Inactive">
      <Allow>192.168.5.1</Allow>
      <Allow>192.168.6.0/24</Allow>
   </SNMP>

-  ``<SNMP>`` プロパティを介してSNMPの動作を設定する。

   - ``Port (基本: 161)`` SNMPサービスポート

   - ``Status (基本: Inactive)`` SNMPを有効にするには、この値を ``Active`` に設定する。

-  ``<Allow>`` SNMPアクセスを許可するIPアドレスを設定する。
    IPの指定、IPアドレスの範囲を指定、ビットマスク、サブネット上四つの形式をサポートします。 接続したソケットが許可されたIPアドレスがなければ、応答を与えない。



仮想ホスト/ View変数
====================================

SNMPを介して提供される仮想ホスト/ View数と基本時間（分）を設定する。 ::

   # server.xml - <Server><Host>

   <SNMP VHostCount=0, VHostMin=5 ViewCount=0, ViewMin=5 />

-  ``VHostCount (基本: 0)`` 0の場合、存在する仮想ホストまでの応答をする。 0よりも大きい値である場合は、仮想ホストの存在の有無に関係なく、設定された仮想ホストまでの応答である。

-  ``ViewCount (基本: 0)`` Viewに適用します。 ( ``VHostCount`` と同じ)

-  ``VHostMin (基本: 5分, 最大: 60分)``  ``[vhostMin]``  の値を設定する。 0〜60までの値を有する。 0の場合、リアルタイムのデータを提供して、1〜60の間である場合は、その分だけの平均値を提供する。

-  ``ViewMin (基本: 0)`` Viewに適用します。 ( ``VHostMin`` と同じ)

たとえば、3つの仮想ホストが設定されている環境でSNMPWalkの動作が変わる。

- VHostCount=0の場合 ::

    SNMPv2-SMI::enterprises.40001.1.4.2.1.2.1 = STRING: "web.winesoft.co.kr"
    SNMPv2-SMI::enterprises.40001.1.4.2.1.2.2 = STRING: "img.winesoft.co.kr"
    SNMPv2-SMI::enterprises.40001.1.4.2.1.2.3 = STRING: "vod.winesoft.co.kr"

- VHostCount=5の場合 ::

    SNMPv2-SMI::enterprises.40001.1.4.2.1.2.1 = STRING: "web.winesoft.co.kr"
    SNMPv2-SMI::enterprises.40001.1.4.2.1.2.2 = STRING: "img.winesoft.co.kr"
    SNMPv2-SMI::enterprises.40001.1.4.2.1.2.3 = STRING: "vod.winesoft.co.kr"
    SNMPv2-SMI::enterprises.40001.1.4.2.1.2.4 = ""
    SNMPv2-SMI::enterprises.40001.1.4.2.1.2.5 = ""



その他の変数
---------------------

その他の変数を設定する。 ::

   # server.xml - <Server><Host>

   <SNMP GlobalMin="5" DiskMin="5" ConfCount="10" />

-  ``GlobalMin (基本: 5分, 最大: 60分)``  ``[globalMin]``  の値を設定する。

-  ``DiskMin (基本: 5分, 最大: 60分)``  ``[diskMin]``  の値を設定する。

-  ``ConfCount (基本: 10)`` 設定のリストをn個まで閲覧する。 1〜100の間で指定可能である。 1は、現在の反映の設定を意味し、2は、以前の設定を意味する。 100は、現在を基準に99回前の設定を意味する。



Community
====================================

Communityを設定して、許可されたOIDのみアクセス/ブロックするように設定する。 ::

   # server.xml - <Server><Host>

   <SNMP UnregisteredCommunity="Allow">
      <Community Name="example1" OID="Allow">
         <OID>1.3.6.1.4.1.40001.1.4.1</OID>
         <OID>1.3.6.1.4.1.40001.1.4.2</OID>
         <OID>1.3.6.1.4.1.40001.1.4.4</OID>
      </Community>
      <Community Name="example2" OID="Deny">
         <OID>1.3.6.1.4.1.40001.1.4.3.1.11.11.10.1-61</OID>
      </Community>
   </SNMP>

``<SNMP>`` の ``UnregisteredCommunity`` を "Deny"に設定すると、登録されていないCommunity要求は遮断する。

-  ``<Community>`` Communityを設定する。

   - ``Name`` Community 名。

   - ``OID (基本: Allow)`` サブ ``<OID>`` タグの値を設定する。 属性値が ``Allow`` であれば、サブ ``<OID>`` リストのみがアクセス可能である。 逆に属性値が ``Deny`` であれば、サブ<OID>リストには、アクセスが不可能である。


明示的なOID（1.3.6.1.4.1.40001.1.4.4）と範囲（OID 1.3.6.1.4.1.40001.1.4.3.1.11.11.10.1-61）表現が可能である。 OIDを許可/遮断する場合は、サブすべてのOIDに対して同じルールが適用される。



.. _snmp-meta:

meta
====================================

::

   OID = 1.3.6.1.4.1.40001.1.1

メタ情報を提供する。

===== ============= ========= ===========================================
OID   Name          Type      Description
===== ============= ========= ===========================================
.1    manufacture   String    "WineSOFT Inc."
.2    software      String    "STON"
.3    version       String    バージョン
.4    hostname      String    ホスト名
.5    state         String    "Healthy" または "Inactive" または "Emergency"
.6    uptime        Integer   実行時間（秒
.7    admin         String    <Admin> ... </Admin>
.10   Conf          OID       Conf 拡張
===== ============= ========= ===========================================



.. _snmp-meta-conf:

meta.conf
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.1.10

``[confIndex]`` は ``<SNMP>`` の ``ConfCount`` 属性で設定します。
``[confIndex]`` が1の場合は、常に現在適用され設定値を、2の場合は、以前の設定値を意味する。 10であれば、現在の（1）から9の前の設定を意味する。

==================== ======= ======= =============================================================================================
OID                  Name    Type    Description
==================== ======= ======= =============================================================================================
.1. ``[confIndex]``  ID      Integer 設定ID
.2. ``[confIndex]``  Time    Integer 設定時間（Unix時間）
.3. ``[confIndex]``  Type    Integer 設定モード (0 = Unknown, 1 = STON 開始, 2 = /conf/reload, 3 = /conf/upload, 4 = /conf/restore)
.4. ``[confIndex]``  Size    Integer 設定ファイルサイズ
.5. ``[confIndex]``  Hash    String  設定ファイルHashの文字列
.6. ``[confIndex]``  Path    String  設定ファイルの保存パス
.7. ``[confIndex]``  Ver     String  設定時のSTONバージョン
==================== ======= ======= =============================================================================================



.. _snmp-meta-system:

system
====================================

::

   OID = 1.3.6.1.4.1.40001.1.2

STONが動作するシステムの情報を提供する。
``[sysMin]`` 変数は、0〜60分までの値を持ち、リアルタイムまたは必要な時間だけの平均値を提供する。 SNMPWalkで  ``[sysMin]`` は0に設定され、現在の情報を提供する。

=================== ========================================= ======= ===============================================
OID                 Name                                      Type    Description
=================== ========================================= ======= ===============================================
.1. ``[sysMin]``    cpuTotal                                  Integer 全体のCPU使用率 (100%)
.2. ``[sysMin]``                                                      全体のCPU使用率 (10000%)
.3. ``[sysMin]``    cpuKernel                                 Integer	CPU(Kernel) の使用率 (100%)
.4. ``[sysMin]``                                                      CPU(Kernel) の使用率 (10000%)
.5. ``[sysMin]``    cpuUser                                   Integer CPU(User) の使用率 (100%)
.6. ``[sysMin]``                                                      CPU(User) の使用率 (10000%)
.7. ``[sysMin]``    cpuIdle                                   Integer CPU(Idle) の使用率 (100%)
.8. ``[sysMin]``                                                      CPU(Idle) の使用率 (10000%)
.9                  memTotal                                  Integer システム全体のメモリ (KB)
.10. ``[sysMin]``   memUse                                    Integer システムの使用メモリ (KB)
.11. ``[sysMin]``   memFree                                   Integer システムの空きメモリ (KB)
.12. ``[sysMin]``   memSTON                                   Integer STON使用メモリ (KB)
.13. ``[sysMin]``   memUseRatio                               Integer システムメモリの使用率 (100%)
.14. ``[sysMin]``                                                     システムメモリの使用率 (10000%)
.15. ``[sysMin]``   memSTONRatio                              Integer STON メモリ使用率 (100%)
.16. ``[sysMin]``                                                     STON メモリ使用率 (10000%)
.17                 diskCount                                 Integer disk数
.18.1               diskInfo                                  OID     diskInfo拡張
.19.1               diskPerf                                  OID     diskPerf拡張
.20. ``[sysMin]``   cpuProcKernel                             Integer STONが使用するCPU（Kernel）の使用率 (100%)
.21. ``[sysMin]``                                                     STONが使用するCPU（Kernel）の使用率 (10000%)
.22. ``[sysMin]``   cpuProcUser                               Integer STONが使用するCPU（User）の使用率 (100%)
.23. ``[sysMin]``                                                     STONが使用するCPU（User）の使用率 (10000%)
.24. ``[sysMin]``   sysLoadAverage                            Integer Load Average 1分平均 (0.01)
.25. ``[sysMin]``                                                     Load Average 5分平均 (0.01)
.26. ``[sysMin]``                                                     Load Average 15分平均 (0.01)
.27. ``[sysMin]``   cpuNice                                   Integer CPU(Nice) (100%)
.28. ``[sysMin]``                                                     CPU(Nice) (10000%)
.29. ``[sysMin]``   cpuIOWait                                 Integer CPU(IOWait) (100%)
.30. ``[sysMin]``                                                     CPU(IOWait) (10000%)
.31. ``[sysMin]``   cpuIRQ                                    Integer CPU(IRQ) (100%)
.32. ``[sysMin]``                                                     CPU(IRQ) (10000%)
.33. ``[sysMin]``   cpuSoftIRQ                                Integer CPU(SoftIRQ) (100%)
.34. ``[sysMin]``                                                     CPU(SoftIRQ) (10000%)
.35. ``[sysMin]``   cpuSteal                                  Integer CPU(Steal) (100%)
.36. ``[sysMin]``   CPU(Steal)                                Integer (10000%)
.40. ``[sysMin]``   TCPSocket.Established. ``[globalMin]``    Integer Established状態のTCP接続数
.41. ``[sysMin]``   TCPSocket.Timewait. ``[globalMin]``       Integer TIME_WAIT状態のTCP接続数
.42. ``[sysMin]``   TCPSocket.Orphan. ``[globalMin]``         Integer まだfile handleにattachされていないTCP接続
.43. ``[sysMin]``   TCPSocket.Alloc. ``[globalMin]``          Integer 割り当てられたTCP接続
.44. ``[sysMin]``   TCPSocket.Mem. ``[globalMin]``            Integer undocumented
=================== ========================================= ======= ===============================================



.. _snmp-meta-system-diskinfo:

system.diskInfo
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.2.18.1

ディスクの情報を提供する。

======================= ================== =========== =========================================
OID                     Name               Type        Description
======================= ================== =========== =========================================
.2. ``[diskIndex]``     diskInfoPath       String      ディスクパス
.3. ``[diskIndex]``     diskInfoTotalSize  Integer     ディスク全体の容量 (MB)
.4. ``[diskIndex]``     diskInfoUseSize    Integer     ディスク使用量 (MB)
.5. ``[diskIndex]``     diskInfoFreeSize   Integer     ディスクの使用可能量 (MB)
.6. ``[diskIndex]``     diskInfoUseRatio   Integer     ディスク使用率 (100%)
.7. ``[diskIndex]``                                    ディスク使用率 (10000%)
.8. ``[diskIndex]``     diskInfoStatus     String      "Normal" または "Invalid" または "Unmounted"
======================= ================== =========== =========================================



.. _snmp-meta-system-diskperf:

system.diskPerf
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.2.19.1

ディスクパフォーマンスの状態を提供する。

======================================== =========================== ========== ===============================
OID                                      Name                        Type       Description
======================================== =========================== ========== ===============================
.2. ``[diskMin]`` . ``[diskIndex]``      diskPerfReadCount           Integer    読み込み成功回数
.3. ``[diskMin]`` . ``[diskIndex]``      diskPerfReadMergedCount     Integer    読み込みがマージされた回数
.4. ``[diskMin]`` . ``[diskIndex]``      diskPerfReadSectorsCount    Integer    読んだセクタ数
.5. ``[diskMin]`` . ``[diskIndex]``      diskPerfReadTime            Integer    読む時間(ms)
.6. ``[diskMin]`` . ``[diskIndex]``      diskPerfWriteCount          Integer    執筆成功回数
.7. ``[diskMin]`` . ``[diskIndex]``      diskPerfWriteMergedCount    Integer    書き込みがマージされた回数
.8. ``[diskMin]`` . ``[diskIndex]``      diskPerfWriteSectorsCount   Integer    書かれているセクタ数
.9. ``[diskMin]`` . ``[diskIndex]``      diskPerfWriteTime           Integer    書き込み時間(ms)
.10. ``[diskMin]`` . ``[diskIndex]``     diskPerfIOProgressCount     Integer    進行中のIO数
.11. ``[diskMin]`` . ``[diskIndex]``     diskPerfIOTime              Integer    IO 所要時間(ms)
.12. ``[diskMin]`` . ``[diskIndex]``     diskPerfIOTimeWeighted      Integer    IO 所要時間(ms、重み付け)
======================================== =========================== ========== ===============================



.. _snmp-global:

global
====================================

::

   OID = 1.3.6.1.4.1.40001.1.3

STONのすべてのモジュールが共通で使用するリソース情報（ソケット、イベントなど）を提供する。

-  **ServerSocket**

   クライアント〜STON区間。 STONがクライアントの要求を処理する目的で使用されるソケット

-  **ClientSocket**

   STON〜ソースサーバー区間。 STONが元のサーバーに要求を送信する目的で使用されるソケット

===== =========================================== ========== ==================================================
OID   Name                                        Type       Description
===== =========================================== ========== ==================================================
.5    EQ. ``[globalMin]``                         Integer    STON Frameworkでまだ処理されていないEvent数
.6    RQ. ``[globalMin]``                         Integer    最近サービスされたコンテンツを参照キューに格納されたEvent数
.7    waitingFiles2Write. ``[globalMin]``         Integer    書き込み待機中のファイルの数
.10   ServerSocket.Total. ``[globalMin]``         Integer    サーバー全体のソケット数
.11   ServerSocket.Established. ``[globalMin]``   Integer    接続された状態のサーバソケット数
.12   ServerSocket.Accepted. ``[globalMin]``      Integer    新たに接続されたサーバソケットができ
.13   ServerSocket.Closed. ``[globalMin]``        Integer    接続が終了されたサーバーソケット数
.20   ClientSocket.Total. ``[globalMin]``         Integer    全クライアントソケットの数
.21   ClientSocket.Established. ``[globalMin]``   Integer    接続された状態のクライアントソケット数
.22   ClientSocket.Accepted. ``[globalMin]``      Integer    新たに接続されたクライアントソケット数
.23   ClientSocket.Closed. ``[globalMin]``        Integer    接続が終了したクライアントソケット数
.30   ServiceAccess.Allow. ``[globalMin]``        Integer    ServiceAccessによって許可（Allow）されたソケット数
.31   ServiceAccess.Deny. ``[globalMin]``         Integer    ServiceAccessによって拒否（Deny）されたソケット数
===== =========================================== ========== ==================================================



.. _snmp-cache:

cache
====================================

::

    OID = 1.3.6.1.4.1.40001.1.4

キャッシュサービスの統計情報は、仮想ホストごとに詳細に収集/提供される。

====== ============== ========= ============================================================
OID    Name           Type      Description
====== ============== ========= ============================================================
.1     host           OID       ホスト（拡張)
.2     vhostCount     Integer   仮想ホストの数
.3.1   vhost          OID       仮想ホスト別の統計
.4     vhostIndexMax  Integer    ``[vhostIndex]``  最大値。 SNMPWalkはこの数値を基準に動作する。
.10    viewCount      Integer   View数
.11.1  view           OID       View別統計
.12    viewIndexMax   Integer   [viewIndex] 最大値。 SNMPWalkはこの数値を基準に動作する。
====== ============== ========= ============================================================



.. _snmp-cache-host:

cache.host
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.1

ホスト（=すべての仮想ホスト）の情報を提供する。

===== ========= =========== =========================
OID   Name      Type        Description
===== ========= =========== =========================
.2    name      String      ホスト名
.3    status    String      "Healthy" または "Inactive"
.4    uptime    Integer     STON実行時間（秒）
.10   contents  OID         コンテンツ情報（拡張）
.11   traffic   OID         統計（拡張）
===== ========= =========== =========================



.. _snmp-cache-host-contents:

cache.host.contents
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.1.10

ホスト（=すべての仮想ホスト）がサービスするコンテンツ情報を提供する。

====== ================ ========== ============================
OID    Name             Type       Description
====== ================ ========== ============================
.1     memory           Integer    メモリキャッシュサイズ(KB)
.2     filesTotalCount  Integer    サービス中のファイル数
.3     filesTotalSize   Integer    サービス中の全ファイル量(MB)
.10    filesCountU1KB   Integer    1KB未満のファイル数
.11    filesCountU2KB   Integer    2KB未満のファイル数
.12    filesCountU4KB   Integer    4KB未満のファイル数
.13    filesCountU8KB   Integer    8KB未満のファイル数
.14    filesCountU16KB  Integer    16KB未満のファイル数
.15    filesCountU32KB  Integer    32KB未満のファイル数
.16    filesCountU64KB  Integer    64KB未満のファイル数
.17    filesCountU128KB Integer    128KB未満のファイル数
.18    filesCountU256KB Integer    256KB未満のファイル数
.19    filesCountU512KB Integer    512KB未満のファイル数
.20    filesCountU1MB   Integer    1MB未満のファイル数
.21    filesCountU2MB   Integer    2MB未満のファイル数
.22    filesCountU4MB   Integer    4MB未満のファイル数
.23    filesCountU8MB   Integer    8MB未満のファイル数
.24    filesCountU16MB  Integer    16MB未満のファイル数
.25    filesCountU32MB  Integer    32MB未満のファイル数
.26    filesCountU64MB  Integer    64MB未満のファイル数
.27    filesCountU128MB Integer    128MB未満のファイル数
.28    filesCountU256MB Integer    256MB未満のファイル数
.29    filesCountU512MB Integer    512MB未満のファイル数
.30    filesCountU1GB   Integer    1GB未満のファイル数
.31    filesCountU2GB   Integer    2GB未満のファイル数
.32    filesCountU4GB   Integer    4GB未満のファイル数
.33    filesCountU8GB   Integer    8GB未満のファイル数
.34    filesCountU16GB  Integer    16GB未満のファイル数
.35    filesCountO16GB  Integer    16GB以上のファイル数
====== ================ ========== ============================



.. _snmp-cache-host-traffic:

cache.host.traffic
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.1.11

ホスト（=すべての仮想ホスト）のキャッシュサービスとトラフィックの統計情報を提供する。 trafficのすべての統計情報は、最大60分までの平均で提供する。 minは、 '分'を意味し、最大60までの値を有する。 minが省略されたり0であれば、リアルタイムの情報を提供する。

===================== =============== ======= ==============================
OID                   Name            Type    Description
===================== =============== ======= ==============================
.1. ``[vhostMin]``    requestHitRatio Integer Request Hit Ratio(100%)
.2. ``[vhostMin]``                            Request Hit Ratio(10000%)
.3. ``[vhostMin]``    bytesHitRatio   Integer Bytes Hit Ratio(100%)
.4. ``[vhostMin]``                            Bytes Hit Ratio(10000%)
.10                   origin          OID     元のトラフィック情報（拡張）
.11                   client          OID     クライアントのトラフィック情報（拡張）
===================== =============== ======= ==============================



.. _snmp-cache-host-traffic-origin:

cache.host.traffic.origin
---------------------

::

    OID = 1.3.6.1.4.1.40001.1.4.1.11.10

ソースサーバートラフィックの統計情報を提供する。 ソースサーバーのトラフィックは、HTTPトラフィックとPortバイパストラフィックに区分する。

========================== =================================== ========== ===================================================================
OID                        Name                                Type       Description
========================== =================================== ========== ===================================================================
.1. ``[vhostMin]``         inbound                             Integer    ソースサーバーから受信し、平均トラフィック(Bytes)
.2. ``[vhostMin]``         outbound                            Integer    ソースサーバーに送信平均トラフィック(Bytes)
.3. ``[vhostMin]``         sessionAverage                      Integer    全体元のサーバーの平均セッション数
.4. ``[vhostMin]``         activesessionAverage                Integer    全体元のサーバーセッションの中で送信されているセッションの平均数 
.10                        http                                OID        ソースサーバーのHTTPトラフィック情報
.10.1. ``[vhostMin]``      http.inbound                        Integer    ソースサーバーから受信し、平均のHTTPトラフィック(Bytes)
.10.2. ``[vhostMin]``      http.outbound                       Integer    ソースサーバーに送信平均HTTPトラフィック(Bytes)
.10.3. ``[vhostMin]``      http.sessionAverage                 Integer    ソースサーバーの平均HTTPセッション数
.10.4. ``[vhostMin]``      http.reqHeaderSize                  Integer    ソースサーバーに送信平均HTTP Headerトラフィック(Bytes)
.10.5. ``[vhostMin]``      http.reqBodySize                    Integer    ソースサーバーに送信平均HTTP Bodyトラフィック(Bytes)
.10.6. ``[vhostMin]``      http.resHeaderSize                  Integer    ソースサーバーから受信し、平均HTTP Headerトラフィック(Bytes)
.10.7. ``[vhostMin]``      http.resBodySize                    Integer    ソースサーバーから受信し、平均HTTP Bodyトラフィック(Bytes)
.10.8. ``[vhostMin]``      http.reqAverage                     Integer    ソースサーバーに送信される平均HTTPリクエスト数
.10.9. ``[vhostMin]``      http.reqCount                       Integer    ソースサーバーに送信されるHTTPリクエスト数
.10.10. ``[vhostMin]``     http.resTotalAverage                Integer    ソースサーバーが送信した全体の平均HTTP応答数
.10.11. ``[vhostMin]``     http.resTotalCompleteAverage        Integer    ソースサーバーから成功した平均HTTPトランザクション数
.10.12. ``[vhostMin]``     http.resTotalTimeRes                Integer    元のサーバーからの応答ヘッダを受信するまでの平均所要時間(0.01ms)
.10.13. ``[vhostMin]``     http.resTotalTimeComplete           Integer    ソースサーバーからの応答HTTP Transaction平均完了時間(0.01ms)
.10.14. ``[vhostMin]``     http.resTotalCount                  Integer    ソースサーバーが送信した完全なHTTP応答数
.10.15. ``[vhostMin]``     http.resTotalCompleteCount          Integer    ソースサーバーから成功したHTTPトランザクション数
.10.20. ``[vhostMin]``     http.res2xxAverage                  Integer    ソースサーバーが送信した平均2xx応答数
.10.21. ``[vhostMin]``     http.res2xxCompleteAverage          Integer    ソースサーバーから成功した平均2xxトランザクション数
.10.22. ``[vhostMin]``     http.res2xxTimeRes                  Integer    ソースサーバーから2xxレスポンスヘッダを受信するまでの平均所要時間(0.01ms)
.10.23. ``[vhostMin]``     http.res2xxTimeComplete             Integer    ソースサーバーから2xx応答HTTP Transaction平均完了時間(0.01ms)
.10.24. ``[vhostMin]``     http.res2xxCount                    Integer    ソースサーバーが送信した2xx応答数
.10.25. ``[vhostMin]``     http.res2xxCompleteCount            Integer    ソースサーバーから成功した2xxトランザクション数
.10.30. ``[vhostMin]``     http.res3xxAverage                  Integer    ソースサーバーが送信した平均3xx応答数
.10.31. ``[vhostMin]``     http.res3xxCompleteAverage          Integer    ソースサーバーから成功した平均3xxトランザクション数
.10.32. ``[vhostMin]``     http.res3xxTimeRes                  Integer    ソースサーバーから3xx応答ヘッダを受信するまでの平均所要時間(0.01ms)
.10.33. ``[vhostMin]``     http.res3xxTimeComplete             Integer    ソースサーバーから3xx応答HTTP Transaction平均完了時間(0.01ms)
.10.34. ``[vhostMin]``     http.res3xxCount                    Integer    ソースサーバーが送信した3xx応答数
.10.35. ``[vhostMin]``     http.res3xxCompleteCount            Integer    ソースサーバーから成功した3xxトランザクション数
.10.40. ``[vhostMin]``     http.res4xxAverage                  Integer    ソースサーバーが送信した平均4xx応答数
.10.41. ``[vhostMin]``     http.res4xxCompleteAverage          Integer    ソースサーバーから成功した平均4xxトランザクション数수
.10.42. ``[vhostMin]``     http.res4xxTimeRes                  Integer    ースサーバーから4xx応答ヘッダを受信するまでの平均所要時間(0.01ms)
.10.43. ``[vhostMin]``     http.res4xxTimeComplete             Integer    ソースサーバーから4xx応答HTTP Transaction平均完了時間(0.01ms)
.10.44. ``[vhostMin]``     http.res4xxCount                    Integer    ソースサーバーが送信した4xx応答数
.10.45. ``[vhostMin]``     http.res4xxCompleteCount            Integer    ソースサーバーから成功した4xxトランザクション数
.10.50. ``[vhostMin]``     http.res5xxAverage                  Integer    ソースサーバーが送信した平均5xx応答数
.10.51. ``[vhostMin]``     http.res5xxCompleteAverage          Integer    ソースサーバーから成功した平均5xxトランザクション数
.10.52. ``[vhostMin]``     http.res5xxTimeRes                  Integer    ソースサーバーから5xx応答ヘッダを受信するまでの平均所要時間(0.01ms)
.10.53. ``[vhostMin]``     http.res5xxTimeComplete             Integer    ソースサーバーから5xx応答HTTP Transaction平均完了時間(0.01ms)
.10.54. ``[vhostMin]``     http.res5xxCount                    Integer    ソースサーバーが送信した5xx応答数
.10.55. ``[vhostMin]``     http.res5xxCompleteCount            Integer    ソースサーバーから成功した5xxトランザクション数
.10.60. ``[vhostMin]``     http.connectTimeoutAverage          Integer    平均ソースサーバー接続に失敗した回数
.10.61. ``[vhostMin]``     http.receiveTimeoutAverage          Integer    平均元サーバー送信に失敗した回数
.10.62. ``[vhostMin]``     http.connectAverage                 Integer    平均ソースサーバー接続成功回数
.10.63. ``[vhostMin]``     http.dnsQueryTime                   Integer    ソースサーバー接続時の平均DNSクエリの所要時間
.10.64. ``[vhostMin]``     http.connectTime                    Integer    ソースサーバーの平均接続時間(0.01ms)
.10.65. ``[vhostMin]``     http.connectTimeoutCount            Integer    ソースサーバー接続に失敗した回数
.10.66. ``[vhostMin]``     http.receiveTimeoutCount            Integer    ソースサーバーの転送に失敗した回数
.10.67. ``[vhostMin]``     http.connectCount                   Integer    ソースサーバー接続成功回数
.10.68. ``[vhostMin]``     http.closeAverage                   Integer    転送中のソースサーバーから先にソケットを終了した平均回数
.10.69. ``[vhostMin]``     http.closeCount                     Integer    転送中のソースサーバーから先にソケットを終了した回数
.11                        portbypass                          OID        Portバイパス元サーバーのトラフィック情報
.11.1. ``[vhostMin]``      portbypass.inbound                  Integer    Portバイパスを介して、元のサーバーから受け取る平均トラフィック(Bytes)
.11.2. ``[vhostMin]``      portbypass.outbound                 Integer    Portバイパスを介して、元のサーバーに送信する平均トラフィック(Bytes)
.11.3. ``[vhostMin]``      portbypass.sessionAverage           Integer    Portバイパス中の平均元のサーバーのセッション数
.11.4. ``[vhostMin]``      portbypass.closedAverage            Integer    Portバイパス中のソースサーバーが接続を終了した平均回数
.11.5. ``[vhostMin]``      portbypass.connectTimeoutAverage    Integer    Portバイパス元のサーバーの平均接続失敗回数
.11.6. ``[vhostMin]``      portbypass.closedCount              Integer    Portバイパス中のソースサーバーが接続を終了した回数
.11.7. ``[vhostMin]``      portbypass.connectTimeoutCount      Integer    Portバイパス元のサーバー接続に失敗した回数
========================== =================================== ========== ===================================================================



.. _snmp-cache-host-traffic-client:

cache.host.traffic.client
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.1.11.11

クライアントトラフィックの統計情報を提供する。 クライアントのトラフィックは、HTTPトラフィックは、SSLトラフィック、Portバイパストラフィックに区分される。 SNMPでは、ディレクトリごとの統計を提供しない。 たとえディレクトリの統計情報が設定されているとしても、合算されています。

========================== ========================================== ========== =============================================================
OID                        Name                                       Type       Description
========================== ========================================== ========== =============================================================
.1. ``[vhostMin]``         inbound                                    Integer    クライアントから受信平均トラフィック(Bytes)
.2. ``[vhostMin]``         outbound                                   Integer    クライアントに送信平均トラフィック(Bytes)
.3. ``[vhostMin]``         sessionAverage                             Integer    完全なクライアントの平均セッション数
.4. ``[vhostMin]``         activesessionAverage                       Integer    全クライアントの転送中の平均セッション数
.10                        http                                       OID        クライアントのHTTPトラフィック情報
.10.1. ``[vhostMin]``      http.inbound                               Integer    クライアントから受信平均HTTPトラフィック(Bytes)
.10.2. ``[vhostMin]``      http.outbound                              Integer    クライアントに送信平均HTTPトラフィック(Bytes)
.10.3. ``[vhostMin]``      http.sessionAverage                        Integer    クライアントの平均HTTPセッション数
.10.4. ``[vhostMin]``      http.reqHeaderSize                         Integer    クライアントから受信平均HTTP Headerトラフィック(Bytes)
.10.5. ``[vhostMin]``      http.reqBodySize                           Integer    クライアントから受信平均HTTP Bodyトラフィック(Bytes)
.10.6. ``[vhostMin]``      http.resHeaderSize                         Integer    クライアントに送信平均HTTP Headerトラフィック(Bytes)
.10.7. ``[vhostMin]``      http.resBodySize                           Integer    クライアントに送信平均HTTP Bodyトラフィック(Bytes)
.10.8. ``[vhostMin]``      http.reqAverage                            Integer    クライアントから受信した平均HTTPリクエスト数
.10.9. ``[vhostMin]``      http.reqCount                              Integer    クライアントから受信したHTTPリクエスト数
.10.10. ``[vhostMin]``     http.resTotalAverage                       Integer    クライアントに送信平均全体の応答数
.10.11. ``[vhostMin]``     http.resTotalCompleteAverage               Integer    クライアントが完了した平均HTTPトランザクション数
.10.12. ``[vhostMin]``     http.resTotalTimeRes                       Integer    クライアントの応答の平均所要時間(0.01ms)
.10.13. ``[vhostMin]``     http.resTotalTimeComplete                  Integer    クライアントHTTP Transaction平均完了時間(0.01ms)
.10.14. ``[vhostMin]``     http.resTotalCount                         Integer    クライアントに送信全体の応答数
.10.15. ``[vhostMin]``     http.resTotalCompleteCount                 Integer    クライアントが完了したHTTPトランザクション数
.10.20. ``[vhostMin]``     http.res2xxAverage                         Integer    クライアントに送信平均2xx応答数
.10.21. ``[vhostMin]``     http.res2xxCompleteAverage                 Integer    クライアントが完了した平均2xxトランザクション数
.10.22. ``[vhostMin]``     http.res2xxTimeRes                         Integer    クライアント2xx応答の平均所要時間(0.01ms)
.10.23. ``[vhostMin]``     http.res2xxTimeComplete                    Integer    クライアント2xx応答HTTP Transaction平均完了時間(0.01ms)
.10.24. ``[vhostMin]``     http.res2xxCount                           Integer    クライアントに送信2xx応答数
.10.25. ``[vhostMin]``     http.res2xxCompleteCount                   Integer    クライアントが完了した2xxトランザクション数
.10.30. ``[vhostMin]``     http.res3xxAverage                         Integer    クライアントに送信平均3xx応答数
.10.31. ``[vhostMin]``     http.res3xxCompleteAverage                 Integer    クライアントが完了した平均3xxトランザクション数
.10.32. ``[vhostMin]``     http.res3xxTimeRes                         Integer    クライアント3xx応答の平均所要時間(0.01ms)
.10.33. ``[vhostMin]``     http.res3xxTimeComplete                    Integer    クライアント3xx応答HTTP Transaction平均完了時間(0.01ms)
.10.34. ``[vhostMin]``     http.res3xxCount                           Integer    クライアントに送信3xx応答数
.10.35. ``[vhostMin]``     http.res3xxCompleteCount                   Integer    クライアントが完了した3xxトランザクション数
.10.40. ``[vhostMin]``     http.res4xxAverage                         Integer    クライアントに送信平均4xx応答数
.10.41. ``[vhostMin]``     http.res4xxCompleteAverage                 Integer    クライアントが完了した平均4xxトランザクション数
.10.42. ``[vhostMin]``     http.res4xxTimeRes                         Integer    クライアント4xx応答の平均所要時間(0.01ms)
.10.43. ``[vhostMin]``     http.res4xxTimeComplete                    Integer    クライアント4xx応答HTTP Transaction平均完了時間(0.01ms)
.10.44. ``[vhostMin]``     http.res4xxCount                           Integer    クライアントに送信4xx応答数
.10.45. ``[vhostMin]``     http.res4xxCompleteCount                   Integer    クライアントが完了した4xxトランザクション数
.10.50. ``[vhostMin]``     http.res5xxAverage                         Integer    クライアントに送信平均5xx応答数
.10.51. ``[vhostMin]``     http.res5xxCompleteAverage                 Integer    クライアントが完了した平均5xxトランザクション数
.10.52. ``[vhostMin]``     http.res5xxTimeRes                         Integer    クライアント5xx応答の平均所要時間(0.01ms)
.10.53. ``[vhostMin]``     http.res5xxTimeComplete                    Integer    クライアント5xx応答HTTP Transaction平均完了時間(0.01ms)
.10.54. ``[vhostMin]``     http.res5xxCount                           Integer    クライアントに送信5xx応答数
.10.55. ``[vhostMin]``     http.res5xxCompleteCount                   Integer    クライアントが完了した5xxトランザクション数
.10.60. ``[vhostMin]``     http.reqDeniedAverage                      Integer    ブロックされたリクエストの平均
.10.61. ``[vhostMin]``     http.reqDeniedCount                        Integer    ブロックされたリクエスト数
.11                        portbypass                                 OID        Portバイパスクライアントのトラフィック情報
.11.1. ``[vhostMin]``      portbypass.inbound                         Integer    Portバイパスを介してクライアントから受信平均トラフィック(Bytes)
.11.2. ``[vhostMin]``      portbypass.outbound                        Integer    Portバイパスを介してクライアントに送信平均トラフィック(Bytes)
.11.3. ``[vhostMin]``      portbypass.sessionAverage                  Integer    Portバイパスしているクライアントの平均セッション数
.11.4. ``[vhostMin]``      portbypass.closedAverage                   Integer    Portバイパス中のクライアントが接続を終了した平均回数
.11.5. ``[vhostMin]``      portbypass.closedCount                     Integer    Portバイパス中のクライアントが接続を終了した回数
.12                        ssl                                        OID        SSLクライアントのトラフィック情報보
.12.2. ``[vhostMin]``      ssl.inbound                                Integer    SSLを介してクライアントから受信平均トラフィック(Bytes)
.12.3. ``[vhostMin]``      ssl.outbound                               Integer    SSLを介してクライアントに送信平均トラフィック(Bytes)
.13                        requestHitAverage                          OID        平均キャッシュHIT結果
.13.1. ``[vhostMin]``      requestHitAverage.TCP_HIT                  Integer    TCP_HIT
.13.2. ``[vhostMin]``      requestHitAverage.TCP_IMS_HIT              Integer    TCP_IMS_HIT
.13.3. ``[vhostMin]``      requestHitAverage.TCP_REFRESH_HIT          Integer    TCP_REFRESH_HIT
.13.4. ``[vhostMin]``      requestHitAverage.TCP_REF_FAIL_HIT         Integer    TCP_REF_FAIL_HIT
.13.5. ``[vhostMin]``      requestHitAverage.TCP_NEGATIVE_HIT         Integer    TCP_NEGATIVE_HIT
.13.6. ``[vhostMin]``      requestHitAverage.TCP_MISS                 Integer    TCP_MISS
.13.7. ``[vhostMin]``      requestHitAverage.TCP_REFRESH_MISS         Integer    TCP_REFRESH_MISS
.13.8. ``[vhostMin]``      requestHitAverage.TCP_CLIENT_REFRESH_MISS  Integer    TCP_CLIENT_REFRESH_MISS
.13.9. ``[vhostMin]``      requestHitAverage.TCP_DENIED               Integer    TCP_DENIED
.13.10. ``[vhostMin]``     requestHitAverage.TCP_ERROR                Integer    TCP_ERROR
.13.11. ``[vhostMin]``     requestHitAverage.TCP_REDIRECT_HIT         Integer    TCP_REDIRECT_HIT
.14                        requestHitCount                            OID        キャッシュHIT結果数
.14.1. ``[vhostMin]``      requestHitCount.TCP_HIT                    Integer    TCP_HIT
.14.2. ``[vhostMin]``      requestHitCount.TCP_IMS_HIT                Integer    TCP_IMS_HIT
.14.3. ``[vhostMin]``      requestHitCount.TCP_REFRESH_HIT            Integer    TCP_REFRESH_HIT
.14.4. ``[vhostMin]``      requestHitCount.TCP_REF_FAIL_HIT           Integer    TCP_REF_FAIL_HIT
.14.5. ``[vhostMin]``      requestHitCount.TCP_NEGATIVE_HIT           Integer    TCP_NEGATIVE_HIT
.14.6. ``[vhostMin]``      requestHitCount.TCP_MISS                   Integer    TCP_MISS
.14.7. ``[vhostMin]``      requestHitCount.TCP_REFRESH_MISS           Integer    TCP_REFRESH_MISS
.14.8. ``[vhostMin]``      requestHitCount.TCP_CLIENT_REFRESH_MISS    Integer    TCP_CLIENT_REFRESH_MISS
.14.9. ``[vhostMin]``      requestHitCount.TCP_DENIED                 Integer    TCP_DENIED
.14.10. ``[vhostMin]``     requestHitCount.TCP_ERROR                  Integer    TCP_ERROR
.14.11. ``[vhostMin]``     requestHitCount.TCP_REDIRECT_HIT           Integer    TCP_REDIRECT_HIT
========================== ========================================== ========== =============================================================



.. _snmp-cache-host-traffic-filesystem:

cache.host.traffic.filesystem
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.1.11.20

HostのFile I / O統計を提供する。

======================== ============================================ ========== =============================================
OID                      Name                                         Type       Description
======================== ============================================ ========== =============================================
.1. ``[vhostMin]``       requestHitRatio                              Integer    Request Hit Ratio(100%)
.2. ``[vhostMin]``                                                               Request Hit Ratio(10000%)
.3. ``[vhostMin]``       byteHitRatio                                 Integer    Byte Hit Ratio(100%)
.4. ``[vhostMin]``                                                               Byte Hit Ratio(10000%)
.5. ``[vhostMin]``       outbound                                     Integer    File I / Oデバイスに送信平均トラフィック (Bytes)
.6. ``[vhostMin]``       session                                      Integer    File I / Oを実行中の平均Thread数
.7                       requestHitAverage                            OID        平均キャッシュHIT結果
.7.1. ``[vhostMin]``     requestHitAverage.TCP_HIT                    Integer    TCP_HIT
.7.2. ``[vhostMin]``     requestHitAverage.TCP_IMS_HIT                Integer    TCP_IMS_HIT
.7.3. ``[vhostMin]``     requestHitAverage.TCP_REFRESH_HIT            Integer    TCP_REFRESH_HIT
.7.4. ``[vhostMin]``     requestHitAverage.TCP_REF_FAIL_HIT           Integer    TCP_REF_FAIL_HIT
.7.5. ``[vhostMin]``     requestHitAverage.TCP_NEGATIVE_HIT           Integer    TCP_NEGATIVE_HIT
.7.6. ``[vhostMin]``     requestHitAverage.TCP_MISS                   Integer    TCP_MISS
.7.7. ``[vhostMin]``     requestHitAverage.TCP_REFRESH_MISS           Integer    TCP_REFRESH_MISS
.7.8. ``[vhostMin]``     requestHitAverage.TCP_CLIENT_REFRESH_MISS    Integer    TCP_CLIENT_REFRESH_MISS
.7.9. ``[vhostMin]``     requestHitAverage.TCP_DENIED                 Integer    TCP_DENIED
.7.10. ``[vhostMin]``    requestHitAverage.TCP_ERROR                  Integer    TCP_ERROR
.7.11. ``[vhostMin]``    requestHitAverage.TCP_REDIRECT_HIT           Integer    TCP_REDIRECT_HIT
.8                       requestHitCount                              OID        キャッシュHIT結果数
.8.1. ``[vhostMin]``     requestHitCount.TCP_HIT                      Integer    TCP_HIT
.8.2. ``[vhostMin]``     requestHitCount.TCP_IMS_HIT                  Integer    TCP_IMS_HIT
.8.3. ``[vhostMin]``     requestHitCount.TCP_REFRESH_HIT              Integer    TCP_REFRESH_HIT
.8.4. ``[vhostMin]``     requestHitCount.TCP_REF_FAIL_HIT             Integer    TCP_REF_FAIL_HIT
.8.5. ``[vhostMin]``     requestHitCount.TCP_NEGATIVE_HIT             Integer    TCP_NEGATIVE_HIT
.8.6. ``[vhostMin]``     requestHitCount.TCP_MISS                     Integer    TCP_MISS
.8.7. ``[vhostMin]``     requestHitCount.TCP_REFRESH_MISS             Integer    TCP_REFRESH_MISS
.8.8. ``[vhostMin]``     requestHitCount.TCP_CLIENT_REFRESH_MISS      Integer    TCP_CLIENT_REFRESH_MISS
.8.9. ``[vhostMin]``     requestHitCount.TCP_DENIED                   Integer    TCP_DENIED
.8.10. ``[vhostMin]``    requestHitCount.TCP_ERROR                    Integer    TCP_ERROR
.8.11. ``[vhostMin]``    requestHitCount.TCP_REDIRECT_HIT             Integer    TCP_REDIRECT_HIT
.10. ``[vhostMin]``      getattr.filecount                            Integer    （getattr関数呼び出し）FILEに応答した回数
.11. ``[vhostMin]``      getattr.dircount                             Integer    （getattr関数呼び出し）DIRで応答した回数
.12. ``[vhostMin]``      getattr.failcount                            Integer    （getattr関数呼び出し）が失敗に応答した回数
.13. ``[vhostMin]``      getattr.timeres                              Integer    （getattr関数の呼び出し）の反応時間 (0.01ms)
.14. ``[vhostMin]``      open.count                                   Integer    open関数の呼び出し回数
.15. ``[vhostMin]``      open.timeres                                 Integer    open関数の反応時間 (0.01ms)
.16. ``[vhostMin]``      read.count                                   Integer    read関数の呼び出し回数
.17. ``[vhostMin]``      read.timeres                                 Integer    read関数の反応時間 (0.01ms)
.18. ``[vhostMin]``      read.buffersize                              Integer    read関数で要求されたバッファサイズ (Bytes)
.19. ``[vhostMin]``      read.bufferfilled                            Integer    read関数で要求されたバッファに詰めたサイズ (Bytes)
======================== ============================================ ========== =============================================



.. _snmp-cache-host-traffic-dims:

cache.host.traffic.dims
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.1.11.21

HostのDIMS変換統計を提供する。

======================== ============================================ ========== =============================================
OID                      Name                                         Type       Description
======================== ============================================ ========== =============================================
.1. ``[vhostMin]``       requests                                     Integer    DIMS変換要求数
.2. ``[vhostMin]``       converted                                    Integer    変換に成功回数
.3. ``[vhostMin]``       failed                                       Integer    変換に失敗した回数
.4. ``[vhostMin]``       avgsrcsize                                   Integer    元の画像の平均サイズ (Bytes)
.5. ``[vhostMin]``       avgdestsize                                  Integer    変換された画像の平均サイズ (Bytes)
.6. ``[vhostMin]``       avgtime                                      Integer    変換所要時間 (ms)
======================== ============================================ ========== =============================================



.. _snmp-cache-host-traffic-compression:

cache.host.traffic.compression
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.1.11.22

Hostの圧縮統計を提供する。

======================== ============================================ ========== =============================================
OID                      Name                                         Type       Description
======================== ============================================ ========== =============================================
.1. ``[vhostMin]``       requests                                     Integer    圧縮要求数
.2. ``[vhostMin]``       converted                                    Integer    圧縮成功回数
.3. ``[vhostMin]``       failed                                       Integer    圧縮失敗回数
.4. ``[vhostMin]``       avgsrcsize                                   Integer    元のファイルの平均サイズ (Bytes)
.5. ``[vhostMin]``       avgdestsize                                  Integer    圧縮されたファイルの平均サイズ (Bytes)
.6. ``[vhostMin]``       avgtime                                      Integer    圧縮時間 (ms)
======================== ============================================ ========== =============================================





.. _snmp-cache-vhost:

cache.vhost
====================================

::

   OID = 1.3.6.1.4.1.40001.1.4.3.1

仮想ホストの情報を提供する。  ``[vhostIndex]`` は1から仮想ホスト数の範囲を持つ。

======================= ========= ========== ============================================
OID                     Name      Type       Description
======================= ========= ========== ============================================
.2. ``[vhostIndex]``    name      String     仮想ホスト名
.3. ``[vhostIndex]``    status    String     "Healthy" または "Inactive" または "Emergency"
.4. ``[vhostIndex]``    uptime    Integer    仮想ホストの実行時間（秒）
.10                     contents  OID        コンテンツ情報（拡張）
.11                     traffic   OID        統計（拡張）
======================= ========= ========== ============================================



.. _snmp-cache-vhost-contents:

cache.vhost.contents
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.3.1.10

仮想ホストがサービスするコンテンツ情報を提供する。

========================= =================== ========== =============================
OID                       Name                Type       Description
========================= =================== ========== =============================
.1. ``[vhostIndex]``      memory              Integer    メモリキャッシュサイズ(KB)
.2. ``[vhostIndex]``      filesTotalCount     Integer    サービス中のファイル数
.3. ``[vhostIndex]``      filesTotalSize      Integer    サービス中の全ファイル量(MB)
.10. ``[vhostIndex]``     filesCountU1KB      Integer    1KB未満のファイル数
.11. ``[vhostIndex]``     filesCountU2KB      Integer    2KB未満のファイル数
.12. ``[vhostIndex]``     filesCountU4KB      Integer    4KB未満のファイル数
.13. ``[vhostIndex]``     filesCountU8KB      Integer    8KB未満のファイル数
.14. ``[vhostIndex]``     filesCountU16KB     Integer    16KB未満のファイル数
.15. ``[vhostIndex]``     filesCountU32KB     Integer    32KB未満のファイル数
.16. ``[vhostIndex]``     filesCountU64KB     Integer    64KB未満のファイル数수
.17. ``[vhostIndex]``     filesCountU128KB    Integer    128KB未満のファイル数
.18. ``[vhostIndex]``     filesCountU256KB    Integer    256KB未満のファイル数
.19. ``[vhostIndex]``     filesCountU512KB    Integer    512KB未満のファイル数
.20. ``[vhostIndex]``     filesCountU1MB      Integer    1MB未満のファイル数
.21. ``[vhostIndex]``     filesCountU2MB      Integer    2MB未満のファイル数
.22. ``[vhostIndex]``     filesCountU4MB      Integer    4MB未満のファイル数
.23. ``[vhostIndex]``     filesCountU8MB      Integer    8MB未満のファイル数
.24. ``[vhostIndex]``     filesCountU16MB     Integer    16MB未満のファイル数
.25. ``[vhostIndex]``     filesCountU32MB     Integer    32MB未満のファイル数
.26. ``[vhostIndex]``     filesCountU64MB     Integer    64MB未満のファイル数
.27. ``[vhostIndex]``     filesCountU128MB    Integer    128MB未満のファイル数
.28. ``[vhostIndex]``     filesCountU256MB    Integer    256MB未満のファイル数
.29. ``[vhostIndex]``     filesCountU512MB    Integer    512MB未満のファイル数
.30. ``[vhostIndex]``     filesCountU1GB      Integer    1GB未満のファイル数
.31. ``[vhostIndex]``     filesCountU2GB      Integer    2GB未満のファイル数
.32. ``[vhostIndex]``     filesCountU4GB      Integer    4GB未満のファイル数
.33. ``[vhostIndex]``     filesCountU8GB      Integer    8GB未満のファイル数
.34. ``[vhostIndex]``     filesCountU16GB     Integer    16GB未満のファイル数
.35. ``[vhostIndex]``     filesCountO16GB     Integer    16GB以上のファイル数
========================= =================== ========== =============================



.. _snmp-cache-vhost-traffic:

cache.vhost.traffic
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.3.1.11

仮想ホストのキャッシュサービスとトラフィックの統計情報を提供する。trafficのすべての統計情報は、最大60分までの平均で提供される。minは、 '分'を意味し、最大60までの値を有する。minが省略されたり0であれば、リアルタイムの情報を提供する。

========================================= ================= =========== ==============================
OID                                       Name              Type        Description
========================================= ================= =========== ==============================
.1. ``[vhostMin]`` . ``[vhostIndex]``     requestHitRatio   Integer     Request Hit Ratio(100%)
.2. ``[vhostMin]`` . ``[vhostIndex]``                                   Request Hit Ratio(10000%)
.3. ``[vhostMin]`` . ``[vhostIndex]``     bytesHitRatio     Integer     Bytes Hit Ratio(100%)
.4. ``[vhostMin]`` . ``[vhostIndex]``                                   Bytes Hit Ratio(10000%)
.10                                       origin            OID         元のトラフィック情報（拡張）
.11                                       client            OID         クライアントのトラフィック情報（拡張）
========================================= ================= =========== ==============================



.. _snmp-cache-vhost-traffic-origin:

cache.vhost.traffic.origin
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.3.1.11.10

ソースサーバートラフィックの統計情報を提供する。ソースサーバーのトラフィックは、HTTPトラフィックとPortバイパストラフィックに区分される。

============================================= ===================================== ========== =================================================================
OID                                           Name                                  Type       Description
============================================= ===================================== ========== =================================================================
.1. ``[vhostMin]`` . ``[vhostIndex]``         inbound                               Integer    ソースサーバーから受信し、平均トラフィック(Bytes)
.2. ``[vhostMin]`` . ``[vhostIndex]``         outbound                              Integer    ソースサーバーに送信平均トラフィック(Bytes)
.3. ``[vhostMin]`` . ``[vhostIndex]``         sessionAverage                        Integer    全体元のサーバーの平均セッション数
.4. ``[vhostMin]`` . ``[vhostIndex]``         activesessionAverage                  Integer    全体元のサーバーセッションの中で送信されているセッションの平均数
.10                                           http                                  OID        ソースサーバーのHTTPトラフィック情報
.10.1. ``[vhostMin]`` . ``[vhostIndex]``      http.inbound                          Integer    ソースサーバーから受信し、平均のHTTPトラフィック(Bytes)
.10.2. ``[vhostMin]`` . ``[vhostIndex]``      http.outbound                         Integer    ソースサーバーに送信平均HTTPトラフィック(Bytes)
.10.3. ``[vhostMin]`` . ``[vhostIndex]``      http.sessionAverage                   Integer    ソースサーバーの平均HTTPセッション数
.10.4. ``[vhostMin]`` . ``[vhostIndex]``      http.reqHeaderSize                    Integer    ソースサーバーに送信平均HTTP Headerトラフィック(Bytes)
.10.5. ``[vhostMin]`` . ``[vhostIndex]``      http.reqBodySize                      Integer    ソースサーバーに送信平均HTTP Bodyトラフィック(Bytes)
.10.6. ``[vhostMin]`` . ``[vhostIndex]``      http.resHeaderSize                    Integer    ソースサーバーから受信し、平均HTTP Headerトラフィック(Bytes)
.10.7. ``[vhostMin]`` . ``[vhostIndex]``      http.resBodySize                      Integer    ソースサーバーから受信し、平均HTTP Bodyトラフィック(Bytes)
.10.8. ``[vhostMin]`` . ``[vhostIndex]``      http.reqAverage                       Integer    ソースサーバーに送信される平均HTTPリクエスト数
.10.9. ``[vhostMin]`` . ``[vhostIndex]``      http.reqCount                         Integer    ソースサーバーに送信されるHTTPリクエスト数
.10.10. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalAverage                  Integer    ソースサーバーが送信した全体の平均HTTP応答数
.10.11. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalCompleteAverage          Integer    ソースサーバーから成功した平均HTTPトランザクション数
.10.12. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalTimeRes                  Integer    元のサーバーからの応答ヘッダを受信するまでの平均所要時間(0.01ms)
.10.13. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalTimeComplete             Integer    ソースサーバーからの応答HTTP Transaction平均完了時間(0.01ms)
.10.14. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalCount                    Integer    ソースサーバーが送信した完全なHTTP応答数
.10.15. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalCompleteCount            Integer    ソースサーバーから成功したHTTPトランザクション数
.10.20. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxAverage                    Integer    ソースサーバーが送信した平均2xx応答数
.10.21. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxCompleteAverage            Integer    ソースサーバーから成功した平均2xxトランザクション数
.10.22. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxTimeRes                    Integer    ソースサーバーから2xxレスポンスヘッダを受信するまでの平均所要時間(0.01ms)
.10.23. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxTimeComplete               Integer    ソースサーバーから2xx応答HTTP Transaction平均完了時間(0.01ms)
.10.24. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxCount                      Integer    ソースサーバーが送信した2xx応答数
.10.25. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxCompleteCount              Integer    ソースサーバーから成功した2xxトランザクション数
.10.30. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxAverage                    Integer    ソースサーバーが送信した平均3xx応答数
.10.31. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxCompleteAverage            Integer    ソースサーバーから成功した平均3xxトランザクション数
.10.32. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxTimeRes                    Integer    ソースサーバーから3xx応答ヘッダを受信するまでの平均所要時間(0.01ms)
.10.33. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxTimeComplete               Integer    ソースサーバーから3xx応答HTTP Transaction平均完了時間(0.01ms)
.10.34. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxCount                      Integer    ソースサーバーが送信した3xx応答数
.10.35. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxCompleteCount              Integer    ソースサーバーから成功した3xxトランザクション数
.10.40. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxAverage                    Integer    ソースサーバーが送信した平均4xx応答数
.10.41. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxCompleteAverage            Integer    ソースサーバーから成功した平均4xxトランザクション数
.10.42. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxTimeRes                    Integer    ソースサーバーから4xx応答ヘッダを受信するまでの平均所要時間(0.01ms)
.10.43. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxTimeComplete               Integer    ソースサーバーから4xx応答HTTP Transaction平均完了時間(0.01ms)
.10.44. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxCount                      Integer    ソースサーバーが送信した4xx応答数
.10.45. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxCompleteCount              Integer    ソースサーバーから成功した4xxトランザクション数
.10.50. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxAverage                    Integer    ソースサーバーが送信した平均5xx応答数
.10.51. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxCompleteAverage            Integer    ソースサーバーから成功した平均5xxトランザクション数
.10.52. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxTimeRes                    Integer    ソースサーバーから5xx応答ヘッダを受信するまでの平均所要時間(0.01ms)
.10.53. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxTimeComplete               Integer    ソースサーバーから5xx応答HTTP Transaction平均完了時間(0.01ms)
.10.54. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxCount                      Integer    ソースサーバーが送信した5xx応答数
.10.55. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxCompleteCount              Integer    ソースサーバーから成功した5xxトランザクション数
.10.60. ``[vhostMin]`` . ``[vhostIndex]``     http.connectTimeoutAverage            Integer    平均ソースサーバー接続に失敗した回数
.10.61. ``[vhostMin]`` . ``[vhostIndex]``     http.receiveTimeoutAverage            Integer    平均元サーバー送信に失敗した回数수
.10.62. ``[vhostMin]`` . ``[vhostIndex]``     http.connectAverage                   Integer    平均ソースサーバー接続成功回数
.10.63. ``[vhostMin]`` . ``[vhostIndex]``     http.dnsQueryTime                     Integer    ソースサーバー接続時の平均DNSクエリの所要時間
.10.64. ``[vhostMin]`` . ``[vhostIndex]``     http.connectTime                      Integer    ソースサーバーの平均接続時間(0.01ms)
.10.65. ``[vhostMin]`` . ``[vhostIndex]``     http.connectTimeoutCount              Integer    ソースサーバー接続に失敗した回数
.10.66. ``[vhostMin]`` . ``[vhostIndex]``     http.receiveTimeoutCount              Integer    ソースサーバーの転送に失敗した回数
.10.67. ``[vhostMin]`` . ``[vhostIndex]``     http.connectCount                     Integer    ソースサーバー接続成功回数
.10.68. ``[vhostMin]`` . ``[vhostIndex]``     http.closeAverage                     Integer    転送中のソースサーバーから先にソケットを終了した平均回数
.10.69. ``[vhostMin]`` . ``[vhostIndex]``     http.closeCount                       Integer    転送中のソースサーバーから先にソケットを終了した回数
.11                                           portbypass                            OID        Portバイパス元サーバーのトラフィック情報
.11.1. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.inbound                    Integer    Portバイパスを介して、元のサーバーから受け取る平均トラフィック(Bytes)
.11.2. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.outbound                   Integer    Portバイパスを介して、元のサーバーに送信する平均トラフィック(Bytes)
.11.3. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.sessionAverage             Integer    Portバイパス中の平均元のサーバーのセッション数
.11.4. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.closedAverage              Integer    Portバイパス中のソースサーバーが接続を終了した平均回数
.11.5. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.connectTimeoutAverage      Integer    Portバイパス元のサーバーの平均接続失敗回数
.11.6. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.closedCount                Integer    Portバイパス中のソースサーバーが接続を終了した回数
.11.7. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.connectTimeoutCount        Integer    Portバイパス元のサーバー接続に失敗した回数
============================================= ===================================== ========== =================================================================



.. _snmp-cache-vhost-traffic-client:

cache.vhost.traffic.client
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.3.1.11.11

クライアントトラフィックの統計情報を提供する。クライアントのトラフィックは、HTTPトラフィックは、SSLトラフィック、Portバイパストラフィックに区分される。SNMPでは、ディレクトリごとの統計を提供しない。たとえディレクトリの統計情報が設定されているとしても、合算されています。

============================================= ========================================= ========== ==============================================================
OID                                           Name                                      Type       Description
============================================= ========================================= ========== ==============================================================
.1. ``[vhostMin]`` . ``[vhostIndex]``         inbound                                   Integer    クライアントから受信平均トラフィック(Bytes)
.2. ``[vhostMin]`` . ``[vhostIndex]``         outbound                                  Integer    クライアントに送信平均トラフィック(Bytes)
.3. ``[vhostMin]`` . ``[vhostIndex]``         sessionAverage                            Integer    完全なクライアントの平均セッション数
.4. ``[vhostMin]`` . ``[vhostIndex]``         activesessionAverage                      Integer    全クライアントの転送中の平均セッション数균 세션수
.10                                           http                                      OID        クライアントのHTTPトラフィック情報
.10.1. ``[vhostMin]`` . ``[vhostIndex]``      http.inbound                              Integer    クライアントから受信平均HTTPトラフィック(Bytes)
.10.2. ``[vhostMin]`` . ``[vhostIndex]``      http.outbound                             Integer    クライアントに送信平均HTTPトラフィック(Bytes)
.10.3. ``[vhostMin]`` . ``[vhostIndex]``      http.sessionAverage                       Integer    クライアントの平均HTTPセッション数
.10.4. ``[vhostMin]`` . ``[vhostIndex]``      http.reqHeaderSize                        Integer    クライアントから受信平均HTTP Headerトラフィック(Bytes)
.10.5. ``[vhostMin]`` . ``[vhostIndex]``      http.reqBodySize                          Integer    クライアントから受信平均HTTP Bodyトラフィック(Bytes)
.10.6. ``[vhostMin]`` . ``[vhostIndex]``      http.resHeaderSize                        Integer    クライアントに送信平均HTTP Headerトラフィック(Bytes)
.10.7. ``[vhostMin]`` . ``[vhostIndex]``      http.resBodySize                          Integer    クライアントに送信平均HTTP Bodyトラフィック(Bytes)
.10.8. ``[vhostMin]`` . ``[vhostIndex]``      http.reqAverage                           Integer    クライアントから受信した平均HTTPリクエスト数
.10.9. ``[vhostMin]`` . ``[vhostIndex]``      http.reqCount                             Integer    クライアントから受信したHTTPリクエスト数
.10.10. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalAverage                      Integer    クライアントに送信平均全体の応答数
.10.11. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalCompleteAverage              Integer    クライアントが完了した平均HTTPトランザクション数
.10.12. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalTimeRes                      Integer    クライアントの応答の平均所要時間(0.01ms)
.10.13. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalTimeComplete                 Integer    クライアントHTTP Transaction平均完了時間(0.01ms)
.10.14. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalCount                        Integer    クライアントに送信全体の応答数
.10.15. ``[vhostMin]`` . ``[vhostIndex]``     http.resTotalCompleteCount                Integer    クライアントが完了したHTTPトランザクション数
.10.20. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxAverage                        Integer    クライアントに送信平均2xx応答数
.10.21. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxCompleteAverage                Integer    クライアントが完了した平均2xxトランザクション数
.10.22. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxTimeRes                        Integer    クライアント2xx応答の平均所要時間(0.01ms)
.10.23. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxTimeComplete                   Integer    クライアント2xx応答HTTP Transaction平均完了時間(0.01ms)
.10.24. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxCount                          Integer    クライアントに送信2xx応答数
.10.25. ``[vhostMin]`` . ``[vhostIndex]``     http.res2xxCompleteCount                  Integer    クライアントが完了した2xxトランザクション数
.10.30. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxAverage                        Integer    クライアントに送信平均3xx応答数
.10.31. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxCompleteAverage                Integer    クライアントが完了した平均3xxトランザクション数
.10.32. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxTimeRes                        Integer    クライアント3xx応答の平均所要時間(0.01ms)
.10.33. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxTimeComplete                   Integer    クライアント3xx応答HTTP Transaction平均完了時間(0.01ms)
.10.34. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxCount                          Integer    クライアントに送信3xx応答数
.10.35. ``[vhostMin]`` . ``[vhostIndex]``     http.res3xxCompleteCount                  Integer    クライアントが完了した3xxトランザクション数
.10.40. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxAverage                        Integer    クライアントに送信平均4xx応答数
.10.41. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxCompleteAverage                Integer    クライアントが完了した平均4xxトランザクション数
.10.42. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxTimeRes                        Integer    クライアント4xx応答の平均所要時間(0.01ms)
.10.43. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxTimeComplete                   Integer    クライアント4xx応答HTTP Transaction平均完了時間(0.01ms)
.10.44. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxCount                          Integer    クライアントに送信4xx応答数
.10.45. ``[vhostMin]`` . ``[vhostIndex]``     http.res4xxCompleteCount                  Integer    クライアントが完了した4xxトランザクション数
.10.50. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxAverage                        Integer    クライアントに送信平均5xx応答数
.10.51. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxCompleteAverage                Integer    クライアントが完了した平均5xxトランザクション数
.10.52. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxTimeRes                        Integer    クライアント5xx応答の平均所要時間(0.01ms)
.10.53. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxTimeComplete                   Integer    クライアント5xx応答HTTP Transaction平均完了時間(0.01ms)
.10.54. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxCount                          Integer    クライアントに送信5xx応答数
.10.55. ``[vhostMin]`` . ``[vhostIndex]``     http.res5xxCompleteCount                  Integer    クライアントが完了した5xxトランザクション数
.10.60. ``[vhostMin]`` . ``[vhostIndex]``     http.reqDeniedAverage                     Integer    ブロックされたリクエストの平均
.10.61. ``[vhostMin]`` . ``[vhostIndex]``     http.reqDeniedCount                       Integer    ブロックされたリクエスト数
.11                                           portbypass                                OID        Portバイパスクライアントのトラフィック情報
.11.1. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.inbound                        Integer    Portバイパスを介してクライアントから受信平均トラフィック(Bytes)
.11.2. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.outbound                       Integer    Portバイパスを介してクライアントに送信平均トラフィック(Bytes)
.11.3. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.sessionAverage                 Integer    Portバイパスしているクライアントの平均セッション数
.11.4. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.closedAverage                  Integer    Portバイパス中のクライアントが接続を終了した平均回数
.11.5. ``[vhostMin]`` . ``[vhostIndex]``      portbypass.closedCount                    Integer    Portバイパス中のクライアントが接続を終了した回数
.12                                           ssl                                       OID        SSLクライアントのトラフィック情報
.12.2. ``[vhostMin]`` . ``[vhostIndex]``      ssl.inbound                               Integer    SSLを介してクライアントから受信平均トラフィック(Bytes)
.12.3. ``[vhostMin]`` . ``[vhostIndex]``      ssl.outbound                              Integer    SSLを介してクライアントに送信平均トラフィック(Bytes)
.13                                           requestHitAverage                         OID        平均キャッシュHIT結果
.13.1. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_HIT                 Integer    TCP_HIT
.13.2. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_IMS_HIT             Integer    TCP_IMS_HIT
.13.3. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_REFRESH_HIT         Integer    TCP_REFRESH_HIT
.13.4. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_REF_FAIL_HIT        Integer    TCP_REF_FAIL_HIT
.13.5. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_NEGATIVE_HIT        Integer    TCP_NEGATIVE_HIT
.13.6. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_MISS                Integer    TCP_MISS
.13.7. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_REFRESH_MISS        Integer    TCP_REFRESH_MISS
.13.8. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_CLIENT_REFRESH_MISS Integer    TCP_CLIENT_REFRESH_MISS
.13.9. ``[vhostMin]`` . ``[vhostIndex]``      requestHitAverage.TCP_DENIED              Integer    TCP_DENIED
.13.10. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_ERROR               Integer    TCP_ERROR
.13.11. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_REDIRECT_HIT        Integer    TCP_REDIRECT_HIT
.14                                           requestHitCount                           OID        キャッシュHIT結果数
.14.1. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_HIT                   Integer    TCP_HIT
.14.2. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_IMS_HIT               Integer    TCP_IMS_HIT
.14.3. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_REFRESH_HIT           Integer    TCP_REFRESH_HIT
.14.4. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_REF_FAIL_HIT          Integer    TCP_REF_FAIL_HIT
.14.5. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_NEGATIVE_HIT          Integer    TCP_NEGATIVE_HIT
.14.6. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_MISS                  Integer    TCP_MISS
.14.7. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_REFRESH_MISS          Integer    TCP_REFRESH_MISS
.14.8. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_CLIENT_REFRESH_MISS   Integer    TCP_CLIENT_REFRESH_MISS
.14.9. ``[vhostMin]`` . ``[vhostIndex]``      requestHitCount.TCP_DENIED                Integer    TCP_DENIED
.14.10. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_ERROR                 Integer    TCP_ERROR
.14.11. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_REDIRECT_HIT          Integer    TCP_REDIRECT_HIT
============================================= ========================================= ========== ==============================================================



.. _snmp-cache-vhost-traffic-filesystem:

cache.vhost.traffic.filesystem
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.3.1.11.20

仮想ホストのFile I / O統計を提供する。

=========================================== =========================================== ========== ==============================================
OID                                         Name                                        Type       Description
=========================================== =========================================== ========== ==============================================
.1. ``[vhostMin]`` . ``[vhostIndex]``       requestHitRatio                             Integer    Request Hit Ratio(100%)
.2. ``[vhostMin]`` . ``[vhostIndex]``                                                              Request Hit Ratio(10000%)
.3. ``[vhostMin]`` . ``[vhostIndex]``       byteHitRatio                                Integer    Byte Hit Ratio(100%)
.4. ``[vhostMin]`` . ``[vhostIndex]``                                                              Byte Hit Ratio(10000%)
.5. ``[vhostMin]`` . ``[vhostIndex]``       outbound                                    Integer    File I / Oデバイスに送信平均トラフィック (Bytes)
.6. ``[vhostMin]`` . ``[vhostIndex]``       session                                     Integer    File I / Oを実行中の平均Thread数
.7                                          requestHitAverage                           OID        平均キャッシュHIT結果
.7.1. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_HIT                   Integer    TCP_HIT
.7.2. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_IMS_HIT               Integer    TCP_IMS_HIT
.7.3. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_REFRESH_HIT           Integer    TCP_REFRESH_HIT
.7.4. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_REF_FAIL_HIT          Integer    TCP_REF_FAIL_HIT
.7.5. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_NEGATIVE_HIT          Integer    TCP_NEGATIVE_HIT
.7.6. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_MISS                  Integer    TCP_MISS
.7.7. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_REFRESH_MISS          Integer    TCP_REFRESH_MISS
.7.8. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_CLIENT_REFRESH_MISS   Integer    TCP_CLIENT_REFRESH_MISS
.7.9. ``[vhostMin]`` . ``[vhostIndex]``     requestHitAverage.TCP_DENIED                Integer    TCP_DENIED
.7.10. ``[vhostMin]`` . ``[vhostIndex]``    requestHitAverage.TCP_ERROR                 Integer    TCP_ERROR
.7.11. ``[vhostMin]`` . ``[vhostIndex]``    requestHitAverage.TCP_REDIRECT_HIT          Integer    TCP_REDIRECT_HIT
.8                                          requestHitCount                             OID        キャッシュHIT結果数
.8.1. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_HIT                     Integer    TCP_HIT
.8.2. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_IMS_HIT                 Integer    TCP_IMS_HIT
.8.3. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_REFRESH_HIT             Integer    TCP_REFRESH_HIT
.8.4. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_REF_FAIL_HIT            Integer    TCP_REF_FAIL_HIT
.8.5. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_NEGATIVE_HIT            Integer    TCP_NEGATIVE_HIT
.8.6. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_MISS                    Integer    TCP_MISS
.8.7. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_REFRESH_MISS            Integer    TCP_REFRESH_MISS
.8.8. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_CLIENT_REFRESH_MISS     Integer    TCP_CLIENT_REFRESH_MISS
.8.9. ``[vhostMin]`` . ``[vhostIndex]``     requestHitCount.TCP_DENIED                  Integer    TCP_DENIED
.8.10. ``[vhostMin]`` . ``[vhostIndex]``    requestHitCount.TCP_ERROR                   Integer    TCP_ERROR
.8.11. ``[vhostMin]`` . ``[vhostIndex]``    requestHitCount.TCP_REDIRECT_HIT            Integer    TCP_REDIRECT_HIT
.10. ``[vhostMin]`` . ``[vhostIndex]``      getattr.filecount                           Integer    （getattr関数呼び出し）FILEに応答した回数
.11. ``[vhostMin]`` . ``[vhostIndex]``      getattr.dircount                            Integer    （getattr関数呼び出し）DIRで応答した回数
.12. ``[vhostMin]`` . ``[vhostIndex]``      getattr.failcount                           Integer    （getattr関数呼び出し）が失敗に応答した回数
.13. ``[vhostMin]`` . ``[vhostIndex]``      getattr.timeres                             Integer    （getattr関数の呼び出し）の反応時間 (0.01ms)
.14. ``[vhostMin]`` . ``[vhostIndex]``      open.count                                  Integer    open関数の呼び出し回数
.15. ``[vhostMin]`` . ``[vhostIndex]``      open.timeres                                Integer    open関数の反応時間 (0.01ms)
.16. ``[vhostMin]`` . ``[vhostIndex]``      read.count                                  Integer    read関数の呼び出し回数
.17. ``[vhostMin]`` . ``[vhostIndex]``      read.timeres                                Integer    read関数の反応時間 (0.01ms)
.18. ``[vhostMin]`` . ``[vhostIndex]``      read.buffersize                             Integer    read関数で要求されたバッファサイズ (Bytes)
.19. ``[vhostMin]`` . ``[vhostIndex]``      read.bufferfilled                           Integer    read関数で要求されたバッファに詰めたサイズ (Bytes)
=========================================== =========================================== ========== ==============================================



.. _snmp-cache-vhost-traffic-dims:

cache.vhost.traffic.dims
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.3.1.11.21

仮想ホストのDIMS変換統計を提供する。

=========================================== =========================================== ========== ==============================================
OID                                         Name                                        Type       Description
=========================================== =========================================== ========== ==============================================
.1. ``[vhostMin]`` . ``[vhostIndex]``       requests                                    Integer    DIMS変換要求数
.2. ``[vhostMin]`` . ``[vhostIndex]``       converted                                   Integer    変換に成功回数
.3. ``[vhostMin]`` . ``[vhostIndex]``       failed                                      Integer    変換に失敗した回数
.4. ``[vhostMin]`` . ``[vhostIndex]``       avgsrcsize                                  Integer    元の画像の平均サイズ (Bytes)
.5. ``[vhostMin]`` . ``[vhostIndex]``       avgdestsize                                 Integer    変換された画像の平均サイズ (Bytes)
.6. ``[vhostMin]`` . ``[vhostIndex]``       avgtime                                     Integer    変換所要時間 (ms)
=========================================== =========================================== ========== ==============================================



.. _snmp-cache-vhost-traffic-compression:

cache.vhost.traffic.compression
---------------------

::

   OID = 1.3.6.1.4.1.40001.1.4.3.1.11.22

仮想ホストの圧縮統計を提供する。

=========================================== =========================================== ========== ==============================================
OID                                         Name                                        Type       Description
=========================================== =========================================== ========== ==============================================
.1. ``[vhostMin]`` . ``[vhostIndex]``       requests                                    Integer    圧縮要求数
.2. ``[vhostMin]`` . ``[vhostIndex]``       converted                                   Integer    圧縮成功回数
.3. ``[vhostMin]`` . ``[vhostIndex]``       failed                                      Integer    圧縮失敗回数
.4. ``[vhostMin]`` . ``[vhostIndex]``       avgsrcsize                                  Integer    元のファイルの平均サイズ (Bytes)
.5. ``[vhostMin]`` . ``[vhostIndex]``       avgdestsize                                 Integer    圧縮されたファイルの平均サイズ (Bytes)
.6. ``[vhostMin]`` . ``[vhostIndex]``       avgtime                                     Integer    圧縮時間 (ms)
=========================================== =========================================== ========== ==============================================



.. _snmp-cache-view:

cache.view
====================================

::

   OID = 1.3.6.1.4.1.40001.1.4.11.1

仮想ホストの統計と同じ情報を提供する。
``[viewIndex]`` は1からView数の範囲を持つ。

- 1.3.6.1.4.1.40001.1.4.3 - 仮想ホストの統計

- 1.3.6.1.4.1.40001.1.4.11 - View 統計
