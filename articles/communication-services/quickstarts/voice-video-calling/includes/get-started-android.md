---
title: Краткое руководство. Добавление вызова VOIP к приложению Android с помощью Служб коммуникации Azure
description: Из этого руководства вы узнаете, как использовать пакет SDK Служб коммуникации Azure для вызовов для Android.
author: chpalm
ms.author: mikben
ms.date: 03/10/2021
ms.topic: quickstart
ms.service: azure-communication-services
ms.openlocfilehash: 38194ed7290d6cd9c4889d27ff458f950603c5be
ms.sourcegitcommit: edc7dc50c4f5550d9776a4c42167a872032a4151
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "106073277"
---
Из этого краткого руководства вы узнаете, как начать вызов с помощью пакета SDK Служб коммуникации Azure для вызовов для Android.

[!INCLUDE [Public Preview Notice](../../../includes/public-preview-include-android-ios.md)]

> [!NOTE]
> В этом документе используется пакет SDK для вызовов версии 1.0.0-beta.8.

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

Выберите минимальную версию пакета SDK: "API 26: Android 8.0 (Oreo)" или более позднюю.

:::image type="content" source="../media/android/studio-calling-min-api.png" alt-text="Снимок экрана с экраном шаблона проекта, где выбран вариант Empty Activity (Пустое действие) (2).":::


### <a name="install-the-package"></a>Установка пакета

<!-- TODO: update with instructions on how to download, install and add package to project -->
Выберите build.gradle на уровне проекта и добавьте `mavenCentral()` в список репозиториев в разделах `buildscript` и `allprojects`.
```groovy
buildscript {
    repositories {
    ...
        mavenCentral()
    ...
    }
}
```

```groovy
allprojects {
    repositories {
    ...
        mavenCentral()
    ...
    }
}
```
Затем добавьте в build.gradle на уровне модуля следующие строки в разделах dependencies и android.

```groovy
android {
    ...
    packagingOptions {
        pickFirst  'META-INF/*'
    }
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}

dependencies {
    ...
    implementation 'com.azure.android:azure-communication-calling:1.0.0-beta.8'
    ...
}
```

### <a name="add-permissions-to-application-manifest"></a>Добавление разрешений в манифест приложения

Чтобы запросить разрешения, необходимые для выполнения вызова, их нужно сначала объявить в манифесте приложения (`app/src/main/AndroidManifest.xml`). Замените содержимое этого файла приведенным ниже:

```xml
    <?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.contoso.acsquickstart">

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.RECORD_AUDIO" />
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <!--Our Calling SDK depends on the Apache HTTP SDK.
When targeting Android SDK 28+, this library needs to be explicitly referenced.
See https://developer.android.com/about/versions/pie/android-9.0-changes-28#apache-p-->
        <uses-library android:name="org.apache.http.legacy" android:required="false"/>
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
    
```

### <a name="set-up-the-layout-for-the-app"></a>Настройка макета для приложения

Нам нужны два элемента: текстовое поле для идентификатора вызываемого участника и кнопка для начала вызова. Их можно добавить с помощью конструктора или прямым редактированием XML-документа макета. Создайте кнопку с идентификатором `call_button` и текстовое поле `callee_id`. Перейдите к `app/src/main/res/layout/activity_main.xml` и замените содержимое этого файла приведенным ниже:

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <Button
        android:id="@+id/call_button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="16dp"
        android:text="Call"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent" />

    <EditText
        android:id="@+id/callee_id"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:ems="10"
        android:hint="Callee Id"
        android:inputType="textPersonName"
        app:layout_constraintBottom_toTopOf="@+id/call_button"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />
</androidx.constraintlayout.widget.ConstraintLayout>
```

### <a name="create-the-main-activity-scaffolding-and-bindings"></a>Создание шаблонов и привязок для основных событий

После создания макета вы можете добавить в действие привязки и основные шаблоны. Действие будет обрабатывать запросы разрешений в среде выполнения, создавать агент вызова и выполнять вызов при нажатии кнопки. Каждый из этих элементов рассматривается в отдельном разделе. Мы переопределим метод `onCreate`, добавив вызов `getAllPermissions` и `createAgent`, а также привязки для кнопки вызова. Это будет происходить только один раз при создании действия. Дополнительные сведения о `onCreate` см. в руководстве [Сведения о жизненном цикле действий](https://developer.android.com/guide/components/activities/activity-lifecycle).

Перейдите к файлу **MainActivity.java** и замените его содержимое следующим кодом:

```java
package com.contoso.acsquickstart;

import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;

import android.Manifest;
import android.content.pm.PackageManager;
import android.os.Bundle;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;

import com.azure.android.communication.common.CommunicationUserIdentifier;
import com.azure.android.communication.common.CommunicationTokenCredential;
import com.azure.android.communication.calling.CallAgent;
import com.azure.android.communication.calling.CallClient;
import com.azure.android.communication.calling.StartCallOptions;


import java.util.ArrayList;

public class MainActivity extends AppCompatActivity {
    
    private CallAgent callAgent;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        getAllPermissions();
        createAgent();
        
        // Bind call button to call `startCall`
        Button callButton = findViewById(R.id.call_button);
        callButton.setOnClickListener(l -> startCall());
    }

    /**
     * Request each required permission if the app doesn't already have it.
     */
    private void getAllPermissions() {
        // See section on requesting permissions
    }

    /**
      * Create the call agent for placing calls
      */
    private void createAgent() {
        // See section on creating the call agent
    }

    /**
     * Place a call to the callee id provided in `callee_id` text input.
     */
    private void startCall() {
        // See section on starting the call
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
    String[] requiredPermissions = new String[]{Manifest.permission.RECORD_AUDIO, Manifest.permission.CAMERA, Manifest.permission.WRITE_EXTERNAL_STORAGE, Manifest.permission.READ_PHONE_STATE};
    ArrayList<String> permissionsToAskFor = new ArrayList<>();
    for (String permission : requiredPermissions) {
        if (ActivityCompat.checkSelfPermission(this, permission) != PackageManager.PERMISSION_GRANTED) {
            permissionsToAskFor.add(permission);
        }
    }
    if (!permissionsToAskFor.isEmpty()) {
        ActivityCompat.requestPermissions(this, permissionsToAskFor.toArray(new String[0]), 1);
    }
}
```

> [!NOTE]
> При проектировании приложения учитывайте, когда будут запрошены такие разрешения. Разрешения следует запрашивать по мере необходимости, а не заранее. Дополнительные сведения см. в [руководстве по разрешениям Android](https://developer.android.com/training/permissions/requesting).

## <a name="object-model"></a>Объектная модель

Следующие классы и интерфейсы реализуют некоторые основные функции пакета SDK Служб коммуникации Azure для вызовов:

| Имя                                  | Описание                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| CallClient| CallClient — это основная точка входа в пакет SDK для вызовов.|
| CallAgent | CallAgent используется для инициирования вызовов и управления ими. |
| CommunicationUserCredential | CommunicationUserCredential используется в качестве учетных данных маркера для создания экземпляра CallAgent.|
| CommunicationIdentifier | CommunicationIdentifier используется в качестве участника другого типа, который может быть частью вызова.|

## <a name="create-an-agent-from-the-user-access-token"></a>Создание агента на основе маркера доступа пользователя

Наличие маркера пользователя позволяет создать экземпляр агента вызова с пройденной аутентификацией. Как правило, этот маркер создается в службе, которая отвечает за аутентификацию для приложения. Дополнительные сведения о маркерах доступа пользователей см. в [этом руководстве](../../access-tokens.md). Для целей этого краткого руководства замените `<User_Access_Token>` маркером доступа пользователя, который был создан для вашего ресурса Службы коммуникации Azure.

```java

/**
 * Create the call agent for placing calls
 */
private void createAgent() {
    String userToken = "<User_Access_Token>";

    try {
        CommunicationTokenCredential credential = new CommunicationTokenCredential(userToken);
        callAgent = new CallClient().createCallAgent(getApplicationContext(), credential).get();
    } catch (Exception ex) {
        Toast.makeText(getApplicationContext(), "Failed to create call agent.", Toast.LENGTH_SHORT).show();
    }
}

```

## <a name="start-a-call-using-the-call-agent"></a>Инициирование вызова с помощью агента вызовов

Вызов можно выполнить через агент вызова — для этого достаточно предоставить список идентификаторов вызываемых участников и параметры вызова. Для примера в этом кратком руководстве используются параметры вызова по умолчанию, без поддержки видео, и один идентификатор вызываемого участника из текстового поля.

```java
/**
 * Place a call to the callee id provided in `callee_id` text input.
 */
private void startCall() {
    EditText calleeIdView = findViewById(R.id.callee_id);
    String calleeId = calleeIdView.getText().toString();
    
    StartCallOptions options = new StartCallOptions();

    callAgent.startCall(
        getApplicationContext(),
        new CommunicationUserIdentifier[] {new CommunicationUserIdentifier(calleeId)},
        options);
}
```


## <a name="launch-the-app-and-call-the-echo-bot"></a>Запуск приложения и вызов эхо-бота

Теперь вы можете запустить приложение с помощью кнопки Run App (Запустить приложение) на панели инструментов или нажав клавиши SHIFT+F10. Убедитесь, что вы можете выполнять вызовы, инициировав вызов к `8:echo123`. В ответ вы должны услышать предварительно записанное сообщение и повтор сказанного вами сообщения.

:::image type="content" source="../media/android/quickstart-android-call-echobot.png" alt-text="Снимок экрана с готовым приложением.":::

## <a name="sample-code"></a>Пример кода

Пример приложения можно скачать в репозитории [GitHub](https://github.com/Azure-Samples/communication-services-android-quickstarts/tree/main/Add%20Voice%20Calling).
