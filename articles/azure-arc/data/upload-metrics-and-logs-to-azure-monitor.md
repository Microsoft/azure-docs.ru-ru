---
title: Передача данных об использовании, метрик и журналов в Azure Monitor
description: Отправляйте данные инвентаризации ресурсов, сведения об использовании, метрики и журналы в Azure Monitor
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
zone_pivot_groups: client-operating-system-macos-and-linux-windows-powershell
ms.openlocfilehash: a522a650413be056ff64d26e90b6c15cf88d9a7d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101643496"
---
# <a name="upload-usage-data-metrics-and-logs-to-azure-monitor"></a>Передача данных об использовании, метрик и журналов в Azure Monitor

Периодически можно экспортировать сведения об использовании для выставления счетов, метрики мониторинга и журналы, а затем передать их в Azure. При экспорте и передаче любого из этих трех типов данных также будут созданы и обновлены ресурсы контроллера данных, управляемого экземпляра SQL и PostgreSQL в Azure.

> [!NOTE] 
> В течение периода действия предварительной версии не взимается плата за использование служб данных, включенных в службу Arc Azure.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

Перед отправкой данных об использовании, метрик или журналов необходимо выполнить следующие действия.

* Средства установки 
* [Регистрация `Microsoft.AzureArcData` поставщика ресурсов](#register-the-resource-provider) 
* [Создание субъекта-службы](#create-service-principal)

## <a name="install-tools"></a>Средства установки

К необходимым средствам относятся: 
* Azure CLI (AZ) 
* [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)] 

См. раздел [Установка средств](./install-client-tools.md).

## <a name="register-the-resource-provider"></a>Регистрация поставщика ресурсов

Прежде чем отправлять метрики или данные пользователя в Azure, необходимо убедиться, что в подписке Azure `Microsoft.AzureArcData` зарегистрирован поставщик ресурсов.

Чтобы проверить поставщик ресурсов, выполните следующую команду:

```azurecli
az provider show -n Microsoft.AzureArcData -o table
```

Если поставщик ресурсов в настоящее время не зарегистрирован в вашей подписке, его можно зарегистрировать. Чтобы зарегистрировать его, выполните следующую команду.  Выполнение команды может занять одну-две минуты.

```azurecli
az provider register -n Microsoft.AzureArcData --wait
```

## <a name="create-service-principal"></a>Создание субъекта-службы

Субъект-служба используется для отправки данных об использовании и метриках.

Выполните следующие команды, чтобы создать субъект-службу передачи метрик.

> [!NOTE]
> Для создания субъекта-службы требуются [определенные разрешения в Azure](../../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app).

Чтобы создать субъект-службу, обновите следующий пример. Замените `<ServicePrincipalName>` `SubscriptionId` и `resourcegroup` значениями и выполните команду:

```azurecli
az ad sp create-for-rbac --name <ServicePrincipalName> --role Contributor --scopes /subscriptions/{SubscriptionId}/resourceGroups/{resourcegroup}
```

Если вы создали субъект-службу ранее и просто хотите получить текущие учетные данные, выполните следующую команду, чтобы сбросить учетные данные.

```azurecli
az ad sp credential reset --name <ServicePrincipalName>
```

Например, чтобы создать субъект-службу с именем `azure-arc-metrics` , выполните следующую команду:

```azurecli
az ad sp create-for-rbac --name azure-arc-metrics --role Contributor --scopes /subscriptions/a345c178a-845a-6a5g-56a9-ff1b456123z2/resourceGroups/myresourcegroup
```

Выходные данные примера:

```output
"appId": "2e72adbf-de57-4c25-b90d-2f73f126e123",
"displayName": "azure-arc-metrics",
"name": "http://azure-arc-metrics",
"password": "5039d676-23f9-416c-9534-3bd6afc78123",
"tenant": "72f988bf-85f1-41af-91ab-2d7cd01ad1234"
```

Сохраните `appId` значения, `password` и `tenant` в переменной среды для последующего использования. 

::: zone pivot="client-operating-system-windows-command"

```console
SET SPN_CLIENT_ID=<appId>
SET SPN_CLIENT_SECRET=<password>
SET SPN_TENANT_ID=<tenant>
```

::: zone-end

::: zone pivot="client-operating-system-macos-and-linux"

```console
export SPN_CLIENT_ID='<appId>'
export SPN_CLIENT_SECRET='<password>'
export SPN_TENANT_ID='<tenant>'
```

::: zone-end

::: zone pivot="client-operating-system-powershell"

```console
$Env:SPN_CLIENT_ID="<appId>"
$Env:SPN_CLIENT_SECRET="<password>"
$Env:SPN_TENANT_ID="<tenant>"
```

::: zone-end

После создания субъекта-службы назначьте субъекту-службе соответствующую роль. 

## <a name="assign-roles-to-the-service-principal"></a>Назначение ролей субъекту-службе

Выполните следующую команду, чтобы назначить субъекту-службе `Monitoring Metrics Publisher` роль в подписке, в которой находятся ресурсы экземпляра базы данных:

::: zone pivot="client-operating-system-windows-command"

> [!NOTE]
> При запуске из среды Windows необходимо использовать двойные кавычки для имен ролей.

```azurecli
az role assignment create --assignee <appId> --role "Monitoring Metrics Publisher" --scope subscriptions/{SubscriptionID}/resourceGroups/{resourcegroup}

```
::: zone-end

::: zone pivot="client-operating-system-macos-and-linux"

```azurecli
az role assignment create --assignee <appId> --role 'Monitoring Metrics Publisher' --scope subscriptions/{SubscriptionID}/resourceGroups/{resourcegroup}
```

::: zone-end

::: zone pivot="client-operating-system-powershell"

```powershell
az role assignment create --assignee <appId> --role 'Monitoring Metrics Publisher' --scope subscriptions/{SubscriptionID}/resourceGroups/{resourcegroup}
```

::: zone-end

Выходные данные примера:

```output
{
  "canDelegate": null,
  "id": "/subscriptions/<Subscription ID>/providers/Microsoft.Authorization/roleAssignments/f82b7dc6-17bd-4e78-93a1-3fb733b912d",
  "name": "f82b7dc6-17bd-4e78-93a1-3fb733b9d123",
  "principalId": "5901025f-0353-4e33-aeb1-d814dbc5d123",
  "principalType": "ServicePrincipal",
  "roleDefinitionId": "/subscriptions/<Subscription ID>/providers/Microsoft.Authorization/roleDefinitions/3913510d-42f4-4e42-8a64-420c39005123",
  "scope": "/subscriptions/<Subscription ID>",
  "type": "Microsoft.Authorization/roleAssignments"
}
```

С субъектом-службой, назначенным соответствующей роли, можно перейти к передаче метрик или данных пользователя. 

## <a name="upload-logs-metrics-or-user-data"></a>Отправка журналов, метрик или данных пользователя

Конкретные действия по передаче журналов, метрик или данных пользователей зависят от типа данных, которые вы отправляете. 

[Отправка журналов в Azure Monitor](upload-logs.md)

[Отправка метрик в Azure Monitor](upload-metrics.md)

[Отправка данных об использовании в Azure Monitor](upload-usage-data.md)

## <a name="general-guidance-on-exporting-and-uploading-usage-metrics"></a>Общие рекомендации по экспорту и передаче сведений об использовании, метриках

Операции создания, чтения, обновления и удаления (CRUD) в службе данных с включенной службой Arc Azure регистрируются в целях выставления счетов и мониторинга. Существуют фоновые службы, которые отслеживают эти операции CRUD и вычисляют потребление соответствующим образом. Фактическое вычисление использования или потребления происходит по расписанию и выполняется в фоновом режиме. 

Во время действия предварительной версии этот процесс происходит ночью. Общее руководство заключается в передаче сведений об использовании только один раз в день. Если сведения об использовании экспортируются и передаются несколько раз в течение одного 24-часового периода, то в портал Azure, но не на использование ресурсов обновляется только Инвентаризация ресурсов.

Для отправки метрик Azure Monitor принимает только последние 30 минут данных (дополнительные[сведения](../../azure-monitor/essentials/metrics-store-custom-rest-api.md#troubleshooting)). Рекомендации по передаче метрик можно передать сразу после создания файла экспорта, чтобы вы могли просмотреть весь набор данных в портал Azure. Например, если вы экспортировали метрики в 2:00 PM и выполнили команду upload в 2:50 РМ. Поскольку Azure Monitor принимает данные только за последние 30 минут, на портале могут не отображаться данные. 

## <a name="next-steps"></a>Дальнейшие действия

[Дополнительные сведения о субъектах-службах](/powershell/azure/azurerm/create-azure-service-principal-azureps#what-is-a-service-principal)

[Отправьте данные о выставлении счетов в Azure и просмотрите их в портал Azure](view-billing-data-in-azure.md)

[Просмотр ресурса контроллера данных Arc Azure в портал Azure](view-data-controller-in-azure-portal.md)
