---
title: Подключение виртуальной файловой системы в пуле
description: Узнайте, как подключить виртуальную файловую систему в пуле пакетной службы.
ms.topic: how-to
ms.custom: devx-track-csharp
ms.date: 08/13/2019
ms.openlocfilehash: df03275fdeea88df1a2f2b6e2cda55021497cdf7
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89145490"
---
# <a name="mount-a-virtual-file-system-on-a-batch-pool"></a>Подключение виртуальной файловой системы в пуле пакетной службы

Пакетная служба Azure теперь поддерживает подключение облачного хранилища или внешней файловой системы на вычисленных узлах Windows или Linux в пулах пакетной службы. Когда вычислительный узел присоединяется к пулу, виртуальная файловая система подключается и рассматривается как локальный диск на этом узле. Вы можете подключать файловые системы, такие как Файлы Azure, хранилище BLOB-объектов Azure, сетевая файловая система (NFS), а также [кэш Avere vFXT](../avere-vfxt/avere-vfxt-overview.md) или Common Internet File System (CIFS).

Из этой статьи вы узнаете, как подключить виртуальную файловую систему в пуле вычислительных узлов с помощью [библиотеки управления пакетной службой для .NET](/dotnet/api/overview/azure/batch).

> [!NOTE]
> Подключение виртуальной файловой системы поддерживается в пулах пакетов, созданных 19 августа 2019 г или позже. Пулы пакетов, созданные до 19 августа 2019 г, не поддерживают эту функцию.
> 
> API-интерфейсы для подключения файловых систем в вычислительном узле входят в библиотеку [.NET для пакетной службы](/dotnet/api/microsoft.azure.batch).

## <a name="benefits-of-mounting-on-a-pool"></a>Преимущества подключения в пуле

Подключение файловой системы к пулу упрощает и повышает эффективность доступа к необходимым данным для задач, вместо того, чтобы позволить им получать собственные данные из большого набора данных.

Рассмотрим сценарий с несколькими задачами, требующими доступ к общему набору данных, например для отрисовки фильма. Каждая задача преобразовывает для просмотра один или несколько кадров за один раз из файлов сцены. Подключив диск, содержащий файлы сцены, компьютерным узлам легче получить доступ к общим данным. Кроме того, базовую файловую систему можно выбирать и независимо масштабировать на основе производительности и масштабирования (пропускной способности и операции ввода-вывода в секунду), необходимых для вычислительных узлов, получающих доступ к веб-сайту. Например, [Avere vFXT](../avere-vfxt/avere-vfxt-overview.md), распределенный в кэше памяти, можно использовать для поддержки отрисовки кинокартин в больших масштабах, используя тысячи параллельных узлов отрисовки, получающих доступ к исходным данным, которые находятся в локальной среде. Кроме того, для данных, которые уже находятся в облачном хранилище BLOB-объектов, [blobfuse](../storage/blobs/storage-how-to-mount-container-linux.md) можно использовать для подключения этих данных в качестве локальной файловой системы. Blobfuse доступен только на узлах Linux, но [Файлы Azure](https://azure.microsoft.com/blog/a-new-era-for-azure-files-bigger-faster-better/) обеспечивают аналогичный рабочий процесс и доступны как в Windows, так и в Linux.

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

Другой вариант — использовать хранилище BLOB-объектов Azure через [blobfuse](../storage/blobs/storage-how-to-mount-container-linux.md). Для подключения файловой системы больших двоичных объектов требуется иметь `AccountKey` или `SasKey` учетной записи хранения. Дополнительные сведения о получении этих ключей см. в статьях [Управление ключами доступа к учетной записи хранения](../storage/common/storage-account-keys-manage.md) и [Использование подписанных URL-адресов SAS в службе хранилища Azure](../storage/common/storage-sas-overview.md). Дополнительные сведения об использовании blobfuse см. в статье [Вопросы и ответы об устранении неполадок](https://github.com/Azure/azure-storage-fuse/wiki/3.-Troubleshoot-FAQ). Чтобы получить доступ по умолчанию к подключенному каталогу blobfuse, запустите задачу от имени **администратора**. Blobfuse подключает каталог в пространстве пользователя. При создании пула он подключается в качестве корневого каталога. В Linux все задачи **администратора** являются корневыми. Все параметры для модуля FUSE описаны на странице [справочника по FUSE](https://manpages.ubuntu.com/manpages/xenial/man8/mount.fuse.8.html).

В дополнение к руководству по устранению неполадок раздел с описанием проблем на GitHub в репозитории blobfuse содержит полезную информацию для проверки текущих проблем и решений blobfuse. Дополнительные сведения см. в статье о [проблемах blobfuse](https://github.com/Azure/azure-storage-fuse/issues).

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

NFS можно подключать к узлам пула, что позволяет узлам пакетной службы Azure легко получать доступ к традиционным файловым системам узлов пакетной службы. Это может быть один сервер NFS, развернутый в облаке, или локальный сервер NFS, доступ к которому осуществляется через виртуальную сеть. Кроме того, воспользуйтесь преимуществами [Avere vFXT](../avere-vfxt/avere-vfxt-overview.md), распределенного в решении кэша памяти, который обеспечивает бесперебойное подключение к локальному хранилищу, возможность считывать данные по запросу в свой кэш и высокую производительность и масштабируемость облачных кластерных узлов.

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

Протокол Common Internet File System также можно подключать к узлам пула, что позволяет легко получить доступ к традиционным файловым системам с помощью узлов пакетной службы Azure. CIFS — это протокол общего доступа к папкам, обеспечивающий открытый и кросс-платформенный механизм для запроса файлов и служб сетевого сервера. CIFS основан на усовершенствованной версии протокола SMB, который предоставляет общий доступ к файлам в Интернете и интрасети и позволяет выполнить подключение внешних файловых систем на узлах Windows. Дополнительные сведения о протоколе SMB см. в статье [Общие сведения о совместном использовании файлов с помощью протокола SMB 3 в Windows Server](/windows-server/storage/file-server/file-server-smb-overview).

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

В случае сбоя конфигурации подключения вычислительный узел в пуле возвращает ошибку и состояние узла становится непригодным к использованию. Чтобы диагностировать сбой конфигурации подключения, изучите свойство [`ComputeNodeError`](/rest/api/batchservice/computenode/get#computenodeerror) для получения сведений об ошибке.

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

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о подключении общего ресурса службы файлов Azure с помощью [Windows](../storage/files/storage-how-to-use-files-windows.md) или [Linux](../storage/files/storage-how-to-use-files-linux.md).
- Дополнительные сведения об использовании и подключении виртуальных файловых систем [blobfuse](https://github.com/Azure/azure-storage-fuse).
- Дополнительные сведения о NFS и ее приложениях см. в [этой статье](/windows-server/storage/nfs/nfs-overview).
- Дополнительные сведения о CIFS см. в статье [Протокол SMB Майкрософт и обзор протокола CIFS](/windows/desktop/fileio/microsoft-smb-protocol-and-cifs-protocol-overview).
