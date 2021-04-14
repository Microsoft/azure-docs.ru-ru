---
title: Включить файл
description: Включить файл
services: notification-hubs
author: spelluru
ms.service: notification-hubs
ms.topic: include
ms.date: 02/05/2019
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: cd202c98ed605209f5600965ecdb6c0b4c03c17e
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "67509096"
---
1. В Android Studio в меню выберите **Средства**, а затем — **Диспетчер пакетов SDK**. 
2. Выберите целевую версию пакета SDK для Android, который используется в проекте. Затем выберите **Show Package Details** (Показать сведения о пакете). 

    ![Диспетчер пакетов SDK Android: выбор целевой версии](./media/notification-hubs-android-studio-add-google-play-services/notification-hubs-android-studio-sdk-manager.png)
3. Выберите **Google APIs** (API-интерфейсы Google), если они еще не установлены.

    ![Диспетчер пакетов SDK для Android с выбранными API-интерфейсами Google](./media/notification-hubs-android-studio-add-google-play-services/googole-apis-selected.png)
4. Переключитесь на вкладку **SDK Tools**. Если сервисы Google Play еще не установлены, щелкните **Сервисы Google Play**, как показано на рисунке ниже. Затем выберите **Применить** для установки. Запишите путь к пакету SDK. Он вам потребуется в дальнейшем.

    ![Диспетчер пакетов SDK для Android с выбранными Сервисами Google Play](./media/notification-hubs-android-studio-add-google-play-services/google-play-services-selected.png)
3. Если вы видите диалоговое окно **Подтвердить изменение**, выберите **ОК**. Установщик компонентов устанавливает запрошенные компоненты. После установки компонентов выберите **Готово**.
4. Выберите **ОК**, чтобы закрыть диалоговое окно **Settings for New Projects** (Параметры для новых проектов).  
5. Откройте файл build.gradle в каталоге **app** и в разделе `dependencies` добавьте следующую строку. 

    ```gradle
    implementation 'com.google.android.gms:play-services-gcm:16.0.0'
    ```
5. Выберите значок **Синхронизировать сейчас** на панели инструментов.

    ![Синхронизация с Gradle](./media/notification-hubs-android-studio-add-google-play-services/gradle-sync.png)
1. Откройте файл AndroidManifest.xml, а затем добавьте следующий тег для тега *приложения*.

    ```xml
    <meta-data android:name="com.google.android.gms.version"
         android:value="@integer/google_play_services_version" />
    ```
