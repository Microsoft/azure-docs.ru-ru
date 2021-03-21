---
title: Создание настраиваемых политик безопасности в Центре безопасности Azure | Документация Майкрософт
description: Определения настраиваемых политик Azure, отслеживаемых в Центре безопасности Azure.
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: how-to
ms.date: 02/25/2021
ms.author: memildin
zone_pivot_groups: manage-asc-initiatives
ms.openlocfilehash: a901e71da640f8413e5714ad59073324f582c1b9
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102441063"
---
# <a name="create-custom-security-initiatives-and-policies"></a>Создание пользовательских инициатив и политик безопасности

Для защиты систем и среды Центр безопасности Azure создает рекомендации по безопасности. Эти рекомендации основаны на рекомендациях отрасли, которые включены в общую политику безопасности по умолчанию, предоставляемую всем клиентам. Они также могут основываться на сведениях Центра безопасности об отраслевых и нормативных стандартах.

С помощью этой функции можно добавить свои собственные *пользовательские* инициативы. В этом случае вы будете получать рекомендации, если среда не будет соответствовать созданным вами политикам. Все созданные вами пользовательские инициативы будут отображаться рядом со встроенными инициативами на панели мониторинга соответствия нормативным требованиям, как описано в руководстве по [обеспечению соответствия нормативным требованиям на высоком уровне](security-center-compliance-dashboard.md).

Как описано в [документации по политикам Azure](../governance/policy/concepts/definition-structure.md#definition-location), при указании расположения для пользовательской инициативы это должна быть группа управления или подписка. 

> [!TIP]
> Общие сведения о ключевых понятиях на этой странице см. в разделе [что такое политики безопасности, инициативы и рекомендации?](security-policy-concept.md).

::: zone pivot="azure-portal"

## <a name="to-add-a-custom-initiative-to-your-subscription"></a>Добавление пользовательской инициативы в подписку 

1. На боковой панели Центра безопасности откройте страницу **Политика безопасности**.

1. Выберите подписку или группу управления, в которую вы хотите добавить пользовательскую инициативу.

    [![Выбор подписки, для которой создается пользовательская политика](media/custom-security-policies/custom-policy-selecting-a-subscription.png)](media/custom-security-policies/custom-policy-selecting-a-subscription.png#lightbox)

    > [!NOTE]
    > Для оценки и отображения в Центре безопасности пользовательские стандарты необходимо добавлять на уровне подписки (или выше). 
    >
    > При добавлении пользовательского стандарта он назначает этой области *инициативу*. Поэтому рекомендуется выбирать самую широкую область, необходимую для этого назначения.

1. На странице "Политика безопасности" в разделе "Пользовательские инициативы" щелкните **Добавить пользовательскую инициативу**.

    [![Щелкните "добавить настраиваемую инициативу"](media/custom-security-policies/custom-policy-add-initiative.png)](media/custom-security-policies/custom-policy-add-initiative.png#lightbox)

    Откроется следующая страница.

    ![Создание или добавление политики](media/custom-security-policies/create-or-add-custom-policy.png)

1. На странице добавления пользовательских инициатив просмотрите список уже созданных в организации пользовательских политик. Если вы нашли политику, которую хотите назначить своей подписке, щелкните **Добавить**. Если в списке нет инициативы, удовлетворяющей вашим потребностям, пропустите этот шаг.

1. Чтобы создать новую пользовательскую инициативу, выполните следующие действия.

    1. Щелкните кнопку **Создать новую**.
    1. Введите расположение и имя определения.
    1. Выберите политики для включения и щелкните кнопку **Добавить**.
    1. Введите все необходимые параметры.
    1. Выберите команду **Сохранить**.
    1. На странице "Добавление пользовательских инициатив" нажмите кнопку "Обновить". Новая инициатива отобразится как доступная.
    1. Щелкните кнопку **Добавить** и назначьте ее своей подписке.

    > [!NOTE]
    > Для создания новых инициатив требуются учетные данные владельца подписки. Дополнительные сведения о ролях Azure см. в разделе [Разрешения в Центре безопасности Azure](security-center-permissions.md).

    Новая инициатива вступит в силу, и вы можете увидеть ее влияние следующими двумя способами.

    * На боковой панели Центра безопасности в разделе "Политика и соответствие требованиям" выберите **Соответствие нормативным требованиям**. Откроется панель мониторинга соответствия требованиям для отображения вашей новой пользовательской инициативы наряду со встроенными инициативами.
    
    * Вы начнете получать рекомендации, если среда не будет соответствовать определенным вами политикам.

1. Для просмотра итоговых рекомендаций для вашей политики щелкните **Рекомендации** на боковой панели, чтобы открыть страницу рекомендаций. Рекомендации будут отображаться с меткой "пользовательская" и будут доступны в течение примерно одного часа.

    [![Пользовательские рекомендации](media/custom-security-policies/custom-policy-recommendations.png)](media/custom-security-policies/custom-policy-recommendations-in-context.png#lightbox)

::: zone-end

::: zone pivot="rest-api"

## <a name="configure-a-security-policy-in-azure-policy-using-the-rest-api"></a>Настройка политики безопасности в политике Azure с помощью REST API

Как часть интеграции платформенной функциональности с Политикой Azure Центр безопасности Azure позволяет создавать назначения политик с помощью REST API для Политики Azure. Приведенные ниже инструкции помогут создать назначения политик, а также настроить существующие назначения. 

Важные концепции в Политике Azure: 

- **Определение политики** — это правило. 

- **Инициатива** — это набор определений политик (правил). 

- **Назначение** — это приложение инициативы или политика для определенной области (Группа управления, подписка и т. д.). 

Центр безопасности имеет встроенную инициативу Azure Security, включающую все политики безопасности. Чтобы оценить политики центра безопасности в ресурсах Azure, следует создать назначение в группе управления или подписке, которую необходимо оценить.

Встроенные инициативы имеют все политики Центра безопасности Azure, которые включены по умолчанию. Вы можете отключить определенные политики от встроенной инициативы. Например, чтобы применить все политики центра безопасности, кроме **брандмауэра веб-приложения**, измените значение параметра "воздействие политики" на " **отключено**".

## <a name="api-examples"></a>Примеры с API

В следующих примерах замените указанные ниже переменные:

- **{Scope}** введите имя группы управления или подписки, к которой применяется политика
- **{полициассигнментнаме}** введите имя соответствующего назначения политики.
- **{Name}** введите имя или имя администратора, который утвердил изменение политики

В этом примере показано, как присвоить встроенную инициативу Центра безопасности Azure подписке или группе управления.
 
 ```
    PUT  
    https://management.azure.com/{scope}/providers/Microsoft.Authorization/policyAssignments/{policyAssignmentName}?api-version=2018-05-01 

    Request Body (JSON) 

    { 

      "properties":{ 

    "displayName":"Enable Monitoring in Azure Security Center", 

    "metadata":{ 

    "assignedBy":"{Name}" 

    }, 

    "policyDefinitionId":"/providers/Microsoft.Authorization/policySetDefinitions/1f3afdf9-d0c9-4c3d-847f-89da613e70a8", 

    "parameters":{}, 

    } 

    } 
 ```

В этом примере показано, как присвоить подписке встроенную инициативу Центра безопасности Azure со следующими отключенными политиками: 

- обновления системы (systemUpdatesMonitoringEffect); 

- конфигурации безопасности (systemConfigurationsMonitoringEffect); 

- Endpoint Protection (endpointProtectionMonitoringEffect). 

 ```
    PUT https://management.azure.com/{scope}/providers/Microsoft.Authorization/policyAssignments/{policyAssignmentName}?api-version=2018-05-01 
    
    Request Body (JSON) 
    
    { 
    
      "properties":{ 
    
    "displayName":"Enable Monitoring in Azure Security Center", 
    
    "metadata":{ 
    
    "assignedBy":"{Name}" 
    
    }, 
    
    "policyDefinitionId":"/providers/Microsoft.Authorization/policySetDefinitions/1f3afdf9-d0c9-4c3d-847f-89da613e70a8", 
    
    "parameters":{ 
    
    "systemUpdatesMonitoringEffect":{"value":"Disabled"}, 
    
    "systemConfigurationsMonitoringEffect":{"value":"Disabled"}, 
    
    "endpointProtectionMonitoringEffect":{"value":"Disabled"}, 
    
    }, 
    
     } 
    
    } 
 ```
В этом примере показано, как удалять назначения:
 ```
    DELETE   
    https://management.azure.com/{scope}/providers/Microsoft.Authorization/policyAssignments/{policyAssignmentName}?api-version=2018-05-01 
 ```

::: zone-end


## <a name="enhance-your-custom-recommendations-with-detailed-information"></a>Улучшенные пользовательские рекомендации с подробными сведениями

Встроенные рекомендации, предоставляемые Центром безопасности Azure, включают такие сведения, как уровни серьезности и инструкции по исправлению. Если вы хотите добавить эти сведения в пользовательские рекомендации, чтобы они появлялись на портале Azure или при каждом доступе к рекомендациям, необходимо использовать REST API. 

Можно добавить два типа сведений.

- **RemediationDescription** (описание исправления) — строка
- **Severity** (серьезность) — перечисление [Low, Medium, High] (низкий, средний, высокий)

Метаданные должны быть добавлены в определение политики, которая является частью пользовательской инициативы. Они должны находиться в свойстве securityCenter, как показано ниже.

```json
 "metadata": {
    "securityCenter": {
        "RemediationDescription": "Custom description goes here",
        "Severity": "High"
    },
```

Ниже приведен пример настраиваемой политики со свойством metadata/securityCenter.

  ```json
  {
"properties": {
    "displayName": "Security - ERvNet - AuditRGLock",
    "policyType": "Custom",
    "mode": "All",
    "description": "Audit required resource groups lock",
    "metadata": {
        "securityCenter": {
            "RemediationDescription": "Resource Group locks can be set via Azure Portal -> Resource Group -> Locks",
            "Severity": "High"
        }
    },
    "parameters": {
        "expressRouteLockLevel": {
            "type": "String",
            "metadata": {
                "displayName": "Lock level",
                "description": "Required lock level for ExpressRoute resource groups."
            },
            "allowedValues": [
                "CanNotDelete",
                "ReadOnly"
            ]
        }
    },
    "policyRule": {
        "if": {
            "field": "type",
            "equals": "Microsoft.Resources/subscriptions/resourceGroups"
        },
        "then": {
            "effect": "auditIfNotExists",
            "details": {
                "type": "Microsoft.Authorization/locks",
                "existenceCondition": {
                    "field": "Microsoft.Authorization/locks/level",
                    "equals": "[parameters('expressRouteLockLevel')]"
                }
            }
        }
    }
}
}
  ```

Еще один пример использования свойства securityCenter см. в [этом разделе документации по REST API](/rest/api/securitycenter/assessmentsmetadata/createinsubscription#examples).


## <a name="next-steps"></a>Дальнейшие действия

В этой статье научились создавать пользовательские политики безопасности. 

Другие связанные материалы см. в следующих статьях: 

- [Обзор политик безопасности](tutorial-security-policy.md)
- [Список встроенных политик безопасности](./policy-reference.md)