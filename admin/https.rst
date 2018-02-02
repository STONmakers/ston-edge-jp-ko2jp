.. _https:

第9章 HTTPS
******************

この章では、HTTPSの構成について説明する。 TLS 1.2をサポートし、SSL 2.0は、セキュリティ上の理由から、アップグレードのみを許可する。 HTTPSは、クライアントとSTON区間でのみ使用される。 STONは、元のサーバーとHTTPSで通信していない。 なぜなら、セキュリティ的にも性能的にSTONがHTTPSを中継することは適切でないからである。 もし元のサーバーと、必ずHTTPSで通信が必要な場合 :ref:`bypass-port` を推奨する。


.. toctree::
   :maxdepth: 2



サービスの構成
====================================

別のIPアドレスまたはポートを指定しない場合、デフォルトでバインドされているサービスアドレスは、 "*:443"である。
グローバル設定（server.xml）に設定する。 ::

   # server.xml - <Server>

   <Https>
      <Cert>/usr/ssl/cert.pem</Cert>
      <Key>/usr/ssl/certkey.pem</Key>
      <CA>/usr/ssl/CA.pem</CA>
   </Https>

   <Https Listen="1.1.1.1:443">
      <Cert>/usr/ssl_ip_port/cert.pem</Cert>
      <Key>/usr/ssl_ip_port/certkey.pem</Key>
      <CA>/usr/ssl_ip_port/CA.pem</CA>
   </Https>

   <Https Listen="*:886">
      <Cert>/usr/ssl_port886/cert.pem</Cert>
      <Key>/usr/ssl_port886/certkey.pem</Key>
      <CA>/usr/ssl_port886/CA.pem</CA>
   </Https>

-  ``<Https>`` HTTPSを構成する。

   -  ``<Cert>`` サーバー証明書

   -  ``<Key>`` サーバ証明書の秘密鍵。 暗号化された形式はサポートしていない。

   -  ``<CA>`` CA(Certificate Authority) チェーンの証明書

同じPortをサービスしても、より明確な表現が優先する。

たとえば、上記のNICが3個で、それぞれのIPアドレスが1.1.1.1、1.1.1.2、1.1.1.3である場合を想定してみよう。 1.1.1.1:443に接続したクライアントは、最も明示的な表現である2番目（<Https Listen = "1.1.1.1:443">）証明書にサービスされる。 一方、1.1.1.3:443に接続したクライアントは、IPが一致する1番目（<Https>、Listen = " * ：443"属性が省略されている）証明書にサービスされる。 証明書ファイルを同じ名前で上書きしてもReloadするときに反映される。

.. note::

   証明書のフォーマットは、PEM（Privacy Enhanced Mail）、非対称キーアルゴリズムは、RSAのみをサポートする。


.. _https-aes-ni:

SSL / TLS加速
====================================

CPU（AES-NI）を介してSSL / TLSを加速する。 AES-NIをサポートしているCPUである場合、SSL / TLSでAESアルゴリズムを優先的に使用するように動作する。 AES-NIが認識された場合は、次のようにInfo.logに記録される。 ::

   AES-NI : ON (SSL/TLS accelerated)

管理者は、AES-NIを使用するかどうかを選択することができる。 ::

   # server.xml - <Server><Cache>

   <AES-NI>ON</AES-NI>

-  ``<AES-NI> (基本: ON)`` AES-NIを使用するかどうかを選択する。



.. _https-ciphersuite:

CipherSuiteを選択
====================================

サポートしているCipherSuitesは以下の通りである。

================================================ ======== =========== =======
Cipher Suite                                     TLS1.2   TLS1.1/1.0  SSL3.0
================================================ ======== =========== =======
TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384	(0xc030)   O
TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384	(0xc028)   O
TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256	(0xc02F)   O
TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256	(0xC027)   O
TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA (0xC014)      O        O
TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA (0xC013)      O        O
TLS_RSA_WITH_AES_256_GCM_SHA384	(0x009D)         O
TLS_RSA_WITH_AES_128_GCM_SHA256	(0x009C)         O
TLS_RSA_WITH_AES_256_CBC_SHA256	(0x003D)         O
TLS_RSA_WITH_AES_128_CBC_SHA256	(0x003C)         O
TLS_RSA_WITH_AES_256_CBC_SHA (0x0035)            O        O
TLS_RSA_WITH_AES_128_CBC_SHA (0x002F)            O        O
TLS_RSA_WITH_3DES_EDE_CBC_SHA (0x000A)           O        O
TLS_RSA_WITH_RC4_128_SHA (0x0005)                                     O
TLS_RSA_WITH_RC4_128_MD5 (0x0004)                                     O
================================================ ======== =========== =======

``<Https>`` の ``CipherSuite`` 属性を使用すると、使用CipherSuiteを設定することができる。 ::

   # server.xml - <Server>

   <Https CipherSuite="ALL:!ADH:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP">
      <Cert>/usr/ssl/cert.pem</Cert>
      <Key>/usr/ssl/certkey.pem</Key>
      <CA>/usr/ssl/CA.pem</CA>
   </Https>

-  ``CipherSuite`` `Apache mod_sslの SSL CipherSuite表現 <http://httpd.apache.org/docs/2.2/mod/mod_ssl.html#sslciphersuite>`_ に続く。

`Forward Secrecy <https://en.wikipedia.org/wiki/Forward_secrecy>`_ を保証する、より高い安全性を得ることができる。 （下のリンク参照）

   - `SSL Labs: Deploying Forward Secrecy <https://community.qualys.com/blogs/securitylabs/2013/06/25/ssl-labs-deploying-forward-secrecy>`_
   - `SSL/TLS & Perfect Forward Secrecy <http://vincent.bernat.im/en/blog/2011-ssl-perfect-forward-secrecy.html>`_
   - `Configuring Apache, Nginx, and OpenSSL for Forward Secrecy <https://community.qualys.com/blogs/securitylabs/2013/08/05/configuring-apache-nginx-and-openssl-for-forward-secrecy>`_

基本的には FS（Forward Secrecy）を保証するCipherSuiteを優先的に選択する。 ::

   # server.xml - <Server>

   <Https FS="ON"> ...  </Https>

-  ``FS``

   - ``ON (基本)`` Forward Secrecyを保証するCipherSuiteを優先的に選択する。
   - ``OFF`` ClientHelloに記載順に選択する。

``FS`` 属性は ``CipherSuite`` 属性よりも優先します。

.. note::

   パフォーマンス上の理由から、ECDHEのみをサポートする。 DHEは対応していない。



.. _https-ciphersuite-query:

CipherSuite照会
====================================

CipherSuite設定結果を照会する。 CipherSuite式は `OpenSSL 1.0.0E <https://www.openssl.org/docs/apps/ciphers.html>`_ を遵守する。 ::

   http://127.0.0.1:10040/monitoring/ssl?ciphersuite=...

結果は、JSON形式で提供される。 ::

  {
      "version": "2.0.0",
      "method": "ssl",
      "status": "OK",
      "result":
      [
          {
              "Name" : "AES128-SHA",
              "Ver" : "SSLv3",
              "Kx" : "RSA",
              "Au" : "RSA",
              "Enc" : "AES(128)",
              "Mac" : "SHA1"
          },
          {
              "Name" : "AES256-SHA",
              "Ver" : "SSLv3",
              "Kx" : "RSA",
              "Au" : "RSA",
              "Enc" : "AES(256)",
              "Mac" : "SHA1"
          }
      ]
  }



マルチDomain構成
====================================

一台のサーバーで複数のサービスを同時に運用する場合は、SSLの設定が問題になることができる。 ほとんどのWeb / Cacheサーバは、HTTPリクエストのHostヘッダを見ていくつかの仮想ホストでサービスするかどうかを決定する。

.. figure:: img/ssl_alert.png
   :align: center

   一般HTTPS通信

一般的に、SSLは、クライアント（Browser）が、自分が接続しようとするサーバーのドメイン名（winesoft.co.kr）を証明書を使用して確認すること身元確認をする。 もし証明書で身元確認がされていない場合（無効な証明書または有効期間満了など）は、次のようにユーザーに信頼するかどうかを尋ねる（最初から遮断する場合もある）。 信頼は、クライアントがするので、通常の身元確認にならなくても続行したい場合はSSL通信が行われる。

.. figure:: img/faq_ssl1.jpg
   :align: center

   ユーザーに判断を任せる。

サーバーでSSLを使用する仮想ホストが一つであれば問題にならない。 しかし、複数の仮想ホストを同時に運営するサーバーでは、問題になることができる。 なぜなら、サーバーがクライアントに証明書を転送するときに（"一般HTTPS通信"の "2.証明書伝達")
クライアントがどのようなHostに接続しようとして知ることができないからである。

この問題を克服する代表的な方法は次のとおりである。

=================== ====================================== ========================================================================
방식	            장점	                               단점
=================== ====================================== ========================================================================
SNI                 서버설정만으로 동작 (표준)	           Windows XP와 IE6 미지원
Multi Certificate	인증서만 교체하여 동작	               메인 도메인 또는 서비스 주체가 같아야 하며 자칫 재발급이 빈번할 수 있음
Multi Port          포트만 변경하여 동작	               웹 페이지에서 HTTPS포트를 명시해주어야 함
Multi NIC	        서버설정만으로 동작 (가장 널리쓰임)    NIC와 IP추가 구성필요
=================== ====================================== ========================================================================



.. _https_sni:

SNI (Server Name Indication)
--------------------------

SSL / TLSの `SNI(Server Name Indication) <http://en.wikipedia.org/wiki/Server_Name_Indication>`_
拡張フィールドを使用する方式である。 この方式は、クライアントがサーバーにSSL接続を要求するときServer Name拡張フィールドを明示することで可能である。 ::

   # server.xml - <Server><Cache>

   <HttpsSNI>OFF</HttpsSNI>

-  ``<HttpsSNI>``

   - ``OFF (基本)`` `Multi Port`_ または `Multi NIC`_ 方法で複数の証明書をサポートします。

   - ``ON`` のようなIP + Portの組み合わせで複数の証明書をサポートする。 以下の場合のように、ポート443で複数の証明書をサポートすることができる。 ::

      # server.xml - <Server>

      <Https>
         <Cert>/usr/ssl/cert.pem</Cert>
         <Key>/usr/ssl/certkey.pem</Key>
         <CA>/usr/ssl/CA.pem</CA>
      </Https>

      <Https>
         <Cert>/usr/ssl_another/cert.pem</Cert>
         <Key>/usr/ssl_another/certkey.pem</Key>
         <CA>/usr/ssl_another/CA.pem</CA>
      </Https>


``<HttpsSNI>`` は、動的に変更が不可能である。 設定の変更後は、必ずサービスを再起動しなければならない。

.. note::

   SNIは2003年6月 `RFC 3546 <https://tools.ietf.org/html/rfc3546#page-8>`_ を介してTLS 1.0以降でのみ定義された。 したがってSSL v3ではSNIをサポートしていません。 参考までにOpenSSLのs_clientにSSL-3.0オプションを適用すると、SNI拡張フィールドを送信しません。

現在までで最もエレガントな方法ですが、いくつかの古いクライアントでサポートされていない。 以下は、SNIをサポートしていないクライアントのリストである。 （出典: `Wikipedia - Server Name Indication <http://en.wikipedia.org/wiki/Server_Name_Indication#Client_side>`_ )

- Internet Explorer (any version) on Windows XP or Internet Explorer 6 or earlier
- Safari on Windows XP
- BlackBerry Browser
- Windows Mobile up to 6.5
- Android default browser on Android 2.x[34] (Fixed in Honeycomb for tablets and Ice Cream Sandwich for phones)
- wget before 1.14
- Java before 1.7




Multi Certificate
--------------------------

証明書に複数のドメインを入れたりWildcard(i.e. *.winesoft.co.kr)を明示して1つの証明書で複数のドメインの身元を確認することができる方法である。

.. figure:: img/faq_ssl2.jpg
   :align: center

   一つの証明書で複数Domainを認証する。

サービス主体が同じなら、効果的な方法ですが、無関係であれば、同じ証明書を共有することは現実的に困難である。 この方法は、証明書のみを交換すればれるものなのでSTONで個別に設定することはない。
[ `DigiCert <http://www.digicert.com/wildcard-ssl-certificates.htm>`_ 参考].



Multi Port
--------------------------

SSL / TLSはポート443を使用する。 重複していないポートを利用して、証明書を複数インストールすることができる。 クライアントでは、次のようにポートを明示することにより、SSL通信が可能である。 ::

    https://winesoft.co.kr:543/

STONでは、次のようにListen属性にポートを明示して、証明書を複数に設定する。 ::

   # server.xml - <Server>

   <Https> ..A社 の証明書.. </Https>
   <Https Listen="*:543"> ..B社 の証明書.. </Https>
   <Https Listen="*:544"> ..C社 の証明書.. </Https>

この方法は、最も経済的ではあるが、すべてのWebページへのリンクにHTTPSポートを指定しなければなら問題がある。


Multi NIC
--------------------------

サーバーのNICが複数で構成されている場合NICごとにIPアドレスを個別に割り当てることができる。 したがって、サーバーのIPごとに個別の証明書をインストールして、クライアントが接続したサーバーのIPに基づいて証明書を決定するように設定する。 STONでは、次のようにListen属性にIPアドレスを明示して、証明書を複数に設定する。. ::

   # server.xml - <Server>

   <Https Listen="10.10.10.10"> ..A社 の証明書.. </Https>
   <Https Listen="10.10.10.11"> ..B社 の証明書.. </Https>
   <Https Listen="10.10.10.12"> ..C社 の証明書.. </Https>

この方法は、最も一般的に使用される方式である。

.. note::

   設定を共有すると、IPアドレスのために問題になることができる。 このような場合は、IPの代わりにNICの名前に設定する。 ::

      # server.xml - <Server>

      <Https Listen="eth0"> ... </Https>
      <Https Listen="eth1"> ... </Https>
      <Https Listen="eth2"> ... </Https>



プロトコルの設定
====================================

``<Https>`` にプロトコルを構成する。 ::

   # server.xml - <Server>

   <Https TLS1.2="ON" TLS1.1="ON" TLS1.0="ON" SSL3.0="ON"> ...  </Https>

- ``TLS1.2 (基本: ON)`` TLS1.2を使用する。

- ``TLS1.1 (基本: ON)`` TLS1.1を使用する。

- ``TLS1.0 (基本: ON)`` TLS1.0を使用する。

- ``SSL3.0 (基本: ON)`` SSL3.0を使用する。


.. _https-hsts:

HSTS
====================================

`HSTS(HTTP Strict Transport Security) <https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security>`_ は、
:ref:`handling_http_requests_modify_client` を利用して簡単に実装が可能である。
::

   # /svc/www.example.com/headers.txt

   *, $RES[Strict-Transport-Security: max-age=31536000; includeSubDomains], set

`Qualys SSL Server Test <https://www.ssllabs.com/ssltest/>`_ ではHSTSが適用されたサイトにのみA +の評価を与えている。

.. figure:: img/qualys_a_plus.png
   :align: center

   STON v2.2からA +を受けることができる
