---
title: Настройка портала разработчика для управления API с помощью шаблонов
titleSuffix: Azure API Management
description: Получите дополнительные сведения о настройке портала разработчика в службе управления API Azure с помощью шаблонов
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: a195675b-f7d0-4fc9-90bf-860e6f17ccf7
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 11/04/2019
ms.author: apimpm
ms.openlocfilehash: 16c0b3e6dc4bd9097312e3a8c43618e66781ef85
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92092891"
---
# <a name="how-to-customize-the-azure-api-management-developer-portal-using-templates"></a>Настройка портала разработчика в службе управления API Azure с помощью шаблонов

Существуют три основных способа настройки портала разработчика в службе управления Azure API.

* [Изменение содержимого статических страниц и элементов макета страницы.][modify-content-layout]
* [Обновление стилей, которые используются для элементов страницы на портале разработчика][customize-styles].
* [Изменение шаблонов, используемых для страниц, созданных порталом][portal-templates] (описывается в этом руководстве).

Шаблоны используются для настройки содержимого страниц портала разработчика, создаваемых системой (например, документы API, продукты, аутентификация пользователей и т. д.). С помощью синтаксиса [DotLiquid](http://dotliquidmarkup.org/) и указанного набора локализованных строковых ресурсов, значков и элементов управления на странице можно гибко настраивать содержимое страниц по своему усмотрению.

[!INCLUDE [api-management-portal-legacy.md](../../includes/api-management-portal-legacy.md)]

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="developer-portal-templates-overview"></a>Обзор шаблонов портала разработчика

Чтобы изменить шаблоны, войдите на **портал разработчика** в качестве администратора. Чтобы начать сначала, откройте портал Azure и щелкните **портал разработчика** на панели инструментов экземпляра службы управления API.

Чтобы открыть шаблоны портала разработчиков, щелкните значок настройки слева, чтобы открылось меню настройки, а затем **Шаблоны**.

![Снимок экрана, на котором показан значок настройки для открытия меню настройки.][api-management-customize-menu]

В списке шаблонов отображается несколько категорий шаблонов, охватывающих различные страницы портала разработчика. Все шаблоны отличаются друг от друга, но действия, которые нужно выполнить для их изменения и публикации, одинаковы. Чтобы изменить шаблон, щелкните имя шаблона.

![Шаблоны портала разработчика][api-management-templates-menu]

Щелкнув шаблон, можно перейти на страницу портала разработчика, которая настраивается с помощью этого шаблона. В этом примере отображается шаблон **Список продуктов** . Шаблон **Список продуктов** управляет областью экрана, обозначенной красным прямоугольником.

![Шаблон "Список продуктов"][api-management-developer-portal-templates-overview]

Некоторые шаблоны, например шаблоны **Профиля пользователя**, настраивают другие области на той же странице.

![Шаблоны "Профиль пользователя"][api-management-user-profile-templates]

Редактор для каждого шаблона портала разработчика состоит из двух разделов, отображаемых в нижней части страницы. Слева отображается область редактирования шаблона, а справа — модель данных для шаблона.

Область редактирования шаблона содержит разметку, которая определяет внешний вид и поведение соответствующей страницы на портале разработчика. В разметке шаблона используется синтаксис [DotLiquid](http://dotliquidmarkup.org/) . Один из популярных редакторов для DotLiquid — [DotLiquid для конструкторов](https://github.com/dotliquid/dotliquid/wiki/DotLiquid-for-Designers). Любые изменения, внесенные в шаблон во время редактирования, отображаются в браузере в режиме реального времени, но не будут видны клиентам, пока вы не [сохраните](#to-save-a-template) и не [опубликуете](#to-publish-a-template) шаблон.

![Разметка шаблона][api-management-template]

В области **Данные шаблона** содержится руководство по модели данных для сущностей, которые доступны для использования в данном шаблоне. Это руководство предоставляется путем вывода реальных данных, отображаемых в данный момент на портале разработчика. Панели шаблонов можно развернуть, щелкнув прямоугольник в правом верхнем углу области **Данные шаблона** .

![Модель данных шаблона][api-management-template-data]

В предыдущем примере на портале разработчика отображаются два продукта, которые были получены из данных, отображаемых в области **данные шаблона** , как показано в следующем примере:

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
            "Id": "56ec64c380ed850042060001",
            "Title": "Starter",
            "Description": "Subscribers will be able to run 5 calls/minute up to a maximum of 100 calls/week.",
            "Terms": "",
            "ProductState": 1,
            "AllowMultipleSubscriptions": false,
            "MultipleSubscriptionsCount": 1
        },
        {
            "Id": "56ec64c380ed850042060002",
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

Для получения необходимых выходных данных разметка в шаблоне **Список продуктов** обрабатывает данные путем итерации по коллекции продуктов, отображая сведения о каждом отдельном продукте и ссылку на этот продукт. Обратите внимание на элементы `<search-control>` и `<page-control>` в разметке. С их помощью можно контролировать отображение элементов управления поиском и разбивкой на страницы. `ProductsStrings|PageTitleProducts` — это ссылка на локализованную строку, которая содержит текст заголовка `h2` для страницы. Список строковых ресурсов, элементов управления страницы и значков, которые доступны для использования в шаблонах портала разработчиков, см. в статье [Azure API Management Templates](api-management-developer-portal-templates-reference.md) (Шаблоны в службе управления API).

```html
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

## <a name="to-save-a-template"></a>Сохранение шаблона
Чтобы сохранить шаблон, нажмите кнопку "Сохранить" в редакторе шаблона.

![Сохранение шаблона][api-management-save-template]

Сохраненные изменения не отобразятся на портале разработчика, пока шаблон не будет опубликован.

## <a name="to-publish-a-template"></a>Публикация шаблона
Сохраненные шаблоны могут быть опубликованы по отдельности или все вместе. Чтобы опубликовать отдельный шаблон, нажмите кнопку "Опубликовать" в редакторе шаблона.

![Публикация шаблона][api-management-publish-template]

Щелкните **Да** для подтверждения публикации шаблона, после которой он станет доступным на портале разработчика.

![Снимок экрана, на котором показано, где вы наберете "Да", чтобы сделать шаблон активным.][api-management-publish-template-confirm]

Чтобы опубликовать все неопубликованные версии шаблонов, нажмите **Опубликовать** в списке шаблонов. Неопубликованные шаблоны обозначены звездочкой рядом с именем шаблона. В этом примере публикуются шаблоны **Список продуктов** и **Продукт**.

![Публикация шаблонов][api-management-publish-templates]

Щелкните **Опубликовать настройки** для подтверждения.

![Подтверждение публикации][api-management-publish-customizations]

Недавно опубликованные шаблоны немедленно вступают в силу на портале разработчика.

## <a name="to-revert-a-template-to-the-previous-version"></a>Возврат к предыдущей версии шаблона
Чтобы вернуться к предыдущей опубликованной версии шаблона, нажмите кнопку "Отменить изменения" в редакторе шаблонов.

![Снимок экрана, на котором показан значок, используемый для отмены шаблона.][api-management-revert-template]

Нажмите кнопку **Да** для подтверждения.

![Снимок экрана, на котором показано, где выбрать Да для подтверждения изменений.][api-management-revert-template-confirm]

Ранее опубликованная версия шаблона станет активной на портале разработчика после завершения операции восстановления.

## <a name="to-restore-a-template-to-the-default-version"></a>Восстановление шаблона до версии по умолчанию
Восстановление шаблонов до версии по умолчанию выполняется в два этапа. Сначала необходимо восстановить шаблоны, а затем опубликовать восстановленные версии.

Чтобы восстановить одиночный шаблон до версии по умолчанию, нажмите кнопку "Восстановить" в редакторе шаблона.

![Отмена изменений шаблона][api-management-reset-template]

Нажмите кнопку **Да** для подтверждения.

![Подтвердить][api-management-reset-template-confirm]

Чтобы восстановить все шаблоны до версии по умолчанию, нажмите кнопку **Восстановить шаблоны по умолчанию** в списке шаблонов.

![Восстановление шаблонов][api-management-restore-templates]

Затем необходимо опубликовать восстановленные шаблоны вместе или по отдельности, выполнив действия, описанные в разделе [Публикация шаблона](#to-publish-a-template).

## <a name="next-steps"></a>Дальнейшие действия
Справочную информацию о шаблонах портала разработчика, строковых ресурсах, значках и элементах управления страницы см. в разделе [Azure API Management Templates](api-management-developer-portal-templates-reference.md) (Шаблоны в службе управления API).

[modify-content-layout]: api-management-modify-content-layout.md
[customize-styles]: api-management-customize-styles.md
[portal-templates]: api-management-developer-portal-templates.md

[api-management-customize-menu]: ./media/api-management-developer-portal-templates/api-management-customize-menu.png
[api-management-templates-menu]: ./media/api-management-developer-portal-templates/api-management-templates-menu.png
[api-management-developer-portal-templates-overview]: ./media/api-management-developer-portal-templates/api-management-developer-portal-templates-overview.png
[api-management-template]: ./media/api-management-developer-portal-templates/api-management-template.png
[api-management-template-data]: ./media/api-management-developer-portal-templates/api-management-template-data.png
[api-management-developer-portal-menu]: ./media/api-management-developer-portal-templates/api-management-developer-portal-menu.png
[api-management-management-console]: ./media/api-management-developer-portal-templates/api-management-management-console.png
[api-management-browse]: ./media/api-management-developer-portal-templates/api-management-browse.png
[api-management-user-profile-templates]: ./media/api-management-developer-portal-templates/api-management-user-profile-templates.png
[api-management-save-template]: ./media/api-management-developer-portal-templates/api-management-save-template.png
[api-management-publish-template]: ./media/api-management-developer-portal-templates/api-management-publish-template.png
[api-management-publish-template-confirm]: ./media/api-management-developer-portal-templates/api-management-publish-template-confirm.png
[api-management-publish-templates]: ./media/api-management-developer-portal-templates/api-management-publish-templates.png
[api-management-publish-customizations]: ./media/api-management-developer-portal-templates/api-management-publish-customizations.png
[api-management-revert-template]: ./media/api-management-developer-portal-templates/api-management-revert-template.png
[api-management-revert-template-confirm]: ./media/api-management-developer-portal-templates/api-management-revert-template-confirm.png
[api-management-reset-template]: ./media/api-management-developer-portal-templates/api-management-reset-template.png
[api-management-reset-template-confirm]: ./media/api-management-developer-portal-templates/api-management-reset-template-confirm.png
[api-management-restore-templates]: ./media/api-management-developer-portal-templates/api-management-restore-templates.png
