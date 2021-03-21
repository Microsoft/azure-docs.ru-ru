---
title: Azure Monitor книг и шаблонов Azure Resource Manager
description: Упрощение сложных отчетов с помощью предварительно созданных и настраиваемых параметризованных Azure Monitor книг, развернутых с помощью шаблонов Azure Resource Manager
services: azure-monitor
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 04/30/2020
ms.openlocfilehash: 77190b85da08d09cf05a02dcc5787f0c24229948
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100624639"
---
# <a name="programmatically-manage-workbooks"></a>Программное управление книгами

Владельцы ресурсов имеют возможность программно создавать книги и управлять ими с помощью шаблонов диспетчер ресурсов.

Это может быть полезно в таких сценариях, как:
* Развертывание отчетов по анализу организационной или доменной аналитики вместе с развертываниями ресурсов. Например, вы можете развертывать связанные с конкретной организацией книги производительности и сбоев для новых приложений или виртуальных машин.
* Развертывание стандартных отчетов или панелей мониторинга с помощью книг для существующих ресурсов.

Книга будет создана в нужной подгруппе или группе ресурсов с содержимым, указанным в шаблонах диспетчер ресурсов.

Существует два типа ресурсов книги, которыми можно управлять программно:
* [Шаблоны книг](#azure-resource-manager-template-for-deploying-a-workbook-template)
* [Экземпляры книги](#azure-resource-manager-template-for-deploying-a-workbook-instance)

## <a name="azure-resource-manager-template-for-deploying-a-workbook-template"></a>Шаблон Azure Resource Manager для развертывания шаблона книги

1. Откройте книгу, которую требуется развернуть программно.
2. Переключите книгу в режим редактирования, щелкнув элемент панели инструментов _изменить_ .
3. Откройте _Расширенный редактор_ с помощью _</>_ кнопки на панели инструментов.
4. Убедитесь, что вы используете вкладку _шаблон коллекции_ .

    ![Вкладка шаблона коллекции](./media/workbooks-automate/gallery-template.png)
1. Скопируйте JSON в шаблон коллекции в буфер обмена.
2. Ниже приведен пример шаблона Azure Resource Manager, который развертывает шаблон книги для Azure Monitor коллекции книг. Вставьте код JSON, скопированный вместо `<PASTE-COPIED-WORKBOOK_TEMPLATE_HERE>` . Справочный шаблон Azure Resource Manager, создающий шаблон книги, можно найти [здесь](https://github.com/microsoft/Application-Insights-Workbooks/blob/master/Documentation/ARM-template-for-creating-workbook-template).

    ```json
    {
        "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "resourceName": {
                "type": "string",
                "defaultValue": "my-workbook-template",
                "metadata": {
                    "description": "The unique name for this workbook template instance"
                }
            }
        },
        "resources": [
            {
                "name": "[parameters('resourceName')]",
                "type": "microsoft.insights/workbooktemplates",
                "location": "[resourceGroup().location]",
                "apiVersion": "2019-10-17-preview",
                "dependsOn": [],
                "properties": {
                    "galleries": [
                        {
                            "name": "A Workbook Template",
                            "category": "Deployed Templates",
                            "order": 100,
                            "type": "workbook",
                            "resourceType": "Azure Monitor"
                        }
                    ],
                    "templateData": <PASTE-COPIED-WORKBOOK_TEMPLATE_HERE>
                }
            }
        ]
    }
    ```
1. В `galleries` объекте заполните `name` ключи и `category` значениями. Дополнительные сведения о [параметрах](#parameters) см. в следующем разделе.
2. Разверните этот шаблон Azure Resource Manager с помощью [портал Azure](../../azure-resource-manager/templates/deploy-portal.md#deploy-resources-from-custom-template), [интерфейса командной строки](../../azure-resource-manager/templates/deploy-cli.md), [PowerShell](../../azure-resource-manager/templates/deploy-powershell.md)и т. д.
3. Откройте портал Azure и перейдите к коллекции книг, выбранной в шаблоне Azure Resource Manager. В примере шаблона перейдите к коллекции книг Azure Monitor:
    1. Откройте портал Azure и перейдите к Azure Monitor
    2. Открыть `Workbooks` из оглавления
    3. Найдите шаблон в коллекции в разделе Категория `Deployed Templates` (он будет одним из фиолетовых элементов).

### <a name="parameters"></a>Параметры

|Параметры                |Пояснение                                                                                             |
|:-------------------------|:-------------------------------------------------------------------------------------------------------|
| `name`                   | Имя ресурса шаблона книги в Azure Resource Manager.                                  |
|`type`                    | Всегда Microsoft. Insights/воркбуктемплатес                                                            |
| `location`               | Расположение Azure, в котором будет создана книга.                                               |
| `apiVersion`             | Предварительная версия 2019-10-17                                                                                     |
| `type`                   | Всегда Microsoft. Insights/воркбуктемплатес                                                            |
| `galleries`              | Набор коллекций для отображения этого шаблона книги в.                                                |
| `gallery.name`           | Понятное имя шаблона книги в коллекции.                                             |
| `gallery.category`       | Группа в коллекции, в которой размещается шаблон.                                                     |
| `gallery.order`          | Число, которое определяет порядок отображения шаблона в категории в коллекции. Более низкий порядок подразумевает более высокий приоритет. |
| `gallery.resourceType`   | Тип ресурса, соответствующий коллекции. Обычно это строка типа ресурса, соответствующая ресурсу (например, Microsoft. operationalinsights/workspaces). |
|`gallery.type`            | Этот ключ, называемый типом книги, является уникальным ключом, который отличает галерею от типа ресурса. Application Insights, например, имеют типы `workbook` и `tsg` соответствуют различным галереям книг. |

### <a name="galleries"></a>Коллекции

| Коллекции                                        | Тип ресурса                                      | Тип книги |
| :--------------------------------------------- |:---------------------------------------------------|:--------------|
| Книги в Azure Monitor                     | `Azure Monitor`                                    | `workbook`    |
| Аналитика VM в Azure Monitor                   | `Azure Monitor`                                    | `vm-insights` |
| Книги в рабочей области log Analytics           | `microsoft.operationalinsights/workspaces`         | `workbook`    |
| Книги в Application Insights              | `microsoft.insights/component`                     | `workbook`    |
| Руководства по устранению неполадок в Application Insights | `microsoft.insights/component`                     | `tsg`         |
| Использование в Application Insights                  | `microsoft.insights/component`                     | `usage`       |
| Книги в службе Kubernetes                | `Microsoft.ContainerService/managedClusters`       | `workbook`    |
| Книги в группах ресурсов                   | `microsoft.resources/subscriptions/resourcegroups` | `workbook`    |
| Книги в Azure Active Directory            | `microsoft.aadiam/tenant`                          | `workbook`    |
| Аналитика ВМ на виртуальных машинах                | `microsoft.compute/virtualmachines`                | `insights`    |
| Аналитика ВМ в масштабируемых наборах виртуальных машин      | `microsoft.compute/virtualmachinescalesets`        | `insights`    |

## <a name="azure-resource-manager-template-for-deploying-a-workbook-instance"></a>Шаблон Azure Resource Manager для развертывания экземпляра книги

1. Откройте книгу, которую требуется развернуть программно.
2. Переключите книгу в режим редактирования, щелкнув элемент панели инструментов _изменить_ .
3. Откройте _Расширенный редактор_ с помощью _</>_ кнопки на панели инструментов.
4. В редакторе переключите _тип шаблона_ в _Диспетчер ресурсов шаблон_.
5. Шаблон диспетчер ресурсов для создания отображается в редакторе. Скопируйте содержимое и используйте "как есть" или объедините его с большим шаблоном, который также развертывает целевой ресурс.

    ![Изображение, показывающее, как получить шаблон диспетчер ресурсов в пользовательском интерфейсе книги](./media/workbooks-automate/programmatic-template.png)

## <a name="sample-azure-resource-manager-template"></a>Пример шаблона Azure Resource Manager
В этом шаблоне показано, как развернуть простую книгу, в которой отображается "Hello World!"
```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "workbookDisplayName":  {             
            "type":"string",
            "defaultValue": "My Workbook",
            "metadata": {
                "description": "The friendly name for the workbook that is used in the Gallery or Saved List. Needs to be unique in the scope of the resource group and source" 
            }
        },
        "workbookType":  {             
            "type":"string",
            "defaultValue": "tsg",
            "metadata": {
                "description": "The gallery that the workbook will been shown under. Supported values include workbook, `tsg`, Azure Monitor, etc." 
            }
        },
        "workbookSourceId":  {             
            "type":"string",
            "defaultValue": "<insert-your-resource-id-here>",
            "metadata": {
                "description": "The id of resource instance to which the workbook will be associated" 
            }
        },
        "workbookId": {
            "type":"string",
            "defaultValue": "[newGuid()]",
            "metadata": {
                "description": "The unique guid for this workbook instance" 
            }
        }
    },    
    "resources": [
        {
            "name": "[parameters('workbookId')]",
            "type": "Microsoft.Insights/workbooks",
            "location": "[resourceGroup().location]",
            "kind": "shared",
            "apiVersion": "2018-06-17-preview",
            "dependsOn": [],
            "properties": {
                "displayName": "[parameters('workbookDisplayName')]",
                "serializedData": "{\"version\":\"Notebook/1.0\",\"items\":[{\"type\":1,\"content\":\"{\\\"json\\\":\\\"Hello World!\\\"}\",\"conditionalVisibility\":null}],\"isLocked\":false}",
                "version": "1.0",
                "sourceId": "[parameters('workbookSourceId')]",
                "category": "[parameters('workbookType')]"
            }
        }
    ],
    "outputs": {
        "workbookId": {
            "type": "string",
            "value": "[resourceId( 'Microsoft.Insights/workbooks', parameters('workbookId'))]"
        }
    }
}
```

### <a name="template-parameters"></a>Параметры шаблона

| Параметр | Объяснение |
| :------------- |:-------------|
| `workbookDisplayName` | Понятное имя для книги, используемой в коллекции или сохраненном списке. Должно быть уникальным в области группы ресурсов и источника |
| `workbookType` | Коллекция, в которой будет отображаться книга. Поддерживаются следующие значения: книга, `tsg` , Azure Monitor и т. д. |
| `workbookSourceId` | Идентификатор экземпляра ресурса, с которым будет связана книга. Новая книга будет отображена, связанная с этим экземпляром ресурса, например, в таблице ресурсов в содержимом _книги_. Если требуется, чтобы книга отображалась в коллекции книг в Azure Monitor, используйте строку _Azure Monitor_ вместо идентификатора ресурса. |
| `workbookId` | Уникальный идентификатор GUID для этого экземпляра книги. Используйте _[newGuid ()]_ для автоматического создания нового идентификатора GUID. |
| `kind` | Используется для указания, является ли созданная книга общей или частной. Используйте значение _Shared_ для общих книг и _пользователя_ для частных. |
| `location` | Расположение Azure, в котором будет создана книга. Используйте _[resourceGroup (). Location]_ , чтобы создать его в том же расположении, что и группа ресурсов. |
| `serializedData` | Содержит содержимое или полезные данные, используемые в книге. Использование шаблона диспетчер ресурсов из пользовательского интерфейса книг для получения значения |

### <a name="workbook-types"></a>Типы книг
Типы книг укажите тип коллекции книг, в которой будет отображаться новый экземпляр книги. Возможны следующие значения.

| Type | Расположение коллекции |
| :------------- |:-------------|
| `workbook` | Значение по умолчанию, используемое в большинстве отчетов, включая коллекцию книг Application Insights, Azure Monitor и т. д.  |
| `tsg` | Коллекция руководств по устранению неполадок в Application Insights |
| `usage` | _Больше_ коллекций в условиях _использования_ в Application Insights |

### <a name="limitations"></a>Ограничения
По техническим причинам этот механизм нельзя использовать для создания экземпляров книг в коллекции _книг_ Application Insights. Мы работаем над устранением этого ограничения. В то же время `tsg` для развертывания Application Insights связанных книг рекомендуется использовать коллекцию руководств по устранению неполадок (воркбуктипе:).

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как используются книги для создания новых [Azure Monitor для работы с хранилищем](../insights/storage-insights-overview.md).
