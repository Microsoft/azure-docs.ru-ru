---
title: Отправка push-уведомлений в Android с помощью Центров уведомлений Microsoft Azure и SDK Firebase версии 1.0.0-preview1
description: Из этого руководства вы узнаете, как с помощью Центров уведомлений Microsoft Azure и Google Firebase Cloud Messaging отправлять push-уведомления на устройства Android (с использованием пакета SDK Firebase версии 1.0.0, предварительная версия 1).
author: sethmanheim
ms.author: sethm
ms.date: 5/28/2020
ms.topic: tutorial
ms.service: notification-hubs
ms.reviewer: thsomasu
ms.lastreviewed: 05/27/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 091ada7f099aca17152fc806e61dce6ab979852e
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106055167"
---
# <a name="tutorial-send-push-notifications-to-android-devices-using-firebase-sdk-version-100-preview1"></a>Руководство по отправке push-уведомлений на устройства Android с помощью SDK Firebase версии 1.0.0-preview1

В этом руководстве показано, как использовать Центры уведомлений Microsoft Azure и обновленный SDK Firebase Cloud Messaging (FCM) версии 1.0.0-preview1 для отправки push-уведомлений в приложение на платформе Android. Следуя инструкциям этого руководства, вы создадите пустое приложение Android, которое получает push-уведомления с помощью Firebase Cloud Messaging (FCM).

Полный код для этого руководства вы можете скачать на сайте [GitHub](https://github.com/Azure/azure-notificationhubs-android/tree/v1-preview/notification-hubs-test-app-refresh).

В этом руководстве рассматриваются следующие действия:

- Создание проекта Android Studio.
- Создание проекта Firebase с поддержкой Firebase Cloud Messaging.
- Создание центра уведомлений.
- Подключение своего приложения к концентратору.
- Тестирование приложения.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим учебником необходима активная учетная запись Azure. Если ее нет, можно создать бесплатную пробную учетную запись всего за несколько минут. Дополнительные сведения см. в разделе [Бесплатная пробная версия Azure](https://azure.microsoft.com/free/).

Вам также понадобятся следующее.

- Рекомендуется последняя версия [Android Studio](https://go.microsoft.com/fwlink/?LinkId=389797).
- Минимальный уровень поддержки — API 19.

## <a name="create-an-android-studio-project"></a>Создание проекта Android Studio

Сначала нужно создать проект в Android Studio:

1. Запустите Android Studio.

2. В меню выберите **Файл**, потом **Новый**, а затем — **Новый проект**.

3. На странице **Choose your project** (Выбор проекта) выберите **Empty Activity** (Пустое действие) и щелкните **Далее**.

4. На странице **Настройка проекта** выполните следующее:
   1. Введите имя приложения.
   2. Укажите расположение для сохранения файлов проекта.
   3. Нажмите кнопку **Готово**.

   :::image type="content" source="media/android-sdk/configure-project.png" alt-text="Настройка проекта":::

## <a name="create-a-firebase-project-that-supports-fcm"></a>Создание проекта Firebase с поддержкой FCM

1. Войдите в [консоль Firebase](https://firebase.google.com/console/). Создайте проект Firebase, если его еще нет.

2. После создания проекта выберите **Add Firebase to your Android app** (Добавить Firebase в приложение Android).

   :::image type="content" source="media/android-sdk/get-started.png" alt-text="Добавление Firebase":::

3. Выполните следующие действия на странице **Добавление Firebase в приложение для Android**.

   1. Для параметра **Имя пакета Android** скопируйте свое значение **applicationId** в файл приложения **build.gradle**. В нашем примере он выглядит следующим образом: `com.fabrikam.fcmtutorial1app`.

      :::image type="content" source="media/android-sdk/specify-package-name-fcm-settings.png" alt-text="Указание имени пакета":::

   2. Выберите **Регистрация приложения**.

4. Выберите **Download google-services.json** (Загрузить google-services.json), сохраните файл в папку **приложения** проекта, а затем выберите **Далее**.

   :::image type="content" source="media/android-sdk/download-google-service-button.png" alt-text="Скачивание службы Google":::

5. В консоли Firebase щелкните значок шестеренки возле имени проекта. Выберите пункт **Project Settings** (Параметры проекта).

   :::image type="content" source="media/android-sdk/notification-hubs-firebase-console-project-settings.png" alt-text="Параметры проекта":::

6. Если вы еще не скачали файл **google-services.json** в папку **app** проекта Android Studio, это можно сделать на этой странице.

7. Переключитесь на вкладку **Обмен сообщениями в облаке**.

8. Скопируйте и сохраните **Ключ сервера** для последующего использования. Это значение используется для настройки имени центра.

## <a name="configure-a-notification-hub"></a>Настройка центра уведомлений

1. Войдите на [портал Azure](https://portal.azure.com/).

2. Выберите **Все службы** в меню слева и щелкните **Центры уведомлений** в разделе **Мобильный**. Щелкните значок звездочки рядом с именем службы, чтобы добавить ее в раздел **Избранное** в меню слева. После добавления **Центров уведомлений** в **Избранное** выберите их в меню слева.

3. На странице **Центры уведомлений** выберите **Добавить** на панели инструментов.

   :::image type="content" source="media/android-sdk/add-hub.png" alt-text="Добавление центра":::

4. На странице **Центры уведомлений** выполните следующее:

   1. Введите имя в поле **Центр уведомлений**.

   2. Введите имя в поле **Создать пространство имен**. Пространство имен содержит один или несколько центров.

   3. Выберите значение в раскрывающемся списке **Расположение**. Это значение определяет расположение, в котором создается центр.

   4. В разделе **Группа ресурсов** создайте группу ресурсов или выберите имеющуюся.

   5. Выберите **Создать**.

      :::image type="content" source="media/android-sdk/create-hub.png" alt-text="Создание центра":::

5. Последовательно выберите **Уведомления** (значок колокольчика) и **Перейти к ресурсу**. Вы также можете обновить список на странице **Центры уведомлений** и выбрать свой центр.

   :::image type="content" source="media/android-sdk/notification-hubs.png" alt-text="Выбор центра":::

6. Выберите **Политики доступа** в списке. Обратите внимание, что станут доступны две строки подключения. Они потребуются позже для обработки push-уведомлений.

   :::image type="content" source="media/android-sdk/access-policies.png" alt-text="Политики доступа":::

   > [!IMPORTANT]
   > Не используйте в приложении политику **DefaultFullSharedAccessSignature**. Эта политика используется только в серверной части приложения.

## <a name="configure-firebase-cloud-messaging-settings-for-the-hub"></a>Настройка параметров Firebase Cloud Messaging для центра

1. На панели слева в разделе **Параметры**, выберите **Google (GCM/FCM)** .

2. Введите **ключ сервера** для проекта FCM, сохраненного ранее.

3. На панели инструментов нажмите кнопку **Сохранить**.

   :::image type="content" source="media/android-sdk/fcm-server-key.png" alt-text="Ключ сервера":::

4. Портал Azure отобразит сообщение о том, что центр был успешно обновлен. Кнопка **Сохранить** отключена.

Теперь ваш центр уведомлений настроен для работы с Firebase Cloud Messaging. У вас также есть строки подключения, которые необходимы для отправки уведомлений на устройство и регистрации приложения для получения уведомлений.

## <a name="connect-your-app-to-the-notification-hub"></a>Подключение приложения к центру уведомлений

### <a name="add-google-play-services-to-the-project"></a>Добавление служб Google Play в проект

1. В Android Studio в меню выберите **Средства**, а затем — **Диспетчер пакетов SDK**.

2. Выберите целевую версию пакета SDK для Android, который используется в проекте. Затем выберите **Show Package Details** (Показать сведения о пакете).

   :::image type="content" source="media/android-sdk/notification-hubs-android-studio-sdk-manager.png" alt-text="Диспетчер пакетов SDK":::

3. Выберите **Google APIs** (API-интерфейсы Google), если они еще не установлены.

   :::image type="content" source="media/android-sdk/google-apis-selected.png" alt-text="Программные интерфейсы":::

4. Переключитесь на вкладку **SDK Tools**. Если сервисы Google Play еще не установлены, щелкните **Сервисы Google Play**, как показано на рисунке ниже. Затем выберите **Применить** для установки. Запишите путь к пакету SDK. Он вам потребуется в дальнейшем.

   :::image type="content" source="media/android-sdk/google-play-services-selected.png" alt-text="Службы Google Play":::

5. Если вы видите диалоговое окно **Подтвердить изменение**, выберите **ОК**. Установщик компонентов устанавливает запрошенные компоненты. После установки компонентов выберите **Готово**.

6. Выберите **ОК**, чтобы закрыть диалоговое окно **Settings for New Projects** (Параметры для новых проектов).

### <a name="add-azure-notification-hubs-libraries"></a>Затем добавьте библиотеки Центров уведомлений Azure.

1. В файле **build.gradle** для приложений добавьте следующие строки в раздел dependencies:

   ```gradle
   implementation 'com.microsoft.azure:notification-hubs-android-sdk-fcm:1.1.4'
   implementation 'androidx.appcompat:appcompat:1.0.0'
   ```

2. После раздела dependencies добавьте следующий репозиторий:

   ```gradle
   repositories {
      maven {
         url "https://dl.bintray.com/microsoftazuremobile/SDK"
      }
   }
   ```

### <a name="add-google-firebase-support"></a>Добавление поддержки Google Firebase

1. Добавьте следующий подключаемый модуль в конце файла, если он еще не выбран.

   ```gradle
   apply plugin: 'com.google.gms.google-services'
   ```

2. На панели инструментов щелкните **Синхронизировать сейчас**.

### <a name="add-code"></a>Добавление кода

1. Создайте объект **NotificationHubListener**, который обрабатывает перехват сообщений из Центров уведомлений Azure.

   ```java
   public class CustomNotificationListener implements NotificationListener {

      @override
      public void onNotificationReceived(Context context, RemoteMessage message) {
    
         /* The following notification properties are available. */
         Notification notification = message.getNotification();
         String title = notification.getTitle();
         String body = notification.getBody();
         Map<String, String> data = message.getData();
    
         if (message != null) {
            Log.d(TAG, "Message Notification Title: " + title);
            Log.d(TAG, "Message Notification Body: " + message);
         }

         if (data != null) {
             for (Map.Entry<String, String> entry : data.entrySet()) {
                 Log.d(TAG, "key, " + entry.getKey() + " value " + entry.getValue());
             }
         }
      }
   }
   ```

2. В методе `OnCreate` класса `MainActivity` добавьте следующий код, чтобы запустить инициализацию Центров уведомлений при создании действия:

   ```java
   @Override
   protected void onCreate(Bundle savedInstanceState) {

      super.onCreate(savedInstanceState);
      setContentView(R.layout.activity\_main);
      NotificationHub.setListener(new CustomNotificationListener());
      NotificationHub.start(this.getApplication(), “Connection-String”, "Hub Name");

   }
   ```

3. В Android Studio в строке меню выберите **Сборка**, затем выберите **Пересобрать проект** чтобы убедиться, что в вашем коде нет ошибок. Если появляется сообщение об ошибке о значке **ic_launcher**, удалите следующую инструкцию из файла AndroidManifest.xml:

   ```xml
   android:icon="@mipmap/ic_launcher"
   ```

4. Убедитесь, что у вас есть виртуальное устройство для запуска приложения. Если его нет, добавьте его следующим образом:

   1. :::image type="content" source="media/android-sdk/open-device-manager.png" alt-text="Диспетчер устройств":::
   2. :::image type="content" source="media/android-sdk/your-virtual-devices.png" alt-text="Виртуальные устройства":::
   3. Запустите приложение на выбранном устройстве и убедитесь, что оно успешно зарегистрировано в центре.

      :::image type="content" source="media/android-sdk/device-registration.png" alt-text="Регистрация устройств":::

      > [!NOTE]
      > Во время первоначального запуска регистрация может завершиться неудачно, пока не будет вызван метод `onTokenRefresh()` службы ИД экземпляра. Чтобы заново начать регистрацию в центре уведомлений, обновите страницу.

## <a name="send-a-test-notification"></a>отправка проверочного уведомления.

Push-уведомления можно отправлять в центр уведомлений из [портала Azure](https://portal.azure.com/) следующим образом:

1. На портале Azure на странице "Центра уведомлений" для своего центра выберите **Тестовая отправка** в разделе **Устранение неполадок**.

2. В качестве **платформы** выберите **Android**.

3. Нажмите кнопку **Отправить**. Вы пока не увидите уведомление на устройстве Android, потому что на нем еще не запущено мобильное приложение. После запуска мобильного приложения нажмите еще раз кнопку **Send** (Отправить), чтобы просмотреть уведомление.

4. Результат операции можно увидеть в списке в нижней части портала.

   :::image type="content" source="media/android-sdk/notification-hubs-test-send.png" alt-text="Отправка тестового уведомления":::

5. Вы увидите сообщение уведомления на своем устройстве.

Push-уведомления обычно отправляются во внутренней службе, например мобильных приложениях или службе ASP.NET, с помощью совместимой библиотеки. Если для серверной части библиотека недоступна, для отправки уведомлений также можно напрямую использовать REST API.

## <a name="run-the-mobile-app-on-emulator"></a>Запуск мобильного приложения в эмуляторе

Перед проверкой отправки push-уведомлений в эмуляторе, убедитесь, что образ эмулятора поддерживает уровень API Google, выбранный для приложения. Если образ не поддерживает собственные API-интерфейсы Google, создается исключение **SERVICE_NOT_AVAILABLE**.

Кроме того, добавьте учетную запись Google в запущенный эмулятор. Для этого щелкните **Settings** (Параметры) > **Accounts** (Учетные записи). В противном случае попытки регистрации в FCM могут привести к исключению **AUTHENTICATION_FAILED**.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы использовали Firebase Cloud Messaging, для отправки уведомлений на все устройства Android, зарегистрированных в службе. Чтобы узнать, как отправлять push-уведомления на конкретные устройства, перейдите к следующему руководству:

> [!div class="nextstepaction"]
>[Руководство. Отправка уведомлений определенным пользователям](push-notifications-android-specific-users-firebase-cloud-messaging.md)

Ниже приведен список других учебников, касающихся отправки уведомлений:

- Мобильные приложения Azure. Пример отправки уведомлений с сервера мобильных приложений, интегрированного с Центрами уведомлений, см. в статье [Добавление push-уведомлений в приложение iOS](/previous-versions/azure/app-service-mobile/app-service-mobile-ios-get-started-push).

- ASP.NET: [Отправка push-уведомлений определенным пользователям с помощью Центров уведомлений](notification-hubs-aspnet-backend-ios-apple-apns-notification.md).

- Пакет SDK для Центров уведомлений Azure для Java: сведения об отправке уведомлений Java см. в статье [Использование концентраторов уведомлений из Java](notification-hubs-java-push-notification-tutorial.md). Было протестировано в Eclipse для разработки для Android.

- PHP: [Использование Центров уведомлений из PHP](notification-hubs-php-push-notification-tutorial.md).
