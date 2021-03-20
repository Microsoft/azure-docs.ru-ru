---
title: Приступая к работе с Azure Data Lake Storage 1-го поколения PowerShell | Документация Майкрософт
description: Используйте Azure PowerShell для создания учетной записи Azure Data Lake Storage 1-го поколения и выполнения основных операций.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 06/27/2018
ms.author: twooley
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 485b14f85d68290c5447c885b9bc4974318f7952
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92103736"
---
# <a name="get-started-with-azure-data-lake-storage-gen1-using-azure-powershell"></a>Начало работы с Azure Data Lake Storage Gen1 с помощью Azure PowerShell

> [!div class="op_single_selector"]
> * [Портал](data-lake-store-get-started-portal.md)
> * [PowerShell](data-lake-store-get-started-powershell.md)
> * [Azure CLI](data-lake-store-get-started-cli-2.0.md)
>
>

[!INCLUDE [data-lake-storage-gen1-rename-note.md](../../includes/data-lake-storage-gen1-rename-note.md)]

Узнайте, как использовать Azure PowerShell для создания учетной записи Azure Data Lake Storage 1-го поколения и выполнения основных операций, таких как создание папок, отправка и скачивание файлов данных, удаление учетной записи и т. д. Дополнительные сведения о Data Lake Storage 1-го поколения см. в разделе [обзор Data Lake Storage 1-го поколения](data-lake-store-overview.md).

## <a name="prerequisites"></a>Предварительные условия

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

* **Подписка Azure**. См. страницу [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).
* **Azure PowerShell 1,0 или более поздней** версии. Ознакомьтесь со статьей [Установка и настройка Azure PowerShell](/powershell/azure/).

## <a name="authentication"></a>Аутентификация

В этой статье используется упрощенный подход к проверке подлинности с Data Lake Storage 1-го поколения, где предлагается ввести учетные данные учетной записи Azure. Уровень доступа к учетной записи Data Lake Storage 1-го поколения и файловой системе зависит от уровня доступа пользователя, который вошел в систему. Однако существуют и другие подходы к проверке подлинности с помощью Data Lake Storage 1-го поколения, которые являются проверкой подлинности пользователей или проверкой подлинности между службами. Инструкции и дополнительные сведения об аутентификации см. в разделах [Аутентификация пользователей](data-lake-store-end-user-authenticate-using-active-directory.md) и [Аутентификация между службами](./data-lake-store-service-to-service-authenticate-using-active-directory.md).

## <a name="create-a-data-lake-storage-gen1-account"></a>Создание учетной записи Data Lake Storage 1-го поколения

1. На рабочем столе откройте новое окно Windows PowerShell. Введите следующий фрагмент кода, чтобы войти в свою учетную запись Azure, выбрать подписку и зарегистрировать поставщик Data Lake Storage 1-го поколения. Когда вам будет предложено войти, введите учетные данные администратора или владельца подписки.

    ```PowerShell
    # Log in to your Azure account
    Connect-AzAccount

    # List all the subscriptions associated to your account
    Get-AzSubscription

    # Select a subscription
    Set-AzContext -SubscriptionId <subscription ID>

    # Register for Azure Data Lake Storage Gen1
    Register-AzResourceProvider -ProviderNamespace "Microsoft.DataLakeStore"
    ```

1. Учетная запись Data Lake Storage 1-го поколения связывается с группой ресурсов Azure. Для начала создайте группу ресурсов.

    ```PowerShell
    $resourceGroupName = "<your new resource group name>"
    New-AzResourceGroup -Name $resourceGroupName -Location "East US 2"
    ```

    ![Создание группы ресурсов Azure](./media/data-lake-store-get-started-powershell/ADL.PS.CreateResourceGroup.png "Создание группы ресурсов Azure")

1. Создайте учетную запись Data Lake Storage 1-го поколения. Указанное имя должно содержать только строчные буквы и цифры.

    ```PowerShell
    $dataLakeStorageGen1Name = "<your new Data Lake Storage Gen1 account name>"
    New-AzDataLakeStoreAccount -ResourceGroupName $resourceGroupName -Name $dataLakeStorageGen1Name -Location "East US 2"
    ```

    ![Создание учетной записи Data Lake Storage 1-го поколения](./media/data-lake-store-get-started-powershell/ADL.PS.CreateADLAcc.png "Создание учетной записи Data Lake Storage 1-го поколения")

1. Убедитесь, что учетная запись создана.

    ```PowerShell
    Test-AzDataLakeStoreAccount -Name $dataLakeStorageGen1Name
    ```

    Результат должен иметь значение **True**.

## <a name="create-directory-structures"></a>Создание структур каталогов

Чтобы хранить данные и управлять ими, вы можете создать каталоги в своей учетной записи Data Lake Storage 1-го поколения.

1. Укажите корневой каталог.

    ```PowerShell
    $myrootdir = "/"
    ```

1. Создайте новый каталог с именем **mynewdirectory** в указанном корневом каталоге.

    ```PowerShell
    New-AzDataLakeStoreItem -Folder -AccountName $dataLakeStorageGen1Name -Path $myrootdir/mynewdirectory
    ```

1. Убедитесь, что новый каталог создан.

    ```PowerShell
    Get-AzDataLakeStoreChildItem -AccountName $dataLakeStorageGen1Name -Path $myrootdir
    ```

    Отобразятся выходные данные, как показано на следующем снимке экрана:

    ![Проверка каталога](./media/data-lake-store-get-started-powershell/ADL.PS.Verify.Dir.Creation.png "Проверка каталога")

## <a name="upload-data"></a>Отправка данных

Данные можно передавать в Data Lake Storage 1-го поколения непосредственно на корневой уровень или в каталог, созданный в учетной записи. Фрагменты кода в этом разделе показывают, как отправить некоторые образцы данных в каталог (**mynewdirectory**), который был создан на предыдущем шаге.

Если у вас нет под рукой подходящих для этих целей данных, передайте папку **Ambulance Data** из [репозитория Git для озера данных Azure](https://github.com/MicrosoftBigData/usql/tree/master/Examples/Samples/Data/AmbulanceData). Загрузите файл и сохраните его в локальном каталоге на компьютере, например в расположении C:\sampledata\.

```PowerShell
Import-AzDataLakeStoreItem -AccountName $dataLakeStorageGen1Name `
   -Path "C:\sampledata\vehicle1_09142014.csv" `
   -Destination $myrootdir\mynewdirectory\vehicle1_09142014.csv
```

## <a name="rename-download-and-delete-data"></a>Переименование, скачивание и удаление данных

Чтобы переименовать файл, используйте следующую команду:

```PowerShell
Move-AzDataLakeStoreItem -AccountName $dataLakeStorageGen1Name `
    -Path $myrootdir\mynewdirectory\vehicle1_09142014.csv `
    -Destination $myrootdir\mynewdirectory\vehicle1_09142014_Copy.csv
```

Чтобы загрузить файл, используйте следующую команду:

```PowerShell
Export-AzDataLakeStoreItem -AccountName $dataLakeStorageGen1Name `
    -Path $myrootdir\mynewdirectory\vehicle1_09142014_Copy.csv `
    -Destination "C:\sampledata\vehicle1_09142014_Copy.csv"
```

Чтобы удалить файл, используйте следующую команду:

```PowerShell
Remove-AzDataLakeStoreItem -AccountName $dataLakeStorageGen1Name `
    -Paths $myrootdir\mynewdirectory\vehicle1_09142014_Copy.csv
```

При появлении запроса введите **Y** , чтобы удалить элемент. Если нужно удалить несколько файлов, можно указать все пути через запятую.

```PowerShell
Remove-AzDataLakeStoreItem -AccountName $dataLakeStorageGen1Name `
    -Paths $myrootdir\mynewdirectory\vehicle1_09142014.csv, $myrootdir\mynewdirectoryvehicle1_09142014_Copy.csv
```

## <a name="delete-your-account"></a>Удалить учетную запись

Чтобы удалить учетную запись Data Lake Storage 1-го поколения, используйте следующую команду.

```PowerShell
Remove-AzDataLakeStoreAccount -Name $dataLakeStorageGen1Name
```

При появлении запроса введите **Y** , чтобы удалить учетную запись.

## <a name="next-steps"></a>Дальнейшие действия

* [Рекомендации по настройке производительности для использования PowerShell с Azure Data Lake Storage 1-го поколения](data-lake-store-performance-tuning-powershell.md)
* [Использование Azure Data Lake Storage 1-го поколения для обеспечения соответствия требованиям больших данных](data-lake-store-data-scenarios.md)
* [Защита данных в Data Lake Storage Gen1](data-lake-store-secure-data.md)
* [Начало работы с Azure Data Lake Analytics с помощью портала Azure](../data-lake-analytics/data-lake-analytics-get-started-portal.md)
* [Создание кластеров HDInsight, использующих Data Lake Store, с помощью портала Azure](data-lake-store-hdinsight-hadoop-use-portal.md)