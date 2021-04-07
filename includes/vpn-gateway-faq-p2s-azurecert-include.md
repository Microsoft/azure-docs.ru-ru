---
title: включить файл
description: включить файл
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 08/14/2019
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 5ef67580928a45609f50d3fe798eb9d054265c0a
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "93375910"
---
[!INCLUDE [P2S FAQ All](vpn-gateway-faq-p2s-all-include.md)]

### <a name="what-should-i-do-if-im-getting-a-certificate-mismatch-when-connecting-using-certificate-authentication"></a>Что делать, если при подключении с использованием проверки подлинности сертификата обнаружено несоответствие сертификата?

Снимите флажок **Подтверждать удостоверение сервера с помощью проверки сертификата** или **добавьте FQDN сервера вместе с сертификата** при создании профиля вручную. Это можно сделать, выполнив команду **rasphone** в командной строке. Затем нужно выбрать профиль в раскрывающемся списке.

Обход проверки удостоверения сервера в целом не рекомендуется, но при аутентификации сертификата Azure для проверки сервера в протоколе туннелирования VPN (IKEv2/SSTP) и протоколе EAP используется один и тот же сертификат. Так как сертификат сервера и FQDN уже проверены протоколом туннелирования VPN, повторная проверка в EAP не требуется.

![Аутентификация "точка — сеть"](./media/vpn-gateway-faq-p2s-all-include/servercert.png "Сертификат сервера")

### <a name="can-i-use-my-own-internal-pki-root-ca-to-generate-certificates-for-point-to-site-connectivity"></a>Можно ли с помощью корневого ЦС собственной внутренней системы PKI создать сертификаты для подключения "точка — сеть"?

Да. Раньше поддерживались только самозаверяющие корневые сертификаты. Вы по-прежнему можете загружать до 20 корневых сертификатов.

### <a name="can-i-use-certificates-from-azure-key-vault"></a>Можно ли использовать сертификаты из Azure Key Vault?

Нет.

### <a name="what-tools-can-i-use-to-create-certificates"></a>Какие инструменты можно использовать для создания сертификатов?

Можно использовать решение Enterprise PKI (внутренняя инфраструктура открытых ключей), Azure PowerShell, MakeCert и OpenSSL.

### <a name="are-there-instructions-for-certificate-settings-and-parameters"></a><a name="certsettings"></a>Есть инструкции по выбору параметров сертификата?

* **Внутренняя инфраструктура открытых ключей и решение Enterprise PKI:** см. инструкции по [созданию сертификатов](../articles/vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md#generatecert).

* **Azure PowerShell:** см. инструкции по работе с [Azure PowerShell](../articles/vpn-gateway/vpn-gateway-certificates-point-to-site.md).

* **MakeCert:** см. инструкции по работе с [MakeCert](../articles/vpn-gateway/vpn-gateway-certificates-point-to-site-makecert.md).

* **OpenSSL:** 

    * При экспорте сертификатов преобразуйте корневой сертификат в кодировку Base64.

    * Для сертификата клиента:

      * При создании закрытого ключа укажите длину 4096.
      * При создании сертификата для параметра *-extensions* укажите значение *usr_cert*.
