---
title: Отправка безопасных push-уведомлений в центрах уведомлений Azure
description: Узнайте, как отправлять безопасные push-уведомления в приложение Android из Azure. Примеры кода написаны на Java и C#.
documentationcenter: android
keywords: push-уведомление, push-уведомления, push-сообщения, push-уведомления android
author: sethmanheim
manager: femila
services: notification-hubs
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: android
ms.devlang: java
ms.topic: article
ms.date: 08/07/2020
ms.author: sethm
ms.reviewer: thsomasu
ms.lastreviewed: 01/04/2019
ms.openlocfilehash: f2d5d618fabbe7400ce825f984ace1622a524f05
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88004020"
---
# <a name="send-secure-push-notifications-with-azure-notification-hubs"></a>Отправка безопасных push-уведомлений в центрах уведомлений Azure

> [!div class="op_single_selector"]
> * [Windows Universal](notification-hubs-aspnet-backend-windows-dotnet-wns-secure-push-notification.md)
> * [iOS](notification-hubs-aspnet-backend-ios-push-apple-apns-secure-notification.md)
> * [Android](notification-hubs-aspnet-backend-android-secure-google-gcm-push-notification.md)

## <a name="overview"></a>Обзор

> [!IMPORTANT]
> Для работы с этим учебником необходима активная учетная запись Azure. Если ее нет, можно создать бесплатную пробную учетную запись всего за несколько минут. Дополнительные сведения см. в разделе [Бесплатная пробная версия Azure](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A643EE910&amp;returnurl=http%3A%2F%2Fazure.microsoft.com%2Fen-us%2Fdocumentation%2Farticles%2Fpartner-xamarin-notification-hubs-ios-get-started).

Поддержка push-уведомлений в Microsoft Azure позволяет получить доступ к простой в использовании мультиплатформенной масштабируемой инфраструктуре для обмена push-сообщениями, которая значительно упрощает реализацию push-уведомлений для пользовательских и корпоративных приложений для мобильных платформ.

Из-за ограничений, связанных с правовыми нормами или обеспечением безопасности, иногда в уведомлении могут присутствовать данные, которые нельзя передать через стандартную инфраструктуру push-уведомлений. В руководстве описывается, как реализовать этот принцип при отправке конфиденциальной информации через защищенное соединение с проверкой подлинности между клиентским устройством Android и серверной частью приложения.

На высоком уровне поток можно представить следующим образом.

- Серверная часть приложения:
  * Сохраняет полезную нагрузку в базе данных серверной части.
  * Отправляет идентификатор этого уведомления на устройство Android (защищаемые сведения не передаются).
- Приложение на устройстве при получении уведомления:
  * Устройство Android связывается с серверной частью и запрашивает полезные данные.
  * Приложение может показывать полезную нагрузку в виде уведомления на устройстве.

Стоит отметить, что в предыдущей процедуре (и в этом руководстве) мы предположили, что устройство сохраняет маркер аутентификации в локальном хранилище после входа пользователя. Такой подход гарантирует удобство работы, так как устройство может получать защищенные полезные данные уведомления с помощью этого маркера. Если приложение не сохраняет маркеры проверки подлинности на устройстве или если срок действия этих токенов истек, приложение устройства при получении push-уведомления должно отображать общее уведомление, предлагающее пользователю запустить приложение. Затем приложение выполняет проверку подлинности пользователя и отображает полезную нагрузку уведомления.

В этом руководстве описывается, как отправлять безопасные push-уведомления. Он основан на руководстве по [уведомлению пользователей](notification-hubs-aspnet-backend-gcm-android-push-to-user-google-notification.md) , поэтому сначала необходимо выполнить действия, описанные в этом руководстве.

> [!NOTE]
> В этом руководстве предполагается, что вы создали и настроили центр уведомлений, как описано в разделе [Приступая к работе с центрами уведомлений (Android)](notification-hubs-android-push-notification-google-gcm-get-started.md).

[!INCLUDE [notification-hubs-aspnet-backend-securepush](../../includes/notification-hubs-aspnet-backend-securepush.md)]

## <a name="modify-the-android-project"></a>Внесение изменений в проект Android

Теперь, когда вы изменили серверную часть приложения для отправки только идентификатора push-уведомления, необходимо изменить приложение Android, чтобы оно обрабатывало это уведомление, и выполнить обратный вызов к серверной части, чтобы получить отображаемое защищенное сообщение.
Для этого убедитесь, что ваше приложение Android может пройти проверку подлинности на вашем сервере при получении push-уведомлений.

Теперь измените процесс входа для того, чтобы сохранить значение заголовка аутентификации в общих настройках приложения. Аналогичные механизмы можно использовать для хранения любых маркеров аутентификации (например, маркеров OAuth), которые потребуются приложению без необходимости ввода учетных данных пользователя.

1. В проекте приложения Android добавьте следующие константы в начало класса `MainActivity`.

    ```java
    public static final String NOTIFY_USERS_PROPERTIES = "NotifyUsersProperties";
    public static final String AUTHORIZATION_HEADER_PROPERTY = "AuthorizationHeader";
    ```

2. Там же в классе `MainActivity` добавьте в метод `getAuthorizationHeader()` следующий код:

    ```java
    private String getAuthorizationHeader() throws UnsupportedEncodingException {
        EditText username = (EditText) findViewById(R.id.usernameText);
        EditText password = (EditText) findViewById(R.id.passwordText);
        String basicAuthHeader = username.getText().toString()+":"+password.getText().toString();
        basicAuthHeader = Base64.encodeToString(basicAuthHeader.getBytes("UTF-8"), Base64.NO_WRAP);

        SharedPreferences sp = getSharedPreferences(NOTIFY_USERS_PROPERTIES, Context.MODE_PRIVATE);
        sp.edit().putString(AUTHORIZATION_HEADER_PROPERTY, basicAuthHeader).commit();

        return basicAuthHeader;
    }
    ```

3. Добавьте следующие инструкции `import` в начало файла `MainActivity`.

    ```java
    import android.content.SharedPreferences;
    ```

Теперь измените обработчик, который вызывается при получении уведомления.

1. В классе `MyHandler` добавьте в метод `OnReceive()` следующий код:

    ```java
    public void onReceive(Context context, Bundle bundle) {
        ctx = context;
        String secureMessageId = bundle.getString("secureId");
        retrieveNotification(secureMessageId);
    }
    ```

2. Затем добавьте метод `retrieveNotification()`, заменив заполнитель `{back-end endpoint}` на конечную точку серверной части, полученную во время развертывания серверной части:

    ```java
    private void retrieveNotification(final String secureMessageId) {
        SharedPreferences sp = ctx.getSharedPreferences(MainActivity.NOTIFY_USERS_PROPERTIES, Context.MODE_PRIVATE);
        final String authorizationHeader = sp.getString(MainActivity.AUTHORIZATION_HEADER_PROPERTY, null);

        new AsyncTask<Object, Object, Object>() {
            @Override
            protected Object doInBackground(Object... params) {
                try {
                    HttpUriRequest request = new HttpGet("{back-end endpoint}/api/notifications/"+secureMessageId);
                    request.addHeader("Authorization", "Basic "+authorizationHeader);
                    HttpResponse response = new DefaultHttpClient().execute(request);
                    if (response.getStatusLine().getStatusCode() != HttpStatus.SC_OK) {
                        Log.e("MainActivity", "Error retrieving secure notification" + response.getStatusLine().getStatusCode());
                        throw new RuntimeException("Error retrieving secure notification");
                    }
                    String secureNotificationJSON = EntityUtils.toString(response.getEntity());
                    JSONObject secureNotification = new JSONObject(secureNotificationJSON);
                    sendNotification(secureNotification.getString("Payload"));
                } catch (Exception e) {
                    Log.e("MainActivity", "Failed to retrieve secure notification - " + e.getMessage());
                    return e;
                }
                return null;
            }
        }.execute(null, null, null);
    }
    ```

Этот метод вызывает серверную часть вашего приложения для извлечения содержимого уведомления с использованием учетных данных, хранящихся в общих настройках, и отображения в виде обычного уведомления. Для пользователя приложения уведомление ничем не отличается от других уведомлений.

Случаи отсутствующего свойства заголовка аутентификации или отклонения предпочтительнее обрабатывать в серверной части. Обработка таких случаев в основном зависит от взаимодействия с пользователем на целевой платформе. Одним из вариантов является отображение уведомления с обычным предложением пользователю для проверки подлинности, чтобы получить текущее уведомление.

## <a name="run-the-application"></a>Запуск приложения

Для запуска приложения выполните следующие действия:

1. Убедитесь, что **AppBackend** развернут в Azure. При использовании Visual Studio, запустите приложение веб-API **AppBackend** . Отобразится веб-страница ASP.NET.
2. В Eclipse запустите приложение на физическом устройстве Android или на эмуляторе.
3. В пользовательском интерфейсе приложения Android введите имя пользователя и пароль. Это могут быть любые совпадающие строки.
4. В пользовательском интерфейсе приложения Android нажмите **Вход**. Затем нажмите **Отправить push-уведомление**.
