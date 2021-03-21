---
title: Создание, просмотр оповещений журнала действий и управление ими в Azure Monitor
description: Создавайте оповещения журнала действий с помощью портала Azure, шаблона Azure Resource Manager и Azure PowerShell.
ms.topic: conceptual
ms.date: 06/25/2019
ms.openlocfilehash: 26ca755f6675fa19c3b122c3528e05d1e8d76845
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102045536"
---
# <a name="create-view-and-manage-activity-log-alerts-by-using-azure-monitor"></a>Создание, просмотр оповещений журнала действий и управление ими с помощью Azure Monitor  

## <a name="overview"></a>Обзор

Оповещения журнала действий создаются в тех случаях, когда происходит новое событие, записываемое в журнал действий, которое соответствует определенным условиям для оповещения.

Эти оповещения предназначены для ресурсов Azure. Их можно создать с помощью шаблона Azure Resource Manager. Их также можно создать, обновить или удалить на портале Azure. Обычно оповещения журнала действий нужны для того, чтобы получать уведомления при внесении определенных изменений в ресурсы в подписке Azure. Как правило, оповещения действуют для конкретных групп ресурсов или ресурсов. Например, можно получать уведомления при удалении любой виртуальной машины в группе ресурсов **myProductionResourceGroup** или при назначении новых ролей пользователю в подписке.

> [!IMPORTANT]
> Оповещения для уведомлений о работоспособности служб невозможно создавать через интерфейс для создания оповещений журнала действий. Дополнительные сведения о создании и использовании уведомлений о работоспособности служб см. в статье [Получение оповещений журнала действий для уведомлений о работоспособности служб](../../service-health/alerts-activity-log-service-notifications-portal.md).

При создании правил генерации оповещений убедитесь в следующем:

- Подписка в области ничем не отличается от подписки, в которой создается оповещение.
- Условия должны соответствовать уровню, состоянию, вызывающей стороне, группе ресурсов, идентификатору ресурса или категории события для типа ресурса, на основе которых настраивается оповещение.
- Допускается только одно условие "allOf".
- "AnyOf" можно использовать, чтобы разрешить несколько условий для нескольких полей (например, если поля "состояние" или "подсостояние" равны определенному значению). Обратите внимание, что в настоящее время использование "AnyOf" ограничено созданием правила генерации оповещений с помощью развертывания шаблона ARM.
- "Контаинсани" можно использовать для разрешения нескольких значений одного поля (например, если "операция" равна "Delete" или "Modify"). Обратите внимание, что использование "Контаинсани" сейчас ограничено созданием правила генерации оповещений с помощью развертывания шаблона ARM.
- Если используется категория Administrative (Административное), в оповещении нужно указать по крайней мере одно из указанных выше условий. Вы не можете создать оповещение, которое активируется каждый раз при создании события в журналах действий.
- Невозможно создать оповещения для событий в категории "оповещения" журнала действий.

## <a name="azure-portal"></a>Портал Azure

Для создания и изменения правил генерации оповещений журнала действий можно использовать портал Azure. В интерфейс интегрирован журнал действий Azure, чтобы вы могли легко создавать оповещения для конкретных интересующих вас событий.

### <a name="create-with-the-azure-portal"></a>Создание с помощью портала Azure

Выполните указанные ниже действия.

1. На портале Azure выберите **Монитор** > **Оповещения**.
2. Выберите **Новое правило генерации оповещений** в левом верхнем углу окна **Оповещения**.

     ![Новое правило генерации оповещений](media/alerts-activity-log/AlertsPreviewOption.png)

     Откроется окно **создания правила**.

      ![Параметры нового правила генерации оповещений](media/alerts-activity-log/create-new-alert-rule-options.png)

3. В разделе **Определение условия оповещения** введите указанные ниже сведения и нажмите **Готово**.

   - **Целевой объект оповещения.** Чтобы просмотреть и выбрать целевой объект для нового оповещения, используйте **фильтр по подписке** / **фильтр по типу ресурса**. Выберите ресурс или группу ресурсов из отображаемого списка.

     > [!NOTE]
     > 
     > В качестве сигнала журнала действий вы можете выбрать ресурс [Azure Resource Manager](../../azure-resource-manager/management/overview.md), группу ресурсов или целую подписку. 

     **Пример представления целевого объекта оповещения**

     ![Выбор цели](media/alerts-activity-log/select-target.png)

   - В разделе **Критерии целевого объекта** выберите **Добавить критерии**. Отображаются все доступные сигналы для целевого объекта, в том числе из различных категорий **журнала действий**. Имя категории добавляется к имени **Monitor Service** (Служба Monitor).

   - Выберите сигнал из отображенного списка различных операций, возможных для типа **журнал действий**.

     Вы можете выбрать временную шкалу истории журнала и соответствующую логику оповещений для целевого сигнала.

     **Снимок экрана: (Add criteria) (Добавить критерии)**

     ![Добавление условий](media/alerts-activity-log/add-criteria.png)
     
     > [!NOTE]
     > 
     >  Чтобы обеспечить высокое качество и эффективность правил, к правилам с сигналом All Administrative (Все административные) нужно добавить по крайней мере еще одно условие. 
     > В качестве части определения оповещения необходимо выбрать значение в одном из раскрывающихся списков — "Уровень события", "Состояние" или "Инициатор", — чтобы сделать правило более конкретным.

     - **History time** (Диапазон времени). События, доступные для выбранной операции, можно отобразить на графике за последние 6, 12, 24 часа или за последнюю неделю.

     - **Логика оповещений.**

       - **Уровень события.** Уровень серьезности события: _Командный_, _Информационный_, _Предупреждение_, _Ошибка_ или _Критический_.
       - **Состояние**: Состояние события. _Запущено_, _Сбой_ или _Успешно выполнено_.
       - **Кем инициировано событие**. Это условие также называется "вызывающей стороной". Адрес электронной почты или идентификатор Azure Active Directory пользователя, выполнившего операцию.

       Пример диаграммы сигнала с применением логики оповещений:

       ![Выбранные критерии](media/alerts-activity-log/criteria-selected.png)

4. В разделе **Определение сведений об оповещении** укажите следующие сведения:

    - **Имя правила генерации оповещений**: имя нового правила генерации оповещений.
    - **Описание**: описание для нового правила генерации оповещений.
    - **Save alert to resource group** (Сохранить оповещение в группу ресурсов): выберите группу ресурсов, в которой вы хотите сохранить новое правило.

5. В разделе **Группа действий** в раскрывающемся меню укажите группу действий, которую нужно назначить новому правилу генерации оповещений. Или [создайте группу действий](./action-groups.md) и назначьте ее новому правилу. Чтобы создать группу, выберите **+ Создать группу**.

6. Чтобы включить правило после его создания, выберите **Да** для параметра **Включить правило при создании**.
7. Выберите **Создать правило генерации оповещений**.

    Будет создано правило генерации оповещений для журнала действий. В правом верхнем углу окна отобразится сообщение с подтверждением.

    Вы можете включить, отключить, изменить или удалить правило. Узнайте больше об управлении правилами журнала действий.


Простая аналогия для описания условий, при которых могут создаваться правила генерации оповещений для журнала действий, — это просмотр или фильтрация событий с помощью [журнала действий на портале Azure](../essentials/activity-log.md#view-the-activity-log). В Azure Monitor на экране **Журнал действий** вы можете отфильтровать журнал действий или найти необходимые события, а затем создать оповещение с помощью кнопки **Добавить оповещение журнала действий**. Затем выполните шаги с 4 по 7, как показано выше.
    
 ![Добавление оповещения из журнала действий](media/alerts-activity-log/add-activity-log.png)
    

### <a name="view-and-manage-in-the-azure-portal"></a>Просмотр и управление на портале Azure

1. На портале Azure выберите **Монитор** > **Оповещения**. Выберите **Управление правилами генерации оповещений** в левом верхнем углу окна.

    ![На снимке экрана показан журнал действий с выделенным полем поиска.](media/alerts-activity-log/manage-alert-rules.png)

    Откроется список доступных правил.

2. Выполните поиск правила журнала действий, которое необходимо изменить.

    ![Поиск правил генерации оповещений журнала действий](media/alerts-activity-log/searth-activity-log-rule-to-edit.png)

    Вы можете использовать доступные фильтры: _Подписка_, _Группа ресурсов_, _Ресурс_, _Тип сигнала_ или _Состояние_, чтобы найти действующее правило, которое необходимо изменить.

   > [!NOTE]
   > 
   > Изменить можно только **описание**, **целевые критерии** и **группы действий**.

3. Выберите правило и дважды щелкните его, чтобы изменить параметры. Внесите необходимые изменения и выберите **Сохранить**.

   ![Управление правилами генерации оповещений](media/alerts-activity-log/activity-log-rule-edit-page.png)

4. Вы можете включить, отключить или удалить правило. Выберите соответствующий параметр в верхней части окна после выбора правила, как описано на шаге 2.


## <a name="azure-resource-manager-template"></a>Шаблон Azure Resource Manager
Чтобы создать оповещение журнала действий с помощью шаблона Azure Resource Manager, создайте ресурс типа `microsoft.insights/activityLogAlerts`. Затем следует заполнить все связанные свойства. Вот шаблон, создающий правило генерации оповещений журнала действий:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "activityLogAlertName": {
      "type": "string",
      "metadata": {
        "description": "Unique name (within the Resource Group) for the Activity log alert."
      }
    },
    "activityLogAlertEnabled": {
      "type": "bool",
      "defaultValue": true,
      "metadata": {
        "description": "Indicates whether or not the alert is enabled."
      }
    },
    "actionGroupResourceId": {
      "type": "string",
      "metadata": {
        "description": "Resource Id for the Action group."
      }
    }
  },
  "resources": [   
    {
      "type": "Microsoft.Insights/activityLogAlerts",
      "apiVersion": "2017-04-01",
      "name": "[parameters('activityLogAlertName')]",      
      "location": "Global",
      "properties": {
        "enabled": "[parameters('activityLogAlertEnabled')]",
        "scopes": [
            "[subscription().id]"
        ],        
        "condition": {
          "allOf": [
            {
              "field": "category",
              "equals": "Administrative"
            },
            {
              "field": "operationName",
              "equals": "Microsoft.Resources/deployments/write"
            },
            {
              "field": "resourceType",
              "equals": "Microsoft.Resources/deployments"
            }
          ]
        },
        "actions": {
          "actionGroups":
          [
            {
              "actionGroupId": "[parameters('actionGroupResourceId')]"
            }
          ]
        }
      }
    }
  ]
}
```
Приведенный выше пример JSON можно сохранить с именем, например sampleActivityLogAlert.json, чтобы использовать в данном пошаговом руководстве. Его можно развернуть с помощью [Azure Resource Manager на портале Azure](../../azure-resource-manager/templates/deploy-portal.md).

  > [!NOTE]
  > 
  > Обратите внимание, что можно определить наиболее подписку на оповещения журнала действий высшего уровня.
  > Это означает, что вы не можете определить предупреждение для нескольких подписок, поэтому определение должно быть оповещено на подписку.

Следующие поля — это параметры, которые можно использовать в шаблоне Azure Resource Manager для полей условий. Обратите внимание, что "Работоспособность ресурсов", "Помощник" и "Работоспособность служб" имеют дополнительные поля свойств для своих особых полей. 
1. resourceId —  идентификатор затронутого ресурса в событии журнала действий, для которого должно быть создано оповещение.
2. category — категория в событии журнала действий. Пример: Administrative (Административное), ServiceHealth (Работоспособность служб), ResourceHealth (Работоспособность ресурсов), Autoscale (Автомасштабирование), Security (Безопасность), Recommendation (Рекомендация), Policy (Политика).
3. caller — адрес электронной почты или идентификатор Azure Active Directory пользователя, выполнившего операцию события в журнале действий.
4. level — уровень действия в событии журнала действий, для которого должно быть создано оповещение. Пример: Critical (Критическое), Error (Ошибка), Warning (Предупреждение), Informational (Информационное), Verbose (Подробные сведения).
5. operationName — имя операции в событии журнала действий. Пример: Microsoft.Resources/deployments/write
6. resourceGroup — имя группы ресурсов для затронутого ресурса в событии журнала действий.
7. resourceProvider — см. статью [Поставщики и типы ресурсов Azure](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Fdocs.microsoft.com%2Fen-us%2Fazure%2Fazure-resource-manager%2Fmanagement%2Fresource-providers-and-types&data=02%7C01%7CNoga.Lavi%40microsoft.com%7C90b7c2308c0647c0347908d7c9a2918d%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637199572373543634&sdata=4RjpTkO5jsdOgPdt%2F%2FDOlYjIFE2%2B%2BuoHq5%2F7lHpCwQw%3D&reserved=0). Список поставщиков ресурсов для служб Azure см. в статье [Поставщики ресурсов для служб Azure](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Fdocs.microsoft.com%2Fen-us%2Fazure%2Fazure-resource-manager%2Fmanagement%2Fazure-services-resource-providers&data=02%7C01%7CNoga.Lavi%40microsoft.com%7C90b7c2308c0647c0347908d7c9a2918d%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637199572373553639&sdata=0ZgJPK7BYuJsRifBKFytqphMOxMrkfkEwDqgVH1g8lw%3D&reserved=0).
8. status — строка, описывающая состояние операции в событии журнала действий. Пример: Started (Запущено), In Progress (Выполняется), Succeeded (Успешно), Failed (Сбой), Active (Активно), Resolved (Разрешено).
9. subStatus — Обычно код состояния HTTP соответствующего вызова REST, но может также включать другие строки, описывающие подсостояние.   Пример: OK (код состояния HTTP: 200), Created (Создано) (код состояния HTTP: 201), Accepted (Принято) (код состояния HTTP: 202), No Content (Нет содержимого) (код состояния HTTP: 204), Bad Request (Недопустимый запрос) (код состояния HTTP: 400), Not Found (Не найдено) (код состояния HTTP: 404), Conflict (Конфликт) (код состояния HTTP: 409), Internal Server Error (Внутренняя ошибка сервера) (код состояния HTTP: 500), Service Unavailable (Служба недоступна) (код состояния HTTP: 503), Gateway Timeout (Истекло время ожидания шлюза) (код состояния HTTP: 504).
10. resourceType — тип ресурса, на который повлияло событие. Пример: Microsoft.Resources/deployments

Пример:

```json
"condition": {
          "allOf": [
            {
              "field": "category",
              "equals": "Administrative"
            },
            {
              "field": "resourceType",
              "equals": "Microsoft.Resources/deployments"
            }
          ]
        }

```
Дополнительные сведения о полях журнала действий можно найти [здесь](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Fdocs.microsoft.com%2Fen-us%2Fazure%2Fazure-monitor%2Fplatform%2Factivity-log-schema&data=02%7C01%7CNoga.Lavi%40microsoft.com%7C90b7c2308c0647c0347908d7c9a2918d%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637199572373563632&sdata=6QXLswwZgUHFXCuF%2FgOSowLzA8iOALVgvL3GMVhkYJY%3D&reserved=0).



> [!NOTE]
> Чтобы правило генерации оповещений журнала действий стало активным, может потребоваться до пяти минут.

## <a name="rest-api"></a>REST API 
[API оповещений журнала действий Azure Monitor](/rest/api/monitor/activitylogalerts) является REST API. Он полностью совместим с REST API Azure Resource Manager. Поэтому он может использоваться в PowerShell с помощью командлета Resource Manager или Azure CLI.

## <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

### <a name="deploy-the-resource-manager-template-with-powershell"></a>Развертывание шаблона Resource Manager с помощью PowerShell
Чтобы использовать PowerShell для развертывания примера шаблона Resource Manager из предыдущего раздела [Шаблон Azure Resource Manager](#azure-resource-manager-template), выполните следующую команду:

```powershell
New-AzResourceGroupDeployment -ResourceGroupName "myRG" -TemplateFile sampleActivityLogAlert.json -TemplateParameterFile sampleActivityLogAlert.parameters.json
```

В sampleActivityLogAlert.parameters.json указаны значения для параметров, необходимых для создания правила генерации оповещений.

### <a name="use-activity-log-powershell-cmdlets"></a>Использование командлетов PowerShell журнала действий

Для оповещений журнала действий предусмотрены специальные командлеты PowerShell.

- [Set-AzActivityLogAlert](/powershell/module/az.monitor/set-azactivitylogalert) — создает новое оповещение журнала действий или обновляет существующее оповещение журнала действий.
- [Get-AzActivityLogAlert](/powershell/module/az.monitor/get-azactivitylogalert) — получает один или несколько ресурсов оповещений журнала действий.
- [Enable-AzActivityLogAlert](/powershell/module/az.monitor/enable-azactivitylogalert) — включает существующее оповещение журнала действий и задает его теги.
- [Disable-AzActivityLogAlert](/powershell/module/az.monitor/disable-azactivitylogalert) — отключает существующее оповещение журнала действий и задает его теги.
- [Remove-AzActivityLogAlert](/powershell/module/az.monitor/remove-azactivitylogalert) — удаляет оповещение из журнала действий.

## <a name="azure-cli"></a>Azure CLI

Команды Azure CLI, представленные в наборе [az monitor activity-log alert](/cli/azure/monitor/activity-log/alert), доступны для управления правилами генерации оповещений для журнала действий.

Чтобы создать новое правило генерации оповещений для журнала действий, выполните следующие команды в таком порядке:

1. [az monitor activity-log alert create](/cli/azure/monitor/activity-log/alert#az-monitor-activity-log-alert-create). Создание нового ресурса правила генерации оповещений для журнала действий.
1. [az monitor activity-log alert scope](/cli/azure/monitor/activity-log/alert/scope). Добавление области для созданного правила генерации оповещений для журнала действий.
1. [az monitor activity-log alert action-group](/cli/azure/monitor/activity-log/alert/action-group). Добавление группы действий для правила генерации оповещений для журнала действий.

Чтобы получить один ресурс правила генерации оповещений для журнала действий, используйте команду Azure CLI [az monitor activity-log alert show](/cli/azure/monitor/activity-log/alert#az-monitor-activity-log-alert-show
). Чтобы просмотреть все ресурсы правила генерации оповещений для журнала активности в группе ресурсов, используйте команду [az monitor activity-log alert list](/cli/azure/monitor/activity-log/alert#az-monitor-activity-log-alert-list).
Чтобы удалить ресурсы правила генерации оповещений для журнала действий, используйте команду Azure CLI [az monitor activity-log alert delete](/cli/azure/monitor/activity-log/alert#az-monitor-activity-log-alert-delete).

## <a name="next-steps"></a>Дальнейшие действия

- Изучите [схему веб-перехватчиков для журналов действий](./activity-log-alerts-webhook.md).
- Прочитайте [обзор журналов действий](./activity-log-alerts.md).
- Дополнительные сведения о группах действий см. в статье [Создание групп действий и управление ими на портале Azure](./action-groups.md).  
- Дополнительные сведения об уведомлениях о работоспособности службы см. в [этой статье](../../service-health/service-notifications.md).
