---
author: baanders
description: Включаемый файл для настройки локальной проверки подлинности для DefaultAzureCredential в примерах Azure Digital Twins — с вводными сведениями
ms.service: digital-twins
ms.topic: include
ms.date: 10/22/2020
ms.author: baanders
ms.openlocfilehash: dcef8030c47e771e5cd771dac09b5f96e3d38abd
ms.sourcegitcommit: 6386854467e74d0745c281cc53621af3bb201920
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/08/2021
ms.locfileid: "102473753"
---
### <a name="set-up-local-azure-credentials"></a>Настройка локальных учетных данных Azure

В этом примере используется [DefaultAzureCredential](/dotnet/api/azure.identity.defaultazurecredential) (часть библиотеки `Azure.Identity`) для аутентификации пользователей с помощью экземпляра Azure Digital Twins, запускаемого на локальном компьютере. Дополнительные сведения о различных способах аутентификации клиентского приложения в Azure Digital Twins см. в [*практическом руководстве по написанию кода аутентификации приложения*](../articles/digital-twins/how-to-authenticate-client.md).

[!INCLUDE [Azure Digital Twins: local credentials prereq (inner)](digital-twins-local-credentials-inner.md)]