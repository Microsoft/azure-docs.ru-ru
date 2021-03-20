---
title: Шаблоны продуктов в службе управления API Azure | Документация Майкрософт
description: Сведения о настройке содержимого страниц продуктов на портале разработчика в службе управления API Azure.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: 49f9254c-4c5f-4ed4-9c8d-798f44e805ee
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 11/04/2019
ms.author: apimpm
ms.openlocfilehash: 4c8cd4aa3e91c5d69c40e47683818ed8bc9be338
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86249909"
---
# <a name="product-templates-in-azure-api-management"></a>Шаблоны продуктов в службе управления API Azure

Служба управления API Azure позволяет настраивать содержимое страниц портала разработчика с помощью набора шаблонов. С помощью этих шаблонов вы можете гибко настраивать содержимое страниц, используя синтаксис [DotLiquid](http://dotliquidmarkup.org/), любой удобный текстовый редактор, например [DotLiquid для разработчиков](https://github.com/dotliquid/dotliquid/wiki/DotLiquid-for-Designers), и предоставленный набор локализованных [строковых ресурсов](api-management-template-resources.md#strings), [ресурсов глифов](api-management-template-resources.md#glyphs), а также [элементов управления страницы](api-management-page-controls.md).  
  
 С помощью шаблонов в этом разделе вы сможете настроить содержимое страниц продуктов на портале разработчика.  
  
-   [Список продуктов](#ProductList)  
  
-   [Продукт](#Product)  
  
> [!NOTE]
>  Примеры шаблонов по умолчанию включены в следующую документацию, но могут в любой момент измениться, так как ведется постоянная работа по их улучшению. Актуальные шаблоны по умолчанию можно просмотреть на портале разработчика, перейдя к требуемому отдельному шаблону. Дополнительные сведения о работе с шаблонами см. в статье [Настройка портала разработчика в службе управления API Azure с помощью шаблонов](./api-management-developer-portal-templates.md).  

[!INCLUDE [api-management-portal-legacy.md](../../includes/api-management-portal-legacy.md)]

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]
  
##  <a name="product-list"></a><a name="ProductList"></a> Список продуктов  
 Шаблон **списка продуктов** позволяет настроить текст страницы со списком продуктов на портале разработчика.  
  
 ![Список продуктов](./media/api-management-product-templates/APIM_ProductsListTemplatePage.png "APIM_ProductsListTemplatePage")  
  
### <a name="default-template"></a>Шаблон по умолчанию  
  
```xml  
<search-control></search-control>  
<div class="row">  
    <div class="col-md-9">  
        <h2>{% localized "ProductsStrings|PageTitleProducts" %}</h2>  
    </div>  
</div>  
<div class="row">  
    <div class="col-md-12">  
    {% if products.size > 0 %}  
    <ul class="list-unstyled">  
    {% for product in products %}  
        <li>  
            <h3><a href="/products/{{product.id}}">{{product.title}}</a></h3>  
            {{product.description}}  
        </li>     
    {% endfor %}  
    </ul>  
    <paging-control></paging-control>  
    {% else %}  
    {% localized "CommonResources|NoItemsToDisplay" %}  
    {% endif %}  
    </div>  
</div>  
```  
  
### <a name="controls"></a>Элементы управления  
 В шаблоне `Product list` можно использовать следующие [элементы управления страницы](api-management-page-controls.md).  
  
-   [Управление разбиением на страницы](api-management-page-controls.md#paging-control)  
  
-   [Поиск — контроль](api-management-page-controls.md#search-control)  
  
### <a name="data-model"></a>Модель данных  
  
|Свойство.|Type|Описание|  
|--------------|----------|-----------------|  
|Разбивка на страницы|Сущность [разбиения по страницам](api-management-template-data-model-reference.md#Paging).|Сведения о разбиении по страницам для коллекции продуктов.|  
|Фильтрация|Сущность [фильтрации](api-management-template-data-model-reference.md#Filtering).|Сведения о фильтрации для страницы со списком продуктов.|  
|Продукты|Коллекция сущностей [Продукт](api-management-template-data-model-reference.md#Product).|Все продукты, которые доступны для текущего пользователя.|  
  
### <a name="sample-template-data"></a>Пример данных шаблона  
  
```json  
{  
    "Paging": {  
        "Page": 1,  
        "PageSize": 10,  
        "TotalItemCount": 2,  
        "ShowAll": false,  
        "PageCount": 1  
    },  
    "Filtering": {  
        "Pattern": null,  
        "Placeholder": "Search products"  
    },  
    "Products": [  
        {  
            "Id": "56f9445ffaf7560049060001",  
            "Title": "Starter",  
            "Description": "Subscribers will be able to run 5 calls/minute up to a maximum of 100 calls/week.",  
            "Terms": "",  
            "ProductState": 1,  
            "AllowMultipleSubscriptions": false,  
            "MultipleSubscriptionsCount": 1  
        },  
        {  
            "Id": "56f9445ffaf7560049060002",  
            "Title": "Unlimited",  
            "Description": "Subscribers have completely unlimited access to the API. Administrator approval is required.",  
            "Terms": null,  
            "ProductState": 1,  
            "AllowMultipleSubscriptions": false,  
            "MultipleSubscriptionsCount": 1  
        }  
    ]  
}  
```  
  
##  <a name="product"></a><a name="Product"></a> Продукта  
 Шаблон **продукта** позволяет настроить текст страницы со информацией о продукте на портале разработчика.  
  
 ![Страница продукта портала разработчика](./media/api-management-product-templates/APIM_ProductPage.png "APIM_ProductPage")  
  
### <a name="default-template"></a>Шаблон по умолчанию  
  
```xml  
<h2>{{Product.Title}}</h2>  
<p>{{Product.Description}}</p>  
  
{% assign replaceString0 = '{0}' %}  
  
{% if Limits and Limits.size > 0 %}  
<h3>{% localized "ProductDetailsStrings|WebProductsUsageLimitsHeader"%}</h3>  
<ul>  
  {% for limit in Limits %}  
  <li>{{limit.DisplayName}}</li>  
  {% endfor %}  
</ul>  
{% endif %}  
  
{% if apis.size > 0 %}  
<p>  
  <b>  
    {% if apis.size == 1 %}  
    {% capture apisCountText %}{% localized "ProductDetailsStrings|TextblockSingleApisCount" %}{% endcapture %}  
    {% else %}  
    {% capture apisCountText %}{% localized "ProductDetailsStrings|TextblockMultipleApisCount" %}{% endcapture %}  
    {% endif %}  
  
    {% capture apisCount %}{{apis.size}}{% endcapture %}  
    {{ apisCountText | replace : replaceString0, apisCount }}  
  </b>  
</p>  
  
<ul>  
  {% for api in Apis %}  
  <li>  
    <a href="/docs/services/{{api.Id}}">{{api.Name}}</a>  
  </li>  
  {% endfor %}  
</ul>  
{% endif %}  
  
{% if subscriptions.size > 0 %}  
<p>  
  <b>  
    {% if subscriptions.size == 1 %}  
    {% capture subscriptionsCountText %}{% localized "ProductDetailsStrings|TextblockSingleSubscriptionsCount" %}{% endcapture %}  
    {% else %}  
    {% capture subscriptionsCountText %}{% localized "ProductDetailsStrings|TextblockMultipleSubscriptionsCount" %}{% endcapture %}  
    {% endif %}  
  
    {% capture subscriptionsCount %}{{subscriptions.size}}{% endcapture %}  
    {{ subscriptionsCountText | replace : replaceString0, subscriptionsCount }}  
  </b>  
</p>  
  
<ul>  
  {% for subscription in subscriptions %}  
  <li>  
    <a href="/developer#{{subscription.Id}}">{{subscription.DisplayName}}</a>  
  </li>  
  {% endfor %}  
</ul>  
{% endif %}  
{% if CannotAddBecauseSubscriptionNumberLimitReached %}  
<b>{% localized "ProductDetailsStrings|TextblockSubscriptionLimitReached" %}</b>  
{% elsif CannotAddBecauseMultipleSubscriptionsNotAllowed == false %}  
<subscribe-button></subscribe-button>  
{% endif %}  
```  
  
### <a name="controls"></a>Элементы управления  
 В шаблоне `Product list` можно использовать следующие [элементы управления страницы](api-management-page-controls.md).  
  
-   [кнопка подписки](api-management-page-controls.md#subscribe-button)  
  
### <a name="data-model"></a>Модель данных  
  
|Свойство.|Type|Описание|  
|--------------|----------|-----------------|  
|Продукт|[Продукт](api-management-template-data-model-reference.md#Product)|Выбранный продукт.|  
|IsDeveloperSubscribed|Логическое|Указывает, подписан ли текущий пользователь на этот продукт.|  
|SubscriptionState|Число|Состояние подписки. Возможны следующие состояния.<br /><br /> —-   `0 - suspended`: подписка заблокирована, и подписчик не может вызвать ни один API продукта.<br />— -   `1 - active`: подписка активна.<br />— -   `2 - expired`: срок действия подписки истек, и она была деактивирована.<br />— -   `3 - submitted`: запрос разработчика на подписку выполнен, но еще не был утвержден или отклонен.<br />—-   `4 - rejected`: администратор отклонил запрос на подписку.<br />— -   `5 - cancelled`: подписка была отменена разработчиком или администратором.|  
|Ограничения|array|Это свойство является устаревшим и не должно использоваться.|  
|DelegatedSubscriptionEnabled|Логическое|Указывает, включено ли [делегирование](./api-management-howto-setup-delegation.md) для этой подписки.|  
|DelegatedSubscriptionUrl|строка|Если делегирование включено, содержит URL-адрес делегированной подписки.|  
|IsAgreed|Логическое|Указывает, принял ли текущий пользователь условия использования продукта, если они определены.|  
|Подписки|Коллекция сущностей [Сводка по подписке](api-management-template-data-model-reference.md#SubscriptionSummary).|Подписки на продукт.|  
|Apis|коллекция сущностей [API](api-management-template-data-model-reference.md#API)|API-интерфейсы, существующие для этого продукта.|  
|CannotAddBecauseSubscriptionNumberLimitReached|Логическое|Определяет, имеет ли текущий пользователь право подписаться на этот продукт в контексте лимита подписки.|  
|CannotAddBecauseMultipleSubscriptionsNotAllowed|Логическое|Определяет, имеет ли текущий пользователь право подписаться на этот продукт в контексте допустимости нескольких подписок.|  
  
### <a name="sample-template-data"></a>Пример данных шаблона  
  
```json  
{  
    "Product": {  
        "Id": "56f9445ffaf7560049060001",  
        "Title": "Starter",  
        "Description": "Subscribers will be able to run 5 calls/minute up to a maximum of 100 calls/week.",  
        "Terms": "",  
        "ProductState": 1,  
        "AllowMultipleSubscriptions": false,  
        "MultipleSubscriptionsCount": 1  
    },  
    "IsDeveloperSubscribed": true,  
    "SubscriptionState": 1,  
    "Limits": [],  
    "DelegatedSubscriptionEnabled": false,  
    "DelegatedSubscriptionUrl": null,  
    "IsAgreed": false,  
    "Subscriptions": [  
        {  
            "Id": "56f9445ffaf7560049070001",  
            "DisplayName": "Starter  (default)"  
        }  
    ],  
    "Apis": [  
        {  
            "id": "56f9445ffaf7560049040001",  
            "name": "Echo API",  
            "description": null,  
            "serviceUrl": "http://echoapi.cloudapp.net/api",  
            "path": "echo",  
            "protocols": [  
                2  
            ],  
            "authenticationSettings": null,  
            "subscriptionKeyParameterNames": null  
        }  
    ],  
    "CannotAddBecauseSubscriptionNumberLimitReached": false,  
    "CannotAddBecauseMultipleSubscriptionsNotAllowed": true  
}  
```

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о работе с шаблонами см. в статье [Настройка портала разработчика в службе управления API Azure с помощью шаблонов](api-management-developer-portal-templates.md).
