.. _cacti:

Appendix B：Cacti監視 
*************************

この章では、 `Cacti <http://www.cacti.net/>`_ のGraph Treeを使用して、多数のSTONを統合監視する方法について説明します。 次の2つの条件が前提となります。

-  Cactiをインストールするサーバー 
-  SNMPの有効化 ( :ref:`snmp` 参照)


.. toctree::
   :maxdepth: 2


.. _cacti_template:

Template追加
====================================

STONで提供されるHost Templateを使用すると、監視環境を容易に構築することができます。
( `ダウンロード  <http://webhard.winesoft.co.kr/ston/monitoring/cacti/ston_host_template.xml>`_ )

.. figure:: img/cacti01.png
   :align: center

   Import Templatesメニューを選択します。

.. figure:: img/cacti02.png
   :align: center

   cacti_host_template_ston.xmlをImportします。


.. _cacti_device_add:

Device登録
====================================

STONをCactiのDeviceに登録します。

.. figure:: img/cacti03.png
   :align: center

   [Devices]メニューを選択します。

.. figure:: img/cacti04.png
   :align: center

   [Devices]メニューの[Add]ボタンをクリックします。

.. figure:: img/cacti05.png
   :align: center

   Devices項目を作成します。


-  ① 対象STONの名前を作成します。
-  ② 対象STONのIPアドレスを入力します。
-  ③ ”STON” を選択します。
-  ④ “Public” を選択します。
-  ⑤ デフォルトのポート161を入力します。


Createボタンをクリックして、Deviceを連動します。

.. figure:: img/cacti06.png
   :align: center

   正常連動された。

.. figure:: img/cacti07.png
   :align: center

   連動に失敗しました。

.. note::

   SNMP連動失敗時

   -  STONのSNMPが有効になっていることを確認します。
   -  SNMP Port番号がSTONのSNMP Port番号と一致することを確認します。


Device連動に成功するとSTON Templateで提供される18種類の項目のグラフを使用することができます。

.. figure:: img/cacti08.png
   :align: center

   "Create Graphs for this Host" リンクをクリックします。

.. figure:: img/cacti09.png
   :align: center

   18種類のグラフが提供される。

[Create] ボタンをクリックして、生成されたグラフを確認します。

.. figure:: img/cacti10.png
   :align: center

   グラフが作成された。


.. _cacti_graph_tree:

Graph Tree生成 
====================================

Graph Treeを生成します。

.. figure:: img/cacti11.png
   :align: center

   [Graph Trees] をクリックします。

.. figure:: img/cacti12.png
   :align: center

   右上の [Add]をクリックします。

.. figure:: img/cacti13.png
   :align: center

   Graph Tree生成します。


STONをGraph Treeに追加します。

.. figure:: img/cacti14.png
   :align: center

   [Tree Items] メニューから[Add]をクリックします。

.. figure:: img/cacti15.png
   :align: center

   [Tree Items]項目を作成します。


-  ①	“Host”を選択します
-  ②	追加する “Devices” を選択します。
-  ③	“Graph Template” を選択します。


.. _cacti_graph_confirm:

Graphs確認
====================================

左上の [graphs] メニューをクリックして、グラフが正常に出るかを確認します。

.. figure:: img/cacti16.png
   :align: center

   定期的に、動作を確認します。
