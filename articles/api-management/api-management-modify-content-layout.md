---
title: Изменение содержимого страницы на портале разработчика в управлении API
titleSuffix: Azure API Management
description: Узнайте, как изменить содержимое страниц на портале разработчика в службе управления API Azure.
services: api-management
documentationcenter: ''
author: vlvinogr
manager: vlvinogr
editor: ''
ms.assetid: 186128fe-41c0-4efb-9efe-2478ad4d103f
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 02/09/2017
ms.author: vlvinogr
ms.openlocfilehash: ebf2cbd430339378a09d10d91ad61327d24842e4
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "75430633"
---
# <a name="modify-the-content-and-layout-of-pages-on-the-developer-portal-in-azure-api-management"></a>Изменение содержимого и макета страниц на портале разработчика в службе управления API Azure
Существуют три основных способа настройки портала разработчика в службе управления Azure API.

* [Изменение содержимого статических страниц и элементов макета страницы][modify-content-layout] (описывается в этом руководстве).
* [Обновление стилей, которые используются для элементов страницы на портале разработчика][customize-styles].
* [Изменение шаблонов, используемых для страниц, созданных порталом][portal-templates] (например, документы по API, продукты, аутентификация пользователей и т. д.).

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="structure-of-developer-portal-pages"></a><a name="page-structure"> </a>Структура страниц портала разработчика

Портал разработчика основан на системе управления контентом. Макет каждой страницы состоит из набора небольших элементов страницы, называемых мини-приложениями.

![Структура страницы портала разработчика][api-management-customization-widget-structure]

Все мини-приложения можно изменять.
* Основное содержимое для каждой отдельной страницы находится в мини-приложении "Содержимое". Изменение страницы означает изменение содержимого этого мини-приложения.
* Все элементы макета страницы содержатся в остальных мини-приложениях. Изменения, внесенные в эти мини-приложения, применяются ко всем страницам. Они называются "мини-приложениями макета".

При повседневном редактировании страниц часто изменяется только содержимое мини-приложения "Содержимое": оно будет отличаться для каждой отдельной страницы.

## <a name="modifying-the-contents-of-a-layout-widget"></a><a name="modify-layout-widget"> </a>Изменение содержимого мини-приложения макета

Доступ к порталу разработчика можно получить на портале Azure.

1. Щелкните **Портал разработчика** на панели инструментов своего экземпляра службы управления API.
2. Чтобы изменить содержимое мини-приложений, щелкните значок двух кистей в меню, расположенном в левой части **портала разработчика**.
3. Чтобы изменить содержимое заголовка, прокрутите страницу до пункта **Заголовок** в списке слева.

    Мини-приложения редактируются с помощью полей.
4. Когда вы будете готовы опубликовать изменения, нажмите кнопку **публикации** в нижней части страницы.

Теперь новый заголовок будет отображаться на каждой странице портала разработчика.

## <a name="next-steps"></a><a name="next-steps"> </a>Дальнейшие действия
* [Обновление стилей, которые используются для элементов страницы на портале разработчика][customize-styles].
* [Изменение шаблонов, используемых для страниц, созданных порталом][portal-templates] (например, документы по API, продукты, аутентификация пользователей и т. д.).

[Structure of developer portal pages]: #page-structure
[Modifying the contents of a layout widget]: #modify-layout-widget
[Edit the contents of a page]: #edit-page-contents
[Next steps]: #next-steps

[modify-content-layout]: api-management-modify-content-layout.md
[customize-styles]: api-management-customize-styles.md
[portal-templates]: api-management-developer-portal-templates.md

[api-management-customization-widget-structure]: ./media/api-management-modify-content-layout/portal-widget-structure.png
[api-management-management-console]: ./media/api-management-modify-content-layout/api-management-management-console.png
[api-management-widgets-header]: ./media/api-management-modify-content-layout/api-management-widgets-header.png
[api-management-customization-manage-content]: ./media/api-management-modify-content-layout/api-management-customization-manage-content.png
