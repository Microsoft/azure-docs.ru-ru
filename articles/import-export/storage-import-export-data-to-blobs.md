---
title: Использование службы "Импорт и экспорт Azure" для передачи данных в большие двоичные объекты Azure | Документация Майкрософт
description: Сведения о создании заданий импорта и экспорта на портале Azure для передачи данных в большие двоичные объекты Azure и из них.
author: alkohli
services: storage
ms.service: storage
ms.topic: how-to
ms.date: 03/03/2021
ms.author: alkohli
ms.subservice: common
ms.custom: devx-track-azurepowershell, devx-track-azurecli, contperf-fy21q3
ms.openlocfilehash: 77a1c02c1ec59778521104e57f3bf3de8e52fa44
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102177420"
---
# <a name="use-the-azure-importexport-service-to-import-data-to-azure-blob-storage"></a>Использование службы "Импорт и экспорт Azure" для импорта данных в хранилище BLOB-объектов Azure

Эта статья содержит пошаговые инструкции по использованию службы "Импорт и экспорт Azure" для безопасного импорта больших объемов данных в хранилище BLOB-объектов Azure. Чтобы служба осуществила импорт данных в большие двоичные объекты Azure, нужно отправить в центр обработки данных Azure зашифрованные диски с данными.

## <a name="prerequisites"></a>Предварительные требования

Прежде чем создавать задание импорта для передачи данных в хранилище BLOB-объектов Azure, внимательно просмотрите и выполните описанные ниже предварительные требования для этой службы.
Необходимо сделать следующее:

* Активная подписка Azure, которую можно использовать для службы "Импорт и экспорт".
* Хотя бы одна учетная запись хранения Azure с контейнером хранилища. Просмотрите список [поддерживаемых типов хранилища и учетных записей хранения для службы "Импорт и экспорт"](storage-import-export-requirements.md).
  * Сведения о создании учетной записи хранения см. в разделе [Создание учетной записи хранения](../storage/common/storage-account-create.md).
  * Сведения о контейнере хранилища см. в статье [о создании контейнера хранилища](../storage/blobs/storage-quickstart-blobs-portal.md#create-a-container).
* Соответствующее количество дисков [поддерживаемых типов](storage-import-export-requirements.md#supported-disks).
* Система с ОС Windows [поддерживаемой версии](storage-import-export-requirements.md#supported-operating-systems).
* Включить технологию BitLocker в системе Windows. Дополнительные сведения о включении BitLocker см. в [этой статье](https://thesolving.com/storage/how-to-enable-bitlocker-on-windows-server-2012-r2/).
* [Скачайте последнюю версию WAImportExport версии 1](https://www.microsoft.com/download/details.aspx?id=42659) в системе Windows. Последняя версия средства содержит обновления для системы безопасности, позволяющие получить внешний предохранитель ключа BitLocker и обновленную функцию режима разблокировки.

  * Распакуйте содержимое в папку по умолчанию: `waimportexportv1`. Например, `C:\WaImportExportV1`.
* Учетная запись FedEx или DHL. Если вы хотите использовать перевозчик, отличный от FedEx/DHL, обратитесь в службу поддержки Azure Data Box по адресу `adbops@microsoft.com` .
  * Учетная запись должна быть действительной, иметь баланс и возможности возврата.
  * Создайте номер отслеживания для задания экспорта.
  * Каждое задание должно иметь отдельный номер отслеживания. Несколько заданий с одним и тем же номером отслеживания не поддерживаются.
  * Если у вас нет учетной записи оператора, перейдите по ссылке:
    * [Создайте учетную запись FedEx](https://www.fedex.com/en-us/create-account.html)или
    * [Создать учетную запись DHL](http://www.dhl-usa.com/en/express/shipping/open_account.html).

## <a name="step-1-prepare-the-drives"></a>Шаг 1. Подготовка дисков

На этом шаге создается файл журнала. В файле журнала хранятся основные сведения, например серийный номер диска, ключ шифрования и сведения об учетной записи хранения.

Выполните следующие действия для подготовки дисков.

1. Подключите свои диски к системе Windows с использованием соединителей SATA.
2. Создайте один том NTFS на каждом диске. Присвойте ему букву диска. Не используйте точки подключения.
3. Включите шифрование BitLocker в томе NTFS. Если используется система Windows Server, используйте инструкции из статьи [о включении BitLocker в Windows Server 2012 R2](https://thesolving.com/storage/how-to-enable-bitlocker-on-windows-server-2012-r2/).
4. Скопируйте данные в зашифрованный том. Используйте перетаскивание, программу Robocopy или аналогичное средство копирования. Файл журнала (*. jrn*) создается в той же папке, где запускается средство.

   Если диск заблокирован и необходимо разблокировать диск, то действия по разблокировке могут отличаться в зависимости от варианта использования.

   * Если вы добавили данные на предварительно зашифрованный диск (средство WAImportExport не использовалось для шифрования), используйте ключ BitLocker (указанный числовой пароль) во всплывающем окне, чтобы разблокировать диск.

   * Если вы добавили данные на диск, который был зашифрован с помощью средства WAImportExport, используйте следующую команду для разблокировки диска:

        `WAImportExport Unlock /externalKey:<BitLocker key (base 64 string) copied from journal (*.jrn*) file>`

5. Откройте PowerShell или окно командной строки с правами администратора. Чтобы перейти в распакованную папку, выполните следующую команду:

    `cd C:\WaImportExportV1`
6. Чтобы получить ключ BitLocker диска, выполните следующую команду:

    `manage-bde -protectors -get <DriveLetter>:`
7. Чтобы подготовить диск, выполните следующую команду. **В зависимости от размера данных подготовка диска может занять несколько часов.**

    ```powershell
    ./WAImportExport.exe PrepImport /j:<journal file name> /id:session<session number> /t:<Drive letter> /bk:<BitLocker key> /srcdir:<Drive letter>:\ /dstdir:<Container name>/ /blobtype:<BlockBlob or PageBlob> /skipwrite
    ```

    Файл журнала создается в той же папке, где вы запустили средство. Также создаются два других файла: файл *XML* (папка, в которой выполняется средство) и файл *drive-manifest.xml* (папка, в которой хранятся данные).

    В следующей таблице описаны параметры, используемые в этой команде.

    |Параметр  |Описание  |
    |---------|---------|
    |/j:     |Имя файла журнала с расширением .jrn. Файл журнала создается для каждого диска. Рекомендуем использовать в качестве имени файла журнала серийный номер диска.         |
    |/id:     |Идентификатор сеанса. Используйте уникальный номер сеанса для каждого экземпляра команды.      |
    |/t:     |Буква отправляемого диска. Например, диск `D`.         |
    |/bk:     |Ключ BitLocker для диска. Числовой пароль из выходных данных `manage-bde -protectors -get D:`      |
    |/srcdir:     |Буква отправляемого диска и `:\`. Например, `D:\`.         |
    |/dstdir:     |Имя целевого контейнера в службе хранилища Azure.         |
    |/blobtype     |Этот параметр указывает тип больших двоичных объектов, в которые необходимо импортировать данные. Для блочных BLOB-объектов тип большого двоичного объекта —, `BlockBlob` а для страничных BLOB-объектов — `PageBlob` .         |
    |/skipwrite:     | Указывает, что новые данные для копирования и данные на диске должны быть подготовлены.          |
    |/enablecontentmd5:     |Параметр при включении гарантирует, что MD5 будет вычисляться и устанавливаться как `Content-md5` свойство для каждого большого двоичного объекта. Используйте этот параметр, только если вы хотите использовать `Content-md5` поле после передачи данных в Azure. <br> Этот параметр не влияет на проверку целостности данных (которая происходит по умолчанию). Этот параметр увеличивает время, затрачиваемое на загрузку данных в облако.          |
8. Повторите предыдущий шаг для каждого диска, который необходимо отправить. Файл журнала с предоставленным именем создается при каждом запуске командной строки.

    > [!IMPORTANT]
    > * Вместе с файлом журнала в той же папке, где находится средство, также создается файл `<Journal file name>_DriveInfo_<Drive serial ID>.xml`. XML-файл используется вместо файла журнала при создании задания, если файл журнала имеет слишком большой размер.
   > * Максимальный размер файла журнала, разрешенный порталом, — 2 МБ. Если размер файла журнала превышает этот предел, возвращается ошибка.

## <a name="step-2-create-an-import-job"></a>Шаг 2. Создание задания импорта

### <a name="portal"></a>[Портал](#tab/azure-portal)

Чтобы создать задание импорта на портале Azure, выполните следующие шаги.

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

   1. Отправьте файлы журнала, созданные на [шаге 1: подготовка дисков](#step-1-prepare-the-drives). В случае использования `waimportexport.exe version1` необходимо передать один файл для каждого подготовленного диска. Если размер файла журнала превышает 2 МБ, то можно использовать `<Journal file name>_DriveInfo_<Drive serial ID>.xml`, который также создан с использованием файла журнала.
   1. Выберите целевой регион Azure для заказа.
   1. Выберите учетную запись хранения для импорта.
      
      Расположение места назначения автоматически заполняется с учетом выбранного региона учетной записи хранения.
   1. Если вы не хотите сохранять подробный журнал, снимите флажок **сохранить подробный журнал в контейнере больших двоичных объектов ' waimportexport '** .

   ![Создание задания импорта — шаг 2](./media/storage-import-export-data-to-blobs/import-to-blob-4.png).

   Нажмите кнопку **Далее: доставка >** для продолжения.

6. В **поставку**:

   1. В раскрывающемся списке выберите перевозчика. Если вы хотите использовать перевозчика, отличный от FedEx/DHL, выберите существующий параметр из раскрывающегося списка. Свяжитесь с командой Operations Azure Data Box `adbops@microsoft.com`  с информацией о перевозчике, который планируется использовать.
   1. Введите допустимый номер учетной записи, созданный в системе этого перевозчика. Корпорация Майкрософт использует эту учетную запись для отправки дисков обратно после завершения задания импорта. Если у вас нет номера учетной записи, создайте учетную запись перевозчика [FedEx](https://www.fedex.com/us/oadr/) или [DHL](https://www.dhl.com/).
   1.  Укажите полное и допустимое имя контактного лица, телефон, адрес электронной почты, адрес, город, почтовый индекс, страну (провинцию) и страну (регион).

       > [!TIP]
       > Вместо указания адреса электронной почты для отдельного пользователя укажите электронную почту группы. Это гарантирует, что вы получите уведомления, даже если администратор уйдет.

   ![Создание задание импорта – шаг 3](./media/storage-import-export-data-to-blobs/import-to-blob-5.png)

   Щелкните **Просмотр и создание**, чтобы продолжить.

7. В сводке заказа:

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
    az storage account create --resource-group myierg --name myssdocsstorage --https-only
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

## <a name="step-3-optional-configure-customer-managed-key"></a>Шаг 3 (необязательно): Настройка управляемого клиентом ключа

Пропустите этот шаг и перейдите к следующему шагу, если хотите использовать управляемый корпорацией Майкрософт ключ для защиты ключей BitLocker для дисков. Чтобы настроить собственный ключ для защиты ключа BitLocker, следуйте инструкциям в статье [Настройка управляемых клиентом ключей с помощью Azure Key Vault для импорта и экспорта Azure в портал Azure](storage-import-export-encryption-key-portal.md).

## <a name="step-4-ship-the-drives"></a>Шаг 4. Отправка дисков

[!INCLUDE [storage-import-export-ship-drives](../../includes/storage-import-export-ship-drives.md)]

## <a name="step-5-update-the-job-with-tracking-information"></a>Шаг 5. обновление задания с данными отслеживания

[!INCLUDE [storage-import-export-update-job-tracking](../../includes/storage-import-export-update-job-tracking.md)]

## <a name="step-6-verify-data-upload-to-azure"></a>Шаг 6. Проверка отправки данных в Azure

Отслеживание задания до завершения. После завершения задания убедитесь, что данные переданы в Azure. Удалите локальные данные только после подтверждения, что загрузка прошла успешно.

## <a name="next-steps"></a>Дальнейшие действия

* [Просмотр состояния задания и диска](storage-import-export-view-drive-status.md)
* [Сведения о требованиях службы "Импорт и экспорт"](storage-import-export-requirements.md)
