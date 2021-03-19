---
title: Развертывание средства управления для виртуальных рабочих столов Windows (классическая модель) с помощью субъекта-службы Azure
description: Развертывание средства управления для виртуальных рабочих столов Windows (классическая модель) с помощью PowerShell.
author: Heidilohr
ms.topic: how-to
ms.date: 03/30/2020
ms.author: helohr
ms.custom: devx-track-azurepowershell
manager: lizross
ms.openlocfilehash: d7219751d584eb458cded9f4e30cccb1439dfa1b
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89069023"
---
# <a name="deploy-a-windows-virtual-desktop-classic-management-tool-with-powershell"></a>Развертывание средства управления виртуальными рабочими столами Windows (классическая модель) с помощью PowerShell

>[!IMPORTANT]
>Это содержимое применимо к Виртуальному рабочему столу Windows (классическому), который не поддерживает объекты Azure Resource Manager для Виртуального рабочего стола Windows.

В этой статье показано, как развернуть средство управления с помощью PowerShell.

## <a name="important-considerations"></a>Важные замечания

Каждой подписке клиента Azure Active Directory (Azure AD) требуется отдельное развертывание средства управления. Это средство не поддерживает сценарии Azure AD "бизнес — бизнес" (B2B).

Это средство управления является примером. Майкрософт предоставит важные обновления безопасности и исправления. [Исходный код доступен в GitHub](https://github.com/Azure/RDS-Templates/tree/master/wvd-templates/wvd-management-ux/deploy). Независимо от того, являетесь ли Вы клиентом или партнером, мы рекомендуем вам настроить это средство в соответствии с потребностями бизнеса.

Следующие браузеры совместимы с средством управления:

- Google Chrome версии 68 и выше
- Microsoft Edge версии 40.15063 и выше
- Mozilla Firefox версии 52.0 и выше
- Safari версии 10 и выше (только macOS)

## <a name="what-you-need-to-deploy-the-management-tool"></a>Что необходимо для развертывания средства управления

Перед развертыванием средства управления потребуется пользователь Azure Active Directory (Azure AD), чтобы создать регистрацию приложения и развернуть пользовательский интерфейс управления. Этот пользователь должен:

- получить разрешение на создание ресурсов в своей подписке Azure;
- получить разрешение на создание приложения Azure AD. Чтобы проверить, есть ли у пользователя необходимые разрешения, выполните действия, перечисленные в разделе [Необходимые разрешения](../../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app).

После развертывания и настройки средства управления рекомендуется попросить пользователя запустить пользовательский интерфейс управления, чтобы убедиться в том, что все работает. Пользователю, запускающему пользовательский интерфейс управления, должна быть назначена роль, которая позволяет просматривать или изменять клиент виртуальных рабочих столов Windows.

## <a name="set-up-powershell"></a>Настройка PowerShell

Начните с входа в модули AZ и Azure AD PowerShell. Вот как войти:

1. Откройте PowerShell с правами администратора и перейдите в каталог, в котором были сохранены сценарии PowerShell.
2. Войдите в Azure с помощью учетной записи с правами владельца или участника в подписке Azure, которую вы планируете использовать для создания средства управления, выполнив следующий командлет:

    ```powershell
    Login-AzAccount
    ```

3. Выполните следующий командлет, чтобы войти в Azure AD с той же учетной записью, которая использовалась для модуля AZ PowerShell:

    ```powershell
    Connect-AzureAD
    ```

4. После этого перейдите в папку, где были сохранены два сценария PowerShell из репозитория RDS-Templates GitHub.

Не закрывайте окно PowerShell, которое использовалось для входа в систему, чтобы запустить дополнительные командлеты PowerShell при входе.

## <a name="create-an-azure-active-directory-app-registration"></a>Создание регистрации приложения Azure Active Directory

Чтобы успешно развернуть и настроить средство управления, сначала необходимо скачать следующие скрипты PowerShell из [репозитория GitHub RDS-Templates](https://github.com/Azure/RDS-Templates/tree/master/wvd-templates/wvd-management-ux/deploy/scripts) .

```powershell
Set-Location -Path "c:\temp"
$uri = "https://raw.githubusercontent.com/Azure/RDS-Templates/master/wvd-templates/wvd-management-ux/deploy/scripts/createWvdMgmtUxAppRegistration.ps1"
Invoke-WebRequest -Uri $uri -OutFile ".\createWvdMgmtUxAppRegistration.ps1"
$uri = "https://raw.githubusercontent.com/Azure/RDS-Templates/master/wvd-templates/wvd-management-ux/deploy/scripts/updateWvdMgmtUxApiUrl.ps1"
Invoke-WebRequest -Uri $uri -OutFile ".\updateWvdMgmtUxApiUrl.ps1"
```

Выполните следующие команды, чтобы создать регистрацию приложения с необходимыми разрешениями API:

```powershell
$appName = Read-Host -Prompt "Enter a unique name for the management tool's app registration. The name can't contain spaces or special characters."
$azureSubscription = Get-AzSubscription | Out-GridView -PassThru
$subscriptionId = $azureSubscription.Id
Get-AzSubscription -SubscriptionId $subscriptionId | Select-AzSubscription

.\createWvdMgmtUxAppRegistration.ps1 -AppName $appName -SubscriptionId $subscriptionId
```

После завершения регистрации приложения Azure AD можно развернуть средство управления.

## <a name="deploy-the-management-tool"></a>Развертывание средства управления

Выполните следующие команды PowerShell, чтобы развернуть средство управления и связать его с только что созданным субъектом-службой.

```powershell
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$templateParameters = @{}
$templateParameters.Add('isServicePrincipal', $true)
$templateParameters.Add('azureAdminUserPrincipalNameOrApplicationId', $ServicePrincipalCredentials.UserName)
$templateParameters.Add('azureAdminPassword', $servicePrincipalCredentials.Password)
$templateParameters.Add('applicationName', $appName)

Get-AzSubscription -SubscriptionId $subscriptionId | Select-AzSubscription
New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName `
    -TemplateUri "https://raw.githubusercontent.com/Azure/RDS-Templates/master/wvd-templates/wvd-management-ux/deploy/mainTemplate.json" `
    -TemplateParameterObject $templateParameters `
    -Verbose
```

После создания веб-приложения необходимо добавить URI перенаправления в приложение Azure AD для успешного входа пользователей.

## <a name="set-the-redirect-uri"></a>Задание URI перенаправления

Выполните следующие команды PowerShell, чтобы получить URL-адрес приложения и задать его в качестве URI перенаправления проверки подлинности (также называемого URL-адресом ответа):

```powershell
$webApp = Get-AzWebApp -ResourceGroupName $resourceGroupName -Name $appName
$redirectUri = "https://" + $webApp.DefaultHostName + "/"
Get-AzureADApplication -All $true | where { $_.AppId -match $servicePrincipalCredentials.UserName } | Set-AzureADApplication -ReplyUrls $redirectUri
```

Теперь, когда вы добавили URI перенаправления, вам потребуется обновить URL-адрес API, чтобы средство управления могла взаимодействовать со службой внутренней службы API.

## <a name="update-the-api-url-for-the-web-application"></a>Обновите URL-адрес API для приложения

Выполните следующий скрипт, чтобы обновить конфигурацию URL-адреса API в интерфейсе веб-приложения:

```powershell
.\updateWvdMgmtUxApiUrl.ps1 -AppName $appName -SubscriptionId $subscriptionId
```

Теперь, когда вы полностью настроили веб-приложение средства управления, пришло время проверить приложение Azure AD и предоставить согласие.

## <a name="verify-the-azure-ad-application-and-provide-consent"></a>Проверка приложения Azure AD и предоставление согласия

Чтобы проверить конфигурацию приложения Azure AD и предоставить согласие:

1. Откройте веб браузер и войдите в [портал Azure](https://portal.azure.com/) с помощью учетной записи администратора.
2. На панели поиска в верхней части портал Azure найдите **Регистрация приложений** и выберите элемент в разделе **службы**.
3. Выберите **все приложения** и найдите уникальное имя приложения, указанное для сценария PowerShell в окне [Создание Azure Active Directory регистрации приложения](#create-an-azure-active-directory-app-registration).
4. На панели в левой части браузера выберите **Проверка подлинности** и убедитесь, что универсальный код ресурса (URI) перенаправления совпадает с URL-адресом приложения для средства управления, как показано на следующем рисунке.

   [![Страница проверки подлинности с указанным URI ](../media/management-ui-redirect-uri-inline.png) перенаправления](../media/management-ui-redirect-uri-expanded.png#lightbox)

5. На панели слева выберите **разрешения API** , чтобы подтвердить добавление разрешений. Если вы являетесь глобальным администратором, нажмите кнопку **предоставить согласие администратора `tenantname` для** , а затем следуйте указаниям в диалоговом окне запрос на предоставление согласия администратора для вашей организации.

    [![Страница ](../media/management-ui-permissions-inline.png) разрешений API](../media/management-ui-permissions-expanded.png#lightbox)

Теперь можно приступить к использованию средства управления.

## <a name="use-the-management-tool"></a>Использование средства управления

Теперь, когда вы настроили средство управления в любое время, вы можете запустить его в любой момент в любом месте. Вот как запустить средство:

1. Откройте URL веб-приложения в веб-браузере. Если вы не помните URL-адрес, вы можете войти в Azure, найти службу приложений, развернутую для средства управления, а затем выбрать URL-адрес.
2. Войдите в систему, используя свои учетные данные для Виртуального рабочего стола Windows.

   > [!NOTE]
   > Если вы не предоставите согласие администратора во время настройки средства управления, каждый пользователь, вошедший в систему, должен предоставить собственное согласие пользователя для использования этого средства.

3. При появлении запроса на выбор группы клиентов выберите **группу клиентов по умолчанию** из раскрывающегося списка.
4. После выбора **Default Tenant Group** (Группа клиентов по умолчанию) в левой части окна должно появиться меню. В этом меню найдите и выберите имя группы клиентов.

   > [!NOTE]
   > Если у вас есть настраиваемая группа клиентов, введите имя вручную вместо выбора из раскрывающегося списка.

## <a name="report-issues"></a>Сообщить о проблеме

Если у вас возникли проблемы со средством управления или другими средствами Виртуального рабочего стола Windows, следуйте указаниям в [шаблонах Azure Resource Manager для службы удаленных рабочих столов](https://github.com/Azure/RDS-Templates/blob/master/README.md), чтобы сообщить о них в GitHub.

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы научились развертывать средство управления и подключаться к нему, ознакомьтесь со сведениями об использовании Работоспособности служб Azure для мониторинга проблем служб и рекомендаций по работоспособности. Дополнительные сведения см. в руководстве [Настройка оповещений служб](set-up-service-alerts-2019.md).
