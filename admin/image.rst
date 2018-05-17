.. _media-dims:

第17章 画像/ DIMS
******************

.. note::

   - `[動画講座]みよう！ STON Edge Server - Chapter 4.リアルタイムイメージ処理  <https://youtu.be/Pdfe-HbtXVs?list=PLqvIfHb2IlKeZ-Eym_UPsp6hbpeF-a2gE>`_

この章ではイメージを転送時点でon-the-flyに変換/転送するDIMS（ディムス）について取り上げる。 DIMS（Dynamic Image Management System）は元のイメージを様々な形に加工する機能です。

.. figure:: img/dims.png
   :align: center

   多様な動的イメージ処理

イメージは動的に生成され元のイメージのURLの後ろに規定のキーワードと加工オプションを付けて呼び出します。 加工されたイメージはキャッシュされてオリジンサーバーのイメージが変わらない限り再処理されません。

元のファイルが/img.jpgの場合、次のような形式でイメージを加工することができます。
("12AB"は規定のKeywordです。) ::

   http://image.example.com/img.jpg    // origin content
   http://image.example.com/img.jpg/12AB/optimize
   http://image.example.com/img.jpg/12AB/resize/500x500/
   http://image.example.com/img.jpg/12AB/crop/400x400/
   http://image.example.com/img.jpg/12AB/composite/watermark1/

``<Dims>`` は別に設定しなければすべて無効にされている。 ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <Dims Status="Active" Keyword="dims" MaxSourceSize="10" OnFailure="message" />

-  ``<Dims>``

   - ``Status`` DIMS有効 ( ``Active`` または ``Inactive`` )
   - ``Keyword`` オリジンサーバーとDIMSを区別するキーワード 
   - ``MaxSourceSize (デフォルト: 10MB)`` 変換可能な最大イメージサイズ（単位：MB）
   - ``OnFailure`` イメージ変換失敗時の動作方式

     - ``message (デフォルト)`` 500 Internal Errorで応答します。 本文には具体的な失敗の理由を明示します。

       - ``The original file was not successfully downloaded.`` 元のイメージを完全にダウンロードできなかった。
       - ``The original file size is too large.`` 元イメージのサイズが ``MaxSourceSize`` を超えて変換していなかった。
       - ``The original file loading failed.`` 元のイメージデータを読み込まなかった。
       - ``Image converting failed or invalid DIMS command.`` 正しくない命令またはサポートされていないイメージなどが原因で変換していなかった。

     - ``redirect`` 元のイメージのアドレスに302 Redirectします。



.. toctree::
   :maxdepth: 2


最適化
====================================

品質を低下せずにイメージを圧縮します。 JPEG、JPEG-2000、Loseless-JPEGイメージのみをサポートが可能です。 既に他のツールを使用して最適化されたイメージは最適化されない。 ::

   http://image.example.com/img.jpg/dims/optimize

最適化はキーワード以外の別のオプションはありません。 他の変換条件と組み合わせたときに一番後ろ明示する方法が望ましいです。 ::

   http://image.example.com/img.jpg/dims/resize/100x100/optimize

すべてのDIMSの機能はシステムリソースを大量に使用しますが、その中でも最適化が最も重い作業です。 以下はHitRatioが0％の状態でイメージサイズ別パフォーマンステストの結果です。

-  ``OS`` CentOS 6.2 (Linux version 2.6.32-220.el6.x86_64 (mockbuild@c6b18n3.bsys.dev.centos.org) (gcc version 4.4.6 20110731 (Red Hat 4.4.6-3) (GCC) ) #1 SMP Tue Dec 6 19:48:22 GMT 2011)
-  ``CPU`` `Intel(R) Xeon(R) CPU E3-1230 v3 @ 3.30GHz (8 processors) <http://www.cpubenchmark.net/cpu.php?cpu=Intel+Xeon+E3-1230+v3+%40+3.30GHz>`_
-  ``RAM`` 16GB
-  ``HDD`` SMC2108 SAS 275GB X 3EA

====== ============ ============= ================================== ====================== =====================
サイズ スループット 応答速度(ms)    クライアントのトラフィック(Mbps)   元トラフィック(Mbps)     トラフィックの削減率(%)
====== ============ ============= ================================== ====================== =====================
16KB   720          19.32         46.32                              92.62                  49.99
32KB   680          20.68         86.42                              165.08                 47.65
64KB   285          50.16         80.67                              150.96                 46.56
128KB  274          57.80         164.35                             276.52                 40.56
256KB  210          80.74         99.42                              432.35                 77.00
512KB  113          156.18        160.54                             436.04                 63.18
1MB    20           981.07        90.62                              179.88                 49.62
====== ============ ============= ================================== ====================== =====================

約50％内外のトラフィックを削減がでいるのでで非常に有効です。最適化は非常に重い作業です。 参照表でわかるようにイメージサイズの影響が大きいです。

そのためサービスに適用するためには十分な検討が必須になります。 適切な :ref:`adv_topics_req_hit_ratio` がある状況が望ましいがそうでない場合はサービスの規模に合わせて物理的なCPUリソースを十分に確保する必要があります。


.. note::

   メタ情報のみを削除したい場合は以下のコマンドを使用します。 ::

      http://image.example.com/img.jpg/dims/strip/true



カット (Cropping)
====================================

左上を基準でイメージを切り取ります。 範囲は **width x height{+-}x{+-}y{%}** で表現します。 次は左上端x = 20、y = 30を基準にwidth = 100、height = 200だけ切り取る例だ。 ::

   http://image.example.com/img.jpg/dims/crop/100x200+20+30/


イメージの中央を基準にしたい場合cropcenterコマンドを使用します。 ::

   http://image.example.com/img.jpg/dims/cropcenter/100x200+20+30/



Thumbnail生成 
====================================

Thumbnailを生成します。 サイズとオプションは **width x height{%} {@} {!} {<} {>}** で表現します。
デフォルトはイメージの横と縦の最大値を使用します。 イメージを拡大または縮小してもアスペクト比は維持されます。 サイズを指定する場合は（！）を追加します。
**640X480!** という表現は正確に640x480サイズのThumbnailを生成します。 もし水平方向または垂直方向のサイズのみ指定した場合省略された値は水平/垂直比によって自動的に決定されます。

例えば **/thumbnail/100/** は横幅に合わせて縦サイズが決定され
**/thumbnail/x200/** は縦サイズに合わせて横幅が決定されます。 水平/垂直サイズをイメージのサイズに合わせて割合（％）で表現することができます。 イメージのサイズを増やすには100よりも大きい値（例えば125％）を使用します。 イメージのサイズを小さくするには100未満の割合を使用します。 URL Encodingルールに基づいて％の文字が％25にエンコードされることを覚えておかなければならない。

例えば50％という表現は50％25でエンコードされます。 以下はwidth = 78、height = 110サイズのThumbnailを生成する例である。 ::

   http://image.example.com/img.jpg/dims/thumbnail/78x110/


Resizing
====================================

イメージのサイズを変更します。
サイズは **width x height** で表現します。
イメージは変更されても比率は維持されます。 以下は元のイメージをwidth = 200、height = 200サイズに変更する例です。 ::

   http://image.example.com/img.jpg/dims/resize/200x200/

その他のコマンドは、次のとおりである。

-  **resizec** - 縮小するとresizeと同じですが拡大するとイメージは維持されてキャンバスサイズだけ拡大されます。
-  **extent** - キャンバスのみ調節するコマンドです。 縮小するとcropと同じ効果を出すが、拡大するとresizecと同じように拡大される。 
-  **trim** - 上下左右の白い背景を削除します。



Format変更
====================================

イメージフォーマットを変更します。
サポートされるフォーマットは "png"、 "jpg"、 "gif" です。 以下はJPGをPNGへ変換する例です。 ::

   http://image.example.com/img.jpg/dims/format/png/


品質変更
====================================

画質を調節します。 この機能で送信されるイメージの容量が削減できます。 有効範囲は0から100までだ。 次はイメージの品質を25％に調節する例です。 ::

   http://image.example.com/img.jpg/dims/quality/25/


エフェクト
====================================

イメージに多様なエフェクトを与えることができます。

================ ===================== =================
説明             コマンド              変数 
================ ===================== =================
反転             invert                 true または false
グレースケール   grayscale             true または false
反転             flipflop              vertical または horizontal
明るさ         bright                0 ~ 100
回転             rotate                0 ~ 360 (度)
セピア           sepia                 0 ~ 1
ラウンドエッ      round                 0 ~ 90
================ ===================== =================



合成
====================================

二つのイメージを合成します。 前述の機能とは別の方法で合成条件はあらかじめ設定されてなければならない。 主にウォーターマーク効果を出すために使用されます。 ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <Dims Status="Active" Keyword="dims" port="8500">
      <Composite Name="water1" File="/img/small.jpg" />
      <Composite Name="water2" File="/img/medium.jpg" Gravity="se" Geometry="+0+0" Dissolve="50" />
      <Composite Name="water_ratio" File="/img/wmark_s.png" Gravity="s" Geometry="+0+15%" Dissolve="100" />
   </Dims>

-  ``<Composite>``

    イメージ合成条件を設定します。 属性によって決まり別の値を持たない。

    -  ``Name`` 呼び出される名前を指定します。
       '/'文字は入力できない。
       URLの "/composite/" の後に位置します。

    -  ``File`` 合成するイメージファイルのパスを指定します。

    -  ``Gravity (デフォルト: c)`` 合成する位置は左上から9つのポイント(nw, n, ne, w, c, e, sw, s, se)が存在します。

       .. figure:: img/conf_dims2.png
          :align: center

          Gravity基準点

    -  ``Geometry (デフォルト: +0+0)`` ``Gravity`` 基準で合成するイメージの位置を設定します。 {+ - } x {+ - } y。 赤丸はGravity属性に基づいて+ 0 + 0が意味する基準点に+ x + yの値が大きくなるほどイメージの中に配置されます。 緑の矢印は+ x紫の矢印は+ yが増加する方向です。 -x-yを使用すると対象イメージの外側に位置するようにされ結果のイメージは見られない。 この属性は多少複雑に見えますがイメージのサイズを自動的に計算して配置するので一貫性のある結果を得ることができて有効です。 また+ x％+ y％のように％オプションを与え割合で配置することもできる。

    -  ``Dissolve (デフォルト: 50)`` 合成するイメージの透明度(0~100).

``<Composite>`` を設定した場合 ``Name`` プロパティを使用してイメージを合成することができます。 ::

    http://image.example.com/img.jpg/dims/composite/water1/



オリジンのイメージの条件判断
====================================

元のイメージの条件に応じて動的に加工オプションを別の方法で適用することができます。 たとえば1024 X 768以下のイメージは品質を50％に落としそれ以上のイメージは1024 X 768にサイズ変換をするには次のように ``<ByOriginal>`` を設定します。 ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <Dims Status="Active" Keyword="dims" port="8500">
      <ByOriginal Name="size1">
         <Condition Width="1024" Height="768">/quality/50/</Condition>
         <Condition>/resize/1024x768/</Condition>
      </ByOriginal>
   </Dims>

-  ``<ByOriginal>``
   ``Name`` 属性で呼び出します。 サブ多様な条件の ``<Condition>`` を設定します。

-  ``<Condition>``
   条件に満足している場合設定された変換を実行します。

   - ``Width`` 幅が設定値よりも小さい場合に適用されます。
   - ``Height`` 縦の長さが設定値よりも小さい場合に適用されます。

   条件を設定しないと元のイメージのサイズに関係なく変換されます。

``<Condition>`` は指定された順序で適用されます。 したがって小さなイメージの条件を最初に配置する必要があります。 次のように呼び出したら原本イメージの大きさに準じて合成が適用されます。 ::

   http://image.example.com/img.jpg/dims/byoriginal/size1/

別の例としてイメージサイズに応じて他の ``<Composite>`` 条件を与えることができます。 このような場合は次のように事前に定義された ``<Composite>`` の ``Name`` に設定します。 ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <Dims Status="Active" Keyword="dims" port="8500">
      <Composite Name="water1" File="/img/small.jpg" />
      <Composite Name="water2" File="/img/medium.jpg" Gravity="se" Geometry="+0+0" Dissolve="50" />
      <Composite Name="water3" File="/img/big.jpg" Gravity="se" Geometry="+10+10" Dissolve="50" />
      <ByOriginal Name="size_water">
         <Condition Width="400">/composite/water1/</Condition>
         <Condition Width="800">/composite/water2/</Condition>
         <Condition>/composite/water3/</Condition>
      </ByOriginal>
   </Dims>

次のように呼び出すと元のイメージのサイズに応じて合成が適用されます。 ::

   http://image.example.com/img.jpg/dims/byoriginal/size_water/


.. _media-dims-anigif:

Animated GIF
====================================

Animated GIFにもすべてのDIMS変換が同様に適用されます。 処理順序は次のとおりです。

1. Animated GIFを別途のイメージに分解します。
2. それぞれのイメージを変換します。
3. 変換されたイメージをAnimated GIFに結合します。

結合されたイメージが多いほど処理コストが高くサービスの品質が低下することができます。 このような場合最初のイメージにのみ変換するように設定すると処理コストを下げることができます。 ::

   # server.xml - <Server><VHostDefault><Options>
   # vhosts.xml - <Vhosts><Vhost><Options>

   <Dims FirstFrameOnly="OFF" />

-  ``FirstFrameOnly (デフォルト: OFF)`` ONの場合Animated GIFの最初のシーンだけを変換します。

次のようにURLを呼び出すときに ``FirstFrameOnly`` オプションを明示的に指定することができます。 ::

   http://image.example.com/img.jpg/dims/firstframeonly/on/resize/200x200/
   http://image.example.com/img.jpg/dims/firstframeonly/off/resize/200x200/

上記のようにURLに明示的に指定されている場合設定よりも優先されます。 


.. note::

   ``limit`` コマンドを使用してAnimated GIFのフレーム数を調節することができます。 ::
      
      http://image.example.com/img.jpg/dims/limit/3
      http://image.example.com/img.jpg/dims/limit/3/resize/200x200



その他
====================================

以上のデフォルト的な機能を組み合わせて複合的なイメージ処理を行うことができます。 たとえばThumbnail生成（78x110）はフォーマットをJPGからPNGに変換すると品質の50％以上のオプションを一度の呼び出しで実行することができます。 ::

   http://image.example.com/img.jpg/dims/thumbnail/78x110/format/png/quality/50/

DIMSはURLを利用してイメージ加工が行われる。 したがってURLに影響を与える他のオプションのために望ましくない結果が得られないように注意する必要があります。

-  :ref:`caching-policy-applyquerystring` が ``OFF`` であればキーワード前のQueryStringが無視されます。 ::

      http://image.example.com/img.jpg?session=5234&type=37/dims/resize/200x200/

   上記のような呼び出しにこの設定が ``ON`` であれば入力されたURLのまま認識され ``OFF`` であれば次のように認識されます。 ::

      http://image.example.com/img.jpg/dims/resize/200x200/

-  :ref:`caching-policy-casesensitive` が ``OFF`` であればすべてのURLを小文字に変換して処理します。 したがってDIMSキーワードに大文字が含まれている場合キーワードを認識しません。 常にキーワードは小文字で使用するのがよい。
