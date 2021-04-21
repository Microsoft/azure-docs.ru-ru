---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 04/14/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 4f02a1cb7efacadbdd3f77ddceebb6792eb2f95c
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/14/2021
ms.locfileid: "107513535"
---
|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Для серверов баз данных PostgreSQL должно быть включено принудительное использование SSL-соединения](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fd158790f-bfb0-486c-8631-2dc6b4e8e6af) |База данных Azure для PostgreSQL поддерживает подключение сервера Базы данных Azure для PostgreSQL к клиентским приложениям с помощью протокола SSL. Принудительное использование SSL-соединений между сервером базы данных и клиентскими приложениями помогает обеспечить защиту от атак "злоумышленник в середине" за счет шифрования потока данных между сервером и приложением. Эта конфигурация обеспечивает постоянную поддержку протокола SSL для доступа к серверу базы данных. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/PostgreSQL_EnableSSL_Audit.json) |
