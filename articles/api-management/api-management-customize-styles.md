---
title: Настройка стиля страницы на портале разработчика в службе "Управление API"
titleSuffix: Azure API Management
description: Следуйте инструкциям из этого руководства, чтобы настроить стиль элементов на портале разработчика в службе управления API Azure.
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.custom: mvc
ms.topic: article
ms.date: 11/04/2019
ms.author: apimpm
ms.openlocfilehash: 0cdad245981ad02d3cdaefba447d131c775e242f
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "93145775"
---
# <a name="customize-the-style-of-the-developer-portal-pages"></a>Настройка стиля страниц для портала разработчика

Есть три распространенных способа настроить портал разработчика в службе управления API Azure:
 
* [Изменение содержимого статических страниц и элементов макета страницы.](api-management-modify-content-layout.md)
* Обновление стилей, которые используются для элементов страницы на портале разработчика (как описано в этом руководстве).
* [Изменение шаблонов, используемых для страниц, созданных порталом](api-management-developer-portal-templates.md) (например, документы по API, продукты, аутентификация пользователей и т. д.).

Из этой статьи вы узнаете, как настроить стиль элементов на страницах классического **портала разработчика** и просмотреть изменения.

![Снимок экрана, на котором показана область для изменения параметров на классическим портале разработчика.](./media/modify-developer-portal-style/developer_portal.png)

[!INCLUDE [api-management-portal-legacy.md](../../includes/api-management-portal-legacy.md)]

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="prerequisites"></a>предварительные требования

+ Ознакомьтесь с [терминологией службы управления API в Azure](api-management-terminology.md).
+ Выполните задачи в кратком руководстве по [созданию экземпляра службы управления API Azure](get-started-create-service-instance.md).
+ Также выполните задачи из руководства по [импорту и публикации первого API](import-and-publish.md).

## <a name="customize-the-developer-portal"></a>Настройка портала разработчика

1. Щелкните **Обзор**.
2. Нажмите кнопку **Портал разработчика (классический)** в верхней части окна **Обзор**.
3. В левом верхнем углу экрана вы увидите значок, которых состоит из двух кистей. Наведите на него указатель, чтобы открыть меню настройки портала.

    ![Снимок экрана, на котором изображен значок двух кистей.](./media/modify-developer-portal-style/modify-developer-portal-style01.png)
4. В меню выберите **Стили**, чтобы открыть панель настройки стилей.

    На странице отобразятся все элементы, которые можно настроить на панели **Стили**.
5. В поле **Change variable values to customize developer portal appearance** (Изменить значения переменных, чтобы настроить вид портала разработчика) введите "headings-color".

    На странице появится элемент **\@headings-color**. Эта переменная определяет цвет текста.

    ![Настройка стиля](./media/modify-developer-portal-style/modify-developer-portal-style02.png)
    
6. Щелкните поле для переменной **\@headings-color**. 
    
    Откроется раскрывающееся меню палитры.
7. В этом меню выберите новый цвет.

    > [!TIP]
    > Предварительный просмотр в реальном времени доступен для всех изменений. Индикатор хода выполнения отображается в верхней части панели настройки. Через несколько секунд цвет текста заголовка изменится на только что выбранный.

8. В меню панели настройки нажмите кнопку **Опубликовать** в нижнем левом углу.
9. Нажмите кнопку **Опубликовать настройки**, чтобы изменения стали доступны для всех.

## <a name="view-your-change"></a>просмотр изменений.

1. Перейдите на портал разработчика.
2. Вы увидите внесенные изменения.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в статье [Настройка портала разработчика в службе управления API Azure с помощью шаблонов](api-management-developer-portal-templates.md).