---
title: Режим совместной работы с устройством iOS
titleSuffix: Microsoft identity platform | Azure
description: Узнайте, как включить режим общего устройства, чтобы разрешить оказался работникам совместно использовать устройство iOS.
services: active-directory
author: brandwe
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 03/31/2020
ms.author: brandwe
ms.reviewer: brandwe
ms.custom: aaddev
ms.openlocfilehash: c67c5d7b46c04e7f1aea020127ee798878c43d60
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104578793"
---
# <a name="shared-device-mode-for-ios-devices"></a>Режим совместной работы с устройством iOS

>[!IMPORTANT]
> Эта функция [!INCLUDE [PREVIEW BOILERPLATE](../../../includes/active-directory-develop-preview.md)]

Оказалсяные работники, такие как розничные организации, участники рейсов, а также сотрудники службы "поле" часто используют общее мобильное устройство для выполнения своих задач. Эти общие устройства могут представлять угрозу безопасности, если пользователи совместно используют свои пароли или ПИН-коды, намеренно или нет для доступа к данным клиента и бизнеса на общем устройстве.

Режим общего устройства позволяет настроить устройство iOS 13 или более высокого уровня для более простого и безопасного совместного использования сотрудниками. Сотрудники могут быстро входить в систему и получать доступ к информации о клиентах. Когда они закончат свою смену или задачу, они могут выйти из устройства и сразу же приступать к использованию следующим сотрудником.

Режим общего устройства также обеспечивает управление устройством с помощью обратной идентификации Майкрософт.

Эта функция использует [приложение Microsoft Authenticator](../user-help/user-help-auth-app-overview.md) для управления пользователями на устройстве и распространения [ПОДКЛЮЧАЕМОГО модуля единого входа Microsoft Enterprise для устройств Apple](apple-sso-plugin.md).

## <a name="create-a-shared-device-mode-app"></a>Создание приложения в режиме общего устройства

Чтобы создать приложение в режиме общего устройства, разработчики и администраторы облачных устройств работают вместе:

1. **Разработчики приложений** пишут приложения с одной учетной записью (приложения с несколькими учетными записями не поддерживаются в режиме общего устройства) и пишут код для выполнения таких операций, как выход из общего устройства.

1. **Администраторы устройств** подготавливают устройство к совместному использованию с помощью поставщика управления мобильными устройствами (MDM), например Microsoft Intune для управления устройствами в Организации. MDM отправляет Microsoft Authenticator приложение на устройства и включает "общий режим" для каждого устройства через обновление профиля на устройстве. Этот параметр общего режима изменяет поведение поддерживаемых приложений на устройстве. Эта конфигурация от поставщика MDM задает режим общего устройства для устройства и включает [подключаемый модуль единого входа Microsoft Enterprise для устройств Apple](apple-sso-plugin.md) , которые необходимы для режима общего устройства.

1. [**Требуется только во время общедоступной предварительной версии**] Пользователь с ролью " [Администратор облачного устройства](../roles/permissions-reference.md#cloud-device-administrator) " должен запустить [приложение Microsoft Authenticator](../user-help/user-help-auth-app-overview.md) и присоединить его устройство к Организации.

    Настройка членства в ролях Организации в портал Azure: **Azure Active Directory**  >  **роли и администраторы**  >  **облачных устройств администратора**

Следующие разделы содержат сведения о том, как обновить приложение для поддержки режима общего устройства.

## <a name="use-intune-to-enable-shared-device-mode--sso-extension"></a>Использование Intune для включения режима общего устройства & расширение SSO

> [!NOTE]
> Следующий шаг требуется только во время общедоступной предварительной версии.

Устройство должно быть настроено для поддержки режима общего устройства. На нем должен быть установлен iOS 13 + и регистрация MDM. Для настройки MDM также необходимо включить [подключаемый модуль единого входа Microsoft Enterprise для устройств Apple](apple-sso-plugin.md). Дополнительные сведения о расширениях единого входа см. в [видеоролике Apple](https://developer.apple.com/videos/play/tech-talks/301/).

1. На портале настройки Intune укажите устройство для включения [подключаемого модуля единого входа Microsoft Enterprise для устройств Apple](apple-sso-plugin.md) со следующей конфигурацией:

    - **Тип**: перенаправление
    - **Идентификатор расширения**: com. Microsoft. азуреаусентикатор. ссоекстенсион
    - **Идентификатор команды**: (это поле не требуется для iOS)
    - **URL-адреса**:   
        - `https://login.microsoftonline.com`
        - `https://login.microsoft.com`
        - `https://sts.windows.net`
        - `https://login.partner.microsoftonline.cn`
        - `https://login.chinacloudapi.cn`
        - `https://login.microsoftonline.de`
        - `https://login.microsoftonline.us`
        - `https://login.usgovcloudapi.net`
        - `https://login-us.microsoftonline.com`
    - **Дополнительные данные для настройки**:
      - Ключ: Шареддевицемоде
      - Тип: логический
      - Значение: true

    Дополнительные сведения о настройке с помощью Intune см. в [документации по конфигурации Intune](/intune/configuration/ios-device-features-settings).

1. Затем настройте MDM для отправки приложения Microsoft Authenticator на устройство через профиль MDM.

    Задайте следующие параметры конфигурации, чтобы включить режим общего устройства:

    - Конфигурация 1.
      - Ключ: Шареддевицемоде
      - Тип: логический
      - Значение: true

## <a name="modify-your-ios-application-to-support-shared-device-mode"></a>Изменение приложения iOS для поддержки режима общего устройства

Ваши пользователи зависят от вас, чтобы гарантировать, что их данные не будут утечки другому пользователю. В следующих разделах приводятся полезные сигналы, указывающие, что в приложение произошло изменение и должно быть обработано.

Вы несете ответственность за проверку состояния пользователя на устройстве каждый раз, когда используется приложение, а затем очищаете данные предыдущего пользователя. Сюда входит, если она перезагружена из фона при многозадачности.

При изменении пользователя необходимо убедиться в том, что данные предыдущего пользователя удалены и все кэшированные данные, отображаемые в приложении, удаляются. Мы настоятельно рекомендуем вам и вашей компании провести процесс проверки безопасности после обновления приложения для поддержки общего режима устройства.

### <a name="detect-shared-device-mode"></a>Обнаружение режима общего устройства

Для вашего приложения важен режим обнаружения общего устройства. Во многих приложениях потребуется изменить взаимодействие с пользователем (UX), если приложение используется на общем устройстве. Например, приложение может иметь функцию регистрации, которая не подходит для рабочей роли оказался, так как у них уже есть учетная запись. Кроме того, может потребоваться дополнительный уровень безопасности обработки данных в приложении, если он находится в режиме общего устройства.

Используйте `getDeviceInformationWithParameters:completionBlock:` API в, `MSALPublicClientApplication` чтобы определить, выполняется ли приложение на устройстве в режиме общего устройства.

В следующих фрагментах кода показаны примеры использования `getDeviceInformationWithParameters:completionBlock:` API.

#### <a name="swift"></a>Swift

```swift
application.getDeviceInformation(with: nil, completionBlock: { (deviceInformation, error) in

    guard let deviceInfo = deviceInformation else {
        return
    }

    let isSharedDevice = deviceInfo.deviceMode == .shared
    // Change your app UX if needed
})
```

#### <a name="objective-c"></a>Objective-C

```objective-c
[application getDeviceInformationWithParameters:nil
                                completionBlock:^(MSALDeviceInformation * _Nullable deviceInformation, NSError * _Nullable error)
{
    if (!deviceInformation)
    {
        return;
    }

    BOOL isSharedDevice = deviceInformation.deviceMode == MSALDeviceModeShared;
    // Change your app UX if needed
}];
```

### <a name="get-the-signed-in-user-and-determine-if-a-user-has-changed-on-the-device"></a>Получение пользователя, выполнившего вход, и определение того, изменился ли пользователь на устройстве

Еще одна важная часть поддержки режима общего устройства — определение состояния пользователя на устройстве и очистка данных приложения, если пользователь изменился или если на устройстве нет пользователя. Вы несете ответственность за обеспечение утечки данных другому пользователю.

Вы можете использовать `getCurrentAccountWithParameters:completionBlock:` API для запроса учетной записи, выполнившего вход на устройстве.

#### <a name="swift"></a>Swift

```swift
let msalParameters = MSALParameters()
msalParameters.completionBlockQueue = DispatchQueue.main

application.getCurrentAccount(with: msalParameters, completionBlock: { (currentAccount, previousAccount, error) in

    // currentAccount is the currently signed in account
    // previousAccount is the previously signed in account if any
})
```

#### <a name="objective-c"></a>Objective-C

```objective-c
MSALParameters *parameters = [MSALParameters new];
parameters.completionBlockQueue = dispatch_get_main_queue();

[application getCurrentAccountWithParameters:parameters
                             completionBlock:^(MSALAccount * _Nullable account, MSALAccount * _Nullable previousAccount, NSError * _Nullable error)
{
    // currentAccount is the currently signed in account
    // previousAccount is the previously signed in account if any
}];
```

### <a name="globally-sign-in-a-user"></a>Глобальный вход пользователя

Если устройство настроено в качестве общего устройства, приложение может вызвать `acquireTokenWithParameters:completionBlock:` API для входа в учетную запись. Учетная запись будет доступна глобально для всех соответствующих приложений на устройстве после того, как первое приложение подпишет учетную запись.

#### <a name="objective-c"></a>Objective-C

```objective-c
MSALInteractiveTokenParameters *parameters = [[MSALInteractiveTokenParameters alloc] initWithScopes:@[@"api://myapi/scope"] webviewParameters:[self msalTestWebViewParameters]];

parameters.loginHint = self.loginHintTextField.text;

[application acquireTokenWithParameters:parameters completionBlock:completionBlock];
```

### <a name="globally-sign-out-a-user"></a>Глобальный выход пользователя

Следующий код удаляет учетную запись входа и очищает кэшированные токены не только от приложения, но и с устройства, которое находится в режиме общего устройства. Однако это не приводит к очистке *данных* из приложения. Необходимо очистить данные приложения, а также очистить все кэшированные данные, которые приложение может отобразить для пользователя.

#### <a name="clear-browser-state"></a>Очистить состояние браузера

> [!NOTE]
> Следующий шаг требуется только во время общедоступной предварительной версии.

В этой общедоступной предварительной версии [подключаемый модуль единого входа Microsoft Enterprise для устройств Apple](apple-sso-plugin.md) очищает состояние только для приложений. Он не очищает состояние в браузере Safari. Рекомендуется вручную очистить сеанс браузера, чтобы убедиться, что никакие трассировки состояния пользователя не остались. Можно использовать необязательное `signoutFromBrowser` свойство, показанное ниже, чтобы очистить файлы cookie. Это приведет к краткому запуску браузера на устройстве.

#### <a name="swift"></a>Swift

```swift
let account = .... /* account retrieved above */

let signoutParameters = MSALSignoutParameters(webviewParameters: self.webViewParamaters!)
signoutParameters.signoutFromBrowser = true // Only needed for Public Preview.

application.signout(with: account, signoutParameters: signoutParameters, completionBlock: {(success, error) in

    if let error = error {
        // Signout failed
        return
    }

    // Sign out completed successfully
})
```

#### <a name="objective-c"></a>Objective-C

```objective-c
MSALAccount *account = ... /* account retrieved above */;

MSALSignoutParameters *signoutParameters = [[MSALSignoutParameters alloc] initWithWebviewParameters:webViewParameters];
signoutParameters.signoutFromBrowser = YES; // Only needed for Public Preview.

[application signoutWithAccount:account signoutParameters:signoutParameters completionBlock:^(BOOL success, NSError * _Nullable error)
{
    if (!success)
    {
        // Signout failed
        return;
    }

    // Sign out completed successfully
}];
```

## <a name="next-steps"></a>Дальнейшие действия

Чтобы увидеть режим общего устройства в действии, следующий пример кода в GitHub содержит пример запуска оказался рабочего приложения на устройстве iOS в режиме общего устройства:

[Пример API MSAL iOS SWIFT Microsoft Graph](https://github.com/Azure-Samples/ms-identity-mobile-apple-swift-objc)
