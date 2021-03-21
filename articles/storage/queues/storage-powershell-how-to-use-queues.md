---
title: Как использовать хранилище очередей Azure из PowerShell в службе хранилища Azure
description: Выполнение операций с хранилищем очередей Azure с помощью PowerShell. С помощью хранилища очередей Azure можно хранить большое количество сообщений, доступных по протоколу HTTP/HTTPS.
author: mhopkins-msft
ms.author: mhopkins
ms.reviewer: dineshm
ms.date: 05/15/2019
ms.topic: how-to
ms.service: storage
ms.subservice: queues
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: fba288f76377e744b1fe21a52e03a43409c505bf
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97585621"
---
# <a name="how-to-use-azure-queue-storage-from-powershell"></a>Использование хранилища очередей Azure из PowerShell

Хранилище очередей Azure — это служба для хранения большого количества сообщений, доступ к которым можно получить из любой точки мира по протоколу HTTP или HTTPS. Подробные сведения см. [в статье Введение в хранилище очередей Azure](storage-queues-introduction.md). В этой статье рассматриваются распространенные операции с хранилищем очередей. Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
>
> - Создание очереди
> - Извлечение очереди
> - Добавление сообщения
> - Чтение сообщения
> - Удаление сообщения
> - Удаление очереди

Для работы с этим руководством требуется модуль Azure PowerShell ( `Az` ) версии 0,7 или более поздней. Выполните команду `Get-Module -ListAvailable Az` , чтобы найти текущую установленную версию. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-az-ps).

В PowerShell нет командлетов плоскости данных для очередей. Чтобы выполнять операции плоскости данных, такие как добавление, чтение и удаление сообщений, необходимо использовать клиентскую библиотеку хранилища .NET, так как она предоставлена в PowerShell. Вы создаете объект сообщения, а затем можете использовать такие команды, как `AddMessage` для выполнения операций с этим сообщением. В этой статье показано, как это сделать.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="sign-in-to-azure"></a>Вход в Azure

Войдите в подписку Azure с помощью команды `Connect-AzAccount` и следуйте инструкциям на экране.

```powershell
Connect-AzAccount
```

## <a name="retrieve-list-of-locations"></a>Получение списка расположений

Если вы не знаете, какое расположение нужно использовать, можно получить список доступных расположений. Получив список, найдите расположение, которое нужно использовать. В этом упражнении будет использоваться `eastus` . Сохраните это в переменной `location` для будущего использования.

```powershell
Get-AzLocation | Select-Object Location
$location = "eastus"
```

## <a name="create-resource-group"></a>Создать группу ресурсов

Создайте группу ресурсов с помощью команды [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup).

Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими. Сохраните имя группы ресурсов в переменной для будущего использования. В этом примере создается группа ресурсов `howtoqueuesrg` в регионе `eastus`.

```powershell
$resourceGroup = "howtoqueuesrg"
New-AzResourceGroup -ResourceGroupName $resourceGroup -Location $location
```

## <a name="create-storage-account"></a>Создание учетной записи хранения

Создайте стандартную учетную запись хранения общего назначения с локально избыточным хранилищем (LRS) с помощью команды [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount). Получите контекст учетной записи хранения, определяющий необходимую учетную запись хранения. Действуя в учетной записи хранения, ссылайтесь на контекст, вместо того чтобы многократно предоставлять учетные данные.

```powershell
$storageAccountName = "howtoqueuestorage"
$storageAccount = New-AzStorageAccount -ResourceGroupName $resourceGroup `
  -Name $storageAccountName `
  -Location $location `
  -SkuName Standard_LRS

$ctx = $storageAccount.Context
```

## <a name="create-a-queue"></a>Создание очереди

Сначала в этом примере устанавливается соединение со службой хранилища Azure, используя контекст учетной записи хранения, который включает имя учетной записи хранения и ее ключ доступа. Затем вызывается командлет [New-азсторажекуеуе](/powershell/module/az.storage/new-azstoragequeue) для создания очереди с именем `howtoqueue` .

```powershell
$queueName = "howtoqueue"
$queue = New-AzStorageQueue –Name $queueName -Context $ctx
```

Сведения о соглашениях об именовании для хранилища очередей Azure см. в статье [именование очередей и метаданных](/rest/api/storageservices/naming-queues-and-metadata).

## <a name="retrieve-a-queue"></a>Извлечение очереди

Вы можете запросить и получить определенную очередь или список всех очередей в учетной записи хранения. В следующих примерах демонстрируется извлечение всех очередей в учетной записи хранения и определенной очереди. Обе команды используют командлет [Get-AzStorageQueue](/powershell/module/az.storage/get-azstoragequeue).

```powershell
# Retrieve a specific queue
$queue = Get-AzStorageQueue –Name $queueName –Context $ctx
# Show the properties of the queue
$queue

# Retrieve all queues and show their names
Get-AzStorageQueue -Context $ctx | Select-Object Name
```

## <a name="add-a-message-to-a-queue"></a>Добавление сообщения в очередь

Для операций с фактическими сообщениями в очереди используйте клиентскую библиотеку хранилища .NET, предоставленную в PowerShell. Чтобы добавить сообщение в очередь, создайте новый экземпляр объекта Message ( [`Microsoft.Azure.Storage.Queue.CloudQueueMessage`](/java/api/com.microsoft.azure.storage.queue.cloudqueuemessage) класс). Затем вызовите [`AddMessage`](/java/api/com.microsoft.azure.storage.queue.cloudqueue.addmessage) метод. `CloudQueueMessage`Можно создать либо из строки (в формате UTF-8), либо в массиве байтов.

В следующем примере демонстрируется добавление сообщений в очередь.

```powershell
# Create a new message using a constructor of the CloudQueueMessage class
$queueMessage = [Microsoft.Azure.Storage.Queue.CloudQueueMessage]::new("This is message 1")

# Add a new message to the queue
$queue.CloudQueue.AddMessageAsync($QueueMessage)

# Add two more messages to the queue
$queueMessage = [Microsoft.Azure.Storage.Queue.CloudQueueMessage]::new("This is message 2")
$queue.CloudQueue.AddMessageAsync($QueueMessage)

$queueMessage = [Microsoft.Azure.Storage.Queue.CloudQueueMessage]::new("This is message 3")
$queue.CloudQueue.AddMessageAsync($QueueMessage)
```

Если вы используете [Обозреватель службы хранилища Azure](https://storageexplorer.com), можно подключиться к учетной записи хранения Azure и вывести на экран содержащиеся в ней очереди, а также детализировать одну из них, чтобы просмотреть сообщения в ней.

## <a name="read-a-message-from-the-queue-then-delete-it"></a>Считывание сообщения из очереди и его удаление

Сообщения считываются в порядке их поступления. Это не гарантируется. При считывании сообщения из очереди оно становится невидимым для других процессов, просматривающих очередь. Это позволяет удостовериться, что если коду не удастся обработать сообщение из-за сбоя оборудования или программного обеспечения, другой экземпляр кода сможет получить то же сообщение и повторить попытку.

Это **время ожидания невидимости** определяет, как долго сообщение остается невидимым, прежде чем снова станет доступным для обработки. По умолчанию это 30 секунд.

Ваш код считывает сообщение из очереди в два этапа. При вызове [`Microsoft.Azure.Storage.Queue.CloudQueue.GetMessage`](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.getmessage) метода вы получаете следующее сообщение в очереди. Сообщение, возвращаемое методом `GetMessage`, становится невидимым для другого кода, считывающего сообщения из этой очереди. Чтобы завершить удаление сообщения из очереди, вызовите [`Microsoft.Azure.Storage.Queue.CloudQueue.DeleteMessage`](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.deletemessage) метод.

В следующем примере можно прочитать три сообщения в очереди, после чего следует время ожидания 10 секунд (время ожидания невидимости). Затем снова считываются три сообщения, удаляя их после их чтения путем вызова `DeleteMessage` . При попытке чтения очереди после удаления сообщений `$queueMessage` будет возвращено значение `$null` .

```powershell
# Set the amount of time you want to entry to be invisible after read from the queue
# If it is not deleted by the end of this time, it will show up in the queue again
$invisibleTimeout = [System.TimeSpan]::FromSeconds(10)

# Read the message from the queue, then show the contents of the message. Read the other two messages, too.
$queueMessage = $queue.CloudQueue.GetMessageAsync($invisibleTimeout,$null,$null)
$queueMessage.Result
$queueMessage = $queue.CloudQueue.GetMessageAsync($invisibleTimeout,$null,$null)
$queueMessage.Result
$queueMessage = $queue.CloudQueue.GetMessageAsync($invisibleTimeout,$null,$null)
$queueMessage.Result

# After 10 seconds, these messages reappear on the queue.
# Read them again, but delete each one after reading it.
# Delete the message.
$queueMessage = $queue.CloudQueue.GetMessageAsync($invisibleTimeout,$null,$null)
$queueMessage.Result
$queue.CloudQueue.DeleteMessageAsync($queueMessage.Result.Id,$queueMessage.Result.popReceipt)
$queueMessage = $queue.CloudQueue.GetMessageAsync($invisibleTimeout,$null,$null)
$queueMessage.Result
$queue.CloudQueue.DeleteMessageAsync($queueMessage.Result.Id,$queueMessage.Result.popReceipt)
$queueMessage = $queue.CloudQueue.GetMessageAsync($invisibleTimeout,$null,$null)
$queueMessage.Result
$queue.CloudQueue.DeleteMessageAsync($queueMessage.Result.Id,$queueMessage.Result.popReceipt)
```

## <a name="delete-a-queue"></a>Удаление очереди

Чтобы удалить очередь и все сообщения, содержащиеся в ней, вызовите `Remove-AzStorageQueue` командлет. В следующем примере показано, как удалить конкретную очередь, используемую в этом упражнении, с помощью `Remove-AzStorageQueue` командлета.

```powershell
# Delete the queue
Remove-AzStorageQueue –Name $queueName –Context $ctx
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы удалить все ресурсы, созданные в этом упражнении, удалите группу ресурсов. При этом будут также удалены все ресурсы, содержащиеся в группе. В этом случае происходит удаление созданной учетной записи хранения и самой группы ресурсов.

```powershell
Remove-AzResourceGroup -Name $resourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

В этом пошаговом руководстве вы узнали об базовом управлении хранилищем очередей с помощью PowerShell, в том числе:

> [!div class="checklist"]
>
> - Создание очереди
> - Извлечение очереди
> - Добавление сообщения
> - Чтение следующего сообщения
> - Удаление сообщения
> - Удаление очереди

### <a name="microsoft-azure-powershell-storage-cmdlets"></a>Командлеты хранилища Microsoft Azure PowerShell

- [Командлеты PowerShell для службы хранилища](/powershell/module/az.storage)

### <a name="microsoft-azure-storage-explorer"></a>Обозреватель службы хранилища Microsoft Azure

- [Обозреватель хранилищ Microsoft Azure](../../vs-azure-tools-storage-manage-with-storage-explorer.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json) — это бесплатное автономное приложение от корпорации Майкрософт, позволяющее визуализировать данные из службы хранилища Azure на платформе Windows, macOS и Linux.
