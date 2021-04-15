---
title: Краткое руководство. Добавление возможности присоединения к собранию Teams к приложению Android с помощью Служб коммуникации Azure
description: Из этого краткого руководства вы узнаете, как использовать библиотеку Teams Embed Служб коммуникации Azure для Android.
author: palatter
ms.author: palatter
ms.date: 01/25/2021
ms.topic: quickstart
ms.service: azure-communication-services
ms.openlocfilehash: 5ac4c53550468d33e9ed533303749d29e772d766
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105108487"
---
В этом кратком руководстве описывается, как присоединиться к собранию Teams с помощью библиотеки Teams Embed Служб коммуникации Azure для Android.

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- [Android Studio](https://developer.android.com/studio) для создания приложения Android.
- Развернутый ресурс Служб коммуникации. [Создайте ресурс Служб коммуникации.](../../create-communication-resource.md)
- [Маркер доступа пользователя](../../access-tokens.md) для Службы коммуникации Azure.

## <a name="setting-up"></a>Настройка

### <a name="create-an-android-app-with-an-empty-activity"></a>Создание приложения Android с пустым действием

В Android Studio щелкните Start a new Android Studio project (Создать проект Android Studio).

:::image type="content" source="../media/android/studio-new-project.png" alt-text="Снимок экрана с выбранной кнопкой создания нового проекта в Android Studio.":::

Выберите шаблон проекта Empty Activity (Пустое действие) из раздела Phone and Tablet (Телефон и планшет).

:::image type="content" source="../media/android/studio-blank-activity.png" alt-text="Снимок экрана с экраном шаблона проекта, где выбран вариант Empty Activity (Пустое действие).":::

Присвойте проекту имя `TeamsEmbedAndroidGettingStarted`, задайте для параметра language значение java и выберите минимальный пакет "API 21: Android 5.0 (Lollipop)" или выше.

:::image type="content" source="../media/android/studio-calling-min-api.png" alt-text="Снимок экрана с экраном шаблона проекта, где выбран вариант Empty Activity (Пустое действие) (2).":::


### <a name="install-the-azure-package"></a>Установка пакета Azure

Добавьте в build.gradle на уровне приложения следующие строки в разделах dependencies и android.

```groovy
android {
    ...
    packagingOptions {
        pickFirst  'META-INF/*'
    }
}
```

```groovy
dependencies {
    ...
    implementation 'com.azure.android:azure-communication-common:1.0.0-beta.6'
    ...
}
```

### <a name="install-the-teams-embed-package"></a>Установка пакета Teams Embed

Скачайте пакет `MicrosoftTeamsSDK`.

Затем распакуйте папку MicrosoftTeamsSDK в папку приложений проектов. Например: `TeamsEmbedAndroidGettingStarted/app/MicrosoftTeamsSDK`.

### <a name="add-teams-embed-package-to-your-buildgradle"></a>Добавление пакета Teams Embed в build.gradle

В конце файла в `build.gradle` на уровне приложения добавьте следующую строку:

```groovy
apply from: 'MicrosoftTeamsSDK/MicrosoftTeamsSDK.gradle'
```

Синхронизируйте проект с файлами gradle.

### <a name="create-application-class"></a>Создание класса приложения

Создайте новый файл класса Java с именем `TeamsEmbedAndroidGettingStarted`. Это будет класс приложения, который должен расширять `TeamsSDKApplication`. [Документация по Android](https://developer.android.com/reference/android/app/Application)

:::image type="content" source="../media/android/application-class-location.png" alt-text="Снимок экрана: где создать класс приложения в Android Studio":::

```java
package com.microsoft.teamsembedandroidgettingstarted;

import com.microsoft.teamssdk.app.TeamsSDKApplication;

public class TeamsEmbedAndroidGettingStarted extends TeamsSDKApplication {
}
```

### <a name="update-themes"></a>Обновление тем

Задайте имя стиля `AppTheme` в файлах `theme.xml` и `theme.xml (night)`.

:::image type="content" source="../media/android/theme-settings.png" alt-text="Снимок экрана: файлы theme.xml в Android Studio":::

```xml
<style name="AppTheme" parent="Theme.AppCompat.DayNight.DarkActionBar">
```

```xml
<style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
```

### <a name="add-permissions-to-application-manifest"></a>Добавление разрешений в манифест приложения

Добавьте разрешение `RECORD_AUDIO` в манифест приложения (`app/src/main/AndroidManifest.xml`):  


```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.yourcompany.TeamsEmbedAndroidGettingStarted">
    <uses-permission android:name="android.permission.RECORD_AUDIO"/>
    <application
```

### <a name="add-app-name-and-theme-to-application-manifest"></a>Добавление имени и темы приложения в манифест приложения

Добавьте `xmlns:tools="http://schemas.android.com/tools" в манифест.

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    package="com.yourcompany.TeamsEmbedAndroidGettingStarted">
```

Добавьте `.TeamsEmbedAndroidGettingStarted` в `android:name`, `android:name` в `tools:replace` и измените `android:theme` на `@style/AppTheme`

```xml
<application
    android:name=".TeamsEmbedAndroidGettingStarted"
    tools:replace="android:name"
    android:theme="@style/AppTheme"
    android:allowBackup="true"
    android:icon="@mipmap/ic_launcher"
    android:label="@string/app_name"
    android:roundIcon="@mipmap/ic_launcher_round"
    android:supportsRtl="true">
```

### <a name="set-up-the-layout-for-the-app"></a>Настройка макета для приложения

Создайте кнопку с ИД `join_meeting`. Перейдите к `app/src/main/res/layout/activity_main.xml` и замените содержимое этого файла приведенным ниже:

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/join_meeting"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Join Meeting"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />
</androidx.constraintlayout.widget.ConstraintLayout>
```

### <a name="create-the-main-activity-scaffolding-and-bindings"></a>Создание шаблонов и привязок для основных событий

После создания макета можно добавить основной процесс формирования шаблонов для действия, а также необходимые привязки. Действие будет обрабатывать запросы разрешений в среде выполнения, создавать клиент собрания и присоединять к собранию при нажатии кнопки. Каждый из этих элементов рассматривается в отдельном разделе. 

Мы переопределим метод `onCreate`, добавив вызов `getAllPermissions` и `createAgent`, а также привязки для кнопки `Join Meeting`. Это будет происходить только один раз при создании действия. Дополнительные сведения о `onCreate` см. в руководстве [Сведения о жизненном цикле действий](https://developer.android.com/guide/components/activities/activity-lifecycle).

Перейдите к файлу **MainActivity.java** и замените его содержимое следующим кодом:

```java
package com.yourcompany.teamsembedandroidgettingstarted;

import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;

import android.Manifest;
import android.content.pm.PackageManager;
import android.os.Bundle;
import android.widget.Button;
import android.widget.Toast;

import com.azure.android.communication.common.CommunicationTokenCredential;
import com.azure.android.communication.common.CommunicationTokenRefreshOptions;
import com.azure.android.communication.ui.meetings.MeetingJoinOptions;
import com.azure.android.communication.ui.meetings.MeetingUIClient;

import java.util.ArrayList;
import java.util.concurrent.Callable;

public class MainActivity extends AppCompatActivity {

    private final String displayName = "John Smith";

    private MeetingUIClient meetingUIClient;
    private MeetingJoinOptions meetingJoinOptions;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        meetingJoinOptions = new MeetingJoinOptions(displayName);
        
        getAllPermissions();
        createMeetingClient();

        Button joinMeeting = findViewById(R.id.join_meeting);
        joinMeeting.setOnClickListener(l -> joinMeeting());
    }

    private void createMeetingClient() {
        // See section on creating meeting client
    }

    private void joinMeeting() {
        // See section on joining a meeting
    }

    private void getAllPermissions() {
        // See section on getting permissions
    }
}
```

### <a name="request-permissions-at-runtime"></a>Запрос разрешений во время выполнения

В Android 6.0 и более поздних версий (API уровня 23) или `targetSdkVersion` версии 23 или более поздней разрешения предоставляются не при установке приложения, а во время выполнения. С целью поддержки этого механизма можно реализовать `getAllPermissions` для вызова `ActivityCompat.checkSelfPermission` и `ActivityCompat.requestPermissions` для каждого необходимого разрешения.

```java
/**
 * Request each required permission if the app doesn't already have it.
 */
private void getAllPermissions() {
    String[] requiredPermissions = new String[]{Manifest.permission.RECORD_AUDIO};
    ArrayList<String> permissionsToAskFor = new ArrayList<>();
    for (String permission : requiredPermissions) {
        if (ActivityCompat.checkSelfPermission(this, permission) != PackageManager.PERMISSION_GRANTED) {
            permissionsToAskFor.add(permission);
        }
    }
    if (!permissionsToAskFor.isEmpty()) {
        ActivityCompat.requestPermissions(this, permissionsToAskFor.toArray(new String[0]), 202);
    }
}
```

> [!NOTE]
> При проектировании приложения учитывайте, когда будут запрошены такие разрешения. Разрешения следует запрашивать по мере необходимости, а не заранее. Дополнительные сведения см. в [руководстве по разрешениям Android](https://developer.android.com/training/permissions/requesting).

## <a name="object-model"></a>Объектная модель

Следующие классы и интерфейсы реализуют некоторые основные функции Teams Embed Служб коммуникации Azure:

| Имя                                  | Описание                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| MeetingUIClient| MeetingUIClient — это главная точка входа в библиотеку Teams Embed. |
| MeetingJoinOptions | MeetingJoinOptions используются для настраиваемых параметров, таких как отображаемое имя. |
| CallState | CallState используется для создания отчетов об изменении состояния вызова. Существуют следующие состояния: `connecting`, `waitingInLobby`, `connected` и `ended`. |

## <a name="create-a-meetingclient-from-the-user-access-token"></a>Создание MeetingClient на основе маркера доступа пользователя

Для клиента собрания, прошедшего проверку подлинности, можно создать экземпляр с маркером доступа пользователя. Этот маркер обычно создается службой с применением проверки подлинности, характерной для приложения. Дополнительные сведения о маркерах доступа пользователей см. в руководстве [Маркеры доступа пользователя](../../access-tokens.md). Для целей этого краткого руководства замените `<USER_ACCESS_TOKEN>` маркером доступа пользователя, который был создан для вашего ресурса Службы коммуникации Azure.

```java
private void createMeetingClient() {
    try {
        CommunicationTokenRefreshOptions refreshOptions = new CommunicationTokenRefreshOptions(tokenRefresher, true, "<USER_ACCESS_TOKEN>");
        CommunicationTokenCredential credential = new CommunicationTokenCredential(refreshOptions);
        meetingUIClient = new MeetingUIClient(credential);
    } catch (Exception ex) {
        Toast.makeText(getApplicationContext(), "Failed to create meeting client: " + ex.getMessage(), Toast.LENGTH_SHORT).show();
    }
}
```

## <a name="setup-token-refreshing"></a>Обновление маркера установки

Создание вызываемого метода `tokenRefresher`. Затем создайте метод `fetchToken`, чтобы получить маркер пользователя. [Описание процедуры см. здесь](https://docs.microsoft.com/azure/communication-services/quickstarts/access-tokens?pivots=programming-language-java)

```java
Callable<String> tokenRefresher = () -> {
    return fetchToken();
};

public String fetchToken() {
    // Get token
    return USER_ACCESS_TOKEN;
}
```

## <a name="get-the-teams-meeting-link"></a>Получение ссылки на собрание Teams

Ссылку на собрание Teams можно получить с помощью интерфейсов API Graph. Это действие подробно описано в [документации по Graph](/graph/api/onlinemeeting-createorget?tabs=http&view=graph-rest-beta&preserve-view=true).
Пакет SDK вызовов Служб коммуникации принимает полную ссылку на собрание Teams. Эта ссылка возвращается как часть ресурса `onlineMeeting`, доступного в [свойстве `joinWebUrl`](/graph/api/resources/onlinemeeting?view=graph-rest-beta&preserve-view=true). Кроме того, вы можете получить необходимые сведения о собрании, воспользовавшись URL-адресом для **участия в собрании** в самом приглашении на собрание Teams.

## <a name="start-a-meeting-using-the-meeting-client"></a>Запуск собрания с помощью клиента собрания

Присоединиться к собранию можно с помощью `MeetingClient`. Для этого требуются `meetingURL` и `JoinOptions`. Замените `<MEETING_URL>` на URL-адрес собрания в Teams.

```java
/**
 * Join a meeting with a meetingURL.
 */
private void joinMeeting() {
    try {
        meetingUIClient.join("<MEETING_URL>", meetingJoinOptions);
    } catch (Exception ex) {
        Toast.makeText(getApplicationContext(), "Failed to join meeting: " + ex.getMessage(), Toast.LENGTH_SHORT).show();
    }
}
```

## <a name="launch-the-app-and-join-a-meeting"></a>Запуск приложения и присоединение к собранию

Теперь вы можете запустить приложение с помощью кнопки Run App (Запустить приложение) на панели инструментов или нажав клавиши SHIFT+F10. 

:::image type="content" source="../media/android/quickstart-android-join-meeting.png" alt-text="Снимок экрана с готовым приложением.":::

:::image type="content" source="../media/android/quickstart-android-joined-meeting.png" alt-text="Снимок экрана: готовое приложение после присоединения к собранию.":::

## <a name="sample-code"></a>Пример кода

Пример приложения можно скачать в репозитории [GitHub](https://github.com/Azure-Samples/teams-embed-android-getting-started).
