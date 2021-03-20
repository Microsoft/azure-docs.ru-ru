---
title: Требовать безопасное перемещение для обеспечения безопасных соединений
titleSuffix: Azure Storage
description: Узнайте, как требовать безопасной пересылки запросов в службу хранилища Azure. Если для учетной записи хранения требуется безопасное перемещение, все запросы, поступающие от небезопасного подключения, отклоняются.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 04/21/2020
ms.author: tamram
ms.reviewer: fryu
ms.subservice: common
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: a0a942daae6b106bce928f8214e92c0c6294203d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100555721"
---
# <a name="require-secure-transfer-to-ensure-secure-connections"></a>Требовать безопасное перемещение для обеспечения безопасных соединений

Вы можете настроить учетную запись хранения для приема запросов только от безопасных подключений, задав для учетной записи хранения свойство **требуется безопасное перемещение** . При необходимости безопасной пересылки все запросы, исходящие из незащищенного соединения, отклоняются. Корпорация Майкрософт рекомендует всегда требовать безопасной пересылки для всех учетных записей хранения.

Если требуется безопасная операция передачи, вызов операции REST API службы хранилища Azure должен выполняться по протоколу HTTPS. Любой запрос, отправленный по протоколу HTTP, отклоняется.

Подключение к файловому ресурсу Azure через SMB без шифрования завершается сбоем, если для учетной записи хранения требуется безопасная отправка. Примеры небезопасных подключений включают в себя протоколы SMB 2,1, SMB 3,0 без шифрования или некоторые версии клиента SMB для Linux.

По умолчанию при создании учетной записи хранения включается свойство **Обязательное безопасное перемещение** .

> [!NOTE]
> Так как служба хранилища Azure не поддерживает протокол HTTPS для имен личных доменов, при использовании данных имен этот параметр не применяется. Классические учетные записи хранения не поддерживаются.

## <a name="require-secure-transfer-in-the-azure-portal"></a>Требовать безопасной пересылки в портал Azure

При создании учетной записи хранения в [портал Azure](https://portal.azure.com)можно включить свойство **требуется безопасное перемещение** . Его также можно включить для имеющихся учетных записей хранения.

### <a name="require-secure-transfer-for-a-new-storage-account"></a>Включение требования безопасной передачи для новой учетной записи хранения

1. Откройте область **Создание учетной записи хранения** на портале Azure.
1. В разделе **Требуется безопасное перемещение** щелкните **Включено**.

   ![Колонка "Создание учетной записи хранения"](./media/storage-require-secure-transfer/secure_transfer_field_in_portal_en_1.png)

### <a name="require-secure-transfer-for-an-existing-storage-account"></a>Включение требования безопасной передачи для существующей учетной записи хранения

1. Выберите учетную запись хранения на портале Azure.
1. В области меню учетной записи хранения в разделе **Параметры** щелкните **Конфигурация**.
1. В разделе **Требуется безопасное перемещение** щелкните **Включено**.

   ![Область меню учетной записи хранения](./media/storage-require-secure-transfer/secure_transfer_field_in_portal_en_2.png)

## <a name="require-secure-transfer-from-code"></a>Требовать безопасной перенос из кода

Чтобы обеспечить безопасное перемещение программными средствами, установите для свойства _enablehttpstrafficonly._ в учетной записи хранения _значение true_ . Это свойство можно задать с помощью REST API поставщика ресурсов хранилища, клиентских библиотек или средств.

* [REST API](/rest/api/storagerp/storageaccounts)
* [PowerShell](/powershell/module/az.storage/set-azstorageaccount)
* [CLI](/cli/azure/storage/account)
* [NodeJS](https://www.npmjs.com/package/@azure/arm-storage/)
* [Пакет SDK для .NET](https://www.nuget.org/packages/Microsoft.Azure.Management.Storage)
* [Пакет SDK для Python](https://pypi.org/project/azure-mgmt-storage)
* [Пакет SDK для Ruby](https://rubygems.org/gems/azure_mgmt_storage)

## <a name="require-secure-transfer-with-powershell"></a>Требовать безопасное перемещение с помощью PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Для работы с этим примером требуется модуль Azure PowerShell Az 0.7 или более поздней версии. Чтобы узнать версию, выполните команду `Get-Module -ListAvailable Az`. Если вам необходимо выполнить установку или обновление, см. статью [об установке модуля Azure PowerShell](/powershell/azure/install-Az-ps).

Выполните команду `Connect-AzAccount`, чтобы создать подключение к Azure.

 Для проверки параметра можно использовать командную строку ниже:

```powershell
Get-AzStorageAccount -Name "{StorageAccountName}" -ResourceGroupName "{ResourceGroupName}"
StorageAccountName     : {StorageAccountName}
Kind                   : Storage
EnableHttpsTrafficOnly : False
...

```

Для включения параметра можно использовать командную строку ниже:

```powershell
Set-AzStorageAccount -Name "{StorageAccountName}" -ResourceGroupName "{ResourceGroupName}" -EnableHttpsTrafficOnly $True
StorageAccountName     : {StorageAccountName}
Kind                   : Storage
EnableHttpsTrafficOnly : True
...

```

## <a name="require-secure-transfer-with-azure-cli"></a>Требовать безопасное перемещение с помощью Azure CLI

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

 Для проверки параметра используйте следующую команду:

```azurecli-interactive
az storage account show -g {ResourceGroupName} -n {StorageAccountName}
{
  "name": "{StorageAccountName}",
  "enableHttpsTrafficOnly": false,
  "type": "Microsoft.Storage/storageAccounts"
  ...
}

```

Чтобы включить этот параметр, используйте следующую команду:

```azurecli-interactive
az storage account update -g {ResourceGroupName} -n {StorageAccountName} --https-only true
{
  "name": "{StorageAccountName}",
  "enableHttpsTrafficOnly": true,
  "type": "Microsoft.Storage/storageAccounts"
  ...
}

```

## <a name="next-steps"></a>Дальнейшие действия

[Рекомендации по обеспечению безопасности для хранилища BLOB-объектов](../blobs/security-recommendations.md)
