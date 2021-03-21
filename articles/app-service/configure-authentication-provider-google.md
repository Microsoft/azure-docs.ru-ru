---
title: Настройка проверки подлинности Google
description: Узнайте, как настроить аутентификацию Google в качестве поставщика удостоверений для службы приложений или приложения функций Azure.
ms.assetid: 2b2f9abf-9120-4aac-ac5b-4a268d9b6e2b
ms.topic: article
ms.date: 09/02/2019
ms.custom:
- seodec18
- fasttrack-edit
ms.openlocfilehash: e8a9fbe6072f3628d755ad3ad5aa5a623fc3ab23
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "80519947"
---
# <a name="configure-your-app-service-or-azure-functions-app-to-use-google-login"></a>Настройка службы приложений или приложения "функции Azure" для использования входа Google

[!INCLUDE [app-service-mobile-selector-authentication](../../includes/app-service-mobile-selector-authentication.md)]

В этом разделе показано, как настроить службу приложений Azure или функцию Azure для использования Google в качестве поставщика проверки подлинности.

Чтобы выполнить процедуру, описанную в этом разделе, необходимо иметь учетную запись Google с проверенным адресом электронной почты. Чтобы создать учетную запись Google, перейдите по ссылке [accounts.google.com](https://go.microsoft.com/fwlink/p/?LinkId=268302).

## <a name="register-your-application-with-google"></a><a name="register"> </a>Регистрация приложения с помощью Google

1. Для создания идентификатора клиента и секрета клиента следуйте инструкциям по Google [Sign-In для приложений на стороне сервера](https://developers.google.com/identity/sign-in/web/server-side-flow) . Нет необходимости вносить изменения в код. Просто используйте следующую информацию:
    - Для **полномочных источников JavaScript** используйте `https://<app-name>.azurewebsites.net` с именем приложения в *\<app-name>* .
    - Для **разрешения URI перенаправления** используйте `https://<app-name>.azurewebsites.net/.auth/login/google/callback` .
1. Скопируйте идентификатор приложения и значения секрета приложения.

    > [!IMPORTANT]
    > Секрет приложения — это важные учетные данные безопасности. Не сообщайте этот секрет никому и не раскрывайте его в клиентском приложении.

## <a name="add-google-information-to-your-application"></a><a name="secrets"> </a>Добавление данных Google в приложение

1. В [портал Azure]перейдите к приложению службы приложений.
1. Последовательно выберите **Параметры** > **Аутентификация или авторизация** и установите для параметра **Проверка подлинности Службы приложений** значение **Вкл**.
1. Выберите **Google**, а затем вставьте значения в поле идентификатор приложения и секрет приложения, полученные ранее. Включите все области, необходимые для приложения.
1. Нажмите кнопку **ОК**.

   Служба приложений обеспечивает проверку подлинности, но не ограничивает разрешенный доступ к содержимому и API-интерфейсам сайта. Дополнительные сведения см. в разделе [Авторизация или отказ пользователей](app-service-authentication-how-to.md#authorize-or-deny-users).

1. Используемых Чтобы ограничить доступ к сайту только для пользователей, прошедших проверку подлинности в Google, задайте **действие, которое будет выполняться, если запрос не прошел проверку подлинности** в **Google**. При установке этой функции приложение требует, чтобы все запросы прошли проверку подлинности. Он также перенаправляет все запросы без проверки подлинности в Google для проверки подлинности.

    > [!CAUTION]
    > Таким образом, ограниченный доступ применяется ко всем вызовам приложения, что может быть нежелательно для приложений, у которых есть общедоступная домашняя страница (как во многих одностраничных приложениях). Для таких приложений, возможно, стоит установить параметр **Разрешить анонимные запросы (нет действия)** , чтобы приложение самостоятельно обрабатывало проверку подлинности. Дополнительные сведения см. в разделе [Поток проверки подлинности](overview-authentication-authorization.md#authentication-flow).

1. Щелкните **Сохранить**.

Теперь вы готовы использовать Google для проверки подлинности в приложении.

## <a name="next-steps"></a><a name="related-content"> </a>Дальнейшие действия

[!INCLUDE [app-service-mobile-related-content-get-started-users](../../includes/app-service-mobile-related-content-get-started-users.md)]

<!-- Anchors. -->

<!-- Images. -->

[0]: ./media/app-service-mobile-how-to-configure-google-authentication/mobile-app-google-redirect.png
[1]: ./media/app-service-mobile-how-to-configure-google-authentication/mobile-app-google-settings.png

<!-- URLs. -->

[Google apis]: https://go.microsoft.com/fwlink/p/?LinkId=268303

[Портал Azure]: https://portal.azure.com/

