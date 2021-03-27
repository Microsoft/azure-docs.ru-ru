---
title: Подключение виртуальной файловой системы в пуле
description: Узнайте, как подключить виртуальную файловую систему в пуле пакетной службы.
ms.topic: how-to
ms.custom: devx-track-csharp
ms.date: 03/26/2021
ms.openlocfilehash: dc5fbdf9ca0df8362a8999856c3f7163dd5e59b9
ms.sourcegitcommit: a9ce1da049c019c86063acf442bb13f5a0dde213
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/27/2021
ms.locfileid: "105626033"
---
# <a name="mount-a-virtual-file-system-on-a-batch-pool"></a>Подключение виртуальной файловой системы в пуле пакетной службы

Пакетная служба Azure поддерживает подключение облачного хранилища или внешней файловой системы на вычисленных узлах Windows или Linux в пулах пакетной службы. Когда вычислительный узел присоединяется к пулу, виртуальная файловая система подключается и рассматривается как локальный диск на этом узле. Вы можете подключать файловые системы, такие как Файлы Azure, хранилище BLOB-объектов Azure, сетевая файловая система (NFS), а также [кэш Avere vFXT](../avere-vfxt/avere-vfxt-overview.md) или Common Internet File System (CIFS).

Из этой статьи вы узнаете, как подключить виртуальную файловую систему в пуле вычислительных узлов с помощью [библиотеки управления пакетной службой для .NET](/dotnet/api/overview/azure/batch).

> [!NOTE]
> Подключение виртуальной файловой системы поддерживается только для пулов пакетной службы, созданных до 8 августа 2019 или после него. Пулы пакетной службы, созданные до этой даты, не будут поддерживать эту функцию.

## <a name="benefits-of-mounting-on-a-pool"></a>Преимущества подключения в пуле

Подключение файловой системы к пулу упрощает и повышает эффективность доступа к необходимым данным для задач, вместо того, чтобы позволить им получать собственные данные из большого набора данных.

Рассмотрим сценарий с несколькими задачами, требующими доступ к общему набору данных, например для отрисовки фильма. Каждая задача преобразовывает для просмотра один или несколько кадров за один раз из файлов сцены. Подключив диск, содержащий файлы сцены, компьютерным узлам легче получить доступ к общим данным.

Кроме того, базовую файловую систему можно выбирать и независимо масштабировать на основе производительности и масштабирования (пропускной способности и операции ввода-вывода в секунду), необходимых для вычислительных узлов, получающих доступ к веб-сайту. Например, [Avere vFXT](../avere-vfxt/avere-vfxt-overview.md), распределенный в кэше памяти, можно использовать для поддержки отрисовки кинокартин в больших масштабах, используя тысячи параллельных узлов отрисовки, получающих доступ к исходным данным, которые находятся в локальной среде. Кроме того, для данных, которые уже находятся в облачном хранилище BLOB-объектов, [blobfuse](../storage/blobs/storage-how-to-mount-container-linux.md) можно использовать для подключения этих данных в качестве локальной файловой системы. Blobfuse доступен только на узлах Linux, хотя служба [файлов Azure](../storage/files/storage-files-introduction.md) предоставляет аналогичный рабочий процесс и доступна как в Windows, так и в Linux.

## <a name="mount-a-virtual-file-system-on-a-pool"></a>Подключение виртуальной файловой системы в пуле  

После подключения виртуальной файловой системы в пуле файловая система становится доступной каждому вычислительному узлу в пуле. Файловая система настраивается либо при присоединении вычислительного узла к пулу, либо при перезапуске или пересоздании образа узла.

Чтобы подключить файловую систему в пуле, создайте объект `MountConfiguration`. Выберите объект, соответствующий виртуальной файловой системе: `AzureBlobFileSystemConfiguration`, `AzureFileShareConfiguration`, `NfsMountConfiguration` или `CifsMountConfiguration`.

Все объекты конфигурации подключения должны иметь указанные ниже базовые параметры. Некоторые конфигурации подключения имеют параметры, относящиеся к используемой файловой системе, которые более подробно описаны в примерах кода.

- **Имя или источник учетной записи.** Чтобы подключить виртуальную общую папку, вам понадобится имя учетной записи хранения или ее источник.
- **Относительный путь подключения или источник.** Расположение файловой системы, подключенной на вычислительном узле, относительно стандартного каталога `fsmounts`, доступного на узле с помощью `AZ_BATCH_NODE_MOUNTS_DIR`. Точное расположение зависит от операционной системы, используемой на узле. Например, физическое расположение на узле Ubuntu сопоставляется с `mnt\batch\tasks\fsmounts`, а на узле CentOS оно сопоставляется с `mnt\resources\batch\tasks\fsmounts`.
- **Параметры подключения или blobfuse.** Описывают конкретные параметры для подключения файловой системы.

После создания объекта `MountConfiguration` назначьте объект свойству `MountConfigurationList` при создании пула. Файловая система подключается либо при присоединении узла к пулу, либо при перезапуске или пересоздании образа узла.

При подключении файловой системы создается переменная среды `AZ_BATCH_NODE_MOUNTS_DIR`, которая указывает расположение подключенных файловых систем, а также файлов журналов, которые полезны для устранения неполадок и отладки. Сведения о файлах журнала более подробно предоставлены в разделе [Диагностика ошибок подключения](#diagnose-mount-errors).  

> [!IMPORTANT]
> Максимальное число подключенных файловых систем в пуле — 10. Подробные сведения и другие ограничения см. в [этом разделе](batch-quota-limit.md#other-limits).

## <a name="examples"></a>Примеры

В следующих примерах кода демонстрируется подключение множества общих папок к пулу вычислительных узлов.

### <a name="azure-files-share"></a>Общий ресурс службы файлов Azure

Файлы Azure являются стандартным предложением облачной файловой системы Azure. Дополнительные сведения о том, как получить любой из параметров в примере кода конфигурации подключения см. в статье [Использование общей папки Azure в Windows](../storage/files/storage-how-to-use-files-windows.md).

```csharp
new PoolAddParameter
{
    Id = poolId,
    MountConfiguration = new[]
    {
        new MountConfiguration
        {
            AzureFileShareConfiguration = new AzureFileShareConfiguration
            {
                AccountName = "{storage-account-name}",
                AzureFileUrl = "https://{storage-account-name}.file.core.windows.net/{file-share-name}",
                AccountKey = "{storage-account-key}",
                RelativeMountPath = "S",
                MountOptions = "-o vers=3.0,dir_mode=0777,file_mode=0777,sec=ntlmssp"
            },
        }
    }
}
```

### <a name="azure-blob-file-system"></a>Azure Blob file system

Другой вариант — использовать хранилище BLOB-объектов Azure через [blobfuse](../storage/blobs/storage-how-to-mount-container-linux.md). Для подключения файловой системы больших двоичных объектов требуется иметь `AccountKey` или `SasKey` учетной записи хранения. Сведения о получении этих ключей см. в разделах [Управление ключами доступа учетной записи хранения](../storage/common/storage-account-keys-manage.md) или [предоставление ограниченного доступа к ресурсам службы хранилища Azure с помощью подписанных URL-адресов (SAS)](../storage/common/storage-sas-overview.md). Дополнительные сведения и советы по использованию blobfuse см. в разделе blobfuse.

Чтобы получить доступ по умолчанию к подключенному каталогу blobfuse, запустите задачу от имени **администратора**. Blobfuse подключает каталог в пространстве пользователя. При создании пула он подключается в качестве корневого каталога. В Linux все задачи **администратора** являются корневыми. Все параметры для модуля ПРЕДОХРАНИТЕЛей описаны на [странице справочника по предохранителю](https://manpages.ubuntu.com/manpages/xenial/man8/mount.fuse.8.html).

Дополнительные сведения и советы по использованию blobfuse см. в [разделе часто задаваемые вопросы по устранению неполадок](https://github.com/Azure/azure-storage-fuse/wiki/3.-Troubleshoot-FAQ) . Вы также можете просмотреть [проблемы GitHub в репозитории blobfuse](https://github.com/Azure/azure-storage-fuse/issues) , чтобы проверить текущие проблемы и решения blobfuse.

```csharp
new PoolAddParameter
{
    Id = poolId,
    MountConfiguration = new[]
    {
        new MountConfiguration
        {
            AzureBlobFileSystemConfiguration = new AzureBlobFileSystemConfiguration
            {
                AccountName = "StorageAccountName",
                ContainerName = "containerName",
                AccountKey = "StorageAccountKey",
                SasKey = "",
                RelativeMountPath = "RelativeMountPath",
                BlobfuseOptions = "-o attr_timeout=240 -o entry_timeout=240 -o negative_timeout=120 "
            },
        }
    }
}
```

### <a name="network-file-system"></a>NFS

Сетевые файловые системы (NFS) могут быть подключены к узлам пула, что позволяет пакетной службе Azure получать доступ к традиционным файловым системам. Это может быть один сервер NFS, развернутый в облаке, или локальный сервер NFS, доступ к которому осуществляется через виртуальную сеть. Кроме того, можно использовать решение распределенного кэша в памяти [Авере вфкст](../avere-vfxt/avere-vfxt-overview.md) для задач высокопроизводительных вычислений (HPC), интенсивно использующих большие объемы данных.

```csharp
new PoolAddParameter
{
    Id = poolId,
    MountConfiguration = new[]
    {
        new MountConfiguration
        {
            NfsMountConfiguration = new NFSMountConfiguration
            {
                Source = "source",
                RelativeMountPath = "RelativeMountPath",
                MountOptions = "options ver=1.0"
            },
        }
    }
}
```

### <a name="common-internet-file-system"></a>Протокол Common Internet File System

Подключение [стандартных файловых систем Интернета (CIFS)](/windows/desktop/fileio/microsoft-smb-protocol-and-cifs-protocol-overview) к узлам пула — еще один способ предоставления доступа к традиционным файловым системам. CIFS — это протокол общего доступа к папкам, обеспечивающий открытый и кросс-платформенный механизм для запроса файлов и служб сетевого сервера. CIFS основан на расширенной версии протокола [SMB](/windows-server/storage/file-server/file-server-smb-overview) для общего доступа к файлам в Интернете и интрасети и может использоваться для подключения внешних файловых систем к узлам Windows.

```csharp
new PoolAddParameter
{
    Id = poolId,
    MountConfiguration = new[]
    {
        new MountConfiguration
        {
            CifsMountConfiguration = new CIFSMountConfiguration
            {
                Username = "StorageAccountName",
                RelativeMountPath = "cifsmountpoint",
                Source = "source",
                Password = "StorageAccountKey",
                MountOptions = "-o vers=3.0,dir_mode=0777,file_mode=0777,serverino"
            },
        }
    }
}
```

## <a name="diagnose-mount-errors"></a>Диагностика ошибок подключения

Если конфигурация подключения завершается неудачно, то вычисление узла в пуле завершится ошибкой, и для состояния узла будет установлено значение `unusable` . Чтобы диагностировать сбой конфигурации подключения, изучите свойство [`ComputeNodeError`](/rest/api/batchservice/computenode/get#computenodeerror) для получения сведений об ошибке.

Чтобы получить файлы журналов для отладки, используйте [OutputFiles](batch-task-output-files.md) для отправки файлов `*.log`. Файлы `*.log` содержат сведения о подключении файловой системы в расположении `AZ_BATCH_NODE_MOUNTS_DIR`. Файлы журнала подключения имеют формат `<type>-<mountDirOrDrive>.log` для каждого подключения. Например, подключение `cifs` в каталоге подключения`test` будет иметь файл журнала подключения `cifs-test.log`.

## <a name="supported-skus"></a>Поддерживаемые номера SKU

| Издатель | ПРЕДЛОЖЕНИЕ | номер SKU | Общий ресурс службы файлов Azure | Blobfuse | Подключение NFS | Подключение CIFS |
|---|---|---|---|---|---|---|
| пакет (1) | rendering-centos73 | rendering | :heavy_check_mark: <br>Примечание. Совместимо с CentOS 7.7</br>| :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| Canonical | UbuntuServer | 16.04-LTS, 18.04-LTS | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| Credativ | Debian | 8| :heavy_check_mark: | :x: | :heavy_check_mark: | :heavy_check_mark: |
| Credativ | Debian | 9 | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| microsoft-ads | linux-data-science-vm | linuxdsvm | :heavy_check_mark: <br>Примечание. Совместимо с CentOS 7.4. </br> | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| microsoft-azure-batch | centos-container | 7.6 | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| microsoft-azure-batch | centos-container-rdma | 7.4 | :heavy_check_mark: <br>Примечание. Поддержка хранилища A_8 или 9</br> | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| microsoft-azure-batch | ubuntu-server-container | 16.04-LTS | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| microsoft-dsvm | linux-data-science-vm-ubuntu | linuxdsvmubuntu | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| OpenLogic | CentOS | 7.6 | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| OpenLogic | CentOS-HPC | 7.4, 7.3, 7.1 | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| Oracle; | Oracle-Linux | 7.6 | :x: | :x: | :x: | :x: |
| Windows | WindowsServer | 2012, 2016, 2019 | :heavy_check_mark: | :x: | :x: | :x: |

## <a name="networking-requirements"></a>Требования к сети

При использовании подключений виртуальных файлов с [пулами пакетной службы Azure в виртуальной сети](batch-virtual-network.md)учитывайте следующие требования и убедитесь, что требуемый трафик не заблокирован.

- **Файлы Azure**:
  - Требуется, чтобы TCP-порт 445 был открыт для входящего и исходящего трафика тега службы хранилища. Дополнительные сведения см. в статье [использование файлового ресурса Azure с Windows](../storage/files/storage-how-to-use-files-windows.md#prerequisites).
- **Blobfuse**:
  - Требуется, чтобы TCP-порт 443 был открыт для входящего и исходящего трафика тега службы хранилища.
  - Для https://packages.microsoft.com загрузки пакетов blobfuse и GPG виртуальные машины должны иметь доступ к. В зависимости от конфигурации также может потребоваться доступ к другим URL-адресам для загрузки дополнительных пакетов.
- **Сетевая файловая система (NFS)**:
  - Требуется доступ к порту 2049 (по умолчанию, у вашей конфигурации могут быть другие требования).
  - Виртуальные машины должны иметь доступ к соответствующему диспетчеру пакетов, чтобы скачать пакет NFS-Common (для Debian или Ubuntu) или NFS-utils (для CentOS). Этот URL-адрес может отличаться в зависимости от версии ОС. В зависимости от конфигурации также может потребоваться доступ к другим URL-адресам для загрузки дополнительных пакетов.
- **Общая файловая система Интернета (CIFS)**:
  - Требуется доступ к TCP-порту 445.
  - Чтобы скачать пакет CIFS-utils, виртуальные машины должны иметь доступ к соответствующим диспетчерам пакетов. Этот URL-адрес может отличаться в зависимости от версии ОС.

## <a name="next-steps"></a>Дальнейшие шаги

- Дополнительные сведения о подключении общей папки службы файлов Azure с [Windows](../storage/files/storage-how-to-use-files-windows.md) или [Linux](../storage/files/storage-how-to-use-files-linux.md).
- Дополнительные сведения об использовании и подключении виртуальных файловых систем [blobfuse](https://github.com/Azure/azure-storage-fuse).
- Дополнительные сведения о NFS и ее приложениях см. в [этой статье](/windows-server/storage/nfs/nfs-overview).
- Дополнительные сведения о CIFS см. в статье [Протокол SMB Майкрософт и обзор протокола CIFS](/windows/desktop/fileio/microsoft-smb-protocol-and-cifs-protocol-overview).
