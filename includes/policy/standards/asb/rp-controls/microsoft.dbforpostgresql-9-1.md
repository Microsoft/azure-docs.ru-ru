---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 04/14/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 18cbe3f50ee107d710b21e95f6f4774ecd65263c
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/14/2021
ms.locfileid: "107511887"
---
|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[База данных Azure для PostgreSQL должна использовать геоизбыточное резервное копирование](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F48af4db5-9b8b-401c-8e74-076be876a430) |База данных Azure для PostgreSQL позволяет выбрать вариант избыточности для сервера базы данных. Можно задать геоизбыточное хранилище резервных копий, в котором данные хранятся не только в том регионе, в котором размещен сервер, — они также реплицируются в парный регион для восстановления в случае сбоя в регионе. Геоизбыточное хранилище резервных копий можно настроить только на этапе создания сервера. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/GeoRedundant_DBForPostgreSQL_Audit.json) |
