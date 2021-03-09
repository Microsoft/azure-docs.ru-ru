---
title: Примеры скриптов PowerShell
description: Примеры, демонстрирующие использование интерфейса с помощью скриптов PowerShell
author: florianborn71
ms.author: flborn
ms.date: 02/12/2020
ms.topic: sample
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 53f09dd042543ebc433e9d9cbbd65db332e8fb6d
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102518217"
---
# <a name="example-powershell-scripts"></a>Примеры скриптов PowerShell

Служба "Удаленная отрисовка Azure" предоставляет следующие два REST API:

- [REST API преобразования](../how-tos/conversion/conversion-rest-api.md);
- [REST API сеанса](../how-tos/session-rest-api.md).

[Репозиторий примеров для ARR](https://github.com/Azure/azure-remote-rendering) содержит в папке *Scripts* примеры скриптов для взаимодействия с REST API этой службы. В этой статье описывается их использование.

> [!TIP]
> Существует также [средство ARRT на базе пользовательского интерфейса](azure-remote-rendering-asset-tool.md), предназначенное для взаимодействия со службой. Это хорошая альтернатива использованию скриптов. ![ARRT](./media/azure-remote-rendering-asset-tool.png "Снимок экрана ARRT")

> [!CAUTION]
> Слишком частые вызовы функций REST API приведут к тому, что сервер начнет использовать регулирование и в итоге вернет ошибку. В этом случае идентификатор кода ошибки HTTP будет таким: 429 ("Слишком много запросов"). Поэтому очень важно, чтобы между последовательными вызовами была задержка в **5–10 секунд**.

## <a name="prerequisites"></a>Предварительные требования

Для выполнения примеров скриптов требуется действующая оболочка [Azure PowerShell](/powershell/azure/).

1. Чтобы установить Azure PowerShell, выполните следующие действия.
    1. Откройте окно PowerShell с правами администратора.
    1. Выполните `Install-Module -Name Az -AllowClobber`.

1. Если при выполнении скриптов будут возникать ошибки, проверьте правильность [политики выполнения](/powershell/module/microsoft.powershell.core/about/about_execution_policies).
    1. Откройте окно PowerShell с правами администратора.
    1. Выполните `Set-ExecutionPolicy -ExecutionPolicy Unrestricted`.

1. [Подготовьте учетную запись хранения Azure](../how-tos/conversion/blob-storage.md#prepare-azure-storage-accounts)

1. Войдите в подписку, которая содержит учетную запись Удаленной отрисовки Azure.
    1. Откройте окно PowerShell.
    1. Выполните `Connect-AzAccount` и следуйте инструкциям на экране.

    > [!NOTE]
    > Если в организации есть несколько подписок, может потребоваться указать аргументы SubscriptionId и Tenant. См. документацию по [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount).

1. Скачайте папку *Scripts* в [репозитории службы "Удаленная отрисовка Azure" на сайте GitHub](https://github.com/Azure/azure-remote-rendering).

## <a name="configuration-file"></a>Файл конфигурации

Рядом с файлами `.ps1` есть файл `arrconfig.json`, который нужно заполнить следующим образом.

```json
{
    "accountSettings": {
        "arrAccountId": "<fill in the account ID from the Azure Portal>",
        "arrAccountKey": "<fill in the account key from the Azure Portal>",
        "region": "<select from available regions>"
    },
    "renderingSessionSettings": {
        "vmSize": "<select standard or premium>",
        "maxLeaseTime": "<hh:mm:ss>"
    },
  "assetConversionSettings": {
    "resourceGroup": "<resource group which contains the storage account you created, only needed when uploading or generating SAS>",
    "storageAccountName": "<name of the storage account you created>",
    "blobInputContainerName": "<input container inside the storage container>",
    "blobOutputContainerName": "<output container inside the storage container>",
    "localAssetDirectoryPath": "<fill in a path to a local directory containing your asset (and files referenced from it like textures)>",
    "inputFolderPath": "<optional: base folderpath in the input container for asset upload. uses / as dir separator>",
    "inputAssetPath": "<the path to the asset under inputcontainer/inputfolderpath pointing to the input asset e.g. box.fbx>",
    "outputFolderPath": "<optional: base folderpath in the output container - the converted asset and log files will be placed here>",
    "outputAssetFileName": "<optional: filename for the converted asset, this will be placed in the output container under the outputpath>"
  }
}
```

> [!CAUTION]
> Не забудьте правильно экранировать символы обратной косой черты в пути LocalAssetDirectoryPath, используя двойные косые черты, вот так: "\\\\". Во всех остальных путях, таких как inputFolderPath и inputAssetPath, используйте символы прямой косой черты: "/".

> [!CAUTION]
> Необязательные значения должны быть заполнены либо необходимо удалить ключ и значение. Например, если вы не используете параметр `"outputAssetFileName"`, нужно удалить всю строку в файле `arrconfig.json`.

### <a name="accountsettings"></a>accountSettings

Инструкции по `arrAccountId` и `arrAccountKey` см. в статье [Создание учетной записи для Удаленной отрисовки Azure](../how-tos/create-an-account.md).
Сведения о `region` см. в [списке доступных регионов](../reference/regions.md).

### <a name="renderingsessionsettings"></a>renderingSessionSettings

Эта структура должна быть заполнена, если вы намерены выполнять **RenderingSession.ps1**:

- **vmSize.** Определяет размер виртуальной машины. Выберите [*standard*](../reference/vm-sizes.md) или [*premium*](../reference/vm-sizes.md). Завершайте сеансы отрисовки, когда они становятся не нужны.
- **maxLeaseTime.** Прогнозируемая продолжительность аренды виртуальной машины. По истечении срока действия аренды работа виртуальной машины будет прекращена. Срок аренды можно продлить позже (как описано ниже).

### <a name="assetconversionsettings"></a>assetConversionSettings

Эта структура должна быть заполнена, если вы намерены выполнять **Conversion.ps1**.

Подробные сведения см. в статье [Подготовка учетной записи хранения Azure](../how-tos/conversion/blob-storage.md#prepare-azure-storage-accounts).

## <a name="script-renderingsessionps1"></a>Скрипт: RenderingSession.ps1

Этот скрипт используется для создания, запрашивания и завершения сеансов отрисовки.

> [!IMPORTANT]
> Обязательно заполните разделы *accountSettings* и *renderingSessionSettings* в файле arrconfig.json.

### <a name="create-a-rendering-session"></a>Создание сеанса отрисовки

Обычное использование с полностью заполненным файлом arrconfig.json:

```PowerShell
.\RenderingSession.ps1
```

Этот скрипт вызовет [REST API управления сеансами](../how-tos/session-rest-api.md) для запуска виртуальной машины отрисовки с указанными параметрами. При успешном выполнении он получит *sessionId*. После этого он будет опрашивать свойства этого сеанса, пока тот не будет завершен (успешно или с ошибкой).

Чтобы использовать **альтернативный файл конфигурации**, выполните:

```PowerShell
.\RenderingSession.ps1 -ConfigFile D:\arr\myotherconfigFile.json
```

Вы можете **переопределить отдельные параметры** из файла конфигурации:

```PowerShell
.\RenderingSession.ps1 -Region <region> -VmSize <vmsize> -MaxLeaseTime <hh:mm:ss>
```

Вы можете **запустить сеанс без опроса**:

```PowerShell
.\RenderingSession.ps1 -CreateSession
```

Полученный скриптом параметр *sessionId* нужно передавать в большинство команд сеанса.

### <a name="retrieve-session-properties"></a>Получение свойств сеанса

Чтобы получить свойства сеанса, выполните:

```PowerShell
.\RenderingSession.ps1 -GetSessionProperties -Id <sessionID> [-Poll]
```

Команда `-Poll` ожидает, пока сеанс не перейдет в состояние *готовности* или не завершится ошибкой.

### <a name="list-active-sessions"></a>Перечисление активных сеансов

```PowerShell
.\RenderingSession.ps1 -GetSessions
```

### <a name="stop-a-session"></a>Остановка сеанса

```PowerShell
.\RenderingSession.ps1 -StopSession -Id <sessionID>
```

### <a name="change-session-properties"></a>Изменение свойств сеанса

Сейчас мы поддерживаем для сеанса только изменение параметра maxLeaseTime.

> [!NOTE]
> Срок аренды всегда отсчитывается от момента первоначального создания виртуальной машины сеанса. Чтобы продлить аренду сеанса еще на час, увеличьте значение *maxLeaseTime* на один час.

```PowerShell
.\RenderingSession.ps1 -UpdateSession -Id <sessionID> -MaxLeaseTime <hh:mm:ss>
```

## <a name="script-conversionps1"></a>Скрипт: Conversion.ps1

Этот скрипт используется для преобразования входных моделей в формат среды выполнения, характерный для Удаленной отрисовки Azure.

> [!IMPORTANT]
> Обязательно заполните разделы *accountSettings* и *assetConversionSettings* в файле arrconfig.json.

В этом скрипте демонстрируются два варианта использования учетных записей хранения со службой:

- учетная запись хранения, связанная с учетной записью Удаленной отрисовки Azure;
- предоставление доступа к хранилищу через подписанный URL-адрес (SAS).

### <a name="linked-storage-account"></a>Связанная учетная запись хранения

Полностью заполнив файл arrconfig.json и настроив связанную учетную запись хранения, вы сможете выполнить следующую команду. Связывание учетной записи хранения описано в разделе [о создании учетной записи](../how-tos/create-an-account.md#link-storage-accounts).

Использование связанной учетной записи хранения является предпочтительным способом использования службы преобразования, так как для него не нужно создавать подписанные URL-адреса.

```PowerShell
.\Conversion.ps1
```

1. Отправьте все файлы, содержащиеся в `assetConversionSettings.modelLocation`, во входной контейнер больших двоичных объектов для заданного пути `inputFolderPath`.
1. Вызовите [REST API преобразования модели](../how-tos/conversion/conversion-rest-api.md), чтобы начать [преобразование модели](../how-tos/conversion/model-conversion.md).
1. Состояние преобразования будет опрашиваться, пока преобразование не завершится (успешно или с ошибкой).
1. Будут выведены сведения о расположении преобразованного файла (учетная запись хранения, контейнер выходных данных, путь к файлу в контейнере).

### <a name="access-to-storage-via-shared-access-signatures"></a>Доступ к хранилищу через подписанные URL-адреса

```PowerShell
.\Conversion.ps1 -UseContainerSas
```

В результате:

1. Будет передан файл из локальной среды `assetConversionSettings.localAssetDirectoryPath` во входной контейнер больших двоичных объектов.
1. Будет создан подписанный URL-адрес для контейнера входных данных.
1. Будет создан подписанный URL-адрес для контейнера выходных данных.
1. Будет вызван [REST API преобразования модели](../how-tos/conversion/conversion-rest-api.md), чтобы начать [преобразование модели](../how-tos/conversion/model-conversion.md).
1. Состояние преобразования будет опрашиваться, пока преобразование не завершится (успешно или с ошибкой).
1. Будут выведены сведения о расположении преобразованного файла (учетная запись хранения, контейнер выходных данных, путь к файлу в контейнере).
1. Будет выведен подписанный URL-адрес преобразованной модели в выходном контейнере больших двоичных объектов.

### <a name="additional-command-line-options"></a>Дополнительные параметры командной строки

Чтобы использовать **альтернативный файл конфигурации**, выполните:

```PowerShell
.\Conversion.ps1 -ConfigFile D:\arr\myotherconfigFile.json
```

Вы можете **запустить преобразование модели без опроса**:

```PowerShell
.\Conversion.ps1 -ConvertAsset
```

Вы можете **переопределить отдельные параметры** из файла конфигурации, указав следующие аргументы командной строки:

* **Id:** идентификатор преобразования, указанный для получения состояния преобразования (GetConversionStatus).
* **ArrAccountId:** значение arrAccountId из accountSettings.
* **ArrAccountKey:** переопределение значения arrAccountKey из accountSettings.
* **Region:** переопределение значения региона из accountSettings.
* **ResourceGroup:** переопределение значения resourceGroup из assetConversionSettings.
* **StorageAccountName:** переопределение значения storageAccountName из assetConversionSettings.
* **BlobInputContainerName:** переопределение значения blobInputContainer из assetConversionSettings.
* **LocalAssetDirectoryPath:** переопределение значения localAssetDirectoryPath из assetConversionSettings.
* **InputAssetPath:** переопределение значения inputAssetPath из assetConversionSettings.
* **BlobOutputContainerName:** переопределение значения blobOutputContainerName из assetConversionSettings.
* **OutputFolderPath:** переопределение значения outputFolderPath из assetConversionSettings.
* **OutputAssetFileName:** переопределение значения outputAssetFileName из assetConversionSettings.

Например, можно объединить несколько параметров следующим образом:

```PowerShell
.\Conversion.ps1 -LocalAssetDirectoryPath "C:\\models\\box" -InputAssetPath box.fbx -OutputFolderPath another/converted/box -OutputAssetFileName newConversionBox.arrAsset
```

### <a name="run-the-individual-conversion-stages"></a>Запуск отдельных этапов преобразования

Если вы хотите выполнить отдельные шаги этого процесса, можно сделать следующее.

Отправить данные только из заданного пути LocalAssetDirectoryPath.

```PowerShell
.\Conversion.ps1 -Upload
```

Запустить только процесс преобразования модели, уже отправленной в хранилище BLOB-объектов (без отправки и без опроса состояния преобразования). Скрипт вернет значение *conversionId*.

```PowerShell
.\Conversion.ps1 -ConvertAsset
```

Чтобы получить состояние преобразования в этом случае, используйте:

```PowerShell
.\Conversion.ps1 -GetConversionStatus -Id <conversionId> [-Poll]
```

Команда `-Poll` ожидает, пока преобразование не завершится (успешно или с ошибкой).

## <a name="next-steps"></a>Дальнейшие действия

- [Краткое руководство. Отрисовка модели с помощью Unity](../quickstarts/render-model.md)
- [Краткое руководство. Преобразование модели для отрисовки](../quickstarts/convert-model.md)
- [Преобразование модели](../how-tos/conversion/model-conversion.md)