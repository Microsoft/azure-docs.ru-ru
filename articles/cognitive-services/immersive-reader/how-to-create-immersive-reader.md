---
title: Создание ресурса иммерсивного средства чтения
titleSuffix: Azure Cognitive Services
description: В этой статье показано, как создать новый иммерсивное устройство чтения с помощью пользовательского поддомена, а затем настроить Azure AD в клиенте Azure.
services: cognitive-services
author: rwaller
manager: guillasi
ms.service: cognitive-services
ms.subservice: immersive-reader
ms.topic: how-to
ms.date: 07/22/2019
ms.author: rwaller
ms.openlocfilehash: d7a8a598ab2d0c873cf07510cf40a08c22c0f1d4
ms.sourcegitcommit: d135e9a267fe26fbb5be98d2b5fd4327d355fe97
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102608603"
---
# <a name="create-an-immersive-reader-resource-and-configure-azure-active-directory-authentication"></a>Создание иммерсивного ресурса чтения и Настройка проверки подлинности Azure Active Directory

В этой статье мы предоставляем скрипт, который создаст иммерсивное устройство чтения и настроит проверку подлинности Azure Active Directory (Azure AD). Каждый раз, когда создается иммерсивное устройство чтения, как с помощью этого скрипта, так и на портале, его также необходимо настроить с помощью разрешений Azure AD. Этот сценарий поможет вам.

Этот сценарий предназначен для создания и настройки всех необходимых иммерсивное средство чтения и ресурсов Azure AD за один шаг. Тем не менее можно также настроить проверку подлинности Azure AD для существующего иммерсивного ресурса чтения, если это возможно, но вы уже создали его в портал Azure.

Для некоторых клиентов может потребоваться создать несколько впечатляющих ресурсов чтения, для разработки и производства, или, возможно, для нескольких регионов, в которых развернута служба. В этих случаях можно вернуться к использованию сценария несколько раз, чтобы создать различные впечатляющие ресурсы чтения и настроить их с помощью разрешений Azure AD.

Этот сценарий предназначен для гибкой работы. Сначала он будет искать существующие иммерсивное средство чтения и ресурсы Azure AD в подписке и создавать их, если они еще не существуют. Если вы впервые создаете иммерсивное устройство чтения, сценарий сделает все необходимое. Если вы хотите использовать его только для настройки Azure AD для существующего иммерсивного ресурса чтения, созданного на портале, это тоже будет сделано. Его также можно использовать для создания и настройки нескольких впечатляющих ресурсов модуля чтения.

## <a name="set-up-powershell-environment"></a>Настройка среды PowerShell

1. Для начала откройте [Azure Cloud Shell](../../cloud-shell/overview.md). Убедитесь, что для Cloud Shell задано значение PowerShell в левом верхнем углу раскрывающегося списка или с помощью команды `pwsh` .

1. Скопируйте и вставьте следующий фрагмент кода в оболочку.

    ```azurepowershell-interactive
    function Create-ImmersiveReaderResource(
        [Parameter(Mandatory=$true, Position=0)] [String] $SubscriptionName,
        [Parameter(Mandatory=$true)] [String] $ResourceName,
        [Parameter(Mandatory=$true)] [String] $ResourceSubdomain,
        [Parameter(Mandatory=$true)] [String] $ResourceSKU,
        [Parameter(Mandatory=$true)] [String] $ResourceLocation,
        [Parameter(Mandatory=$true)] [String] $ResourceGroupName,
        [Parameter(Mandatory=$true)] [String] $ResourceGroupLocation,
        [Parameter(Mandatory=$true)] [String] $AADAppDisplayName="ImmersiveReaderAAD",
        [Parameter(Mandatory=$true)] [String] $AADAppIdentifierUri,
        [Parameter(Mandatory=$true)] [String] $AADAppClientSecret,
        [Parameter(Mandatory=$true)] [String] $AADAppClientSecretExpiration
    )
    {
        $unused = ''
        if (-not [System.Uri]::TryCreate($AADAppIdentifierUri, [System.UriKind]::Absolute, [ref] $unused)) {
            throw "Error: AADAppIdentifierUri must be a valid URI"
        }

        Write-Host "Setting the active subscription to '$SubscriptionName'"
        $subscriptionExists = Get-AzSubscription -SubscriptionName $SubscriptionName
        if (-not $subscriptionExists) {
            throw "Error: Subscription does not exist"
        }
        az account set --subscription $SubscriptionName

        $resourceGroupExists = az group exists --name $ResourceGroupName
        if ($resourceGroupExists -eq "false") {
            Write-Host "Resource group does not exist. Creating resource group"
            $groupResult = az group create --name $ResourceGroupName --location $ResourceGroupLocation
            if (-not $groupResult) {
                throw "Error: Failed to create resource group"
            }
            Write-Host "Resource group created successfully"
        }

        # Create an Immersive Reader resource if it doesn't already exist
        $resourceId = az cognitiveservices account show --resource-group $ResourceGroupName --name $ResourceName --query "id" -o tsv
        if (-not $resourceId) {
            Write-Host "Creating the new Immersive Reader resource '$ResourceName' (SKU '$ResourceSKU') in '$ResourceLocation' with subdomain '$ResourceSubdomain'"
            $resourceId = az cognitiveservices account create `
                            --name $ResourceName `
                            --resource-group $ResourceGroupName `
                            --kind ImmersiveReader `
                            --sku $ResourceSKU `
                            --location $ResourceLocation `
                            --custom-domain $ResourceSubdomain `
                            --query "id" `
                            -o tsv

            if (-not $resourceId) {
                throw "Error: Failed to create Immersive Reader resource"
            }
            Write-Host "Immersive Reader resource created successfully"
        }

        # Create an Azure Active Directory app if it doesn't already exist
        $clientId = az ad app show --id $AADAppIdentifierUri --query "appId" -o tsv
        if (-not $clientId) {
            Write-Host "Creating new Azure Active Directory app"
            $clientId = az ad app create --password $AADAppClientSecret --end-date "$AADAppClientSecretExpiration" --display-name $AADAppDisplayName --identifier-uris $AADAppIdentifierUri --query "appId" -o tsv

            if (-not $clientId) {
                throw "Error: Failed to create Azure Active Directory app"
            }
            Write-Host "Azure Active Directory app created successfully."
            Write-Host "NOTE: To manage your Active Directory app client secrets after this Immersive Reader Resource has been created please visit https://portal.azure.com and go to Home -> Azure Active Directory -> App Registrations -> $AADAppDisplayName -> Certificates and Secrets blade -> Client Secrets section" -ForegroundColor Yellow
        }

        # Create a service principal if it doesn't already exist
        $principalId = az ad sp show --id $AADAppIdentifierUri --query "objectId" -o tsv
        if (-not $principalId) {
            Write-Host "Creating new service principal"
            az ad sp create --id $clientId | Out-Null
            $principalId = az ad sp show --id $AADAppIdentifierUri --query "objectId" -o tsv

            if (-not $principalId) {
                throw "Error: Failed to create new service principal"
            }
            Write-Host "New service principal created successfully"
        }

        # Sleep for 5 seconds to allow the new service principal to propagate
        Write-Host "Sleeping for 5 seconds"
        Start-Sleep -Seconds 5

        Write-Host "Granting service principal access to the newly created Immersive Reader resource"
        $accessResult = az role assignment create --assignee $principalId --scope $resourceId --role "Cognitive Services User"
        if (-not $accessResult) {
            throw "Error: Failed to grant service principal access"
        }
        Write-Host "Service principal access granted successfully"

        # Grab the tenant ID, which is needed when obtaining an Azure AD token
        $tenantId = az account show --query "tenantId" -o tsv

        # Collect the information needed to obtain an Azure AD token into one object
        $result = @{}
        $result.TenantId = $tenantId
        $result.ClientId = $clientId
        $result.ClientSecret = $AADAppClientSecret
        $result.Subdomain = $ResourceSubdomain

        Write-Host "Success! " -ForegroundColor Green -NoNewline
        Write-Host "Save the following JSON object to a text file for future reference:"
        Write-Output (ConvertTo-Json $result)
    }
    ```

1. Выполните функцию `Create-ImmersiveReaderResource` , указав заполнители "<PARAMETER_VALUES>" ниже с собственными значениями.

    ```azurepowershell-interactive
    Create-ImmersiveReaderResource -SubscriptionName '<SUBSCRIPTION_NAME>' -ResourceName '<RESOURCE_NAME>' -ResourceSubdomain '<RESOURCE_SUBDOMAIN>' -ResourceSKU '<RESOURCE_SKU>' -ResourceLocation '<RESOURCE_LOCATION>' -ResourceGroupName '<RESOURCE_GROUP_NAME>' -ResourceGroupLocation '<RESOURCE_GROUP_LOCATION>' -AADAppDisplayName '<AAD_APP_DISPLAY_NAME>' -AADAppIdentifierUri '<AAD_APP_IDENTIFIER_URI>' -AADAppClientSecret '<AAD_APP_CLIENT_SECRET>' -AADAppClientSecretExpiration '<AAD_APP_CLIENT_SECRET_EXPIRATION>'
    ```

    Полная команда будет выглядеть примерно следующим образом. Здесь мы поместили каждый параметр в отдельной строке для ясности, чтобы можно было увидеть команду целиком. Не копируйте или не используйте эту команду "как есть". Скопируйте и используйте приведенную выше команду с собственными значениями. Этот пример содержит фиктивные значения для "<PARAMETER_VALUES>" выше. Ваши значения будут отличаться, так как вы будете использовать собственные имена для этих значений.

    ```
    Create-ImmersiveReaderResource
        -SubscriptionName 'MyOrganizationSubscriptionName'
        -ResourceName 'MyOrganizationImmersiveReader'
        -ResourceSubdomain 'MyOrganizationImmersiveReader'
        -ResourceSKU 'S0'
        -ResourceLocation 'westus2'
        -ResourceGroupName 'MyResourceGroupName'
        -ResourceGroupLocation 'westus2'
        -AADAppDisplayName 'MyOrganizationImmersiveReaderAADApp'
        -AADAppIdentifierUri 'https://MyOrganizationImmersiveReaderAADApp'
        -AADAppClientSecret 'SomeStrongPassword'
        -AADAppClientSecretExpiration '2021-12-31'
    ```

    | Параметр | Комментарии |
    | --- | --- |
    | Параметр SubscriptionName |Имя подписки Azure, которая будет использоваться для вашего иммерсивного ресурса чтения. Чтобы создать ресурс, необходимо иметь подписку. |
    | ResourceName |  Должен быть буквенно-цифровым и может содержать символ "-", если "-" не является первым или последним знаком. Длина не может превышать 63 символов.|
    | ресаурцесубдомаин |Для иммерсивного ресурса чтения требуется пользовательский поддомен. Поддомен используется пакетом SDK при вызове иммерсивное службу чтения для запуска средства чтения. Поддомен должен быть глобально уникальным. Поддомен должен состоять из букв и цифр и может содержать символ "-", если "-" не является первым или последним знаком. Длина не может превышать 63 символов. Этот параметр является необязательным, если ресурс уже существует. |
    | ресаурцеску |Параметры: `S0` (уровень Standard) или `S1` ("образование" и "некоммерческие организации"). Дополнительные сведения о каждом доступном номере SKU см. на [странице цен на Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services/immersive-reader/) . Этот параметр является необязательным, если ресурс уже существует. |
    | ResourceLocation |Параметры: `eastus` , `eastus2` , `southcentralus` , `westus` , `westus2` , `australiaeast` , `southeastasia` , `centralindia` , `japaneast` , `northeurope` , `uksouth` , `westeurope` . Этот параметр является необязательным, если ресурс уже существует. |
    | ResourceGroupName |Ресурсы создаются в группах ресурсов в рамках подписок. Укажите имя существующей группы ресурсов. Если группа ресурсов еще не существует, будет создана новая учетная часть с таким именем. |
    | ResourceGroupLocation |Если группа ресурсов не существует, необходимо указать расположение, в котором будет создана группа. Чтобы найти список расположений, выполните команду `az account list-locations` . Используйте свойство *Name* (без пробелов) возвращаемого результата. Этот параметр является необязательным, если группа ресурсов уже существует. |
    | аадаппдисплайнаме |Отображаемое имя Azure Active Directory приложения. Если существующее приложение Azure AD не найдено, будет создано новое имя с таким именем. Этот параметр является необязательным, если приложение Azure AD уже существует. |
    | аадаппидентифиерури |Универсальный код ресурса (URI) для приложения Azure AD. Если существующее приложение Azure AD не найдено, будет создан новый объект с этим URI. Например, `https://immersivereaderaad-mycompany`. |
    | аадаппклиентсекрет |Созданный пароль, который будет использоваться позже для проверки подлинности при получении маркера для запуска иммерсивное средство чтения. Длина пароля должна составлять не менее 16 символов, содержать по крайней мере один специальный символ и содержать по крайней мере один числовой символ. Чтобы управлять секретами клиента приложения Azure AD после создания этого ресурса, посетите страницу https://portal.azure.com Home-> Azure Active Directory-> регистрация приложений-> `[AADAppDisplayName]` -> сертификаты и секреты колонка — > секреты клиента (как показано на снимке экрана "Управление секретами приложения Azure AD"). |
    | аадаппклиентсекретекспиратион |Дата или время, после которых `[AADAppClientSecret]` истекает срок действия (например, "2020-12-31T11:59:59 + 00:00" или "2020-12-31"). |

    Управление секретами приложения Azure AD

    ![Колонка сертификатов и секретов портала Azure](./media/client-secrets-blade.png)

1. Скопируйте выходные данные JSON в текстовый файл для последующего использования. Полученный результат должен выглядеть примерно так:

    ```json
    {
      "TenantId": "...",
      "ClientId": "...",
      "ClientSecret": "...",
      "Subdomain": "..."
    }
    ```

## <a name="next-steps"></a>Дальнейшие действия

* Ознакомьтесь с [кратким руководством для разработчиков Node.js](./quickstarts/client-libraries.md?pivots=programming-language-nodejs), чтобы узнать другие возможности пакета SDK иммерсивного средства чтения при использовании Node.js
* Ознакомьтесь с [руководством для разработчиков Android](./tutorial-android.md), чтобы узнать о других возможностях пакета SDK иммерсивного средства чтения при использовании Java или Kotlin для Android.
* Ознакомьтесь с [руководством для разработчиков iOS](./tutorial-ios.md), чтобы узнать о других возможностях пакета SDK иммерсивного средства чтения при использовании Swift для iOS.
* Ознакомьтесь с [руководством для разработчиков Python](./tutorial-python.md), чтобы узнать другие возможности пакета SDK иммерсивного средства чтения при использовании Python.
* Ознакомьтесь с разделом о [пакете SDK для иммерсивного средства чтения](https://github.com/microsoft/immersive-reader-sdk) и [справочнике по этому пакету](./reference.md).