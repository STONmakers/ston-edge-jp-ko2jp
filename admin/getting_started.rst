.. _getting-started:

第2章 開始する
******************

.. note::

   - `[動画講座]みよう！ STON Edge Server - Chapter 1.インストールと実行 <https://youtu.be/sMfp728pMtM?list=PLqvIfHb2IlKeZ-Eym_UPsp6hbpeF-a2gE>`_
   - `[Q&A] STON Edge Serverの2ティアの構成動作方式について知りたいです。 <https://www.youtube.com/watch?v=0Q6OgTadtHM&index=5&list=PLqvIfHb2IlKfRKtHg7vEZtrP7web63sW8>`_


この章ではシステム構成からインストールおよびサンプルの仮想ホストまで構成してみます。 テキストエディタがあれば誰でもすることができます。

STONは標準のLinuxサーバで動作するように開発されました。 開発段階からHWだけでなくOSファイルシステムなどの依存関係を持たないように設計しました。顧客が合理的な機器を選択できるように助けることは非常に重要です。 サービスの特性や規模に応じて適切なサーバーを構成することがサービス開始の大事な一歩になるためです。


.. toctree::
   :maxdepth: 2


.. _getting-started-serverconf:

サーバーの構成
====================================

一般的にサーバーを構成するときはCPUメモリディスクを主に考慮します。 10Gbps級の高い性能を必要とするサービスであれば各構成要素がサービスの特性を満足しなければなら所望の性能を得ることができます。

-  **CPU**

   Quadコア以上を推奨します。 STONはMany-Core Scalabilityシステムです。 コアが多ければ多いほど毎秒スループットは増加します。 ただし高スループットが必ずしも高いトラフィックを意味することではありません。

   .. figure:: img/10g_cpu.jpg
      :align: center

      クライアントが多いほど多くのCPUは力になります。

   4KBのファイルを約26万回を送信することと1GBのファイルを一度送信することは同じ帯域幅を使用します。 CPUの選択の最大の基準はどのように多くの同時接続を処理するかです。


-  **メモリ**

   Memory-Indexing方式を使用するため4GB以上を推奨します。 ( :ref:`adv_topics_mem` を参照)
   頻繁にアクセスされるコンテンツはメモリに常駐しますがそうではないコンテンツはディスクからロードします。 したがってコンテンツが多く集中度が低い場合（Long-tail）ディスク負荷の増加しパフォーマンスが低下する場合があります。 サービスされているコンテンツの数に関係なくコンテンツサイズが大きくディスクI / Oの負荷が高い場合メモリを増設して負荷を下げることができます。


-  **ディスク**

   OSを含む少なくとも3つ以上を推奨します。 ディスクも多ければ多いほど多くのコンテンツをキャッシュすることができるだけでなくI / Oの負荷も分散されます。

   .. figure:: img/02_disk.png
      :align: center

      必ずOSとSTONはコンテンツとは別のディスクで構成します

   一般的にOSがインストールされてディスクにSTONを設置します。 ログも同じディスクに設定するのが一般的です。 ログはサービスの状況をリアルタイムで記録するためWrite負荷が発生します。

   STONはディスクをRAID 0のように使用します。 性能とRAIDの関係有無は顧客サービスの特性に応じて変わります。 しかしファイルの変更が頻繁せずコンテンツのサイズが物理メモリのサイズよりもはるかに大きい場合はRAIDを使用したRead速度の向上が効果的な場合があります。


.. _getting-started-os:

OSの構成
====================================

最もデフォルト的な形で設置します。 標準64bit Linuxディストリビューション（Cent 6.2以上Ubuntu 10.04以上）であれば正常に動作します。パッケージの依存関係はありません。


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


2. 圧縮を解凍します。 ::

		[root@localhost ~]# tar -zxf ston.2.0.0.rhel.2.6.32.x64.tar.gz

3. インストールスクリプトを実行します。 ::

		[root@localhost ~]# ./ston.2.0.0.rhel.2.6.32.x64.sh

4. インストールプロセスはinstall.logに記録されます。 ログを使用してインストール中に発生する問題を検知することができます。 ::

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

新規顧客の場合次の手順でライセンスを発行します。

* `申込書 <http://ston.winesoft.co.kr/EULR.doc>`_ の作成
* license@winesoft.co.kr に転送
* 確認手続き後ライセンス発行

ライセンスファイル（license.xml）は必ずインストールパスに存在する必要がSTONが正常に駆動されます。


.. _getting-started-update:

アップデート
====================================
最新バージョンが配布されるとstonuコマンドで更新できます。 ::

	./stonu 2.0.1

または :ref:`wm` の :ref:`wm-update` を通じて簡単にアップデートを行うことができます。

   .. figure:: img/conf_update1.png
      :align: center


.. _getting-started-update-manual:

外部接続ができない場合
------------------------------------------------

STONがインストールされるサーバーで外部接続がされていない場合は次のようにマニュアル方式のアップデートが可能です。


.. note::

   インストール/アップデートは必ずroot権限で行う必要があります。


1. 外部接続が可能なPCでSTONをダウンロードします。 ダウンロードURLは公式releaseメールを介して配布されます。


2. ダウンロードしたファイルをPCからサーバにコピーします。 ファイル名の形式は次のとおりです。

   * RHEL/CentOS/openSUSE - ston. ``version`` .rhel.2.6.32.x64.tar.gz
   * Ubuntu - ston. ``version`` .ubuntu.2.6.32.x64.tar.gz


   CentOSのバージョン ``2.4.9`` であればファイル名は ston.2.4.9.rhel.2.6.32.x64.tar.gz です。


3. STONが実行中であれば停止します。 ::

      service ston stop


4. サーバ内のコピーされたパスで圧縮を解除します。 ::

      tar zxvf ston.2.4.9.rhel.2.6.32.x64.tar.gz


   .. figure:: img/update_manual1.png
      :align: center

5. インストールスクリプトを実行します。 STONが実行中であれば 「yes」を入力して停止します。 ::

      sh ston.2.4.9.rhel.2.6.32.x64.sh


   .. figure:: img/update_manual2.png
      :align: center

6. インストールの完了後STONを開始します。 ::

      service ston start


   .. figure:: img/update_manual3.png
      :align: center


7. 正常に起動されたことを確認します。 ::

      ps -ef | grep ston


   .. figure:: img/update_manual4.png
      :align: center


.. _getting-started-run:

実行する
====================================

通常デフォルトのパスにSTONを設置します。 ::

    /usr/local/ston/

次のファイルのいずれかが存在しないかXML構文に合わない場合は実行できません。

- license.xml
- server.xml
- vhosts.xml

最初のインストール時にすべてのXMLファイルはありません。 配布されたライセンスファイルをインストールパスにコピーします。 そしてインストールパスのserver.xml.defaultとvhosts.xml.defaultをコピーまたは変更して設定します。 *.defaultファイルは常に最新のパッケージと一緒に配布されます。


.. _getting-started-samplevhost:

Hello World
====================================
vhosts.xml ファイルを開き次のように編集します。 ::

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

2. server.xmlを開き<Storage>を構成します。 ::

    <Server>
        <Cache>
            <Storage>
                <Disk>/cache1/</Disk>
                <Disk>/cache2/</Disk>
            </Storage>
        </Cache>
    </Server>

.. note::

   STONはデフォルト的にディスクをストレージとして使用するためディスクが設定されていない場合起動しません。 詳細については次の章で説明します。 

3. STONを実行します。  ::

      [root@localhost ~]# service ston start

   STONを停止したい場合はstopコマンドを使用します。  ::

      [root@localhost ~]# service ston stop


.. _getting-started-runcheck:

仮想ホストの動作確認
-----------------------------------------------

(Windows 7 ベース) C:\Windows\System32\drivers\etc\hosts ファイルに次のようにwww.example.comドメインを設定する。 ::

    192.168.0.100        www.example.com

ブラウザでwww.example.comにアクセスしたときに次のページが正常にサービスされると成功です。

   .. figure:: img/helloworld3.png
      :align: center


.. _getting-started-rrderr:

WMが遅くなったりグラフが表示されない問題
-----------------------------------------------

インストールプロセス中にRRDグラフは動的にダウンロードしてインストールされます。 限られたネットワークでSTONをインストールする場合はRRDが正しくインストールされないことがあります。 これにより :ref:`wm` が非常に遅くなるか :ref:`api-graph` が動作しません。次のようにrrdtoolを設置して修正します。


**1. rrdtoolのインストールの確認**

   次のようにインストール有無を確認します。 ::

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

   次はUbuntu系の場合です。 ::

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

   もしrrdtoolがyumを使用してインストールがされていない場合はOSのバージョンに合ったパッケージを `ダウンロードし <http://pkgs.repoforge.org/rrdtool/>`_ た後手動でインストールします。

======================================== =================== ======= ============================
Name                                     Last Modified       Size    Description
======================================== =================== ======= ============================
tcl-rrdtool-1.4.7-1.el5.rf.i386.rpm      06-Apr-2012 16:57   36K     RHEL5 and CentOS-5 x86 32bit
tcl-rrdtool-1.4.7-1.el5.rf.x86_64.rpm	 06-Apr-2012 16:57   37K     RHEL5 and CentOS-5 x86 64bit
tcl-rrdtool-1.4.7-1.el6.rfx.i686.rpm     06-Apr-2012 16:57   35K     RHEL6 and CentOS-6 x86 32bit
tcl-rrdtool-1.4.7-1.el6.rfx.x86_64.rpm	 06-Apr-2012 16:57   35K     RHEL6 and CentOS-6 x86 64bit
======================================== =================== ======= ============================


.. _env-vhost-activeorigin:

オリジンサーバー
============================================

仮想ホストはオリジンサーバーに代わってコンテンツをサービスすることが目的です。 サービス形態に合わせて多様なオリジンサーバーで多様なアプローチが可能です。 ::

    <Vhosts>
        <Vhost Name="www.example.com">
            <Origin>
                <Address>1.1.1.1</Address>
                <Address>1.1.1.2</Address>
            </Origin>
        </Vhost>
    </Vhosts>

-  ``<Address>``
   仮想ホストがコンテンツを複製するオリジンサーバーのアドレス。 数の制限はない。 アドレスが2つ以上の場合はActive / Active方式（Round-Robin）で選択されます。 オリジンサーバーのポートが80の場合は省略することができます。

例えば別のポート（8080）でサービスされている場合1.1.1.1:8080のようにポート番号を明示する必要があります。 アドレスは{IP | Domain} {Port} {Path}の形式で以下の8つの表現が可能です。

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

:ref:`origin-httprequest` 中Hostヘッダを別途設定しない限り表Hostヘッダを送信します。 ::

    <Vhosts>
        <Vhost Name="www.example.com">
            <Origin>
                <Address>origin.com:8888/account/dir</Address>
            </Origin>
        </Vhost>
    </Vhosts>

例えば上記のように設定するとオリジンサーバーに次のように要請します。 ::

   GET / HTTP/1.1
   Host: origin.com:8888

.. note::

   オリジンサーバーにexample.com/account/dirようパスがついている場合は要求されたURLはオリジンサーバーのアドレスのパスの後ろにつく。 クライアントが/img.jpgを要求すると最終的なアドレスはexample.com/account/dir/img.jpgになる。


.. _env-vhost-standbyorigin:

サブオリジンサーバーアドレス
------------------------------------------------

サブオリジンサーバーを設定します。 ::

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

   すべての ``<Address>`` が正常に動作する(Active)場合 ``<Address2>`` はサービスに投入されない(Standby)。 Activeサーバーに障害が検出されるとそのサーバーを置き換えるために投入されてActiveサーバーが復旧すると再びStandby状態に戻る。 もしStandbyサーバに障害が検出されるとStandbyサーバが復旧するまでサービスに投入されない。


.. _api-etc-help:

API呼び出し
====================================

HTTPベースのAPIを提供します。 API呼び出しの権限は :ref:`env-host` の影響を受けます。 許可されていない場合すぐに接続を終了します。

STONバージョンを確認します。 ::

   http://127.0.0.1:10040/version

同じAPIをLinux Shellからのコマンドで実行します。 ::

   ./stonapi version

.. note::

   HTTP APIは＆をQueryStringの区切り文字として認識しますがLinuxコンソールでは別の意味があります。 ＆が入るコマンドを呼び出す場合は＆にとして記入するか必ず括弧（"/...＆..."）で呼び出すURLを囲む必要があります。


ハードウェア情報照会
====================================

ハードウェア情報を照会します。 ::

   http://127.0.0.1:10040/monitoring/hwinfo

結果はJSON形式で提供されます。 ::

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

コマンドを実行してSTONを再起動/終了することができます。 予期しない結果を避けるためにWebページを介して確認作業が必要ように開発された。 ::

   http://127.0.0.1:10040/command/restart
   http://127.0.0.1:10040/command/restart?key=JUSTDOIT       // すぐに実行
   http://127.0.0.1:10040/command/terminate
   http://127.0.0.1:10040/command/terminate?key=JUSTDOIT       // すぐに実行


.. _getting-started-reset:

Caching初期化
====================================

サービスを中断しキャッシュされたすべてのコンテンツを削除します。 設定されたすべてのディスクをフォーマットする作業の完了後サービスを再開します。 ::

   http://127.0.0.1:10040/command/cacheclear
   http://127.0.0.1:10040/command/cacheclear?key=JUSTDOIT       // すぐに実行

コンソールでは次のコマンドを使用して全体または1つの仮想ホストを初期化します。 ::

   ./stonapi reset
   ./stonapi reset/ston.winesoft.co.kr
