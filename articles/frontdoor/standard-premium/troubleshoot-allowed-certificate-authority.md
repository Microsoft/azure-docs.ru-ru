---
title: Разрешенные центры сертификации для передней дверцы Azure уровня "Стандартный" или "Премиум" (Предварительная версия)
description: В этой статье перечислены все центры сертификации, разрешенные при создании собственного сертификата.
services: frontdoor
author: duongau
ms.service: frontdoor
ms.topic: troubleshooting
ms.date: 02/18/2021
ms.author: qixwang
ms.openlocfilehash: 35c1e4f6c700333c72b97289cda1772335037211
ms.sourcegitcommit: 97c48e630ec22edc12a0f8e4e592d1676323d7b0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/18/2021
ms.locfileid: "101100239"
---
# <a name="allowed-certificate-authorities-for-azure-front-door-standardpremium-preview"></a>Разрешенные центры сертификации для передней дверцы Azure уровня "Стандартный" или "Премиум" (Предварительная версия)

При включении функции HTTPS с помощью собственного сертификата для пользовательского домена уровня "Стандартный" или "Премиум" основной дверцы Azure. Для создания сертификата TLS/SSL требуется разрешенный центр сертификации (ЦС). В противном случае при использовании недопустимого ЦС или самозаверяющего сертификата ваш запрос будет отклонен.

При создании собственного сертификата разрешены следующие ЦС:

- AddTrust External CA Root
- Корневой ЦС AlphaSSL
- ЦС 01 AME Infra
- ЦС 02 AME Infra
- Ameroot
- APCA-DM3P
- Атос Трустедрут 2011
- Autopilot Root CA
- Baltimore CyberTrust Root
- Class 3 Public Primary Certification Authority
- COMODO RSA Certification Authority
- COMODO RSA Domain Validation Secure Server CA
- D-TRUST Root Class 3 CA 2 2009
- DigiCert Cloud Services CA-1
- DigiCert Global Root CA
- DigiCert Global Root G2
- DigiCert High Assurance CA-3
- DigiCert High Assurance EV Root CA
- ЦС расширенной проверки сервера DigiCert SHA2
- DigiCert SHA2 High Assurance Server CA
- DigiCert SHA2 Secure Server CA
- Корневой ЦС X3 DST
- Корневой ЦС 2 класса 3 D-trust 2009
- ЦС Encryption Everywhere DV TLS
- Корневой ЦС Entrust
- Корневой ЦС Entrust — G2
- ЦС Entrust.net (2048)
- Глобальный ЦС GeoTrust
- Основной ЦС GeoTrust
- Основной ЦС GeoTrust — G2
- ЦС Geotrust RSA 2018
- GlobalSign;
- GlobalSign Extended Validation CA - SHA256 - G2
- GlobalSign Organization Validation CA - G2
- GlobalSign Root CA
- Корневой ЦС Go Daddy — G2
- Защищенный ЦС Go Daddy — G2
- Давайте шифровать центр X3
- МС e-Сзигно root ЦС 2009
- QuoVadis Root CA2 G3
- ЦС RSA RapidSSL 2018
- RootCA1 Communication Security
- RootCA2 Communication Security
- RootCA3 Communication Security
- Symantec Class 3 EV SSL CA - G3
- Symantec Class 3 Secure Server CA - G4
- Symantec Enterprise Mobile Root for Microsoft
- Основной корневой ЦС Thawte
- Основной корневой ЦС Thawte — G2
- Основной корневой ЦС Thawte — G3
- ЦС Thawte RSA 2018
- Thawte Timestamping CA
- ЦС TrustAsia TLS RSA
- VeriSign Class 3 Extended Validation SSL CA
- VeriSign Class 3 Extended Validation SSL SGC CA
- VeriSign Class 3 Public Primary Certification Authority - G5
- VeriSign International Server CA - Class 3
- VeriSign Time Stamping Service Root
- VeriSign Universal Root Certification Authority
