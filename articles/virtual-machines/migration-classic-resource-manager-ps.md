---
title: Переход на диспетчер ресурсов с помощью PowerShell
description: В этой статье рассматриваются поддерживаемые платформой перенос ресурсов IaaS, таких как виртуальные машины (ВМ), виртуальные сети и учетные записи хранения, из классической модели в Azure Resource Manager с помощью Azure PowerShellных команд.
author: tanmaygore
manager: vashan
ms.service: virtual-machines
ms.subservice: classic-to-arm-migration
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 02/06/2020
ms.author: tagore
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: c750e34e8081cf5a8b3d41cc8c52584a4353a336
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101695164"
---
# <a name="migrate-iaas-resources-from-classic-to-azure-resource-manager-by-using-powershell"></a>Перенос ресурсов IaaS из классической модели в Azure Resource Manager с помощью PowerShell

> [!IMPORTANT]
> Сегодня около 90% виртуальных машин IaaS используют [Azure Resource Manager](https://azure.microsoft.com/features/resource-manager/). По состоянию на 28 февраля 2020, классические виртуальные машины являются устаревшими и будут полностью сняты с 1 марта 2023. [Узнайте больше]( https://aka.ms/classicvmretirement) об этой нерекомендуемости и [о том, как она влияет на вас](classic-vm-deprecation.md#how-does-this-affect-me).

Ниже последовательно описано, как использовать команды Azure PowerShell для переноса ресурсов IaaS из классической модели развертывания в модель развертывания с помощью Azure Resource Manager.

При необходимости можно также перенести ресурсы с помощью [Azure CLI](migration-classic-resource-manager-cli.md).

* Общие сведения о поддерживаемых сценариях переноса см. в разделе [Поддерживаемый платформой перенос ресурсов IaaS из классической модели в модель Azure Resource Manager](migration-classic-resource-manager-overview.md).
* Подробное руководство и инструкции по переносу см. в разделе [Техническое руководство по поддерживаемому платформой переносу из классической модели в модель Azure Resource Manager](migration-classic-resource-manager-deep-dive.md).
* [Ознакомьтесь с наиболее распространенными ошибками миграции](migration-classic-resource-manager-errors.md).

<br>
Ниже приведена блок-схема для указания порядка, в котором необходимо выполнить шаги в процессе миграции.

![Screenshot that shows the migration steps](media/migration-classic-resource-manager/migration-flow.png)

 

## <a name="step-1-plan-for-migration"></a>Шаг 1. Планирование миграции
Ниже приведены некоторые рекомендации, которые мы рекомендуем использовать при оценке необходимости переноса ресурсов IaaS из классической модели в диспетчер ресурсов.

* Ознакомьтесь с [поддерживаемыми и неподдерживаемыми компонентами и конфигурациями](migration-classic-resource-manager-overview.md). Если у вас есть виртуальные машины, использующие неподдерживаемые конфигурации или функции, дождитесь объявления о конфигурации или функции. Также вы можете удалить такую функцию или вынести ее за пределы конфигурации, чтобы выполнить перенос.
* Если у вас есть текущие автоматизированные сценарии, которые развертывают инфраструктуру и приложения, попробуйте создать аналогичную программу установки для миграции с помощью этих сценариев. Вы можете также настроить примеры среды с помощью портала Azure.

> [!IMPORTANT]
> Шлюзы приложений сейчас не поддерживают миграцию из классической модели в диспетчер ресурсов. Чтобы перенести виртуальную сеть с помощью шлюза приложений, удалите шлюз перед выполнением операции подготовки для перемещения сети. После завершения переноса повторно подключите шлюз в Azure Resource Manager.
>
> Шлюзы Azure ExpressRoute, которые подключаются к каналам ExpressRoute в другой подписке, нельзя перенести автоматически. В таких случаях удалите шлюз ExpressRoute, перенесите виртуальную сеть и повторно создайте шлюз. Дополнительные сведения см. в статье [Перенос каналов ExpressRoute и связанных виртуальных сетей из классической модели в модель развертывания Диспетчер ресурсов](../expressroute/expressroute-migration-classic-resource-manager.md).

## <a name="step-2-install-the-latest-version-of-powershell"></a>Шаг 2. Установка последней версии PowerShell
Есть два основных способа установки Azure PowerShell — с помощью [коллекции PowerShell](https://www.powershellgallery.com/profiles/azure-sdk/) и [установщика веб-платформы (WebPI)](https://aka.ms/webpi-azps). Обновления для установщика веб-платформы выпускаются ежемесячно. Обновления для коллекции PowerShell выпускаются на постоянной основе. В этой статье используется Azure PowerShell 2.1.0.

Инструкции по установке см. в статье [Установка и настройка Azure PowerShell](/powershell/azure/).

## <a name="step-3-ensure-that-youre-an-administrator-for-the-subscription"></a>Шаг 3. Убедитесь, что вы являетесь администратором подписки.
Чтобы выполнить эту миграцию, необходимо добавить учетную запись соадминистратора для подписки в [портал Azure](https://portal.azure.com).

1. Войдите на [портал Azure](https://portal.azure.com).
2. В меню **концентратора** выберите пункт **Подписка**. Если вы не видите этот пункт, щелкните **Все службы**.
3. Найдите соответствующую запись подписки и посмотрите на поле **Моя роль** . Для соадминистратора это значение должно быть _администратором учетной записи_.

Если вы не можете добавить соадминистратора, обратитесь к администратору службы или соадминистратору, чтобы получить возможность добавить подписку.

## <a name="step-4-set-your-subscription-and-sign-up-for-migration"></a>Шаг 4. Настройка подписки и регистрация для миграции
Сначала запустите командную строку PowerShell. Для миграции настройте среду как для классической, так и для диспетчер ресурсов.

Войдите в учетную запись для модели Resource Manager.

```powershell
    Connect-AzAccount
```

Получите доступные подписки с помощью следующей команды.

```powershell
    Get-AzSubscription | Sort Name | Select Name
```

Задайте подписку Azure для текущего сеанса. В этом примере задается имя подписки по умолчанию **My Azure Subscription**. Замените имя подписки в примере своим собственным значением.

```powershell
    Select-AzSubscription –SubscriptionName "My Azure Subscription"
```

> [!NOTE]
> Регистрация выполняется один раз, но ее необходимо выполнить один раз, прежде чем пытаться выполнить миграцию. Если вы не зарегистрируетесь, отобразится такое сообщение об ошибке:
>
> *Неправильный запрос: Подписка не зарегистрирована для миграции.*

Выполните регистрацию в поставщике ресурсов миграции с помощью приведенной ниже команды.

```powershell
    Register-AzResourceProvider -ProviderNamespace Microsoft.ClassicInfrastructureMigrate
```

Подождите пять минут, пока не завершится регистрация. Проверьте состояние утверждения с помощью следующей команды:

```powershell
    Get-AzResourceProvider -ProviderNamespace Microsoft.ClassicInfrastructureMigrate
```

Убедитесь, что RegistrationState имеет значение `Registered` , прежде чем продолжить.

Перед переключением на классическую модель развертывания убедитесь в наличии достаточного Azure Resource Manager виртуальной машины виртуальных ЦП в регионе Azure текущего развертывания или виртуальной сети. Чтобы проверить текущее количество виртуальных ЦП в Azure Resource Manager, используйте приведенную ниже команду PowerShell. Чтобы узнать больше о квотах на виртуальные ЦП, см. соответствующий раздел статьи [Подписка Azure, границы, квоты и ограничения службы](../azure-resource-manager/management/azure-subscription-service-limits.md#managing-limits).

В этом примере проверяется доступность в регионе **Западная часть США**. Замените регион в примере своим собственным значением.

```powershell
    Get-AzVMUsage -Location "West US"
```

Теперь войдите в свою учетную запись для классической модели развертывания.

```powershell
    Add-AzureAccount
```

Получите доступные подписки с помощью следующей команды.

```powershell
    Get-AzureSubscription | Sort SubscriptionName | Select SubscriptionName
```

Задайте подписку Azure для текущего сеанса. В этом примере задается имя подписки по умолчанию **My Azure Subscription**. Замените имя подписки в примере своим собственным значением.

```powershell
    Select-AzureSubscription –SubscriptionName "My Azure Subscription"
```


## <a name="step-5-run-commands-to-migrate-your-iaas-resources"></a>Шаг 5. Выполнение команд для переноса ресурсов IaaS
* [Перенос виртуальных машин в облачную службу (не в виртуальной сети)](#step-51-option-1---migrate-virtual-machines-in-a-cloud-service-not-in-a-virtual-network)
* [Перенос виртуальных машин в виртуальную сеть](#step-51-option-2---migrate-virtual-machines-in-a-virtual-network).
* [Перенос учетной записи хранения](#step-52-migrate-a-storage-account)

> [!NOTE]
> Все операции, описанные здесь, являются идемпотентными. Если вы столкнетесь с какой-либо проблемой, не связанной с неподдерживаемой функцией или ошибкой конфигурации, мы рекомендуем повторить подготовку, прервать или зафиксировать текущую операцию. Платформа попытается повторить это действие.


### <a name="step-51-option-1---migrate-virtual-machines-in-a-cloud-service-not-in-a-virtual-network"></a>Шаг 5,1. вариант 1. Перенос виртуальных машин в облачной службе (не в виртуальной сети)
Получите список облачных служб с помощью следующей команды. Затем выберите облачную службу, которую требуется перенести. Если виртуальные машины в облачной службе размещены в виртуальной сети или им назначены веб-роли или рабочие роли, то команда возвращает сообщение об ошибке.

```powershell
    Get-AzureService | ft Servicename
```

Получите имя развертывания для облачной службы. В этом примере имя службы — **My Service**. Замените имя службы в примере своим собственным значением.

```powershell
    $serviceName = "My Service"
    $deployment = Get-AzureDeployment -ServiceName $serviceName
    $deploymentName = $deployment.DeploymentName
```

Подготовьте к переносу виртуальные машины в облачной службе. Возможно два варианта.

* **Вариант 1. Миграция виртуальных машин в виртуальную сеть, созданную платформой.**

    Сначала проверьте, можно выполнить миграцию облачной службы с помощью следующих команд:

    ```powershell
    $validate = Move-AzureService -Validate -ServiceName $serviceName `
        -DeploymentName $deploymentName -CreateNewVirtualNetwork
    $validate.ValidationMessages
    ```

    Приведенная ниже команда позволяет отобразить все предупреждения и ошибки, которые мешают миграции. Если сообщения о проверке не содержат сообщение об ошибке типа, можно перейти к шагу подготовки.

    ```powershell
    Move-AzureService -Prepare -ServiceName $serviceName `
        -DeploymentName $deploymentName -CreateNewVirtualNetwork
    ```
* **Вариант 2. Миграция в существующую виртуальную сеть в модели развертывания диспетчер ресурсов.**

    В этом примере задается имя группы ресурсов **myResourceGroup**, имя виртуальной сети — **myVirtualNetwork**, а имя подсети — **mySubNet**. Замените имена в примере именами своих ресурсов.

    ```powershell
    $existingVnetRGName = "myResourceGroup"
    $vnetName = "myVirtualNetwork"
    $subnetName = "mySubNet"
    ```

    Сначала проверьте, можно выполнить миграцию виртуальной сети с помощью следующей команды:

    ```powershell
    $validate = Move-AzureService -Validate -ServiceName $serviceName `
        -DeploymentName $deploymentName -UseExistingVirtualNetwork -VirtualNetworkResourceGroupName $existingVnetRGName -VirtualNetworkName $vnetName -SubnetName $subnetName
    $validate.ValidationMessages
    ```

    Приведенная ниже команда позволяет отобразить все предупреждения и ошибки, которые мешают миграции. Если сообщения проверки не содержат ошибок, можно перейти к следующему шагу подготовки:

    ```powershell
        Move-AzureService -Prepare -ServiceName $serviceName -DeploymentName $deploymentName `
        -UseExistingVirtualNetwork -VirtualNetworkResourceGroupName $existingVnetRGName `
        -VirtualNetworkName $vnetName -SubnetName $subnetName
    ```

После успешного завершения операции подготовки для одного из перечисленных выше вариантов запросите состояние миграции виртуальных машин. Убедитесь, что они находятся в `Prepared` состоянии.

В этом примере виртуальной машине присваивается имя **myVM**. Замените имя в примере именем своей виртуальной машины.

```powershell
    $vmName = "myVM"
    $vm = Get-AzureVM -ServiceName $serviceName -Name $vmName
    $vm.VM.MigrationState
```

Проверьте конфигурацию для подготовленных ресурсов с помощью PowerShell или портала Azure. Если вы еще не готовы выполнить миграцию и хотите вернуться к старому состоянию, используйте следующую команду:

```powershell
    Move-AzureService -Abort -ServiceName $serviceName -DeploymentName $deploymentName
```

Если подготовленная конфигурация вас устраивает, можете продолжить процесс. Выполните следующую команду, чтобы зафиксировать ресурсы.

```powershell
    Move-AzureService -Commit -ServiceName $serviceName -DeploymentName $deploymentName
```

### <a name="step-51-option-2---migrate-virtual-machines-in-a-virtual-network"></a>Шаг 5,1. вариант 2. Миграция виртуальных машин в виртуальной сети

Для миграции виртуальных машин в виртуальной сети переносится сама виртуальная сеть. Виртуальные машины автоматически переносятся вместе с ней. Выберите виртуальную сеть, в которую будете переносить ресурсы.
> [!NOTE]
> [Перенос одной виртуальной машины](./windows/create-vm-specialized-portal.md) , созданной с помощью классической модели развертывания, путем создания новой диспетчер ресурсов виртуальной машины с управляемыми дисками с помощью файлов виртуального жесткого диска (ОС и данных) виртуальной машины.
<br>

> [!NOTE]
> Имя виртуальной сети может отличаться от имени, отображаемого на новом портале. Новое портал Azure отображает имя как `[vnet-name]` , но фактическое имя виртуальной сети имеет тип `Group [resource-group-name] [vnet-name]` . Прежде чем начать миграцию, найдите фактическое имя виртуальной сети с помощью команды `Get-AzureVnetSite | Select -Property Name` или просмотрите его в старом портал Azure. 

В этом примере виртуальной сети присваивается имя **myVnet**. Замените имя виртуальной сети в примере своим собственным значением.

```powershell
    $vnetName = "myVnet"
```

> [!NOTE]
> Если в виртуальной сети есть виртуальные машины, веб-роли или рабочие роли с неподдерживаемыми конфигурациями, то отображается сообщение об ошибке проверки.

Сначала проверьте, можно выполнить миграцию виртуальной сети с помощью следующей команды:

```powershell
    Move-AzureVirtualNetwork -Validate -VirtualNetworkName $vnetName
```

Приведенная ниже команда позволяет отобразить все предупреждения и ошибки, которые мешают миграции. Если проверка прошла успешно, можно перейти к следующему шагу подготовки:

```powershell
    Move-AzureVirtualNetwork -Prepare -VirtualNetworkName $vnetName
```

Проверьте конфигурацию для подготовленных виртуальных машин с помощью Azure PowerShell или портала Azure. Если вы еще не готовы выполнить миграцию и хотите вернуться к старому состоянию, используйте следующую команду:

```powershell
    Move-AzureVirtualNetwork -Abort -VirtualNetworkName $vnetName
```

Если подготовленная конфигурация вас устраивает, можете продолжить процесс. Выполните следующую команду, чтобы зафиксировать ресурсы.

```powershell
    Move-AzureVirtualNetwork -Commit -VirtualNetworkName $vnetName
```

### <a name="step-52-migrate-a-storage-account"></a>Шаг 5,2. Миграция учетной записи хранения
После завершения миграции виртуальных машин выполните следующие проверки готовности перед переносом учетных записей хранения.

> [!NOTE]
> Если у вашей учетной записи хранения нет связанных дисков или данных виртуальной машины, можно сразу перейти к разделу "проверка учетных записей хранения и запуск миграции". Также обратите внимание, что удаление классических дисков, образов виртуальных машин или образов ОС не приводит к удалению исходных VHD-файлов в учетной записи хранения. Однако он нарушает аренду этих VHD-файлов, чтобы их можно было повторно использовать для создания дисков ARM или образов после миграции.

* Проверки готовности при переносе виртуальных машин или учетной записи хранения имеют дисковые ресурсы:
    * Выполните миграцию виртуальных машин, диски которых хранятся в учетной записи хранения.

        Следующая команда возвращает свойства RoleName и DiskName всех дисков виртуальной машины в учетной записи хранения. RoleName — это имя виртуальной машины, к которой подключен диск. Если эта команда возвращает диски, то перед переносом учетной записи хранения убедитесь, что виртуальные машины, к которым подключены эти диски, перенесены.
        ```powershell
         $storageAccountName = 'yourStorageAccountName'
          Get-AzureDisk | where-Object {$_.MediaLink.Host.Contains($storageAccountName)} | Select-Object -ExpandProperty AttachedTo -Property `
          DiskName | Format-List -Property RoleName, DiskName

        ```
    * Удаление неподключенных дисков виртуальной машины, хранящихся в учетной записи хранения.

        Найдите в учетной записи хранения неподключенные диски виртуальных машин с помощью следующей команды:

        ```powershell
            $storageAccountName = 'yourStorageAccountName'
            Get-AzureDisk | where-Object {$_.MediaLink.Host.Contains($storageAccountName)} | Where-Object -Property AttachedTo -EQ $null | Format-List -Property DiskName  

        ```
        Если предыдущая команда возвращает диски, удалите эти диски с помощью следующей команды:

        ```powershell
           Remove-AzureDisk -DiskName 'yourDiskName'
        ```
    * Удаление образов виртуальных машин, хранящихся в учетной записи хранения.

        Следующая команда возвращает все образы виртуальных машин с дисками ОС, хранящимися в учетной записи хранения.
         ```powershell
            Get-AzureVmImage | Where-Object { $_.OSDiskConfiguration.MediaLink -ne $null -and $_.OSDiskConfiguration.MediaLink.Host.Contains($storageAccountName)`
                                    } | Select-Object -Property ImageName, ImageLabel
         ```
         Приведенная ниже команда возвращает сведения об образах всех виртуальных машин, диски данных которых хранятся в учетной записи хранения.
         ```powershell

            Get-AzureVmImage | Where-Object {$_.DataDiskConfigurations -ne $null `
                                             -and ($_.DataDiskConfigurations | Where-Object {$_.MediaLink -ne $null -and $_.MediaLink.Host.Contains($storageAccountName)}).Count -gt 0 `
                                            } | Select-Object -Property ImageName, ImageLabel
         ```
        Удалите все образы виртуальных машин, возвращенные предыдущими командами, с помощью следующей команды:
        ```powershell
        Remove-AzureVMImage -ImageName 'yourImageName'
        ```
* Проверьте учетные записи хранения и начните миграцию.

    Подготовьте каждую учетную запись хранения к переносу, используя следующую команду. В этом примере имя учетной записи хранения — **myStorageAccount**. Замените имя в примере именем своей учетной записи хранения.

    ```powershell
        $storageAccountName = "myStorageAccount"
        Move-AzureStorageAccount -Validate -StorageAccountName $storageAccountName
    ```

    Следующим шагом является подготовка учетной записи хранения для миграции.

    ```powershell
        $storageAccountName = "myStorageAccount"
        Move-AzureStorageAccount -Prepare -StorageAccountName $storageAccountName
    ```

    Проверьте конфигурацию для подготовленной учетной записи хранения с помощью Azure PowerShell или портала Azure. Если вы еще не готовы выполнить миграцию и хотите вернуться к старому состоянию, используйте следующую команду:

    ```powershell
        Move-AzureStorageAccount -Abort -StorageAccountName $storageAccountName
    ```

    Если подготовленная конфигурация вас устраивает, можете продолжить процесс. Выполните следующую команду, чтобы зафиксировать ресурсы.

    ```powershell
        Move-AzureStorageAccount -Commit -StorageAccountName $storageAccountName
    ```

## <a name="next-steps"></a>Дальнейшие действия
* [Обзор поддерживаемого платформой переноса ресурсов IaaS из классической модели в модель Azure Resource Manager](migration-classic-resource-manager-overview.md)
* [Техническое руководство по поддерживаемому платформой переносу из классической модели в модель Azure Resource Manager](migration-classic-resource-manager-deep-dive.md)
* [Planning for migration of IaaS resources from classic to Azure Resource Manager](migration-classic-resource-manager-plan.md) (Планирование переноса ресурсов IaaS из классической модели в модель Azure Resource Manager)
* [Перенос ресурсов IaaS из классической модели в модель Azure Resource Manager с помощью интерфейса командной строки Azure](migration-classic-resource-manager-cli.md)
* [Community tools to migrate IaaS resources from classic to Azure Resource Manager](migration-classic-resource-manager-community-tools.md) (Инструменты сообщества для перемещения ресурсов IaaS из классической модели в модель Azure Resource Manager)
* [Распространенные ошибки миграции](migration-classic-resource-manager-errors.md)
* [Часто задаваемые вопросы о миграции из классической модели в модель Azure Resource Manager](migration-classic-resource-manager-faq.md)
