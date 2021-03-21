---
title: Обзор. Автоматизация развертывания для Azure Logic Apps
description: Сведения о шаблонах Azure Resource Manager для автоматизации развертывания Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: logicappspm
ms.topic: article
ms.date: 11/06/2020
ms.openlocfilehash: 2e1536d4f2ea7d71691c611e9127109c154f3266
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99807349"
---
# <a name="overview-automate-deployment-for-azure-logic-apps-by-using-azure-resource-manager-templates"></a>Обзор: Автоматизация развертывания для Azure Logic Apps с помощью шаблонов Azure Resource Manager

Когда вы будете готовы к автоматизации создания и развертывания приложения логики, вы можете расширить базовое определение рабочего процесса приложения логики в [шаблон Azure Resource Manager](../azure-resource-manager/management/overview.md). Этот шаблон определяет инфраструктуру, ресурсы, параметры и другую информацию для подготовки и развертывания приложения логики. Определяя параметры для значений, которые зависят от развертывания (также называются *параметризациями*), можно многократно и согласованно развертывать приложения логики на основе различных нужд развертывания.

Например, при развертывании в средах для разработки, тестирования и производства, скорее всего, будут использоваться разные строки подключения для каждой среды. Можно объявить параметры шаблона, которые принимают разные строки соединения, а затем сохранить эти строки в отдельном [файле параметров](../azure-resource-manager/templates/parameter-files.md). Таким образом, эти значения можно изменить без необходимости обновления и повторного развертывания шаблона. Для сценариев, в которых имеются конфиденциальные или защищенные значения параметров, например пароли и секреты, их можно сохранить в [Azure Key Vault](../azure-resource-manager/templates/key-vault-parameter.md) и получить эти значения с помощью файла параметров. Однако в этих сценариях необходимо повторно выполнить развертывание для получения текущих значений.

В этом обзоре описываются атрибуты в шаблоне диспетчер ресурсов, который включает определение рабочего процесса приложения логики. Как шаблон, так и определение рабочего процесса используют синтаксис JSON, но существуют некоторые отличия, поскольку определение рабочего процесса также соответствует [схеме языка определения рабочего процесса](../logic-apps/logic-apps-workflow-definition-language.md). Например, выражения шаблона и выражения определения рабочего процесса отличаются тем, как они [ссылаются на параметры](#parameter-references) и значения, которые они могут принимать.

> [!TIP]
> Чтобы получить доступ к правильному параметризованному шаблону приложения логики, который в основном готов к развертыванию, используйте Visual Studio (бесплатный Community Edition или более позднюю версию) и средства Azure Logic Apps для Visual Studio. Затем можно либо [создать приложение логики в Visual Studio](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md) , либо [найти и скачать существующее приложение логики из Azure в Visual Studio](../logic-apps/manage-logic-apps-with-visual-studio.md).
>
> Вы также можете создать шаблоны приложений логики, используя [Azure PowerShell с модулем логикапптемплате](../logic-apps/logic-apps-create-azure-resource-manager-templates.md#azure-powershell).

В примере приложения логики в этом разделе используется [триггер Outlook Office 365](/connectors/office365/) , который срабатывает при поступлении нового электронного письма и [действие хранилища BLOB-объектов Azure](/connectors/azureblob/) , которое создает большой двоичный объект для текста сообщения электронной почты и отправляет его в контейнер хранилища Azure. В примерах также показано, как параметризовать значения, которые зависят от развертывания.

Дополнительные сведения о шаблонах диспетчер ресурсов см. в следующих разделах:

* [Структура и синтаксис шаблона Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md)
* [Рекомендации по работе с шаблонами Azure Resource Manager](../azure-resource-manager/templates/template-best-practices.md)
* [Разработка шаблонов Azure Resource Manager для обеспечения согласованности с облаком](../azure-resource-manager/templates/templates-cloud-consistency.md)

Сведения о ресурсах шаблонов, связанных с приложениями логики, учетными записями интеграции, артефактами учетной записи интеграции и средами службы интеграции, см. в разделе [типы ресурсов Microsoft. Logic](/azure/templates/microsoft.logic/allversions).

Примеры шаблонов приложений логики см. в следующих примерах:

* [Полный шаблон](#full-example-template) , используемый для примеров этого раздела
* [Пример шаблона учебного приложения логики](https://github.com/Azure/azure-quickstart-templates/blob/master/101-logic-app-create) в GitHub

Для Logic Apps REST API Начните с [обзора Azure Logic Apps REST API](/rest/api/logic).

<a name="template-structure"></a>

## <a name="template-structure"></a>Структура шаблона

На верхнем уровне шаблон диспетчер ресурсов соответствует этой структуре, которая полностью описана в разделе [Структура шаблона Azure Resource Manager и раздел синтаксиса](../azure-resource-manager/templates/template-syntax.md) :

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {},
   "variables": {},
   "functions": [],
   "resources": [],
   "outputs": {}
}
```

Для шаблона приложения логики вы в основном работаете с этими объектами шаблона:

| Атрибут | Описание |
|-----------|-------------|
| `parameters` | Объявляет [Параметры шаблона](../azure-resource-manager/templates/template-syntax.md#parameters) для приема значений, используемых при создании и настройке ресурсов для развертывания в Azure. Например, эти параметры принимают значения для имени и расположения приложения логики, а также подключения и другие ресурсы, необходимые для развертывания. Эти значения параметров можно сохранить в [файле параметров](#template-parameter-files), который описывается далее в этом разделе. Общие сведения см. в разделе [Parameters-диспетчер ресурсов структура шаблона и синтаксис](../azure-resource-manager/templates/template-syntax.md#parameters). |
| `resources` | Определяет [ресурсы](../azure-resource-manager/templates/template-syntax.md#resources) для создания или обновления и развертывания в группе ресурсов Azure, такие как приложение логики, подключения, учетные записи хранения Azure и т. д. Общие сведения см. в разделе [ресурсы-Диспетчер ресурсов структура шаблона и синтаксис](../azure-resource-manager/templates/template-syntax.md#resources). |
||||

В шаблоне приложения логики используется следующий формат имени файла:

**<*Logic-App-Name* # C0.json**

> [!IMPORTANT]
> Синтаксис шаблона учитывает регистр, поэтому убедитесь, что используется последовательный регистр. 

<a name="template-parameters"></a>

## <a name="template-parameters"></a>Параметры шаблона

Шаблон приложения логики имеет несколько `parameters` объектов, которые существуют на разных уровнях и выполняют различные функции. Например, на верхнем уровне можно объявить [Параметры шаблона](../azure-resource-manager/templates/template-syntax.md#parameters) для значений, которые будут приниматься и использоваться при развертывании при создании и развертывании ресурсов в Azure, например:

* Приложение логики
* Подключения, используемые логикой для доступа к другим службам и системам через [управляемые соединители](../connectors/apis-list.md)
* Другие ресурсы, необходимые для развертывания приложением логики

  Например, если приложение логики использует [учетную запись интеграции](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) для сценариев B2B, то объект верхнего уровня шаблона `parameters` объявляет параметр, который принимает идентификатор ресурса для этой учетной записи интеграции.

Ниже приведена общая структура и синтаксис для определения параметра, который полностью описан в разделе [Parameters-диспетчер ресурсов структура шаблона и синтаксис](../azure-resource-manager/templates/template-syntax.md#parameters):

```json
"<parameter-name>": {
   "type": "<parameter-type>",
   "defaultValue": <default-parameter-value>,
   <other-parameter-attributes>,
   "metadata": {
      "description": "<parameter-description>"
   }
},
```

В этом примере показаны только параметры шаблона для значений, используемых для создания и развертывания этих ресурсов в Azure:

* Имя и расположение приложения логики
* Идентификатор, используемый для учетной записи интеграции, связанной с приложением логики

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   // Template parameters
   "parameters": {
      "LogicAppName": {
         "type": "string",
         "minLength": 1,
         "maxLength": 80,
         "defaultValue": "MyLogicApp",
         "metadata": {
            "description": "The resource name for the logic app"
         }
      },
      "LogicAppLocation": {
         "type": "string",
         "minLength": 1,
         "defaultValue": "[resourceGroup().location]",
         "metadata": {
            "description": "The resource location for the logic app"
         }
      },
      "LogicAppIntegrationAccount": {
         "type":"string",
         "minLength": 1,
         "defaultValue": "/subscriptions/<Azure-subscription-ID>/resourceGroups/fabrikam-integration-account-rg/providers/Microsoft.Logic/integrationAccounts/fabrikam-integration-account",
         "metadata": {
            "description": "The ID to use for the integration account"
         }
      }
   },
   "variables": {},
   "functions": [],
   "resources": [],
   "outputs": {}
}
```

За исключением параметров, обрабатывающих конфиденциальные или защищенные значения, такие как имена пользователей, пароли и секреты, все эти параметры включают `defaultValue` атрибуты, хотя в некоторых случаях значения по умолчанию являются пустыми значениями. Значения развертывания, используемые для этих параметров шаблона, предоставляются в [файле примеров параметров](#template-parameter-files) , описанном далее в этом разделе.

Дополнительные сведения о защите параметров шаблонов см. в следующих разделах:

* [Рекомендации по безопасности для параметров шаблона](../azure-resource-manager/templates/template-best-practices.md#parameters)
* [Повышение безопасности параметров шаблона](../logic-apps/logic-apps-securing-a-logic-app.md#secure-parameters-deployment-template)
* [Передача защищенных значений параметров с помощью Azure Key Vault](../azure-resource-manager/templates/key-vault-parameter.md)

Другие объекты шаблона часто ссылаются на параметры шаблона, чтобы они могли использовать значения, которые проходят через параметры шаблона, например:

* [Объект ресурсов вашего шаблона](#template-resources), описанный далее в этом разделе, определяет каждый ресурс в Azure, который требуется создать и развернуть, например [Определение ресурса приложения логики](#logic-app-resource-definition). Эти ресурсы часто используют значения параметров шаблона, такие как имя и расположение приложения логики, а также сведения о соединении.

* На более глубоком уровне в определении ресурса приложения логики [объект параметров определения рабочего процесса](#workflow-definition-parameters) объявляет параметры для значений, используемых в среде выполнения приложения логики. Например, можно объявить параметры определения рабочего процесса для имени пользователя и пароля, которые триггер HTTP использует для проверки подлинности. Чтобы указать значения для параметров определения рабочего процесса, используйте `parameters` объект, *находящийся за пределами* определения рабочего процесса, но по-прежнему *находящийся в* определении ресурса приложения логики. В этом внешнем `parameters` объекте можно ссылаться на ранее объявленные параметры шаблона, которые могут принимать значения при развертывании из файла параметров.

При ссылке на параметры выражения и функции шаблона используют другой синтаксис и ведут себя иначе в выражениях и функциях определения рабочего процесса. Дополнительные сведения об этих различиях см. Далее в подразделе [ссылки на параметры](#parameter-references) .

<a name="best-practices-template-parameters"></a>

## <a name="best-practices---template-parameters"></a>Рекомендации — параметры шаблона

Ниже приведены некоторые рекомендации по определению параметров.

* Объявите параметры только для значений, которые различаются в зависимости от потребностей развертывания. Не объявляйте параметры для значений, которые остаются одинаковыми в разных требованиях к развертыванию.

* Включите `defaultValue` атрибут, который может указывать пустые значения для всех параметров, кроме значений, которые являются конфиденциальными или должны быть защищены. Всегда используйте защищенные параметры для имен пользователей, паролей и секретов. Чтобы скрыть или защитить конфиденциальные значения параметров, следуйте указаниям в следующих разделах:

  * [Рекомендации по безопасности для параметров шаблона](../azure-resource-manager/templates/template-best-practices.md#parameters)

  * [Повышение безопасности параметров шаблона](../logic-apps/logic-apps-securing-a-logic-app.md#secure-parameters-deployment-template)

  * [Передача защищенных значений параметров с помощью Azure Key Vault](../azure-resource-manager/templates/key-vault-parameter.md)

* Чтобы отличить имена параметров шаблона от имен параметров определения рабочего процесса, можно использовать описательные имена параметров шаблонов, например: `TemplateFabrikamPassword`

Дополнительные рекомендации по использованию шаблонов см. в разделе рекомендации [по использованию параметров шаблона](../azure-resource-manager/templates/template-best-practices.md#parameters).

<a name="template-parameter-files"></a>

## <a name="template-parameters-file"></a>Файл параметров шаблона

Чтобы предоставить значения для параметров шаблона, сохраните эти значения в [файле параметров](../azure-resource-manager/templates/parameter-files.md). Таким образом, можно использовать различные файлы параметров в зависимости от потребностей развертывания. Ниже приведен формат имени файла для использования:

* Имя файла шаблона приложения логики: **< *Logic-App-Name* # C0.json**
* Имя файла параметров: **< *Logic-App-Name* # C0.parameters.json**

Ниже приведена структура в файле параметров, которая включает ссылку на хранилище ключей для [передачи защищенного значения параметра с Azure Key Vault](../azure-resource-manager/templates/key-vault-parameter.md):

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
   "contentVersion": "1.0.0.0",
   // Template parameter values
   "parameters": {
      "<parameter-name-1>": {
         "value": "<parameter-value>"
      },
      "<parameter-name-2>": {
         "value": "<parameter-value>"
      },
      "<secured-parameter-name>": {
         "reference": {
            "keyVault": {
               "id": "/subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group-name>/Microsoft.KeyVault/vaults/<key-vault-name>",
            },
            "secretName: "<secret-name>"
         }
      },
      <other-parameter-values>
   }
}
```

В этом примере файла параметров указываются значения для параметров шаблона, объявленных ранее в этом разделе:

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
   "contentVersion": "1.0.0.0",
   // Template parameter values
   "parameters": {
      "LogicAppName": {
         "value": "Email-Processor-Logic-App"
      },
      "LogicAppLocation": {
         "value": "westeurope"
      }
   }
}
```

<a name="template-resources"></a>

## <a name="template-resources"></a>Ресурсы шаблона

В шаблоне есть `resources` объект, который содержит определения для каждого ресурса, создаваемого и развертываемого в Azure, например [Определение ресурса приложения логики](#logic-app-resource-definition), [определения ресурсов подключения](#connection-resource-definitions)и другие ресурсы, необходимые приложению логики для развертывания.

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {<template-parameters>},
   "variables": {},
   "functions": [],
   "resources": [
      {
         <logic-app-resource-definition>
      },
      // Start connection resource definitions
      {
         <connection-resource-definition-1>
      },
      {
         <connection-resource-definition-2>
      }
   ],
   "outputs": {}
}
```

> [!NOTE]
> Шаблоны могут включать определения ресурсов для нескольких приложений логики, поэтому убедитесь, что все ресурсы приложения логики указывают одну и ту же группу ресурсов Azure. При развертывании шаблона в группе ресурсов Azure с помощью Visual Studio появится запрос на выбор приложения логики, которое необходимо открыть. Кроме того, проект группы ресурсов Azure может содержать более одного шаблона, поэтому при появлении запроса убедитесь, что выбран правильный файл параметров.

<a name="view-resource-definitions"></a>

### <a name="view-resource-definitions"></a>Просмотр определений ресурсов

Чтобы проверить определения ресурсов для всех ресурсов в группе ресурсов Azure, [скачайте приложение логики из Azure в Visual Studio](../logic-apps/manage-logic-apps-with-visual-studio.md). это самый простой способ создать допустимый параметризованный шаблон приложения логики, который в основном готов к развертыванию.

Общие сведения о шаблонных ресурсах и их атрибутах см. в следующих разделах:

* [Ресурсы — структура и синтаксис шаблона диспетчер ресурсов](../azure-resource-manager/templates/template-syntax.md#resources)
* [Рекомендации по использованию ресурсов шаблонов](../azure-resource-manager/templates/template-best-practices.md#resources)

<a name="logic-app-resource-definition"></a>

### <a name="logic-app-resource-definition"></a>Определение ресурса приложения логики

[Определение ресурса рабочего процесса приложения логики в шаблоне](/azure/templates/microsoft.logic/workflows) начинается с `properties` объекта, который содержит следующие сведения:

* Состояние приложения логики при развертывании
* Идентификатор любой учетной записи интеграции, используемой приложением логики
* Определение рабочего процесса приложения логики
* `parameters`Объект, который задает значения для использования во время выполнения
* Другие сведения о ресурсах приложения логики, такие как имя, тип, расположение, параметры конфигурации среды выполнения и т. д.

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {<template-parameters>},
   "variables": {},
   "functions": [],
   "resources": [
      {
         // Start logic app resource definition
         "properties": {
            "state": "<Enabled or Disabled>",
            "integrationAccount": {
               "id": "[parameters('LogicAppIntegrationAccount')]" // Template parameter reference
            },
            "definition": {<workflow-definition>},
            "parameters": {<workflow-definition-parameter-values>},
            "accessControl": {},
            "runtimeConfiguration": {}
         },
         "name": "[parameters('LogicAppName')]", // Template parameter reference
         "type": "Microsoft.Logic/workflows",
         "location": "[parameters('LogicAppLocation')]", // Template parameter reference
         "tags": {
           "displayName": "LogicApp"
         },
         "apiVersion": "2016-06-01",
         "dependsOn": [
         ]
      }
      // End logic app resource definition
   ],
   "outputs": {}
}
```

Ниже приведены атрибуты, относящиеся к определению ресурса приложения логики.

| Атрибут | Обязательно | Тип | Описание |
|-----------|----------|------|-------------|
| `state` | Да | Строка | Состояние приложения логики при развертывании `Enabled` . Это означает, что приложение логики работает в реальном времени и `Disabled` означает, что приложение логики неактивно. Например, если вы не готовы к работе с приложением логики, но хотите развернуть черновую версию, можно использовать `Disabled` параметр. |
| `integrationAccount` | Нет | Объект | Если приложение логики использует учетную запись интеграции, которая хранит артефакты для сценариев "бизнес — бизнес" (B2B), этот объект включает `id` атрибут, указывающий идентификатор для учетной записи интеграции. |
| `definition` | Да | Объект | Базовое определение рабочего процесса приложения логики, которое является тем же объектом, которое отображается в представлении кода, и полностью описано в [справочнике по схемам в разделе Язык определения рабочего процесса](../logic-apps/logic-apps-workflow-definition-language.md) . В этом определении рабочего процесса `parameters` объект объявляет параметры для значений, используемых в среде выполнения приложения логики. Дополнительные сведения см. в разделе [Определение рабочего процесса и параметры](#workflow-definition-parameters). <p><p>Чтобы просмотреть атрибуты в определении рабочего процесса приложения логики, переключитесь с "представление конструктора" на "представление кода" в портал Azure или Visual Studio или с помощью такого средства, как [Обозреватель ресурсов Azure](https://resources.azure.com). |
| `parameters` | Нет | Объект | [Значения параметров определения рабочего процесса](#workflow-definition-parameters) , используемые во время выполнения приложения логики. Определения параметров для этих значений отображаются в [объекте параметров определения рабочего процесса](#workflow-definition-parameters). Кроме того, если приложение логики использует [управляемые соединители](../connectors/apis-list.md) для доступа к другим службам и системам, этот объект включает `$connections` объект, который задает значения соединения для использования во время выполнения. |
| `accessControl` | Нет | Объект | Для указания атрибутов безопасности приложения логики, таких как ограниченный доступ по протоколу IP к триггерам запросов или входные и выходные данные журнала выполнения. Дополнительные сведения см. [в статье безопасный доступ к Logic Apps](../logic-apps/logic-apps-securing-a-logic-app.md). |
| `runtimeConfiguration` | Нет | Объект | Для указания `operationOptions` свойств, управляющих тем, как приложение логики ведет себя во время выполнения. Например, приложение логики можно запустить в [режиме высокой пропускной способности](../logic-apps/logic-apps-limits-and-config.md#run-high-throughput-mode). |
|||||

Дополнительные сведения об определениях ресурсов для этих Logic Apps объектов см. в разделе [типы ресурсов Microsoft. Logic](/azure/templates/microsoft.logic/allversions):

* [Определение ресурса рабочего процесса](/azure/templates/microsoft.logic/workflows)
* [Определение ресурса среды службы интеграции](/azure/templates/microsoft.logic/integrationserviceenvironments)
* [Определение ресурса управляемого API среды службы интеграции](/azure/templates/microsoft.logic/integrationserviceenvironments/managedapis)

* [Определение ресурса для учетной записи интеграции](/azure/templates/microsoft.logic/integrationaccounts)

* Артефакты учетной записи интеграции:

  * [Определение ресурса соглашения](/azure/templates/microsoft.logic/integrationaccounts/agreements)

  * [Определение ресурса сборки](/azure/templates/microsoft.logic/integrationaccounts/assemblies)

  * [Определение ресурса конфигурации пакетной службы](/azure/templates/microsoft.logic/integrationaccounts/batchconfigurations)

  * [Определение ресурса сертификата](/azure/templates/microsoft.logic/integrationaccounts/certificates)

  * [Определение ресурса Map](/azure/templates/microsoft.logic/integrationaccounts/maps)

  * [Определение ресурса партнера](/azure/templates/microsoft.logic/integrationaccounts/partners)

  * [Определение ресурса схемы](/azure/templates/microsoft.logic/integrationaccounts/schemas)

  * [Определение ресурса сеанса](/azure/templates/microsoft.logic/integrationaccounts/sessions)

<a name="workflow-definition-parameters"></a>

## <a name="workflow-definition-and-parameters"></a>Определение и параметры рабочего процесса

Определение рабочего процесса приложения логики отображается в `definition` объекте, который отображается в `properties` объекте внутри определения ресурса приложения логики. Этот `definition` объект является тем же объектом, который отображается в представлении кода, и полностью описан в [справочнике по схемам в разделе Язык определения рабочего процесса](../logic-apps/logic-apps-workflow-definition-language.md) . Определение рабочего процесса включает объект внутреннего `parameters` объявления, в котором можно определить новые или изменить существующие параметры для значений, используемых определением рабочего процесса во время выполнения. Затем можно сослаться на эти параметры внутри триггера или действий в рабочем процессе. По умолчанию этот `parameters` объект пуст, если приложение логики не создает подключения к другим службам и системам через [управляемые соединители](../connectors/apis-list.md).

Чтобы задать значения для параметров определения рабочего процесса, используйте `parameters` объект, *находящийся за пределами* определения рабочего процесса, но по-прежнему *находящийся в* определении ресурса приложения логики. В этом внешнем `parameters` объекте можно сослаться на ранее объявленные параметры шаблона, которые могут принимать значения при развертывании из файла параметров.

> [!TIP]
>
> Рекомендуется не ссылаться непосредственно на параметры шаблона, которые оцениваются при развертывании, из определения рабочего процесса. Вместо этого объявите параметр определения рабочего процесса, который затем можно задать в `parameters` объекте *вне* определения рабочего процесса, но по-прежнему *в* определении ресурса приложения логики. Дополнительные сведения см. [в разделе ссылки на параметры](#parameter-references).

Этот синтаксис показывает, где можно объявить параметры на уровнях определения шаблона и рабочего процесса, а также где можно задать значения этих параметров, ссылаясь на параметры определения шаблона и рабочего процесса:

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   // Template parameters
   "parameters": {
      "<template-parameter-name>": {
         "type": "<parameter-type>",
         "defaultValue": "<parameter-default-value>",
         "metadata": {
            "description": "<parameter-description>"
         }
      }
   },
   "variables": {},
   "functions": [],
   "resources": [
      {
         // Start logic app resource definition
         "properties": {
            <other-logic-app-resource-properties>,
            "definition": {
               "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
               "actions": {<action-definitions>},
               // Workflow definition parameters
               "parameters": {
                  "<workflow-definition-parameter-name>": {
                     "type": "<parameter-type>",
                     "defaultValue": "<parameter-default-value>",
                     "metadata": {
                        "description": "<parameter-description>"
                     }
                  }
               },
               "triggers": {
                  "<trigger-name>": {
                     "type": "<trigger-type>",
                     "inputs": {
                         // Workflow definition parameter reference
                         "<attribute-name>": "@parameters('<workflow-definition-parameter-name')"
                     }
                  }
               },
               <...>
            },
            // Workflow definition parameter value
            "parameters": {
               "<workflow-definition-parameter-name>": { 
                  "value": "[parameters('<template-parameter-name>')]"
               }
            },
            "accessControl": {}
         },
         <other-logic-app-resource-definition-attributes>
      }
      // End logic app resource definition
   ],
   "outputs": {}
}
```

<a name="secure-workflow-definition-parameters"></a>

### <a name="secure-workflow-definition-parameters"></a>Параметры определения безопасного рабочего процесса

Для параметра определения рабочего процесса, который обрабатывает конфиденциальную информацию, пароли, ключи доступа или секреты во время выполнения, объявите или измените параметр для `securestring` использования `secureobject` типа параметра или. Вы можете ссылаться на этот параметр по всему определению рабочего процесса и в нем. На верхнем уровне шаблона объявите параметр, имеющий тот же тип для работы с этими сведениями при развертывании.

Чтобы задать значение для параметра определения рабочего процесса, используйте `parameters` объект, *находящийся за пределами* определения рабочего процесса, но по-прежнему *находящийся* в определении ресурса приложения логики, для ссылки на параметр шаблона. Наконец, чтобы передать значение в параметр шаблона во время развертывания, сохраните это значение в [Azure Key Vault](../azure-resource-manager/templates/key-vault-parameter.md) и укажите в [файле параметров](#template-parameter-files) , который используется шаблоном при развертывании, ссылку на это хранилище ключей.

В этом примере шаблона показано, как можно выполнить эти задачи, определив защищенные параметры, чтобы можно было хранить их значения в Azure Key Vault:

* Объявите защищенные параметры для значений, используемых для проверки подлинности доступа.
* Используйте эти значения на уровнях определения шаблона и рабочего процесса.
* Укажите эти значения с помощью файла параметров.

**Шаблон**

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {
      <previously-defined-template-parameters>,
      // Additional template parameters for passing values to use in workflow definition
      "TemplateAuthenticationType": {
         "type": "string",
         "defaultValue": "",
         "metadata": {
            "description": "The type of authentication used for the Fabrikam portal"
         }
      },
      "TemplateFabrikamPassword": {
         "type": "securestring",
         "metadata": {
            "description": "The password for the Fabrikam portal"
         }
      },
      "TemplateFabrikamUserName": {
         "type": "securestring",
         "metadata": {
            "description": "The username for the Fabrikam portal"
         }
      }
   },
   "variables": {},
   "functions": [],
   "resources": [
      {
         // Start logic app resource definition
         "properties": {
            <other-logic-app-resource-properties>,
            // Start workflow definition
            "definition": {
               "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
               "actions": {<action-definitions>},
               // Workflow definition parameters
               "parameters": {
                  "authenticationType": {
                     "type": "string",
                     "defaultValue": "",
                     "metadata": {
                        "description": "The type of authentication used for the Fabrikam portal"
                     }
                  },
                  "fabrikamPassword": {
                     "type": "securestring",
                     "metadata": {
                        "description": "The password for the Fabrikam portal"
                     }
                  },
                  "fabrikamUserName": {
                     "type": "securestring",
                     "metadata": {
                        "description": "The username for the Fabrikam portal"
                     }
                  }
               },
               "triggers": {
                  "HTTP": {
                     "inputs": {
                        "authentication": {
                           // Reference workflow definition parameters
                           "password": "@parameters('fabrikamPassword')",
                           "type": "@parameters('authenticationType')",
                           "username": "@parameters('fabrikamUserName')"
                        }
                     },
                     "recurrence": {<...>},
                     "type": "Http"
                  }
               },
               <...>
            },
            // End workflow definition
            // Start workflow definition parameter values
            "parameters": {
               "authenticationType": "[parameters('TemplateAuthenticationType')]", // Template parameter reference
               "fabrikamPassword": "[parameters('TemplateFabrikamPassword')]", // Template parameter reference
               "fabrikamUserName": "[parameters('TemplateFabrikamUserName')]" // Template parameter reference
            },
            "accessControl": {}
         },
         <other-logic-app-resource-attributes>
      }
      // End logic app resource definition
   ],
   "outputs": {}
}
```

**Файл параметров**

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
   "contentVersion": "1.0.0.0",
   // Template parameter values
   "parameters": {
      <previously-defined-template-parameter-values>,
     "TemplateAuthenticationType": {
        "value": "Basic"
     },
     "TemplateFabrikamPassword": {
        "reference": {
           "keyVault": {
              "id": "/subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group-name>/Microsoft.KeyVault/vaults/fabrikam-key-vault"
           },
           "secretName": "FabrikamPassword"
        }
     },
     "TemplateFabrikamUserName": {
        "reference": {
           "keyVault": {
              "id": "/subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group-name>/Microsoft.KeyVault/vaults/fabrikam-key-vault"
           },
           "secretName": "FabrikamUserName"
        }
     }
   }
}
```

<a name="best-practices-workflow-definition-parameters"></a>

## <a name="best-practices---workflow-definition-parameters"></a>Рекомендации. параметры определения рабочего процесса

Чтобы убедиться, что конструктор приложений логики может правильно отображать параметры определения рабочего процесса, следуйте приведенным ниже рекомендациям.

* Включите `defaultValue` атрибут, который может указывать пустые значения для всех параметров, кроме значений, которые являются конфиденциальными или должны быть защищены.

* Всегда используйте защищенные параметры для имен пользователей, паролей и секретов. Чтобы скрыть или защитить конфиденциальные значения параметров, следуйте указаниям в следующих разделах:

  * [Рекомендации по безопасности для параметров действий](../logic-apps/logic-apps-securing-a-logic-app.md#secure-action-parameters)

  * [Рекомендации по безопасности для параметров в определениях рабочих процессов](../logic-apps/logic-apps-securing-a-logic-app.md#secure-parameters-workflow)

  * [Передача безопасных значений параметров с помощью Azure Key Vault](../azure-resource-manager/templates/key-vault-parameter.md)

Дополнительные сведения о параметрах определения рабочего процесса см. в разделе [Параметры — язык определения рабочего процесса](../logic-apps/logic-apps-workflow-definition-language.md#parameters).

<a name="connection-resource-definitions"></a>

## <a name="connection-resource-definitions"></a>Определения ресурсов подключения

Когда приложение логики создает и использует подключения к другим службам и системам с помощью [управляемых соединителей](../connectors/apis-list.md), `resources` объект шаблона содержит определения ресурсов для этих подключений. Хотя подключения создаются в приложении логики, подключения представляют собой отдельные ресурсы Azure с собственными определениями ресурсов. Чтобы проверить эти определения ресурсов подключения, [скачайте приложение логики из Azure в Visual Studio](../logic-apps/manage-logic-apps-with-visual-studio.md). это самый простой способ создать допустимый параметризованный шаблон приложения логики, который в основном готов к развертыванию.

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {<template-parameters>},
   "variables": {},
   "functions": [],
   "resources": [
      {
         <logic-app-resource-definition>
      },
      // Start connection resource definitions
      {
         <connection-resource-definition-1>
      },
      {
         <connection-resource-definition-2>
      }
   ],
   "outputs": {}
}
```

Определения ресурсов подключения ссылаются на параметры верхнего уровня шаблона, чтобы эти значения можно было указать при развертывании с помощью файла параметров. Убедитесь, что в подключениях используется та же группа ресурсов Azure и расположение, что и для приложения логики.

Ниже приведен пример определения ресурса для подключения Outlook Office 365 и соответствующих параметров шаблона.

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   // Template parameters
   "parameters": {
      "LogicAppName": {<parameter-definition>},
      "LogicAppLocation": {<parameter-definition>},
      "office365_1_Connection_Name": {
         "type": "string",
         "defaultValue": "office365",
         "metadata": {
            "description": "The resource name for the Office 365 Outlook connection"
         }
      },
      "office365_1_Connection_DisplayName": {
         "type": "string",
         "defaultValue": "",
         "metadata": {
            "description": "The display name for the Office 365 Outlook connection"
         }
      }
   },
   "variables": {},
   "functions": [],
   "resources": [
      {
         <logic-app-resource-definition>
      },
      // Office 365 Outlook API connection resource definition
      {
         "type": "MICROSOFT.WEB/CONNECTIONS",
         "apiVersion": "2016-06-01",
         // Template parameter reference for connection name
         "name": "[parameters('office365_1_Connection_Name')]",
         // Template parameter reference for connection resource location. Must match logic app location.
         "location": "[parameters('LogicAppLocation')]",
         "properties": {
            "api": {
               // Connector ID
               "id": "[concat(subscription().id, '/providers/Microsoft.Web/locations/', parameters('LogicAppLocation'), '/managedApis/', 'office365')]"
            },
            // Template parameter reference for connection display name
            "displayName": "[parameters('office365_1_Connection_DisplayName')]"
         }
      }
   ],
   "outputs": {}
}
```

Определение ресурса приложения логики также работает с определениями ресурсов подключения следующим образом:

* Внутри определения рабочего процесса `parameters` объект объявляет `$connections` параметр для значений соединения для использования в среде выполнения приложения логики. Кроме того, триггер или действие, которое создает соединение, использует соответствующие значения, которые проходят через этот `$connections` параметр.

* *За пределами* определения рабочего процесса, но по-прежнему *внутри* определения ресурса приложения логики другой `parameters` объект задает значения, которые будут использоваться во время выполнения для `$connections` параметра, ссылаясь на соответствующие параметры шаблона. Эти значения используют выражения шаблона для ссылки на ресурсы, которые безопасно хранят метаданные для подключений в приложении логики.

  Например, метаданные могут содержать строки подключения и маркеры доступа, которые можно хранить в [Azure Key Vault](../azure-resource-manager/templates/key-vault-parameter.md). Чтобы передать эти значения в параметры шаблона, вы ссылаетесь на это хранилище ключей в [файле параметров](#template-parameter-files) , который используется шаблоном при развертывании. Дополнительные сведения о различиях между ссылочными параметрами см. в подразделе [ссылки на параметры](#parameter-references) далее в этой статье.

  При открытии определения рабочего процесса приложения логики в представлении кода с помощью портал Azure или Visual Studio `$connections` объект отображается за пределами определения рабочего процесса, но на том же уровне. Этот порядок в представлении кода упрощает ссылку на эти параметры при ручном обновлении определения рабочего процесса:

  ```json
  {
     "$connections": {<workflow-definition-parameter-connection-values-runtime},
     "definition": {<workflow-definition>}
  }
  ```

* Определение ресурса приложения логики содержит `dependsOn` объект, указывающий зависимости от подключений, используемых приложением логики.

Каждое создаваемое подключение имеет уникальное имя в Azure. При создании нескольких подключений к одной и той же службе или системе каждое имя соединения добавляется с числом, которое увеличивается на новое созданное соединение, например,, `office365` `office365-1` и т. д.

В этом примере показано взаимодействие между определением ресурса приложения логики и определением ресурса подключения для Office 365 Outlook.

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   // Template parameters
   "parameters": {
      "LogicAppName": {<parameter-definition>},
      "LogicAppLocation": {<parameter-definition>},
      "office365_1_Connection_Name": {<parameter-definition>},
      "office365_1_Connection_DisplayName": {<parameter-definition>}
   },
   "variables": {},
   "functions": [],
   "resources": [
      {
         // Start logic app resource definition
         "properties": {
            <...>,
            "definition": {
               <...>,
               "parameters": {
                  // Workflow definition "$connections" parameter
                  "$connections": {
                     "defaultValue": {},
                     "type": "Object"
                  }
               },
               <...>
            },
            "parameters": {
               // Workflow definition "$connections" parameter values to use at runtime
               "$connections": {
                  "value": {
                     "office365": {
                        // Template parameter references
                        "id": "[concat(subscription().id, '/providers/Microsoft.Web/locations/', parameters('LogicAppLocation'), '/managedApis/', 'office365')]",
                        "connectionId": "[resourceId('Microsoft.Web/connections', parameters('office365_1_Connection_Name'))]",
                        "connectionName": "[parameters('office365_1_Connection_Name')]"
                     }
                  }
               }
            }
         },
         <other-logic-app-resource-information>,
         "dependsOn": [
            "[resourceId('Microsoft.Web/connections', parameters('office365_1_Connection_Name'))]"
         ]
         // End logic app resource definition
      },
      // Office 365 Outlook API connection resource definition
      {
         "type": "MICROSOFT.WEB/CONNECTIONS",
         "apiVersion": "2016-06-01",
         // Template parameter reference for connection name
         "name": "[parameters('office365_1_Connection_Name')]",
         // Template parameter reference for connection resource location. Must match logic app location.
         "location": "[parameters('LogicAppLocation')]",
         "properties": {
            "api": {
               // Connector ID
               "id": "[concat(subscription().id, '/providers/Microsoft.Web/locations/', parameters('LogicAppLocation'), '/managedApis/', 'office365')]"
            },
            // Template parameter reference for connection display name
            "displayName": "[parameters('office365_1_Connection_DisplayName')]"
         }
      }
   ],
   "outputs": {}
}
```

<a name="secure-connection-parameters"></a>

### <a name="secure-connection-parameters"></a>Параметры безопасного подключения

Для параметра подключения, который обрабатывает конфиденциальную информацию, пароли, ключи доступа или секреты, определение ресурса подключения включает `parameterValues` объект, указывающий эти значения в формате пар «имя-значение». Чтобы скрыть эти сведения, можно объявить или изменить параметры шаблона для этих значений с помощью `securestring` `secureobject` типов параметров или. Затем эти сведения можно сохранить в [Azure Key Vault](../azure-resource-manager/templates/key-vault-parameter.md). Чтобы передать эти значения в параметры шаблона, вы ссылаетесь на это хранилище ключей в [файле параметров](#template-parameter-files) , который используется шаблоном при развертывании.

Ниже приведен пример, в котором указаны имя учетной записи и ключ доступа для подключения к хранилищу BLOB-объектов Azure.

**Файл параметров**

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
   "contentVersion": "1.0.0.0",
   // Template parameter values
   "parameters": {
      "LogicAppName": {
         "value": "Email-Processor-Logic-App"
      },
      "LogicAppLocation": {
         "value": "westeurope"
      },
      "azureblob_1_Connection_Name": {
         "value": "Fabrikam-Azure-Blob-Storage-Connection"
      },
      "azureblob_1_Connection_DisplayName": {
         "value": "Fabrikam-Storage"
      },
      "azureblob_1_accountName": {
         "value": "Fabrikam-Storage-Account"
      },
      "azureblob_1_accessKey": {
         "reference": {
            "keyVault": {
               "id": "/subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group-name>/Microsoft.KeyVault/vaults/fabrikam-key-vault"
            },
            "secretName": "FabrikamStorageKey"
         }
      }
   }
}
```

**Шаблон**

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   // Template parameters
   "parameters": {
      "LogicAppName": {<parameter-definition>},
      "LogicAppLocation": {<parameter-definition>},
      "azureblob_1_Connection_Name": {<parameter-definition>},
      "azureblob_1_Connection_DisplayName": {<parameter-definition>},
      "azureblob_1_accountName": {
         "type": "string",
         "defaultValue": "",
         "metadata": {
            "description": "Name of the storage account the connector should use."
         }
      },
      "azureblob_1_accessKey": {
         "type": "secureobject",
         "metadata": {
            "description": "Specify a valid primary/secondary storage account access key."
         }
      }
   },
   "variables": {},
   "functions": [],
   "resources": [
      {
         "properties": {
            "state": "Disabled",
            "definition": {
               "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
               "actions": {
                  // Azure Blob Storage action
                  "Create_blob": {
                     "type": "ApiConnection",
                     "inputs": {
                        "host": {
                           "connection": {
                              // Workflow definition parameter reference for values to use at runtime
                              "name": "@parameters('$connections')['azureblob']['connectionId']"
                           }
                        },
                     },
                     "method": "post",
                     "body": "@triggerBody()?['Body']",
                     "path": "/datasets/default/files",
                     "queries": {
                        "folderPath": "/emails",
                        "name": "@triggerBody()?['Subject']",
                        "queryParametersSingleEncoded": true
                     },
                     "runAfter": {},
                     "runtimeConfiguration": {
                        "contentTransfer": {
                           "transferMode": "Chunked"
                        }
                     }
                  }
               },
               "parameters": {
                  // Workflow definition parameter for values to use at runtime
                  "$connections": {
                     "defaultValue": {},
                     "type": "Object"
                  }
               },
               "triggers": {<trigger-definition>},
               "contentVersion": "1.0.0.0",
               "outputs": {}
            },
            "parameters": {
               "$connections": {
                  "value": {
                     // Template parameter references for values to use at runtime
                     "azureblob": {
                        "id": "[concat(subscription().id, '/providers/Microsoft.Web/locations/', parameters('LogicAppLocation'), '/managedApis/', 'azureblob')]",
                        "connectionId": "[resourceId('Microsoft.Web/connections', parameters('azureblob_1_Connection_Name'))]",
                        "connectionName": "[parameters('azureblob_1_Connection_Name')]"
                    }
                  }
               }
            },
            "name": "[parameters('LogicAppName')]",
            "type": "Microsoft.Logic/workflows",
            "location": "[parameters('LogicAppLocation')]",
            "tags": {
               "displayName": "LogicApp"
            },
            "apiVersion": "2016-06-01",
            // Template parameter reference for value to use at deployment
            "dependsOn": [
               "[resourceId('Microsoft.Web/connections', parameters('azureblob_1_Connection_Name'))]"
            ]
         }
      },
      // Azure Blob Storage API connection resource definition
      {
         "type": "MICROSOFT.WEB/CONNECTIONS",
         "apiVersion": "2016-06-01",
         "name": "[parameters('azureblob_1_Connection_Name')]",
         "location": "[parameters('LogicAppLocation')]",
         "properties": {
            "api": {
               "id": "[concat(subscription().id, '/providers/Microsoft.Web/locations/', parameters('LogicAppLocation'), '/managedApis/', 'azureblob')]"
            },
            "displayName": "[parameters('azureblob_1_Connection_DisplayName')]",
            // Template parameter reference for values to use at deployment
            "parameterValues": {
               "accountName": "[parameters('azureblob_1_accountName')]",
               "accessKey": "[parameters('azureblob_1_accessKey')]"
            }
         }
      }
   ],
   "outputs": {}
}
```

<a name="authenticate-connections"></a>

### <a name="authenticate-connections"></a>Проверка подлинности подключений

После развертывания приложение логики работает в сквозном виде с допустимыми параметрами. Однако по-прежнему необходимо авторизовать подключения OAuth, чтобы создать допустимые маркеры доступа для [проверки подлинности учетных данных](../active-directory/develop/authentication-vs-authorization.md). Дополнительные сведения см. в разделе [Авторизация подключений OAuth](../logic-apps/logic-apps-deploy-azure-resource-manager-templates.md#authorize-oauth-connections).

Некоторые подключения поддерживают использование [субъекта-службы](../active-directory/develop/app-objects-and-service-principals.md) Azure Active Directory (Azure AD) для авторизации подключений к приложению логики, [зарегистрированному в Azure AD](../active-directory/develop/quickstart-register-app.md). Например, в этом Azure Data Lake определении ресурса соединения показано, как ссылаться на параметры шаблона, которые обрабатывали сведения о субъекте-службе, а также о том, как шаблон объявляет эти параметры:

**Определение ресурса подключения**

```json
{
   <other-template-objects>
   "type": "MICROSOFT.WEB/CONNECTIONS",
   "apiVersion": "2016-06-01",
   "name": "[parameters('azuredatalake_1_Connection_Name')]",
   "location": "[parameters('LogicAppLocation')]",
   "properties": {
      "api": {
         "id": "[concat(subscription().id, '/providers/Microsoft.Web/locations/', 'resourceGroup().location', '/managedApis/', 'azuredatalake')]"
      },
      "displayName": "[parameters('azuredatalake_1_Connection_DisplayName')]",
      "parameterValues": {
         "token:clientId": "[parameters('azuredatalake_1_token:clientId')]",
         "token:clientSecret": "[parameters('azuredatalake_1_token:clientSecret')]",
         "token:TenantId": "[parameters('azuredatalake_1_token:TenantId')]",
         "token:grantType": "[parameters('azuredatalake_1_token:grantType')]"
      }
   }
}
```

| Атрибут | Описание |
|-----------|-------------|
| `token:clientId` | Идентификатор приложения или клиента, связанный с субъектом-службой. |
| `token:clientSecret` | Значение ключа, связанное с субъектом-службой |
| `token:TenantId` | Идентификатор каталога для вашего клиента Azure AD |
| `token:grantType` | Запрошенный тип предоставления, который должен иметь значение `client_credentials` . Дополнительные сведения см. [в разделе Платформа Microsoft Identity и поток учетных данных клиента OAuth 2,0](../active-directory/develop/v2-oauth2-client-creds-grant-flow.md). |
|||

**Определения параметров шаблона**

Объект верхнего уровня шаблона `parameters` объявляет эти параметры для примера соединения:

```json
{
   "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
   "contentVersion": "1.0.0.0",
   "parameters": {
      "azuredatalake_1_Connection_Name": {
        "type": "string",
        "defaultValue": "azuredatalake"
      },
      "azuredatalake_1_Connection_DisplayName": {
        "type": "string",
        "defaultValue": "<connection-name>"
      },
      "azuredatalake_1_token:clientId": {
        "type": "securestring",
        "metadata": {
          "description": "Client (or Application) ID of the Azure Active Directory application."
        }
      },
      "azuredatalake_1_token:clientSecret": {
        "type": "securestring",
        "metadata": {
          "description": "Client secret of the Azure Active Directory application."
        }
      },
      "azuredatalake_1_token:TenantId": {
        "type": "securestring",
        "metadata": {
          "description": "The tenant ID of for the Azure Active Directory application."
        }
      },
      "azuredatalake_1_token:resourceUri": {
        "type": "string",
        "metadata": {
          "description": "The resource you are requesting authorization to use."
        }
      },
      "azuredatalake_1_token:grantType": {
        "type": "string",
        "metadata": {
          "description": "Grant type"
        },
        "defaultValue": "client_credentials",
        "allowedValues": [
          "client_credentials"
        ]
      },
      // Other template parameters
   }
   // Other template objects
}
```

Дополнительные сведения о работе с субъектами-службами см. в следующих разделах:

* [Создание приложения Azure Active Directory и субъекта-службы с доступом к ресурсам с помощью портала](../active-directory/develop/howto-create-service-principal-portal.md)
* [Создание субъекта-службы Azure с помощью Azure PowerShell](/powershell/azure/create-azure-service-principal-azureps)
* [Создание субъекта-службы с сертификатом с помощью Azure PowerShell](../active-directory/develop/howto-authenticate-service-principal-powershell.md)

<a name="parameter-references"></a>

## <a name="references-to-parameters"></a>Ссылки на параметры

Для ссылки на параметры шаблона можно использовать выражения шаблонов с [функциями шаблонов](../azure-resource-manager/templates/template-functions.md), которые оцениваются при развертывании. В выражениях шаблона используются квадратные скобки (**[]**):

`"<attribute-name>": "[parameters('<template-parameter-name>')]"`

Для ссылки на параметры определения рабочего процесса используются [выражения и функции языка определения рабочих процессов](../logic-apps/workflow-definition-language-functions-reference.md), которые оцениваются во время выполнения. Вы можете заметить, что некоторые функции шаблона и функции определения рабочего процесса имеют одинаковые имена. Выражения определения рабочего процесса начинаются с символа "at" ( **@** ):

`"<attribute-name>": "@parameters('<workflow-definition-parameter-name>')"`

Вы можете передать значения параметров шаблона в определение рабочего процесса, чтобы приложение логики использовалось во время выполнения. Однако следует избегать использования параметров шаблона, выражений и синтаксиса в определении рабочего процесса, так как конструктор приложений логики не поддерживает элементы шаблона. Кроме того, синтаксис и выражения шаблона могут усложнить код из-за различий в момент вычисления выражений.

Вместо этого выполните следующие общие действия для объявления и ссылки на параметры определения рабочего процесса для использования во время выполнения, объявления и ссылки на параметры шаблона для использования при развертывании и указания значений для передачи при развертывании с помощью файла параметров. Полные сведения см. в разделе [Определение рабочего процесса и параметры](#workflow-definition-parameters) ранее в этом разделе.

1. Создайте шаблон и объявите параметры шаблона для значений, которые следует принимать и использовать при развертывании.

1. В определении рабочего процесса Объявите параметры для значений, которые необходимо принять и использовать во время выполнения. Затем можно сослаться на эти значения по всему определению рабочего процесса и в нем.

1. В `parameters` объекте, который находится *за пределами* определения рабочего процесса, но по-прежнему *находится в* определении ресурса приложения логики, задайте значения параметров определения рабочего процесса, указав соответствующие параметры шаблона. Таким образом, можно передать значения параметров шаблона в параметры определения рабочего процесса.

1. В файле параметров укажите значения для шаблона, который будет использоваться при развертывании.

<a name="full-example-template"></a>

## <a name="full-example-template"></a>Полный пример шаблона

Ниже приведен пример параметризованного шаблона, который используется в примерах этого раздела:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
   "parameters": {
      "LogicAppName": {
         "type": "string",
         "minLength": 1,
         "maxLength": 80,
         "defaultValue": "MyLogicApp",
         "metadata": {
            "description": "The resource name to use for the logic app"
         }
      },
      "LogicAppLocation": {
         "type": "string",
         "minLength": 1,
         "defaultValue": "[resourceGroup().location]",
         "metadata": {
            "description": "The resource location to use for the logic app"
         }
      },
      "office365_1_Connection_Name": {
         "type": "string",
         "defaultValue": "office365",
         "metadata": {
            "description": "The resource name to use for the Office 365 Outlook connection"
         }
      },
      "office365_1_Connection_DisplayName": {
         "type": "string",
         "defaultValue": "",
         "metadata": {
            "description": "The display name to use for the Office 365 Outlook connection"
         }
      },
      "azureblob_1_Connection_Name": {
         "type": "string",
         "defaultValue": "azureblob",
         "metadata": {
            "description": "The resource name to use for the Azure Blob storage account connection"
         }
      },
      "azureblob_1_Connection_DisplayName": {
         "type": "string",
         "defaultValue": "",
         "metadata": {
            "description": "Name of the storage account the connector should use."
         },

      },
      "azureblob_1_accountName": {
         "type": "string",
         "defaultValue": "",
         "metadata": {
            "description": "Name of the storage account the connector should use."
         }
      },
      "azureblob_1_accessKey": {
         "type": "securestring",
         "metadata": {
            "description": "Specify a valid primary/secondary storage account access key."
         }
      },
      "LogicAppIntegrationAccount": {
         "type":"string",
         "minLength": 1,
         "defaultValue": "/subscriptions/<Azure-subscription-ID>/resourceGroups/fabrikam-integration-account-rg/providers/Microsoft.Logic/integrationAccounts/fabrikam-integration-account",
         "metadata": {
            "description": "The ID to use for the integration account"
         }
      }
   },
   "variables": {},
   "resources": [
      {
         "properties": {
            "state": "Disabled",
            "integrationAccount": {
              "id": "[parameters('LogicAppIntegrationAccount')]"
            },
            "definition": {
               "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
               "actions": {
                  "Create_blob": {
                     "type": "ApiConnection",
                     "inputs": {
                        "host": {
                           "connection": {
                              "name": "@parameters('$connections')['azureblob']['connectionId']"
                           }
                        },
                     },
                     "method": "post",
                     "body": "@triggerBody()?['Body']",
                     "path": "/datasets/default/files",
                     "queries": {
                        "folderPath": "/emails",
                        "name": "@triggerBody()?['Subject']",
                        "queryParametersSingleEncoded": true
                     },
                     "runAfter": {},
                     "runtimeConfiguration": {
                        "contentTransfer": {
                           "transferMode": "Chunked"
                        }
                     }
                  }
               },
               "parameters": {
                  "$connections": {
                     "defaultValue": {},
                     "type": "Object"
                  }
               },
               "triggers": {
                  "When_a_new_email_arrives": {
                     "type": "ApiConnection",
                     "inputs": {
                        "host": {
                           "connection": {
                              "name": "@parameters('$connections')['office365']['connectionId']"
                           }
                        },
                        "method": "get",
                        "path": "/Mail/OnNewEmail",
                        "queries": {
                           "folderPath": "Inbox",
                           "importance": "Any",
                           "fetchOnlyWithAttachment": false,
                           "includeAttachments": false
                        }
                     },
                     "recurrence": {
                        "frequency": "Day",
                        "interval": 1
                     },
                     "splitOn": "@triggerBody()?['value']"
                  }
               },
               "contentVersion": "1.0.0.0",
               "outputs": {}
            },
            "parameters": {
               "$connections": {
                  "value": {
                     "azureblob": {
                        "id": "[concat(subscription().id, '/providers/Microsoft.Web/locations/', parameters('LogicAppLocation'), '/managedApis/', 'azureblob')]",
                        "connectionId": "[resourceId('Microsoft.Web/connections', parameters('azureblob_1_Connection_Name'))]",
                        "connectionName": "[parameters('azureblob_1_Connection_Name')]"
                     },
                     "office365": {
                        "id": "[concat(subscription().id, '/providers/Microsoft.Web/locations/', parameters('LogicAppLocation'), '/managedApis/', 'office365')]",
                        "connectionId": "[resourceId('Microsoft.Web/connections', parameters('office365_1_Connection_Name'))]",
                        "connectionName": "[parameters('office365_1_Connection_Name')]"
                     }
                  }
               }
            },
            "accessControl": {}
         },
         "name": "[parameters('LogicAppName')]",
         "type": "Microsoft.Logic/workflows",
         "location": "[parameters('LogicAppLocation')]",
         "tags": {
            "displayName": "LogicApp"
         },
         "apiVersion": "2016-06-01",
         "dependsOn": [
            "[resourceId('Microsoft.Web/connections', parameters('azureblob_1_Connection_Name'))]",
            "[resourceId('Microsoft.Web/connections', parameters('office365_1_Connection_Name'))]"
         ]
      },
      {
         "type": "MICROSOFT.WEB/CONNECTIONS",
         "apiVersion": "2016-06-01",
         "name": "[parameters('office365_1_Connection_Name')]",
         "location": "[parameters('LogicAppLocation')]",
         "properties": {
            "api": {
                "id": "[concat(subscription().id, '/providers/Microsoft.Web/locations/', parameters('LogicAppLocation'), '/managedApis/', 'office365')]"
            },
            "displayName": "[parameters('office365_1_Connection_DisplayName')]"
         }
      },
      {
         "type": "MICROSOFT.WEB/CONNECTIONS",
         "apiVersion": "2016-06-01",
         "name": "[parameters('azureblob_1_Connection_Name')]",
         "location": "[parameters('LogicAppLocation')]",
         "properties": {
            "api": {
               "id": "[concat(subscription().id, '/providers/Microsoft.Web/locations/', parameters('LogicAppLocation'), '/managedApis/', 'azureblob')]"
            },
            "displayName": "[parameters('azureblob_1_Connection_DisplayName')]",
            "parameterValues": {
               "accountName": "[parameters('azureblob_1_accountName')]",
               "accessKey": "[parameters('azureblob_1_accessKey')]"
            }
         }
      }
   ],
   "outputs": {}
}
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Создание шаблонов приложений логики](../logic-apps/logic-apps-create-azure-resource-manager-templates.md)
