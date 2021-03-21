---
title: Управление записью пакетов с помощью Наблюдателя за сетями Azure (Azure CLI) | Документация Майкрософт
description: В этой статье объясняется, как с помощью Azure CLI управлять функцией записи пакетов в службе "Наблюдатель за сетями".
services: network-watcher
documentationcenter: na
author: damendo
ms.assetid: cb0c1d10-f7f2-4c34-b08c-f73452430be8
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/07/2021
ms.author: damendo
ms.openlocfilehash: efb8fba655570a661b6b001eb93942475eb2170d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101698643"
---
# <a name="manage-packet-captures-with-azure-network-watcher-using-the-azure-cli"></a>Управление записью пакетов с помощью службы "Наблюдатель за сетями Azure" посредством Azure CLI

> [!div class="op_single_selector"]
> - [Портал Azure](network-watcher-packet-capture-manage-portal.md)
> - [PowerShell](network-watcher-packet-capture-manage-powershell.md)
> - [Azure CLI](network-watcher-packet-capture-manage-cli.md)
> - [Azure REST API](network-watcher-packet-capture-manage-rest.md)

Возможность записи пакетов Наблюдателя за сетями позволяет создавать сеансы записи для отслеживания входящего и исходящего трафика виртуальной машины. Для сеанса записи предоставляются фильтры, которые позволяют убедиться, что записывается только требуемый трафик. Записи пакетов помогают выявить аномалии в работе сети по факту или заранее. Они также помогают выполнять сбор сетевой статистики, получать сведения о сетевых вторжениях, выполнять отладку передачи данных между клиентом и сервером и многое другое. Так как запись пакетов активируется удаленно, ее не нужно запускать вручную. К тому же она сразу выполняется на требуемой виртуальной машине, что также позволяет сэкономить ценное время.

Чтобы выполнить действия, описанные в этой статье, необходимо [установить интерфейс Azure Command-Line для Mac, Linux и Windows (Azure CLI)](/cli/azure/install-azure-cli).

В этой статье вы ознакомитесь с разными задачами управления, доступными в настоящее время для записи пакетов.

- [**Запуск записи пакета**](#start-a-packet-capture)
- [**Прекращение записи пакета**](#stop-a-packet-capture)
- [**Удаление записи пакета**](#delete-a-packet-capture)
- [**Скачивание записи пакета**](#download-a-packet-capture)

## <a name="before-you-begin"></a>Перед началом

В данной статье предполагается, что у вас есть следующие ресурсы:

- экземпляр Наблюдателя за сетями в регионе, в котором нужно создать запись пакетов;
- виртуальная машина с включенным расширением для записи пакетов.

> [!IMPORTANT]
> Для записи пакетов требуется, чтобы на виртуальной машине был запущен агент. Агент устанавливается в качестве расширения. Инструкции по расширениям виртуальных машин см. в статье [Обзор расширений и компонентов виртуальной машины под управлением Windows](../virtual-machines/extensions/features-windows.md).

## <a name="install-vm-extension"></a>Установка расширения виртуальной машины

### <a name="step-1"></a>Шаг 1

Выполните `az vm extension set` команду, чтобы установить агент записи пакетов на гостевой виртуальной машине.

Для виртуальных машин Windows:

```azurecli-interactive
az vm extension set --resource-group resourceGroupName --vm-name virtualMachineName --publisher Microsoft.Azure.NetworkWatcher --name NetworkWatcherAgentWindows --version 1.4
```

Для виртуальных машин Linux:

```azurecli-interactive
az vm extension set --resource-group resourceGroupName --vm-name virtualMachineName --publisher Microsoft.Azure.NetworkWatcher --name NetworkWatcherAgentLinux --version 1.4
```

### <a name="step-2"></a>Шаг 2

Чтобы убедиться, что агент установлен, выполните `vm extension show` команду и передайте ей имя группы ресурсов и виртуальной машины. Проверьте итоговый список, чтобы убедиться, что агент установлен.

Для виртуальных машин Windows:

```azurecli-interactive
az vm extension show --resource-group resourceGroupName --vm-name virtualMachineName --name NetworkWatcherAgentWindows
```

Для виртуальных машин Linux:

```azurecli-interactive
az vm extension show --resource-group resourceGroupName --vm-name virtualMachineName --name AzureNetworkWatcherExtension
```

Ниже приведен пример ответа после выполнения операции `az vm extension show`.

```json
{
  "autoUpgradeMinorVersion": true,
  "forceUpdateTag": null,
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}/extensions/NetworkWatcherAgentWindows",
  "instanceView": null,
  "location": "westcentralus",
  "name": "NetworkWatcherAgentWindows",
  "protectedSettings": null,
  "provisioningState": "Succeeded",
  "publisher": "Microsoft.Azure.NetworkWatcher",
  "resourceGroup": "{resourceGroupName}",
  "settings": null,
  "tags": null,
  "type": "Microsoft.Compute/virtualMachines/extensions",
  "typeHandlerVersion": "1.4",
  "virtualMachineExtensionType": "NetworkWatcherAgentWindows"
}
```

## <a name="start-a-packet-capture"></a>Запуск записи пакета

После выполнения предыдущих шагов на виртуальной машине будет установлен агент записи пакетов.

### <a name="step-1"></a>Шаг 1

Получите учетную запись хранения. Она используется для хранения файла записи пакетов.

```azurecli-interactive
az storage account list
```

### <a name="step-2"></a>Шаг 2

На этом этапе можно приступать к созданию записи пакетов.  Сначала давайте рассмотрим параметры, которые можно настроить. Фильтры — это один из таких параметров, который можно использовать для ограничения данных, хранящихся в записи пакета. В следующем примере настраивается запись пакетов с несколькими фильтрами.  Первые три фильтра собирают исходящий TCP-трафик только с локального IP-адреса 10.0.0.3 на порты назначения 20, 80 и 443.  Последний фильтр собирает только трафик, передаваемый по протоколу UDP.

```azurecli-interactive
az network watcher packet-capture create --resource-group {resourceGroupName} --vm {vmName} --name packetCaptureName --storage-account {storageAccountName} --filters "[{\"protocol\":\"TCP\", \"remoteIPAddress\":\"1.1.1.1-255.255.255\",\"localIPAddress\":\"10.0.0.3\", \"remotePort\":\"20\"},{\"protocol\":\"TCP\", \"remoteIPAddress\":\"1.1.1.1-255.255.255\",\"localIPAddress\":\"10.0.0.3\", \"remotePort\":\"80\"},{\"protocol\":\"TCP\", \"remoteIPAddress\":\"1.1.1.1-255.255.255\",\"localIPAddress\":\"10.0.0.3\", \"remotePort\":\"443\"},{\"protocol\":\"UDP\"}]"
```

Ниже приведен пример ожидаемого результата выполнения `az network watcher packet-capture create` команды.

```json
{
  "bytesToCapturePerPacket": 0,
  "etag": "W/\"b8cf3528-2e14-45cb-a7f3-5712ffb687ac\"",
  "filters": [
    {
      "localIpAddress": "10.0.0.3",
      "localPort": "",
      "protocol": "TCP",
      "remoteIpAddress": "1.1.1.1-255.255.255",
      "remotePort": "20"
    },
    {
      "localIpAddress": "10.0.0.3",
      "localPort": "",
      "protocol": "TCP",
      "remoteIpAddress": "1.1.1.1-255.255.255",
      "remotePort": "80"
    },
    {
      "localIpAddress": "10.0.0.3",
      "localPort": "",
      "protocol": "TCP",
      "remoteIpAddress": "1.1.1.1-255.255.255",
      "remotePort": "443"
    },
    {
      "localIpAddress": "",
      "localPort": "",
      "protocol": "UDP",
      "remoteIpAddress": "",
      "remotePort": ""
    }
  ],
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/NetworkWatcherRG/providers/Microsoft.Network/networkWatchers/NetworkWatcher_westcentralus/pa
cketCaptures/packetCaptureName",
  "name": "packetCaptureName",
  "provisioningState": "Succeeded",
  "resourceGroup": "NetworkWatcherRG",
  "storageLocation": {
    "filePath": null,
    "storageId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/gwteststorage123abc",
    "storagePath": "https://gwteststorage123abc.blob.core.windows.net/network-watcher-logs/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/{resourceGroupName}/p
roviders/microsoft.compute/virtualmachines/{vmName}/2017/05/25/packetcapture_16_22_34_630.cap"
  },
  "target": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}",
  "timeLimitInSeconds": 18000,
  "totalBytesPerSession": 1073741824
}
```

## <a name="get-a-packet-capture"></a>Получение записи пакета

Выполнение `az network watcher packet-capture show-status` команды, получение состояния выполняемой в данный момент или завершенной записи пакетов.

```azurecli-interactive
az network watcher packet-capture show-status --name packetCaptureName --location {networkWatcherLocation}
```

Ниже приведен пример выходных данных `az network watcher packet-capture show-status` команды. Следующий пример можно получить, когда для остановки указано значение Stopped, а для параметра StopReason задано значение TimeExceeded.

```
{
  "additionalProperties": {
    "status": "Succeeded"
  },
  "captureStartTime": "2016-12-06T17:20:01.5671279Z",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/NetworkWatcherRG/providers/Microsoft.Network/networkWatchers/NetworkWatcher_westcentralus/packetCaptures/packetCaptureName",
  "name": "packetCaptureName",
  "packetCaptureError": [],
  "packetCaptureStatus": "Stopped",
  "stopReason": "TimeExceeded"
}
```

## <a name="stop-a-packet-capture"></a>Прекращение записи пакета

Выполнив `az network watcher packet-capture stop` команду, если сеанс записи выполняется, он останавливается.

```azurecli-interactive
az network watcher packet-capture stop --name packetCaptureName --location westcentralus
```

> [!NOTE]
> Команда не возвращает ответ при выполнении в запущенном в данный момент сеансе записи или существующем сеансе, который уже остановлен.

## <a name="delete-a-packet-capture"></a>Удаление записи пакета

```azurecli-interactive
az network watcher packet-capture delete --name packetCaptureName --location westcentralus
```

> [!NOTE]
> При удалении записи пакета файл в учетной записи хранения не удаляется.

## <a name="download-a-packet-capture"></a>Скачивание записи пакета

После завершения сеанса записи пакета файл записи можно передать в хранилище BLOB-объектов или в локальный файл на виртуальной машине. Место хранения записи пакетов определяется при создании сеанса. Удобное средство для доступа к этим файлам записи, сохраненным в учетной записи хранения, — обозреватель службы хранилища Microsoft Azure, который можно скачать по адресу https://storageexplorer.com/

При указании учетной записи хранения файлы записи пакетов сохраняются в ней по следующему адресу:

```
https://{storageAccountName}.blob.core.windows.net/network-watcher-logs/subscriptions/{subscriptionId}/resourcegroups/{storageAccountResourceGroup}/providers/microsoft.compute/virtualmachines/{VMName}/{year}/{month}/{day}/packetCapture_{creationTime}.cap
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об автоматизации записи пакетов с помощью оповещений на виртуальной машине см. в статье, посвященной [созданию записи пакетов, активируемой с использованием оповещений](network-watcher-alert-triggered-packet-capture.md).

Сведения о состоянии (разрешен или запрещен) входящего и исходящего трафика виртуальной машины см. в статье, посвященной [проверке потока IP-адресов](diagnose-vm-network-traffic-filtering-problem.md).

<!-- Image references -->