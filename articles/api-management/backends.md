---
title: Служба управления API Azure | Документация Майкрософт
description: Дополнительные сведения о пользовательских элементах управления в API
services: api-management
documentationcenter: ''
author: dlepow
editor: ''
ms.service: api-management
ms.topic: article
ms.date: 01/29/2021
ms.author: apimpm
ms.openlocfilehash: 54a46e999391507f5ec7d927f62b88fcd2169b75
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99500621"
---
# <a name="backends-in-api-management"></a>Конец в управлении API

*Серверная* часть (или *Серверная часть API*) в службе управления API — это служба HTTP, которая реализует интерфейсный API и его операции.

При импорте некоторых API Служба управления API автоматически настраивает серверную часть API. Например, служба управления API настраивает серверную часть при импорте [спецификации OpenAPI](import-api-from-oas.md), [API SOAP](import-soap-api.md)или ресурсов azure, таких как [приложение-функция Azure](import-function-app-as-api.md) или [приложение логики](import-logic-app-as-api.md), активируемые HTTP.

Управление API также поддерживает использование других ресурсов Azure, таких как [Service Fabricный кластер](how-to-configure-service-fabric-backend.md) или настраиваемая служба в качестве серверной части API. Использование этих пользовательских серверных компонентов требует дополнительной настройки, например для авторизации учетных данных запросов к серверной службе и определения операций API. Вы настраиваете эти серверные компоненты и управляете ими в портал Azure или с помощью API или средств Azure.

После создания серверной части можно ссылаться на URL-адрес внутреннего сервера в интерфейсах API. Используйте [`set-backend-service`](api-management-transformation-policies.md#SetBackendService) политику, чтобы перенаправить входящий запрос API на пользовательский сервер, а не серверную часть по умолчанию для этого API.

## <a name="benefits-of-backends"></a>Преимущества незавершенных работы

Пользовательская Серверная часть имеет несколько преимуществ, в том числе:

* Абстрагирует сведения о серверной службе, повышению удобства использования в API и улучшенном управлении  
* Простота использования путем настройки политики преобразования для существующего API
* Использует преимущества функции управления API для сохранения секретов в Azure Key Vault если [именованные значения](api-management-howto-properties.md) настроены для проверки подлинности заголовка или параметра запроса.

## <a name="next-steps"></a>Дальнейшие действия

* Настройте [Service Fabric серверную](how-to-configure-service-fabric-backend.md) часть с помощью портал Azure.
* Кроме того, с помощью шаблонов управления API [REST API](/rest/api/apimanagement), [Azure PowerShell](/powershell/module/az.apimanagement/new-azapimanagementbackend)или [Azure Resource Manager](../service-fabric/service-fabric-tutorial-deploy-api-management.md)можно настроить незавершенные работы.

