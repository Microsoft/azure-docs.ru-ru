---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 04/21/2020
ms.author: glenga
ms.openlocfilehash: 380878fedaa2f7ea6160c3c4801c8dece6e1a9ff
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "83648962"
---
#### <a name="enable-app-service-authenticationauthorization"></a>Включение проверки подлинности и авторизация службы приложений

Платформа службы приложений позволяет использовать Azure Active Directory (AAD) и несколько сторонних поставщиков удостоверений для проверки подлинности клиентов. Эта стратегия позволяет реализовать пользовательские правила авторизации для функций и работать с информацией пользователя из кода функции. Дополнительные сведения см. в разделе [Работа с удостоверениями клиентов](../articles/azure-functions/functions-bindings-http-webhook-trigger.md#working-with-client-identities) и статье [Проверка подлинности и авторизация в службе приложений Azure](../articles/app-service/overview-authentication-authorization.md).

#### <a name="use-azure-api-management-apim-to-authenticate-requests"></a>Использование службы Управления API Azure (APIM) для проверки подлинности запросов

APIM предоставляет широкий набор параметров безопасности API для входящих запросов. Дополнительные сведения см. [в статье о политиках аутентификации службы "Управление API"](../articles/api-management/api-management-authentication-policies.md). С помощью службы "Управление API" можно настроить приложение-функцию на прием запросов только с IP-адреса вашего экземпляра этой службы. Дополнительные сведения см. в разделе [Ограничения IP-адресов](../articles/azure-functions/ip-addresses.md#ip-address-restrictions).
