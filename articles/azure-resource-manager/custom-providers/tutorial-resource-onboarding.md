---
title: Руководство по подключению ресурсов
description: Подключение ресурсов с помощью настраиваемых поставщиков позволяет управлять существующими ресурсами Azure и расширять их.
ms.topic: tutorial
ms.author: jobreen
author: jjbfour
ms.date: 09/17/2019
ms.openlocfilehash: 22d1dcd997a4ddb94aba184c5dace4c00509054d
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "75648611"
---
# <a name="tutorial-resource-onboarding-with-azure-custom-providers"></a>Руководство. Подключение ресурсов с помощью Настраиваемых поставщиков Azure

В этом руководстве вы развернете в Azure пользовательский поставщик ресурсов, расширяющий API Azure Resource Manager с типом ресурса Microsoft.CustomProviders/связи. В этом руководстве показано, как расширить существующие ресурсы, находящиеся за пределами группы ресурсов, в которой находится пользовательский экземпляр поставщика. В этом руководстве поставщик пользовательских ресурсов работает на базе приложения логики Azure, но вы можете использовать любую общедоступную конечную точку API.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим учебником вам необходимо ознакомиться со следующими статьями:

* Возможности [Настраиваемых поставщиков Azure](overview.md).
* Основные сведения о [подключении ресурсов с помощью настраиваемых поставщиков](concepts-resource-onboarding.md).

## <a name="get-started-with-resource-onboarding"></a>Начало работы с подключением ресурсов

В этом учебнике необходимо развернуть два элемента: пользовательский поставщик и связь. Чтобы упростить этот процесс, вы можете дополнительно использовать один шаблон, который развертывает оба элемента.

В шаблоне будут использоваться следующие ресурсы:

* Microsoft.CustomProviders/resourceProviders
* Microsoft.Logic/workflows
* Microsoft.CustomProviders/associations

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
            "type": "string",
            "allowedValues": [
                "australiaeast",
                "eastus",
                "westeurope"
            ],
            "metadata": {
                "description": "Location for the resources."
            }
        },
        "logicAppName": {
            "type": "string",
            "defaultValue": "[uniqueString(resourceGroup().id)]",
            "metadata": {
                "description": "Name of the logic app to be created."
            }
        },
        "customResourceProviderName": {
            "type": "string",
            "defaultValue": "[uniqueString(resourceGroup().id)]",
            "metadata": {
                "description": "Name of the custom provider to be created."
            }
        },
        "customResourceProviderId": {
            "type": "string",
            "defaultValue": "",
            "metadata": {
                "description": "The resource ID of an existing custom provider. Provide this to skip deployment of new logic app and custom provider."
            }
        },
        "associationName": {
            "type": "string",
            "defaultValue": "myAssociationResource",
            "metadata": {
                "description": "Name of the custom resource that is being created."
            }
        }
    },
    "resources": [
        {
            "type": "Microsoft.Resources/deployments",
            "apiVersion": "2017-05-10",
            "condition": "[empty(parameters('customResourceProviderId'))]",
            "name": "customProviderInfrastructureTemplate",
            "properties": {
                "mode": "Incremental",
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                    "contentVersion": "1.0.0.0",
                    "parameters": {
                        "logicAppName": {
                            "type": "string",
                            "defaultValue": "[parameters('logicAppName')]"
                        }
                    },
                    "resources": [
                        {
                            "type": "Microsoft.Logic/workflows",
                            "apiVersion": "2017-07-01",
                            "name": "[parameters('logicAppName')]",
                            "location": "[parameters('location')]",
                            "properties": {
                                "state": "Enabled",
                                "definition": {
                                    "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
                                    "actions": {
                                        "Switch": {
                                            "cases": {
                                                "Case": {
                                                    "actions": {
                                                        "CreateCustomResource": {
                                                            "inputs": {
                                                                "body": {
                                                                    "properties": "@addProperty(triggerBody().Body['properties'], 'myDynamicProperty', 'myDynamicValue')"
                                                                },
                                                                "statusCode": 200
                                                            },
                                                            "kind": "Http",
                                                            "type": "Response"
                                                        }
                                                    },
                                                    "case": "CREATE"
                                                }
                                            },
                                            "default": {
                                                "actions": {
                                                    "DefaultHttpResponse": {
                                                        "inputs": {
                                                            "statusCode": 200
                                                        },
                                                        "kind": "Http",
                                                        "type": "Response"
                                                    }
                                                }
                                            },
                                            "expression": "@triggerBody().operationType",
                                            "type": "Switch"
                                        }
                                    },
                                    "contentVersion": "1.0.0.0",
                                    "outputs": {},
                                    "parameters": {},
                                    "triggers": {
                                        "CustomProviderWebhook": {
                                            "inputs": {
                                                "schema": {}
                                            },
                                            "kind": "Http",
                                            "type": "Request"
                                        }
                                    }
                                }
                            }
                        },
                        {
                            "type": "Microsoft.CustomProviders/resourceProviders",
                            "apiVersion": "2018-09-01-preview",
                            "name": "[parameters('customResourceProviderName')]",
                            "location": "[parameters('location')]",
                            "properties": {
                                "resourceTypes": [
                                    {
                                        "name": "associations",
                                        "mode": "Secure",
                                        "routingType": "Webhook,Cache,Extension",
                                        "endpoint": "[[listCallbackURL(concat(resourceId('Microsoft.Logic/workflows', parameters('logicAppName')), '/triggers/CustomProviderWebhook'), '2017-07-01').value]"
                                    }
                                ]
                            }
                        }
                    ],
                    "outputs": {
                        "customProviderResourceId": {
                            "type": "string",
                            "value": "[resourceId('Microsoft.CustomProviders/resourceProviders', parameters('customResourceProviderName'))]"
                        }
                    }
                }
            }
        },
        {
            "type": "Microsoft.CustomProviders/associations",
            "apiVersion": "2018-09-01-preview",
            "name": "[parameters('associationName')]",
            "location": "global",
            "properties": {
                "targetResourceId": "[if(empty(parameters('customResourceProviderId')), reference('customProviderInfrastructureTemplate').outputs.customProviderResourceId.value, parameters('customResourceProviderId'))]",
                "myCustomInputProperty": "myCustomInputValue",
                "myCustomInputObject": {
                    "Property1": "Value1"
                }
            }
        }
    ],
    "outputs": {
        "associationResource": {
            "type": "object",
            "value": "[reference(parameters('associationName'), '2018-09-01-preview', 'Full')]"
        }
    }
}
```

### <a name="deploy-the-custom-provider-infrastructure"></a>Развертывание инфраструктуры настраиваемого поставщика

Первая часть шаблона развертывает инфраструктуру настраиваемого поставщика. Эта инфраструктура определяет воздействие ресурса ассоциаций. Если вы не знакомы с настраиваемыми поставщиками, см. статью [Custom provider basics](overview.md) (Основные сведения о настраиваемых поставщиках).

Приступим к развертыванию инфраструктуры настраиваемого поставщика. Скопируйте, сохраните и разверните предыдущий шаблон или следуйте указаниям по развертыванию инфраструктуры, используя портал Azure.

1. Перейдите на [портал Azure](https://portal.azure.com).

2. Найдите **шаблоны** в разделе **Все службы** или с помощью основного поля поиска:

   ![Поиск шаблонов](media/tutorial-resource-onboarding/templates.png)

3. В области **Шаблоны** выберите **Добавить**:

   ![Выбор "Добавить"](media/tutorial-resource-onboarding/templatesadd.png)

4. В разделе **Общие** введите **имя** и **описание** для нового шаблона:

   ![Имя и описание шаблона](media/tutorial-resource-onboarding/templatesdescription.png)

5. Создание шаблона Resource Manager, скопировав шаблон JSON из раздела "Начало работы с подключением ресурсов" этой статьи:

   ![Создание шаблона Resource Manager](media/tutorial-resource-onboarding/templatesarmtemplate.png)

6. Чтобы создать шаблон, выберите **Добавить**. Если новый шаблон не отображается, выберите **Обновить**.

7. Выберите только что созданный шаблон, а затем нажмите **Развернуть**:

   ![Выберите шаблон, а затем нажмите "Развернуть"](media/tutorial-resource-onboarding/templateselectspecific.png)

8. Введите параметры для обязательных полей, а затем выберите подписку и группу ресурсов. Поле **идентификатора настраиваемого поставщика ресурсов** можно оставить пустым.

   | Имя параметра | Необходим? | Описание |
   | ------------ | -------- | ----------- |
   | Расположение | Да | Расположение ресурсов в шаблоне. |
   | Название приложения логики | Нет | Имя приложения логики. |
   | Имя настраиваемого поставщика ресурсов | Нет | Имя настраиваемого поставщика ресурсов. |
   | Идентификатор настраиваемого поставщика ресурсов | Нет | Существующий настраиваемый поставщик ресурсов, который поддерживает ресурс связи. Если указать значение здесь, приложение логики и развертывание пользовательского поставщика будут пропущены. |
   | Имя ассоциации | Нет | Имя ресурса связи. |

   Примеры параметров:

   ![Ввод параметров шаблона](media/tutorial-resource-onboarding/templatescustomprovider.png)

9. Перейдите к развертыванию и дождитесь его завершения. Результат будет подобен приведенному на следующем снимке экрана. Новый ресурс связи появится в виде выходных данных:

   ![Развертывание завершено успешно](media/tutorial-resource-onboarding/customproviderdeployment.png)

   Вот группа ресурсов, в которой выбрано **Показывать скрытые типы**:

   ![Развертывание пользовательского поставщика](media/tutorial-resource-onboarding/showhidden.png)

10. Просмотрите вкладку приложения логики **Журнал выполнения**, чтобы увидеть вызовы к созданию связи:

    ![Журнал выполнения приложения логики](media/tutorial-resource-onboarding/logicapprun.png)

## <a name="deploy-additional-associations"></a>Развертывание дополнительных связей

Дополнительные связи можно легко развернуть после настройки инфраструктуры настраиваемого поставщика. Группа ресурсов для дополнительных связей может не совпадать с группой ресурсов, в которой развернута инфраструктура настраиваемого поставщика. Чтобы создать связь, необходимо иметь разрешения Microsoft.CustomProviders/resourceproviders/write для указанного идентификатора настраиваемого поставщика ресурса.

1. Перейдите к ресурсу настраиваемого поставщика **Microsoft.CustomProviders/resourceProviders** в группе ресурсов предыдущего развертывания. Вам понадобится установить флажок **Показывать скрытые типы**.

   ![Переход к ресурсу](media/tutorial-resource-onboarding/showhidden.png)

2. Скопируйте свойство "идентификатор ресурса" настраиваемого поставщика.

3. Найдите **шаблоны** в разделе **Все службы** или с помощью основного поля поиска:

   ![Поиск шаблонов](media/tutorial-resource-onboarding/templates.png)

4. Выберите созданный ранее шаблон и щелкните **Развернуть**:

   ![Выберите созданный ранее шаблон и щелкните "Развернуть"](media/tutorial-resource-onboarding/templateselectspecific.png)

5. Введите параметры для обязательных полей, а затем выберите подписку и другую группу ресурсов. Для параметра **идентификатор настраиваемого поставщика ресурсов** введите идентификатор ресурса, скопированный из ранее развернутого настраиваемого поставщика.

6. Перейдите к развертыванию и дождитесь его завершения. Теперь он должен развертывать только новый ресурс связей:

   ![Новый ресурс связей](media/tutorial-resource-onboarding/createdassociationresource.png)

При необходимости можно вернуться к полю приложения логики **Журнал выполнения** и увидеть, что к приложению логики был сделан другой вызов. Вы можете обновить приложение логики, добавляя дополнительные функциональные возможности для каждой созданной связи.

## <a name="getting-help"></a>Получение справки

Если у вас возникли вопросы о настраиваемых поставщиках Azure, попробуйте задать их в [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-custom-providers). Подобный вопрос, возможно, уже был задан, поэтому перед его публикацией сначала проверьте наличие ответа. Чтобы быстрее получить ответ, добавьте к вопросу тег `azure-custom-providers`!

