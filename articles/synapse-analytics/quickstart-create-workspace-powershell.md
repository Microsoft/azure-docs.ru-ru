---
title: Краткое руководство. Создание рабочей области Synapse с помощью Azure PowerShell
description: Создайте рабочую область Azure Synapse с помощью Azure PowerShell, выполнив действия, описанные в этом руководстве.
services: synapse-analytics
author: alehall
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: overview
ms.date: 10/19/2020
ms.author: alehall
ms.reviewer: jrasnick
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: a19d1f26616697e15ae4c39a63c44bdaf83f78f9
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101675771"
---
# <a name="quickstart-create-an-azure-synapse-workspace-with-azure-powershell"></a>Краткое руководство. Создание рабочей области Azure Synapse с помощью Azure PowerShell

Azure PowerShell — это набор командлетов для управления ресурсами Azure непосредственно из PowerShell. Вы можете использовать его в браузере с Azure Cloud Shell. Его также можно установить на macOS, Linux или Windows.

В этом кратком руководстве вы узнаете, как создать рабочую область Synapse с помощью Azure PowerShell.

Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

- [Учетная запись хранения Azure Data Lake Storage 2-го поколения](../storage/common/storage-account-create.md)

    > [!IMPORTANT]
    > В рабочей области Azure Synapse должна быть возможность считывать данные в выбранной учетной записи ADLS 2-го поколения и выполнять запись в нее. Для любой учетной записи хранения, связываемой в качестве основной учетной записи хранения, при создании учетной записи хранения необходимо включить **иерархическое пространство имен**, как описано на странице [Создание учетной записи хранения](../storage/common/storage-account-create.md?tabs=azure-powershell#create-a-storage-account).

Если вы решили использовать Cloud Shell, дополнительные сведения см. в статье [Обзор Azure Cloud Shell](../cloud-shell/overview.md).

### <a name="install-the-azure-powershell-module-locally"></a>Локальная установка модуля Azure PowerShell

Если вы решили использовать PowerShell локально, для работы с этой статьей установите модуль PowerShell Az и подключитесь к учетной записи Azure с помощью командлета [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount). См. сведения об [установке модуля Azure PowerShell](/powershell/azure/install-az-ps).

Дополнительные сведения о проверке подлинности с помощью Azure PowerShell см. в статье [Вход с помощью Azure PowerShell](/powershell/azure/authenticate-azureps).

### <a name="install-the-azure-synapse-powershell-module"></a>Установка модуля Azure Synapse PowerShell

> [!IMPORTANT]
> Так как модуль **Az.Synapse** в PowerShell предоставляется в режиме предварительной версии, его нужно установить отдельно с помощью командлета `Install-Module`. Как только этот модуль PowerShell станет общедоступным, он будет включен в один из будущих выпусков модуля Az PowerShell и по умолчанию встроен в Azure Cloud Shell.

```azurepowershell-interactive
Install-Module -Name Az.Synapse
```

## <a name="create-an-azure-synapse-workspace-using-azure-powershell"></a>Создание рабочей области Azure Synapse с помощью Azure PowerShell

1. Задайте необходимые переменные среды, чтобы создать ресурсы для рабочей области Azure Synapse.

   |        Имя переменной        |                                                 Описание                                                 |
   | --------------------------- | ----------------------------------------------------------------------------------------------------------- |
   | Имя_учетной_записи_хранения          | Имя существующей учетной записи хранения ADLS 2-го поколения.                                                           |
   | StorageAccountResourceGroup | Имя существующей группы ресурсов учетной записи хранения ADLS 2-го поколения.                                             |
   | FileShareName               | Имя существующей файловой системы хранилища.                                                                  |
   | SynapseResourceGroup        | Выберите новое имя для группы ресурсов Azure Synapse.                                                    |
   | Регион                      | Выберите один из [регионов Azure](https://azure.microsoft.com/global-infrastructure/geographies/#overview). |
   | SynapseWorkspaceName        | Выберите уникальное имя для новой рабочей области Azure Synapse.                                                  |
   | SqlUser                     | Выберите значение для нового имени пользователя.                                                                          |
   | SqlPassword                 | Выберите надежный пароль.                                                                                   |
   | ClientIP                    | Общедоступный IP-адрес системы, из которой вы используете PowerShell.                                             |
   |                             |                                                                                                             |

1. Создайте группу ресурсов в форме контейнера для рабочей области Azure Synapse:

   ```azurepowershell-interactive
   New-AzResourceGroup -Name $SynapseResourceGroup -Location $Region
   ```

1. Создайте рабочую область Azure Synapse:

   ```azurepowershell-interactive
   $Cred = New-Object -TypeName System.Management.Automation.PSCredential ($SqlUser, (ConvertTo-SecureString $SqlPassword -AsPlainText -Force))

   $WorkspaceParams = @{
     Name = $SynapseWorkspaceName
     ResourceGroupName = $SynapseResourceGroup
     DefaultDataLakeStorageAccountName = $StorageAccountName
     DefaultDataLakeStorageFilesystem = $FileShareName
     SqlAdministratorLoginCredential = $Cred
     Location = $Region
   }
   New-AzSynapseWorkspace @WorkspaceParams
   ```

1. Получите URL-адрес Web и Dev для рабочей области Azure Synapse:

   ```azurepowershell-interactive
   $WorkspaceWeb = (Get-AzSynapseWorkspace -Name $SynapseWorkspaceName -ResourceGroupName $StorageAccountResourceGroup).ConnectivityEndpoints.Web
   $WorkspaceDev = (Get-AzSynapseWorkspace -Name $SynapseWorkspaceName -ResourceGroupName $StorageAccountResourceGroup).ConnectivityEndpoints.Dev
   ```

1. Создайте правило брандмауэра, которое разрешит доступ к рабочей области Azure Synapse с компьютера:

   ```azurepowershell-interactive
   $FirewallParams = @{
     WorkspaceName = $SynapseWorkspaceName
     Name = 'Allow Client IP'
     ResourceGroupName = $StorageAccountResourceGroup
     StartIpAddress = $ClientIP
     EndIpAddress = $ClientIP
   }
   New-AzSynapseFirewallRule @FirewallParams
   ```

1. Чтобы получить доступ к рабочей области, откройте URL-адрес Web рабочей области Azure Synapse, который хранится в переменной среды`WorkspaceWeb`:

   ```azurepowershell-interactive
   Start-Process $WorkspaceWeb
   ```

   ![web для рабочей области Azure Synapse](media/quickstart-create-synapse-workspace-powershell/create-workspace-powershell-1.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

Выполните действия ниже, чтобы удалить рабочую область Azure Synapse.

> [!WARNING]
> Удаление рабочей области Azure Synapse приведет к удалению модулей аналитики и данных, хранящихся в базе данных автономных пулов SQL и метаданных рабочей области. В этом случае вы больше не сможете подключиться к конечным точкам SQL и Apache Spark. Все артефакты кода будут удалены (запросы, записные книжки, определения заданий и конвейеры). Удаление рабочей области **не** влияет на данные в Azure Data Lake Store 2-го поколения, связанные с рабочей областью.

Если созданная при работе с этой статьей рабочая область Azure Synapse больше не нужна, ее можно удалить с помощью команды из следующего примера.

```azurepowershell-interactive
Remove-AzSynapseWorkspace -Name $SynapseWorkspaceNam -ResourceGroupName $SynapseResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

Теперь можно создать пулы [SQL](quickstart-create-sql-pool-studio.md) или [Apache Spark](quickstart-create-apache-spark-pool-studio.md), чтобы начать анализ и изучение данных.