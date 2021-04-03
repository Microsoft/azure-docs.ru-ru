---
title: Отправка push-уведомлений в Android с помощью Центров уведомлений Azure и Firebase SDK версии 0.6 | Документация Майкрософт
description: Из этого руководства вы узнаете, как с помощью Центров уведомлений Microsoft Azure и Google Firebase Cloud Messaging отправлять push-уведомления на устройства Android (с использованием пакета SDK Firebase версии 0.6).
services: notification-hubs
documentationcenter: android
keywords: push-уведомления, push-уведомление, push-уведомление android, FCM, Firebase Cloud Messaging
author: sethmanheim
manager: femila
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-android
ms.devlang: java
ms.topic: tutorial
ms.custom: mvc, devx-track-java
ms.date: 06/22/2020
ms.author: sethm
ms.reviewer: thsomasu
ms.lastreviewed: 09/11/2019
ms.openlocfilehash: c5485dacc4d9e3210ad69819caf4e36f96c626da
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92428381"
---
# <a name="tutorial-send-push-notifications-to-android-devices-using-firebase-sdk-version-06"></a>Руководство по отправке push-уведомлений на устройства Android с помощью Firebase SDK версии 0.6

[!INCLUDE [notification-hubs-selector-get-started](../../includes/notification-hubs-selector-get-started.md)]

В этом руководстве показано, как использовать Центры уведомлений Azure и Firebase Cloud Messaging (FCM) SDK версии 0.6 для отправки push-уведомлений в приложение на платформе Android. Следуя инструкциям этого руководства, вы создадите пустое приложение Android, которое получает push-уведомления с помощью Firebase Cloud Messaging.

Полный код для этого руководства можно скачать на сайте [GitHub](https://github.com/Azure/azure-notificationhubs-android/tree/master/FCMTutorialApp).

При работе с этим руководством вы выполните следующие задачи:

> [!div class="checklist"]
> * Создание проекта Android Studio.
> * Создание проекта Firebase с поддержкой Firebase Cloud Messaging.
> * Создание концентратора.
> * Подключение своего приложения к концентратору.
> * Тестирование приложения.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим учебником необходима активная учетная запись Azure. Если ее нет, можно создать бесплатную пробную учетную запись всего за несколько минут. Дополнительные сведения см. в разделе [Бесплатная пробная версия Azure](https://azure.microsoft.com/free/). 

Вам также понадобятся следующее. 

* Последняя версия [Android Studio](https://go.microsoft.com/fwlink/?LinkId=389797).
* Android версии 2.3 или более поздней.
* Репозиторий Google версии 27 или более поздней.
* Службы Google Play версии 9.0.2 или более поздней.

Завершение изучения этого учебника является необходимым условием для работы со всеми другими учебниками, посвященными Центрам уведомлений для приложений Android.

## <a name="create-an-android-studio-project"></a>Создание проекта Android Studio

1. Запустите Android Studio.
2. В меню выберите **File** (Файл), **New** (Создать), а затем — **New Project** (Создать проект). 
2. На странице **Choose your project** (Выбор проекта) выберите **Empty Activity** (Пустое действие) и щелкните **Далее**. 
3. На странице **Configure your project** (Настройка проекта) выполните следующие действия. 
    1. Введите имя приложения.
    2. Укажите расположение для сохранения файлов проекта. 
    3. Нажмите кнопку **Готово**. 

        ![Настройка проекта](./media/notification-hubs-android-push-notification-google-fcm-get-started/configure-project.png)

## <a name="create-a-firebase-project-that-supports-fcm"></a>Создание проекта Firebase с поддержкой FCM

[!INCLUDE [notification-hubs-enable-firebase-cloud-messaging](../../includes/notification-hubs-enable-firebase-cloud-messaging.md)]

## <a name="configure-a-hub"></a>Настройка концентратора

[!INCLUDE [notification-hubs-portal-create-new-hub](../../includes/notification-hubs-portal-create-new-hub.md)]

### <a name="configure-firebase-cloud-messaging-settings-for-the-hub"></a>Настройка параметров Firebase Cloud Messaging для центра

1. На панели слева в разделе **Параметры**, выберите **Google (GCM/FCM)** . 
2. Введите **ключ сервера** для проекта FCM, сохраненного ранее. 
3. На панели инструментов нажмите кнопку **Сохранить**. 

    ![Центры уведомлений Azure в Google (FCM)](./media/notification-hubs-android-push-notification-google-fcm-get-started/fcm-server-key.png)
4. Портал Azure отображает предупреждения о том, что концентратор был успешно обновлен. Кнопка **Сохранить** отключена. 

Теперь ваша служба настроена для работы с Firebase Cloud Messaging. У вас также есть строки подключения, которые необходимы для отправки уведомлений на устройство и регистрации приложения для получения уведомлений.

## <a name="connect-your-app-to-the-notification-hub"></a><a id="connecting-app"></a>Подключение приложения к центру уведомлений

### <a name="add-google-play-services-to-the-project"></a>Добавление служб Google Play в проект

1. В Android Studio в меню выберите **Средства**, а затем — **Диспетчер пакетов SDK**. 
2. Выберите целевую версию пакета SDK для Android, который используется в проекте. Затем выберите **Show Package Details** (Показать сведения о пакете). 

    ![Диспетчер пакетов SDK Android: выбор целевой версии](./media/notification-hubs-android-studio-add-google-play-services/notification-hubs-android-studio-sdk-manager.png)
3. Выберите **Google APIs** (API-интерфейсы Google), если они еще не установлены.

    ![Диспетчер пакетов SDK для Android с выбранными API-интерфейсами Google](./media/notification-hubs-android-studio-add-google-play-services/googole-apis-selected.png)
4. Переключитесь на вкладку **SDK Tools**. Если сервисы Google Play еще не установлены, щелкните **Сервисы Google Play**, как показано на рисунке ниже. Затем выберите **Применить** для установки. Запишите путь к пакету SDK. Он вам потребуется в дальнейшем.

    ![Диспетчер пакетов SDK для Android с выбранными Сервисами Google Play](./media/notification-hubs-android-studio-add-google-play-services/google-play-services-selected.png)
3. Если вы видите диалоговое окно **Подтвердить изменение**, выберите **ОК**. Установщик компонентов устанавливает запрошенные компоненты. После установки компонентов выберите **Готово**.
4. Выберите **ОК**, чтобы закрыть диалоговое окно **Settings for New Projects** (Параметры для новых проектов).  
1. Откройте файл AndroidManifest.xml, а затем добавьте следующий тег для тега *приложения*.

    ```xml
    <meta-data android:name="com.google.android.gms.version"
         android:value="@integer/google_play_services_version" />
    ```


### <a name="add-azure-notification-hubs-libraries"></a>Затем добавьте библиотеки Центров уведомлений Azure.

1. В файле Build.Gradle для приложений добавьте следующие строки в раздел dependencies.

    ```gradle
    implementation 'com.microsoft.azure:notification-hubs-android-sdk:0.6@aar'
    ```

2. После раздела dependencies добавьте следующий репозиторий.

    ```gradle
    repositories {
        maven {
            url "https://dl.bintray.com/microsoftazuremobile/SDK"
        }
    }
    ```

### <a name="add-google-firebase-support"></a>Добавление поддержки Google Firebase

1. В файле Build.Gradle для приложения добавьте следующие строки в раздел **dependencies**, если они еще не существуют. 

    ```gradle
    implementation 'com.google.firebase:firebase-core:16.0.8'
    implementation 'com.google.firebase:firebase-messaging:17.3.4'
    ```

2. Добавьте следующий подключаемый модуль в конце файла, если он еще не выбран. 

    ```gradle
    apply plugin: 'com.google.gms.google-services'
    ```
3. На панели инструментов щелкните **Синхронизировать сейчас**.

### <a name="update-the-androidmanifestxml-file"></a>Обновите файл AndroidManifest.xml

1. После получения маркера регистрации в FCM используйте его для [регистрации в Центрах уведомлений Azure](notification-hubs-push-notification-registration-management.md). Регистрация в фоновом режиме выполняется с помощью службы `IntentService` с именем `RegistrationIntentService`. Эта служба также обновит ваш маркер регистрации FCM. Также создается класс с именем `FirebaseService` в качестве подкласса `FirebaseMessagingService` и переопределяется метод `onMessageReceived` для получения и обработки уведомлений. 

    Добавьте приведенное ниже определение службы внутри тега `<application>` в файле AndroidManifest.xml.

    ```xml
    <service
        android:name=".RegistrationIntentService"
        android:exported="false">
    </service>
    <service
        android:name=".FirebaseService"
        android:exported="false">
        <intent-filter>
            <action android:name="com.google.firebase.MESSAGING_EVENT" />
        </intent-filter>
    </service>
    ```
3. Добавьте следующие разрешения, связанные с FCM, под тегом `</application>`.

    ```xml
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.GET_ACCOUNTS"/>
    <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
    ```

### <a name="add-code"></a>Добавление кода

1. В представлении проекта разверните узел **app** > **src** > **main** > **java**. Щелкните правой кнопкой мыши папку своего пакета в разделе **java**, выберите **New** (Создать), а затем выберите **класс Java**. Укажите **NotificationSettings** для имени, а затем нажмите кнопку **ОК**.

    Обязательно обновите эти три заполнителя в следующем коде для класса `NotificationSettings`:

   * **HubListenConnectionString** — укажите для центра строку подключения **DefaultListenAccessSignature**. Чтобы скопировать эту строку подключения, щелкните пункт **Политики доступа** в своем центре на [Портал Azure].
   * **HubName**: Используйте имя своего концентратора, которое отображается на странице концентратора на [Портал Azure].

     `NotificationSettings` :

        ```java
        public class NotificationSettings {
            public static String HubName = "<Your HubName>";
            public static String HubListenConnectionString = "<Enter your DefaultListenSharedAccessSignature connection string>";
        }
        ```

     > [!IMPORTANT]
     > Прежде чем продолжить, введите **имя** и **DefaultListenSharedAccessSignature** вашего концентратора. 

2. Добавьте еще один новый класс в проект `RegistrationIntentService`. Этот класс реализует интерфейс `IntentService`. Он также выполняет [обновление маркера FCM](https://developers.google.com/instance-id/guides/android-implementation#refresh_tokens) и [регистрацию в центре уведомлений](notification-hubs-push-notification-registration-management.md).

    Используйте для этого класса следующий код:

    ```java
    import android.app.IntentService;
    import android.content.Intent;
    import android.content.SharedPreferences;
    import android.preference.PreferenceManager;
    import android.util.Log;
    import com.google.android.gms.tasks.OnSuccessListener;
    import com.google.firebase.iid.FirebaseInstanceId;
    import com.google.firebase.iid.InstanceIdResult;
    import com.microsoft.windowsazure.messaging.NotificationHub;
    import java.util.concurrent.TimeUnit;

    public class RegistrationIntentService extends IntentService {

        private static final String TAG = "RegIntentService";
        String FCM_token = null;

        private NotificationHub hub;

        public RegistrationIntentService() {
            super(TAG);
        }

        @Override
        protected void onHandleIntent(Intent intent) {

            SharedPreferences sharedPreferences = PreferenceManager.getDefaultSharedPreferences(this);
            String resultString = null;
            String regID = null;
            String storedToken = null;

            try {
                FirebaseInstanceId.getInstance().getInstanceId().addOnSuccessListener(new OnSuccessListener<InstanceIdResult>() { 
                    @Override 
                    public void onSuccess(InstanceIdResult instanceIdResult) { 
                        FCM_token = instanceIdResult.getToken(); 
                        Log.d(TAG, "FCM Registration Token: " + FCM_token); 
                    } 
                }); 
                TimeUnit.SECONDS.sleep(1);

                // Storing the registration ID that indicates whether the generated token has been
                // sent to your server. If it is not stored, send the token to your server.
                // Otherwise, your server should have already received the token.
                if (((regID=sharedPreferences.getString("registrationID", null)) == null)){

                    NotificationHub hub = new NotificationHub(NotificationSettings.HubName,
                            NotificationSettings.HubListenConnectionString, this);
                    Log.d(TAG, "Attempting a new registration with NH using FCM token : " + FCM_token);
                    regID = hub.register(FCM_token).getRegistrationId();

                    // If you want to use tags...
                    // Refer to : https://azure.microsoft.com/documentation/articles/notification-hubs-routing-tag-expressions/
                    // regID = hub.register(token, "tag1,tag2").getRegistrationId();

                    resultString = "New NH Registration Successfully - RegId : " + regID;
                    Log.d(TAG, resultString);

                    sharedPreferences.edit().putString("registrationID", regID ).apply();
                    sharedPreferences.edit().putString("FCMtoken", FCM_token ).apply();
                }

                // Check to see if the token has been compromised and needs refreshing.
               else if (!(storedToken = sharedPreferences.getString("FCMtoken", "")).equals(FCM_token)) {

                    NotificationHub hub = new NotificationHub(NotificationSettings.HubName,
                            NotificationSettings.HubListenConnectionString, this);
                    Log.d(TAG, "NH Registration refreshing with token : " + FCM_token);
                    regID = hub.register(FCM_token).getRegistrationId();

                    // If you want to use tags...
                    // Refer to : https://azure.microsoft.com/documentation/articles/notification-hubs-routing-tag-expressions/
                    // regID = hub.register(token, "tag1,tag2").getRegistrationId();

                    resultString = "New NH Registration Successfully - RegId : " + regID;
                    Log.d(TAG, resultString);

                    sharedPreferences.edit().putString("registrationID", regID ).apply();
                    sharedPreferences.edit().putString("FCMtoken", FCM_token ).apply();
                }

                else {
                    resultString = "Previously Registered Successfully - RegId : " + regID;
                }
            } catch (Exception e) {
                Log.e(TAG, resultString="Failed to complete registration", e);
                // If an exception happens while fetching the new token or updating registration data
                // on a third-party server, this ensures that we'll attempt the update at a later time.
            }

            // Notify UI that registration has completed.
            if (MainActivity.isVisible) {
                MainActivity.mainActivity.ToastNotify(resultString);
            }
        }
    }
    ```

3. В классе `MainActivity` добавьте следующие операторы `import` выше объявления класса.

    ```java
    import com.google.android.gms.common.ConnectionResult;
    import com.google.android.gms.common.GoogleApiAvailability;
    import android.content.Intent;
    import android.util.Log;
    import android.widget.TextView;
    import android.widget.Toast;
    ```

4. Добавьте следующие элементы в верхней части класса. Используйте эти поля для [проверки доступности служб Google Play в соответствии с рекомендациями Google](https://developers.google.com/android/guides/setup#ensure_devices_have_the_google_play_services_apk).

    ```java
    public static MainActivity mainActivity;
    public static Boolean isVisible = false;
    private static final String TAG = "MainActivity";
    private static final int PLAY_SERVICES_RESOLUTION_REQUEST = 9000;
    ```

5. В классе `MainActivity` добавьте следующий метод проверки доступности сервисов Google Play.

    ```java
    /**
    * Check the device to make sure it has the Google Play Services APK. If
    * it doesn't, display a dialog box that enables  users to download the APK from
    * the Google Play Store or enable it in the device's system settings.
    */

    private boolean checkPlayServices() {
        GoogleApiAvailability apiAvailability = GoogleApiAvailability.getInstance();
        int resultCode = apiAvailability.isGooglePlayServicesAvailable(this);
        if (resultCode != ConnectionResult.SUCCESS) {
            if (apiAvailability.isUserResolvableError(resultCode)) {
                apiAvailability.getErrorDialog(this, resultCode, PLAY_SERVICES_RESOLUTION_REQUEST)
                        .show();
            } else {
                Log.i(TAG, "This device is not supported by Google Play Services.");
                ToastNotify("This device is not supported by Google Play Services.");
                finish();
            }
            return false;
        }
        return true;
    }
    ```

6. В классе `MainActivity` добавьте следующий код, который проверяет сервисы Google Play, прежде чем вызывать `IntentService`. Таким образом вы получите маркер регистрации в FCM и выполните регистрацию в своем центре.

    ```java
    public void registerWithNotificationHubs()
    {
        if (checkPlayServices()) {
            // Start IntentService to register this application with FCM.
            Intent intent = new Intent(this, RegistrationIntentService.class);
            startService(intent);
        }
    }
    ```

7. В методе `OnCreate` класса `MainActivity` добавьте следующий код, чтобы начать регистрацию при создании действия.

    ```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mainActivity = this;
        registerWithNotificationHubs();
        FirebaseService.createChannelAndHandleNotifications(getApplicationContext());
    }
    ```

8. Добавьте эти дополнительные методы в класс `MainActivity`, чтобы проверять состояние приложения и отображать в нем полученные данные.

    ```java
    @Override
    protected void onStart() {
        super.onStart();
        isVisible = true;
    }

    @Override
    protected void onPause() {
        super.onPause();
        isVisible = false;
    }

    @Override
    protected void onResume() {
        super.onResume();
        isVisible = true;
    }

    @Override
    protected void onStop() {
        super.onStop();
        isVisible = false;
    }

    public void ToastNotify(final String notificationMessage) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                Toast.makeText(MainActivity.this, notificationMessage, Toast.LENGTH_LONG).show();
                TextView helloText = (TextView) findViewById(R.id.text_hello);
                helloText.setText(notificationMessage);
            }
        });
    }
    ```

9. Метод `ToastNotify` использует элемент управления `TextView` *Hello World*, чтобы постоянно передавать в приложение сведения о состоянии и уведомления. В файле макета **res** > **layout** > **activity_main.xml** добавьте следующий идентификатор для этого элемента управления.

    ```java
    android:id="@+id/text_hello"
    ```

    ![Снимок экрана: идентификатор android:id="@+id/text_hello", примененный к элементу управления TextView.](./media/notification-hubs-android-push-notification-google-fcm-get-started/activity-main-xml.png)

10. Затем добавьте подкласс для получателя, определенного в AndroidManifest.xml. Добавьте еще один новый класс в проект `FirebaseService`.

11. Добавьте в начало файла `FirebaseService.java` следующие операторы импорта:

    ```java
    import com.google.firebase.messaging.FirebaseMessagingService;
    import com.google.firebase.messaging.RemoteMessage;
    import android.util.Log;
    import android.app.NotificationChannel;
    import android.app.NotificationManager;
    import android.app.PendingIntent;
    import android.content.Context;
    import android.content.Intent;
    import android.media.RingtoneManager;
    import android.net.Uri;
    import android.os.Build;
    import android.os.Bundle;
    import androidx.core.app.NotificationCompat;
    ```

12. Добавьте в класс `FirebaseService` следующий код, чтобы сделать его подклассом класса `FirebaseMessagingService`.

    Этот код переопределяет метод `onMessageReceived` и сообщает о полученных уведомлениях. Кроме того, он отправляет push-уведомление в диспетчер уведомлений Android с помощью метода `sendNotification()`. Вызовите метод `sendNotification()`, если получено уведомление, а приложение не запущено.

    ```java
    public class FirebaseService extends FirebaseMessagingService
    {
        private String TAG = "FirebaseService";
    
        public static final String NOTIFICATION_CHANNEL_ID = "nh-demo-channel-id";
        public static final String NOTIFICATION_CHANNEL_NAME = "Notification Hubs Demo Channel";
        public static final String NOTIFICATION_CHANNEL_DESCRIPTION = "Notification Hubs Demo Channel";
    
        public static final int NOTIFICATION_ID = 1;
        private NotificationManager mNotificationManager;
        NotificationCompat.Builder builder;
        static Context ctx;
    
        @Override
        public void onMessageReceived(RemoteMessage remoteMessage) {
            // ...
    
            // TODO(developer): Handle FCM messages here.
            // Not getting messages here? See why this may be: https://goo.gl/39bRNJ
            Log.d(TAG, "From: " + remoteMessage.getFrom());
    
            String nhMessage;
            // Check if message contains a notification payload.
            if (remoteMessage.getNotification() != null) {
                Log.d(TAG, "Message Notification Body: " + remoteMessage.getNotification().getBody());
    
                nhMessage = remoteMessage.getNotification().getBody();
            }
            else {
                nhMessage = remoteMessage.getData().values().iterator().next();
            }
    
            // Also if you intend on generating your own notifications as a result of a received FCM
            // message, here is where that should be initiated. See sendNotification method below.
            if (MainActivity.isVisible) {
                MainActivity.mainActivity.ToastNotify(nhMessage);
            }
            sendNotification(nhMessage);
        }
    
        private void sendNotification(String msg) {
    
            Intent intent = new Intent(ctx, MainActivity.class);
            intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);
    
            mNotificationManager = (NotificationManager)
                    ctx.getSystemService(Context.NOTIFICATION_SERVICE);
    
            PendingIntent contentIntent = PendingIntent.getActivity(ctx, 0,
                    intent, PendingIntent.FLAG_ONE_SHOT);
    
            Uri defaultSoundUri = RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION);
            NotificationCompat.Builder notificationBuilder = new NotificationCompat.Builder(
                    ctx,
                    NOTIFICATION_CHANNEL_ID)
                    .setContentText(msg)
                    .setPriority(NotificationCompat.PRIORITY_HIGH)
                    .setSmallIcon(android.R.drawable.ic_popup_reminder)
                    .setBadgeIconType(NotificationCompat.BADGE_ICON_SMALL);
    
            notificationBuilder.setContentIntent(contentIntent);
            mNotificationManager.notify(NOTIFICATION_ID, notificationBuilder.build());
        }
    
        public static void createChannelAndHandleNotifications(Context context) {
            ctx = context;
    
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
                NotificationChannel channel = new NotificationChannel(
                        NOTIFICATION_CHANNEL_ID,
                        NOTIFICATION_CHANNEL_NAME,
                        NotificationManager.IMPORTANCE_HIGH);
                channel.setDescription(NOTIFICATION_CHANNEL_DESCRIPTION);
                channel.setShowBadge(true);
    
                NotificationManager notificationManager = context.getSystemService(NotificationManager.class);
                notificationManager.createNotificationChannel(channel);
             }
        }
    }
    ```

13. В Android Studio в строке меню выберите **Build** > **Rebuild Project** (Сборка > Пересобрать проект) чтобы убедиться, что в вашем коде нет ошибок. Если появляется сообщение об ошибке о значке `ic_launcher`, удалите следующую инструкцию из файла AndroidManifest.xml. 

    ```
        android:icon="@mipmap/ic_launcher"
    ```
14. Убедитесь, что у вас есть виртуальное устройство для запуска приложения. Если его нет, добавьте его следующим образом:
    1. ![Запуск диспетчера устройств](./media/notification-hubs-android-push-notification-google-fcm-get-started/open-device-manager.png)
    2. ![Создание виртуального устройства](./media/notification-hubs-android-push-notification-google-fcm-get-started/your-virtual-devices.PNG)

15. Запустите приложение на выбранном устройстве и убедитесь, что оно успешно зарегистрировано в концентраторе.

    > [!NOTE]
    > Во время первоначального запуска регистрация может завершиться неудачно, пока не будет вызван метод `onTokenRefresh()` службы ИД экземпляра. Чтобы заново начать регистрацию в центре уведомлений, обновите страницу.

    ![Устройство успешно зарегистрировано](./media/notification-hubs-android-push-notification-google-fcm-get-started/device-registration.png)

## <a name="test-send-notification-from-the-notification-hub"></a>Проверка отправки уведомления из центра уведомлений

Push-уведомления с [Портал Azure] можно отправить, выполнив следующие действия.

1. На портале Azure на странице "Цента уведомлений" для своего центра выберите **Тестовая отправка** в разделе **Устранение неполадок**.
3. В качестве **платформы** выберите **Android**.
4. Нажмите кнопку **Отправить**.  Вы пока не увидите уведомление на устройстве Android, потому что на нем еще не запущено мобильное приложение. После запуска мобильного приложения нажмите еще раз кнопку **Send** (Отправить), чтобы просмотреть уведомление.
5. Результат операции можно увидеть в списке внизу.

    ![Центры уведомлений Azure — тестовая отправка](./media/notification-hubs-android-push-notification-google-fcm-get-started/notification-hubs-test-send.png)
6. Вы увидите сообщение уведомления на своем устройстве. 

    ![Сообщение уведомления на устройстве](./media/notification-hubs-android-push-notification-google-fcm-get-started/notification-on-device.png)
    

[!INCLUDE [notification-hubs-sending-notifications-from-the-portal](../../includes/notification-hubs-sending-notifications-from-the-portal.md)]

### <a name="run-the-mobile-app-on-emulator"></a>Запуск мобильного приложения в эмуляторе

Перед проверкой отправки push-уведомлений в эмуляторе, убедитесь, что образ эмулятора поддерживает уровень API Google, выбранный для приложения. Если образ не поддерживает собственные API-интерфейсы Google, создается исключение **SERVICE\_NOT\_AVAILABLE**.

Кроме того, добавьте учетную запись Google в запущенный эмулятор. Для этого щелкните **Settings** (Параметры) > **Accounts** (Учетные записи). В противном случае попытки регистрации в FCM могут привести к исключению **AUTHENTICATION\_FAILED**.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы использовали Firebase Cloud Messaging, для отправки уведомлений на все устройства Android, зарегистрированных в службе. Чтобы узнать, как отправлять push-уведомления на конкретные устройства, перейдите к следующему руководству:

> [!div class="nextstepaction"]
>[Руководство по отправке push-уведомлений на конкретные устройства Android](push-notifications-android-specific-devices-firebase-cloud-messaging.md)

<!-- Images. -->

<!-- URLs. -->
[Get started with push notifications in Mobile Services]: ../mobile-services-javascript-backend-android-get-started-push.md  
[Mobile Services Android SDK]: https://go.microsoft.com/fwLink/?LinkID=280126&clcid=0x409
[Referencing a library project]: https://go.microsoft.com/fwlink/?LinkId=389800
[Notification Hubs Guidance]: notification-hubs-push-notification-overview.md
[Портал Azure]: https://portal.azure.com
