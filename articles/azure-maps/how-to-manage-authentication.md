---
title: Управление аутентификацией
titleSuffix: Azure Maps
description: Познакомьтесь с Azure Mapsной проверкой подлинности. Узнайте, какой подход лучше подходит в этом сценарии. Узнайте, как использовать портал для просмотра параметров проверки подлинности.
author: anastasia-ms
ms.author: v-stharr
ms.date: 06/12/2020
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.openlocfilehash: 57e847116febcea66e1e3ac4ba131617463b6c94
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92895772"
---
# <a name="manage-authentication-in-azure-maps"></a>Управление аутентификацией в Azure Maps

После создания учетной записи Azure Maps создаются идентификатор клиента и ключи для поддержки проверки подлинности Azure Active Directory (Azure AD) и аутентификации с общим ключом.

## <a name="view-authentication-details"></a>Просмотр сведений об аутентификации

После создания учетной записи Azure Maps создаются первичный и вторичный ключи. Рекомендуется использовать первичный ключ в качестве ключа подписки при [использовании проверки подлинности с общим ключом для вызова Azure Maps](./azure-maps-authentication.md#shared-key-authentication). Вторичный ключ можно использовать в таких сценариях, как изменение ключевых изменений. Дополнительные сведения см. [в разделе Проверка подлинности в Azure Maps](./azure-maps-authentication.md).

Сведения о проверке подлинности можно просмотреть в портал Azure. В учетной записи в меню **Параметры** выберите **Проверка подлинности**.

> [!div class="mx-imgBorder"]
> ![Подробные сведения о проверке подлинности](./media/how-to-manage-authentication/how-to-view-auth.png)

## <a name="discover-category-and-scenario"></a>Определение категории и сценария

В зависимости от потребностей приложения существуют определенные пути для защиты приложения. Azure AD определяет категории для поддержки широкого спектра потоков проверки подлинности. Сведения о том, какая категория подходит для приложения, см. в разделе [категории приложений](../active-directory/develop/authentication-flows-app-scenarios.md#application-categories) .

> [!NOTE]
> Даже если используется проверка подлинности с использованием общего ключа, основные сведения о категориях и сценариях помогут защитить приложение.

## <a name="determine-authentication-and-authorization"></a>Определение проверки подлинности и авторизации

В следующей таблице описаны распространенные сценарии проверки подлинности и авторизации в Azure Maps. В таблице представлено сравнение типов защиты, предлагаемых каждым сценарием.

> [!IMPORTANT]
> Корпорация Майкрософт рекомендует реализовать Azure Active Directory (Azure AD) с помощью управления доступом на основе ролей Azure (Azure RBAC) для рабочих приложений.

| Сценарий                                                                                    | Аутентификация | Авторизация | Усилия по разработке | Рабочие усилия |
| ------------------------------------------------------------------------------------------- | -------------- | ------------- | ------------------ | ------------------ |
| [Доверенная управляющая программа/неинтерактивное клиентское приложение](./how-to-secure-daemon-app.md)        | Общий ключ     | Н/Д           | Средний             | Высокий               |
| [Доверенная управляющая программа/неинтерактивное клиентское приложение](./how-to-secure-daemon-app.md)        | Azure AD       | Высокий          | Низкий                | Средний             |
| [Одностраничное веб-приложение с интерактивным единым входом](./how-to-secure-spa-users.md) | Azure AD       | Высокий          | Средний             | Средн.             |
| [Одностраничное приложение Web с неинтерактивным входом](./how-to-secure-spa-app.md)      | Azure AD       | Высокий          | Средний             | Средн.             |
| [Веб-приложение с интерактивным единым входом](./how-to-secure-webapp-users.md)          | Azure AD       | Высокий          | Высокий               | Средний             |
| [Устройство с ограниченным входным устройством IoT](./how-to-secure-device-code.md)                     | Azure AD       | Высокий          | Средний             | Средн.             |

Ссылки в таблице ведут к подробным сведениям о конфигурации для каждого сценария.

## <a name="view-role-definitions"></a>Просмотр определений ролей

Чтобы просмотреть роли Azure, доступные для Azure Maps, перейдите на страницу **Управление доступом (IAM)**. Выберите **роли**, а затем найдите роли, которые начинаются с *Azure Maps*. Эти Azure Maps роли являются ролями, к которым можно предоставить доступ.

> [!div class="mx-imgBorder"]
> ![Просмотр доступных ролей](./media/how-to-manage-authentication/how-to-view-avail-roles.png)

## <a name="view-role-assignments"></a>Просмотр назначений ролей

Чтобы просмотреть пользователей и приложения, которым был предоставлен доступ к Azure Maps, перейдите в раздел **Управление доступом (IAM)**. Выберите **назначения ролей**, а затем выполните фильтрацию по **Azure Maps**.

> [!div class="mx-imgBorder"]
> ![Просмотр пользователей и приложений, которым был предоставлен доступ](./media/how-to-manage-authentication/how-to-view-amrbac.png)

## <a name="request-tokens-for-azure-maps"></a>Запрос маркеров для Azure Maps

Запросите маркер из конечной точки маркера Azure AD. В запросе Azure AD используйте следующие сведения:

| Среда Azure      | Конечная точка маркера Azure AD             | Идентификатор ресурса Azure              |
| ---------------------- | ----------------------------------- | ------------------------------ |
| Общедоступное облако Azure:     | `https://login.microsoftonline.com` | `https://atlas.microsoft.com/` |
| Облако Azure для государственных организаций | `https://login.microsoftonline.us`  | `https://atlas.microsoft.com/` |

Дополнительные сведения о запросе маркеров доступа из Azure AD для пользователей и субъектов-служб см. в статье [сценарии проверки подлинности для Azure AD](../active-directory/develop/authentication-vs-authorization.md) и просмотрите конкретные сценарии в таблице [сценариев](./how-to-manage-authentication.md#determine-authentication-and-authorization).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в статье [Azure AD и Azure Maps веб-пакет SDK](./how-to-use-map-control.md).

Найдите метрики использования API для учетной записи Azure Maps:
> [!div class="nextstepaction"]
> [Просмотр метрик использования](how-to-view-api-usage.md)

Просмотрите примеры, демонстрирующие интеграцию Azure AD с Azure Maps:

> [!div class="nextstepaction"]
> [Примеры проверки подлинности Azure AD](https://github.com/Azure-Samples/Azure-Maps-AzureAD-Samples)