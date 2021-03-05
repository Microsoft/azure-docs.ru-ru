---
title: Автоматическое обновление расширений для виртуальных машин и масштабируемых наборов в Azure
description: Узнайте, как включить автоматическое обновление расширений для виртуальных машин и масштабируемых наборов виртуальных машин в Azure.
author: mayanknayar
ms.service: virtual-machines
ms.subservice: automatic-extension-upgrade
ms.workload: infrastructure
ms.topic: how-to
ms.date: 02/12/2020
ms.author: manayar
ms.openlocfilehash: fa4fa1c43ab9d31b879bdec8e724e896bd16e14c
ms.sourcegitcommit: dac05f662ac353c1c7c5294399fca2a99b4f89c8
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102123909"
---
# <a name="preview-automatic-extension-upgrade-for-vms-and-scale-sets-in-azure"></a>Предварительная версия: автоматическое обновление расширений для виртуальных машин и масштабируемых наборов в Azure

Автоматическое обновление расширений доступно в предварительной версии для виртуальных машин Azure и масштабируемых наборов виртуальных машин Azure. Если автоматическое обновление расширений включено на виртуальной машине или в масштабируемом наборе, расширение обновляется автоматически при выпуске издателем расширений новой версии для этого расширения.

 Автоматическое обновление расширений имеет следующие особенности.
- Поддерживается для виртуальных машин Azure и масштабируемых наборов виртуальных машин Azure. Service Fabric масштабируемые наборы виртуальных машин в настоящее время не поддерживаются.
- Обновления применяются в модели развертывания, которая является первой по доступности (подробно описывается ниже).
- Для масштабируемого набора виртуальных машин не более 20% виртуальных машин масштабируемого набора будут обновлены в одном пакете. Минимальный размер пакета — одна виртуальная машина.
- Работает для всех размеров виртуальных машин, а для расширений Windows и Linux.
- Вы можете отказаться от автоматического обновления в любое время.
- Автоматическое обновление расширений можно включить в масштабируемых наборах виртуальных машин любого размера.
- Каждое поддерживаемое расширение регистрируется отдельно, и вы можете выбрать, какие расширения будут автоматически обновляться.
- Поддерживается во всех регионах общедоступного облака.


> [!IMPORTANT]
> Автоматическое обновление расширений в настоящее время находится в общедоступной предварительной версии. Для использования функции общедоступной предварительной версии, описанной ниже, требуется процедура согласия.
> Предварительная версия предоставляется без соглашения об уровне обслуживания. Мы не рекомендуем использовать ее в рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены.
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).


## <a name="how-does-automatic-extension-upgrade-work"></a>Как работает автоматическое обновление расширений?
Процесс обновления расширения заменяет существующую версию расширения на виртуальной машине новой версией того же расширения при публикации издателем расширения. Работоспособность виртуальной машины отслеживается после установки нового расширения. Если виртуальная машина находится в неработоспособном состоянии в течение 5 минут после завершения обновления, то версия расширения будет восстановлена до предыдущей версии.

Неудачное обновление расширения автоматически повторяется. Повторная попытка выполняется каждые несколько дней автоматически без вмешательства пользователя.

### <a name="availability-first-updates"></a>Доступность — сначала обновления
Модель доступности-First для обновлений, управляемых платформой, гарантирует, что конфигурации доступности в Azure будут соблюдаться на нескольких уровнях доступности.

Для группы виртуальных машин, которые подчиняются обновлению, Платформа Azure будет управлять обновлениями:

**Между регионами:**
- Обновление будет перемещаться в Azure глобально, чтобы предотвратить сбои при развертывании на уровне Azure.
- "Фаза" может иметь один или несколько регионов, а обновление перемещается по этапам только в случае успешного обновления соответствующих виртуальных машин на предыдущем этапе.
- Геопарные регионы не будут обновляться параллельно и не могут находиться на одном региональном этапе.
- Успешность обновления измеряется путем отслеживания работоспособности виртуальной машины после обновления. Работоспособность виртуальной машины отбирается с помощью индикаторов работоспособности платформы для виртуальной машины. Для масштабируемых наборов виртуальных машин работоспособность виртуальной машины отбирается с помощью проверки работоспособности приложения или расширения работоспособности приложения, если оно применяется к масштабируемому набору.

**В пределах региона:**
- Виртуальные машины в разных Зоны доступности не обновляются одновременно.
- Отдельные виртуальные машины, не входящие в группу доступности, объединяются в пакетном режиме, чтобы избежать параллельных обновлений для всех виртуальных машин в подписке.  

**В наборе:**
- Все виртуальные машины в общей группе доступности или масштабируемом наборе не обновляются одновременно.  
- Виртуальные машины в общей группе доступности обновляются в границах домена обновления, а виртуальные машины в нескольких доменах обновления не обновляются одновременно.  
- Виртуальные машины в общем масштабируемом наборе виртуальных машин группируются в пакетах и обновляются в границах домена обновления.

### <a name="upgrade-process-for-virtual-machine-scale-sets"></a>Процесс обновления для масштабируемых наборов виртуальных машин
1. Прежде чем начать процесс обновления, Orchestrator обеспечит, что не более 20% виртуальных машин во всем масштабируемом наборе будут неработоспособными (по какой-либо причине).

2. Обновление Orchestrator определяет пакет экземпляров виртуальных машин для обновления. Пакет обновления может иметь не более 20% общего числа виртуальных машин, подчиняются минимальному размеру пакета одной виртуальной машине.

3. Для масштабируемых наборов с настроенными зондами работоспособности приложений или расширением работоспособности приложения обновление ожидает до 5 минут (или определенную конфигурацию проверки работоспособности), чтобы виртуальная машина стала работоспособной перед обновлением следующего пакета. Если виртуальная машина не восстанавливает работоспособность после обновления, по умолчанию Предыдущая версия расширения на виртуальной машине переустанавливается.

4. В ходе обновления Orchestrator также отслеживается процент виртуальных машин, которые стали неработоспособными после обновления. Обновление будет прервано, если более чем 20 % обновленных экземпляров утрачивают работоспособность во время обновления.

Описанный выше процесс продолжается, пока все экземпляры в масштабируемом наборе не будут обновлены.

Перед обновлением каждого пакета модуль масштабирования Orchestrator выполняет проверку работоспособности общего масштабируемого набора. При обновлении пакета могут существовать другие параллельные или незапланированные действия по обслуживанию, которые могут повлиять на работоспособность виртуальных машин масштабируемого набора. В таких случаях, если более 20% экземпляров масштабируемого набора становятся неработоспособными, обновление масштабируемого набора останавливается в конце текущего пакета.

## <a name="supported-extensions"></a>Поддерживаемые расширения
Предварительная версия автоматического обновления расширений поддерживает следующие расширения (а также периодически добавляются дополнительные):
- Dependency Agent — [Windows](./extensions/agent-dependency-windows.md) и [Linux](./extensions/agent-dependency-linux.md)
- [Расширение работоспособности приложений](../virtual-machine-scale-sets/virtual-machine-scale-sets-health-extension.md) — Windows и Linux


## <a name="enabling-preview-access"></a>Включение доступа к предварительному просмотру
Для включения функции предварительной версии требуется одноразовый явный выбор функции **аутоматицекстенсионупградепревиев** на подписку, как описано ниже.

### <a name="rest-api"></a>REST API
В следующем примере показано, как включить предварительную версию для подписки.

```
POST on `/subscriptions/{subscriptionId}/providers/Microsoft.Features/providers/Microsoft.Compute/features/AutomaticExtensionUpgradePreview/register?api-version=2015-12-01`
```

Регистрация компонентов может занять до 15 минут. Чтобы проверить состояние регистрации, сделайте следующее:

```
GET on `/subscriptions/{subscriptionId}/providers/Microsoft.Features/providers/Microsoft.Compute/features/AutomaticExtensionUpgradePreview?api-version=2015-12-01`
```

После регистрации функции для подписки завершите процесс согласия, добавив изменения в поставщик ресурсов вычислений.

```
POST on `/subscriptions/{subscriptionId}/providers/Microsoft.Compute/register?api-version=2020-06-01`
```

### <a name="azure-powershell"></a>Azure PowerShell
Используйте командлет [Register-азпровидерфеатуре](/powershell/module/az.resources/register-azproviderfeature) , чтобы включить предварительную версию для подписки.

```azurepowershell-interactive
Register-AzProviderFeature -FeatureName AutomaticExtensionUpgradePreview -ProviderNamespace Microsoft.Compute
```

Регистрация компонентов может занять до 15 минут. Чтобы проверить состояние регистрации, сделайте следующее:

```azurepowershell-interactive
Get-AzProviderFeature -FeatureName AutomaticExtensionUpgradePreview -ProviderNamespace Microsoft.Compute
```

После регистрации функции для подписки завершите процесс согласия, добавив изменения в поставщик ресурсов вычислений.

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.Compute
```

### <a name="azure-cli"></a>Azure CLI
Чтобы включить предварительную версию подписки, используйте команду [AZ Feature Register](/cli/azure/feature#az-feature-register) .

```azurecli-interactive
az feature register --namespace Microsoft.Compute --name AutomaticExtensionUpgradePreview
```

Регистрация компонентов может занять до 15 минут. Чтобы проверить состояние регистрации, сделайте следующее:

```azurecli-interactive
az feature show --namespace Microsoft.Compute --name AutomaticExtensionUpgradePreview
```

После регистрации функции для подписки завершите процесс согласия, добавив изменения в поставщик ресурсов вычислений.

```azurecli-interactive
az provider register --namespace Microsoft.Compute
```


## <a name="enabling-automatic-extension-upgrade"></a>Включение автоматического обновления расширений
Чтобы включить автоматическое обновление расширений для расширения, необходимо убедиться, что свойство *енаблеаутоматикупграде* имеет значение *true* и добавляется в каждое определение расширения по отдельности.


### <a name="rest-api-for-virtual-machines"></a>REST API для виртуальных машин
Чтобы включить автоматическое обновление расширений (в этом примере это расширение Dependency Agent) на виртуальной машине Azure, используйте следующую команду:

```
PUT on `/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Compute/virtualMachines/<vmName>/extensions/<extensionName>?api-version=2019-12-01`
```

```json
{    
    "name": "extensionName",
    "type": "Microsoft.Compute/virtualMachines/extensions",
    "location": "<location>",
    "properties": {
        "autoUpgradeMinorVersion": true,
        "enableAutomaticUpgrade": true, 
        "publisher": "Microsoft.Azure.Monitoring.DependencyAgent",
        "type": "DependencyAgentWindows",
        "typeHandlerVersion": "9.5"
        }
}
```

### <a name="rest-api-for-virtual-machine-scale-sets"></a>REST API для масштабируемых наборов виртуальных машин
Чтобы добавить расширение в модель масштабируемого набора, используйте следующую команду:

```
PUT on `/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Compute/virtualMachineScaleSets/<vmssName>?api-version=2019-12-01`
```

```json
{
   "location": "<location>",
   "properties": {
        "virtualMachineProfile": {
            "extensionProfile": {
                "extensions": [
                {
                "name": "<extensionName>",
                  "properties": {
                        "autoUpgradeMinorVersion": true,
                        "enableAutomaticUpgrade": true,
                    "publisher": "Microsoft.Azure.Monitoring.DependencyAgent",
                    "type": "DependencyAgentWindows",
                    "typeHandlerVersion": "9.5"
                    }
                }
                ]
            }
        }
    }
}
```

### <a name="azure-powershell-for-virtual-machines"></a>Azure PowerShell для виртуальных машин
Используйте командлет [Set-азвмекстенсион](/powershell/module/az.compute/set-azvmextension) :

```azurepowershell-interactive
Set-AzVMExtension -ExtensionName "Microsoft.Azure.Monitoring.DependencyAgent" `
    -ResourceGroupName "myResourceGroup" `
    -VMName "myVM" `
    -Publisher "Microsoft.Azure.Monitoring.DependencyAgent" `
    -ExtensionType "DependencyAgentWindows" `
    -TypeHandlerVersion 9.5 `
    -Location WestUS `
    -EnableAutomaticUpgrade $true
```


### <a name="azure-powershell-for-virtual-machine-scale-sets"></a>Azure PowerShell для масштабируемых наборов виртуальных машин
Добавьте расширение в модель масштабируемого набора с помощью командлета [Add-азвмссекстенсион](/powershell/module/az.compute/add-azvmssextension) .

```azurepowershell-interactive
Add-AzVmssExtension -VirtualMachineScaleSet $vmss
    -Name "Microsoft.Azure.Monitoring.DependencyAgent" `
    -Publisher "Microsoft.Azure.Monitoring.DependencyAgent" `
    -Type "DependencyAgentWindows" `
    -TypeHandlerVersion 9.5 `
    -EnableAutomaticUpgrade $true
```

Обновите масштабируемый набор с помощью команды [Update-азвмсс](/powershell/module/az.compute/update-azvmss) после добавления расширения.


### <a name="azure-cli-for-virtual-machines"></a>Azure CLI для виртуальных машин
Используйте командлет [AZ VM Extension Set](/cli/azure/vm/extension#az_vm_extension_set) :

```azurecli-interactive
az vm extension set \
    --resource-group myResourceGroup \
    --vm-name myVM \
    --name DependencyAgentLinux \
    --publisher Microsoft.Azure.Monitoring.DependencyAgent \
    --version 9.5 \
    --enable-auto-upgrade true
```

### <a name="azure-cli-for-virtual-machine-scale-sets"></a>Azure CLI для масштабируемых наборов виртуальных машин
Чтобы добавить расширение в модель масштабируемого набора, используйте командлет [AZ vmss Extension Set](/cli/azure/vmss/extension#az_vmss_extension_set) .

```azurecli-interactive
az vmss extension set \
    --resource-group myResourceGroup \
    --vmss-name myVMSS \
    --name DependencyAgentLinux \
    --publisher Microsoft.Azure.Monitoring.DependencyAgent \
    --version 9.5 \
    --enable-auto-upgrade true
```

## <a name="extension-upgrades-with-multiple-extensions"></a>Обновления расширений с несколькими расширениями

Виртуальная машина или масштабируемый набор виртуальных машин может иметь несколько расширений с включенным автоматическим обновлением расширений. Одна и та же виртуальная машина или масштабируемый набор также могут иметь другие расширения без включенного автоматического обновления расширений.  

Если для виртуальной машины доступно несколько обновлений расширений, то обновления могут быть объединены в пакет, но каждое обновление расширения применяется отдельно на виртуальной машине. Сбой одного расширения не влияет на другие расширения, которые могут быть обновлены. Например, если для обновления запланировано два расширения и первое обновление расширения завершается сбоем, второе расширение будет по-прежнему обновляться.

Автоматическое обновление расширений можно также применить, если для виртуальной машины или масштабируемого набора виртуальных машин настроено несколько расширений с [последовательностью расширений](../virtual-machine-scale-sets/virtual-machine-scale-sets-extension-sequencing.md). Виртуализация расширения применяется при первом развертывании виртуальной машины, а все будущие обновления расширения применяются независимо друг от друга.


## <a name="next-steps"></a>Дальнейшие действия
> [!div class="nextstepaction"]
> [Дополнительные сведения о расширении работоспособности приложений](../virtual-machine-scale-sets/virtual-machine-scale-sets-health-extension.md)
