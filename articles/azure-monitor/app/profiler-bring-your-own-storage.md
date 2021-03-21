---
title: Настройка BYOS (перевод собственного хранилища) для & профилировщика Snapshot Debugger
description: Настройка BYOS (перевод собственного хранилища) для & профилировщика Snapshot Debugger
ms.topic: conceptual
author: renatosalas
ms.author: regutier
ms.date: 01/14/2021
ms.reviewer: mbullwin
ms.openlocfilehash: 9c3ff91cbfb6423099040a6ea46eeb66f5461f48
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100589673"
---
# <a name="configure-bring-your-own-storage-byos-for-application-insights-profiler-and-snapshot-debugger"></a>Настройка собственного хранилища (BYOS) для Application Insights Profiler и Snapshot Debugger

## <a name="what-is-bring-your-own-storage-byos-and-why-might-i-need-it"></a>Что такое хранилище (BYOS) и зачем оно может потребоваться? 
При использовании Application Insights Profiler или Snapshot Debugger артефакты, созданные приложением, передаются в учетные записи хранения Azure через общедоступный Интернет. Эти учетные записи оплачиваются и контролируются корпорацией Майкрософт для обработки и анализа. Корпорация Майкрософт управляет политиками шифрования неактивных данных и управления жизненным циклом для этих артефактов.

Используя свое собственное хранилище, эти артефакты отправляются в учетную запись хранения, которую вы контролируете. Это означает, что вы управляете политикой шифрования неактивных объектов, политикой управления жизненным циклом и доступом к сети. При этом вы несете ответственность за затраты, связанные с этой учетной записью хранения.

> [!NOTE]
> При включении закрытой ссылки необходимо использовать собственное хранилище. Дополнительные сведения о закрытой ссылке для Application Insights [см. в документации.](../logs/private-link-security.md)
>
> Если вы включаете ключи Customer-Managed, необходимо использовать собственное хранилище. Дополнительные сведения о ключах Customer-Managed для Application Insights [см. в документации.](../logs/customer-managed-keys.md)

## <a name="how-will-my-storage-account-be-accessed"></a>Как будет осуществляться доступ к учетной записи хранения?
1. Агенты, выполняющиеся на виртуальных машинах или в службе приложений, будут отправлять артефакты (профили, моментальные снимки и символы) в контейнеры больших двоичных объектов в вашей учетной записи. Этот процесс включает связь со службой Application Insights Profiler или Snapshot Debugger, чтобы получить маркер SAS (подписанный URL-адрес) для нового большого двоичного объекта в учетной записи хранения.
1. Служба Application Insights Profiler или Snapshot Debugger проанализирует входящий большой двоичный объект и выполнит обратную запись результатов анализа и файлов журнала в хранилище BLOB-объектов. В зависимости от доступной емкости вычислений этот процесс может происходить в любое время после отправки.
1. При просмотре трассировок профилировщика или анализе snapshot Debugger служба получает результаты анализа из хранилища BLOB-объектов.

## <a name="prerequisites"></a>Предварительные условия
* Не забудьте создать учетную запись хранения в том же расположении, что и ресурс Application Insights. Например: Если ресурс Application Insights находится в западной части США 2, ваша учетная запись хранения должна быть также в западной части США 2. 
* Предоставьте роли "участник данных BLOB-объекта хранилища" для приложения AAD "доступ к доверенному хранилищу служб диагностики" в учетной записи хранения через пользовательский интерфейс управления доступом (IAM).
* Если частная связь включена, Настройте дополнительный параметр, чтобы разрешить подключение к нашей доверенной службе Майкрософт из виртуальной сети. 

## <a name="how-to-enable-byos"></a>Включение BYOS

### <a name="create-storage-account"></a>Создание учетной записи хранения
Создайте новую учетную запись хранения (если у вас ее нет) в том же расположении, что и ресурс Application Insights.
Если ваш Application Insights ресурс включен `West US 2` , ваша учетная запись хранения должна быть в `West US 2` .

### <a name="grant-access-to-diagnostic-services-to-your-storage-account"></a>Предоставление доступа к службам диагностики учетной записи хранения
Учетная запись хранения BYOS будет связана с ресурсом Application Insights. Для каждого ресурса Application Insights может существовать только одна учетная запись хранения, и оба они должны находиться в одном расположении. Вы можете использовать одну и ту же учетную запись хранения с более чем одним Application Insights ресурсом.

Сначала службе Application Insights Profiler и Snapshot Debugger необходимо предоставить доступ к учетной записи хранения. Чтобы предоставить доступ, добавьте роль `Storage Blob Data Contributor` в приложение AAD с именем с `Diagnostic Services Trusted Storage Access` помощью страницы управления доступом (IAM) в учетной записи хранения, как показано на рис. 1,0. 

Шаги: 
1. Нажмите кнопку "Добавить" в разделе "Добавление назначения роли". 
1. Выберите роль "Участник хранилища для больших двоичных объектов" 
1. В разделе "назначение доступа к" выберите "пользователь Azure AD, группа или субъект-служба". 
1. Поиск & выберите приложение "доступ к доверенному хранилищу служб диагностики" 
1. Сохранение изменений

_![ Рис. 1,0](media/profiler-bring-your-own-storage/figure-10.png)_. 
 _рис. 1,0_ 

После добавления роли она появится в разделе "назначения ролей", как показано на рисунке 1,1 ниже. 
_![ Рис. 1,1](media/profiler-bring-your-own-storage/figure-11.png)_. 
 _рис. 1,1_ 

Если вы также используете частную ссылку, требуется одна дополнительная настройка, чтобы разрешить подключение к нашей доверенной службе Майкрософт из виртуальной сети. См. [документацию по сетевой безопасности хранилища](../../storage/common/storage-network-security.md#trusted-microsoft-services).

### <a name="link-your-storage-account-with-your-application-insights-resource"></a>Связывание учетной записи хранения с ресурсом Application Insights
Чтобы настроить BYOS для диагностики на уровне кода (профилировщик или отладчик), существует три варианта:

* Использование командлетов Azure PowerShell
* Использование интерфейса командной строки Azure (CLI)
* Использование шаблона Azure Resource Manager

#### <a name="configure-using-azure-powershell-cmdlets"></a>Настройка с помощью командлетов Azure PowerShell

1. Убедитесь, что вы установили команду AZ PowerShell 4.2.0 или более поздней версии.

    Сведения об установке Azure PowerShell см. в [официальной документации по Azure PowerShell](/powershell/azure/install-az-ps).

1. Установите расширение PowerShell для Application Insights.
    ```powershell
    Install-Module -Name Az.ApplicationInsights -Force
    ```

1. Войдите с помощью учетной записи Azure.
    ```powershell
    Connect-AzAccount -Subscription "{subscription_id}"
    ```

    Дополнительные сведения о том, как войти в систему, см. в [документации Connect-азаккаунт](/powershell/module/az.accounts/connect-azaccount).

1. Удалите предыдущую учетную запись хранения, связанную с ресурсом Application Insights.

    Шаблон.
    ```powershell
    $appInsights = Get-AzApplicationInsights -ResourceGroupName "{resource_group_name}" -Name "{application_insights_name}"
    Remove-AzApplicationInsightsLinkedStorageAccount -ResourceId $appInsights.Id
    ```

    Пример
    ```powershell
    $appInsights = Get-AzApplicationInsights -ResourceGroupName "byos-test" -Name "byos-test-westus2-ai"
    Remove-AzApplicationInsightsLinkedStorageAccount -ResourceId $appInsights.Id
    ```

1. Подключите свою учетную запись хранения к ресурсу Application Insights.
    
    Шаблон.
    ```powershell
    $storageAccount = Get-AzStorageAccount -ResourceGroupName "{resource_group_name}" -Name "{storage_account_name}"
    $appInsights = Get-AzApplicationInsights -ResourceGroupName "{resource_group_name}" -Name "{application_insights_name}"
    New-AzApplicationInsightsLinkedStorageAccount -ResourceId $appInsights.Id -LinkedStorageAccountResourceId $storageAccount.Id
    ```

    Пример
    ```powershell
    $storageAccount = Get-AzStorageAccount -ResourceGroupName "byos-test" -Name "byosteststoragewestus2"
    $appInsights = Get-AzApplicationInsights -ResourceGroupName "byos-test" -Name "byos-test-westus2-ai"
    New-AzApplicationInsightsLinkedStorageAccount -ResourceId $appInsights.Id -LinkedStorageAccountResourceId $storageAccount.Id
    ```

#### <a name="configure-using-azure-cli"></a>Настройка с использованием интерфейса командной строки Azure

1. Убедитесь, что установлен Azure CLI.

    Сведения об установке Azure CLI см. в [официальной документации по Azure CLI](/cli/azure/install-azure-cli).

1. Установите Application Insights расширение CLI.
    ```powershell
    az extension add -n application-insights
    ```

1. Подключите свою учетную запись хранения к ресурсу Application Insights.

    Шаблон.
    ```powershell
    az monitor app-insights component linked-storage link --resource-group "{resource_group_name}" --app "{application_insights_name}" --storage-account "{storage_account_name}"
    ```
    
    Пример
    ```powershell
    az monitor app-insights component linked-storage link --resource-group "byos-test" --app "byos-test-westus2-ai" --storage-account "byosteststoragewestus2"
    ```
    
    Ожидаемые выходные данные:
    ```powershell
    {
      "id": "/subscriptions/{subscription}/resourcegroups/byos-test/providers/microsoft.insights/components/byos-test-westus2-ai/linkedstorageaccounts/serviceprofiler",
      "linkedStorageAccount": "/subscriptions/{subscription}/resourceGroups/byos-test/providers/Microsoft.Storage/storageAccounts/byosteststoragewestus2",
      "name": "serviceprofiler",
      "resourceGroup": "byos-test",
      "type": "microsoft.insights/components/linkedstorageaccounts"
    }
    ```

    > [!NOTE]
    > Сведения о выполнении обновлений для связанных учетных записей хранения в ресурсе Application Insights см. в [документации по CLI Application Insights](/cli/azure/ext/application-insights/monitor/app-insights/component/linked-storage).

#### <a name="configure-using-azure-resource-manager-template"></a>Настройка с помощью шаблона Azure Resource Manager

1. Создайте файл шаблона Azure Resource Manager со следующим содержимым (byos.template.jsв).
    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "applicationinsights_name": {
          "type": "String"
        },
        "storageaccount_name": {
          "type": "String"
        }
      },
      "variables": {},
      "resources": [
        {
          "name": "[concat(parameters('applicationinsights_name'), '/serviceprofiler')]",
          "type": "Microsoft.Insights/components/linkedStorageAccounts",
          "apiVersion": "2020-03-01-preview",
          "properties": {
            "linkedStorageAccount": "[resourceId('Microsoft.Storage/storageAccounts', parameters('storageaccount_name'))]"
          }
        }
      ],
      "outputs": {}
    }
    ```

1. Выполните следующую команду PowerShell, чтобы развернуть предыдущий шаблон (создать связанную учетную запись хранения).

    Шаблон.
    ```powershell
    New-AzResourceGroupDeployment -ResourceGroupName "{your_resource_name}" -TemplateFile "{local_path_to_arm_template}"
    ```

    Пример
    ```powershell
    New-AzResourceGroupDeployment -ResourceGroupName "byos-test" -TemplateFile "D:\Docs\byos.template.json"
    ```

1. При появлении запроса в консоли PowerShell укажите следующие параметры:
    
    |           Параметр           |                                Описание                               |
    |-------------------------------|--------------------------------------------------------------------------|
    | application_insights_name     | Имя ресурса Application Insights для включения BYOS.            |
    | storage_account_name          | Имя ресурса учетной записи хранения, которое будет использоваться в качестве BYOS. |
    
    Ожидаемые выходные данные:
    ```powershell
    Supply values for the following parameters:
    (Type !? for Help.)
    application_insights_name: byos-test-westus2-ai
    storage_account_name: byosteststoragewestus2
    
    DeploymentName          : byos.template
    ResourceGroupName       : byos-test
    ProvisioningState       : Succeeded
    Timestamp               : 4/16/2020 1:24:57 AM
    Mode                    : Incremental
    TemplateLink            :
    Parameters              :
                              Name                            Type                       Value
                              ==============================  =========================  ==========
                              application_insights_name        String                     byos-test-westus2-ai
                              storage_account_name             String                     byosteststoragewestus2
                              
    Outputs                 :
    DeploymentDebugLogLevel :
    ```

1. Включите диагностику на уровне кода (профилировщик или отладчик) в интересующей рабочей нагрузке с помощью портал Azure. (> службы приложений Application Insights) _![ Рис. 2,0](media/profiler-bring-your-own-storage/figure-20.png)_. 
 _рис. 2,0_

## <a name="troubleshooting"></a>Устранение неполадок
### <a name="template-schema-schema_uri-isnt-supported"></a>Схема шаблона "{schema_uri}" не поддерживается.
* Убедитесь, что `$schema` свойство шаблона является допустимым. Он должен соответствовать следующему шаблону: `https://schema.management.azure.com/schemas/{schema_version}/deploymentTemplate.json#`
* Убедитесь, что `schema_version` шаблон находится в допустимых значениях: `2014-04-01-preview, 2015-01-01, 2018-05-01, 2019-04-01, 2019-08-01` .
    Сообщение об ошибке:
    ```powershell
    New-AzResourceGroupDeployment : 11:53:49 AM - Error: Code=InvalidTemplate; Message=Deployment template validation failed: 'Template schema
    'https://schema.management.azure.com/schemas/2020-01-01/deploymentTemplate.json#' is not supported. Supported versions are
    '2014-04-01-preview,2015-01-01,2018-05-01,2019-04-01,2019-08-01'. Please see https://aka.ms/arm-template for usage details.'.
    ```

### <a name="no-registered-resource-provider-found-for-location-location"></a>Не найден зарегистрированный поставщик ресурсов для расположения "{Location}".
* Убедитесь, что `apiVersion` ресурс `microsoft.insights/components` имеет значение `2015-05-01` .
* Убедитесь, что `apiVersion` ресурс `linkedStorageAccount` имеет значение `2020-03-01-preview` .
    Сообщение об ошибке:
    ```powershell
    New-AzResourceGroupDeployment : 6:18:03 PM - Resource microsoft.insights/components 'byos-test-westus2-ai' failed with message '{
      "error": {
        "code": "NoRegisteredProviderFound",
        "message": "No registered resource provider found for location 'westus2' and API version '2020-03-01-preview' for type 'components'. The supported api-versions are '2014-04-01,
    2014-08-01, 2014-12-01-preview, 2015-05-01, 2018-05-01-preview'. The supported locations are ', eastus, southcentralus, northeurope, westeurope, southeastasia, westus2, uksouth,
    canadacentral, centralindia, japaneast, australiaeast, koreacentral, francecentral, centralus, eastus2, eastasia, westus, southafricanorth, northcentralus, brazilsouth, switzerlandnorth,
    australiasoutheast'."
      }
    }'
    ```
### <a name="storage-account-location-should-match-ai-component-location"></a>Расположение учетной записи хранения должно соответствовать расположению компонента AI.
* Убедитесь, что расположение Application Insights ресурса совпадает с учетной записью хранения.
    Сообщение об ошибке:
    ```powershell
    New-AzResourceGroupDeployment : 1:01:12 PM - Resource microsoft.insights/components/linkedStorageAccounts 'byos-test-centralus-ai/serviceprofiler' failed with message '{
      "error": {
        "code": "BadRequest",
        "message": "Storage account location should match AI component location",
        "innererror": {
          "trace": [
            "System.ArgumentException"
          ]
        }
      }
    }'
    ```

Общие сведения об устранении неполадок профилировщика см. в [документации по устранению неполадок профилировщика](profiler-troubleshooting.md).

Общие Snapshot Debugger устранении неполадок см. в [документации по snapshot Debugger устранению неполадок](snapshot-debugger-troubleshoot.md). 

## <a name="faqs"></a>Часто задаваемые вопросы
* Если у меня есть профилировщик или моментальный снимок, а затем включен BYOS, данные будут перенесены в учетную запись хранения?
    _Нет, он не будет._

* Будет ли BYOS работать с шифрованием неактивных и Customer-Managed ключом?
    _Да, чтобы быть точным, BYOS является необходимым условием включения профилировщика или отладчика с Customer-Manager ключами._

* Будет ли BYOS работать в среде, изолированной от Интернета?
    _Да. На самом деле, BYOS является обязательным требованием для сценариев изолированной сети._

* Будет ли BYOS работать, когда включены и Customer-Managed ключи, и частная связь? 
    _Да, это возможно._

* Если я включил BYOS, можно ли вернуться к использованию учетных записей хранения служб диагностики для хранения собранных данных? 
    _Да, вы можете, но сейчас мы не поддерживаем перенос данных из BYOS._

* После включения BYOS отправит все связанные с ним затраты, которые являются хранилищем и сетью? 
    _Да_
