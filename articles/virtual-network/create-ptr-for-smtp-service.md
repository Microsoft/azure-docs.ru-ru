---
title: Настройка зон обратного просмотра для проверки заголовка SMTP
titlesuffix: Azure Virtual Network
description: В этой статье описывается, как настроить зоны обратного просмотра для проверки заголовка SMTP в Azure
services: virtual-network
documentationcenter: virtual-network
author: genlin
manager: dcscontentpm
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 10/31/2018
ms.author: genli
ms.openlocfilehash: c8ffadb8d54db0c2a99dc12e45b5990155a0505e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86135053"
---
# <a name="configure-reverse-lookup-zones-for-an-smtp-banner-check"></a>Настройка зон обратного просмотра для проверки заголовка SMTP

В этой статье описывается, как использовать обратную зону в Azure DNS и создавать обратную запись DNS (PTR) для проверки баннера SMTP.

## <a name="symptom"></a>Симптом

Если вы размещаете сервер SMTP в Microsoft Azure, при отправке или получении сообщения с удаленных почтовых серверов может появиться следующее сообщение об ошибке:

**554: No PTR Record** (554: запись типа PTR отсутствует).

## <a name="solution"></a>Решение

Для виртуального IP-адреса в Azure обратные записи создаются не в пользовательских доменах, а в доменах, принадлежащих Майкрософт.

Чтобы настроить запись типа PTR в зонах, принадлежащих Майкрософт, используйте свойство -ReverseFqdn на ресурсе PublicIpAddress. Дополнительные сведения см. в статье [Настройка обратного просмотра DNS для размещенных в Azure служб](../dns/dns-reverse-dns-for-azure-services.md).

Прежде чем настраивать запись типа PTR, убедитесь, что IP-адрес и обратное FQDN (полное доменное имя) принадлежат подписке. Если вы попытаетесь настроить обратное FQDN, не принадлежащее подписке, появится следующее сообщение об ошибке:

```output
Set-AzPublicIpAddress : ReverseFqdn mail.contoso.com that PublicIPAddress ip01 is trying to use does not belong to subscription <Subscription ID>. One of the following conditions need to be met to establish ownership:
                    
1) ReverseFqdn matches fqdn of any public ip resource under the subscription;
2) ReverseFqdn resolves to the fqdn (through CName records chain) of any public ip resource under the subscription;
3) It resolves to the ip address (through CName and A records chain) of a static public ip resource under the subscription.
```

Если вы вручную измените заголовок SMTP в соответствии с обратным FQDN по умолчанию, удаленный почтовый сервер все еще может выйти из строя, так как для него необходимо, чтобы узел заголовка SMTP соответствовал записи MX для домена.
