---
title: Отправка push-уведомлений в Android с помощью Центров уведомлений Azure и Google Cloud Messaging | Документация Майкрософт
description: Из этого руководства вы узнаете, как использовать центры уведомлений Azure и Google Firebase Cloud Messaging для отправки push-уведомлений на устройства Android.
services: notification-hubs
documentationcenter: android
keywords: push-уведомления, push-уведомление, push-уведомление android
author: sethmanheim
manager: femila
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-android
ms.devlang: java
ms.topic: tutorial
ms.custom: mvc, devx-track-java
ms.date: 01/04/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 01/04/2019
ms.openlocfilehash: 8c97710202a448c613ab685932cb335bbaed4953
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "87832660"
---
# <a name="tutorial-send-push-notifications-to-android-devices-by-using-azure-notification-hubs-and-google-cloud-messaging-deprecated"></a>Руководство по Отправка push-уведомлений на устройства Android с помощью Центров уведомлений Azure и Google Cloud Messaging (не рекомендуется к использованию)

[!INCLUDE [notification-hubs-selector-get-started](../../includes/notification-hubs-selector-get-started.md)]

> [!WARNING]
> С 10 апреля 2018 года компания Google не рекомендует использовать Google Cloud Messaging (GCM). Интерфейсы API сервера и клиента GCM являются нерекомендуемыми и будут удалены после 29 мая 2019 года. Дополнительные сведения см. в статье [GCM and FCM Frequently Asked Questions](https://developers.google.com/cloud-messaging/faq) (Часто задаваемые вопросы о GCM и FCM).

## <a name="overview"></a>Обзор

В этом учебнике показано, как использовать Центры уведомлений Azure для отправки push-уведомлений в приложение на платформе Android.
Вы создадите пустое приложение Android, которое получает push-уведомления с помощью Google Cloud Messaging (GCM).

> [!IMPORTANT]
> Служба Google Cloud Messaging (GCM) устарела и будет удалена [в ближайшее время](https://developers.google.com/cloud-messaging/faq).

> [!IMPORTANT]
> В этой статье описывается отправка push-уведомлений с помощью Google Cloud Messaging (GCM). Если вы используете Google Firebase Cloud Messaging (GCM), см. статью [Sending push notifications to Android with Azure Notification Hubs](notification-hubs-android-push-notification-google-fcm-get-started.md) (Отправка push-уведомлений в приложения Android с помощью центров уведомлений Azure).

Полный код для этого руководства можно скачать на сайте [GitHub](https://github.com/Azure/azure-notificationhubs-android/tree/master/FCMTutorialApp).

Вот какие действия выполняются в этом руководстве:

> [!div class="checklist"]
> * Создание проекта с поддержкой Google Cloud Messaging.
> * Создание концентратора уведомлений
> * Подключение приложения к центру уведомлений
> * Тестирование приложения

## <a name="prerequisites"></a>Предварительные требования

* **Подписка Azure**. Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.
* [Android Studio](https://go.microsoft.com/fwlink/?LinkId=389797).

## <a name="creating-a-project-that-supports-google-cloud-messaging"></a>Создание проекта с поддержкой Google Cloud Messaging

[!INCLUDE [mobile-services-enable-Google-cloud-messaging](../../includes/mobile-services-enable-google-cloud-messaging.md)]

## <a name="create-a-notification-hub"></a>Создание концентратора уведомлений

[!INCLUDE [notification-hubs-portal-create-new-hub](../../includes/notification-hubs-portal-create-new-hub.md)]

### <a name="configure-gcm-setting-for-the-notification-hub"></a>Настройка параметра GCM для центра уведомлений

1. В разделе **Параметры уведомлений** выберите **Google (GCM)** .
2. Введите **ключ API**, полученный из консоли Google Cloud.
3. На панели инструментов щелкните **Сохранить**.

    ![Центры уведомлений Azure — Google (GCM)](./media/notification-hubs-android-get-started/notification-hubs-gcm-api.png)

Теперь центр уведомлений настроен для работы с GCM, а у вас есть строки подключения, с помощью которых вы можете зарегистрировать приложение для получения и отправки push-уведомлений.

## <a name="connect-your-app-to-the-notification-hub"></a><a id="connecting-app"></a>Подключение приложения к центру уведомлений

### <a name="create-a-new-android-project"></a>Создание нового проекта Android

1. В Android Studio создайте новый проект Android Studio.

   ![Android Studio — новый проект][13]
2. Выберите форм-фактор **Phone and Tablet** (Телефон и планшет) и минимальную версию пакета SDK (с помощью параметра **Minimum SDK**), которые нужно поддерживать. Затем нажмите кнопку **Далее**.

   ![Android Studio — рабочий процесс создания проекта][14]
3. Выберите для основного действия значение **Empty Activity** (Пустое действие), нажмите кнопку **Next** (Далее), а затем — **Finish** (Готово).

### <a name="add-google-play-services-to-the-project"></a>Добавление служб Google Play в проект

[!INCLUDE [Add Play Services](../../includes/notification-hubs-android-studio-add-google-play-services.md)]

### <a name="adding-azure-notification-hubs-libraries"></a>Добавление библиотек Центров уведомлений Azure

1. В файле `Build.Gradle` в классе **app** добавьте следующие строки в раздел **dependencies**.

    ```gradle
    implementation 'com.microsoft.azure:notification-hubs-android-sdk:0.6@aar'
    implementation 'com.microsoft.azure:azure-notifications-handler:1.0.1@aar'
    ```

2. После раздела **dependencies** добавьте следующий репозиторий:

    ```gradle
    repositories {
        maven {
            url "https://dl.bintray.com/microsoftazuremobile/SDK"
        }
    }
    ```

### <a name="updating-the-projects-androidmanifestxml"></a>Обновление файла проекта AndroidManifest.xml

1. Чтобы обеспечить поддержку GCM, реализуйте службу прослушивания идентификаторов экземпляра в коде. Таким образом можно [получать маркеры регистрации](https://developers.google.com/cloud-messaging/) с помощью [API идентификаторов экземпляра Google](https://developers.google.com/instance-id/). В этом руководстве имя класса — `MyInstanceIDService`.

    Добавьте приведенное ниже определение службы внутри тега `<application>` в файле AndroidManifest.xml. Замените заполнитель `<your package>` фактическим именем своего пакета, отображенным в верхней части файла `AndroidManifest.xml`.
  
    ```xml
    <service android:name="<your package>.MyInstanceIDService" android:exported="false">
        <intent-filter>
            <action android:name="com.google.android.gms.iid.InstanceID"/>
        </intent-filter>
    </service>
    ```

2. После получения маркера регистрации в GCM из API идентификаторов экземпляра приложение использует его для [регистрации в центре уведомлений Azure](notification-hubs-push-notification-registration-management.md). Регистрация в фоновом режиме выполняется с помощью службы `IntentService` с именем `RegistrationIntentService`. Кроме того, эта служба отвечает за [обновление маркера регистрации в GCM](https://developers.google.com/instance-id/guides/android-implementation#refresh_tokens).

    Добавьте приведенное ниже определение службы внутри тега `<application>` в файле AndroidManifest.xml. Замените заполнитель `<your package>` фактическим именем своего пакета, отображенным в верхней части файла `AndroidManifest.xml`.

    ```xml
    <service
        android:name="<your package>.RegistrationIntentService"
        android:exported="false">
    </service>
    ```

3. Определите получателя уведомлений. Добавьте следующее определение получателя внутри тега `<application>` в файле AndroidManifest.xml. Замените заполнитель `<your package>` фактическим именем своего пакета, отображенным в верхней части файла `AndroidManifest.xml`.

    ```xml
    <receiver android:name="com.microsoft.windowsazure.notifications.NotificationsBroadcastReceiver"
        android:permission="com.google.android.c2dm.permission.SEND">
        <intent-filter>
            <action android:name="com.google.android.c2dm.intent.RECEIVE" />
            <category android:name="<your package name>" />
        </intent-filter>
    </receiver>
    ```

4. Добавьте следующие разрешения GCM под тегом `<application>`. Замените `<your package>` именем пакета, отображенным в верхней части файла `AndroidManifest.xml`.

    Дополнительные сведения об этих разрешениях см. в статье [Настройка клиентского приложения GCM для Android](https://developers.google.com/cloud-messaging/).

    ```xml
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.GET_ACCOUNTS"/>
    <uses-permission android:name="android.permission.WAKE_LOCK"/>
    <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />

    <permission android:name="<your package>.permission.C2D_MESSAGE" android:protectionLevel="signature" />
    <uses-permission android:name="<your package>.permission.C2D_MESSAGE"/>
    ```

### <a name="adding-code"></a>Добавление кода

1. В представлении проекта разверните узел **app** > **src** > **main** > **java**. Щелкните правой кнопкой мыши папку пакета в **java**, щелкните **New** (Создать) и выберите **Java Class** (Класс Java). Добавьте новый класс с именем `NotificationSettings`.

    ![Android Studio — новый класс Java][6]

    Обновите три заполнителя в следующем коде для класса `NotificationSettings`:

   * `SenderId`: номер проекта, полученный ранее в [консоли Google Cloud](https://cloud.google.com/console).
   * `HubListenConnectionString`: укажите для центра строку подключения **DefaultListenAccessSignature**. Эту строку подключения можно скопировать, щелкнув **Политики доступа** на странице **Параметры** центра на [Портал Azure].
   * `HubName`: используйте имя центра уведомлений, которое отображается на [Портал Azure] на странице центра.

     `NotificationSettings` :

     ```java
     public class NotificationSettings {
        public static String SenderId = "<Your project number>";
        public static String HubName = "<Your HubName>";
        public static String HubListenConnectionString = "<Your default listen connection string>";
     }
     ```

2. Добавьте еще один новый класс с именем `MyInstanceIDService`. Этот класс представляет собой реализацию службы прослушивания идентификаторов экземпляра.

    Код для этого класса вызывает службу `IntentService`, чтобы [обновить маркер GCM](https://developers.google.com/instance-id/guides/android-implementation#refresh_tokens) в фоновом режиме.

    ```java
    import android.content.Intent;
    import android.util.Log;
    import com.google.android.gms.iid.InstanceIDListenerService;

    public class MyInstanceIDService extends InstanceIDListenerService {

        private static final String TAG = "MyInstanceIDService";

        @Override
        public void onTokenRefresh() {

            Log.i(TAG, "Refreshing GCM Registration Token");

            Intent intent = new Intent(this, RegistrationIntentService.class);
            startService(intent);
        }
    };
    ```

3. Добавьте в проект еще один новый класс с именем `RegistrationIntentService`. Этот класс реализует службу `IntentService`, которая [обновляет маркер GCM](https://developers.google.com/instance-id/guides/android-implementation#refresh_tokens) и [выполняет регистрацию в центре уведомлений](notification-hubs-push-notification-registration-management.md).

    Используйте для этого класса следующий код:

    ```java
    import android.app.IntentService;
    import android.content.Intent;
    import android.content.SharedPreferences;
    import android.preference.PreferenceManager;
    import android.util.Log;

    import com.google.android.gms.gcm.GoogleCloudMessaging;
    import com.google.android.gms.iid.InstanceID;
    import com.microsoft.windowsazure.messaging.NotificationHub;

    public class RegistrationIntentService extends IntentService {

        private static final String TAG = "RegIntentService";

        private NotificationHub hub;

        public RegistrationIntentService() {
            super(TAG);
        }

        @Override
        protected void onHandleIntent(Intent intent) {
            SharedPreferences sharedPreferences = PreferenceManager.getDefaultSharedPreferences(this);
            String resultString = null;
            String regID = null;

            try {
                InstanceID instanceID = InstanceID.getInstance(this);
                String token = instanceID.getToken(NotificationSettings.SenderId,
                        GoogleCloudMessaging.INSTANCE_ID_SCOPE);
                Log.i(TAG, "Got GCM Registration Token: " + token);

                // Storing the registration id that indicates whether the generated token has been
                // sent to your server. If it is not stored, send the token to your server,
                // otherwise your server should have already received the token.
                if ((regID=sharedPreferences.getString("registrationID", null)) == null) {
                    NotificationHub hub = new NotificationHub(NotificationSettings.HubName,
                            NotificationSettings.HubListenConnectionString, this);
                    Log.i(TAG, "Attempting to register with NH using token : " + token);

                    regID = hub.register(token).getRegistrationId();

                    // If you want to use tags...
                    // Refer to : https://azure.microsoft.com/documentation/articles/notification-hubs-routing-tag-expressions/
                    // regID = hub.register(token, "tag1", "tag2").getRegistrationId();

                    resultString = "Registered Successfully - RegId : " + regID;
                    Log.i(TAG, resultString);
                    sharedPreferences.edit().putString("registrationID", regID ).apply();
                } else {
                    resultString = "Previously Registered Successfully - RegId : " + regID;
                }
            } catch (Exception e) {
                Log.e(TAG, resultString="Failed to complete token refresh", e);
                // If an exception happens while fetching the new token or updating the registration data
                // on a third-party server, this ensures that we'll attempt the update at a later time.
            }

            // Notify UI that registration has completed.
            if (MainActivity.isVisible) {
                MainActivity.mainActivity.ToastNotify(resultString);
            }
        }
    }
    ```

4. В начале класса `MainActivity` добавьте следующие операторы `import`.

    ```java
    import com.google.android.gms.common.ConnectionResult;
    import com.google.android.gms.common.GoogleApiAvailability;
    import com.google.android.gms.gcm.*;
    import com.microsoft.windowsazure.notifications.NotificationsManager;
    import android.util.Log;
    import android.widget.TextView;
    import android.widget.Toast;
    import android.content.Intent;
    ```

5. В верхней части класса добавьте следующие частные члены. Этот код [проверяет доступность служб Google Play в соответствии с рекомендациями Google](https://developers.google.com/android/guides/setup#ensure_devices_have_the_google_play_services_apk).

    ```java
    public static MainActivity mainActivity;
    public static Boolean isVisible = false;
    private GoogleCloudMessaging gcm;
    private static final int PLAY_SERVICES_RESOLUTION_REQUEST = 9000;
    private static final String TAG = "MainActivity";
    ```

6. В классе `MainActivity` добавьте следующий метод проверки доступности служб Google Play:

    ```java
    /**
        * Check the device to make sure it has the Google Play Services APK. If
        * it doesn't, display a dialog that allows users to download the APK from
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

7. В классе `MainActivity` добавьте следующий код, который проверяет Сервисы Google Play, прежде чем вызывать службу `IntentService`. Таким образом вы получите маркер регистрации в GCM и выполните регистрацию в центре уведомлений.

    ```java
    public void registerWithNotificationHubs()
    {
        Log.i(TAG, " Registering with Notification Hubs");

        if (checkPlayServices()) {
            // Start IntentService to register this application with GCM.
            Intent intent = new Intent(this, RegistrationIntentService.class);
            startService(intent);
        }
    }
    ```

8. В методе `OnCreate` класса `MainActivity` добавьте следующий код, чтобы начать регистрацию при создании действия:

    ```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mainActivity = this;
        NotificationsManager.handleNotifications(this, NotificationSettings.SenderId, MyHandler.class);
        registerWithNotificationHubs();
    }
    ```

9. Добавьте эти дополнительные методы в класс `MainActivity` , чтобы проверять состояние приложения и отображать в нем полученные данные.

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

10. Метод `ToastNotify` использует элемент управления `TextView` *Hello World*, чтобы постоянно передавать в приложение сведения о состоянии и уведомления. В файле макета activity_main.xml добавьте следующий идентификатор для этого элемента управления.

    ```xml
    android:id="@+id/text_hello"
    ```

11. Добавьте подкласс для получателя, определенного в файле AndroidManifest.xml. Добавьте еще один новый класс в проект `MyHandler`.

12. Добавьте в начало файла `MyHandler.java` следующие операторы импорта:

    ```java
    import android.app.NotificationManager;
    import android.app.PendingIntent;
    import android.content.Context;
    import android.content.Intent;
    import android.os.Bundle;
    import android.support.v4.app.NotificationCompat;
    import com.microsoft.windowsazure.notifications.NotificationsHandler;
    import android.net.Uri;
    import android.media.RingtoneManager;
    ```

13. Добавьте в класс `MyHandler` следующий код, чтобы сделать его подклассом класса `com.microsoft.windowsazure.notifications.NotificationsHandler`.

    Этот код переопределяет метод `OnReceive` так, чтобы обработчик сообщал о полученных уведомлениях. Кроме того, обработчик отправляет push-уведомление в диспетчер уведомлений Android с помощью метода `sendNotification()` . Метод `sendNotification()` должен выполняться, когда незапущенное приложение получает уведомление.

    ```java
    public class MyHandler extends NotificationsHandler {
        public static final int NOTIFICATION_ID = 1;
        private NotificationManager mNotificationManager;
        NotificationCompat.Builder builder;
        Context ctx;

        @Override
        public void onReceive(Context context, Bundle bundle) {
            ctx = context;
            String nhMessage = bundle.getString("message");
            sendNotification(nhMessage);
            if (MainActivity.isVisible) {
                MainActivity.mainActivity.ToastNotify(nhMessage);
            }
        }

        private void sendNotification(String msg) {

            Intent intent = new Intent(ctx, MainActivity.class);
            intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);

            mNotificationManager = (NotificationManager)
                    ctx.getSystemService(Context.NOTIFICATION_SERVICE);

            PendingIntent contentIntent = PendingIntent.getActivity(ctx, 0,
                    intent, PendingIntent.FLAG_ONE_SHOT);

            Uri defaultSoundUri = RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION);
            NotificationCompat.Builder mBuilder =
                    new NotificationCompat.Builder(ctx)
                            .setSmallIcon(R.mipmap.ic_launcher)
                            .setContentTitle("Notification Hub Demo")
                            .setStyle(new NotificationCompat.BigTextStyle()
                                    .bigText(msg))
                            .setSound(defaultSoundUri)
                            .setContentText(msg);

            mBuilder.setContentIntent(contentIntent);
            mNotificationManager.notify(NOTIFICATION_ID, mBuilder.build());
        }
    }
    ```

14. В Android Studio в строке меню щелкните **Build** (Сборка)  > **Rebuild Project** (Повторить сборку проекта), чтобы убедиться в отсутствии ошибок.

## <a name="testing-your-app"></a>Тестирование приложения

### <a name="run-the-mobile-application"></a>Запуск мобильного приложения

1. Запустите приложение и убедитесь, что для успешной регистрации сообщается идентификатор регистрации.

      ![Тестирование на устройстве Android — регистрация канала][18]
2. Введите сообщение уведомления для отправки на все устройства Android, которые зарегистрированы в центре.

      ![Тестирование на устройстве Android — отправка сообщения][19]

3. Нажмите кнопку **Send Notification**(Отправить уведомление). На всех устройствах с запущенным приложением отображается экземпляр `AlertDialog` с push-уведомлением. Устройства, на которых приложение не запущено, но которые были ранее зарегистрированы для приема push-уведомлений, получают уведомление в диспетчер уведомлений Android. Уведомления можно просматривать, проводя пальцем вниз от левого верхнего угла.

      ![Тестирование на устройстве Android — уведомления][21]

### <a name="test-send-push-notifications-from-the-azure-portal"></a>Проверка отправки push-уведомлений с портала Azure

Чтобы проверить получение push-уведомлений в приложении, отправьте уведомления на [Портал Azure].

1. В разделе **Устранение неполадок** выберите **Тестовая отправка**.
2. В качестве **платформы** выберите **Android**.
3. Нажмите кнопку **Отправить**, чтобы отправить тестовое уведомление.
4. Проверьте получение уведомления на устройстве Android.

    ![Центры уведомлений Azure — тестовая отправка](./media/notification-hubs-android-get-started/notification-hubs-test-send.png)

[!INCLUDE [notification-hubs-sending-notifications-from-the-portal](../../includes/notification-hubs-sending-notifications-from-the-portal.md)]

#### <a name="push-notifications-in-the-emulator"></a>Тестирование push-уведомлений в эмуляторе

Если вы хотите проверить отправку push-уведомлений в эмуляторе, убедитесь, что образ эмулятора поддерживает уровень API Google, выбранный для приложения. Если образ не поддерживает собственные API-интерфейсы Google, создается исключение **SERVICE\_NOT\_AVAILABLE**.

Кроме того, добавьте учетную запись Google в запущенный эмулятор. Для этого щелкните **Параметры** > **Учетные записи**. В противном случае попытки регистрации в GCM могут привести к исключению **AUTHENTICATION\_FAILED**.

## <a name="optional-send-push-notifications-directly-from-the-app"></a>Отправка push-уведомлений непосредственно из приложения (необязательное действие)

Обычно уведомления отправляются с сервера базы данных. Но в некоторых случаях бывает удобно отправлять push-уведомления непосредственно из клиентского приложения. В этом разделе показано, как отправлять уведомления из клиента с помощью интерфейса [REST API центра уведомлений Azure](/previous-versions/azure/reference/dn223264(v=azure.100)).

1. В представлении проекта Android Studio разверните узел **App** > **src** > **main** > **res** > **layout**. Откройте файл макета `activity_main.xml` и щелкните вкладку **Text** (Текст), чтобы обновить текстовое содержимое файла. Поместите в него приведенный ниже код, который добавляет новые элементы управления `Button` и `EditText` для отправки push-уведомлений в центр уведомлений. Добавьте этот код в конец, непосредственно перед `</RelativeLayout>`.

    ```xml
    <Button
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/send_button"
    android:id="@+id/sendbutton"
    android:layout_centerVertical="true"
    android:layout_centerHorizontal="true"
    android:onClick="sendNotificationButtonOnClick" />

    <EditText
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:id="@+id/editTextNotificationMessage"
    android:layout_above="@+id/sendbutton"
    android:layout_centerHorizontal="true"
    android:layout_marginBottom="42dp"
    android:hint="@string/notification_message_hint" />
    ```

2. В представлении проекта Android Studio разверните узел **App** > **src** > **main** > **res** > **values**. Откройте файл `strings.xml` и добавьте строковые параметры, на которые ссылаются новые элементы управления `Button` и `EditText`. Добавьте следующие строки в конец файла, непосредственно перед `</resources>`.

    ```xml
    <string name="send_button">Send Notification</string>
    <string name="notification_message_hint">Enter notification message text</string>
    ```

3. В файле `NotificationSetting.java` добавьте приведенный ниже параметр в класс `NotificationSettings`.

    Добавьте в `HubFullAccess` следующую строку подключения к центру: **DefaultFullSharedAccessSignature** . Чтобы скопировать эту строку подключения на [Портал Azure], щелкните **Политики доступа** на странице **Параметры** в центре уведомлений.

    ```java
    public static String HubFullAccess = "<Enter Your DefaultFullSharedAccess Connection string>";
    ```

4. В начале файла `MainActivity.java` добавьте следующие операторы `import`.

    ```java
    import java.io.BufferedOutputStream;
    import java.io.BufferedReader;
    import java.io.InputStreamReader;
    import java.io.OutputStream;
    import java.net.HttpURLConnection;
    import java.net.URL;
    import java.net.URLEncoder;
    import javax.crypto.Mac;
    import javax.crypto.spec.SecretKeySpec;
    import android.util.Base64;
    import android.view.View;
    import android.widget.EditText;
    ```

5. В файле `MainActivity.java` в начало класса `MainActivity` добавьте следующие переменные:

    ```java
    private String HubEndpoint = null;
    private String HubSasKeyName = null;
    private String HubSasKeyValue = null;
    ```

6. Создайте маркер подписанного URL-адреса для аутентификации запроса POST для отправки сообщений в центр уведомлений. Проанализируйте данные ключа из строки подключения и создайте токен SAS, как упоминалось в справочнике по [основным понятиям](/previous-versions/azure/reference/dn495627(v=azure.100)) REST API. Ниже приведен пример реализации.

    В файле `MainActivity.java` добавьте в класс `MainActivity` следующий метод, который анализирует строку подключения:

    ```java
    /**
        * Example code from https://msdn.microsoft.com/library/azure/dn495627.aspx
        * to parse the connection string so a SaS authentication token can be
        * constructed.
        *
        * @param connectionString This must be the DefaultFullSharedAccess connection
        *                         string for this example.
        */
    private void ParseConnectionString(String connectionString)
    {
        String[] parts = connectionString.split(";");
        if (parts.length != 3)
            throw new RuntimeException("Error parsing connection string: "
                    + connectionString);

        for (int i = 0; i < parts.length; i++) {
            if (parts[i].startsWith("Endpoint")) {
                this.HubEndpoint = "https" + parts[i].substring(11);
            } else if (parts[i].startsWith("SharedAccessKeyName")) {
                this.HubSasKeyName = parts[i].substring(20);
            } else if (parts[i].startsWith("SharedAccessKey")) {
                this.HubSasKeyValue = parts[i].substring(16);
            }
        }
    }
    ```

7. В файле `MainActivity.java` добавьте в класс `MainActivity` следующий метод, который создает маркер проверки подлинности SaS:

    ```java
    /**
        * Example code from https://msdn.microsoft.com/library/azure/dn495627.aspx to
        * construct a SaS token from the access key to authenticate a request.
        *
        * @param uri The unencoded resource URI string for this operation. The resource
        *            URI is the full URI of the Service Bus resource to which access is
        *            claimed. For example,
        *            "http://<namespace>.servicebus.windows.net/<hubName>"
        */
    private String generateSasToken(String uri) {

        String targetUri;
        String token = null;
        try {
            targetUri = URLEncoder
                    .encode(uri.toString().toLowerCase(), "UTF-8")
                    .toLowerCase();

            long expiresOnDate = System.currentTimeMillis();
            int expiresInMins = 60; // 1 hour
            expiresOnDate += expiresInMins * 60 * 1000;
            long expires = expiresOnDate / 1000;
            String toSign = targetUri + "\n" + expires;

            // Get an hmac_sha1 key from the raw key bytes
            byte[] keyBytes = HubSasKeyValue.getBytes("UTF-8");
            SecretKeySpec signingKey = new SecretKeySpec(keyBytes, "HmacSHA256");

            // Get an hmac_sha1 Mac instance and initialize with the signing key
            Mac mac = Mac.getInstance("HmacSHA256");
            mac.init(signingKey);

            // Compute the hmac on input data bytes
            byte[] rawHmac = mac.doFinal(toSign.getBytes("UTF-8"));

            // Using android.util.Base64 for Android Studio instead of
            // Apache commons codec
            String signature = URLEncoder.encode(
                    Base64.encodeToString(rawHmac, Base64.NO_WRAP).toString(), "UTF-8");

            // Construct authorization string
            token = "SharedAccessSignature sr=" + targetUri + "&sig="
                    + signature + "&se=" + expires + "&skn=" + HubSasKeyName;
        } catch (Exception e) {
            if (isVisible) {
                ToastNotify("Exception Generating SaS : " + e.getMessage().toString());
            }
        }

        return token;
    }
    ```

8. В файле `MainActivity.java` добавьте в класс `MainActivity` метод, который обрабатывает событие нажатия кнопки **Send Notification** (Отправить уведомление) и отправляет push-уведомление в Центр с помощью встроенного интерфейса REST API.

    ```java
    /**
        * Send Notification button click handler. This method parses the
        * DefaultFullSharedAccess connection string and generates a SaS token. The
        * token is added to the Authorization header on the POST request to the
        * notification hub. The text in the editTextNotificationMessage control
        * is added as the JSON body for the request to add a GCM message to the hub.
        *
        * @param v
        */
    public void sendNotificationButtonOnClick(View v) {
        EditText notificationText = (EditText) findViewById(R.id.editTextNotificationMessage);
        final String json = "{\"data\":{\"message\":\"" + notificationText.getText().toString() + "\"}}";

        new Thread()
        {
            public void run()
            {
                try
                {
                    // Based on reference documentation...
                    // https://msdn.microsoft.com/library/azure/dn223273.aspx
                    ParseConnectionString(NotificationSettings.HubFullAccess);
                    URL url = new URL(HubEndpoint + NotificationSettings.HubName +
                            "/messages/?api-version=2015-01");

                    HttpURLConnection urlConnection = (HttpURLConnection)url.openConnection();

                    try {
                        // POST request
                        urlConnection.setDoOutput(true);

                        // Authenticate the POST request with the SaS token
                        urlConnection.setRequestProperty("Authorization",
                            generateSasToken(url.toString()));

                        // Notification format should be GCM
                        urlConnection.setRequestProperty("ServiceBusNotification-Format", "gcm");

                        // Include any tags
                        // Example below targets 3 specific tags
                        // Refer to : https://azure.microsoft.com/documentation/articles/notification-hubs-routing-tag-expressions/
                        // urlConnection.setRequestProperty("ServiceBusNotification-Tags",
                        //        "tag1 || tag2 || tag3");

                        // Send notification message
                        urlConnection.setFixedLengthStreamingMode(json.length());
                        OutputStream bodyStream = new BufferedOutputStream(urlConnection.getOutputStream());
                        bodyStream.write(json.getBytes());
                        bodyStream.close();

                        // Get response
                        urlConnection.connect();
                        int responseCode = urlConnection.getResponseCode();
                        if ((responseCode != 200) && (responseCode != 201)) {
                            BufferedReader br = new BufferedReader(new InputStreamReader((urlConnection.getErrorStream())));
                            String line;
                            StringBuilder builder = new StringBuilder("Send Notification returned " +
                                    responseCode + " : ")  ;
                            while ((line = br.readLine()) != null) {
                                builder.append(line);
                            }

                            ToastNotify(builder.toString());
                        }
                    } finally {
                        urlConnection.disconnect();
                    }
                }
                catch(Exception e)
                {
                    if (isVisible) {
                        ToastNotify("Exception Sending Notification : " + e.getMessage().toString());
                    }
                }
            }
        }.start();
    }
    ```

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого руководства вы отправили широковещательные уведомления на все устройства Android, зарегистрированные в серверной части. Чтобы узнать, как отправлять push-уведомления на конкретные устройства Android, перейдите к следующему руководству:  

 > [!div class="nextstepaction"]
 > [Руководство по отправке push-уведомлений на конкретные устройства Android с помощью Центров уведомлений Azure и Google Cloud Messaging](notification-hubs-aspnet-backend-android-xplat-segmented-gcm-push-notification.md)

<!-- Images. -->
[6]: ./media/notification-hubs-android-get-started/notification-hub-android-new-class.png
[12]: ./media/notification-hubs-android-get-started/notification-hub-connection-strings.png
[13]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-new-project.png
[14]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-choose-form-factor.png
[15]: ./media/notification-hubs-android-get-started/notification-hub-create-android-app4.png
[16]: ./media/notification-hubs-android-get-started/notification-hub-create-android-app5.png
[17]: ./media/notification-hubs-android-get-started/notification-hub-create-android-app6.png
[18]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-registered.png
[19]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-set-message.png
[20]: ./media/notification-hubs-android-get-started/notification-hub-create-console-app.png
[21]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-received-message.png
[22]: ./media/notification-hubs-android-get-started/notification-hub-scheduler1.png
[23]: ./media/notification-hubs-android-get-started/notification-hub-scheduler2.png
[29]: ./media/mobile-services-android-get-started-push/mobile-eclipse-import-Play-library.png
[30]: ./media/notification-hubs-android-get-started/notification-hubs-debug-hub-gcm.png
[31]: ./media/notification-hubs-android-get-started/notification-hubs-android-studio-add-ui.png

<!-- URLs. -->
[Get started with push notifications in Mobile Services]: ../mobile-services-javascript-backend-android-get-started-push.md
[Mobile Services Android SDK]: https://go.microsoft.com/fwLink/?LinkID=280126&clcid=0x409
[Referencing a library project]: https://go.microsoft.com/fwlink/?LinkId=389800
[Notification Hubs Guidance]: /previous-versions/azure/azure-services/jj927170(v=azure.100)
[Use Notification Hubs to push notifications to users]: notification-hubs-aspnet-backend-gcm-android-push-to-user-google-notification.md
[Use Notification Hubs to send breaking news]: notification-hubs-aspnet-backend-android-xplat-segmented-gcm-push-notification.md
[Портал Azure]: https://portal.azure.com
