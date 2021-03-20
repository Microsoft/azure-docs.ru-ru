---
title: Новые возможности кэша Azure для Redis
description: Последние обновления для кэша Azure для Redis
author: yegu-ms
ms.service: cache
ms.topic: reference
ms.date: 09/28/2020
ms.author: yegu
ms.openlocfilehash: b30e83b89b25e6400b8c7e0419406631fa1edcd0
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91492540"
---
# <a name="whats-new-in-azure-cache-for-redis"></a>Новые возможности кэша Azure для Redis

## <a name="azure-tls-certificate-change"></a>Изменение сертификата TLS Azure

Корпорация Майкрософт обновляет службы Azure, чтобы использовать сертификаты сервера TLS из другого набора центров сертификации (ЦС). Это изменение выполняется на этапах с 13 августа 2020 до 26 октября 2020 (приблизительно). Azure вносит это изменение, так как [текущие сертификаты ЦС не соответствуют базовым требованиям форума центра сертификации и браузера](https://bugzilla.mozilla.org/show_bug.cgi?id=1649951). Эта проблема была зарегистрирована 1 июля 2020 и относится к нескольким популярным поставщикам инфраструктуры открытых ключей (PKI) по всему миру. В настоящее время большинство сертификатов TLS, используемых службами Azure, входят в *Baltimore CyberTrust root* PKI. Кэш Azure для службы Redis будет по прежнему связан с корневым каталогом CyberTrust Baltimore. Однако сертификаты сервера TLS будут выдаваться новыми промежуточными центрами сертификации (ИКАС), начиная с 12 октября 2020 г.

> [!NOTE]
> Это изменение ограничено службами в общедоступных [регионах Azure](https://azure.microsoft.com/global-infrastructure/geographies/). Он исключает независимых (например, Китая) или государственные облака.
>
>

### <a name="does-this-change-affect-me"></a>Повлияет ли это изменение на работу?

Мы планируем, что это изменение не повлияет на большую часть кэша Azure для клиентов Redis. Приложение может затронуть, если явно указан список допустимых сертификатов, так называемый «закрепление сертификата». Если он закреплен на промежуточном или конечном сертификате вместо корневого Baltimore CyberTrust, следует **предпринять немедленные действия** для изменения конфигурации сертификата.

В следующей таблице приведены сведения о вызываемых сертификатах. В зависимости от того, какой сертификат используется приложением, может потребоваться обновить его, чтобы предотвратить потери подключения к кэшу Azure для экземпляра Redis.

| Тип ЦС | Текущий | После отката (12 октября, 2020) | Действие |
| ----- | ----- | ----- | ----- |
| Root | Отпечаток: d4de20d05e66fc53fe1a50882c78db2852cae474<br><br> Срок действия: понедельник, 12 мая, 2025, 4:59:00 PM<br><br> Имя субъекта:<br> CN = Baltimore CyberTrust root<br> OU = CyberTrust<br> O = Baltimore<br> C = IE | Не изменяется | Нет |
| Промежуточный | Отпечатки<br> CN = ЦС Microsoft IT TLS 1<br> Отпечаток: 417e225037fbfaa4f95761d5ae729e1aea7e3a42<br><br> CN = Microsoft IT TLS ЦС 2<br> Отпечаток: 54d9d20239080c32316ed9ff980a48988f4adf2d<br><br> CN = ЦС Microsoft IT TLS 4<br> Отпечаток: 8a38755d0996823fe8fa3116a277ce446eac4e99<br><br> CN = Microsoft IT TLS ЦС 5<br> Отпечаток: Ad898ac73df333eb60ac1f5fc6c4b2219ddb79b7<br><br> Срок действия: Пятница, 20 мая, 2024 5:52:38 AM<br><br> Имя субъекта:<br> OU — Microsoft IT<br> O = Корпорация Майкрософт<br> L = Redmond<br> S = Вашингтон<br> C = US<br> | Отпечатки<br> CN = Microsoft RSA TLS CA 01<br> Отпечаток: 703d7a8f0ebf55aaa59f98eaf4a206004eb2516a<br><br> CN = Microsoft RSA TLS ЦС 02<br> Отпечаток: b0c2d2d13cdd56cdaa6ab6e2c04440be4a429c75<br><br> Срок действия: вторник, 8 октября 2024 12:00:00 AM;<br><br> Имя субъекта:<br> O = Корпорация Майкрософт<br> C = US<br> | Обязательно |

### <a name="what-actions-should-i-take"></a>Какие действия следует предпринять?

Если приложение использует хранилище сертификатов операционной системы или закрепляет корневой элемент Baltimore в других случаях, никаких действий не требуется. С другой стороны, если ваше приложение закрепляет любой промежуточный или конечный сертификат TLS, рекомендуется закрепить следующие корни:

| Certificate | Отпечаток |
| ----- | ----- |
| [Корневой ЦС Baltimore](https://cacerts.digicert.com/BaltimoreCyberTrustRoot.crt) | d4de20d05e66fc53fe1a50882c78db2852cae474 |
| [Microsoft RSA Root Certificate Authority 2017](https://www.microsoft.com/pkiops/certs/Microsoft%20RSA%20Root%20Certificate%20Authority%202017.crt) | 73a5e64a3bff8316ff0edccc618a906e4eae4d74 |
| [DigiCert Global Root, G2](https://cacerts.digicert.com/DigiCertGlobalRootG2.crt) | df3c24f9bfd666761b268073fe06d1cc8d4f82a4 |

> [!TIP]
> Как промежуточные, так и конечные сертификаты должны изменяться часто. Не рекомендуется зависеть от них. Вместо этого следует закреплять приложение на корневом сертификате, так как он выполняется менее часто.
>
>

Чтобы продолжить закрепление промежуточных сертификатов, добавьте следующий код в список закрепленных промежуточных сертификатов, который включает в себя несколько дополнительных, чтобы избежать будущих изменений:

| Общее имя ЦС | Отпечаток |
| ----- | ----- |
| [ЦС Microsoft RSA TLS 01](https://www.microsoft.com/pki/mscorp/Microsoft%20RSA%20TLS%20CA%2001.crt) | 703d7a8f0ebf55aaa59f98eaf4a206004eb2516a |
| [ЦС Microsoft RSA TLS 02](https://www.microsoft.com/pki/mscorp/Microsoft%20RSA%20TLS%20CA%2002.crt) | b0c2d2d13cdd56cdaa6ab6e2c04440be4a429c75 |
| [Выдающий ЦС 01 Microsoft Azure TLS](https://www.microsoft.com/pkiops/certs/Microsoft%20Azure%20TLS%20Issuing%20CA%2001.cer) | 2f2877c5d778c31e0f29c7e371df5471bd673173 |
| [Выдающий центр сертификации Microsoft Azure TLS 02](https://www.microsoft.com/pkiops/certs/Microsoft%20Azure%20TLS%20Issuing%20CA%2002.cer) | e7eea674ca718e3befd90858e09f8372ad0ae2aa |
| [Выдающий ЦС 05 Microsoft Azure TLS](https://www.microsoft.com/pkiops/certs/Microsoft%20Azure%20TLS%20Issuing%20CA%2005.cer) | 6c3af02e7f269aa73afd0eff2a88a4a1f04ed1e5 |
| [Выдающий ЦС 06 Microsoft Azure TLS](https://www.microsoft.com/pkiops/certs/Microsoft%20Azure%20TLS%20Issuing%20CA%2006.cer) | 30e01761ab97e59a06b41ef20af6f2de7ef4f7b0 |

Если приложение проверяет сертификат в коде, необходимо изменить его, чтобы распознать свойства (например, издатели, отпечатки) вновь закрепленных сертификатов. Эта дополнительная проверка должна охватывать все прикрепленные сертификаты, чтобы быть более будущим.

## <a name="next-steps"></a>Дальнейшие действия

Если у вас есть дополнительные вопросы, свяжитесь с нами через [службу поддержки](https://azure.microsoft.com/support/options/).  
