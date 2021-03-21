---
title: Управление ватчлистс в Sentinel Azure с помощью REST API | Документация Майкрософт
description: В этой статье описывается, как можно управлять ватчлистс Azure Sentinel с помощью Log Analytics "REST API для создания, изменения и удаления ватчлистс и их элементов.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: reference
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/11/2021
ms.author: yelevin
ms.openlocfilehash: ea571f9b033ba82709a13c6d32649f3228ee04b1
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98798498"
---
# <a name="manage-watchlists-in-azure-sentinel-using-rest-api"></a>Управление ватчлистс в Sentinel Azure с помощью REST API

> [!IMPORTANT]
> Функция ватчлистс сейчас доступна в **предварительной версии**. Ознакомьтесь с дополнительными [условиями использования Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) предварительных версий для дополнительных юридических условий, которые относятся к функциям Azure, которые доступны в бета-версии, предварительном просмотре или еще не выпущены в общедоступную версию.

Azure Sentinel, которая встроена в Azure Monitor Log Analytics, позволяет использовать Log Analytics "REST API для управления ватчлистс. В этом документе показано, как создавать, изменять и удалять ватчлистс и их элементы с помощью REST API.  Ватчлистс, созданные таким образом, будут отображаться в пользовательском интерфейсе Sentinel Azure.

## <a name="common-uri-parameters"></a>Общие параметры URI

Ниже приведены общие параметры URI для всех команд API списка воспроизведения:

| Имя | В | Обязательно | Тип | Описание |
|-|-|-|-|-|
| **Подписки** | path | да | Код GUID | Идентификатор подписки Azure |
| **ResourceGroupName** | path | да | строка | имя группы ресурсов в подписке |
| **WorkspaceName** | path | да | строка | имя рабочей области Log Analytics |
| **{Ватчлисталиас}** | path | да * | строка | имя заданного списка воспроизведения<br>\* Не требуется при получении всех ватчлистс |
| **{Ватчлиститемид}** | path | Да * * | Код GUID | Идентификатор элемента, который необходимо создать в, добавить в списка воспроизведения или удалить из него.<br>\*\* Требуется только для команд элемента списка воспроизведения |
| **{API-Version}** | query | да | строка | Версия протокола, используемого для выполнения этого запроса. Начиная с 29 октября 2020 г. версия API списка воспроизведения — *2019-01-01-Preview* . |
| **{Беарертокен}** | авторизация | да | token | маркер авторизации носителя |
|  

## <a name="retrieve-all-watchlists"></a>Получить все ватчлистс

Эта команда извлекает все ватчлистс, связанные с рабочей областью, без их элементов.

### <a name="request-uri"></a>Универсальный код ресурса (URI) запроса
(Универсальный код ресурса (URI) — это одна строка, которая работает для удобства чтения.

| Метод | Универсальный код ресурса (URI) запроса |
|-|-|
| GET | `https://management.azure.com/subscriptions/{{subscriptionId}}/`<br>`resourceGroups/{{resourceGroupName}}/`<br>`providers/Microsoft.OperationalInsights/`<br>`workspaces/{{workspaceName}}/`<br>`providers/Microsoft.SecurityInsights/`<br>`watchlists?api-version={{api-version}}` |
|

### <a name="responses"></a>Ответы

| Код состояния | Текст ответа | Описание |
|-|-|-|
| 200/ОК | Список существующих ватчлистс или пустых, если списка воспроизведения не найден |  |
| 400/недопустимый запрос |  | Неправильный синтаксис запроса, недопустимый параметр запроса... |
|

## <a name="look-up-a-watchlist-by-name"></a>Поиск списка воспроизведения по имени

Эта команда извлекает определенные списка воспроизведения, связанные с рабочей областью, без элементов.

### <a name="request-uri"></a>Универсальный код ресурса (URI) запроса
(Универсальный код ресурса (URI) — это одна строка, которая работает для удобства чтения.

| Метод | Универсальный код ресурса (URI) запроса |
|-|-|
| GET | `https://management.azure.com/subscriptions/{{subscriptionId}}/`<br>`resourceGroups/{{resourceGroupName}}/`<br>`providers/Microsoft.OperationalInsights/`<br>`workspaces/{{workspaceName}}/`<br>`providers/Microsoft.SecurityInsights/`<br>`watchlists/{{watchlistAlias}}?api-version={{api-version}}` |
|

### <a name="responses"></a>Ответы

| Код состояния | Текст ответа | Описание |
|-|-|-|
| 200/ОК | Запрошенный списка воспроизведения |  |
| 400/недопустимый запрос |  | Неправильный синтаксис запроса, недопустимый параметр запроса... |
| 404/не найдено |  | Не удалось найти списка воспроизведения с запрошенным именем |
|

## <a name="create-a-watchlist"></a>Создание списка воспроизведения

Эта команда создает списка воспроизведения и добавляет в нее элементы. Он выполняет два вызова этой конечной точки для создания списка воспроизведения и ее элементов: первый создает списка воспроизведения (родительский элемент), а второй добавляет элементы.

### <a name="request-uri"></a>Универсальный код ресурса (URI) запроса
(Универсальный код ресурса (URI) — это одна строка, которая работает для удобства чтения.

| Метод | Универсальный код ресурса (URI) запроса |
|-|-|
| PUT | `https://management.azure.com/subscriptions/{{subscriptionId}}/`<br>`resourceGroups/{{resourceGroupName}}/`<br>`providers/Microsoft.OperationalInsights/`<br>`workspaces/{{workspaceName}}/`<br>`providers/Microsoft.SecurityInsights/`<br>`watchlists/{{watchlistAlias}}?api-version={{api-version}}` |
|

### <a name="request-body"></a>Текст запроса

Ниже приведен пример текста запроса для запроса на создание списка воспроизведения.

```json
{
    "properties": {
        "displayName": "High Value Assets Watchlist",
        "source": "Local file",
        "provider": "Microsoft",
        "numberOfLinesToSkip":"1",
        "rawContent": "metadata line\nheader1, header2\nval1, val2",
        "contentType": "text/csv"
    }
}
```

### <a name="responses"></a>Ответы

| Код состояния | Текст ответа | Описание |
|-|-|-|
| 200/ОК | Списка воспроизведения, созданный запросом без элементов |  |
| 400/недопустимый запрос |  | Неправильный синтаксис запроса, недопустимый параметр запроса... |
| 409/конфликт |  | Сбой операции, существует существующий списка воспроизведения с тем же псевдонимом |
|

## <a name="delete-a-watchlist"></a>Удаление списка воспроизведения

Эта команда удаляет списка воспроизведения и его элементы.

### <a name="request-uri"></a>Универсальный код ресурса (URI) запроса
(Универсальный код ресурса (URI) — это одна строка, которая работает для удобства чтения.

| Метод | Универсальный код ресурса (URI) запроса |
|-|-|
| DELETE | `https://management.azure.com/subscriptions/{{subscriptionId}}/`<br>`resourceGroups/{{resourceGroupName}}/`<br>`providers/Microsoft.OperationalInsights/`<br>`workspaces/{{workspaceName}}/`<br>`providers/Microsoft.SecurityInsights/`<br>`watchlists/{{watchlistAlias}}?api-version={{api-version}}` |
|

### <a name="responses"></a>Ответы

| Код состояния | Текст ответа | Описание |
|-|-|-|
| 200/ОК | Пустой текст ответа |  |
| 204/нет содержимого | Пустой текст ответа | Ничего не удалено |
| 400/недопустимый запрос |  | Неправильный синтаксис запроса, недопустимый параметр запроса... |
|

## <a name="add-or-update-a-watchlist-item"></a>Добавление или обновление элемента списка воспроизведения

Если эта команда выполняется на существующем *ватчлиситемид* (GUID), элемент будет обновляться данными, предоставленными в тексте запроса. В противном случае будет создан новый элемент.

### <a name="request-uri"></a>Универсальный код ресурса (URI) запроса
(Универсальный код ресурса (URI) — это одна строка, которая работает для удобства чтения.

| Метод | Универсальный код ресурса (URI) запроса |
|-|-|
| PUT | `https://management.azure.com/subscriptions/{{subscriptionId}}/`<br>`resourceGroups/{{resourceGroupName}}/`<br>`providers/Microsoft.OperationalInsights/`<br>`workspaces/{{workspaceName}}/`<br>`providers/Microsoft.SecurityInsights/`<br>`watchlists/{{watchlistAlias}}/`<br>`watchlistitems/{{watchlistItemId}}?api-version={{api-version}}` |
|

### <a name="request-body"></a>Текст запроса

Ниже приведен пример текста запроса для запроса на добавление или обновление элемента списка воспроизведения.

```json
{
    "properties": {
      "itemsKeyValue": {
           "Gateway subnet": "10.0.255.224/27",
           "Web Tier": "10.0.1.0/24",
           "Business tier": "10.0.2.0/24",
           "Private DMZ in": "10.0.0.0/27",
           "Public DMZ out": "10.0.0.96/27"
      }
}
}
```

### <a name="responses"></a>Ответы

| Код состояния | Текст ответа | Описание |
|-|-|-|
| 200/ОК | Элемент списка воспроизведения, созданный или обновленный запросом |  |
| 400/недопустимый запрос |  | Неправильный синтаксис запроса, недопустимый параметр запроса... |
| 409/конфликт |  | Сбой операции, существует существующий списка воспроизведения с тем же псевдонимом |
|

## <a name="delete-a-watchlist-item"></a>Удаление элемента списка воспроизведения

Эта команда удаляет существующий элемент списка воспроизведения.

### <a name="request-uri"></a>Универсальный код ресурса (URI) запроса
(Универсальный код ресурса (URI) — это одна строка, которая работает для удобства чтения.

| Метод | Универсальный код ресурса (URI) запроса |
|-|-|
| DELETE | `https://management.azure.com/subscriptions/{{subscriptionId}}/`<br>`resourceGroups/{{resourceGroupName}}/`<br>`providers/Microsoft.OperationalInsights/`<br>`workspaces/{{workspaceName}}/`<br>`providers/Microsoft.SecurityInsights/`<br>`watchlists/{{watchlistAlias}}/`<br>`watchlistitems/{{watchlistItemId}}?api-version={{api-version}}` |
|

### <a name="responses"></a>Ответы

| Код состояния | Текст ответа | Описание |
|-|-|-|
| 200/ОК | Пустой текст ответа |  |
| 204/нет содержимого | Пустой текст ответа | Ничего не удалено |
| 400/недопустимый запрос |  | Неправильный синтаксис запроса, недопустимый параметр запроса... |
|

## <a name="next-steps"></a>Дальнейшие действия

Из этой статьи вы узнали, как управлять ватчлистс и их элементами в Sentinel Azure с помощью Log Analytics API. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.

- Дополнительные сведения о [ватчлистс](watchlists.md)
- Изучите другие варианты использования [Azure Sentinel API](/rest/api/securityinsights/)