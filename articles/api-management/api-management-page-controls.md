---
title: Элементы управления страницей в службе управления API Azure | Документация Майкрософт
description: Сведения об элементах управления страницей, доступных для использования в шаблонах портала разработчика в службе управления API Azure.
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 11/04/2019
ms.author: apimpm
ms.openlocfilehash: b826c986a0af26bb79fb0823e4e8626f0165d460
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86250029"
---
# <a name="azure-api-management-page-controls"></a>Элементы управления страницей в службе управления API Azure
Служба управления API Azure предоставляет следующие элементы управления страницей для использования в шаблонах портала разработчика.  
  
Чтобы применить элемент управления, поместите его в нужное место шаблона портала разработчика. Некоторые элементы управления имеют параметры, как например [app-actions](#app-actions), представленный в следующем примере.  
  
```xml  
<app-actions params="{ appId: '{{app.id}}' }"></app-actions>  
```  
  
Значения для этих параметров передаются в составе модели данных для шаблона. В большинстве случаев достаточно просто скопировать представленный пример для элемента управления, и он сразу будет правильно работать. Дополнительные сведения о значениях параметров вы найдете в разделе модели данных для каждого шаблона, в котором можно использовать соответствующий элемент управления.  

Дополнительные сведения о работе с шаблонами см. в статье [Настройка портала разработчика в службе управления API Azure с помощью шаблонов](./api-management-developer-portal-templates.md).  

[!INCLUDE [api-management-portal-legacy.md](../../includes/api-management-portal-legacy.md)]

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]
  
## <a name="developer-portal-template-page-controls"></a>Элементы управления страницей в шаблонах портала разработчика  
  
-   [App-действия](#app-actions)  
-   [Базовый — вход](#basic-signin)  
-   [Управление разбиением на страницы](#paging-control)  
-   [providers](#providers)  
-   [Поиск — контроль](#search-control)  
-   [регистрация](#sign-up)  
-   [кнопка подписки](#subscribe-button)  
-   [подписка — Отмена](#subscription-cancel)  
  
##  <a name="app-actions"></a><a name="app-actions"></a> app-actions  
 Элемент управления `app-actions` предоставляет пользовательский интерфейс для взаимодействия с приложениями на странице профиля пользователя на портале разработчика.  
  
 ![Управление действиями&#45;приложений](./media/api-management-page-controls/APIM-app-actions-control.png "Элемент управления APIM App-Actions")  
  
### <a name="usage"></a>Использование  
  
```xml  
<app-actions params="{ appId: '{{app.id}}' }"></app-actions>  
```  
  
### <a name="parameters"></a>Параметры  
  
|Параметр|Описание|  
|---------------|-----------------|  
|appId|Идентификатор приложения.|  
  
### <a name="developer-portal-templates"></a>Шаблоны портала разработчика  
 Элемент управления `app-actions` можно использовать в следующих шаблонах портала разработчика:  
  
-   [Приложения](api-management-user-profile-templates.md#Applications)  
  
##  <a name="basic-signin"></a><a name="basic-signin"></a> basic-signin  
 Элемент управления `basic-signin` предоставляет интерфейс для сбора учетных данных пользователей на странице входа на портал разработчика.  
  
 ![Базовый&#45;элемент управления для входа](./media/api-management-page-controls/APIM-basic-signin-control.png "APIM Basic — элемент управления для входа")  
  
### <a name="usage"></a>Использование  
  
```xml  
<basic-SignIn></basic-SignIn>  
```  
  
### <a name="parameters"></a>Параметры  
 Нет.  
  
### <a name="developer-portal-templates"></a>Шаблоны портала разработчика  
 Элемент управления `basic-signin` можно использовать в следующих шаблонах портала разработчика:  
  
-   [Выполните вход.](api-management-page-templates.md#SignIn)  
  
##  <a name="paging-control"></a><a name="paging-control"></a> paging-control  
 Элемент `paging-control` предоставляет функцию разбиения по страницам, которую можно использовать на страницах со списками на портале разработчика.  
  
 ![элемент управления разбиением на страницы](./media/api-management-page-controls/APIM-paging-control.png "Элемент управления разбиением на страницы APIM")  
  
### <a name="usage"></a>Использование  
  
```xml  
<paging-control></paging-control>  
```  
  
### <a name="parameters"></a>Параметры  
 Нет.  
  
### <a name="developer-portal-templates"></a>Шаблоны портала разработчика  
 Элемент управления `paging-control` можно использовать в следующих шаблонах портала разработчика:  
  
-   [Список API](api-management-api-templates.md#APIList)  
  
-   [Список проблем](api-management-issue-templates.md#IssueList)  
  
-   [Список продуктов](api-management-product-templates.md#ProductList)  
  
##  <a name="providers"></a><a name="providers"></a> поставщик  
 Элемент управления `providers` предоставляет интерфейс для выбора поставщика проверки подлинности на странице входа на портал разработчика.  
  
 ![элемент управления providers](./media/api-management-page-controls/APIM-providers-control.png "Элемент управления поставщиками APIM")  
  
### <a name="usage"></a>Использование  
  
```xml  
<providers></providers>  
```  
  
### <a name="parameters"></a>Параметры  
 Нет.  
  
### <a name="developer-portal-templates"></a>Шаблоны портала разработчика  
 Элемент управления `providers` можно использовать в следующих шаблонах портала разработчика:  
  
-   [Выполните вход.](api-management-page-templates.md#SignIn)  
  
##  <a name="search-control"></a><a name="search-control"></a> search-control  
 Элемент `search-control` предоставляет функцию поиска, которую можно использовать на страницах со списками на портале разработчика.  
  
 ![элемент управления поиском](./media/api-management-page-controls/APIM-search-control.png "Элемент управления APIM Search")  
  
### <a name="usage"></a>Использование  
  
```xml  
<search-control></search-control>  
```  
  
### <a name="parameters"></a>Параметры  
 Нет.  
  
### <a name="developer-portal-templates"></a>Шаблоны портала разработчика  
 Элемент управления `search-control` можно использовать в следующих шаблонах портала разработчика:  
  
-   [Список API](api-management-api-templates.md#APIList)  
  
-   [Список продуктов](api-management-product-templates.md#ProductList)  
  
##  <a name="sign-up"></a><a name="sign-up"></a> регистрация  
 Элемент управления `sign-up` предоставляет интерфейс для сбора информации о профиле пользователя на странице регистрации на портале разработчика.  
  
 ![знаком&#45;управления](./media/api-management-page-controls/APIM-sign-up-control.png "Управление регистрацией APIM")  
  
### <a name="usage"></a>Использование  
  
```xml  
<sign-up></sign-up>  
```  
  
### <a name="parameters"></a>Параметры  
 Нет.  
  
### <a name="developer-portal-templates"></a>Шаблоны портала разработчика  
 Элемент управления `sign-up` можно использовать в следующих шаблонах портала разработчика:  
  
-   [Регистрация](api-management-page-templates.md#SignUp)  
  
##  <a name="subscribe-button"></a><a name="subscribe-button"></a> subscribe-button  
 Элемент `subscribe-button` предоставляет функцию оформления подписки пользователя на продукт.  
  
 ![Subscribe&#45;элемент управления Button](./media/api-management-page-controls/APIM-subscribe-button-control.png "Элемент управления "Подписка" APIM")  
  
### <a name="usage"></a>Использование  
  
```xml  
<subscribe-button></subscribe-button>  
```  
  
### <a name="parameters"></a>Параметры  
 Нет.  
  
### <a name="developer-portal-templates"></a>Шаблоны портала разработчика  
 Элемент управления `subscribe-button` можно использовать в следующих шаблонах портала разработчика:  
  
-   [Продукт](api-management-product-templates.md#Product)  
  
##  <a name="subscription-cancel"></a><a name="subscription-cancel"></a> subscription-cancel  
 Элемент управления `subscription-cancel` предоставляет интерфейс для отмены подписки на продукт на странице профиля пользователя на портале разработчика.  
  
 ![элемент управления&#45;отмены подписки](./media/api-management-page-controls/APIM-subscription-cancel-control.png "Подписка APIM — отмена элемента управления")  
  
### <a name="usage"></a>Использование  
  
```xml  
<subscription-cancel params="{ subscriptionId: '{{subscription.id}}', cancelUrl: '{{subscription.cancelUrl}}' }">  
</subscription-cancel>  
  
```  
  
### <a name="parameters"></a>Параметры  
  
|Параметр|Описание|  
|---------------|-----------------|  
|subscriptionId|Идентификатор подписки, которую нужно отменить.|  
|cancelUrl|URL-адрес для отмены подписки.|  
  
### <a name="developer-portal-templates"></a>Шаблоны портала разработчика  
 Элемент управления `subscription-cancel` можно использовать в следующих шаблонах портала разработчика:  
  
-   [Продукт](api-management-product-templates.md#Product)

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о работе с шаблонами см. в статье [Настройка портала разработчика в службе управления API Azure с помощью шаблонов](api-management-developer-portal-templates.md).
