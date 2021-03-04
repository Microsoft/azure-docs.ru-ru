---
title: Установка агента Azure Monitor
description: Параметры для установки агента Azure Monitor (АМА) на виртуальных машинах Azure и серверах с поддержкой Arc Azure.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 11/17/2020
ms.openlocfilehash: 6bf0e8b90ca50a90f84c18981ab6accae8477b46
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102034321"
---
# <a name="install-the-azure-monitor-agent-preview"></a>Установка агента Azure Monitor (Предварительная версия)
В этой статье представлены различные варианты установки [агента Azure Monitor](azure-monitor-agent-overview.md) на виртуальных машинах Azure и серверах с поддержкой Arc, а также параметры для создания [взаимосвязей с правилами сбора данных](data-collection-rule-azure-monitor-agent.md) , которые определяют, какие данные должен собирать агент.

## <a name="prerequisites"></a>Предварительные требования
Перед установкой агента Azure Monitor требуются следующие необходимые компоненты.

- [Управляемое удостоверение системы](../../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md) должно быть включено на виртуальных машинах Azure. Это не требуется для серверов с поддержкой ARC в Azure. Удостоверение системы будет включено автоматически, если агент установлен в рамках процесса [создания и назначения правила сбора данных с помощью портал Azure](#install-with-azure-portal).
- [Тег службы AzureResourceManager](../../virtual-network/service-tags-overview.md) должен быть включен в виртуальной сети для виртуальной машины.

## <a name="virtual-machine-extension-details"></a>Сведения о расширении виртуальной машины
Агент Azure Monitor реализуется как [расширение виртуальной машины Azure](../../virtual-machines/extensions/overview.md) и содержит сведения, приведенные в следующей таблице. Его можно установить с помощью любого из методов установки расширений виртуальной машины, включая описанные в этой статье.

| Свойство | Windows | Linux |
|:---|:---|:---|
| Publisher | Microsoft. Azure. Monitor  | Microsoft. Azure. Monitor |
| Тип      | азуремониторвиндовсажент | азуремониторлинуксажент  |
| TypeHandlerVersion  | 1,0 | 1.5 |


## <a name="install-with-azure-portal"></a>Установка с помощью портала Azure
Чтобы установить агент Azure Monitor с помощью портал Azure, выполните процедуру, чтобы [создать правило сбора данных](data-collection-rule-azure-monitor-agent.md#create-rule-and-association-in-azure-portal) в портал Azure. Это позволяет связать правило сбора данных с одной или несколькими виртуальными машинами Azure или серверами с поддержкой ARC в Azure. Агент будет установлен на любой из этих виртуальных машин, у которых его еще нет.


## <a name="install-with-resource-manager-template"></a>Установка с помощью шаблона диспетчер ресурсов
Диспетчер ресурсов шаблоны можно использовать для установки агента Azure Monitor на виртуальных машинах Azure и на серверах с поддержкой Arc, а также для создания связи с правилами сбора данных. Перед созданием связи необходимо создать любое правило сбора данных.

Получите примеры шаблонов для установки агента и создания связи из следующего: 

- [Шаблон для установки агента Azure Monitor (Azure и служба "Дуга Azure")](../agents/resource-manager-agent.md#azure-monitor-agent-preview) 
- [Шаблон для создания связи с правилом сбора данных](./resource-manager-data-collection-rules.md)

Установите шаблоны с помощью [любого метода развертывания для Диспетчер ресурсов шаблонов](../../azure-resource-manager/templates/deploy-powershell.md) , таких как следующие команды.

# <a name="powershell"></a>[PowerShell](#tab/ARMAgentPowerShell)
```powershell
New-AzResourceGroupDeployment -ResourceGroupName "<resource-group-name>" -TemplateFile "<template-filename.json>" -TemplateParameterFile "<parameter-filename.json>"
```
# <a name="cli"></a>[CLI](#tab/ARMAgentCLI)
```powershell
New-AzResourceGroupDeployment -ResourceGroupName "<resource-group-name>" -TemplateFile "<template-filename.json>" -TemplateParameterFile "<parameter-filename.json>"
```
---

## <a name="install-with-powershell"></a>Установка с помощью PowerShell
Вы можете установить агент Azure Monitor на виртуальных машинах Azure и на серверах с поддержкой ARC в Azure с помощью команды PowerShell для добавления расширения виртуальной машины. 

### <a name="azure-virtual-machines"></a>Виртуальные машины Azure
Используйте следующие команды PowerShell для установки агента Azure Monitor на виртуальных машинах Azure.
# <a name="windows"></a>[Windows](#tab/PowerShellWindows)
```powershell
Set-AzVMExtension -Name AMAWindows -ExtensionType AzureMonitorWindowsAgent -Publisher Microsoft.Azure.Monitor -ResourceGroupName <resource-group-name> -VMName <virtual-machine-name> -Location <location>
```
# <a name="linux"></a>[Linux](#tab/PowerShellLinux)
```powershell
Set-AzVMExtension -Name AMALinux -ExtensionType AzureMonitorLinuxAgent -Publisher Microsoft.Azure.Monitor -ResourceGroupName <resource-group-name> -VMName <virtual-machine-name> -Location <location>
```
---

### <a name="azure-arc-enabled-servers"></a>Серверы с поддержкой Azure Arc
Используйте следующие команды PowerShell, чтобы установить серверы, на которых разрешено использование Azure ARC для Azure Monitor агента.
# <a name="windows"></a>[Windows](#tab/PowerShellWindowsArc)
```powershell
New-AzConnectedMachineExtension -Name AMAWindows -ExtensionType AzureMonitorWindowsAgent -Publisher Microsoft.Azure.Monitor -ResourceGroupName <resource-group-name> -MachineName <virtual-machine-name> -Location <location>
```
# <a name="linux"></a>[Linux](#tab/PowerShellLinuxArc)
```powershell
New-AzConnectedMachineExtension -Name AMALinux -ExtensionType AzureMonitorLinuxAgent -Publisher Microsoft.Azure.Monitor -ResourceGroupName <resource-group-name> -MachineName <virtual-machine-name> -Location <location>
```
---
## <a name="azure-cli"></a>Azure CLI
Вы можете установить агент Azure Monitor на виртуальных машинах Azure и на серверах с поддержкой ARC в Azure с помощью команды Azure CLI для добавления расширения виртуальной машины. 

### <a name="azure-virtual-machines"></a>Виртуальные машины Azure
Используйте следующие команды интерфейса командной строки для установки агента Azure Monitor на виртуальных машинах Azure.
# <a name="windows"></a>[Windows](#tab/CLIWindows)
```azurecli
az vm extension set --name AzureMonitorWindowsAgent --publisher Microsoft.Azure.Monitor --ids <vm-resource-id>
```
# <a name="linux"></a>[Linux](#tab/CLILinux)
```azurecli
az vm extension set --name AzureMonitorLinuxAgent --publisher Microsoft.Azure.Monitor --ids <vm-resource-id>
```
---
### <a name="azure-arc-enabled-servers"></a>Серверы с поддержкой Azure Arc
Используйте следующие команды интерфейса командной строки, чтобы установить серверы, на которых разрешено использование Azure ARC для Azure Monitor агента.

# <a name="windows"></a>[Windows](#tab/CLIWindowsArc)
```azurecli
az connectedmachine machine-extension create --name AzureMonitorWindowsAgent --publisher Microsoft.Azure.Monitor --ids <vm-resource-id>
```
# <a name="linux"></a>[Linux](#tab/CLILinuxArc)
```azurecli
az connectedmachine machine-extension create --name AzureMonitorLinuxAgent --publisher Microsoft.Azure.Monitor --ids <vm-resource-id>
```
---


## <a name="next-steps"></a>Дальнейшие действия

- [Создайте правило сбора данных](data-collection-rule-azure-monitor-agent.md) для сбора данных от агента и их отправки в Azure Monitor.