---
title: Изменения TLS-сертификатов Azure
description: Изменения TLS-сертификатов Azure
services: security
author: msmbaldwin
tags: azure-resource-manager
ms.service: security
ms.subservice: security-fundamentals
ms.topic: article
ms.date: 11/10/2020
ms.author: mbaldwin
ms.openlocfilehash: 9e4edbcfd9e4180e727cd885902d9f0150a967ca
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100389405"
---
# <a name="azure-tls-certificate-changes"></a>Изменения TLS-сертификатов Azure  

Корпорация Майкрософт обновляет службы Azure для использования TLS-сертификатов от других корневых центров сертификации (ЦС). Это изменение вызвано тем, что текущие сертификаты ЦС не соответствуют базовым требованиям форума центра сертификации и браузера и будут отозваны 15 февраля 2021.

## <a name="when-will-this-change-happen"></a>Когда будет применено это изменение?

Существующие конечные точки Azure были переведены поэтапно с 13 августа 2020. Все новые конечные точки TLS/SSL Azure содержат обновленные сертификаты, связанные с новыми корневыми ЦС.

Это изменение влияет на все службы Azure. Ниже приведены некоторые дополнительные сведения о конкретных службах.

- Переход для служб [Azure Active Directory (Azure AD)](../../active-directory/index.yml) начался 7 июля 2020 г.
- [Центр Интернета вещей Azure](https://azure.microsoft.com/services/iot-hub) и [DPS](../../iot-dps/index.yml) будут связаны с корневым ЦС Baltimore CyberTrust, но их промежуточные ЦС изменятся. [Щелкните здесь, чтобы узнать больше.](https://techcommunity.microsoft.com/t5/internet-of-things/azure-iot-tls-changes-are-coming-and-why-you-should-care/ba-p/1658456)
- [Хранилище Azure](../../storage/index.yml) будет связано с корневым ЦС Baltimore CyberTrust, но его промежуточные ЦС изменятся. [Щелкните здесь, чтобы узнать больше.](https://techcommunity.microsoft.com/t5/azure-storage/azure-storage-tls-changes-are-coming-and-why-you-care/ba-p/1705518)
- [Кэш Azure для Redis](../../azure-cache-for-redis/index.yml) останется в КОРНЕВОМ центре сертификации Baltimore CyberTrust, но их промежуточные ЦС изменятся. [Щелкните здесь, чтобы узнать больше.](../../azure-cache-for-redis/cache-whats-new.md)
- Служба метаданных экземпляров Azure останется в корневом центре сертификации Baltimore CyberTrust, но их промежуточные ЦС изменятся. [Щелкните здесь, чтобы узнать больше.](/answers/questions/172717/action-required-for-attested-data-tls-with-azure-i.html)

> [!IMPORTANT]
> Возможно, клиентам потребуется обновить свои приложения после этого изменения, чтобы предотвратить сбои при попытке подключения к службам Azure.

## <a name="what-is-changing"></a>Что изменяется?

Сейчас большинство TLS-сертификатов, используемых службами Azure, связаны со следующими корневыми ЦС:

| Общее имя ЦС | Отпечаток (SHA1) |
|--|--|
| [Baltimore CyberTrust Root](https://cacerts.digicert.com/BaltimoreCyberTrustRoot.crt) | d4de20d05e66fc53fe1a50882c78db2852cae474 |

TLS-сертификаты, используемые службами Azure, будут связаны с одним из следующих корневых ЦС:

| Общее имя ЦС | Отпечаток (SHA1) |
|--|--|
| [DigiCert Global Root G2](https://cacerts.digicert.com/DigiCertGlobalRootG2.crt) | df3c24f9bfd666761b268073fe06d1cc8d4f82a4 |
| [DigiCert Global Root CA](https://cacerts.digicert.com/DigiCertGlobalRootCA.crt) | a8985d3a65e5e5c4b2d7d66d40c6dd2fb19c5436 |
| [Baltimore CyberTrust Root](https://cacerts.digicert.com/BaltimoreCyberTrustRoot.crt) | d4de20d05e66fc53fe1a50882c78db2852cae474 |
| [D-TRUST Root Class 3 CA 2 2009](https://www.d-trust.net/cgi-bin/D-TRUST_Root_Class_3_CA_2_2009.crt) | 58e8abb0361533fb80f79b1b6d29d3ff8d5f00f0 |
| [Microsoft RSA Root Certificate Authority 2017](https://www.microsoft.com/pkiops/certs/Microsoft%20RSA%20Root%20Certificate%20Authority%202017.crt) | 73a5e64a3bff8316ff0edccc618a906e4eae4d74 | 
| [Microsoft ECC Root Certificate Authority 2017](https://www.microsoft.com/pkiops/certs/Microsoft%20ECC%20Root%20Certificate%20Authority%202017.crt) | 999a64c37ff47d9fab95f14769891460eec4c3c5 |

## <a name="when-can-i-retire-the-old-intermediate-thumbprint"></a>Когда можно будет прекратить использование старого промежуточного отпечатка?

Текущие сертификаты ЦС *не* будут отозваны до 15 февраля 2021 г. После этой даты вы сможете удалить старые отпечатки из своего кода.

При изменении этой даты вы будете уведомлены о новой дате отзыва.

## <a name="will-this-change-affect-me"></a>Повлияет ли это изменение на работу? 

Мы ожидаем, что это изменение не повлияет на работу **большинства клиентов Azure**.  Но оно может сказаться на вашем приложении, если в нем явно указан список допустимых ЦС. Такой подход называется привязкой сертификатов.

Чтобы определить, повлияет ли изменение на ваше приложение, воспользуйтесь следующими методами:

- Выполните в исходном коде поиск отпечатка, общего имени и других свойств сертификатов для всех центров сертификации Microsoft IT TLS CA, список которых можно найти [здесь](https://www.microsoft.com/pki/mscorp/cps/default.htm). Совпадение будет означать, что изменение затронет ваше приложение. Чтобы решить эту проблему, обновите исходный код, включив данные о новых ЦС. Рекомендуем обеспечить возможность быстрого добавления или изменения данных о ЦС. Отраслевые нормы требуют замены сертификатов ЦС в течение семи дней, поэтому клиентам, которые пользуются привязкой сертификатов, необходимо сделать это как можно скорее.

- Если ваше приложение интегрируется с API-интерфейсами Azure или другими службами Azure и вы не уверены, использует ли оно привязку сертификатов, обратитесь к поставщику приложения.

- Для разных операционных систем и сред выполнения на разных языках, которые обмениваются данными со службами Azure, возможно, потребуется выполнить дополнительные действия, чтобы правильно построить цепочку сертификатов с такими новыми корнями:
    - **Linux**: при работе с многими дистрибутивами требуется добавить центры сертификации в каталог /etc/ssl/certs. Дополнительные инструкции см. в документации по конкретному дистрибутиву.
    - **Java**. убедитесь, что хранилище ключей Java содержит данные о приведенных выше центах сертификации.
    - **Системы Windows, работающие в отключенных средах:** для таких систем необходимо добавить данные о новых корнях в хранилище доверенных корневых центров сертификации, а данные о промежуточных ЦС — в хранилище промежуточных центров сертификации.
    - **Android:** см. документацию по вашему устройству и версии Android.
    - **Другие аппаратные устройства, особенно Интернета вещей:** обратитесь к производителю устройства.

- Если в вашей среде заданы правила брандмауэра для разрешения исходящих вызовов только на скачивание определенного списка отзыва сертификатов (CRL) и (или) к определенным расположениям проверки OCSP, вам потребуется разрешить следующие URL-адреса для CRL и OCSP:

    - http://crl3&#46;digicert&#46;com;
    - http://crl4&#46;digicert&#46;com;
    - http://ocsp&#46;digicert&#46;com;
    - http://www&#46;d-trust&#46;net;
    - http://root-c3-ca2-2009&#46;ocsp&#46;d-trust&#46;net;
    - http://crl&#46;microsoft&#46;com;
    - http://oneocsp&#46;microsoft&#46;com;
    - http://ocsp&#46;msocsp&#46;com.
    - http://www&#46; Microsoft&#46;com/пкиопс

## <a name="next-steps"></a>Дальнейшие действия

Если у вас есть дополнительные вопросы, свяжитесь с нами через [службу поддержки](https://azure.microsoft.com/support/options/).
