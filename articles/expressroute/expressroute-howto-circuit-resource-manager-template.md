---
title: Шаблон Azure ExpressRoute. Создание канала ExpressRoute
description: Узнайте, как создать канал Azure ExpressRoute, развернув шаблон Azure Resource Manager с помощью Azure PowerShell.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 11/13/2019
ms.author: duau
ms.openlocfilehash: 2e9b6ddc9da4467590946af12a47f1473a4ea494
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92202061"
---
# <a name="create-an-expressroute-circuit-by-using-azure-resource-manager-template"></a>Создание канала ExpressRoute с помощью шаблона Azure Resource Manager

> [!div class="op_single_selector"]
> * [Портал Azure](expressroute-howto-circuit-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-circuit-arm.md)
> * [Azure CLI](howto-circuit-cli.md)
> * [Шаблон Azure Resource Manager](expressroute-howto-circuit-resource-manager-template.md)
> * [Видео — портал Azure](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-an-expressroute-circuit)
> * [PowerShell (классическая модель)](expressroute-howto-circuit-classic.md)
>

Узнайте, как создать канал ExpressRoute путем развертывания шаблона Azure Resource Manager с помощью Azure PowerShell. Дополнительные сведения о разработке шаблонов Resource Manager см. в разделе [Документация по Azure Resource Manager](../azure-resource-manager/index.yml) и в [справочнике по шаблонам](/azure/templates/microsoft.network/expressroutecircuits).

## <a name="before-you-begin"></a>Перед началом

* Изучите [предварительные требования](expressroute-prerequisites.md) и [рабочие процессы](expressroute-workflows.md), прежде чем приступить к настройке.
* Убедитесь в том, что у вас есть разрешения на создание сетевых ресурсов. Если у вас нет нужных разрешений, обратитесь к администратору учетной записи.
* Вы можете [просмотреть видео](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-an-expressroute-circuit), прежде чем начать, чтобы лучше понять выполняемые действия.

## <a name="create-and-provision-an-expressroute-circuit"></a><a name="create"></a>Создание и предоставление канала ExpressRoute

[Шаблоны](https://azure.microsoft.com/resources/templates/) быстрого запуска Azure имеют хорошую коллекцию шаблонов диспетчер ресурсов. Для создания канала ExpressRoute используется один из [существующих шаблонов](https://azure.microsoft.com/resources/templates/101-expressroute-circuit-create/) .

[!code-json[create-azure-expressroute-circuit](~/quickstart-templates/101-expressroute-circuit-create/azuredeploy.json)]

Чтобы просмотреть дополнительные связанные шаблоны, выберите [здесь](https://azure.microsoft.com/resources/templates/?term=expressroute).

Чтобы создать канал ExpressRoute путем развертывания шаблона, выполните следующие действия.

1. Выберите вариант **попробовать** в следующем блоке кода, а затем следуйте инструкциям по входу в Azure Cloud Shell.

    ```azurepowershell-interactive
    $circuitName = Read-Host -Prompt "Enter a circuit name"
    $location = Read-Host -Prompt "Enter the location (i.e. centralus)"
    $resourceGroupName = "${circuitName}rg"
    $templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-expressroute-circuit-create/azuredeploy.json"

    $serviceProviderName = "Equinix"
    $peeringLocation = "Silicon Valley"
    $bandwidthInMbps = 500
    $sku_tier = "Premium"
    $sku_family = "MeteredData"

    New-AzResourceGroup -Name $resourceGroupName -Location $location
    New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri -circuitName $circuitName -serviceProviderName $serviceProviderName -peeringLocation $peeringLocation -bandwidthInMbps $bandwidthInMbps -sku_tier $sku_tier -sku_family $sku_family

    Write-Host "Press [ENTER] to continue ..."
    ```

   * **Уровень SKU** определяет, является ли канал ExpressRoute [локальным](expressroute-faqs.md#expressroute-local), стандартным или [Premium](expressroute-faqs.md#expressroute-premium). Вы можете указать *Локальный*, *"Стандартный" или *Премиум*. Изменить номер SKU *Стандартный или Премиум* на *Локальный* нельзя.
   * **Семейство SKU** определяет тип выставления счетов. Вы можете указать *Metereddata* для тарифного тарифного плана и *Unlimiteddata* для неограниченного тарифного плана данных. Тип выставления счетов можно изменить с *Metereddata* на *Unlimiteddata*, но нельзя изменить тип с *Unlimiteddata* на *Metereddata*. *Локальный* канал — только *Unlimiteddata* .
   * **Расположение пиринга** — физическое расположение пиринга с корпорацией Майкрософт.

     > [!IMPORTANT]
     > Параметр "Расположение пиринга" определяет [физическое расположение](expressroute-locations.md) пиринга с корпорацией Майкрософт. Оно **не** связано со свойством Location, которое ссылается на географический регион, в котором находится поставщик сетевых ресурсов Azure. Хотя они не связаны, рекомендуется выбрать поставщик сетевых ресурсов, находящийся недалеко от расположения пиринга канала.

    Имя группы ресурсов — это имя пространства имен служебной шины с добавленным **RG** .

2. Нажмите кнопку **Копировать**, чтобы скопировать сценарий PowerShell.
3. Щелкните правой кнопкой в консоли оболочки и выберите **Вставить**.

Создание концентратора событий занимает несколько секунд.

Azure PowerShell используется для развертывания шаблона в этом руководстве. Другие методы развертывания шаблонов см. в следующих статьях:

* С [помощью портал Azure](../azure-resource-manager/templates/deploy-portal.md).
* С [помощью Azure CLI](../azure-resource-manager/templates/deploy-cli.md).
* С [помощью REST API](../azure-resource-manager/templates/deploy-rest.md).

## <a name="deprovisioning-and-deleting-an-expressroute-circuit"></a><a name="delete"></a>отзыве и удалении канала ExpressRoute

Вы можете удалить канал ExpressRoute, щелкнув значок **удаления** . Обратите внимание на следующие сведения:

* Связь между ExpressRoute и всеми виртуальными сетями необходимо разорвать. Если операция завершится ошибкой, проверьте, не привязаны ли к каналу какие-либо виртуальные сети.
* Если подготовка поставщика услуг канала ExpressRoute находится в состоянии **Идет подготовка** или **Подготовлено** то свяжитесь с поставщиком услуг, чтобы отозвать канал с его стороны. Мы будем резервировать ресурсы и выставлять вам счета до тех пор, пока поставщик услуг не завершит отзыв канала и не отправит нам соответствующее уведомление.
* Если поставщик услуг отзовет канал (состояние подготовки поставщика услуг изменится на **Не подготовлено**), вы можете удалить такой канал. Это приостанавливает выставление счетов для канала.

Вы можете удалить канал ExpressRoute, выполнив следующую команду PowerShell:

```azurepowershell-interactive
$circuitName = Read-Host -Prompt "Enter the same circuit name that you used earlier"
$resourceGroupName = "${circuitName}rg"

Remove-AzExpressRouteCircuit -ResourceGroupName $resourceGroupName -Name $circuitName
```

## <a name="next-steps"></a>Дальнейшие действия

Создав канал, выполните задачи, описанные в следующих статьях:

* [Создание и изменение маршрутизации для канала ExpressRoute](expressroute-howto-routing-portal-resource-manager.md)
* [Связывание виртуальной сети с каналом ExpressRoute](expressroute-howto-linkvnet-arm.md)