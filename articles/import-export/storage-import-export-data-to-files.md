---
title: Использование службы "Импорт и экспорт Azure" для передачи данных в службу "Файлы Azure" | Документация Майкрософт
description: Сведения о создании заданий импорта на портале Azure для передачи данных в службу "Файлы Azure".
author: alkohli
services: storage
ms.service: storage
ms.topic: how-to
ms.date: 03/03/2021
ms.author: alkohli
ms.subservice: common
ms.custom: devx-track-azurepowershell, devx-track-azurecli, contperf-fy21q3
ms.openlocfilehash: b62c3c4be4fdffd9f509b86d248cd028518ae89a
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102181947"
---
# <a name="use-azure-importexport-service-to-import-data-to-azure-files"></a>Использование службы "Импорт и экспорт Azure" для импорта данных в службу "Файлы Azure"

В этой статье содержатся пошаговые инструкции по использованию службы "Импорт и экспорт Azure" для безопасного импорта больших объемов данных в службу "Файлы Azure". Чтобы служба осуществила импорт данных, нужно отправить в центр обработки данных Azure поддерживаемые диски с данными.

Служба "Импорт и экспорт Azure" поддерживает только импорт файлов Azure в службу хранилища Azure. Экспорт файлов Azure не поддерживается.

## <a name="prerequisites"></a>Предварительные требования

Перед созданием задания импорта для передачи данных в службу "Файлы Azure" внимательно просмотрите и выполните описанные ниже предварительные требования. Необходимо сделать следующее:

- Активная подписка Azure для использования со службой импорта и экспорта.
- Как минимум одна учетная запись хранения Azure. Просмотрите список [поддерживаемых типов хранилища и учетных записей хранения для службы "Импорт и экспорт"](storage-import-export-requirements.md). Сведения о создании учетной записи хранения см. в разделе [Создание учетной записи хранения](../storage/common/storage-account-create.md).
- Соответствующее количество дисков [поддерживаемых типов](storage-import-export-requirements.md#supported-disks).
- Система с ОС Windows [поддерживаемой версии](storage-import-export-requirements.md#supported-operating-systems).
- [Скачать WAImportExport версии 2](https://aka.ms/waiev2) в систему Windows. Распакуйте содержимое в папку по умолчанию: `waimportexport`. Например, `C:\WaImportExport`.
- Учетная запись FedEx или DHL. Если вы хотите использовать перевозчик, отличный от FedEx/DHL, обратитесь в службу поддержки Azure Data Box по адресу `adbops@microsoft.com` .
    - Учетная запись должна быть действительной, иметь баланс и возможности возврата.
    - Создайте номер отслеживания для задания экспорта.
    - Каждое задание должно иметь отдельный номер отслеживания. Несколько заданий с одним и тем же номером отслеживания не поддерживаются.
    - Если у вас нет учетной записи перевозчика, перейдите по адресу:
        - [Создайте учетную запись FedEx](https://www.fedex.com/en-us/create-account.html)или
        - [Создать учетную запись DHL](http://www.dhl-usa.com/en/express/shipping/open_account.html).


## <a name="step-1-prepare-the-drives"></a>Шаг 1. Подготовка дисков

На этом шаге создается файл журнала. В файле журнала хранятся основные сведения, например серийный номер диска, ключ шифрования и сведения об учетной записи хранения.

Чтобы подготовить диски, выполните следующие действия.

1. Подключите наши диски к системе Windows через соединители SATA.
2. Создайте один том NTFS на каждом диске. Присвойте ему букву диска. Не используйте точки подключения.
3. Измените файл *dataset.csv* в корневой папке, в которой находится инструмент. В зависимости от того, что импортируется, — файл, папка или и то и другое, добавьте записи в файл *dataset.csv*, как показано в следующем примере.

   - **Чтобы импортировать файл**: в следующем примере данные для копирования находятся на диске F:. Файл *MyFile1.txt* копируется в корень папки *MyAzureFileshare1*. Если *MyAzureFileshare1* не существует, он создается в учетной записи хранения Azure. Структура папок сохраняется.

       ```
           BasePath,DstItemPathOrPrefix,ItemType,Disposition,MetadataFile,PropertiesFile
           "F:\MyFolder1\MyFile1.txt","MyAzureFileshare1/MyFile1.txt",file,rename,"None",None

       ```
   - **Чтобы импортировать папку**. Все файлы и папки, расположенные в *MyFolder2_original*, будут рекурсивно скопированы в общую папку. Структура папок сохраняется.

       ```
           "F:\MyFolder2\","MyAzureFileshare1/",file,rename,"None",None

       ```
     В одном файле можно сделать несколько записей, соответствующих импортируемым папкам или файлам.

       ```
           "F:\MyFolder1\MyFile1.txt","MyAzureFileshare1/MyFile1.txt",file,rename,"None",None
           "F:\MyFolder2\","MyAzureFileshare1/",file,rename,"None",None

       ```
     Дополнительные сведения о подготовке CSV-файла набора данных см. в [этом разделе](/previous-versions/azure/storage/common/storage-import-export-tool-preparing-hard-drives-import).


4. Измените файл *driveset.csv* в корневой папке, в которой находится инструмент. Добавьте записи в файл *driveset.csv*, как показано в следующих примерах. Файл с набором дисков содержит список дисков и соответствующих букв дисков, чтобы средство могло правильно выбрать список дисков, которые будут подготовлены.

    В этом примере предполагается, что подключено два диска, а базовые тома NTFS G:\ и H:\ уже созданы. H:\ не зашифрован, а G: уже зашифрован. Средство форматирует и шифрует диск, который содержит только H:\ (а не G:\).

   - **Для диска, который не зашифрован:** укажите *Encrypt*, чтобы включить шифрование BitLocker на диске.

       ```
       DriveLetter,FormatOption,SilentOrPromptOnFormat,Encryption,ExistingBitLockerKey
       H,Format,SilentMode,Encrypt,
       ```

   - **Для диска, который уже зашифрован:** укажите *AlreadyEncrypted* и введите ключ BitLocker.

       ```
       DriveLetter,FormatOption,SilentOrPromptOnFormat,Encryption,ExistingBitLockerKey
       G,AlreadyFormatted,SilentMode,AlreadyEncrypted,060456-014509-132033-080300-252615-584177-672089-411631
       ```

     В тот же файл можно внести несколько записей, соответствующих нескольким дискам. Дополнительные сведения о подготовке CSV-файла набора дисков см. в [этом разделе](/previous-versions/azure/storage/common/storage-import-export-tool-preparing-hard-drives-import).

5. Используйте параметр `PrepImport`, чтобы скопировать и подготовить данные для диска. Для первого сеанса копирования каталогов и файлов в рамках нового сеанса копирования выполните следующую команду:

    ```cmd
    .\WAImportExport.exe PrepImport /j:<JournalFile> /id:<SessionId> [/logdir:<LogDirectory>] [/sk:<StorageAccountKey>] [/silentmode] [/InitialDriveSet:<driveset.csv>]/DataSet:<dataset.csv>
    ```

   Ниже приведен пример импорта такого файла.

    ```cmd
    .\WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#1  /sk:************* /InitialDriveSet:driveset.csv /DataSet:dataset.csv /logdir:C:\logs
    ```

6. Файл журнала с именем, указанным с параметром `/j:`, создается при каждом запуске командной строки. Каждый подготовленный диск имеет файл журнала, который должен передаваться при создании задания импорта. Диски без файлов журнала не обрабатываются.

    > [!IMPORTANT]
    > - Не изменяйте данные на дисках или в файлах журналов после завершения подготовки этих дисков.

Дополнительные примеры см. в [примерах для файлов журнала](#samples-for-journal-files).

## <a name="step-2-create-an-import-job"></a>Шаг 2. Создание задания импорта

### <a name="portal"></a>[Портал](#tab/azure-portal)

Выполните следующие действия, чтобы создать задание импорта в портал Azure.
1. Войдите в систему по адресу https://portal.azure.com/.
2. Выполните поиск **заданий импорта и экспорта**.

    ![Поиск по заданиям импорта и экспорта](./media/storage-import-export-data-to-blobs/import-to-blob-1.png)

3. Выберите **+ Создать**.

    ![Выберите Создать, чтобы создать новый ](./media/storage-import-export-data-to-blobs/import-to-blob-2.png)

4. В разделе **Основные сведения**:

   1. Выберите подписку.
   1. Выберите группу ресурсов или щелкните **создать** и создайте новую.
   1. Введите описательное имя для задания импорта. Вы будете использовать его для отслеживания хода выполнения заданий.
       * Имя может содержать только строчные буквы, цифры и дефисы.
       * Имя должно начинаться с буквы и не может содержать пробелы.
   1. Выберите **Импорт в Azure**.

    ![Создание задания импорта — шаг 1](./media/storage-import-export-data-to-blobs/import-to-blob-3.png)

   Чтобы **проследить, нажмите кнопку Далее: сведения о задании >** .

5. В разделе **Сведения о задании** сделайте следующее:

   1. Отправьте файлы журнала, созданные на [шаге 1: подготовка дисков](#step-1-prepare-the-drives).
   1. Выберите целевой регион Azure для заказа.
   1. Выберите учетную запись хранения для импорта.

      Расположение места назначения автоматически заполняется с учетом выбранного региона учетной записи хранения.

   1. Если вы не хотите сохранять подробный журнал, снимите флажок **сохранить подробный журнал в контейнере больших двоичных объектов ' waimportexport '** .


   ![Создание задания импорта — шаг 2](./media/storage-import-export-data-to-blobs/import-to-blob-4.png)

   Нажмите кнопку **Далее: доставка >** для продолжения.

4. В **поставку**:

    1. В раскрывающемся списке выберите перевозчика. Если вы хотите использовать перевозчика, отличный от FedEx/DHL, выберите существующий параметр из раскрывающегося списка. Обратитесь в службу Azure Data Box Operations для `adbops@microsoft.com`  получения сведений о перевозчике, который планируется использовать.
    1. Введите допустимый номер учетной записи, созданный в системе этого перевозчика. Корпорация Майкрософт использует эту учетную запись для отправки дисков обратно после завершения задания импорта.
    1. Укажите полное и допустимое имя контактного лица, телефон, адрес электронной почты, адрес, город, почтовый индекс, страну (провинцию) и страну (регион).

        > [!TIP]
        > Вместо указания адреса электронной почты для отдельного пользователя укажите электронную почту группы. Это гарантирует, что вы получите уведомления, даже если администратор уйдет.

    ![Создание задание импорта – шаг 3](./media/storage-import-export-data-to-blobs/import-to-blob-5.png)

   Щелкните **Просмотр и создание**, чтобы продолжить.

5. В сводке заказа:

   1. Просмотрите **условия**, а затем выберите «я подтверждаю, что все предоставленные сведения верны и согласны с условиями». Затем выполняется проверка.
   1. Просмотрите сведения о задании, предоставленные в сводке. Запишите имя задания и адрес центра обработки данных Azure для отправки дисков обратно в Azure. Эта информация указывается позже на транспортной этикетке.
   1. Нажмите кнопку **создания**.

        ![Создание задания импорта — шаг 4](./media/storage-import-export-data-to-blobs/import-to-blob-6.png)

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы создать задание импорта в Azure CLI, выполните следующие действия.

[!INCLUDE [azure-cli-prepare-your-environment-h3.md](../../includes/azure-cli-prepare-your-environment-h3.md)]

### <a name="create-a-job"></a>Создание задания

1. Чтобы добавить расширение " [AZ Import-Export](/cli/azure/ext/import-export/import-export) ", используйте команду [AZ Extension Add](/cli/azure/extension#az_extension_add) .

    ```azurecli
    az extension add --name import-export
    ```

1. Можно использовать существующую группу ресурсов или создать ее. Чтобы создать группу ресурсов, выполните команду [az group create](/cli/azure/group#az_group_create).

    ```azurecli
    az group create --name myierg --location "West US"
    ```

1. Можно использовать существующую учетную запись хранения или создать ее. Чтобы создать учетную запись хранения, выполните команду [AZ Storage Account Create](/cli/azure/storage/account#az_storage_account_create) :

    ```azurecli
    az storage account create -resource-group myierg -name myssdocsstorage --https-only
    ```

1. Чтобы получить список расположений, в которые можно поставлять диски, используйте команду [AZ Import-Export Location List](/cli/azure/ext/import-export/import-export/location#ext_import_export_az_import_export_location_list) :

    ```azurecli
    az import-export location list
    ```

1. Чтобы получить расположения для вашего региона, используйте команду [AZ Import-Export Location](/cli/azure/ext/import-export/import-export/location#ext_import_export_az_import_export_location_show) .

    ```azurecli
    az import-export location show --location "West US"
    ```

1. Выполните следующую команду [AZ Import-Export Create](/cli/azure/ext/import-export/import-export#ext_import_export_az_import_export_create) , чтобы создать задание импорта:

    ```azurecli
    az import-export create \
        --resource-group myierg \
        --name MyIEjob1 \
        --location "West US" \
        --backup-drive-manifest true \
        --diagnostics-path waimportexport \
        --drive-list bit-locker-key=439675-460165-128202-905124-487224-524332-851649-442187 \
            drive-header-hash= drive-id=AZ31BGB1 manifest-file=\\DriveManifest.xml \
            manifest-hash=69512026C1E8D4401816A2E5B8D7420D \
        --type Import \
        --log-level Verbose \
        --shipping-information recipient-name="Microsoft Azure Import/Export Service" \
            street-address1="3020 Coronado" city="Santa Clara" state-or-province=CA postal-code=98054 \
            country-or-region=USA phone=4083527600 \
        --return-address recipient-name="Gus Poland" street-address1="1020 Enterprise way" \
            city=Sunnyvale country-or-region=USA state-or-province=CA postal-code=94089 \
            email=gus@contoso.com phone=4085555555" \
        --return-shipping carrier-name=FedEx carrier-account-number=123456789 \
        --storage-account myssdocsstorage
    ```

   > [!TIP]
   > Вместо указания адреса электронной почты для отдельного пользователя укажите электронную почту группы. Это гарантирует, что вы получите уведомления, даже если администратор уйдет.


1. Используйте команду [AZ Import-Export List](/cli/azure/ext/import-export/import-export#ext_import_export_az_import_export_list) , чтобы просмотреть все задания для группы ресурсов миерг:

    ```azurecli
    az import-export list --resource-group myierg
    ```

1. Чтобы обновить задание или отменить задание, выполните команду [AZ Import-Export Update](/cli/azure/ext/import-export/import-export#ext_import_export_az_import_export_update) :

    ```azurecli
    az import-export update --resource-group myierg --name MyIEjob1 --cancel-requested true
    ```

### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

Чтобы создать задание импорта в Azure PowerShell, выполните следующие действия.

[!INCLUDE [azure-powershell-requirements-h3.md](../../includes/azure-powershell-requirements-h3.md)]

> [!IMPORTANT]
> Пока модуль PowerShell **AZ. ImportExport** находится на этапе предварительной версии, его необходимо установить отдельно с помощью `Install-Module` командлета. Как только этот модуль PowerShell станет общедоступным, он будет включен в один из будущих выпусков модуля Az PowerShell и по умолчанию встроен в Azure Cloud Shell.

```azurepowershell-interactive
Install-Module -Name Az.ImportExport
```

### <a name="create-a-job"></a>Создание задания

1. Можно использовать существующую группу ресурсов или создать ее. Чтобы создать группу ресурсов, выполните командлет [New-азресаурцеграуп](/powershell/module/az.resources/new-azresourcegroup) :

   ```azurepowershell-interactive
   New-AzResourceGroup -Name myierg -Location westus
   ```

1. Можно использовать существующую учетную запись хранения или создать ее. Чтобы создать учетную запись хранения, выполните командлет [New-азсторажеаккаунт](/powershell/module/az.storage/new-azstorageaccount) :

   ```azurepowershell-interactive
   New-AzStorageAccount -ResourceGroupName myierg -AccountName myssdocsstorage -SkuName Standard_RAGRS -Location westus -EnableHttpsTrafficOnly $true
   ```

1. Чтобы получить список расположений, в которые можно поставлять диски, используйте командлет [Get-азимпортекспортлокатион](/powershell/module/az.importexport/get-azimportexportlocation) :

   ```azurepowershell-interactive
   Get-AzImportExportLocation
   ```

1. Используйте `Get-AzImportExportLocation` командлет с `Name` параметром, чтобы получить расположения для вашего региона:

   ```azurepowershell-interactive
   Get-AzImportExportLocation -Name westus
   ```

1. Выполните следующий пример [New-азимпортекспорт](/powershell/module/az.importexport/new-azimportexport) , чтобы создать задание импорта:

   ```azurepowershell-interactive
   $driveList = @(@{
     DriveId = '9CA995BA'
     BitLockerKey = '439675-460165-128202-905124-487224-524332-851649-442187'
     ManifestFile = '\\DriveManifest.xml'
     ManifestHash = '69512026C1E8D4401816A2E5B8D7420D'
     DriveHeaderHash = 'AZ31BGB1'
   })

   $Params = @{
      ResourceGroupName = 'myierg'
      Name = 'MyIEjob1'
      Location = 'westus'
      BackupDriveManifest = $true
      DiagnosticsPath = 'waimportexport'
      DriveList = $driveList
      JobType = 'Import'
      LogLevel = 'Verbose'
      ShippingInformationRecipientName = 'Microsoft Azure Import/Export Service'
      ShippingInformationStreetAddress1 = '3020 Coronado'
      ShippingInformationCity = 'Santa Clara'
      ShippingInformationStateOrProvince = 'CA'
      ShippingInformationPostalCode = '98054'
      ShippingInformationCountryOrRegion = 'USA'
      ShippingInformationPhone = '4083527600'
      ReturnAddressRecipientName = 'Gus Poland'
      ReturnAddressStreetAddress1 = '1020 Enterprise way'
      ReturnAddressCity = 'Sunnyvale'
      ReturnAddressStateOrProvince = 'CA'
      ReturnAddressPostalCode = '94089'
      ReturnAddressCountryOrRegion = 'USA'
      ReturnAddressPhone = '4085555555'
      ReturnAddressEmail = 'gus@contoso.com'
      ReturnShippingCarrierName = 'FedEx'
      ReturnShippingCarrierAccountNumber = '123456789'
      StorageAccountId = '/subscriptions/<SubscriptionId>/resourceGroups/myierg/providers/Microsoft.Storage/storageAccounts/myssdocsstorage'
   }
   New-AzImportExport @Params
   ```

   > [!TIP]
   > Вместо указания адреса электронной почты для отдельного пользователя укажите электронную почту группы. Это гарантирует, что вы получите уведомления, даже если администратор уйдет.

1. Используйте командлет [Get-азимпортекспорт](/powershell/module/az.importexport/get-azimportexport) , чтобы просмотреть все задания для группы ресурсов миерг:

   ```azurepowershell-interactive
   Get-AzImportExport -ResourceGroupName myierg
   ```

1. Чтобы обновить задание или отменить задание, выполните командлет [Update-азимпортекспорт](/powershell/module/az.importexport/update-azimportexport) :

   ```azurepowershell-interactive
   Update-AzImportExport -Name MyIEjob1 -ResourceGroupName myierg -CancelRequested
   ```

---

## <a name="step-3-ship-the-drives-to-the-azure-datacenter"></a>Шаг 3. Отправка дисков в центр обработки данных Azure

[!INCLUDE [storage-import-export-ship-drives](../../includes/storage-import-export-ship-drives.md)]

## <a name="step-4-update-the-job-with-tracking-information"></a>Шаг 4. Указание данных об отслеживании для задания

[!INCLUDE [storage-import-export-update-job-tracking](../../includes/storage-import-export-update-job-tracking.md)]

## <a name="step-5-verify-data-upload-to-azure"></a>Шаг 5: Проверка передачи данных в Azure

Отслеживание задания до завершения. После завершения задания убедитесь, что данные переданы в Azure. Удалите локальные данные только после того, как вы убедитесь, что отправка прошла успешно.

## <a name="samples-for-journal-files"></a>Примеры файлов журнала

Чтобы **добавить дополнительные диски**, создайте файл с набором дисков и выполните приведенную ниже команду.

Для последующих сеансов копирования на диски, отличные от указанных в файле *InitialDriveset. csv* , укажите новый файл driveing *. csv* и укажите его в качестве значения для параметра `AdditionalDriveSet` . Используйте **то же имя файла журнала** и укажите **новый идентификатор сеанса**. Формат CSV-файлов AdditionalDriveset и InitialDriveSet совпадает.

```cmd
WAImportExport.exe PrepImport /j:<JournalFile> /id:<SessionId> /AdditionalDriveSet:<driveset.csv>
```

Ниже приведен пример импорта такого файла.

```cmd
WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#3  /AdditionalDriveSet:driveset-2.csv
```


Чтобы добавить дополнительные данные в тот же файл с набором дисков, используйте команду PrepImport для последующих сеансов копирования и скопируйте дополнительные файлы или папки.

Для последующих сеансов копирования на те же жесткие диски, указанные в файле *InitialDriveset.csv*, укажите имя **того же файла журнала** и **новый идентификатор сеанса**. Указывать ключ учетной записи хранения не требуется.

```cmd
WAImportExport PrepImport /j:<JournalFile> /id:<SessionId> /j:<JournalFile> /id:<SessionId> [/logdir:<LogDirectory>] DataSet:<dataset.csv>
```

Ниже приведен пример импорта такого файла.

```cmd
WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#2  /DataSet:dataset-2.csv
```

## <a name="next-steps"></a>Дальнейшие действия

* [Просмотр состояния задания и диска](storage-import-export-view-drive-status.md)
* [Сведения о требованиях службы "Импорт и экспорт"](storage-import-export-requirements.md)