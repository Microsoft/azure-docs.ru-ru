---
title: Развертывание настраиваемых политик с помощью Azure Pipelines
titleSuffix: Azure AD B2C
description: Узнайте, как развертывать пользовательские политики Azure AD B2C в конвейере CI/CD с помощью Azure Pipelines в Azure DevOps Services.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 02/14/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 13f5f8da0bd58cef0974e8ea8f5f3c5172daa0ba
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96928738"
---
# <a name="deploy-custom-policies-with-azure-pipelines"></a>Развертывание настраиваемых политик с помощью Azure Pipelines

С помощью конвейера непрерывной интеграции и доставки (CI/CD), настроенного в [Azure pipelines][devops-pipelines], можно включать Azure AD B2C настраиваемые политики в вашу доставку программного обеспечения и автоматизацию управления кодом. При развертывании в разных средах Azure AD B2C, например для разработки, тестирования и производства, рекомендуется удалить ручные процессы и выполнить автоматическое тестирование с помощью Azure Pipelines.

Существует три основных шага, которые необходимо выполнить для включения Azure Pipelines управления пользовательскими политиками в Azure AD B2C.

1. Создание регистрации веб-приложения в клиенте Azure AD B2C
1. Настройка репозитория Azure
1. Настройка конвейера Azure

> [!IMPORTANT]
> Управление Azure AD B2C пользовательскими политиками с помощью конвейера Azure в настоящее время использует операции **предварительной версии** , доступные в `/beta` конечной точке API Microsoft Graph. Использование этих API для приложений в рабочей среде не поддерживается. Дополнительные сведения см. в [справочнике по конечной точке бета-версии Microsoft Graph REST API](/graph/api/overview?toc=.%2fref%2ftoc.json&view=graph-rest-beta).

## <a name="prerequisites"></a>Предварительные условия

* [Azure AD B2C клиент](tutorial-create-tenant.md)и учетные данные для пользователя в каталоге с ролью [администратора политики B2C инфраструктура процедур идентификации](../active-directory/roles/permissions-reference.md#b2c-ief-policy-administrator)
* [Пользовательские политики](custom-policy-get-started.md) , отправленные в клиент
* [Приложение управления](microsoft-graph-get-started.md) , зарегистрированное в клиенте с помощью политики разрешений Microsoft Graph API *. ReadWrite. TrustFramework*
* [Конвейер Azure](https://azure.microsoft.com/services/devops/pipelines/)и доступ к [Azure DevOps Services проекту][devops-create-project]

## <a name="client-credentials-grant-flow"></a>Поток предоставления учетных данных клиента

Описываемый здесь сценарий использует вызовы между службами между Azure Pipelines и Azure AD B2C с помощью [потока предоставления учетных данных клиента](../active-directory/azuread-dev/v1-oauth2-client-creds-grant-flow.md)OAuth 2,0. Этот поток предоставления позволяет веб-службе, такой как Azure Pipelines (конфиденциальный клиент), использовать собственные учетные данные вместо олицетворения пользователя для проверки подлинности при вызове другой веб-службы (в данном случае Microsoft Graph API). Azure Pipelines получает маркер в неинтерактивном режиме, а затем выполняет запросы к API Microsoft Graph.

## <a name="register-an-application-for-management-tasks"></a>Регистрация приложения для задач управления

Как упоминалось в [предварительных требованиях](#prerequisites), требуется регистрация приложения, выполняемая сценариями PowerShell, с помощью Azure pipelines--может использоваться для доступа к ресурсам в клиенте.

Если у вас уже есть регистрация приложения, которая используется для задач автоматизации, убедитесь, что ей было предоставлено разрешение политики политики **Microsoft Graph**  >    >  **. ReadWrite. TrustFramework** в **разрешениях API** регистрации приложения.

Инструкции по регистрации приложения управления см. в статье [управление Azure AD B2C с помощью Microsoft Graph](microsoft-graph-get-started.md).

## <a name="configure-an-azure-repo"></a>Настройка репозитория Azure

После регистрации приложения управления вы можете настроить репозиторий для файлов политики.

1. Войдите в организацию Azure DevOps Services.
1. [Создайте новый проект][devops-create-project] или выберите существующий проект.
1. В проекте перейдите по адресу **репозиториев** и выберите страницу **файлы** . Выберите существующий репозиторий или создайте его для этого упражнения.
1. Создайте папку с именем *B2CAssets*. Назовите требуемый файл заполнителя *readme.md* и **зафиксируйте** файл. При необходимости этот файл можно удалить позже.
1. Добавьте файлы политики Azure AD B2C в папку *B2CAssets* . Сюда входят *TrustFrameworkBase.xml*, *TrustFrameWorkExtensions.xml*, *SignUpOrSignin.xml*, *ProfileEdit.xml*, *PasswordReset.xml* и другие созданные политики. Запишите имя каждого файла политики Azure AD B2C для использования на более позднем шаге (они используются в качестве аргументов сценария PowerShell).
1. Создайте папку с именем *Scripts* в корневом каталоге репозитория, присвойте имя файлу заполнителя *DeployToB2c.ps1*. Не зафиксируйте файл на этом этапе, вы сделаете это на следующем шаге.
1. Вставьте следующий скрипт PowerShell в *DeployToB2c.ps1*, а затем **зафиксируйте** файл. Сценарий получает маркер из Azure AD и вызывает API Microsoft Graph, чтобы передать политики в папку *B2CAssets* в клиент Azure AD B2C.

    ```PowerShell
    [Cmdletbinding()]
    Param(
        [Parameter(Mandatory = $true)][string]$ClientID,
        [Parameter(Mandatory = $true)][string]$ClientSecret,
        [Parameter(Mandatory = $true)][string]$TenantId,
        [Parameter(Mandatory = $true)][string]$PolicyId,
        [Parameter(Mandatory = $true)][string]$PathToFile
    )

    try {
        $body = @{grant_type = "client_credentials"; scope = "https://graph.microsoft.com/.default"; client_id = $ClientID; client_secret = $ClientSecret }

        $response = Invoke-RestMethod -Uri https://login.microsoftonline.com/$TenantId/oauth2/v2.0/token -Method Post -Body $body
        $token = $response.access_token

        $headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
        $headers.Add("Content-Type", 'application/xml')
        $headers.Add("Authorization", 'Bearer ' + $token)

        $graphuri = 'https://graph.microsoft.com/beta/trustframework/policies/' + $PolicyId + '/$value'
        $policycontent = Get-Content $PathToFile
        $response = Invoke-RestMethod -Uri $graphuri -Method Put -Body $policycontent -Headers $headers

        Write-Host "Policy" $PolicyId "uploaded successfully."
    }
    catch {
        Write-Host "StatusCode:" $_.Exception.Response.StatusCode.value__

        $_

        $streamReader = [System.IO.StreamReader]::new($_.Exception.Response.GetResponseStream())
        $streamReader.BaseStream.Position = 0
        $streamReader.DiscardBufferedData()
        $errResp = $streamReader.ReadToEnd()
        $streamReader.Close()

        $ErrResp

        exit 1
    }

    exit 0
    ```

## <a name="configure-your-azure-pipeline"></a>Настройка конвейера Azure

После инициализации репозитория и заполнения его файлами настраиваемой политики можно приступать к настройке конвейера выпуска.

### <a name="create-pipeline"></a>Создание конвейера

1. Войдите в организацию Azure DevOps Services и перейдите к своему проекту.
1. В проекте выберите **конвейеры**  >  **выпускают**  >  **Новый конвейер**.
1. В разделе **Выбор шаблона** выберите **пустое задание**.
1. Введите **имя этапа**, например *деплойкустомполиЦиес*, а затем закройте панель.
1. Выберите **Добавить артефакт** и в разделе **тип источника** выберите **репозиторий Azure**.
    1. Выберите исходный репозиторий, содержащий папку *Scripts* , заполненную сценарием PowerShell.
    1. Выберите **ветвь по умолчанию**. Если вы создали новый репозиторий в предыдущем разделе, ветвь по умолчанию — *master*.
    1. Оставьте параметр **версии по умолчанию** *Последняя из ветви по умолчанию*.
    1. Введите **Псевдоним источника** для репозитория. Например, *полицирепо*. Не включайте пробелы в имя псевдонима.
1. Выберите **Добавить**.
1. Переименуйте конвейер, чтобы он отражал свое намерение. Например, *разверните пользовательский конвейер политики*.
1. Нажмите кнопку **сохранить** , чтобы сохранить конфигурацию конвейера.

### <a name="configure-pipeline-variables"></a>Настройка переменных конвейера

1. Перейдите на вкладку **переменные** .
1. Добавьте следующие переменные в **переменные конвейера** и задайте их значения, как указано ниже.

    | Имя | Значение |
    | ---- | ----- |
    | `clientId` | **Идентификатор приложения (клиента)** приложения, зарегистрированного ранее. |
    | `clientSecret` | Значение **секрета клиента** , созданного ранее. <br /> Измените тип переменной на **секрет** (щелкните значок замка). |
    | `tenantId` | `your-b2c-tenant.onmicrosoft.com`, где *B2C* — имя клиента Azure AD B2C. |

1. Нажмите кнопку **сохранить** , чтобы сохранить переменные.

### <a name="add-pipeline-tasks"></a>Добавление задач конвейера

Затем добавьте задачу для развертывания файла политики.

1. Перейдите на вкладку **задачи** .
1. Выберите **Задание агента**, а затем щелкните знак плюса ( **+** ), чтобы добавить задачу в задание агента.
1. Найдите и выберите **PowerShell**. Не выбирайте "Azure PowerShell," PowerShell на целевых компьютерах "или другую запись PowerShell.
1. Выберите только что добавленную задачу « **скрипт PowerShell** ».
1. Введите следующие значения для задачи "скрипт PowerShell":
    * **Версия задачи**: 2. *
    * **Отображаемое имя**: имя политики, которую должна передать эта задача. Например, *B2C_1A_TrustFrameworkBase*.
    * **Тип**: путь к файлу
    * **Путь к сценарию**: нажмите кнопку с многоточием (**_..._* _), перейдите в папку _Scripts *, а затем выберите файл *DeployToB2C.ps1* .
    * **Аргументы:**

        Введите следующие значения **аргументов**. Замените на `{alias-name}` псевдоним, указанный в предыдущем разделе.

        ```PowerShell
        # Before
        -ClientID $(clientId) -ClientSecret $(clientSecret) -TenantId $(tenantId) -PolicyId B2C_1A_TrustFrameworkBase -PathToFile $(System.DefaultWorkingDirectory)/{alias-name}/B2CAssets/TrustFrameworkBase.xml
        ```

        Например, если вы указали псевдоним *полицирепо*, строка аргумента должна иметь следующее значение:

        ```PowerShell
        # After
        -ClientID $(clientId) -ClientSecret $(clientSecret) -TenantId $(tenantId) -PolicyId B2C_1A_TrustFrameworkBase -PathToFile $(System.DefaultWorkingDirectory)/policyRepo/B2CAssets/TrustFrameworkBase.xml
        ```

1. Нажмите кнопку **сохранить** , чтобы сохранить задание агента.

Задача, которую вы только что добавили, отправляет *один* файл политики в Azure AD B2C. Прежде чем продолжать, вручную активируйте задание (**создать выпуск**), чтобы убедиться, что оно успешно завершается, прежде чем создавать дополнительные задачи.

Если задача завершается успешно, добавьте задачи развертывания, выполнив предыдущие шаги для каждого файла настраиваемой политики. Измените `-PolicyId` `-PathToFile` значения аргументов и для каждой политики.

`PolicyId`— Это значение, найденное в начале файла политики XML в узле TrustFrameworkPolicy. Например, `PolicyId` в следующем XML-файле политики *B2C_1A_TrustFrameworkBase*:

```xml
<TrustFrameworkPolicy
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:xsd="http://www.w3.org/2001/XMLSchema"
xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06"
PolicySchemaVersion="0.3.0.0"
TenantId="contoso.onmicrosoft.com"
PolicyId= "B2C_1A_TrustFrameworkBase"
PublicPolicyUri="http://contoso.onmicrosoft.com/B2C_1A_TrustFrameworkBase">
```

При запуске агентов и отправке файлов политики убедитесь, что они передаются в следующем порядке:

1. *TrustFrameworkBase.xml*
1. *TrustFrameworkExtensions.xml*
1. *SignUpOrSignin.xml*
1. *ProfileEdit.xml*
1. *PasswordReset.xml*

Инфраструктура процедур идентификации применяет этот порядок, так как структура файла построена в иерархической цепочке.

## <a name="test-your-pipeline"></a>Тестирование конвейера

Чтобы протестировать конвейер выпуска, выполните следующие действия.

1. Выберите **конвейеры** , а затем **выпуски**.
1. Выберите созданный ранее конвейер, например *деплойкустомполиЦиес*.
1. Выберите **создать выпуск**, а затем выберите **создать** , чтобы поставить в очередь выпуск.

Вы увидите баннер с уведомлением о том, что выпуск поставлен в очередь. Чтобы просмотреть его состояние, выберите ссылку в баннере уведомления или выберите ее в списке на вкладке **выпуски** .

## <a name="next-steps"></a>Дальнейшие действия

См. также:

* [Вызовы между службами с использованием учетных данных клиента](../active-directory/azuread-dev/v1-oauth2-client-creds-grant-flow.md)
* [Azure DevOps Services](/azure/devops/user-guide/)

<!-- LINKS - External -->
[devops]: /azure/devops/
[devops-create-project]:  /azure/devops/organizations/projects/create-project
[devops-pipelines]: /azure/devops/pipelines
