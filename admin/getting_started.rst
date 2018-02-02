.. _getting-started:

第2章 開始する
******************

.. note::

   - `[動画講座]みよう！ STON Edge Server - Chapter 1.インストールと実行 <https://youtu.be/sMfp728pMtM?list=PLqvIfHb2IlKeZ-Eym_UPsp6hbpeF-a2gE>`_

この章では、システム構成からインストールおよびサンプルの仮想ホストまで構成してみる。 テキストエディタさえあれば誰でもすることができる。

STONは標準のLinuxサーバで動作するように開発されました。 開発段階からHWだけでなくOS、ファイルシステムなどの依存関係を持つことができる要素は、可能な限り排除した。 顧客が合理的な機器を選択できるように助けることは非常に重要である。 なぜならサービスの特性や規模に応じて適切なサーバーを構成することがサービスの開始だからだ。


.. toctree::
   :maxdepth: 2


.. _getting-started-serverconf:

サーバーの構成
====================================

一般的に、サーバーを構成するときは、CPU、メモリ、ディスクを主に考慮している。 10Gbps級の高い性能を必要とするサービスであれば、各構成要素がサービスの特性を満足しなければなら所望の性能を得ることができる。

-  **CPU**

   Quadコア以上を推奨する。 STONはMany-Coreの拡張性（Scalability）を有する。 コアが多ければ多いほど、毎秒スループットは増加する。 ただし、高スループットが必ずしも高いトラフィックを意味するものではない。

   .. figure:: img/10g_cpu.jpg
      :align: center

      クライアントが多いほど、多くのCPUは、力になる。

   4KBのファイルを約26万回を送信することと1GBのファイルを一度送信することは、同じ帯域幅を使用する。 CPUの選択の最大の基準は、どのように多くの同時接続を処理するかである。


-  **メモリ**

   Memory-Indexing方式を使用するため、4GB以上を推奨する。 ( :ref:`adv_topics_mem` を参照)
   頻繁にアクセスされるコンテンツは、常にメモリに常駐が、そうでないコンテンツは、ディスクからロードする。 したがって、コンテンツが多く集中度が低い場合（Long-tail）ディスク負荷の増加にパフォーマンスが低下することができる。 サービスされているコンテンツの数に関係なく、コンテンツサイズが大きく、ディスクI / Oの負荷が高い場合、メモリを増設して負荷を下げることができる。


-  **ディスク**

   OSを含む少なくとも3つ以上を推奨している。 ディスクも多ければ多いほど、多くのコンテンツをキャッシュすることができるだけでなく、I / Oの負荷も分散される。

   .. figure:: img/02_disk.png
      :align: center

      常にOSとSTONは、コンテンツとは別のディスクで構成する。

   一般的に、OSがインストールされてディスクにSTONを設置する。 ログも同じディスクに設定するのが一般的である。 ログは、サービスの状況をリアルタイムで記録するため、常にWrite負荷が発生する。

   STONは、ディスクをRAID 0のように使用する。 性能とRAIDの関係するかどうかは、顧客サービスの特性に応じて変わる。 しかし、ファイルの変更が頻繁せず、コンテンツのサイズが物理メモリのサイズよりもはるかに大きい場合は、RAIDを使用したRead速度の向上が効果的であることができる。


.. _getting-started-os:

OSの構成
====================================

最も基本的な形で設置する。 標準64bit Linuxディストリビューション（Cent 6.2以上、Ubuntu 10.04以上）であれば、正常に動作する。 パッケージの依存関係を持たない。


.. _getting-started-install:

インストール
====================================

1. 最新バージョンのSTONをダウンロードします。 ::

      [root@localhost ~]# wget  http://foobar.com/ston/ston.2.0.0.rhel.2.6.32.x64.tar.gz
      --2014-06-17 13:29:14--  http://foobar.com/ston/ston.2.0.0.rhel.2.6.32.x64.tar.gz
      Resolving foobar.com... 192.168.0.14
      Connecting to foobar.com|192.168.0.14|:80... connected.
      HTTP request sent, awaiting response... 200 OK
      Length: 71340645 (68M) [application/x-gzip]
      Saving to: “ston.2.0.0.rhel.2.6.32.x64.tar.gz”

      100%[===============================================>] 71,340,645  42.9M/s   in 1.6s

      2014-06-17 13:29:15 (42.9 MB/s) - “ston.2.0.0.rhel.2.6.32.x64.tar.gz” saved [71340645/71340645]


2. 圧縮を解除する。 ::

		[root@localhost ~]# tar -zxf ston.2.0.0.rhel.2.6.32.x64.tar.gz

3. インストールスクリプトを実行する。 ::

		[root@localhost ~]# ./ston.2.0.0.rhel.2.6.32.x64.sh

4. インストールプロセスは、install.logに記録される。 ログを使用してインストール中に発生する問題を知ることができる。 ::

      #DownloadURL: http://foobar.com/ston/ston.2.0.0.rhel.2.6.32.x64.tar.gz
      #DownloadTime: 13 sec
      #Target: STON 2.0.0
      #Date: 2014.03.03 16:48:35
      Prepare for STON 2.0.0 install process
          Stopping STON...
              STON stopped
      [Copying files]
          `./fuse.conf' -> `/etc/fuse.conf'
          `./libfuse.so.2' -> `/usr/local/ston/libfuse.so.2'
          `./libtbbmalloc_proxy.so' -> `/usr/local/ston/libtbbmalloc_proxy.so'
          `./start-stop-daemon' -> `/usr/sbin/start-stop-daemon'
          `./libtbbmalloc_proxy.so.2' -> `/usr/local/ston/libtbbmalloc_proxy.so.2'
          `./libtbbmalloc.so' -> `/usr/local/ston/libtbbmalloc.so'
          `./libtbbmalloc.so.2' -> `/usr/local/ston/libtbbmalloc.so.2'
          `./libtbb.so' -> `/usr/local/ston/libtbb.so'
          `./libtbb.so.2' -> `/usr/local/ston/libtbb.so.2'
          `./stond' -> `/usr/local/ston/stond'
          `./stonx' -> `/usr/local/ston/stonx'
          `./stonr' -> `/usr/local/ston/stonr'
          `./stonu' -> `/usr/local/ston/stonu'
          `./stonapi' -> `/usr/local/ston/stonapi'
          `./server.xml.default' -> `/usr/local/ston/server.xml.default'
          `./vhosts.xml.default' -> `/usr/local/ston/vhosts.xml.default'
          `./ston_format.sh' -> `/usr/local/ston/ston_format.sh'
          `./ston_diskinfo.sh' -> `/usr/local/ston/ston_diskinfo.sh'
          `./wm.sh' -> `/usr/local/ston/wm.sh'
      [Exporting config files]
          #Export so directory
          /usr/local/ston/ to ld.so.conf
          #Export sysctl to /etc/sysctl.conf
          vm.swappiness=0
          vm.min_free_kbytes=524288
          #Export sudoers for WM
          Defaults    !requiretty
          winesoft ALL=NOPASSWD: /etc/init.d/ston stop, /etc/init.d/ston start, /bin/ps -ef
      [Configuring STON daemon script]
          STON deamon activate in run-level 2345.
      [Installing sub-packages]
          curl installed.
          libjpeg installed.
          libgomp installed.
          rrdtool installed.
      [Installing WM]
          Stopping WM...
          WM stopped
          `./wm.server_default.xml' -> `/usr/local/ston/wm/tmp/conf/server_default.xml'
          `./wm.vhost_default.xml' -> `/usr/local/ston/wm/tmp/conf/vhost_default.xml'
          WM configuration found. Current WM port : 8500
          PHP module for Legacy(CentOS 5.5) installed
          `./libphp5.so.5.5' -> `/usr/local/ston/wm/modules/libphp5.so'
          WM installation almost complete. Changing WM privileges.
      Installation successfully complete


.. _getting-started-license:

ライセンス発行
====================================

新規顧客の場合、次の手順を使用してライセンスを発行する。

* `申込書 <http://ston.winesoft.co.kr/EULR.doc>`_ の作成
* license@winesoft.co.kr に転送
* 確認手続き後に発行さ

ライセンスファイル（license.xml）が必ずインストールパスに存在する必要がSTONが正常に駆動される。


.. _getting-started-update:

アップデート
====================================
最新バージョンが配布されると、stonuコマンドで更新することができる。 ::

	./stonu 2.0.1

または :ref:`wm` の :ref:`wm-update` を通じて簡単にアップデートを行うことができる。

   .. figure:: img/conf_update1.png
      :align: center


.. _getting-started-update-manual:

外部接続がされていない場合
------------------------------------------------

STONがインストールされるサーバーで外部接続がされていない場合は、次のようにマニュアル方式のアップデートが可能である。


.. コメント::

   インストール/アップデートは、必ずroot権限で行わなければならない。


1. 外部接続が可能なPCでSTONをダウンロードする。 ダウンロードURLは、公式releaseメールを介して配布される。


2. ダウンロードしたファイルをPCからサーバにコピーする。 ファイル名の形式は、次のとおりである。

   * RHEL/CentOS/openSUSE - ston. ``version`` .rhel.2.6.32.x64.tar.gz
   * Ubuntu - ston. ``version`` .ubuntu.2.6.32.x64.tar.gz


   CentOSのバージョン ``2.4.9`` であれば、ファイル名は ston.2.4.9.rhel.2.6.32.x64.tar.gz ある。


3. STONが実行中であれば停止しする。 ::

      service ston stop


4. サーバ内のコピーされたパスで圧縮を解除する。 ::

      tar zxvf ston.2.4.9.rhel.2.6.32.x64.tar.gz


   .. figure:: img/update_manual1.png
      :align: center

5. インストールスクリプトを実行する。 STONが実行中であれば、 "yes"を入力して停止しする。 ::

      sh ston.2.4.9.rhel.2.6.32.x64.sh


   .. figure:: img/update_manual2.png
      :align: center

6. インストールの完了後STONを開始します。 ::

      service ston start


   .. figure:: img/update_manual3.png
      :align: center


7. 正常駆動されたことを確認する。 ::

      ps -ef | grep ston


   .. figure:: img/update_manual4.png
      :align: center


.. _getting-started-run:

実行する
====================================

通常、デフォルトのパスにSTONを設置する。 ::

    /usr/local/ston/

次のファイルのいずれかが存在しないかXML構文に合わない場合は、実行されない。

- license.xml
- server.xml
- vhosts.xml

最初のインストール時に、すべてのXMLファイルが存在しない。 配布されたライセンスファイルをインストールパスにコピーします。 そしてインストールパスのserver.xml.defaultとvhosts.xml.defaultをコピーまたは変更して設定してほしい。
*.defaultファイルは、常に最新のパッケージと一緒に配布される。


.. _getting-started-samplevhost:

Hello World
====================================
vhosts.xml ファイルを開き、次のように編集する。 ::

    <Vhosts>
        <Vhost Name="www.example.com">
            <Origin>
                <Address>hello.winesoft.co.kr</Address>
            </Origin>
        </Vhost>
    </Vhosts>


.. _getting-started-runston:

STON実行
-----------------------------------------------
1. 発行されたlicense.xmlをインストールパスにコピーします。

2. server.xmlを開き、<Storage>を構成する。 ::

    <Server>
        <Cache>
            <Storage>
                <Disk>/cache1/</Disk>
                <Disk>/cache2/</Disk>
            </Storage>
        </Cache>
    </Server>

.. コメント::

   STONは、基本的に、ディスクをストレージとして使用するため、ディスクが設定されていない場合、駆動されない。 詳細については、次の章で説明する。

3. STONを実行する。  ::

      [root@localhost ~]# service ston start

   STONを停止したい場合はstopコマンドを使用します。  ::

      [root@localhost ~]# service ston stop


.. _getting-started-runcheck:

仮想ホストの動作確認
-----------------------------------------------

(Windows 7 ベース) C:\\Windows\\System32\\drivers\\etc\\hosts ファイルに次のようにwww.example.comドメインを設定する。 ::

    192.168.0.100        www.example.com

ブラウザでwww.example.comにアクセスしたときに、次のページが正常にサービスされると、成功である。

   .. figure:: img/helloworld3.png
      :align: center


.. _getting-started-rrderr:

WMが遅くなったり、グラフが出てこない問題
-----------------------------------------------

インストールプロセス中にRRDグラフは、動的にダウンロードしてインストールされる。 限られたネットワークでSTONをインストールする場合はRRDが正しくインストールされないことがあります。 これにより、 :ref:`wm` が非常にゆっくりと動作するか :ref:`api-graph` が動作しなくなる。 次のように変更する。


**1. rrdtoolのインストールの確認**

   次のようにインストールするかどうかを確認する。 ::

      [root@localhost ston]# yum install rrdtool
      Loaded plugins: fastestmirror, security
      Loading mirror speeds from cached hostfile
      * base: centos.mirror.cdnetworks.com
      * elrepo: ftp.ne.jp
      * epel: mirror.premi.st
      * extras: centos.mirror.cdnetworks.com
      * updates: centos.mirror.cdnetworks.com
      Setting up Install Process
      Package rrdtool-1.3.8-6.el6.x86_64 already installed and latest version
      Nothing to do

   次はUbuntu系の場合である。 ::

      root@ubuntu:~# apt-get install rrdtool
      Reading package lists... Done
      Building dependency tree
      Reading state information... Done
      rrdtool is already the newest version.
      The following packages were automatically installed and are no longer required:
        libgraphicsmagick3 libgraphicsmagick++3 libgraphicsmagick1-dev libgraphics-magick-perl libgraphicsmagick++1-dev
      Use 'apt-get autoremove' to remove them.
      0 upgraded, 0 newly installed, 0 to remove and 102 not upgraded.


**2. RRD手動インストール**

   もしrrdtoolがyumを使用してインストールがされていない場合は、OSのバージョンに合ったパッケージを `ダウンロードし <http://pkgs.repoforge.org/rrdtool/>`_ た後、手動でインストールする。

======================================== =================== ======= ============================
Name                                     Last Modified       Size    Description
======================================== =================== ======= ============================
tcl-rrdtool-1.4.7-1.el5.rf.i386.rpm      06-Apr-2012 16:57   36K     RHEL5 and CentOS-5 x86 32bit
tcl-rrdtool-1.4.7-1.el5.rf.x86_64.rpm	 06-Apr-2012 16:57   37K     RHEL5 and CentOS-5 x86 64bit
tcl-rrdtool-1.4.7-1.el6.rfx.i686.rpm     06-Apr-2012 16:57   35K     RHEL6 and CentOS-6 x86 32bit
tcl-rrdtool-1.4.7-1.el6.rfx.x86_64.rpm	 06-Apr-2012 16:57   35K     RHEL6 and CentOS-6 x86 64bit
======================================== =================== ======= ============================


.. _env-vhost-activeorigin:

ソースサーバー
============================================

仮想ホストは、元のサーバーに代わってコンテンツをサービスすることが目的である。 サービス形態に合わせて、さまざまなソースサーバーは、さまざまなアプローチが可能である。 ::

    <Vhosts>
        <Vhost Name="www.example.com">
            <Origin>
                <Address>1.1.1.1</Address>
                <Address>1.1.1.2</Address>
            </Origin>
        </Vhost>
    </Vhosts>

-  ``<Address>``
   仮想ホストがコンテンツを複製元サーバーのアドレス。 数の制限はない。 アドレスが2つ以上の場合は、Active / Active方式（Round-Robin）で選択される。 元サーバーのアドレス、ポートが80である場合、省略することができる。

例えば、別のポート（8080）でサービスされている場合、1.1.1.1:8080のようにポート番号を明示しなければならない。 アドレスは{IP | Domain} {Port} {Path}の形式で、8つの形式が可能である。

============================== ==========================
Address                        Hostヘッダ
============================== ==========================
1.1.1.1	                       仮想ホスト名
1.1.1.1:8080	               仮想ホスト名:8080
1.1.1.1/account/dir	       仮想ホスト名
1.1.1.1:8080/account/dir       仮想ホスト名:8080
example.com	                   example.com
example.com:8080	           example.com:8080
example.com/account/dir	       example.com
example.com:8080/account/dir   example.com:8080
============================== ==========================

:ref:`origin-httprequest` 中Hostヘッダを別途設定しない限り、表Hostヘッダを送信する。 ::

    <Vhosts>
        <Vhost Name="www.example.com">
            <Origin>
                <Address>origin.com:8888/account/dir</Address>
            </Origin>
        </Vhost>
    </Vhosts>

例えば上記のように設定すると、ソースとして、次のように要請する。 ::

   GET / HTTP/1.1
   Host: origin.com:8888

.. note:

   원본서버에 example.com/account/dir처럼 경로가 붙어있다면 요청된 URL은 원본서버 주소 경로 뒤에 붙는다.
   클라이언트가 /img.jpg를 요청하면 최종 주소는 example.com/account/dir/img.jpg가 된다.


.. _env-vhost-standbyorigin:

補助送信元アドレス
------------------------------------------------

補助ソースサーバーを設定する。 ::

    <Vhosts>
        <Vhost Name="www.example.com">
            <Origin>
                <Address>1.1.1.1</Address>
                <Address>1.1.1.2</Address>
                <Address2>1.1.1.3</Address2>
                <Address2>1.1.1.4</Address2>
            </Origin>
        </Vhost>
    </Vhosts>

-  ``<Address2>``

   すべての ``<Address>`` が正常に動作している場合 ``<Address2>`` はサービスに投入されない。 Activeサーバーに障害が検出されると、そのサーバーを置き換えるために投入され、Activeサーバーが復旧すると、再びStandby状態に戻る。 もしStandbyサーバに障害が検出されると、Standbyサーバが復旧するまでサービスに投入されない。


.. _api-etc-help:

API呼び出し
====================================

HTTPベースのAPIを提供する。 API呼び出しの権限は、 :ref:`env-host` の影響を受ける。 許可されていない場合、すぐに接続を終了する。

STONバージョンを確認する。 ::

   http://127.0.0.1:10040/version

同じAPIをLinux Shellからの命令に行う。 ::

   ./stonapi version

.. note:

   HTTP API는 &를 QueryString의 구분자로 인식하지만 Linux 콘솔에서는 다른 의미를 가진다.
   &가 들어가는 명령어를 호출하는 경우 \&로 입려하거나 반드시 괄호(" /...&... ")로 호출하는 URL을 묶어야 한다.


ハードウェア情報照会
====================================

ハードウェア情報を照会する。 ::

   http://127.0.0.1:10040/monitoring/hwinfo

結果は、JSON形式で提供される。 ::

   {
      "version": "1.1.9",
      "method": "hwinfo",
      "status": "OK",
      "result":
      {
         "OS" : "Linux version 3.3.0 ...(省略)...",
         "STON" : "1.1.9",
         "CPU" :
         {
            "ProcCount": "4",
            "Model": "Intel(R) Xeon(R) CPU           E5606  @ 2.13GHz",
            "MHz": "1200.000",
            "Cache": "8192 KB"
         },
         "Memory" : "8 GB",
         "NIC" :
         [
            {
               "Dev" : "eth1",
               "Model" : "Intel Corporation 82574L Gigabit Network Connection",
               "IP" : "192.168.0.13",
               "MAC" : "00:25:90:36:f4:cb"
            }
         ],
         "Disk" :
         [
            {
               "Dev" : "sda",
               "Model" : "HP DG0146FAMWL (scsi)",
               "Total" : "238787584",
               "Usage" : "40181760"
            },
            {
               "Dev" : "sdb",
               "Model" : "HITACHI HUC103014CSS600 (scsi)",
               "Total" : "144706478080",
               "Usage" : "2101075968"
            },
            {
               "Dev" : "sdc",
               "Model" : "HITACHI HUC103014CSS600 (scsi)",
               "Total" : "144706478080",
               "Usage" : "2012160000"
            }
         ]
      }
   }


再起動/シャットダウン
====================================

コマンドを使用してSTONを再起動/終了することができる。 予期しない結果を避けるために、Webページを介して確認作業が必要ように開発された。 ::

   http://127.0.0.1:10040/command/restart
   http://127.0.0.1:10040/command/restart?key=JUSTDOIT       // すぐに実行
   http://127.0.0.1:10040/command/terminate
   http://127.0.0.1:10040/command/terminate?key=JUSTDOIT       // すぐに実行


.. _getting-started-reset:

Caching初期化
====================================

サービスを中断し、キャッシュされたすべてのコンテンツを削除する。 設定されたすべてのディスクをフォーマットし作業が完了したら、再度サービスを再開する。 ::

   http://127.0.0.1:10040/command/cacheclear
   http://127.0.0.1:10040/command/cacheclear?key=JUSTDOIT       // すぐに実行

コンソールでは、次のコマンドを使用して全体または1つの仮想ホストを初期化する。 ::

   ./stonapi reset
   ./stonapi reset/ston.winesoft.co.kr
