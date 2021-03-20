---
title: CreateUiDefinition.jsна панели "файл" для портала
description: Описывает создание определений пользовательского интерфейса для портал Azure. Используется при определении управляемых приложений Azure.
author: tfitzmac
ms.topic: conceptual
ms.date: 07/14/2020
ms.author: tomfitz
ms.openlocfilehash: 327fa1d7eb73d8e65bb4f81c1dff0fe2bec2913b
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89319579"
---
# <a name="createuidefinitionjson-for-azure-managed-applications-create-experience"></a>Использование файла CreateUiDefinition.json для создания управляемого приложения Azure

В этом документе представлены основные понятия **createUiDefinition.js** файла. Портал Azure использует этот файл для определения пользовательского интерфейса при создании управляемого приложения.

Шаблон выглядит следующим образом:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/0.1.2-preview/CreateUIDefinition.MultiVm.json#",
    "handler": "Microsoft.Azure.CreateUIDef",
    "version": "0.1.2-preview",
    "parameters": {
        "config": {
            "isWizard": false,
            "basics": { }
        },
        "basics": [ ],
        "steps": [ ],
        "outputs": { },
        "resourceTypes": [ ]
    }
}
```

`CreateUiDefinition`Всегда содержит три свойства:

* handler;
* version
* parameters

Обработчик всегда должен иметь значение `Microsoft.Azure.CreateUIDef` , а последняя поддерживаемая версия — `0.1.2-preview` .

Схема свойства parameters зависит от сочетания определенных значений handler и version. Для управляемых приложений поддерживаются свойства,, `config` `basics` `steps` и `outputs` . Используется `config` только в том случае, если необходимо переопределить поведение по умолчанию для `basics` шага. Свойства basics и steps содержат [элементы](create-uidefinition-elements.md), например текстовые поля и раскрывающиеся списки, которые отображаются на портале Azure. Свойство Outputs используется для отображения выходных значений указанных элементов в параметрах шаблона Azure Resource Manager.

Советуем также использовать параметр `$schema`, но это необязательно. При его использовании значение параметра `version` должно совпадать с версией в URI `$schema`.

С помощью редактора JSON можно создать createUiDefinition, а затем протестировать его в [песочнице createUiDefinition](https://portal.azure.com/?feature.customPortal=false&#blade/Microsoft_Azure_CreateUIDef/SandboxBlade) для предварительного просмотра. Дополнительные сведения о песочнице см. в статье [тестирование интерфейса портала для управляемых приложений Azure](test-createuidefinition.md).

## <a name="config"></a>Config

Свойство `config` необязательное. Используйте его для переопределения поведения по умолчанию на этапе основных действий или для настройки интерфейса в качестве пошагового мастера. Если `config` используется, это первое свойство в **createUiDefinition.js** `parameters` разделе файла. В следующем примере показаны доступные свойства.

```json
"config": {
    "isWizard": false,
    "basics": {
        "description": "Customized description with **markdown**, see [more](https://www.microsoft.com).",
        "subscription": {
            "constraints": {
                "validations": [
                    {
                        "isValid": "[expression for checking]",
                        "message": "Please select a valid subscription."
                    },
                    {
                        "permission": "<Resource Provider>/<Action>",
                        "message": "Must have correct permission to complete this step."
                    }
                ]
            },
            "resourceProviders": [
                "<Resource Provider>"
            ]
        },
        "resourceGroup": {
            "constraints": {
                "validations": [
                    {
                        "isValid": "[expression for checking]",
                        "message": "Please select a valid resource group."
                    }
                ]
            },
            "allowExisting": true
        },
        "location": {
            "label": "Custom label for location",
            "toolTip": "provide a useful tooltip",
            "resourceTypes": [
                "Microsoft.Compute/virtualMachines"
            ],
            "allowedValues": [
                "eastus",
                "westus2"
            ],
            "visible": true
        }
    }
},
```

### <a name="wizard"></a>Мастер

`isWizard`Свойство позволяет потребовать успешного выполнения проверки каждого шага перед переходом к следующему шагу. Если `isWizard` свойство не задано, по умолчанию используется **значение false**, а пошаговая проверка не требуется.

Если параметр `isWizard` включен, установите значение **true**, вкладка « **основные** » доступна, а все остальные вкладки будут отключены. Когда кнопка **Далее** выбрана, значок вкладки указывает, пройдена ли проверка вкладки. После заполнения обязательных полей на вкладке и проверки кнопки **Далее** можно переходить на следующую вкладку. Когда все вкладки проходят проверку, можно перейти на страницу **Проверка и создание** и нажать кнопку **создать** , чтобы начать развертывание.

:::image type="content" source="./media/create-uidefinition-overview/tab-wizard.png" alt-text="Мастер вкладок":::

### <a name="override-basics"></a>Основы переопределения

Основная конфигурация позволяет настроить основные этапы.

Для `description` Укажите строку с поддержкой Markdown, которая описывает ресурс. Поддерживается многострочный формат и ссылки.

`subscription`Элементы и `resourceGroup` позволяют указать дополнительные проверки. Синтаксис для указания проверок аналогичен настраиваемой проверке для [текстового поля](microsoft-common-textbox.md). Также можно указать `permission` проверки подписки или группы ресурсов.  

Элемент управления подписками принимает список пространств имен поставщиков ресурсов. Например, можно указать **Microsoft. COMPUTE**. Если пользователь выбирает подписку, которая не поддерживает поставщик ресурсов, отображается сообщение об ошибке. Эта ошибка возникает, когда поставщик ресурсов не зарегистрирован в этой подписке и пользователь не имеет разрешения на регистрацию поставщика ресурсов.  

Элемент управления "Группа ресурсов" имеет параметр для `allowExisting` . Когда `true` пользователи могут выбрать группы ресурсов, у которых уже есть ресурсы. Этот флаг наиболее применим к шаблонам решений, где поведение по умолчанию требует от пользователя выбрать новую или пустую группу ресурсов. В большинстве случаев указание этого свойства не требуется.  

Для `location` укажите свойства элемента управления расположением, который необходимо переопределить. Все свойства, которые не переопределены, устанавливаются в значения по умолчанию. `resourceTypes` принимает массив строк, содержащих полные имена типов ресурсов. Параметры расположения ограничены только регионами, которые поддерживают типы ресурсов.  `allowedValues`   принимает массив строк региона. В раскрывающемся списке отображаются только регионы.Можно задать `allowedValues`   и, и  `resourceTypes` . Результатом является пересечение обоих списков. Наконец, `visible` свойство можно использовать для условного или полного отключения раскрывающегося списка расположение.  

## <a name="basics"></a>Основы

Этап **основы** — это первый шаг, созданный при портал Azure синтаксического анализа файла. По умолчанию на этапе основ пользователи выбирают подписку, группу ресурсов и расположение для развертывания.

:::image type="content" source="./media/create-uidefinition-overview/basics.png" alt-text="Основные сведения по умолчанию":::

В этом разделе можно добавить дополнительные элементы. По возможности добавляйте элементы, которые запрашивают параметры уровня развертывания, такие как имя кластера или учетные данные администратора.

В следующем примере показано текстовое поле, добавленное в элементы по умолчанию.

```json
"basics": [
    {
        "name": "textBox1",
        "type": "Microsoft.Common.TextBox",
        "label": "Textbox on basics",
        "defaultValue": "my text value",
        "toolTip": "",
        "visible": true
    }
]
```

## <a name="steps"></a>Шаги

Свойство шаги содержит ноль или более дополнительных шагов, отображаемых после основ. Каждый шаг содержит один или несколько элементов. Вы можете добавить действия для каждой роли или уровня развертываемого приложения. Например, добавьте шаг для входных данных главного узла и шаг для рабочих узлов в кластере.

```json
"steps": [
    {
        "name": "demoConfig",
        "label": "Configuration settings",
        "elements": [
          ui-elements-needed-to-create-the-instance
        ]
    }
]
```

## <a name="outputs"></a>Выходные данные

Портал Azure использует свойство `outputs` для сопоставления элементов `basics` и `steps` с параметрами шаблона развертывания Azure Resource Manager. Ключами являются имена параметров шаблона, а значениями — свойства выходных объектов из элементов, на которые даются ссылки.

Чтобы задать имя ресурса управляемого приложения, необходимо включить значение `applicationResourceName` в свойство outputs. Если это значение не задано, приложение назначает идентификатор GUID для имени. В пользовательский интерфейс, запрашивающий имя пользователя, можно включить текстовое поле.

```json
"outputs": {
    "vmName": "[steps('appSettings').vmName]",
    "trialOrProduction": "[steps('appSettings').trialOrProd]",
    "userName": "[steps('vmCredentials').adminUsername]",
    "pwd": "[steps('vmCredentials').vmPwd.password]",
    "applicationResourceName": "[steps('appSettings').vmName]"
}
```

## <a name="resource-types"></a>Типы ресурсов

Чтобы отфильтровать доступные расположения только для тех мест, которые поддерживают типы ресурсов для развертывания, укажите массив типов ресурсов. При предоставлении более одного типа ресурсов возвращаются только те расположения, которые поддерживают все типы ресурсов. Это необязательное свойство.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/0.1.2-preview/CreateUIDefinition.MultiVm.json#",
    "handler": "Microsoft.Azure.CreateUIDef",
    "version": "0.1.2-preview",
    "parameters": {
        "resourceTypes": ["Microsoft.Compute/disks"],
        "basics": [
          ...
```  

## <a name="functions"></a>Функции

CreateUiDefinition предоставляет [функции](create-uidefinition-functions.md) для работы с входными и выходными элементами элементов, а также такими функциями, как условия. Эти функции похожи как в синтаксисе, так и в функциональности для Azure Resource Manager функций шаблонов.

## <a name="next-steps"></a>Дальнейшие действия

Файл createUiDefinition.json имеет простую схему. Его реальные возможности основаны на поддерживаемых элементах и функциях. Эти элементы более подробно описаны здесь:

- [Elements (XElement Dynamic Property)](create-uidefinition-elements.md) (Elements (Динамическое свойство XElement))
- [Функции](create-uidefinition-functions.md)

Текущая схема JSON для createUiDefinition доступна здесь: `https://schema.management.azure.com/schemas/0.1.2-preview/CreateUIDefinition.MultiVm.json`.

Пример файла пользовательского интерфейса: [createUiDefinition.json](https://github.com/Azure/azure-managedapp-samples/blob/master/Managed%20Application%20Sample%20Packages/201-managed-app-using-existing-vnet/createUiDefinition.json).
