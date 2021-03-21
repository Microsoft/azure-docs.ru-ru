---
title: Устранение неполадок Локатионнотфаундфорролесизе при развертывании облачной службы (классическая модель) в Azure | Документация Майкрософт
description: В этой статье показано, как устранить исключение Локатионнотфаундфорролесизе при развертывании облачной службы (классической) в Azure.
services: cloud-services
author: mibufo
ms.author: v-mibufo
ms.service: cloud-services
ms.topic: troubleshooting
ms.date: 02/22/2021
ms.openlocfilehash: b11aedb52be3c263c781c2ac68d1d5197ba4def2
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101746746"
---
# <a name="troubleshoot-locationnotfoundforrolesize-when-deploying-a-cloud-service-classic-to-azure"></a>Устранение неполадок Локатионнотфаундфорролесизе при развертывании облачной службы (классической) в Azure

В этой статье вы узнаете об устранении ошибок выделения ресурсов, когда размер виртуальной машины недоступен при развертывании облачной службы Azure (классическая модель).

При развертывании экземпляров в облачной службе (классической) или добавлении новых экземпляров веб-служб или рабочих ролей Microsoft Azure выделяет ресурсы для вычислений.

Иногда при выполнении этих операций могут возникать ошибки даже до достижения лимита подписки Azure.

> [!TIP]
> Эта информация также может быть полезна при планировании развертывания служб.

## <a name="symptom"></a>Симптом

В портал Azure перейдите к облачной службе (классическая) и в боковой панели выберите *Журнал операций (классическая модель)* , чтобы просмотреть журналы.

![На рисунке показана колонка журнал операций (классическая).](./media/cloud-services-troubleshoot-location-not-found-for-role-size/cloud-services-troubleshoot-allocation-logs.png)

При проверке журналов облачной службы (классическая модель) вы увидите следующее исключение:

|Тип исключения  |Сообщение об ошибке  |
|---------|---------|
|локатионнотфаундфорролесизе     |Операция " `{Operation ID}` " завершилась ошибкой: "запрошенный уровень виртуальной машины в настоящее время недоступен в регионе ( `{Region ID}` ) для этой подписки. Попробуйте другой уровень или выполните развертывание в другом расположении. ".|

## <a name="cause"></a>Причина

Существует ошибка емкости для региона или кластера, в который выполняется развертывание. Исключение *локатионнотфаундфорролесизе* возникает, когда выбранный номер SKU ресурса (размер виртуальной машины) недоступен для указанной области.

## <a name="solution"></a>Решение

В этом сценарии следует выбрать другой регион или номер SKU для развертывания облачной службы (классическая модель) в. Перед развертыванием или обновлением облачной службы (классической) можно определить, какие номера SKU доступны в регионе или зоне доступности. Следуйте приведенным ниже процессам [Azure CLI](#list-skus-in-region-using-azure-cli), [PowerShell](#list-skus-in-region-using-powershell)или [REST API](#list-skus-in-region-using-rest-api) .

### <a name="list-skus-in-region-using-azure-cli"></a>Вывод списка номеров SKU в регионе с помощью Azure CLI

Можно использовать команду [AZ VM List-SKU](https://docs.microsoft.com/cli/azure/vm.html#az_vm_list_skus) .

- Используйте `--location` параметр, чтобы отфильтровать выходные данные в используемое расположение.
- Параметр `--size` позволяет выполнить поиск по частичному названию размера.
- Дополнительные сведения см. в разделе [Устранение ошибки для недоступности SKU](../azure-resource-manager/templates/error-sku-not-available.md#solution-2---azure-cli) .

    **Пример:**

    ```azurecli
    az vm list-skus --location southcentralus --size Standard_F --output table
    ```

    **Пример результатов:** ![ Azure CLI выводятся результаты выполнения команды AZ VM List-SKU--Location southcentralus--size Standard_F--выходной таблице ", которая показывает доступные номера SKU.](./media/cloud-services-troubleshoot-constrained-allocation-failed/cloud-services-troubleshoot-constrained-allocation-failed-1.png)

#### <a name="list-skus-in-region-using-powershell"></a>Вывод списка номеров SKU в регионе с помощью PowerShell

Можно использовать команду [Get-азкомпутересаурцеску](https://docs.microsoft.com/powershell/module/az.compute/get-azcomputeresourcesku) .

- Отфильтруйте результаты по расположению.
- Эта команда поддерживается только в Azure PowerShell последней версии.
- Дополнительные сведения см. в разделе [Устранение ошибки для недоступности SKU](../azure-resource-manager/templates/error-sku-not-available.md#solution-1---powershell) .

**Пример:**

```azurepowershell
Get-AzComputeResourceSku | where {$_.Locations -icontains "centralus"}
```

**Некоторые другие полезные команды:**

Отфильтровать расположения, которые содержат размер (Standard_DS14_v2):

```azurepowershell
Get-AzComputeResourceSku | where {$_.Locations.Contains("centralus") -and $_.ResourceType.Contains("virtualMachines") -and $_.Name.Contains("Standard_DS14_v2")}
```

Отфильтровать все расположения, содержащие размер (v3):

```azurepowershell
Get-AzComputeResourceSku | where {$_.Locations.Contains("centralus") -and $_.ResourceType.Contains("virtualMachines") -and $_.Name.Contains("v3")} | fc
```

#### <a name="list-skus-in-region-using-rest-api"></a>Вывод списка номеров SKU в регионе с помощью REST API

Вы можете использовать операцию " [список номеров SKU ресурсов](https://docs.microsoft.com/rest/api/compute/resourceskus/list) ". Он возвращает доступные номера SKU и регионы в приведенном ниже формате.

```json
{
  "value": [
    {
      "resourceType": "virtualMachines",
      "name": "Standard_A0",
      "tier": "Standard",
      "size": "A0",
      "locations": [
        "eastus"
      ],
      "restrictions": []
    },
    {
      "resourceType": "virtualMachines",
      "name": "Standard_A1",
      "tier": "Standard",
      "size": "A1",
      "locations": [
        "eastus"
      ],
      "restrictions": []
    },
      <<The rest of your file is located here>>
  ]
}
    
```

## <a name="next-steps"></a>Дальнейшие действия

Для получения дополнительных решений об ошибках выделения ресурсов и для лучшего понимания того, как они создаются:

> [!div class="nextstepaction"]
> [Сбои при выделении — облачная служба (классическая)](cloud-services-allocation-failures.md)

Если проблема с Azure не устранена в этой статье, посетите форумы Azure в [MSDN и Stack overflow](https://azure.microsoft.com/support/forums/). Описание своей проблемы можно опубликовать на этих форумах или [в Twitter (@AzureSupport)](https://twitter.com/AzureSupport). Также можно отправить запрос в службу поддержки Azure. Чтобы отправить такой запрос, на странице [поддержки Azure](https://azure.microsoft.com/support/options/) щелкните *Получить поддержку*.
