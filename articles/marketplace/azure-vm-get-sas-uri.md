---
title: Создание URI SAS для образа виртуальной машины — Azure Marketplace
description: Создайте URI подписанного URL-адрес (SAS) для виртуальных жестких дисков (VHD) в Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: iqshahmicrosoft
ms.author: krsh
ms.date: 03/10/2021
ms.openlocfilehash: 21ccafe3e15f902e35657a9aa31516bbaeb3b4c8
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105558012"
---
# <a name="how-to-generate-a-sas-uri-for-a-vm-image"></a>Создание URI SAS для образа виртуальной машины

> [!NOTE]
> Для публикации виртуальной машины не требуется URI SAS. Вы можете просто поделиться изображением в центре части. Дополнительные сведения см. в статье [Создание виртуальной машины с помощью утвержденной базы](./azure-vm-create-using-approved-base.md) или [Создание виртуальной машины с помощью собственных образов](./azure-vm-create-using-own-image.md) .

Создание URI SAS для виртуальных жестких дисков имеет следующие требования:

- Они поддерживают только неуправляемые виртуальные жесткие диски.
- Требуются только разрешения List и Read. Не предоставлять доступ на запись или удаление.
- длительность предоставления доступа (до даты окончания срока действия) должна составлять не менее трех недель с момента создания URI SAS;
- чтобы не зависеть от формата времени UTC, укажите дату начала на один день раньше текущей даты; Например, если текущая дата — 16 июня 2020, то выберите 6/15/2020.

## <a name="extract-vhd-from-a-vm"></a>Извлечение виртуального жесткого диска из виртуальной машины

> [!NOTE]
> Если виртуальный жесткий диск уже передан в учетную запись хранения, этот шаг можно пропустить.

Чтобы извлечь виртуальный жесткий диск из виртуальной машины, необходимо создать моментальный снимок диска виртуальной машины и извлечь VHD из моментального снимка.

Начните с создания моментального снимка диска виртуальной машины:

1. Войдите на портал Azure.
2. Начиная с верхнего левого угла, выберите Создать ресурс, а затем найдите и выберите снимок.
3. В колонке моментальный снимок выберите Создать.
4. Заполните поле Имя для моментального снимка.
5. Выберите существующую группу ресурсов или введите имя для новой.
6. В поле Исходный диск выберите управляемый диск, моментальный снимок которого необходимо создать.
7. Выберите тип учетной записи, которая будет использоваться для хранения моментального снимка. Используйте диск HDD ценовой категории "Стандартный", если вам не нужно хранить моментальный снимок на высокопроизводительном диске SSD.
8. Нажмите кнопку "Создать".

### <a name="extract-the-vhd"></a>Извлечение виртуального жесткого диска

Используйте следующий скрипт для экспорта моментального снимка в виртуальный жесткий диск в вашей учетной записи хранения.

```azurecli
#Provide the subscription Id where the snapshot is created
$subscriptionId=yourSubscriptionId

#Provide the name of your resource group where the snapshot is created
$resourceGroupName=myResourceGroupName

#Provide the snapshot name
$snapshotName=mySnapshot

#Provide Shared Access Signature (SAS) expiry duration in seconds (such as 3600)
#Know more about SAS here: https://docs.microsoft.com/en-us/azure/storage/storage-dotnet-shared-access-signature-part-1
$sasExpiryDuration=3600

#Provide storage account name where you want to copy the underlying VHD file. Currently, only general purpose v1 storage is supported.
$storageAccountName=mystorageaccountname

#Name of the storage container where the downloaded VHD will be stored.
$storageContainerName=mystoragecontainername

#Provide the key of the storage account where you want to copy the VHD 
$storageAccountKey=mystorageaccountkey

#Give a name to the destination VHD file to which the VHD will be copied.
$destinationVHDFileName=myvhdfilename.vhd

az account set --subscription $subscriptionId

$sas=$(az snapshot grant-access --resource-group $resourceGroupName --name $snapshotName --duration-in-seconds $sasExpiryDuration --query [accessSas] -o tsv)

az storage blob copy start --destination-blob $destinationVHDFileName --destination-container $storageContainerName --account-name $storageAccountName --account-key $storageAccountKey --source-uri $sas
```

### <a name="script-explanation"></a>Описание скрипта
Этот скрипт использует следующие команды для создания URI SAS для моментального снимка и копирует базовый виртуальный жесткий диск в учетную запись хранения с помощью URI SAS. Для каждой команды в таблице приведены ссылки на соответствующую документацию.


|Get-Help  |Примечания  |
|---------|---------|
| az disk grant-access    |     Создает SAS только для чтения, который используется для копирования базового VHD-файла в учетную запись хранения или загрузки в локальную среду    |
|  az storage blob copy start   |    Асинхронно копирует большой двоичный объект из одной учетной записи хранения в другую. Чтобы проверить состояние нового большого двоичного объекта, используйте команду AZ Storage BLOB показывать.     |
|

## <a name="generate-the-sas-address"></a>Создание адреса SAS

Существует два стандартных средства, используемых для создания адреса SAS (URL):

1. **Обозреватель службы хранилища Azure** — доступно на портал Azure.
2. **Azure CLI** — рекомендуется для операционных систем, отличных от Windows, и для сред автоматической или непрерывной интеграции.

### <a name="using-tool-1-azure-storage-explorer"></a>Использование средства 1: Обозреватель службы хранилища Azure

1. Перейдите к своей **учетной записи хранения**.
1. Откройте **Обозреватель службы хранилища**.

    :::image type="content" source="media/create-vm/storge-account-explorer.png" alt-text="Окно учетной записи хранения.":::

3. В **контейнере** щелкните правой кнопкой мыши файл VHD и выберите **получить общий доступ к подписи**.
4. В диалоговом окне **подпись общего доступа** заполните следующие поля:

    1. Время начала — дата начала срока действия разрешений на доступ к виртуальному жесткому диску. Укажите дату на день раньше текущей даты.
    2. Время окончания срока действия — дата окончания срока действия разрешений на доступ к виртуальному жесткому диску. Укажите дату по меньшей мере через три недели после текущей даты.
    3. Разрешения. Выберите разрешения на создание списков и чтение.
    4. Уровень контейнера. Установите флажок Создать URI подписанного URL-адреса на уровне контейнера.

    ![Диалоговое окно подпись общего доступа.](media/vm/create-sas-uri-storage-explorer.png)

5. Нажмите **Создать**, чтобы создать URI SAS для этого виртуального жесткого диска.
6. Скопируйте URI и сохраните его в текстовом файле в надежном расположении. Этот URI SAS предоставляет доступ на уровне контейнера. Чтобы сделать его специфическим, измените текстовый файл, добавив имя виртуального жесткого диска.
7. Вставьте имя виртуального жесткого диска в URI SAS после строки vhds (добавьте символ косой черты). В окончательном виде URI SAS должен выглядеть следующим образом:

    `<blob-service-endpoint-url> + /vhds/ + <vhd-name>? + <sas-connection-string>`

8. Повторите эти шаги для каждого виртуального жесткого диска в публикуемом плане.

### <a name="using-tool-2-azure-cli"></a>Использование средства 2: Azure CLI

1. Скачайте и установите [Microsoft Azure CL](/cli/azure/install-azure-cli)I. Доступны версии для macOS, Windows и различных дистрибутивов Linux.
2. Создайте файл PowerShell (с расширением PS1), скопируйте в него следующий код и сохраните его в локальной файловой системе.

    ```azurecli-interactive
    az storage container generate-sas --connection-string ‘DefaultEndpointsProtocol=https;AccountName=<account-name>;AccountKey=<account-key>;EndpointSuffix=core.windows.net’ --name <container-name> --permissions rl --start ‘<start-date>’ --expiry ‘<expiry-date>’
    ```

3. Измените текст файла, добавив следующие значения параметров. Укажите даты в формате UTC DateTime, например 2020-04-01T00:00:00Z.

    - Account-Name — имя учетной записи хранения Azure.
    - ключ учетной записи — ключ учетной записи хранения Azure.
    - Начальная дата — дата начала разрешения для доступа к виртуальному жесткому диску. Укажите дату на день раньше текущей даты.
    - Дата окончания срока действия — срок действия разрешения для доступа к виртуальному жесткому диску. Укажите дату по меньшей мере через три недели после текущей даты.

    Ниже приведен пример правильных значений параметров (на момент написания статьи):

    ```azurecli-interactive
    az storage container generate-sas --connection-string ‘DefaultEndpointsProtocol=https;AccountName=st00009;AccountKey=6L7OWFrlabs7Jn23OaR3rvY5RykpLCNHJhxsbn9ON c+bkCq9z/VNUPNYZRKoEV1FXSrvhqq3aMIDI7N3bSSvPg==;EndpointSuffix=core.windows.net’ --name <container-name> -- permissions rl --start ‘2020-04-01T00:00:00Z’ --expiry ‘2021-04-01T00:00:00Z’
    ```

1. Сохраните изменения.
2. Запустите этот скрипт с правами администратора одним из следующих способов, чтобы создать строку подключения SAS для доступа на уровне контейнера.

    - Запуск скрипта из консоли. В Windows щелкните скрипт правой кнопкой мыши и выберите пункт **Запустить от имени администратора**.
    - Запуск скрипта из редактора скриптов PowerShell, например из [интегрированной среды сценариев Windows PowerShell](/powershell/scripting/components/ise/introducing-the-windows-powershell-ise). На этом экране показано создание строки подключения SAS в этом редакторе:

    [![Создание строки подключения SAS в редакторе PowerShell](media/vm/create-sas-uri-power-shell-ise.png)](media/vm/create-sas-uri-power-shell-ise.png#lightbox)

6. Скопируйте строку подключения SAS и сохраните ее в текстовом файле в надежном расположении. Чтобы завершить создание URI SAS, в эту строку нужно добавить сведения о расположении виртуального жесткого диска.
7. На портале Azure перейдите к хранилищу BLOB-объектов, которое содержит виртуальный жесткий диск, связанный с новым URI.
8. Скопируйте URL-адрес конечной точки службы больших двоичных объектов:

    ![Копирование URL-адреса конечной точки службы больших двоичных объектов.](media/vm/create-sas-uri-blob-endpoint.png)

9. Измените текстовый файл, добавив строку подключения SAS, полученную на шаге 6. Создайте полный URI SAS в следующем формате:

    `<blob-service-endpoint-url> + /vhds/ + <vhd-name>? + <sas-connection-string>`

## <a name="verify-the-sas-uri"></a>Проверка URI SAS

Проверьте URI SAS перед публикацией в центре партнеров, чтобы избежать проблем, связанных с URI SAS после отправки запроса. Этот процесс является необязательным, но рекомендуется.

- URI включает имя файла образа VHD, включая расширение имени файла `.vhd` .
- `Sp=rl` отображается около середины URI. В этой строке указывается доступ на чтение и список.
- Если отображается `sr=c`, эта строка обозначает разрешения на уровне контейнера.
- Скопируйте и вставьте URI в браузер, чтобы проверить скачивание большого двоичного объекта (операцию можно отменить до завершения скачивания).

## <a name="next-steps"></a>Дальнейшие действия

- Если возникли проблемы, см. раздел [сообщения об ошибках SAS виртуальной машины](azure-vm-sas-failure-messages.md).
- [Вход в центр партнеров](https://partner.microsoft.com/dashboard/account/v3/enrollment/introduction/partnership)
- [Создание предложения виртуальной машины в Azure Marketplace](azure-vm-create.md)