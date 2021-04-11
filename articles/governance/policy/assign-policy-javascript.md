---
title: Краткое руководство. Создание назначения политики в JavaScript
description: В рамках этого краткого руководства с помощью JavaScript вы создадите назначение в службе "Политика Azure" для определения ресурсов, которые не соответствуют требованиям.
ms.date: 03/31/2021
ms.topic: quickstart
ms.custom: devx-track-js
ms.openlocfilehash: d42156e68747b66778ea1cf2001d2b4bfc901bb0
ms.sourcegitcommit: 99fc6ced979d780f773d73ec01bf651d18e89b93
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106090192"
---
# <a name="quickstart-create-a-policy-assignment-to-identify-non-compliant-resources-using-javascript"></a>Краткое руководство. Создание назначения политики для выявления ресурсов, не соответствующих требованиям, с использованием JavaScript

Чтобы понять, соответствуют ли ресурсы требованиям в Azure, прежде всего нужно определить их состояние. В рамках этого краткого руководства вы создадите назначение политики для выявления виртуальных машин, которые не используют управляемые диски. После выполнения задач в руководстве вы сможете выявлять _несоответствующие_ виртуальные машины.

Библиотека JavaScript используется для управления ресурсами Azure из командной строки или с помощью сценариев. В этом руководстве объясняется, как создавать назначение политики с помощью библиотеки JavaScript.

## <a name="prerequisites"></a>Предварительные требования

- **Подписка Azure**: Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

- **Node.js**. [Node.js](https://nodejs.org/) 12 или более поздней версии.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="add-the-policy-libraries"></a>Добавление библиотек Политики

Чтобы обеспечить функционирование JavaScript с Политикой Azure, нужно добавить библиотеки. Эти библиотеки подойдут для всех сред, где можно использовать JavaScript, в том числе [Bash в Windows 10](/windows/wsl/install-win10).

1. Настройте новый проект Node.js, выполнив следующую команду.

   ```bash
   npm init -y
   ```

1. Добавьте ссылку на библиотеку yargs.

   ```bash
   npm install yargs
   ```

1. Добавьте ссылку на библиотеки Политики Azure.

   ```bash
   # arm-policy is for working with Azure Policy objects such as definitions and assignments
   npm install @azure/arm-policy

   # arm-policyinsights is for working with Azure Policy compliance data such as events and states
   npm install @azure/arm-policyinsights
   ```

1. Добавьте ссылку на библиотеку аутентификации Azure.

   ```bash
   npm install @azure/ms-rest-nodeauth
   ```

   > [!NOTE]
   > Убедитесь, что в файле _package.json_ указаны `@azure/arm-policy` версии **3.1.0** или более поздней, `@azure/arm-policyinsights` версии **3.2.0** или более поздней, а также `@azure/ms-rest-nodeauth` версии **3.0.5** или более поздней.

## <a name="create-a-policy-assignment"></a>Создание назначения политики

С помощью этого краткого руководства вы создадите назначение политики и назначите определение **Аудит виртуальных машин, которые не используют управляемые диски** (`06a78e20-9358-41c9-923c-fb736d382a4d`). Это определение политики идентифицирует ресурсы, которые не соответствуют заданным в нем условиям.

1. Создайте файл с именем _policyAssignment.js_ и введите следующий код.

   ```javascript
   const argv = require("yargs").argv;
   const authenticator = require("@azure/ms-rest-nodeauth");
   const policyObjects = require("@azure/arm-policy");
   
   if (argv.subID && argv.name && argv.displayName && argv.policyDefID && argv.scope && argv.description) {
   
       const createAssignment = async () => {
           const credentials = await authenticator.interactiveLogin();
           const client = new policyObjects.PolicyClient(credentials, argv.subID);
           const assignments = new policyObjects.PolicyAssignments(client);
   
           const result = await assignments.create(
               argv.scope,
               argv.name,
               {
                   displayName: argv.displayName,
                   policyDefinitionId: argv.policyDefID,
                   description: argv.description
               }
           );
           console.log(result);
       };
   
       createAssignment();
   }
   ```

1. Введите следующую команду в окне терминала.

   ```bash
   node policyAssignment.js `
      --subID "{subscriptionId}" `
      --name "audit-vm-manageddisks" `
      --displayName "Audit VMs without managed disks Assignment" `
      --policyDefID "/providers/Microsoft.Authorization/policyDefinitions/06a78e20-9358-41c9-923c-fb736d382a4d" `
      --description "Shows all virtual machines not using managed disks" `
      --scope "{scope}"
   ```

В указанных выше командах используются следующие сведения:

- **subID** — идентификатор подписки для контекста проверки подлинности. Замените `{subscriptionId}` на свою подписку.
- **name** — уникальное имя для объекта назначения политики. В приведенном выше примере это _audit-vm-manageddisks_.
- **displayName** — отображаемое имя назначения политики. В этом случае используется определение _аудита виртуальных машин без назначения управляемых дисков_.
- **policyDefID** — путь определения политики, на основе которого вы создаете назначение. В нашем случае это идентификатор определения политики _аудита виртуальных машин, которые не используют управляемые диски_.
- **description** — более подробное объяснение того, что делает политика или почему она относится к этой области.
- **scope** — область, которая определяет, к каким ресурсам или группе ресурсов принудительно применяется назначение политики. Она может варьироваться от группы управления до отдельного ресурса. Обязательно замените `{scope}` на один из следующих шаблонов:
  - Группа управления: `/providers/Microsoft.Management/managementGroups/{managementGroup}`
  - Подписка: `/subscriptions/{subscriptionId}`
  - группу ресурсов `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}`;
  - Ресурс: `/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/{resourceProviderNamespace}/[{parentResourcePath}/]`.

Теперь все готово к обнаружению ресурсов, которые не соответствуют требованиям, что позволит оценить состояние соответствия в среде.

## <a name="identify-non-compliant-resources"></a>Выявление несоответствующих ресурсов

Итак, вы создали назначение политики и теперь можете определить ресурсы, не соответствующие требованиям.

1. Создайте файл с именем _policyState.js_ и введите следующий код.

   ```javascript
   const argv = require("yargs").argv;
   const authenticator = require("@azure/ms-rest-nodeauth");
   const policyInsights = require("@azure/arm-policyinsights");
   
   if (argv.subID && argv.name) {
   
       const getStates = async () => {
   
           const credentials = await authenticator.interactiveLogin();
           const client = new policyInsights.PolicyInsightsClient(credentials);
           const policyStates = new policyInsights.PolicyStates(client);
           const result = await policyStates.listQueryResultsForSubscription(
               "latest",
               argv.subID,
               {
                   queryOptions: {
                       filter: "IsCompliant eq false and PolicyAssignmentId eq '" + argv.name + "'",
                       apply: "groupby((ResourceId))"
                   }
               }
           );
           console.log(result);
       };
   
       getStates();
   }
   ```

1. Введите следующую команду в окне терминала.

   ```bash
   node policyState.js --subID "{subscriptionId}" --name "audit-vm-manageddisks"
   ```

Замените `{subscriptionId}` подпиской, в которой вы хотите видеть результаты соответствия для этого назначения политики с именем audit-vm-manageddisks, которое мы создали ранее. Список других областей и способов обобщения данных см. в методах [PolicyStates*](/javascript/api/@azure/arm-policyinsights/).

Результаты должны выглядеть примерно так:

```output
{
    'additional_properties': {
        '@odata.nextLink': None
    },
    'odatacontext': 'https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest',
    'odatacount': 12,
    'value': [{data}]
}
```

Результаты соответствуют тому, что вы видите на вкладке **Соответствие ресурсов** назначения политики в представлении портала Azure.

## <a name="clean-up-resources"></a>Очистка ресурсов

- С помощью портала удалите назначение политики _Аудит виртуальных машин без назначения управляемых дисков_. Определение политики встроено, поэтому его удалять не нужно.

- Если вы хотите удалить установленные библиотеки из приложения, выполните следующую команду.

  ```bash
  npm uninstall @azure/arm-policy @azure/arm-policyinsights @azure/ms-rest-nodeauth yargs
  ```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы назначили определение политики для идентификации ресурсов, не соответствующих требованиям, в среде Azure.

Чтобы узнать больше об определениях политики назначения для проверки новых ресурсов на соответствие требованиям, см. следующее руководство:

> [!div class="nextstepaction"]
> [Создание политик и управление ими](./tutorials/create-and-manage.md)