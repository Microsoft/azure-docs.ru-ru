---
title: Использование службы "Импорт и экспорт Azure" для экспорта данных из больших двоичных объектов Azure | Документация Майкрософт
description: Сведения о создании заданий экспорта на портале Azure для передачи данных из больших двоичных объектов Azure.
author: alkohli
services: storage
ms.service: storage
ms.topic: how-to
ms.date: 03/03/2021
ms.author: alkohli
ms.subservice: common
ms.custom: devx-track-azurepowershell, devx-track-azurecli, contperf-fy21q3
ms.openlocfilehash: e878be5351362923e163c0a6f617b96ab72a36d8
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102177590"
---
# <a name="use-the-azure-importexport-service-to-export-data-from-azure-blob-storage"></a>Использование службы "Импорт и экспорт Azure" для экспорта данных из хранилища BLOB-объектов Azure

В этой статье содержатся пошаговые инструкции по использованию службы "Импорт и экспорт Azure" для безопасного экспорта больших объемов данных из хранилища BLOB-объектов Azure. Эта служба требует отправки пустых дисков в центр обработки данных Azure. Служба экспортирует данные из учетной записи хранения на диски, а затем отправляет эти диски обратно.

## <a name="prerequisites"></a>Предварительные условия

Прежде чем создавать задание экспорта для передачи данных из хранилища BLOB-объектов Azure, внимательно просмотрите и выполните описанные ниже предварительные требования для этой службы.
Необходимо сделать следующее:

- Активная подписка Azure, которую можно использовать для службы "Импорт и экспорт".
- Как минимум одна учетная запись хранения Azure. Просмотрите список [поддерживаемых типов хранилища и учетных записей хранения для службы "Импорт и экспорт"](storage-import-export-requirements.md). Сведения о создании учетной записи хранения см. в разделе [Создание учетной записи хранения](../storage/common/storage-account-create.md).
- Соответствующее количество дисков [поддерживаемых типов](storage-import-export-requirements.md#supported-disks).
- Учетная запись FedEx или DHL. Если вы хотите использовать перевозчик, отличный от FedEx/DHL, обратитесь в службу поддержки Azure Data Box по адресу `adbops@microsoft.com` .
  - Учетная запись должна быть действительной, иметь баланс и возможности возврата.
  - Создайте номер отслеживания для задания экспорта.
  - Каждое задание должно иметь отдельный номер отслеживания. Несколько заданий с одним и тем же номером отслеживания не поддерживаются.
  - Если у вас нет учетной записи оператора, перейдите по ссылке:
    - [Создайте учетную запись FedEx](https://www.fedex.com/en-us/create-account.html)или
    - [Создать учетную запись DHL](http://www.dhl-usa.com/en/express/shipping/open_account.html).

## <a name="step-1-create-an-export-job"></a>Шаг 1. Создание задания экспорта

### <a name="portal"></a>[Портал](#tab/azure-portal)

Чтобы создать задание экспорта, на портале Azure сделайте следующее.

1. Войдите в систему по адресу <https://portal.azure.com/>.
2. Выполните поиск **заданий импорта и экспорта**.

    ![Поиск заданий импорта и экспорта](./media/storage-import-export-data-to-blobs/import-to-blob-1.png)

3. Выберите **+ Создать**.

    ![Выберите + Создать, чтобы создать новый ](./media/storage-import-export-data-to-blobs/import-to-blob-2.png)

4. В разделе **Основные сведения**:

   1. Выберите подписку.
   1. Выберите группу ресурсов или щелкните **создать** и создайте новую.
   1. Введите описательное имя для задания импорта. Вы будете использовать его для отслеживания хода выполнения заданий.
       * Имя может содержать только строчные буквы, цифры и дефисы.
       * Имя должно начинаться с буквы и не может содержать пробелы.

   1. Щелкните **Экспорт из Azure**.

    ![Основные параметры для заказа на экспорт](./media/storage-import-export-data-from-blobs/export-from-blob-3.png)

    Чтобы **проследить, нажмите кнопку Далее: сведения о задании >** .

5. В разделе **Сведения о задании** сделайте следующее:

   1. Выберите регион Azure, в котором находятся ваши данные.
   1. Выберите учетную запись хранения, из которой необходимо экспортировать данные. Используйте учетную запись хранения, расположенную ближе к вашему местоположению.

      Расположение места назначения автоматически заполняется с учетом выбранного региона учетной записи хранения.

   1. Укажите данные большого двоичного объекта для экспорта из вашей учетной записи хранения на чистый диск или диски. Выберите один из трех следующих методов.

      - Выберите **Экспортировать все** для данных больших двоичных объектов в учетной записи хранения.

        ![Экспорт всех данных](./media/storage-import-export-data-from-blobs/export-from-blob-4.png)

      - Выберите **Выбранные контейнеры и большие двоичные объекты** и укажите контейнеры и большие двоичные объекты для экспорта. Можно использовать более одного метода выбора. При выборе параметра **Добавить** открывается панель справа, где можно добавить строки выбора.

        |Параметр|Описание|
        |------|-----------|      
        |**Добавить контейнеры**|Экспортируйте все большие двоичные объекты в контейнере.<br>Выберите **Добавить контейнеры** и введите имя каждого контейнера.|
        |**Добавить BLOB-объекты**|Укажите отдельные большие двоичные объекты для экспорта.<br>Выберите **Добавить большие двоичные объекты**. Затем укажите относительный путь к большому двоичному объекту, начиная с имени контейнера. Используйте *$root* для указания корневого контейнера.<br>Чтобы избежать ошибок во время обработки, необходимо указать пути к большим двоичным объектам в допустимом формате, как показано на приведенном ниже снимке экрана. Дополнительные сведения см. в разделе с [примерами допустимых путей к большим двоичным объектам](#examples-of-valid-blob-paths).|
        |**Добавить префиксы**|Используйте префикс, чтобы выбрать набор похожих контейнеров или BLOB-объектов с похожими именами в контейнере. Префикс может быть префиксом имени контейнера, полным именем контейнера или полным именем контейнера, за которым следует префикс имени большого двоичного объекта. |

        ![Экспорт выбранных контейнеров и больших двоичных объектов](./media/storage-import-export-data-from-blobs/export-from-blob-5.png)

    - Выберите **Экспорт из файла списка BLOB-объектов (XML-формат)** и выберите XML-файл, содержащий список путей и префиксов для экспортируемых BLOB-объектов из учетной записи хранения. Необходимо создать XML-файл и сохранить его в контейнере для учетной записи хранения. Файл не может быть пустым.

      > [!IMPORTANT]
      > Если вы используете XML-файл для выбора больших двоичных объектов для экспорта, убедитесь, что XML-код содержит допустимые пути и (или) префиксы. Если файл является недопустимым или не совпадает с указанными путями, порядок завершается частичными данными или данные не экспортируются.

       Чтобы узнать, как добавить XML-файл в контейнер, см. статью [Экспорт порядка с помощью XML-файла](../databox/data-box-deploy-export-ordered.md#export-order-using-xml-file).

      ![Экспорт из файла списка больших двоичных объектов](./media/storage-import-export-data-from-blobs/export-from-blob-6.png)

   > [!NOTE]
   > Если экспортируемый BLOB-объект используется во время копирования данных, служба импорта и экспорта Azure создает моментальный снимок большого двоичного объекта и копирует моментальный снимок.

   Нажмите кнопку **Далее: доставка >** для продолжения.

6. В **поставку**:

    - В раскрывающемся списке выберите перевозчика. Если вы хотите использовать перевозчика, отличный от FedEx/DHL, выберите существующий параметр из раскрывающегося списка. Свяжитесь с командой Operations Azure Data Box `adbops@microsoft.com`  с информацией о перевозчике, который планируется использовать.
    - Введите допустимый номер учетной записи, созданный в системе этого перевозчика. Корпорация Майкрософт использует эту учетную запись для обратной отправки дисков после завершения задания экспорта.
    - Укажите полное и допустимое имя контактного лица, Телефон, адрес электронной почты, улица, город, почтовый индекс, область, район и страну или регион.

        > [!TIP]
        > Вместо указания адреса электронной почты для отдельного пользователя укажите электронную почту группы. Это гарантирует, что вы получите уведомления, даже если администратор уйдет.

    Щелкните **Просмотр и создание**, чтобы продолжить.

7. На панели **Проверка и создание**:

   1. Просмотрите сведения о задании.
   1. Запишите название задания и предоставленный адрес центра обработки данных Azure для отправки дисков в Azure.

      > [!NOTE]
      > Всегда отправляйте диски в центр обработки данных, указанный на портале Azure. В случае отправки в неправильный центр обработки данных, задание не будет обрабатываться.

   1. Ознакомьтесь с **условиями** для заказа на удаление конфиденциальности и исходных данных. Если вы согласны с условиями, установите флажок под терминами. Начинается проверка порядка.

   ![Проверка и создание заказа на экспорт](./media/storage-import-export-data-from-blobs/export-from-blob-6-a.png)

 1. После завершения проверки щелкните **Создать**.

<!--Replaced text: Steps 4 - end of "Create an export job." Wizard design changes required both screen and text updates.

4. In **Basics**:

    - Select **Export from Azure**.
    - Enter a descriptive name for the export job. Use the name you choose to track the progress of your jobs.
        - The name may contain only lowercase letters, numbers, hyphens, and underscores.
        - The name must start with a letter, and may not contain spaces.
    - Select a subscription.
    - Enter or select a resource group.

        ![Basics](./media/storage-import-export-data-from-blobs/export-from-blob-3.png)

5. In **Job details**:

    - Select the storage account where the data to be exported resides. Use a storage account close to where you are located.
    - The dropoff location is automatically populated based on the region of the storage account selected.
    - Specify the blob data you wish to export from your storage account to your blank drive or drives.
    - Choose to **Export all** blob data in the storage account.

         ![Export all](./media/storage-import-export-data-from-blobs/export-from-blob-4.png)

    - You can specify which containers and blobs to export.
        - **To specify a blob to export**: Use the **Equal To** selector. Specify the relative path to the blob, beginning with the container name. Use *$root* to specify the root container.
        - **To specify all blobs starting with a prefix**: Use the **Starts With** selector. Specify the prefix, beginning with a forward slash '/'. The prefix may be the prefix of the container name, the complete container name, or the complete container name followed by the prefix of the blob name. You must provide the blob paths in valid format to avoid errors during processing, as shown in this screenshot. For more information, see [Examples of valid blob paths](#examples-of-valid-blob-paths).

           ![Export selected containers and blobs](./media/storage-import-export-data-from-blobs/export-from-blob-5.png)

    - You can export from  the blob list file.

        ![Export from blob list file](./media/storage-import-export-data-from-blobs/export-from-blob-6.png)

   > [!NOTE]
   > If the blob to be exported is in use during data copy, Azure Import/Export service takes a snapshot of the blob and copies the snapshot.

6. In **Return shipping info**:

    - Select the carrier from the dropdown list. If you want to use a carrier other than FedEx/DHL, choose an existing option from the dropdown. Contact Azure Data Box Operations team at `adbops@microsoft.com`  with the information regarding the carrier you plan to use.
    - Enter a valid carrier account number that you have created with that carrier. Microsoft uses this account to ship the drives back to you once your export job is complete.
    - Provide a complete and valid contact name, phone, email, street address, city, zip, state/province, and country/region.

        > [!TIP]
        > Instead of specifying an email address for a single user, provide a group email. This ensures that you receive notifications even if an admin leaves.

7. In **Summary**:

    - Review the details of the job.
    - Make a note of the job name and provided Azure datacenter shipping address for shipping disks to Azure.

        > [!NOTE]
        > Always send the disks to the datacenter noted in the Azure portal. If the disks are shipped to the wrong datacenter, the job will not be processed.

    - Click **OK** to complete export job creation.
-->

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы создать задание экспорта в портал Azure, выполните следующие действия.

[!INCLUDE [azure-cli-prepare-your-environment-h3.md](../../includes/azure-cli-prepare-your-environment-h3.md)]

### <a name="create-a-job"></a>Создание задания

1. Чтобы добавить расширение " [AZ Import-Export](/cli/azure/ext/import-export/import-export) ", используйте команду [AZ Extension Add](/cli/azure/extension#az_extension_add) .

    ```azurecli
    az extension add --name import-export
    ```

1. Чтобы получить список расположений, из которых можно получить диски, используйте команду [AZ Import-Export Location List](/cli/azure/ext/import-export/import-export/location#ext_import_export_az_import_export_location_list) :

    ```azurecli
    az import-export location list
    ```

1. Чтобы создать задание экспорта, которое использует имеющуюся учетную запись хранения, выполните следующую команду [AZ Import-Export Create](/cli/azure/ext/import-export/import-export#ext_import_export_az_import_export_create) .

    ```azurecli
    az import-export create \
        --resource-group myierg \
        --name Myexportjob1 \
        --location "West US" \
        --backup-drive-manifest true \
        --diagnostics-path waimportexport \
        --export blob-path=/ \
        --type Export \
        --log-level Verbose \
        --shipping-information recipient-name="Microsoft Azure Import/Export Service" \
            street-address1="3020 Coronado" city="Santa Clara" state-or-province=CA postal-code=98054 \
            country-or-region=USA phone=4083527600 \
        --return-address recipient-name="Gus Poland" street-address1="1020 Enterprise way" \
            city=Sunnyvale country-or-region=USA state-or-province=CA postal-code=94089 \
            email=gus@contoso.com phone=4085555555" \
        --storage-account myssdocsstorage
    ```

    > [!TIP]
    > Вместо указания адреса электронной почты для отдельного пользователя укажите электронную почту группы. Это гарантирует, что вы получите уведомления, даже если администратор уйдет.

   Это задание экспортирует все большие двоичные объекты в учетной записи хранения. Вы можете указать большой двоичный объект для экспорта, заменив это значение параметром **--Export**:

    ```azurecli
    --export blob-path=$root/logo.bmp
    ```

   Это значение параметра экспортирует большой двоичный объект с именем *logo.bmp* в корневом контейнере.

   Вы также можете выбрать все большие двоичные объекты в контейнере с помощью префикса. Замените это значение для **--Export**:

    ```azurecli
    blob-path-prefix=/myiecontainer
    ```

   Дополнительные сведения см. в разделе с [примерами допустимых путей к большим двоичным объектам](#examples-of-valid-blob-paths).

   > [!NOTE]
   > Если большой двоичный объект для экспорта используется во время копирования данных, служба "Импорт и экспорт Azure" делает снимок большого двоичного объекта и копирует моментальный снимок.

1. Используйте команду [AZ Import-Export List](/cli/azure/ext/import-export/import-export#ext_import_export_az_import_export_list) , чтобы просмотреть все задания для группы ресурсов миерг:

    ```azurecli
    az import-export list --resource-group myierg
    ```

1. Чтобы обновить задание или отменить задание, выполните команду [AZ Import-Export Update](/cli/azure/ext/import-export/import-export#ext_import_export_az_import_export_update) :

    ```azurecli
    az import-export update --resource-group myierg --name MyIEjob1 --cancel-requested true
    ```

### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

Чтобы создать задание экспорта в Azure PowerShell, выполните следующие действия.

[!INCLUDE [azure-powershell-requirements-h3.md](../../includes/azure-powershell-requirements-h3.md)]

> [!IMPORTANT]
> Пока модуль PowerShell **AZ. ImportExport** находится на этапе предварительной версии, его необходимо установить отдельно с помощью `Install-Module` командлета. Как только этот модуль PowerShell станет общедоступным, он будет включен в один из будущих выпусков модуля Az PowerShell и по умолчанию встроен в Azure Cloud Shell.

```azurepowershell-interactive
Install-Module -Name Az.ImportExport
```

### <a name="create-a-job"></a>Создание задания

1. Чтобы получить список расположений, из которых можно получить диски, используйте командлет [Get-азимпортекспортлокатион](/powershell/module/az.importexport/get-azimportexportlocation) :

   ```azurepowershell-interactive
   Get-AzImportExportLocation
   ```

1. Выполните следующий пример [New-азимпортекспорт](/powershell/module/az.importexport/new-azimportexport) , чтобы создать задание экспорта, которое использует имеющуюся учетную запись хранения.

   ```azurepowershell-interactive
   $Params = @{
      ResourceGroupName = 'myierg'
      Name = 'Myexportjob1'
      Location = 'westus'
      BackupDriveManifest = $true
      DiagnosticsPath = 'waimportexport'
      ExportBlobListblobPath = '\'
      JobType = 'Export'
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
      StorageAccountId = '/subscriptions/<SubscriptionId>/resourceGroups/myierg/providers/Microsoft.Storage/storageAccounts/myssdocsstorage'
   }
   New-AzImportExport @Params
   ```

    > [!TIP]
    > Вместо указания адреса электронной почты для отдельного пользователя укажите электронную почту группы. Это гарантирует, что вы получите уведомления, даже если администратор уйдет.

   Это задание экспортирует все большие двоичные объекты в учетной записи хранения. Вы можете указать большой двоичный объект для экспорта, заменив это значение параметром **-експортблоблистблобпас**:

   ```azurepowershell-interactive
   -ExportBlobListblobPath $root\logo.bmp
   ```

   Это значение параметра экспортирует большой двоичный объект с именем *logo.bmp* в корневом контейнере.

   Вы также можете выбрать все большие двоичные объекты в контейнере с помощью префикса. Замените это значение параметром **-експортблоблистблобпас**:

   ```azurepowershell-interactive
   -ExportBlobListblobPath '/myiecontainer'
   ```

   Дополнительные сведения см. в разделе с [примерами допустимых путей к большим двоичным объектам](#examples-of-valid-blob-paths).

   > [!NOTE]
   > Если большой двоичный объект для экспорта используется во время копирования данных, служба "Импорт и экспорт Azure" делает снимок большого двоичного объекта и копирует моментальный снимок.

1. Используйте командлет [Get-азимпортекспорт](/powershell/module/az.importexport/get-azimportexport) , чтобы просмотреть все задания для группы ресурсов миерг.

   ```azurepowershell-interactive
   Get-AzImportExport -ResourceGroupName myierg
   ```

1. Чтобы обновить задание или отменить задание, выполните командлет [Update-азимпортекспорт](/powershell/module/az.importexport/update-azimportexport) :

   ```azurepowershell-interactive
   Update-AzImportExport -Name MyIEjob1 -ResourceGroupName myierg -CancelRequested
   ```

---

<!--## (Optional) Step 2: -->

## <a name="step-2-ship-the-drives"></a>Шаг 2. Отправка дисков

Если вам неизвестно необходимое количество дисков, перейдите к разделу [Проверка количества дисков](#check-the-number-of-drives). Узнав количество дисков, приступите к действиям по их отправке.

[!INCLUDE [storage-import-export-ship-drives](../../includes/storage-import-export-ship-drives.md)]

## <a name="step-3-update-the-job-with-tracking-information"></a>Шаг 3. Указание данных об отслеживании для задания

[!INCLUDE [storage-import-export-update-job-tracking](../../includes/storage-import-export-update-job-tracking.md)]

## <a name="step-4-receive-the-disks"></a>Шаг 4. Получение дисков

Если на панели мониторинга появляется сообщение о завершении задания, это означает, что диски доставлены и номер для отслеживания посылки доступен на портале.

1. Чтобы разблокировать полученные диски с экспортированными данными, необходимо получить ключи BitLocker. Перейдите к заданию экспорта на портале Azure. Откройте вкладку **Импорт/экспорт**.
2. Выберите из списка задание экспорта и щелкните его. Перейдите к разделу **Шифрование** и скопируйте ключи.

   ![Просмотр ключей BitLocker для задания экспорта](./media/storage-import-export-data-from-blobs/export-from-blob-7.png)

3. Используйте ключи BitLocker для разблокировки дисков.

Экспорт завершен.

## <a name="step-5-unlock-the-disks"></a>Шаг 5. Разблокируйте диски

Чтобы разблокировать диск, используйте следующую команду:

   `WAImportExport Unlock /bk:<BitLocker key (base 64 string) copied from Encryption blade in Azure portal> /driveLetter:<Drive letter>`

Ниже приведен пример входного примера.

   `WAImportExport.exe Unlock /bk:CAAcwBoAG8AdQBsAGQAIABiAGUAIABoAGkAZABkAGUAbgA= /driveLetter:e`

В настоящее время можно удалить задание или оставить его. Задания автоматически удаляются через 90 дней.

## <a name="check-the-number-of-drives"></a>Проверка количества дисков

Этот *необязательный* шаг помогает определить количество дисков, необходимых для задания экспорта. Выполните этот шаг в системе с ОС Windows [поддерживаемой версии](storage-import-export-requirements.md#supported-operating-systems).

1. [Скачайте WAImportExport версии 1](https://www.microsoft.com/download/details.aspx?id=42659) в систему Windows.
2. Распакуйте содержимое в папку по умолчанию: `waimportexportv1`. Например, `C:\WaImportExportV1`.
3. Откройте PowerShell или окно командной строки с правами администратора. Чтобы перейти в распакованную папку, выполните следующую команду:

   `cd C:\WaImportExportV1`

4. Чтобы проверить количество дисков, необходимых для выбранных больших двоичных объектов, выполните следующую команду:

   `WAImportExport.exe PreviewExport /sn:<Storage account name> /sk:<Storage account key> /ExportBlobListFile:<Path to XML blob list file> /DriveSize:<Size of drives used>`

    Параметры описаны в следующей таблице.

    |Параметр командной строки|Описание|
    |--------------------------|-----------------|
    |**/logdir**|Необязательный параметр. Каталог журналов. В этот каталог записываются файлы подробного журнала. Если каталог не указан, вместо него используется текущий каталог.|
    |**/SN**|Обязательный. Имя учетной записи хранения для задания экспорта.|
    |**/SK**|Требуется, только если не указан SAS контейнера. Ключ учетной записи хранения для задания экспорта.|
    |**/csas:**|Требуется, только если не указан ключ учетной записи хранения. SAS контейнера для получения списка экспортируемых в рамках задания экспорта больших двоичных объектов.|
    |**/ExportBlobListFile:**|Обязательный. Путь к XML-файлу, содержащему список путей к большим двоичным объектам или префиксов путей экспортируемых больших двоичных объектов. Формат файла, используемый в элементе `BlobListBlobPath` в операции [Put Job](/rest/api/storageimportexport/jobs) интерфейса REST API службы импорта и экспорта.|
    |**/DriveSize:**|Обязательный. Размер дисков, используемых для задания экспорта, *например* 500 гб, 1,5 ТБ.|

    Дополнительные сведения см. в разделе с [примером команды PreviewExport](#example-of-previewexport-command).

5. Убедитесь, что вы можете считывать и записывать данные на диск, который будет предоставлен для задания экспорта.

### <a name="example-of-previewexport-command"></a>Пример команды PreviewExport

В следующем примере показано использование команды `PreviewExport`:

```powershell
    WAImportExport.exe PreviewExport /sn:bobmediaaccount /sk:VkGbrUqBWLYJ6zg1m29VOTrxpBgdNOlp+kp0C9MEdx3GELxmBw4hK94f7KysbbeKLDksg7VoN1W/a5UuM2zNgQ== /ExportBlobListFile:C:\WAImportExport\mybloblist.xml /DriveSize:500GB
```

Файл списка больших двоичных объектов экспорта может содержать имена и префиксы больших двоичных объектов, как показано ниже:

```xml
<?xml version="1.0" encoding="utf-8"?>
<BlobList>
<BlobPath>pictures/animals/koala.jpg</BlobPath>
<BlobPathPrefix>/vhds/</BlobPathPrefix>
<BlobPathPrefix>/movies/</BlobPathPrefix>
</BlobList>
```

Инструмент импорта и экспорта Azure получает список всех экспортируемых больших двоичных объектов и вычисляет способ их размещения на дисках определенного размера с учетом любой необходимой нагрузки, а затем определяет количество дисков, необходимое для хранения больших двоичных объектов и сведений об использовании диска.

Ниже приведен пример выходных данных без журналов сведений:

```powershell
Number of unique blob paths/prefixes:   3
Number of duplicate blob paths/prefixes:        0
Number of nonexistent blob paths/prefixes:      1

Drive size:     500.00 GB
Number of blobs that can be exported:   6
Number of blobs that cannot be exported:        2
Number of drives needed:        3
        Drive #1:       blobs = 1, occupied space = 454.74 GB
        Drive #2:       blobs = 3, occupied space = 441.37 GB
        Drive #3:       blobs = 2, occupied space = 131.28 GB
```

## <a name="examples-of-valid-blob-paths"></a>Примеры допустимых путей к большим двоичным объектам

В следующей таблице приведены примеры допустимых путей к большим двоичным объектам.

   | Выбор | Путь к BLOB-объекту | Описание |
   | --- | --- | --- |
   | Начинается с |/ |Экспорт всех blob-объектов в учетной записи хранения |
   | Начинается с |/$root/ |Экспорт всех blob-объектов в корневом контейнере |
   | Начинается с |/book |Экспортирует все blob-объекты в любом контейнере, который начинается с префикса **book** |
   | Начинается с |/music/ |Экспортирует все blob-объекты в контейнере **music** |
   | Начинается с |/music/love |Экспортирует все большие двоичные объекты в контейнере **music**, которые начинаются с префикса **love** |
   | Равно |$root/logo.bmp |Экспортирует blob-объект **logo.bmp** в корневом контейнере |
   | Равно |videos/story.mp4 |Экспортирует большой двоичный объект **story.mp4** в контейнере **videos** |

## <a name="next-steps"></a>Дальнейшие действия

- [Просмотр состояния задания и диска](storage-import-export-view-drive-status.md)
- [Сведения о требованиях службы "Импорт и экспорт"](storage-import-export-requirements.md)
