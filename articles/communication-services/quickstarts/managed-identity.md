---
title: Использование управляемых удостоверений в службах связи
titleSuffix: An Azure Communication Services quickstart
description: Управляемые удостоверения позволяют авторизовать доступ к службам связи Azure из приложений, работающих на виртуальных машинах Azure, в приложениях функций и других ресурсах.
services: azure-communication-services
author: peiliu
ms.service: azure-communication-services
ms.topic: how-to
ms.date: 2/24/2021
ms.author: peiliu
ms.reviewer: mikben
zone_pivot_groups: acs-js-csharp-java-python
ms.openlocfilehash: 2aea4daddb9cccbf7f268c596fd121357cc17b6d
ms.sourcegitcommit: 8d1b97c3777684bd98f2cfbc9d440b1299a02e8f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102486650"
---
# <a name="use-managed-identities"></a>Использование управляемых удостоверений
Приступая к работе со службами связи Azure с помощью управляемых удостоверений. Удостоверения служб связи и клиентские библиотеки SMS поддерживают проверку подлинности Azure Active Directory (Azure AD) с помощью [управляемых удостоверений для ресурсов Azure](../../active-directory/managed-identities-azure-resources/overview.md).

В этом кратком руководстве показано, как авторизовать доступ к клиентским библиотекам удостоверений и SMS из среды Azure, которая поддерживает управляемые удостоверения. В нем также описывается тестирование кода в среде разработки.

::: zone pivot="programming-language-csharp"
[!INCLUDE [.NET](./includes/managed-identity-net.md)]
::: zone-end

::: zone pivot="programming-language-javascript"
[!INCLUDE [JavaScript](./includes/managed-identity-js.md)]
::: zone-end

::: zone pivot="programming-language-java"
[!INCLUDE [Java](./includes/managed-identity-java.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [Python](./includes/managed-identity-python.md)]
::: zone-end

## <a name="next-steps"></a>Дальнейшие действия

- [Дополнительные сведения об управлении доступом на основе ролей в Azure](../../../articles/role-based-access-control/index.yml)
- [Дополнительные сведения о библиотеке удостоверений Azure для .NET](/dotnet/api/overview/azure/identity-readme)
- [Создание маркеров доступа пользователей](../quickstarts/access-tokens.md)
- [Отправка SMS](../quickstarts/telephony-sms/send.md)
- [Дополнительные сведения об SMS](../concepts/telephony-sms/concepts.md)
