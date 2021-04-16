---
title: включить файл
description: включить файл
services: azure-communication-services
author: ddematheu2
manager: chpalm
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 9/1/2020
ms.topic: include
ms.custom: include file
ms.author: dademath
ms.openlocfilehash: c6e8be5462e0caffec7a1c88dae54f3f818ec323
ms.sourcegitcommit: bfa7d6ac93afe5f039d68c0ac389f06257223b42
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2021
ms.locfileid: "106498766"
---
[!INCLUDE [Public Preview Notice](../../includes/public-preview-include-android-ios.md)]


**Пример группового вызова для Android** Служб коммуникации Azure показывает, как с помощью пакета SDK для вызовов Служб коммуникации для Android можно создавать групповые вызовы с поддержкой голоса и видео. В этом кратком руководстве описывается, как настроить и запустить этот пример. Для понимания контекста приводятся общие сведения о примере.

## <a name="download-code"></a>Скачать код

Найдите проект для этого примера на сайте [GitHub](https://github.com/Azure-Samples/communication-services-android-calling-hero). Версию примера с [взаимодействием между командами](../../concepts/teams-interop.md) можно найти в отдельной [ветви](https://github.com/Azure-Samples/communication-services-android-calling-hero/tree/feature/teams_interop).

## <a name="overview"></a>Обзор

Этот пример содержит собственное приложение Android, которое использует пакеты SDK Служб коммуникации Azure для Android для создания вызовов с поддержкой голоса и видео. Это приложение использует компонент на стороне сервера для подготовки маркеров доступа, которые затем используются для инициализации пакета SDK Служб коммуникации Azure. Чтобы настроить этот компонент на стороне сервера, вы можете воспользоваться [учебником по созданию доверенной службы с Функциями Azure](../../tutorials/trusted-service-tutorial.md).

Вот как выглядит этот пример:

:::image type="content" source="../media/calling/landing-page-android.png" alt-text="Снимок экрана: целевая страница примера приложения.":::

При нажатии кнопки Start new call (Начать новый вызов) приложение Android создает новый вызов и присоединяется к нему. Приложение также позволяет подключиться к существующему вызову Служб коммуникации Azure, указав идентификатор этого вызова.

После подключения к вызову вам будет предложено предоставить приложению разрешение на доступ к камере и микрофону. Также будет предложено ввести отображаемое имя.

:::image type="content" source="../media/calling/pre-call-android.png" alt-text="Снимок экрана: экран примера приложения перед звонком.":::

Завершив настройку устройств и отображаемого имени, вы присоединитесь к сеансу вызова. Вы увидите основной холст вызова с основными возможностями.

:::image type="content" source="../media/calling/main-app-android.png" alt-text="Снимок экрана: главный экран примера приложения.":::

Ниже перечислены компоненты главного экрана вызова.

- **Коллекция мультимедиа.** Основная область, в которой отображаются участники. Если у участника есть камера и она включена, здесь отображается видеопоток. Для каждого участника создается отдельная плитка, на которой размещаются его отображаемое имя и видеопоток (если он есть). Коллекция поддерживает несколько участников и автоматически обновляется при добавлении или удалении участников вызова.
- **Панель действий.** Именно здесь находятся основные элементы управления вызовом. Эти элементы управления позволяют включать и отключать видео и микрофон, предоставлять общий доступ к экрану и покидать сеанс вызова.

Ниже вы найдете дополнительные сведения о предварительных требованиях и шагах по настройке примера.

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. Дополнительные сведения см. на странице [Создайте бесплатную учетную запись Azure уже сегодня](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Android Studio](https://developer.android.com/studio), установленное на вашем компьютере
- Ресурс Служб коммуникации Azure. Дополнительные сведения см. в статье [Краткое руководство по созданию ресурсов Служб коммуникации и управлению ими](../../quickstarts/create-communication-resource.md).
- Функция Azure, запускающая [конечную точку проверки подлинности](../../tutorials/trusted-service-tutorial.md) для получения маркеров доступа.

## <a name="running-sample-locally"></a>Локальное выполнение примера

Этот пример группового вызова можно запустить локально с помощью Android Studio. Разработчики могут использовать для тестирования приложения реальное устройство или эмулятор.

### <a name="before-running-the-sample-for-the-first-time"></a>Действия перед первым запуском примера

1. Запустите Android Studio и выберите `Open an Existing Project`
2. Откройте папку `AzureCalling` в загруженном выпуске для примера.
3. Разверните ресурс или приложение, чтобы обновить `appSettings.properties`. Задайте для ключа `communicationTokenFetchUrl` значение, соответствующее URL-адресу конечной точки проверки подлинности.

### <a name="run-sample"></a>Запуск примера

Откройте пример в Android Studio.

## <a name="optional-securing-an-authentication-endpoint"></a>(Необязательно) Защита конечной точки аутентификации

В демонстрационных целях в этом примере по умолчанию используется общедоступная конечная точка для получения маркера Служб коммуникации Azure. Для реальных рабочих сценариев мы рекомендуем создать собственную защищенную конечную точку и выдавать собственные маркеры.

С небольшой дополнительной настройкой этот пример поддерживает подключение к конечной точке с защитой **Azure Active Directory** (Azure AD), чтобы для получения маркера Служб коммуникации Azure требовался вход пользователя в систему. Соответствующие действия описаны ниже.

1. Включите аутентификацию Azure Active Directory в приложении.  
   - [Зарегистрируйте приложение в Azure Active Directory (используя параметры платформ Android)](../../../active-directory/develop/tutorial-v2-android.md) 
    - [Настройка Службы приложений или приложения "Функции Azure" для использования имени для входа в Azure AD](../../../app-service/configure-authentication-provider-aad.md)

2. Перейдите на страницу обзора зарегистрированного приложения в разделе регистрации приложений в Azure Active Directory. Запишите значения `Package name`, `Signature hash`, `MSAL Configutaion`.

:::image type="content" source="../media/calling/aad-overview-android.png" alt-text="Настройка Azure Active Directory на портале Azure.":::

3. Измените `AzureCalling/app/src/main/res/raw/auth_config_single_account.json` и задайте `isAADAuthEnabled`, чтобы включить Azure Active Directory
4. Измените `AndroidManifest.xml` и задайте `android:path` для хэша подписи хранилища ключей. (Необязательно. Текущее значение использует хэш из упакованного debug.keystore. Если используется другое хранилище ключей, его необходимо обновить.)
   ```
   <activity android:name="com.microsoft.identity.client.BrowserTabActivity">
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />
                <data
                    android:host="com.azure.samples.communication.calling"
                    android:path="/Signature hash" <!-- do not remove /. The current hash in AndroidManifest.xml is for debug.keystore. -->
                    android:scheme="msauth" />
            </intent-filter>
        </activity>
   ```
5. Скопируйте конфигурацию Android MSAL из портала Azure и вставьте в `AzureCalling/app/src/main/res/raw/auth_config_single_account.json`. Добавьте "account_mode": "SINGLE"
   ```
      {
         "client_id": "",
         "authorization_user_agent": "DEFAULT",
         "redirect_uri": "",
         "account_mode" : "SINGLE",
         "authorities": [
            {
               "type": "AAD",
               "audience": {
               "type": "AzureADMyOrg",
               "tenant_id": ""
               }
            }
         ]
      }
   ```

6. Измените `AzureCalling/app/src/main/res/raw/auth_config_single_account.json` и задайте для ключа `communicationTokenFetchUrl` значение, соответствующее URL-адресу конечной точки проверки подлинности.
7. Измените `AzureCalling/app/src/main/res/raw/auth_config_single_account.json` и задайте значение для ключа `aadScopes` из областей `Expose an API` `Azure Active Directory`

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите очистить и удалить подписку на Службы коммуникации, вы можете удалить ресурс или группу ресурсов. При этом удаляются все ресурсы, связанные с ней. См. сведения об [очистке ресурсов](../../quickstarts/create-communication-resource.md#clean-up-resources).

## <a name="next-steps"></a>Дальнейшие действия

>[!div class="nextstepaction"]
>[Скачайте пример с GitHub.](https://github.com/Azure-Samples/communication-services-android-calling-hero)

Дополнительные сведения см. в следующих статьях:

- Узнайте, как правильно [использовать пакет SDK для вызовов](../../quickstarts/voice-video-calling/calling-client-samples.md).
- Узнайте больше о [принципе работы функции вызовов](../../concepts/voice-video-calling/about-call-types.md).

### <a name="additional-reading"></a>Дополнительные материалы

- [Службы коммуникации Azure (GitHub)](https://github.com/Azure/communication) — дополнительные примеры и сведения на официальной странице GitHub
- [Примеры](./../overview.md): дополнительные примеры см. на нашей странице обзора примеров.
- [Функции вызовов Служб коммуникации Azure](https://docs.microsoft.com/azure/communication-services/concepts/voice-video-calling/calling-sdk-features). Дополнительные сведения о пакете SDK для Android см. в статье о [пакетах SDK для Android для вызовов в Службе коммуникации Azure](https://search.maven.org/artifact/com.azure.android/azure-communication-calling)
