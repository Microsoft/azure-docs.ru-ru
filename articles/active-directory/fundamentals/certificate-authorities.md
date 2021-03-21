---
title: Центры сертификации Azure Active Directory
description: Список доверенных сертификатов, используемых в Azure
services: active-directory
author: BarbaraSelden
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: conceptual
ms.date: 10/10/2020
ms.author: baselden
ms.reviewer: baselden
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: bea6f4161bcca063cd2c58d4c463473426f159ba
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96859137"
---
# <a name="certificate-authorities-used-by-azure-active-directory"></a>Центры сертификации, используемые Azure Active Directory

> [!IMPORTANT]
> Сведения на этой странице относятся только к сущностям, которые явно указывают список приемлемых центров сертификации (CAs). Такой подход, называемый закреплением сертификатов, следует избегать, если нет других параметров.

Любая сущность, пытающаяся получить доступ к службам удостоверений Azure Active Directory (Azure AD) через протоколы TLS/SSL, будет представлена сертификатами из перечисленных ниже ЦС. Если сущность доверяет этим ЦС, она может использовать сертификаты для проверки удостоверения и позаконного получения служб удостоверений и установления безопасных соединений.

Центры сертификации можно классифицировать в корневые центры сертификации и промежуточные ЦС. Как правило, корневые ЦС имеют один или несколько связанных промежуточных ЦС. В этой статье перечислены корневые центры сертификации, используемые службами удостоверений Azure AD, и промежуточные ЦС, связанные с каждым из этих корней. Для каждого ЦС мы включаем универсальные идентификаторы ресурсов (URI) для скачивания соответствующего доступа к сведениям о центре сертификации (AIA) и файлов CDP для списка отзыва сертификатов. При необходимости мы также предоставляем URI конечной точке Online Certificate Status Protocol (OCSP).

## <a name="cas-used-in-azure-public-and-azure-us-government-clouds"></a>CAs, используемый в общедоступных облачных облаках Azure и Azure для государственных организаций США

Разные службы могут использовать разные корневые или промежуточные ЦС. Поэтому могут потребоваться все перечисленные ниже записи.

### <a name="digicert-global-root-g2"></a>DigiCert Global Root G2


| Корневой ЦС| Серийный номер| Дата истечения срока действия| Отпечаток SHA1| Идентификаторы URI |
| - |- |-|-|-|-|
| DigiCert Global Root G2| 033af1e6a711a 9a0bb2864b11d09fae5| 1 августа 2013 г. <br>15 января 2038 г.| df3c24f9bfd666761b268 073fe06d1cc8d4f82a4| [AIA](http://cacerts.digicert.com/DigiCertGlobalRootG2.crt)<br>[CDP](http://crl3.digicert.com/DigiCertGlobalRootG2.crl) |


#### <a name="associated-intermediate-cas"></a>Связанные промежуточные ЦС

| Выдающий и промежуточный ЦС| Серийный номер| Дата истечения срока действия| Отпечаток SHA1| Идентификаторы URI |
| - | - | - | - | - | 
| Выдающий ЦС 01 Microsoft Azure TLS| 0aafa6c5ca63c45141 ea3be1f7c75317| 29 июля 2020 г.<br>27 июня 2024 г.| 2f2877c5d778c31e0f29c 7e371df5471bd673173| [AIA](https://www.microsoft.com/pkiops/certs/Microsoft%20Azure%20TLS%20Issuing%20CA%2001%20-%20xsign.crt)<br>[CDP](https://www.microsoft.com/pkiops/crl/Microsoft%20Azure%20TLS%20Issuing%20CA%2001.crl)|
|Выдающий центр сертификации Microsoft Azure TLS 02| 0c6ae97cced59983 8690a00a9ea53214| 29 июля 2020 г.<br>27 июня 2024 г.| e7eea674ca718e3befd 90858e09f8372ad0ae2aa| [AIA](https://www.microsoft.com/pkiops/certs/Microsoft%20Azure%20TLS%20Issuing%20CA%2002%20-%20xsign.crt)<br>[CDP](https://www.microsoft.com/pkiops/crl/Microsoft%20Azure%20TLS%20Issuing%20CA%2002.crl) |
| Выдающий ЦС 05 Microsoft Azure TLS| 0d7bede97d8209967a 52631b8bdd18bd| 29 июля 2020 г.<br>27 июня 2024 г.| 6c3af02e7f269aa73a fd0eff2a88a4a1f04ed1e5| [AIA](https://www.microsoft.com/pkiops/certs/Microsoft%20Azure%20TLS%20Issuing%20CA%2005%20-%20xsign.crt)<br>[CDP](https://www.microsoft.com/pkiops/crl/Microsoft%20Azure%20TLS%20Issuing%20CA%2005.crl) |
| Выдающий ЦС 06 Microsoft Azure TLS| 02e79171fb8021e93fe 2d983834c50c0| 29 июля 2020 г.<br>27 июня 2024 г.| 30e01761ab97e59a06b 41ef20af6f2de7ef4f7b0| [AIA](https://www.microsoft.com/pkiops/certs/Microsoft%20Azure%20TLS%20Issuing%20CA%2006.cer)<br>[CDP](https://www.microsoft.com/pkiops/crl/Microsoft%20Azure%20TLS%20Issuing%20CA%2006.crl) |


 ### <a name="baltimore-cybertrust-root"></a>Baltimore CyberTrust Root

| Корневой ЦС| Серийный номер| Дата истечения срока действия| Отпечаток SHA1| Идентификаторы URI |
| - | - | - | - | - | 
| Baltimore CyberTrust Root| 020000b9| 12 мая, 2000<br>12 мая, 2025| d4de20d05e66fc53fe 1a50882c78db2852cae474|<br>[CDP](http://crl3.digicert.com/Omniroot2025.crl)<br>[ВЫПОЛНЕН](http://ocsp.digicert.com/) |


#### <a name="associated-intermediate-cas"></a>Связанные промежуточные ЦС

| Выдающий и промежуточный ЦС| Серийный номер| Дата истечения срока действия| Отпечаток SHA1| Идентификаторы URI |
| - | - | - | - | - | 
| ЦС Microsoft RSA TLS 01| 703d7a8f0ebf55aaa 59f98eaf4a206004eb2516a| 21 июля 2020 г.<br>8 октября 2024 г.| 417e225037fbfaa4f9 5761d5ae729e1aea7e3a42| [AIA](https://www.microsoft.com/pki/mscorp/Microsoft%20RSA%20TLS%20CA%2001.crt)<br>[CDP](https://mscrl.microsoft.com/pki/mscorp/crl/Microsoft%20RSA%20TLS%20CA%2001.crl)<br>[ВЫПОЛНЕН](http://ocsp.msocsp.com/) |
| ЦС Microsoft RSA TLS 02| b0c2d2d13cdd56cdaa 6ab6e2c04440be4a429c75| 21 июля 2020 г.<br>20 мая 2024 г.| 54d9d20239080c32316ed 9ff980a48988f4adf2d| [AIA](https://www.microsoft.com/pki/mscorp/Microsoft%20RSA%20TLS%20CA%2002.crt)<br>[CDP](https://mscrl.microsoft.com/pki/mscorp/crl/Microsoft%20RSA%20TLS%20CA%2002.crl)<br>[ВЫПОЛНЕН](http://ocsp.msocsp.com/) |


 ### <a name="digicert-global-root-ca"></a>DigiCert Global Root CA

| Корневой ЦС| Серийный номер| Дата истечения срока действия| Отпечаток SHA1| Идентификаторы URI |
| - | - | - | - | - | 
| DigiCert Global Root CA| 083be056904246 b1a1756ac95991c74a| 9 ноября 2006 г.<br>9 ноября 2031 г.| a8985d3a65e5e5c4b2d7 d66d40c6dd2fb19c5436| [CDP](http://crl3.digicert.com/DigiCertGlobalRootCA.crl)<br>[ВЫПОЛНЕН](http://ocsp.digicert.com/) |


#### <a name="associated-intermediate-cas"></a>Связанные промежуточные ЦС

| Выдающий и промежуточный ЦС| Серийный номер| Дата истечения срока действия| Отпечаток SHA1| Идентификаторы URI |
| - | - | - | - | - |
| DigiCert SHA2 Secure Server CA| 01fda3eb6eca75c 888438b724bcfbc91| 8 марта 2013 г., 8 марта 2023| 1fb86b1168ec743154062 e8c9cc5b171a4b7ccb4| [AIA](http://cacerts.digicert.com/DigiCertSHA2SecureServerCA.crt)<br>[CDP](http://crl3.digicert.com/ssca-sha2-g6.crl)<br>[ВЫПОЛНЕН](http://ocsp.digicert.com/) |
| DigiCert SHA2 Secure Server CA |02742eaa17ca8e21 c717bb1ffcfd0ca0 |22 сентября 2020 г.<br>22 сентября 2030 г.|626d44e704d1ceabe3bf 0d53397464ac8080142c|[AIA](http://cacerts.digicert.com/DigiCertSHA2SecureServerCA-2.crt)<br>[CDP](http://crl3.digicert.com/DigiCertSHA2SecureServerCA.crl)<br>[ВЫПОЛНЕН](http://ocsp.digicert.com/)|


## <a name="cas-used-in-azure-china-21vianet-cloud"></a>CAs, используемый в облаке Azure для Китая (21Vianet)

### <a name="digicert-global-root-ca"></a>DigiCert Global Root CA


| Корневой ЦС| Серийный номер| Дата истечения срока действия| Отпечаток SHA1| Идентификаторы URI |
| - | - | - | - | - |
| DigiCert Global Root CA| 083be056904246b 1a1756ac95991c74a| 9 ноября, 2006<br>9 ноября, 2031| a8985d3a65e5e5c4b2d7 d66d40c6dd2fb19c5436| [CDP](http://ocsp.digicert.com/)<br>[ВЫПОЛНЕН](http://crl3.digicert.com/DigiCertGlobalRootCA.crl) |


#### <a name="associated-intermediate-ca"></a>Связанный промежуточный ЦС

| Выдающий и промежуточный ЦС| Серийный номер| Дата истечения срока действия| Отпечаток SHA1| Идентификаторы URI |
| - | - | - | - | - | - |
| DigiCert Basic RSA CN CA, G2| 02f7e1f982bad 009aff47dc95741b2f6| 4 марта 2020 г.<br>4 марта 2030 г.| 4d1fa5d1fb1ac3917c08e 43f65015e6aea571179| [AIA](http://cacerts.digicert.cn/DigiCertBasicRSACNCAG2.crt)<br>[CDP](http://crl.digicert.cn/DigiCertBasicRSACNCAG2.crl)<br>[ВЫПОЛНЕН](http://ocsp.digicert.cn/) |

## <a name="next-steps"></a>Дальнейшие действия
[Дополнительные сведения о цепочках шифрования Microsoft 365](/microsoft-365/compliance/encryption-office-365-certificate-chains)
