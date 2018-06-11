.. _release_enterprise:

Appendix E: リリースノート  ``[Enterprise]``
***********************

v18.x
====================================

18.05.1 (2018.5.29)
----------------------------

**機能改善/ポリシーの変更**

- :ref:`media-hls` - キーフレームの間隔が不規則な映像の互換性を強化

.. warning::

   以前のバージョンと :ref:`media-hls` のMPEG2-TSは互換性がありません。


**バグ修正**

 -  :ref:`handling_http_requests_header_lastmodifiedcheck` - ``orlater`` に設定する場合、最初のキャッシュ時に304応答をすることができる問題を修正


18.05.0 (2018.5.15)
----------------------------

-  クライアントの要求 :ref:`handling_http_requests_header_if_range` ヘッダサポート 
-  元の要求時 :ref:`origin_header_if_range` ヘッダサポート
-  :ref:`handling_http_requests_header_lastmodifiedcheck` 設定機能を追加
-  :ref:`bypass-put` 機能を追加



18.04.0 (2018.4.26)
----------------------------

**機能改善/ポリシーの変更**

- :ref:`media-dims` - :ref:`media-dims-annotation` 機能を追加


.. note::

   v2.5.13以降新しいVersioningに提供されます。

   -  ``CDN`` - v2.5.14のような既存のVersioning
   -  ``Enterprise`` - v.18.04.0のような年/月に形の新しいVersioning
