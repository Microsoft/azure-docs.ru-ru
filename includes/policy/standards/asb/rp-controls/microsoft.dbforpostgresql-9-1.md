---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 03/31/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 44ece75ffeabc11bc047ee7ae3886fd5e263e81c
ms.sourcegitcommit: 99fc6ced979d780f773d73ec01bf651d18e89b93
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "107284292"
---
|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[База данных Azure для PostgreSQL должна использовать геоизбыточное резервное копирование](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F48af4db5-9b8b-401c-8e74-076be876a430) |База данных Azure для PostgreSQL позволяет выбрать вариант избыточности для сервера базы данных. Можно задать геоизбыточное хранилище резервных копий, в котором данные хранятся не только в том регионе, в котором размещен сервер, — они также реплицируются в парный регион для восстановления в случае сбоя в регионе. Геоизбыточное хранилище резервных копий можно настроить только на этапе создания сервера. |Audit, Disabled |[1.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/GeoRedundant_DBForPostgreSQL_Audit.json) |
