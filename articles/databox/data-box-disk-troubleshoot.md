---
title: Устранение неполадок диска Azure Data Box | Документация Майкрософт
description: Узнайте, как использовать журналы для устранения проблем проверки, которые могут возникнуть при развертывании Диск Azure Data Box.
services: databox
author: alkohli
ms.service: databox
ms.subservice: disk
ms.topic: troubleshooting
ms.date: 06/14/2019
ms.author: alkohli
ms.openlocfilehash: fa8a6643f1b7bd60fbf6e5950234e0381666177e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97605231"
---
# <a name="use-logs-to-troubleshoot-validation-issues-in-azure-data-box-disk"></a>Используйте журналы для устранения неполадок, связанных с проверкой, в Диск Azure Data Box

Эта статья относится к Microsoft Azure Диск Data Box. В статье описывается, как использовать журналы для устранения неполадок проверки, которые могут возникнуть при развертывании этого решения.

## <a name="validation-tool-log-files"></a>Файлы журналов средства проверки

При проверке данных на дисках с помощью [средства проверки](data-box-disk-deploy-copy-data.md#validate-data)создается *error.xml* для регистрации ошибок. Файл журнала находится в  `Drive:\DataBoxDiskImport\logs` папке на диске. Ссылка на журнал ошибок предоставляется при выполнении проверки.

<!--![Validation tool with link to error log](media/data-box-disk-troubleshoot/validation-tool-link-error-log.png)-->

Если для проверки запущено несколько сеансов, то для каждого сеанса создается один журнал ошибок.

- Ниже приведен пример журнала ошибок, когда данные, загруженные в папку, не выделяются в `PageBlob` 512-байтах. Все данные, передаваемые в PageBlob, должны быть согласованы по 512 байт, например VHD или VHDX. Ошибки в этом файле находятся в `<Errors>` предупреждениях и в `<Warnings>` .

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
        <ErrorLog Version="2018-10-01">
            <SessionId>session#1</SessionId>
            <ItemType>PageBlob</ItemType>
            <SourceDirectory>D:\Dataset\TestDirectory</SourceDirectory>
            <Errors>
                <Error Code="Not512Aligned">
                    <Description>The file is not 512 bytes aligned.</Description>
                    <List>
                        <File Path="\Practice\myScript.ps1" />
                    </List>
                    <Count>1</Count>
                </Error>
            </Errors>
            <Warnings />
        </ErrorLog>
    ```

- Ниже приведен пример журнала ошибок, когда имя контейнера недопустимо. Папка, созданная в разделе `BlockBlob` , `PageBlob` или `AzureFile` папки на диске, станет контейнером в вашей учетной записи хранения Azure. Имя контейнера должно соответствовать [соглашениям об именовании Azure](data-box-disk-limits.md#azure-block-blob-page-blob-and-file-naming-conventions).

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <ErrorLog Version="2018-10-01">
        <SessionId>bbsession</SessionId>
        <ItemType>BlockBlob</ItemType>
        <SourceDirectory>E:\BlockBlob</SourceDirectory>
        <Errors>
        <Error Code="InvalidShareContainerFormat">
            <List>
            <Container Name="Azu-reFile" />
            <Container Name="bbcont ainer1" />
            </List>
            <Count>2</Count>
        </Error>
        </Errors>
        <Warnings />
    </ErrorLog>
    ```

## <a name="validation-tool-errors"></a>Ошибки средства проверки

В следующей таблице приведены ошибки, содержащиеся в *error.xml* с соответствующими рекомендуемыми действиями.

| Код ошибки| Описание                       | Рекомендованные действия               |
|------------|--------------------------|-----------------------------------|
| `None` | Данные успешно проверены. | Никаких действий не требуется. |
| `InvalidXmlCharsInPath` |Не удалось создать файл манифеста, так как путь к файлу содержит недопустимые символы. | Удалите эти символы, чтобы продолжать.  |
| `OpenFileForReadFailed`| Не удалось обработать файл. Это может быть вызвано проблемой с доступом или повреждением файловой системы.|Не удалось прочитать файл из-за ошибки. Сведения об ошибке находятся в исключении. |
| `Not512Aligned` | Недопустимый формат файла для папки PageBlob.| Отправка только данных, 512 байт, которые были согласованы с `PageBlob` папкой. Удалите файл из папки PageBlob или переместите его в папку BlockBlob. Повторите проверку.|
| `InvalidBlobPath` | Путь к файлу не соответствует допустимому пути к большому двоичному объекту в облаке согласно соглашениям об именовании больших двоичных объектов Azure.|Чтобы переименовать путь к файлу, следуйте рекомендациям по именованию Azure. |
| `EnumerationError` | Не удалось перечислить файл для проверки. |Эта ошибка может быть вызвана несколькими причинами. Наиболее вероятной причиной является доступ к файлу. |
| `ShareSizeExceeded` | Этот файл привел к тому, что размер файлового ресурса Azure превысит ограничение в 5 ТБ.|Уменьшите размер данных в общей папке, чтобы они [применялись к пределам размера объектов Azure](data-box-disk-limits.md#azure-object-size-limits). Повторите проверку. |
| `AzureFileSizeExceeded` | Размер файла превышает ограничения на размер файлов Azure.| Сократите размер файла или данных таким образом, чтобы он соответствует [ограничениям на размер объектов Azure](data-box-disk-limits.md#azure-object-size-limits). Повторите проверку.|
| `BlockBlobSizeExceeded` | Размер файла превышает пределы размера блочного BLOB-объекта Azure. | Сократите размер файла или данных таким образом, чтобы он соответствует [ограничениям на размер объектов Azure](data-box-disk-limits.md#azure-object-size-limits). Повторите проверку. |
| `ManagedDiskSizeExceeded` | Размер файла превышает ограничения на размер управляемого диска Azure. | Сократите размер файла или данных таким образом, чтобы он соответствует [ограничениям на размер объектов Azure](data-box-disk-limits.md#azure-object-size-limits). Повторите проверку. |
| `PageBlobSizeExceeded` | Размер файла превышает ограничения на размер управляемого диска Azure. | Сократите размер файла или данных таким образом, чтобы он соответствует [ограничениям на размер объектов Azure](data-box-disk-limits.md#azure-object-size-limits). Повторите проверку. |
| `InvalidShareContainerFormat`  |Имена каталогов не соответствуют соглашениям об именовании Azure для контейнеров или общих папок.         |Первая папка, созданная в уже существующих папках на диске, станет контейнером в вашей учетной записи хранения. Этот общий ресурс или имя контейнера не соответствует соглашениям об именовании Azure. Переименуйте файл, чтобы он соответствует [соглашениям об именовании Azure](data-box-disk-limits.md#azure-block-blob-page-blob-and-file-naming-conventions). Повторите проверку.   |
| `InvalidBlobNameFormat` | Путь к файлу не соответствует допустимому пути к большому двоичному объекту в облаке согласно соглашениям об именовании больших двоичных объектов Azure.|Переименуйте файл, чтобы он соответствует [соглашениям об именовании Azure](data-box-disk-limits.md#azure-block-blob-page-blob-and-file-naming-conventions). Повторите проверку. |
| `InvalidFileNameFormat` | Путь к файлу не соответствует допустимому пути к файлу в облаке согласно соглашениям об именовании файлов Azure. |Переименуйте файл, чтобы он соответствует [соглашениям об именовании Azure](data-box-disk-limits.md#azure-block-blob-page-blob-and-file-naming-conventions). Повторите проверку. |
| `InvalidDiskNameFormat` | Путь к файлу не соответствует допустимому имени диска в облаке согласно соглашениям об именовании управляемых дисков Azure. |Переименуйте файл, чтобы он соответствует [соглашениям об именовании Azure](data-box-disk-limits.md#azure-block-blob-page-blob-and-file-naming-conventions). Повторите проверку.       |
| `NotPartOfFileShare` | Недопустимый путь для передачи файлов. Отправьте файлы в папку в службе файлов Azure.   | Удалите файлы с ошибками и отправьте их в созданную папку. Повторите проверку. |
| `NonVhdFileNotSupportedForManagedDisk` | Не удается отправить файл, не являющийся VHD-файлом, в качестве управляемого диска. |Удалите из папки файлы, не являющиеся VHD-файлами, `ManagedDisk` так как они не поддерживаются, или переместите эти файлы в `PageBlob` папку. Повторите проверку. |


## <a name="next-steps"></a>Дальнейшие действия

- Устранение [ошибок передачи данных](data-box-disk-troubleshoot-upload.md).
