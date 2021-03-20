---
title: Перемещение учетной записи хранения Azure в другой регион | Документация Майкрософт
description: Показано, как переместить учетную запись хранения Azure в другой регион.
services: storage
author: normesta
ms.service: storage
ms.subservice: common
ms.topic: how-to
ms.date: 05/11/2020
ms.author: normesta
ms.reviewer: dineshm
ms.openlocfilehash: d894ce71e0ffa5a0894a1f6b0035efe66271ded8
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100591461"
---
# <a name="move-an-azure-storage-account-to-another-region"></a>Перемещение учетной записи хранения Azure в другой регион

Но вы можете создать копию своей учетной записи хранения в другом регионе. Затем переместите данные в эту учетную запись с помощью AzCopy или другого инструмента по своему усмотрению.

В этой статье вы узнаете, как выполнять следующие задачи.

> [!div class="checklist"]
> 
> * Экспорт шаблона.
> * Измените шаблон, добавив целевую область и имя учетной записи хранения.
> * Разверните шаблон, чтобы создать новую учетную запись хранения.
> * Настройте новую учетную запись хранения.
> * Перемещение данных в новую учетную запись хранения.
> * Удалите ресурсы из исходного региона.

## <a name="prerequisites"></a>Предварительные условия

- Убедитесь, что службы и функции, используемые вашей учетной записью, поддерживаются в целевом регионе.

- Для функций предварительной версии убедитесь, что ваша подписка алловлистед для целевого региона.

<a id="prepare"></a>

## <a name="prepare"></a>Подготовка.

Чтобы приступить к работе, экспортируйте и измените шаблон диспетчер ресурсов. 

### <a name="export-a-template"></a>Экспорт шаблона

Этот шаблон содержит параметры, описывающие вашу учетную запись хранения. 

# <a name="portal"></a>[Портал](#tab/azure-portal)

Чтобы экспортировать шаблон с помощью портала Azure:

1. Войдите на [портал Azure](https://portal.azure.com).

2. Выберите **все ресурсы** , а затем выберите свою учетную запись хранения.

3. Выберите **Параметры**>  >  **Экспорт шаблона**.

4. Выберите **скачать** в колонке **Экспорт шаблона** .

5. Выберите ZIP-файл, скачанный с портала, и распакуйте его в выбранную папку.

   Этот ZIP-файл содержит JSON-файлы, составляющие шаблон и скрипты для развертывания шаблона.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Чтобы экспортировать шаблон с помощью PowerShell, выполните следующие действия.

1. С помощью команды [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount) войдите в подписку Azure и следуйте инструкциям на экране:

   ```azurepowershell-interactive
   Connect-AzAccount
   ```
2. Если ваше удостоверение связано с более чем одной подпиской, установите активную подписку на подписку учетной записи хранения, которую требуется переместить.

   ```azurepowershell-interactive
   $context = Get-AzSubscription -SubscriptionId <subscription-id>
   Set-AzContext $context
   ```

3. Экспортируйте шаблон исходной учетной записи хранения. Эти команды сохраняют шаблон JSON в текущем каталоге.

   ```azurepowershell-interactive
   $resource = Get-AzResource `
     -ResourceGroupName <resource-group-name> `
     -ResourceName <storage-account-name> `
     -ResourceType Microsoft.Storage/storageAccounts
   Export-AzResourceGroup `
     -ResourceGroupName <resource-group-name> `
     -Resource $resource.ResourceId
   ```

---

### <a name="modify-the-template"></a>Изменение шаблона 

Измените шаблон, отредактировав имя учетной записи хранения и регион.

# <a name="portal"></a>[Портал](#tab/azure-portal)

Чтобы развернуть шаблон с помощью портал Azure, выполните следующие действия.

1. На портале Azure выберите **Создать ресурс**.

2. В **поле Поиск в Marketplace** введите **шаблон развертывание** и нажмите клавишу **Ввод**.

3. Выберите **шаблоны развертывания**.

    ![Библиотека шаблонов Azure Resource Manager](./media/storage-account-move/azure-resource-manager-template-library.png)

4. Нажмите кнопку **создания**.

5. Выберите **Создать собственный шаблон в редакторе**.

6. Выберите **загрузить файл** и следуйте инструкциям по загрузке **template.jsв** файл, скачанный в предыдущем разделе.

7. В поле **template.jsв** файле назовите целевую учетную запись хранения, указав значение по умолчанию для имени учетной записи хранения. В этом примере задается значение по умолчанию для имени учетной записи хранения `mytargetaccount` .
    
    ```json
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "storageAccounts_mysourceaccount_name": {
            "defaultValue": "mytargetaccount",
            "type": "String"
        }
    },
 
8. Edit the **location** property in the **template.json** file to the target region. This example sets the target region to `centralus`.

    ```json
    "resources": [{
         "type": "Microsoft.Storage/storageAccounts",
         "apiVersion": "2019-04-01",
         "name": "[parameters('storageAccounts_mysourceaccount_name')]",
         "location": "centralus"
         }]          
    ```
    Чтобы получить коды расположения регионов, см. раздел [расположения Azure](https://azure.microsoft.com/global-infrastructure/locations/).  Код для региона — это имя региона без пробелов, **Центральная американская**  =  **centralus**.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Чтобы развернуть шаблон с помощью PowerShell, выполните следующие действия.

1. В поле **template.jsв** файле назовите целевую учетную запись хранения, указав значение по умолчанию для имени учетной записи хранения. В этом примере задается значение по умолчанию для имени учетной записи хранения `mytargetaccount` .
    
    ```json
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "storageAccounts_mysourceaccount_name": {
            "defaultValue": "mytargetaccount",
            "type": "String"
        }
    },
    ``` 

2. Измените значение свойства **Location** в **template.jsв** файле на целевой регион. В этом примере целевому региону присваивается значение `eastus` .

    ```json
    "resources": [{
         "type": "Microsoft.Storage/storageAccounts",
         "apiVersion": "2019-04-01",
         "name": "[parameters('storageAccounts_mysourceaccount_name')]",
         "location": "eastus"
         }]          
    ```

    Коды регионов можно получить, выполнив команду [Get-азлокатион](/powershell/module/az.resources/get-azlocation) .

    ```azurepowershell-interactive
    Get-AzLocation | format-table 
    ```
---

<a id="move"></a>

## <a name="move"></a>Переместить

Разверните шаблон, чтобы создать новую учетную запись хранения в целевом регионе. 

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. Сохраните **template.jsв** файле.

2. Введите или выберите значения свойств:

- **Подписка**: Выберите подписку Azure.

- **Группа ресурсов**: щелкните **Создать** и укажите имя группы ресурсов.

- **Расположение**. Выберите расположение Azure.

3. Установите флажок **я принимаю указанные выше условия** и нажмите кнопку **выбрать покупку** .

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Получите идентификатор подписки, в которой вы хотите развернуть целевой общедоступный IP-адрес с помощью [Get-азсубскриптион](/powershell/module/az.accounts/get-azsubscription):

   ```azurepowershell-interactive
   Get-AzSubscription
   ```

2. Используйте следующие команды для развертывания шаблона:

   ```azurepowershell-interactive
   $resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
   $location = Read-Host -Prompt "Enter the location (i.e. centralus)"

   New-AzResourceGroup -Name $resourceGroupName -Location "$location"
   New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri "<name of your local template file>"  
   ```
---

### <a name="configure-the-new-storage-account"></a>Настройка новой учетной записи хранения

Некоторые функции не экспортируются в шаблон, поэтому их необходимо добавить в новую учетную запись хранения. 

В следующей таблице перечислены такие функции и приведены рекомендации по их добавлению в новую учетную запись хранения.

| Компонент    | Руководство    |
|--------|-----------|
| **Политики управления жизненным циклом** | [Управление жизненным циклом хранилища BLOB-объектов Azure (предварительная версия)](../blobs/storage-lifecycle-management-concepts.md) |
| **Статические веб-сайты** | [Размещение статического веб-сайта в службе хранилища Azure](../blobs/storage-blob-static-website-how-to.md) |
| **Подписки на события** | [Reacting to Blob storage events (preview)](../blobs/storage-blob-event-overview.md) (Реагирование на события хранилища BLOB-объектов) |
| **Оповещения** | [Создание, просмотр оповещений журнала действий и управление ими с помощью Azure Monitor](../../azure-monitor/alerts/alerts-activity-log.md) |
| **Сеть доставки содержимого (CDN)** | [Получение доступа к большим двоичным объектам с помощью личных доменов по HTTPS с использованием Azure CDN](../blobs/storage-https-custom-domain-cdn.md) |

> [!NOTE] 
> Если вы настроили CDN для исходной учетной записи хранения, просто измените источник существующей сети CDN на конечную точку основной конечной службы BLOB-объектов (или основную конечную точку веб-сайта) новой учетной записи. 

### <a name="move-data-to-the-new-storage-account"></a>Перемещение данных в новую учетную запись хранения

AzCopy — это предпочтительное средство для перемещения данных. Он оптимизирован для производительности.  Одним из способов ускорения является копирование данных непосредственно между серверами хранилища, поэтому AzCopy не использует пропускную способность сети компьютера. Используйте AzCopy в командной строке или в составе пользовательского сценария. См. подробнее о [начале работы с AzCopy](/azure/storage/common/storage-use-azcopy-v10?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).

Вы также можете использовать фабрику данных Azure для перемещения данных. Он предоставляет интуитивно понятный пользовательский интерфейс. Чтобы использовать фабрику данных Azure, ознакомьтесь со следующими ссылками:. 

  - [Копирование данных в хранилище BLOB-объектов Azure и обратно с помощью фабрики данных Azure](/azure/data-factory/connector-azure-blob-storage)
  - [Копирование данных в хранилище Azure Data Lake Storage 2-го поколения и из него с помощью Фабрики данных Azure](/azure/data-factory/connector-azure-data-lake-storage)
  - [Копирование данных из хранилища файлов Azure и обратно с помощью фабрики данных Azure](/azure/data-factory/connector-azure-file-storage)
  - [Копирование данных в службу "Хранилище таблиц Azure" и обратно с помощью фабрики данных Azure](/azure/data-factory/connector-azure-table-storage)

---

## <a name="discard-or-clean-up"></a>Отмена или очистка

Если после развертывания вы хотите начать заново, можно удалить целевую учетную запись хранения и повторить действия, описанные в разделах [Подготовка](#prepare) и [Перемещение](#move) этой статьи.

Чтобы сохранить изменения и завершить перемещение учетной записи хранения, удалите исходную учетную запись хранения.

# <a name="portal"></a>[Портал](#tab/azure-portal)

Чтобы удалить учетную запись хранения с помощью портала Azure:

1. В портал Azure разверните меню в левой части окна, чтобы открыть меню служб, и выберите **учетные записи хранения** , чтобы отобразить список учетных записей хранения.

2. Выберите целевую учетную запись хранения для удаления и щелкните правой кнопкой мыши кнопку **Дополнительно** (**...**) в правой части списка.

3. Выберите **Удалить** и подтвердите.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Чтобы удалить группу ресурсов и связанные с ней ресурсы, включая новую учетную запись хранения, используйте команду [Remove-азсторажеаккаунт](/powershell/module/az.storage/remove-azstorageaccount) :

```powershell
Remove-AzStorageAccount -ResourceGroupName  $resourceGroup -AccountName $storageAccount
```
---

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы переместили учетную запись хранения Azure из одного региона в другой и очистили исходные ресурсы.  Дополнительные сведения о перемещении ресурсов между регионами и аварийном восстановлении в Azure см. по следующей ссылке:


- [Перемещение ресурсов в новую группу ресурсов или подписку](../../azure-resource-manager/management/move-resource-group-and-subscription.md)
- [Перенос виртуальных машин Azure в другой регион](../../site-recovery/azure-to-azure-tutorial-migrate.md)