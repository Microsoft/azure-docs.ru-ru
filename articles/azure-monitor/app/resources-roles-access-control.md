---
title: Ресурсы, роли и контроль доступа в Azure Application Insights | Документация Майкрософт
description: Владельцы, участники и читатели Insights вашей организации.
ms.topic: conceptual
ms.date: 02/14/2019
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 4254170ad100b634439cc846dc381e305ae71d1f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100589606"
---
# <a name="resources-roles-and-access-control-in-application-insights"></a>Ресурсы, роли и контроль доступа в Application Insights

Вы можете контролировать, кто имеет доступ к данным в Azure [Application Insights][start]для чтения и обновления, используя [Управление доступом на основе ролей Azure (Azure RBAC)](../../role-based-access-control/role-assignments-portal.md).

> [!IMPORTANT]
> Вы также можете предоставлять доступ пользователям в **группе ресурсов или подписке** , к которым относится ресурс приложения, а не в самом ресурсе. Назначьте им роль **участника компонента Application Insights** . Это обеспечит универсальный контроль доступа к веб-тестам и оповещениям с помощью ресурса приложения. [Подробнее](#access).


[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="resources-groups-and-subscriptions"></a>Ресурсы, группы и подписки

Сначала рассмотрим некоторые определения:

* **Ресурс** — это экземпляр службы Microsoft Azure. Ресурс Application Insights собирает, анализирует и отображает данные телеметрии, отправленные приложением.  Другие типы ресурсов Azure включают в себя веб-приложения, базы данных и виртуальные машины.
  
    Чтобы просмотреть ресурсы, откройте [портал Azure][portal], войдите и щелкните все ресурсы. Чтобы найти ресурс, введите часть его имени в поле фильтра.
  
    ![Список ресурсов Azure](./media/resources-roles-access-control/10-browse.png)

<a name="resource-group"></a>

* [**Группа ресурсов**][group] — каждый ресурс принадлежит одной группе. Создание группы — это удобный способ управления связанными ресурсами, особенно для контроля доступа. Например, в одну группу ресурсов можно поместить веб-приложение и ресурс Application Insights для мониторинга приложения, а также ресурс хранилища для хранения экспортированных данных.

* [**Подписка**](https://portal.azure.com). Чтобы использовать Application Insights или другие ресурсы Azure, войдите в подписку Azure. Каждая группа ресурсов относится к одной подписке Azure, где вы выбираете пакет по цене и, при использовании подписки для организации, выбираете участников и права доступа для них.
* [**Учетная запись Майкрософт**][account] — имя пользователя и пароль, используемые для входа в подписки Microsoft Azure, Xbox Live, Outlook.com и другие службы Майкрософт.

## <a name="control-access-in-the-resource-group"></a><a name="access"></a> Контроль доступа в группе ресурсов

Важно понимать, что кроме ресурса, созданного для приложения, существуют также отдельные скрытые ресурсы для оповещений и веб-тестов. Они присоединены к той же [группе ресурсов](#resource-group) , что и ресурс Application Insights. В нее также можно поместить другие службы Azure, такие как веб-сайты или службы хранилища.

## <a name="to-provide-access-to-another-user"></a>Предоставление доступа другому пользователю

Для этого у вас должны быть права владельца подписки или группы ресурсов.

У пользователя должна быть [учетная запись Майкрософт][account] или доступ к [корпоративной учетной записи Майкрософт](../../active-directory/fundamentals/sign-up-organization.md). Вы можете предоставлять доступ отдельным пользователям и группам пользователей, определенным в Azure Active Directory.

#### <a name="navigate-to-resource-group-or-directly-to-the-resource-itself"></a>Переход в группу ресурсов или непосредственно к самому ресурсу

В меню слева выберите **Управление доступом (IAM)**.

![Снимок экрана кнопки управления доступом на портале Azure](./media/resources-roles-access-control/0001-access-control.png)

**Добавить назначение ролей**

![Снимок экрана меню управления доступом с кнопкой "Добавить", выделенной красным цветом](./media/resources-roles-access-control/0002-add.png)

Представление **Добавление разрешений**, приведенное ниже, в первую очередь относится к ресурсам Application Insights. Если вы просматриваете разрешения на управление доступом на более высоком уровне, например из группы ресурсов, вы увидите дополнительные роли, не связанные с Application Insights.

Сведения обо всех встроенных ролях управления доступом на основе ролей Azure см. в статье [Встроенные роли в Azure](../../role-based-access-control/built-in-roles.md).

![Снимок экрана списка роли управления доступом пользователя](./media/resources-roles-access-control/0003-user-roles.png)

#### <a name="select-a-role"></a>Выбор роли

Во всех случаях, где это применимо, мы ссылаемся на соответствующую официальную справочную документацию.

| Роль | В группе ресурсов |
| --- | --- |
| [Владелец](../../role-based-access-control/built-in-roles.md#owner) |Может менять любые параметры, в том числе права доступа пользователей. |
| [Участник](../../role-based-access-control/built-in-roles.md#contributor) |Может изменять любое содержимое, в том числе любые ресурсы. |
| [Участник компонента Application Insights](../../role-based-access-control/built-in-roles.md#application-insights-component-contributor) |Может изменять ресурсы Application Insights. |
| [Читатель](../../role-based-access-control/built-in-roles.md#reader) |Может просматривать содержимое, но нельзя ничего изменять. |
| [Отладчик моментальных снимков Application Insights](../../role-based-access-control/built-in-roles.md#application-insights-snapshot-debugger) | Предоставляет пользователю разрешение на использование функций Snapshot Debugger Application Insights. Обратите внимание на то, что эта роль не включает в себя ни роль "Владелец", ни роль "Участник". |
| Участник управления выпусками развертывания служб Azure | Роль участника для развертывания служб с помощью функции развертывания служб Azure. |
| [Средство очистки данных](../../role-based-access-control/built-in-roles.md#data-purger) | Специальная роль для очистки персональных данных. Дополнительные сведения см. в статье [Руководство по хранению персональных данных в Application Insights](../logs/personal-data-mgmt.md).   |
| Администратор ExpressRoute | Может создавать и удалять подключения ExpressRoute, а также управлять ими.|
| [Участник Log Analytics](../../role-based-access-control/built-in-roles.md#log-analytics-contributor) | Участник Log Analytics может считывать все данные мониторинга и изменять параметры мониторинга. Изменение параметров мониторинга подразумевает добавление расширений в виртуальные машины, чтение ключей учетной записи хранения для настройки коллекции журналов в службе хранилища Azure, создание и настройку учетных записей службы автоматизации, добавление решений и настройку диагностики Azure во всех ресурсах Azure.  |
| [Читатель Log Analytics](../../role-based-access-control/built-in-roles.md#log-analytics-reader) | Читатель Log Analytics может просматривать все данные мониторинга, выполнять по ним поиск и просматривать параметры мониторинга, в том числе конфигурацию системы диагностики Azure для всех ресурсов Azure. |
| masterreader | Позволяет пользователю просматривать все элементы, но не вносить изменения. |
| [Monitoring Contributor](../../role-based-access-control/built-in-roles.md#monitoring-contributor) (Участник мониторинга) | Может читать все данные мониторинга и параметры обновления мониторинга.|
| [Издатель метрик мониторинга](../../role-based-access-control/built-in-roles.md#monitoring-metrics-publisher) | Включает публикацию метрик ресурсов Azure. |
| [Monitoring Reader](../../role-based-access-control/built-in-roles.md#monitoring-reader) (Читатель данных мониторинга) | Может читать все данные мониторинга. |
| Участник политики ресурсов (предварительная версия) | Обратное заполнение пользователей из EA с правами на создание и изменение политики ресурсов, создание запросов в службу поддержки, а также чтение ресурсов и иерархии.  |
| [Администратор доступа пользователей](../../role-based-access-control/built-in-roles.md#user-access-administrator) | Позволяет пользователю управлять доступом других пользователей к ресурсам Azure.|
| [Участник веб-сайта](../../role-based-access-control/built-in-roles.md#website-contributor) | Позволяет управлять веб-сайтами (не веб-планами), но не доступом к ним.|

«Изменение» включает в себя создание, удаление и обновление:

* Ресурсы
* Веб-тесты
* видны узлы
* Непрерывный экспорт

#### <a name="select-the-user"></a>Выбор пользователя

Если в каталоге нет необходимого пользователя, вы можете пригласить любого пользователя с учетной записью Майкрософт
(если он использует такие службы, как Outlook.com, OneDrive, Windows Phone или XBox Live, значит, у него есть учетная запись Майкрософт).

## <a name="related-content"></a>См. также

* [Управление доступом Azure на основе ролей (Azure RBAC)](../../role-based-access-control/role-assignments-portal.md)

## <a name="powershell-query-to-determine-role-membership"></a>Запрос PowerShell для определения членства в роли

Так как определенные роли могут быть связаны с уведомлениями и оповещениями по электронной почте, возможность создавать список пользователей, принадлежащих к данной роли, будет полезной. Для создания списков такого типа мы предлагаем следующие примеры запросов, которые можно настроить в соответствии с вашими требованиями.

### <a name="query-entire-subscription-for-admin-roles--contributor-roles"></a>Запрос всей подписки для назначения ролей администратора и ролей участника

```powershell
(Get-AzRoleAssignment -IncludeClassicAdministrators | Where-Object {$_.RoleDefinitionName -in @('ServiceAdministrator', 'CoAdministrator', 'Owner', 'Contributor') } | Select -ExpandProperty SignInName | Sort-Object -Unique) -Join ", "
```

### <a name="query-within-the-context-of-a-specific-application-insights-resource-for-owners-and-contributors"></a>Запрос в контексте конкретного ресурса Application Insights для владельцев и участников

```powershell
$resourceGroup = "RGNAME"
$resourceName = "AppInsightsName"
$resourceType = "microsoft.insights/components"
(Get-AzRoleAssignment -ResourceGroup $resourceGroup -ResourceType $resourceType -ResourceName $resourceName | Where-Object {$_.RoleDefinitionName -in @('Owner', 'Contributor') } | Select -ExpandProperty SignInName | Sort-Object -Unique) -Join ", "
```

### <a name="query-within-the-context-of-a-specific-resource-group-for-owners-and-contributors"></a>Запрос в контексте конкретной группы ресурсов для владельцев и участников

```powershell
$resourceGroup = "RGNAME"
(Get-AzRoleAssignment -ResourceGroup $resourceGroup | Where-Object {$_.RoleDefinitionName -in @('Owner', 'Contributor') } | Select -ExpandProperty SignInName | Sort-Object -Unique) -Join ", "
```

<!--Link references-->

[account]: https://account.microsoft.com
[group]: ../../azure-resource-manager/management/overview.md
[portal]: https://portal.azure.com/
[start]: ./app-insights-overview.md
