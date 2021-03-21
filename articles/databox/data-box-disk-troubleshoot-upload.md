---
title: Устранение неполадок при передаче данных с помощью журналов
titleSuffix: Azure Data Box Disk
description: Описывает использование журналов и устранение неполадок, возникающих при отправке данных в Диск Azure Data Box.
services: databox
author: alkohli
ms.service: databox
ms.subservice: disk
ms.topic: troubleshooting
ms.date: 06/17/2019
ms.author: alkohli
ms.openlocfilehash: 4b53cf607bdf60c785c7324d9ede526a0983b7e6
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97605281"
---
# <a name="understand-logs-to-troubleshoot-data-upload-issues-in-azure-data-box-disk"></a>Сведения о журналах для устранения неполадок, связанных с отправкой данных, в Диск Azure Data Box

Эта статья относится к Microsoft Azure Диск Data Box и описывает проблемы, которые вы видите при отправке данных в Azure.

## <a name="about-upload-logs"></a>Сведения о передаче журналов

Когда данные отправляются в Azure в центре обработки данных, `_error.xml` `_verbose.xml` файлы формируются для каждой учетной записи хранения. Эти журналы отправляются в ту же учетную запись хранения, которая использовалась для отправки данных. 

Оба журнала имеют одинаковый формат и содержат XML-описания событий, произошедших при копировании данных с диска в учетную запись хранения Azure.

Подробный журнал содержит полные сведения о состоянии операции копирования для каждого большого двоичного объекта или файла, а журнал ошибок содержит только сведения о больших двоичных объектах или файлах, в которых возникли ошибки во время передачи.

Журнал ошибок имеет ту же структуру, что и подробный журнал, но отфильтровывает успешные операции.

## <a name="download-logs"></a>Скачивание журналов

Чтобы указать журналы отправки, выполните следующие действия.

1. Если при отправке данных в Azure возникнут ошибки, на портале отобразится путь к папке, в которой находятся журналы диагностики.

    ![Ссылка на журналы на портале](./media/data-box-disk-troubleshoot-upload/data-box-disk-portal-logs.png)

2. Перейдите по адресу **ваиес**.

    ![журналы ошибок и подробные сведения](./media/data-box-disk-troubleshoot-upload/data-box-disk-portal-logs-1.png)

В каждом случае отображаются журналы ошибок и подробные журналы. Выберите каждый журнал и скачайте локальную копию.

## <a name="sample-upload-logs"></a>Примеры журналов отправки

Пример `_verbose.xml` показан ниже. В этом случае порядок успешно завершен без ошибок.

```xml

<?xml version="1.0" encoding="utf-8"?>
<DriveLog Version="2018-10-01">
  <DriveId>184020D95632</DriveId>
  <Blob Status="Completed">
    <BlobPath>$root/botetapageblob.vhd</BlobPath>
    <FilePath>\PageBlob\botetapageblob.vhd</FilePath>
    <Length>1073742336</Length>
    <ImportDisposition Status="Created">rename</ImportDisposition>
    <PageRangeList>
      <PageRange Offset="0" Length="4194304" Status="Completed" />
      <PageRange Offset="4194304" Length="4194304" Status="Completed" />
      <PageRange Offset="8388608" Length="4194304" Status="Completed" />
      --------CUT-------------------------------------------------------
      <PageRange Offset="1061158912" Length="4194304" Status="Completed" />
      <PageRange Offset="1065353216" Length="4194304" Status="Completed" />
      <PageRange Offset="1069547520" Length="4194304" Status="Completed" />
      <PageRange Offset="1073741824" Length="512" Status="Completed" />
    </PageRangeList>
  </Blob>
  <Blob Status="Completed">
    <BlobPath>$root/botetablockblob.txt</BlobPath>
    <FilePath>\BlockBlob\botetablockblob.txt</FilePath>
    <Length>19</Length>
    <ImportDisposition Status="Created">rename</ImportDisposition>
    <BlockList>
      <Block Offset="0" Length="19" Status="Completed" />
    </BlockList>
  </Blob>
  <File Status="Completed">
    <FileStoragePath>botetaazurefilesfolder/botetaazurefiles.txt</FileStoragePath>
    <FilePath>\AzureFile\botetaazurefilesfolder\botetaazurefiles.txt</FilePath>
    <Length>20</Length>
    <ImportDisposition Status="Created">rename</ImportDisposition>
    <FileRangeList>
      <FileRange Offset="0" Length="20" Status="Completed" />
    </FileRangeList>
  </File>
  <Status>Completed</Status>
</DriveLog>
```

Для того же порядка пример `_error.xml` показан ниже.

```xml

<?xml version="1.0" encoding="utf-8"?>
<DriveLog Version="2018-10-01">
  <DriveId>184020D95632</DriveId>
  <Summary>
    <ValidationErrors>
      <None Count="3" />
    </ValidationErrors>
    <CopyErrors>
      <None Count="3" Description="No errors encountered" />
    </CopyErrors>
  </Summary>
  <Status>Completed</Status>
</DriveLog>
```

Пример `_error.xml` показан ниже, где порядок завершен с ошибками. 

В этом случае файл ошибок содержит `Summary` раздел и другой раздел, содержащий все ошибки на уровне файлов. 

Объект `Summary` содержит `ValidationErrors` и `CopyErrors` . В этом случае в Azure было отправлено 8 файлов или папок, и ошибки проверки не обнаружены. При копировании данных в учетную запись хранения Azure 5 файлов или папок успешно отправлены. Остальные 3 файла или папок были переименованы в соответствии с соглашениями об именовании контейнеров Azure и успешно переданы в Azure.

Состояние файла `BlobStatus` содержит сведения о действиях, предпринятых для отправки больших двоичных объектов. В этом случае происходит переименование трех контейнеров, так как папки, в которые были скопированы данные, не соответствуют соглашениям об именовании Azure для контейнеров. Для больших двоичных объектов, переданных в этих контейнерах, добавляется новое имя контейнера, путь к большому двоичному объекту в Azure, исходный путь к файлу и размер большого двоичного объекта.
  
```xml
 <?xml version="1.0" encoding="utf-8"?>
  <DriveLog Version="2018-10-01">
    <DriveId>18041C582D7E</DriveId>
    <Summary>
     <!--Summary for validation and data copy to Azure -->
      <ValidationErrors>
        <None Count="8" />
      </ValidationErrors>
      <CopyErrors>
        <Completed Count="5" Description="No errors encountered" />
        <ContainerRenamed Count="3" Description="Renamed the container as the original container name does not follow Azure conventions." />
      </CopyErrors>
    </Summary>
    <!--List of renamed containers with the new names, new file path in Azure, original invalid file path, and size -->
    <Blob Status="ContainerRenamed">
      <BlobPath>databox-c2073fd1cc379d83e03d6b7acce23a6cf29d1eef/private.vhd</BlobPath>
      <OriginalFilePath>\PageBlob\pageblob test\private.vhd</OriginalFilePath>
      <SizeInBytes>10490880</SizeInBytes>
    </Blob>
    <Blob Status="ContainerRenamed">
      <BlobPath>databox-c2073fd1cc379d83e03d6b7acce23a6cf29d1eef/resource.vhd</BlobPath>
      <OriginalFilePath>\PageBlob\pageblob test\resource.vhd</OriginalFilePath>
      <SizeInBytes>71528448</SizeInBytes>
    </Blob>
    <Blob Status="ContainerRenamed">
      <BlobPath>databox-c2073fd1cc379d83e03d6b7acce23a6cf29d1eef/role.vhd</BlobPath>
      <OriginalFilePath>\PageBlob\pageblob test\role.vhd</OriginalFilePath>
      <SizeInBytes>10490880</SizeInBytes>
    </Blob>
    <Status>CompletedWithErrors</Status>
  </DriveLog>
```

## <a name="data-upload-errors"></a>Ошибки передачи данных

Ошибки, возникшие при отправке данных в Azure, приведены в следующей таблице.

| Код ошибки | Описание                   |
|-------------|------------------------------|
|`None` |  Успешно завершено.           |
|`Renamed` | Большой двоичный объект успешно переименован.   |
|`CompletedWithErrors` | Отправка завершена с ошибками. Подробные сведения о файлах с ошибками содержатся в файле журнала.  |
|`Corrupted`|CRC, вычисленный во время приема данных, не соответствует CRC, вычисленному во время передачи.  |  
|`StorageRequestFailed` | Сбой запроса службы хранилища Azure.   |     
|`LeasePresent` | Этот элемент арендован и используется другим пользователем. |
|`StorageRequestForbidden` |Не удалось отправить из-за проблем с проверкой подлинности. |
|`ManagedDiskCreationTerminalFailure` | Не удалось отправить как управляемые диски. Файлы доступны в учетной записи промежуточного хранения в виде страничных BLOB-объектов. Вы можете вручную преобразовать страничные BLOB-объекты на управляемые диски.  |
|`DiskConversionNotStartedTierInfoMissing` | Так как VHD-файл был скопирован за пределами созданных папок уровня, управляемый диск не был создан. Файл загружается как страничный BLOB-объект в промежуточную учетную запись хранения, как указано во время создания заказа. Его можно преобразовать вручную на управляемый диск.|
|`InvalidWorkitem` | Не удалось передать данные, так как они не соответствуют соглашениям об именовании и ограничениям Azure.|
|`InvalidPageBlobUploadAsBlockBlob` | Передаются как блочные BLOB-объекты в контейнере с префиксом `databoxdisk-invalid-pb-` .|
|`InvalidAzureFileUploadAsBlockBlob` | Передаются как блочные BLOB-объекты в контейнере с префиксом `databoxdisk-invalid-af` .|
|`InvalidManagedDiskUploadAsBlockBlob` | Передаются как блочные BLOB-объекты в контейнере с префиксом `databoxdisk-invalid-md` .|
|`InvalidManagedDiskUploadAsPageBlob` |Передаются как страничные BLOB-объекты в контейнере с префиксом `databoxdisk-invalid-md-` . |
|`MovedToOverflowShare` |Отправленные файлы в новый файловый ресурс, так как размер исходного общего ресурса превысил максимально допустимый размер Azure. Имя нового файлового ресурса имеет суффикс с именем `-2` .   |
|`MovedToDefaultAzureShare` |Отправленные файлы, которые не были частью какой-либо папки, в общую папку по умолчанию. Имя общего ресурса начинается с `databox-` . |
|`ContainerRenamed` |Контейнер для этих файлов не соответствует соглашениям об именовании Azure и переименовывается. Новое имя начинается с `databox-` и добавляется с суффиксом SHA1 исходного имени |
|`ShareRenamed` |Общая папка для этих файлов не соответствует соглашениям об именовании Azure и переименована. Новое имя начинается с `databox-` и добавляется с суффиксом SHA1 исходного имени. |
|`BlobRenamed` |Эти файлы не соответствуют соглашениям об именовании Azure и были переименованы. Проверьте `BlobPath` поле на наличие нового имени. |
|`FileRenamed` |Эти файлы не соответствуют соглашениям об именовании Azure и были переименованы. Проверьте `FileStoragePath` поле на наличие нового имени. |
|`DiskRenamed` |Эти файлы не соответствуют соглашениям об именовании Azure и были переименованы. Проверьте `BlobPath` поле на наличие нового имени. |


## <a name="next-steps"></a>Дальнейшие действия

- [Открывайте запрос в службу поддержки для диск Data Boxных проблем](data-box-disk-contact-microsoft-support.md).
