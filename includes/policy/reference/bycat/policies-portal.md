---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 03/24/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 0df4035c75c124b33e299e1ccb2258ef41caebf6
ms.sourcegitcommit: bb330af42e70e8419996d3cba4acff49d398b399
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105033658"
---
|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Общедоступные панели мониторинга не должны иметь плитки с поддержкой markdown, содержащие встроенное содержимое](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F04c655fe-0ac7-48ae-9a32-3a2e208c7624) |Запрет на создание общедоступной панели мониторинга с плитками с поддержкой Markdown, которые содержат встроенное содержимое, и требование к сохранению содержимого в виде файла Markdown, развернутого на сервере Интернета. Используя для плитки Markdown встроенное содержимое, вы лишитесь возможности управлять шифрованием содержимого. Однако вы сможете выполнить шифрование, двукратное шифрование и даже использовать собственные ключи, настроив собственное хранилище. При включении этой политики пользователи смогут использовать только предварительную версию REST API общедоступных панелей мониторинга за 01.09.2020 или более позднюю версию. |Audit, Deny, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Portal/SharedDashboardInlineContent_Deny.json) |
