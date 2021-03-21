---
title: Перенос классического ресурса Azure Monitor Application Insights в ресурс на основе рабочей области | Документация Майкрософт
description: Сведения о действиях, необходимых для обновления классической модели Azure Monitor Application Insights на основе рабочей области.
ms.topic: conceptual
ms.date: 09/23/2020
ms.openlocfilehash: 5791abe33dee2e62aadb00ae1024338e1e44a900
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100584260"
---
# <a name="migrate-to-workspace-based-application-insights-resources"></a>Переход на ресурсы Application Insights на основе рабочих областей

В этом руководстве рассматривается процесс миграции классического Application Insights ресурса в ресурс на основе рабочей области. Ресурсы рабочей области поддерживают полную интеграцию Application Insights и Log Analytics. Ресурсы на основе рабочей области отправляют Application Insights телеметрии в общую рабочую область Log Analytics, которая позволяет получать доступ к [последним функциям Azure Monitor](#new-capabilities) , сохраняя журналы приложений, инфраструктуры и платформы в едином консолидированном расположении.

Ресурсы на основе рабочей области обеспечивают общий контроль доступа на основе ролей в Azure (Azure RBAC) для ресурсов и устраняют необходимость в запросах между приложениями и рабочими областями.

**Ресурсы на основе рабочей области в настоящее время доступны во всех коммерческих регионах и в Azure для государственных организаций США.**

## <a name="new-capabilities"></a>Новые возможности

Application Insights на основе рабочей области позволяет использовать преимущества всех новейших возможностей Azure Monitor и Log Analytics включая:

* [Ключи, управляемые клиентом (CMK)](../logs/customer-managed-keys.md) , обеспечивают шифрование неактивных данных с ключами шифрования, к которым у вас есть доступ.
* [Приватный канал Azure](../logs/private-link-security.md) позволяет безопасно связать службы Azure PaaS с виртуальной сетью с помощью частных конечных точек.
* [Приведите свое хранилище (BYOS) для профайлера и snapshot Debugger](./profiler-bring-your-own-storage.md) предоставляет полный контроль над политикой шифрования неактивных данных, политикой управления жизненным циклом и сетевым доступом для всех связанных с Application Insights Profiler и snapshot Debugger. 
* [Уровни резервирования емкости](../logs/manage-cost-storage.md#pricing-model) позволяют сэкономить до 25% по сравнению с ценой оплаты по мере использования. 
* Ускорение приема данных за счет Log Analytics приема потоковой передачи.

## <a name="migration-process"></a>Процесс миграции

При переходе на ресурс на основе рабочей области данные не передаются из хранилища классического ресурса в новое хранилище на основе рабочей области. При выборе миграции будет изменено расположение, в которое будут записываться новые данные в Log Analytics рабочей области с сохранением доступа к классическим данным ресурсов. 

Данные классических ресурсов будут сохранены и будут подвергаться параметрам хранения на классическом Application Insights ресурсе. Все новые данные, полученные после миграции, будут подвергаться [параметрам сохранения](../logs/manage-cost-storage.md#change-the-data-retention-period) связанной рабочей области log Analytics, которая также поддерживает [различные параметры хранения по типу данных](../logs/manage-cost-storage.md#retention-by-data-type).
Процесс миграции является **постоянным и не может быть отменен**. После миграции ресурса в Application Insights на основе рабочей области он всегда будет ресурсом на основе рабочей области. Однако после миграции вы сможете изменить целевую рабочую область так часто, насколько это необходимо. 

> [!NOTE]
> Прием и хранение данных для Application Insights ресурсов на основе рабочей области [выставляются в log Analytics рабочей области](../logs/manage-cost-storage.md) , где находятся данные. Если вы выбрали срок хранения данных более 90 дней для данных, полученных в классическом Application Insights ресурсе перед миграцией, срок хранения данных будет по-прежнему взиматься с этого Application Insights ресурса. [Узнайте больше]( ./pricing.md#workspace-based-application-insights) о выставлении счетов за ресурсы рабочей области Application Insights.

Если вам не нужно переносить существующий ресурс, а вместо этого нужно создать новый ресурс Application Insights на основе рабочей области, воспользуйтесь [руководством по созданию ресурсов на основе рабочей области](create-workspace-resource.md).

## <a name="pre-requisites"></a>Предварительные требования 

- Рабочая область Log Analytics с режимом управления доступом, установленным в **`use resource or workspace permissions`** параметре. 

    - Ресурсы Application Insights на основе рабочей области несовместимы с рабочими областями, для которых задан выделенный **`workspace based permissions`** параметр. Дополнительные сведения об управлении доступом к рабочей области Log Analytics см. в [руководстве по настройке режима управления доступом log Analytics](../logs/manage-access.md#configure-access-control-mode) .

    - Если у вас еще нет рабочей области Log Analytics, [обратитесь к документации по созданию рабочей области Log Analytics](../logs/quick-create-workspace.md).
    
- Непрерывный экспорт не поддерживается для ресурсов на основе рабочей области и должен быть отключен.
После завершения миграции можно использовать [параметры диагностики](../essentials/diagnostic-settings.md) для настройки архивации данных в учетную запись хранения или потоковой передачи в концентратор событий Azure.  

- Проверьте текущие параметры хранения в разделе **Общие**  >  сведения **об использовании и предполагаемые затраты** на  >  **хранение данных** для рабочей области log Analytics. Этот параметр влияет на время хранения любых новых принимаемых данных после переноса Application Insights ресурса. Если в настоящее время вы храните Application Insights данные дольше 90 дней по умолчанию и хотите сохранить этот больший срок хранения, может потребоваться настроить параметры хранения рабочей области.

## <a name="migrate-your-resource"></a>Перенос ресурса

В этом разделе описывается процесс переноса классического Application Insights ресурса в новый тип ресурсов на основе рабочей области.

1. В Application Insights ресурс выберите **Свойства** в заголовке **Настройка** в левой строке меню.

    ![Свойства, выделенные красным прямоугольником](./media/convert-classic-resource/properties.png)

2. Выберите **`Migrate to Workspace-based`**.
    
     ![Кнопка "перенести ресурс"](./media/convert-classic-resource/migrate.png)

3. Выберите рабочую область Log Analytics, в которой будут храниться все принимаемые в будущем Application Insights телеметрии.

     ![Пользовательский интерфейс мастера миграции с возможностью выбора рабочей области целева](./media/convert-classic-resource/migration.png)
    

После переноса ресурса в области **Обзор** отобразятся соответствующие сведения о рабочей области:

![Имя рабочей области](./media/create-workspace-resource/workspace-name.png)

Щелкнув синий текст ссылки, вы перейдете в соответствующую рабочую область Log Analytics, где сможете воспользоваться преимуществами новой унифицированной среды обработки запросов рабочей области.

## <a name="understanding-log-queries"></a>Основные сведения о запросах журнала

Мы по-прежнему предоставляем полную обратную совместимость с классическими запросами, ресурсами, книгами и оповещениями на основе журналов Application Insights. 

Для создания запросов к [новой структуре или схеме таблицы на основе рабочей области](apm-tables.md)необходимо сначала выполнить переход в рабочую область log Analytics. 

При выполнении запроса непосредственно из пользовательского интерфейса Log Analytics в рабочей области отображаются только данные, полученные после миграции. Чтобы просмотреть классические Application Insights данные и новые данные, принимаемые после миграции в едином интерфейсе запроса, используйте представление "журналы" (Analytics) в рамках перенесенного ресурса Application Insights.

## <a name="programmatic-resource-migration"></a>Программная миграция ресурсов

### <a name="azure-cli"></a>Azure CLI

Чтобы получить доступ к командам Azure CLI для предварительной версии Application Insights, сначала необходимо выполнить следующее.

```azurecli
 az extension add -n application-insights
```

Если не выполнить команду `az extension add`, появится сообщение об ошибке, в котором будет указано следующее: `az : ERROR: az monitor: 'app-insights' is not in the 'az monitor' command group. See 'az monitor --help'.`

Теперь можно выполнить следующие действия, чтобы создать ресурс Application Insights.

```azurecli
az monitor app-insights component update --app
                                         --resource-group
                                         [--ingestion-access {Disabled, Enabled}]
                                         [--kind]
                                         [--query-access {Disabled, Enabled}]
                                         [--retention-time]
                                         [--workspace]
```

#### <a name="example"></a>Пример

```azurecli
az monitor app-insights component update --app your-app-insights-resource-name -g your_resource_group --workspace "/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/test1234/providers/microsoft.operationalinsights/workspaces/test1234555"
```

Полную документацию Azure CLI по этой команде можно найти [здесь](/cli/azure/ext/application-insights/monitor/app-insights/component#ext-application-insights-az-monitor-app-insights-component-update).

### <a name="azure-powershell"></a>Azure PowerShell

В `Update-AzApplicationInsights` настоящее время команда PowerShell не поддерживает перенос классического Application Insights ресурса в рабочую область. Чтобы создать ресурс рабочей области с помощью PowerShell, можно использовать приведенные ниже шаблоны Azure Resource Manager и выполнить развертывание с помощью PowerShell.

### <a name="azure-resource-manager-templates"></a>Шаблоны Azure Resource Manager

#### <a name="template-file"></a>Файл шаблона

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "name": {
            "type": "string"
        },
        "type": {
            "type": "string"
        },
        "regionId": {
            "type": "string"
        },
        "tagsArray": {
            "type": "object"
        },
        "requestSource": {
            "type": "string"
        },
        "workspaceResourceId": {
            "type": "string"
        }
    },
    "resources": [
        {
            "name": "[parameters('name')]",
            "type": "microsoft.insights/components",
            "location": "[parameters('regionId')]",
            "tags": "[parameters('tagsArray')]",
            "apiVersion": "2020-02-02-preview",
            "properties": {
                "ApplicationId": "[parameters('name')]",
                "Application_Type": "[parameters('type')]",
                "Flow_Type": "Redfield",
                "Request_Source": "[parameters('requestSource')]",
                "WorkspaceResourceId": "[parameters('workspaceResourceId')]"
            }
        }
    ]
}
```

#### <a name="parameters-file"></a>Файл параметров

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "type": {
            "value": "web"
        },
        "name": {
            "value": "customresourcename"
        },
        "regionId": {
            "value": "eastus"
        },
        "tagsArray": {
            "value": {}
        },
        "requestSource": {
            "value": "Custom"
        },
        "workspaceResourceId": {
            "value": "/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/my_resource_group/providers/microsoft.operationalinsights/workspaces/myworkspacename"
        }
    }
}

```

## <a name="modifying-the-associated-workspace"></a>Изменение связанной рабочей области

После создания ресурса рабочей области Application Insights можно изменить связанную с ним рабочую область Log Analytics.

В области ресурсов Application Insights выберите **Свойства**  >  **изменить рабочую область**  >  **log Analytics рабочие области**.

## <a name="troubleshooting"></a>Устранение неполадок

### <a name="access-mode"></a>Режим доступа

**Сообщение об ошибке:** *Выбранная рабочая область настроена с режимом доступа на основе рабочей области. Может повлиять на некоторые функции APM. Выберите другую рабочую область или разрешите доступ на основе ресурсов в параметрах рабочей области. Эту ошибку можно переопределить с помощью интерфейса командной строки.* 

Чтобы ресурс Application Insights на основе рабочей области работал правильно, необходимо изменить режим управления доступом целевой Log Analytics рабочей области на параметр **разрешения ресурса или рабочей области** . Этот параметр находится в пользовательском интерфейсе рабочей области log Analytics в разделе **Свойства**  >  **режим управления доступом**. Подробные инструкции см. в [руководстве по настройке режима управления доступом log Analytics](../logs/manage-access.md#configure-access-control-mode). Если в качестве режима управления доступом выбран параметр исключительная **необходимость в рабочей области** , миграция с помощью портала будет заблокирована.

Если не удается изменить режим управления доступом в целях безопасности для текущей целевой рабочей области, рекомендуется создать новую рабочую область Log Analytics, которая будет использоваться для миграции. 

### <a name="continuous-export"></a>Непрерывный экспорт

**Сообщение об ошибке:** *для продолжения необходимо отключить непрерывный экспорт. После миграции используйте параметры диагностики для экспорта.* 

Устаревшие функции непрерывного экспорта не поддерживаются для ресурсов рабочей области. Перед миграцией необходимо отключить непрерывный экспорт.

1. В представлении ресурсов Application Insights под заголовком **Настройка** выберите **непрерывный экспорт**.

    ![Пункт меню непрерывного экспорта](./media/convert-classic-resource/continuous-export.png)

2. Выберите **Отключить**.

    ![Кнопка отключения непрерывного экспорта](./media/convert-classic-resource/disable.png)

- После выбора отключить можно вернуться к пользовательскому интерфейсу миграции. Если на странице Изменение непрерывного экспорта запрашивается, что параметры не будут сохранены, можно нажать кнопку ОК для этого запроса, так как он не относится к отключению и включению непрерывного экспорта.

- После успешного переноса Application Insightsного ресурса в рабочую область можно использовать параметры диагностики, чтобы заменить функциональные возможности, которые используются для обеспечения непрерывного экспорта. Выберите **параметры диагностики**  >  **Добавить параметр диагностики** в ресурсе Application Insights. Можно выбрать все таблицы или подмножество таблиц для архивации в учетную запись хранения или использовать потоковую передачу в концентратор событий Azure. Подробные указания по параметрам диагностики см. в [руководстве по параметрам диагностики Azure Monitor](../essentials/diagnostic-settings.md).

### <a name="retention-settings"></a>Параметры удержания

**Предупреждающее сообщение.** *настроенные параметры хранения Application Insights не будут применяться к данным, отправляемым в рабочую область. Его необходимо будет настроить отдельно.*

Вам не нужно вносить какие-либо изменения перед миграцией, но это сообщение о том, что для текущих параметров хранения Application Insights не задан срок хранения 90 дней по умолчанию. Это предупреждающее сообщение означает, что может потребоваться изменить параметры хранения для рабочей области Log Analytics до миграции и начала приема новых данных. 

Вы можете проверить текущие параметры хранения для log Analytics в разделе **Общие** сведения  >  **об использовании и оценочных затратах** в  >   пользовательском интерфейсе log Analytics. Этот параметр влияет на время хранения любых новых принимаемых данных после переноса Application Insights ресурса.

## <a name="next-steps"></a>Дальнейшие действия

* [Изучение метрик](../essentials/metrics-charts.md)
* [Написание запросов аналитики](../logs/log-query-overview.md)
