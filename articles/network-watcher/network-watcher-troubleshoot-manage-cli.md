---
title: Устранение неполадок шлюза виртуальной сети Azure и подключений — Azure CLI
titleSuffix: Azure Network Watcher
description: На этой странице объясняется, как устранить неполадки с помощью Наблюдателя за сетями и Azure CLI
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/07/2021
ms.author: damendo
ms.openlocfilehash: d04ba6cc5f8527f8c05fbeb323c65230447e1e2e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98019709"
---
# <a name="troubleshoot-virtual-network-gateway-and-connections-using-azure-network-watcher-azure-cli"></a>Устранение неполадок шлюза виртуальной сети и подключений с помощью Наблюдателя за сетями Azure и Azure CLI

> [!div class="op_single_selector"]
> - [Портал](diagnose-communication-problem-between-networks.md)
> - [PowerShell](network-watcher-troubleshoot-manage-powershell.md)
> - [Azure CLI](network-watcher-troubleshoot-manage-cli.md)
> - [REST API](network-watcher-troubleshoot-manage-rest.md)

Наблюдатель за сетями предоставляет множество возможностей, так как он позволяет проанализировать сетевые ресурсы в Azure. Одна из этих возможностей — устранение неполадок в ресурсах. Процедуру устранения неполадок с ресурсами можно вызывать с помощью портала, PowerShell, интерфейса командной строки или API-интерфейса REST. При вызове Наблюдатель за сетями проверяет работоспособность шлюза виртуальной сети или подключения и возвращает результаты.

Для выполнения действий, описанных в этой статье, требуется [установить Azure CLI для Mac, Linux и Windows](/cli/azure/install-azure-cli).

## <a name="before-you-begin"></a>Перед началом

В этом сценарии предполагается, что вы создали Наблюдатель за сетями в соответствии с инструкциями в статье [Create a Network Watcher](network-watcher-create.md) (Создание Наблюдателя за сетями).

Список поддерживаемых типов шлюзов см. в разделе [Поддерживаемые типы шлюзов](network-watcher-troubleshoot-overview.md#supported-gateway-types).

## <a name="overview"></a>Обзор

В ходе устранения неполадок с ресурсами вы можете устранить проблемы, возникающие со шлюзами виртуальной сети и подключениями. При запросе на устранение неполадок с ресурсами запрашиваются и проверяются журналы. По завершении проверки возвращаются результаты. Запросы на устранение неполадок с ресурсами выполняются долго, поэтому для возвращения результатов может потребоваться несколько минут. Журналы, используемые при устранении неполадок, хранятся в контейнере указанной учетной записи хранения.

## <a name="retrieve-a-virtual-network-gateway-connection"></a>Получение сведений о подключении к шлюзу виртуальной сети

В этом примере выполняется устранение неполадок с таким ресурсом, как подключение. Его также можно передать шлюзу виртуальной сети. Следующий командлет выдает список VPN-подключений в группе ресурсов.

```azurecli
az network vpn-connection list --resource-group resourceGroupName
```

Получив имя подключения, можно выполнить приведенную ниже команду, чтобы получить его идентификатор ресурса.

```azurecli
az network vpn-connection show --resource-group resourceGroupName --ids vpnConnectionIds
```

## <a name="create-a-storage-account"></a>Создание учетной записи хранения

При устранении неполадок с ресурсами возвращаются данные о работоспособности ресурса, а также сохраняются журналы в учетную запись хранения для анализа. На этом этапе мы создадим учетную запись хранения. При необходимости можно использовать имеющуюся учетную запись хранения.

1. Создание учетной записи хранения

    ```azurecli
    az storage account create --name storageAccountName --location westcentralus --resource-group resourceGroupName --sku Standard_LRS
    ```

1. Получение ключей учетной записи хранения

    ```azurecli
    az storage account keys list --resource-group resourcegroupName --account-name storageAccountName
    ```

1. Создание контейнера

    ```azurecli
    az storage container create --account-name storageAccountName --account-key {storageAccountKey} --name logs
    ```

## <a name="run-network-watcher-resource-troubleshooting"></a>Выполнение устранения неполадок с ресурсами Наблюдателя за сетями

Для устранения неполадок с ресурсами используется командлет `az network watcher troubleshooting`. В командлет нужно передать группу ресурсов, имя экземпляра Наблюдателя за сетями, идентификатор подключения, идентификатор учетной записи хранения и путь к большому двоичному объекту, в котором будут храниться результаты устранения неполадок.

```azurecli
az network watcher troubleshooting start --resource-group resourceGroupName --resource resourceName --resource-type {vnetGateway/vpnConnection} --storage-account storageAccountName  --storage-path https://{storageAccountName}.blob.core.windows.net/{containerName}
```

После выполнения командлета Наблюдатель за сетями просматривает ресурсы, чтобы проверить их работоспособность. Он возвращает результаты в оболочку и сохраняет журналы результатов в указанной учетной записи хранения.

## <a name="understanding-the-results"></a>Анализ результатов

Текст действий содержит общие рекомендации по устранению проблемы. Если для устранения проблемы можно что-то сделать, предоставляется ссылка на дополнительные инструкции. Если дополнительных рекомендаций нет, в ответе указывается URL-адрес, открыв который можно отправить запрос в службу поддержки.  Дополнительные сведения о свойствах ответа, а также о данных, которые он содержит, см. в статье [Network Watcher Troubleshoot overview](network-watcher-troubleshoot-overview.md) (Обзор устранения неполадок наблюдателя за сетями).

Инструкции по скачиванию файлов из учетных записей хранения Azure см. в статье [Приступая к работе с хранилищем BLOB-объектов Azure с помощью .NET](../storage/blobs/storage-quickstart-blobs-dotnet.md). Кроме того, можно использовать такое средство, как Storage Explorer. Дополнительные сведения о Обозреватель службы хранилища можно найти по следующей ссылке: [Обозреватель службы хранилища](https://storageexplorer.com/)

## <a name="next-steps"></a>Дальнейшие действия

Если изменены параметры, которые мешают VPN-подключению, см. статью [Управление группами безопасности сети с помощью портала](../virtual-network/manage-network-security-group.md), чтобы найти сведения о группах безопасности сети и соответствующие правила безопасности.