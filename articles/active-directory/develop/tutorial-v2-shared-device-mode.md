---
title: Руководство по Использование режима общего устройства с библиотекой проверки подлинности Майкрософт (MSAL) для Android | Azure
titleSuffix: Microsoft identity platform
description: В этом руководстве вы узнаете, как подготовить устройство Android для работы в режиме общего доступа и запустить приложение для сотрудников, взаимодействующих с клиентами.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.workload: identity
ms.date: 1/15/2020
ms.author: hahamil
ms.reviewer: brandwe
ms.custom: aaddev, identityplatformtop40
ms.openlocfilehash: 6a173ed4dae9237d8aae991c943817ed70246eea
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101649061"
---
# <a name="tutorial-use-shared-device-mode-in-your-android-application"></a>Руководство по Использование режима общего устройства в приложении Android

В этом руководстве для разработчиков Android и администраторов клиента Azure Active Directory (Azure AD) приводится код, приложение для проверки подлинности и параметры клиента, необходимые для включения режима общего устройства для приложения Android.

В этом руководстве рассматриваются следующие темы:

> [!div class="checklist"]
> * Скачивание примера кода
> * Включение и обнаружение режима общего устройства
> * Обнаружение режима использования одной и нескольких учетных записей
> * Обнаружение смены пользователя и включение глобальных входа и выхода
> * Настройка клиента и регистрация приложения на портале Azure
> * Настройка устройства Android в режиме общего устройства
> * Запуск примера приложения

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.

## <a name="developer-guide"></a>Руководство разработчика

В этом разделе руководства содержатся рекомендации для разработчиков по реализации режима общего устройства в приложении Android с помощью библиотеки проверки подлинности Майкрософт (MSAL). Сведения о том, как интегрировать MSAL с приложением Android, войти в систему, вызвать Microsoft Graph и выйти из учетной записи пользователя, см. в [руководстве по MSAL Android](./tutorial-v2-android.md).

### <a name="download-the-sample"></a>Скачивание примера приложения

Клонируйте [пример приложения](https://github.com/Azure-Samples/ms-identity-android-java/) из GitHub. Этот пример поддерживает работу в [режиме одной или нескольких учетных записей](./single-multi-account.md).

### <a name="add-the-msal-sdk-to-your-local-maven-repository"></a>Добавление пакета SDK для MSAL в локальный репозиторий Maven

Если вы не используете пример приложения, добавьте библиотеку MSAL в качестве зависимости в файл build.gradle, примерно так:

```gradle
dependencies{
  implementation 'com.microsoft.identity.client.msal:1.0.+'
}
```

### <a name="configure-your-app-to-use-shared-device-mode"></a>Настройка приложения для использования режима общего устройства

Дополнительные сведения о настройках в файле конфигурации см. в [документации по настройке](./msal-configuration.md).

В файле конфигурации MSAL укажите для `"shared_device_mode_supported"` значение `true`.

Возможно, вы не планируете поддерживать режим с несколькими учетными записями. Например, если вы не используете общее устройство и пользователь может войти в приложение с несколькими учетными записями одновременно. Если это так, задайте для параметра `"account_mode"` значение `"SINGLE"`. Это гарантирует, что ваше приложение всегда будет получать `ISingleAccountPublicClientApplication`, что значительно упрощает интеграцию MSAL. Для `"account_mode"` по умолчанию используется значение `"MULTIPLE"`. Поэтому для режима `"single account"` важно изменить это значение в файле конфигурации.

Ниже приведен пример файла auth_config.json, который размещен в каталоге **app**>**main**>**res**>**raw** для примера приложения:

```json
{
 "client_id":"Client ID after app registration at https://aka.ms/MobileAppReg",
 "authorization_user_agent":"DEFAULT",
 "redirect_uri":"Redirect URI after app registration at https://aka.ms/MobileAppReg",
 "account_mode":"SINGLE",
 "broker_redirect_uri_registered": true,
 "shared_device_mode_supported": true,
 "authorities":[
  {
   "type":"AAD",
   "audience":{
     "type": "AzureADandPersonalMicrosoftAccount",
     "tenant_id":"common"
   }
  }
 ]
}
```

### <a name="detect-shared-device-mode"></a>Обнаружение режима общего устройства

Режим общего устройства позволяет настроить на устройстве Android возможность совместного использования несколькими сотрудниками, сохраняя для него функцию управления через Microsoft Identity. Сотрудники могут входить на устройства и быстро получать доступ к сведениям о клиентах. Выполнив поставленную задачу или завершив рабочую смену, сотрудник одним щелчком выходит из всех приложений на общем устройстве, которое немедленно подготавливается для входа следующего сотрудника.

С помощью `isSharedDevice()` определите, выполняется ли приложение на устройстве в режиме общего доступа. По значению этого флага приложение может изменять пользовательский интерфейс соответствующим образом.

Ниже приведен фрагмент кода с примером использования `isSharedDevice()`.  Он взят из класса `SingleAccountModeFragment` в нашем примере приложения.

```Java
deviceModeTextView.setText(mSingleAccountApp.isSharedDevice() ? "Shared" : "Non-Shared");
```

### <a name="initialize-the-publicclientapplication-object"></a>Инициализация объекта PublicClientApplication

Если в файле конфигурации MSAL вы задали значение `"account_mode":"SINGLE"`, возвращенный объект приложения можно без опасений приводить к типу `ISingleAccountPublicCLientApplication`.

```java
private ISingleAccountPublicClientApplication mSingleAccountApp;

/*Configure your sample app and save state for this activity*/
PublicClientApplication.create(this.getApplicationCOntext(),
  R.raw.auth_config,
  new PublicClientApplication.ApplicationCreatedListener(){
  @Override
  public void onCreated(IPublicClientApplication application){
  mSingleAccountApp = (ISingleAccountPublicClientApplication)application;
  loadAccount();
  }
  @Override
  public void onError(MsalException exception{
  /*Fail to initialize PublicClientApplication */
  }
});
```

### <a name="detect-single-vs-multiple-account-mode"></a>Различение режимов одной и нескольких учетных записей

Если вы создаете приложение, которое будет использоваться только на общем устройстве и только сотрудниками, взаимодействующими с клиентами, мы рекомендуем поддерживать в этом приложении только режим одной учетной записи. Это относится к большинству приложений, ориентированных на задачи, например к приложениям для учета медицинских записей и выставления счетов, а также к большинству бизнес-приложений. Это упростит разработку, так как вам не придется применять многие функции пакета SDK.

Если приложение поддерживает не только несколько учетных записей, но и режим общего устройства, вам нужно будет выполнять проверку типа и приводить его к соответствующему интерфейсу, как показано ниже.

```java
private IPublicClientApplication mApplication;

        if (mApplication instanceOf IMultipleAccountPublicClientApplication) {
          IMultipleAccountPublicClientApplication multipleAccountApplication = (IMultipleAccountPublicClientApplication) mApplication;
          ...
        } else if (mApplication instanceOf    ISingleAccountPublicClientApplication) {
           ISingleAccountPublicClientApplication singleAccountApplication = (ISingleAccountPublicClientApplication) mApplication;
            ...
        }
```

### <a name="get-the-signed-in-user-and-determine-if-a-user-has-changed-on-the-device"></a>Получение сведений о пользователе, выполнившем вход, и определение того, изменился ли пользователь на устройстве

Метод `loadAccount` позволяет извлечь учетную запись пользователя, выполнившего вход. Метод `onAccountChanged` определяет, изменился ли пользователь, выполнивший вход, и если да, выполняет очистку.

```java
private void loadAccount()
{
  mSingleAccountApp.getCurrentAccountAsync(new ISingleAccountPublicClientApplication.CurrentAccountCallback()
  {
    @Overide
    public void onAccountLoaded(@Nullable IAccount activeAccount)
    {
      if (activeAccount != null)
      {
        signedInUser = activeAccount;
        mSingleAccountApp.acquireTokenSilentAsync(SCOPES,"http://login.microsoftonline.com/common",getAuthSilentCallback());
      }
    }
    @Override
    public void on AccountChanged(@Nullable IAccount priorAccount, @Nullable Iaccount currentAccount)
    {
      if (currentAccount == null)
      {
        //Perform a cleanup task as the signed-in account changed.
        updateSingedOutUI();
      }
    }
    @Override
    public void onError(@NonNull Exception exception)
    {
    }
  }
}
```

### <a name="globally-sign-in-a-user"></a>Глобальный вход пользователя

С помощью приведенного ниже кода выполняется вход пользователя на том же устройстве в другие приложения, которые используют MSAL с приложением Authenticator.

```java
private void onSignInClicked()
{
  mSingleAccountApp.signIn(getActivity(), SCOPES, null, getAuthInteractiveCallback());
}
```

### <a name="globally-sign-out-a-user"></a>Глобальный выход пользователя

В приведенном далее примере удаляется учетная запись пользователя, выполнившего вход, а также кэшированные токены как из текущего приложения, таки и с устройства в режиме общего доступа.

```java
private void onSignOutClicked()
{
  mSingleAccountApp.signOut(new ISingleAccountPublicClientApplication.SignOutCallback()
  {
    @Override
    public void onSignOut()
    {
      updateSignedOutUI();
    }
    @Override
    public void onError(@NonNull MsalException exception)
    {
      /*failed to remove account with an exception*/
    }
  });
}
```

## <a name="administrator-guide"></a>Руководство для администратора

Ниже представлены инструкции по настройке приложения на портале Azure и переводу устройства в режим общего доступа.

### <a name="register-your-application-in-azure-active-directory"></a>Регистрация приложения в Azure Active Directory

Сначала зарегистрируйте приложение в клиенте организации. Затем предоставьте в файле auth_config.json указанные далее значения, чтобы приложение выполнялось правильно.

Дополнительные сведения об этом см. в статье о [регистрации приложения](./tutorial-v2-android.md#register-your-application).

> [!NOTE]
> При регистрации приложения используйте краткое руководство, которое предоставляется на экране слева, а затем выберите **Android**. Вы перейдете к странице, где будет предложено ввести **имя пакета** и **хэш подписи** для приложения. Эти параметры очень важны для того, чтобы конфигурация приложения работала нормально. Затем вы получите объект конфигурации, который можно использовать для приложения. Его нужно скопировать и вставить в файл auth_config.json.

:::image type="content" source="media/tutorial-v2-shared-device-mode/register-app.png" alt-text="Краткое руководство по настройке страницы приложения Android на портале Azure":::

Выберите действие **Make this change for me** (Внести это изменение для меня) и укажите на портале Azure значения, которые приведены в кратком руководстве. Когда это будет сделано, мы автоматически создадим для вас все необходимые файлы конфигурации.

:::image type="content" source="media/tutorial-v2-shared-device-mode/config-info.png" alt-text="Краткое руководство по настройке страницы проекта на портале Azure":::

## <a name="set-up-a-tenant"></a>Настройка клиента

Для тестирования выполните в клиенте настройку для следующих элементов: по меньшей мере для двух сотрудников, одного администратора облачных устройств и одного глобального администратора. На портале Azure выполните настройку для администратора облачных устройств, указав соответствующие роли в организации. На портале Azure откройте раздел "Роли организации", последовательно выбрав элементы **Azure Active Directory** > **Роли и администраторы** > **Администратор облачных устройств**. Добавьте пользователей, которые могут перевести устройство в режим общего доступа.

## <a name="set-up-an-android-device-in-shared-mode"></a>Настройка устройства Android в режиме общего доступа

### <a name="download-the-authenticator-app"></a>Скачивание приложения Authenticator

Скачайте приложение Microsoft Authenticator из магазина Google Play. Если это приложение уже скачано, убедитесь, что используете последнюю версию.

### <a name="authenticator-app-settings--registering-the-device-in-the-cloud"></a>Параметры приложения Authenticator и регистрации устройства в облаке

Запустите приложение Authenticator и перейдите на главную страницу учетной записи. На странице **Добавление учетной записи** вы можете сразу перевести устройство в режим общего доступа.

:::image type="content" source="media/tutorial-v2-shared-device-mode/authenticator-add-account.png" alt-text="Экран добавления учетной записи в Authenticator":::

Перейдите на панель **Параметры**, используя панель меню справа. Щелкните **Регистрации устройства** в разделе **Work & School accounts** (Рабочие и учебные учетные записи).

:::image type="content" source="media/tutorial-v2-shared-device-mode/authenticator-settings.png" alt-text="Экран параметров приложения Authenticator":::

При нажатии этой кнопки вам будет предложено авторизовать доступ к контактам устройства. Это требования к интеграции учетных записей на устройстве Android. Выберите **разрешить**.

:::image type="content" source="media/tutorial-v2-shared-device-mode/authenticator-allow-screen.png" alt-text="Экран подтверждения разрешения доступа приложению Authenticator":::

Администратору облачных устройств следует ввести адрес электронной почты своей организации в разделе **Также возможна регистрация как общего устройства**. После этого нажмите кнопку **Зарегистрировать как общее устройство** и введите учетные данные.

:::image type="content" source="media/tutorial-v2-shared-device-mode/register-device.png" alt-text="Экран регистрации устройства в приложении":::

:::image type="content" source="media/tutorial-v2-shared-device-mode/sign-in.png" alt-text="Снимок экрана приложения: страница входа (Майкрософт)":::

Теперь устройство находится в режиме общего доступа.

:::image type="content" source="media/tutorial-v2-shared-device-mode/shared-device-mode-screen.png" alt-text="Экран приложения: режим общего устройства включен":::

 Все операции входа и выхода на этом устройстве будут глобальными. Это означает, что они будут применяться ко всем приложениям, интегрированным с MSAL и Microsoft Authenticator на этом устройстве. Теперь вы можете развертывать на устройстве приложения, которые используют возможности режима общего устройства.

## <a name="view-the-shared-device-in-the-azure-portal"></a>Просмотр сведений об общем устройстве на портале Azure

Когда вы переводите устройство в режим общего доступа, оно регистрируется для вашей организации и отслеживается в ее арендаторе. Чтобы просмотреть сведения о своих общих устройствах, найдите раздел **Тип соединение** в колонке Azure Active Directory на портале Azure.

:::image type="content" source="media/tutorial-v2-shared-device-mode/registered-device-screen.png" alt-text="Панель &quot;Все устройства&quot; на портале Azure":::

## <a name="running-the-sample-app"></a>Выполнение примера приложения

Это простой пример приложения, которое обращается к API Graph вашей организации. При первом запуске вам будет предложено подтвердить согласие, так как приложение является новым для вашей учетной записи сотрудника.

:::image type="content" source="media/tutorial-v2-shared-device-mode/run-app-permissions-requested.png" alt-text="Экран сведений о конфигурации приложения":::

## <a name="next-steps"></a>Дальнейшие действия

Узнайте больше о работе с библиотекой проверки подлинности Майкрософт и использовании режима общего устройства на устройствах Android:

> [!div class="nextstepaction"]
> [Режим совместной работы с устройством Android](msal-android-shared-devices.md)
