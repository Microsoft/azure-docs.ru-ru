---
title: включить файл
description: включить файл
ms.topic: include
ms.custom: include file
services: time-series-insights
ms.service: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: cshankar
ms.date: 10/02/2020
ms.openlocfilehash: 22411e5a80f555a3ead05d39466a7a175923d9bc
ms.sourcegitcommit: ed7376d919a66edcba3566efdee4bc3351c57eda
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105105191"
---
* После выбора нужной платформы на шаге 4 [настройки параметров платформы](../articles/active-directory/develop/quickstart-register-app.md#configure-platform-settings) настройте **URI перенаправления** и **маркеры доступа** на боковой панели справа от пользовательского интерфейса.

    * **URI перенаправления** должны соответствовать адресу, указанному в запросе аутентификации.

        * Для приложений, размещенных в локальной среде разработки, выберите **Public client (mobile & desktop)** (Общедоступный клиент (мобильный и классический)). Не забудьте задать для **общедоступного клиента** значение **Да**.
        * Для одностраничных приложений, размещенных в Службе приложений Azure, выберите **Интернет**.

    * Определите, подходит ли **URL-адрес выхода**.

    * Включите поток неявного предоставления разрешения, проверив **маркеры доступа** или **токены идентификатора**.

    [![Создание URI перенаправления](media/time-series-insights-aad-registration/active-directory-auth-redirect-uri.png)](media/time-series-insights-aad-registration/active-directory-auth-redirect-uri.png#lightbox)

    Щелкните **Настроить**, а затем **Сохранить**.

* Свяжите свое Azure Active Directory приложение Azure Time Series Insights. Последовательно выберите **Разрешения API** > **Add a permission (Добавить разрешение)**  > **Интерфейсы API, используемые моей организацией**.

    [![Связывание API с приложением Azure Active Directory](media/time-series-insights-aad-registration/active-directory-app-api-permission.png)](media/time-series-insights-aad-registration/active-directory-app-api-permission.png#lightbox)

   Введите `Azure Time Series Insights` в строке поиска, а затем выберите `Azure Time Series Insights`.

* Затем укажите тип разрешения API, необходимый для приложения. По умолчанию будет выделен тип **Делегированные разрешения**. Выберите тип разрешения и щелкните **Добавить разрешения**.

    [![Выбор типа разрешения API, необходимого для приложения](media/time-series-insights-aad-registration/active-directory-app-permission-grant.png)](media/time-series-insights-aad-registration/active-directory-app-permission-grant.png#lightbox)

* [Добавьте учетные данные](../articles/active-directory/develop/quickstart-register-app.md#add-credentials) , если приложение будет вызывать интерфейсы API среды самостоятельно. Учетные данные позволяют приложению проходить самостоятельную проверку подлинности, не требуя взаимодействия с пользователем во время выполнения.