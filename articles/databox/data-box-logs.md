---
title: Отслеживание и Azure Data Box журналов, Azure Data Box Heavy событий для порядка импорта | Документация Майкрософт
description: Описание процесса трассировки и регистрации событий на различных стадиях Azure Data Box и Azure Data Box Heavy порядке импорта.
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: article
ms.date: 07/10/2020
ms.author: alkohli
ms.openlocfilehash: a9304936f746b82b59550d62e8b60a9e0035d188
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92147934"
---
# <a name="tracking-and-event-logging-for-your-azure-data-box-and-azure-data-box-heavy-import-order"></a>Отслеживание и регистрация событий заказа на импорт Azure Data Box и Azure Data Box Heavy

Порядок импорта Data Box или Data Box Heavy проходит следующие шаги: порядок, Настройка, копирование данных, возврат, отправка в Azure и проверка и очистки данных. В соответствии с каждым шагом в заказе можно выполнить несколько действий по управлению доступом к заказу, аудиту событий, отслеживанию порядка и интерпретации различных формируемых журналов.

В следующей таблице приведена сводка этапов заказа на импорт Data Box или Data Box Heavy, а также средства, доступные для отслеживания и аудита заказа на каждом этапе.

| Этап заказа на импорт Data Box       | Средство для отслеживания и аудита                                                                        |
|----------------------------|------------------------------------------------------------------------------------------------|
| Создание заказа               | [Настройка контроля доступа в порядке с помощью Azure RBAC](#set-up-access-control-on-the-order)                                                    |
| Заказ обработан            | [Отслеживание порядка](#track-the-order) по <ul><li> портала Azure; </li><li> веб-сайта оператора доставки; </li><li>уведомления по электронной почте.</ul> |
| Настройка устройства              | [Журналы действий](#query-activity-logs-during-setup) доступа к учетным данным устройства                                              |
| Копирование данных на устройство        | [Просмотр файлов *error.xml*](#view-error-log-during-data-copy) для копирования данных                                                             |
| Подготовка к отправке            | [Проверка файлов спецификации](#inspect-bom-during-prepare-to-ship) или файлов манифеста на устройстве                                      |
| Отправка данных в Azure       | [Просмотрите журналы копирования](#review-copy-log-during-upload-to-azure) для ошибок при отправке данных в центре Azure                         |
| Удаление данных с устройства   | [Просмотр цепочки журналов поставок](#get-chain-of-custody-logs-after-data-erasure) , включая журналы аудита и журнал заказов                |

В этой статье подробно описаны различные механизмы и средства, доступные для контроля и аудита Data Box или Data Box Heavy порядка импорта. Сведения в этой статье применимы как к, так Data Box и Data Box Heavy импорта заказов. В последующих разделах все ссылки на Data Box также применяются к Data Box Heavy.

## <a name="set-up-access-control-on-the-order"></a>Настройка контроля доступа в заказе

Вы можете контролировать, кто имеет доступ к вашему заказу при первом создании заказа. Настройте роли Azure в различных областях для управления доступом к Data Boxму заказу. Роль Azure определяет тип доступа — доступ для чтения и записи, доступный только для чтения и для чтения и записи в подмножестве операций.

Для службы Azure Data Box можно определить две роли:

- **Data Box Reader** — доступ только для чтения к заказам, определенным в области. Они могут только просматривать сведения о заказе. Они не могут получить доступ к другим сведениям, связанным с учетными записями хранения, или изменить сведения о заказе, например адрес и т. д.
- **Участник Data Box** — может создать заказ для перемещения данных в указанную учетную запись хранения, *Если у них уже есть доступ на запись к учетной записи хранения*. Если у них нет доступа к учетной записи хранения, они даже не смогут создать Data Box порядок копирования данных в учетную запись. Эта роль не определяет никакие разрешения, связанные с учетной записью хранения, и не предоставляет доступ к учетным записям хранения.  

Чтобы ограничить доступ к заказу, можно:

- Назначение роли на уровне заказа. Пользователь обладает только такими разрешениями, которые определены ролями для взаимодействия только с определенным порядком Data Box и ничего другого.
- Назначение роли на уровне группы ресурсов пользователь имеет доступ ко всем Data Boxным заказам в группе ресурсов.

Дополнительные сведения о предлагаемом использовании Azure RBAC см. в статье рекомендации [по использованию Azure RBAC](../role-based-access-control/best-practices.md).

## <a name="track-the-order"></a>Отслеживание заказа

Вы можете отвести свой порядок с помощью портал Azure и на веб-сайте перевозчика, поставляющем доставку. Ниже приведены механизмы для трассировки Data Boxного порядка в любое время.

- Чтобы отвести Отслеживание заказа, когда устройство находится в центре обработки данных Azure или на локальном компьютере, перейдите к **Data Box заказа > обзор** в портал Azure.

    ![Просмотр состояния заказа и No отслеживания](media/data-box-logs/overview-view-status-1.png)

- Чтобы отвести свой порядок во время передачи устройства, перейдите на веб-сайт регионального перевозчика, например на веб-сайт UPS. Укажите номер отслеживания, связанный с заказом.
- Data Box также отправляет уведомления по электронной почте в любое время при изменении состояния заказа на основе сообщений электронной почты, указанных при создании заказа. Список всех состояний заказа Data Box см. в разделе [Просмотр состояния заказа](data-box-portal-admin.md#view-order-status). Сведения об изменении параметров уведомлений, связанных с заказом, см. в разделе [изменение сведений об уведомлении](data-box-portal-admin.md#edit-notification-details).

## <a name="query-activity-logs-during-setup"></a>Запрос журналов действий во время установки

- Ваша Data Box приступит к локальной сети в заблокированном состоянии. Для заказа можно использовать учетные данные устройства, доступные в портал Azure.  

    При настройке Data Box может потребоваться знать, кому доступны учетные данные устройства. Чтобы узнать, кто получил доступ к колонке **учетных данных устройства** , можно запросить журналы действий.  Любое действие, включающее доступ к **сведениям об устройстве > учетные данные** , заносится в журналы действий в качестве `ListCredentials` действия.

    ![Запрос журналов действий](media/data-box-logs/query-activity-log-1.png)

- Каждый вход в Data Box регистрируется в режиме реального времени. Однако эти сведения доступны только в [журналах аудита](#audit-logs) после успешного выполнения заказа.

## <a name="view-error-log-during-data-copy"></a>Просмотр журнала ошибок во время копирования данных

При копировании данных в Data Box или Data Box Heavy создается файл ошибок при возникновении каких-либо проблем с копируемыми данными.

### <a name="errorxml-file"></a>Файл Error.xml

Убедитесь, что задания копирования завершились без ошибок. Если в процессе копирования возникают ошибки, скачайте журналы со страницы **Подключение и копирование**.

- Если вы скопировали файл, который не 512 байт в соответствии с папкой управляемого диска на Data Box, файл не передается в качестве страничного BLOB-объекта в промежуточную учетную запись хранения. Вы увидите сообщение об ошибке в журналах. Удалите файл и скопируйте файл,который выровнен по размеру 512 байт.
- Если вы скопировали VHDX-или динамический виртуальный жесткий диск или разностный виртуальный жесткий диск (эти файлы не поддерживаются), вы увидите ошибку в журналах.

Ниже приведен пример *error.xml* для различных ошибок при копировании на управляемые диски.

```xml
<file error="ERROR_BLOB_OR_FILE_TYPE_UNSUPPORTED">\StandardHDD\testvhds\differencing-vhd-022019.vhd</file>
<file error="ERROR_BLOB_OR_FILE_TYPE_UNSUPPORTED">\StandardHDD\testvhds\dynamic-vhd-022019.vhd</file>
<file error="ERROR_BLOB_OR_FILE_TYPE_UNSUPPORTED">\StandardHDD\testvhds\insidefixedvhdx-022019.vhdx</file>
<file error="ERROR_BLOB_OR_FILE_TYPE_UNSUPPORTED">\StandardHDD\testvhds\insidediffvhd-022019.vhd</file>
```

Ниже приведен пример *error.xml* для различных ошибок при копировании в страничные BLOB-объекты.

```xml
<file error="ERROR_BLOB_OR_FILE_SIZE_ALIGNMENT">\PageBlob512NotAligned\File100Bytes</file>
<file error="ERROR_BLOB_OR_FILE_SIZE_ALIGNMENT">\PageBlob512NotAligned\File786Bytes</file>
<file error="ERROR_BLOB_OR_FILE_SIZE_ALIGNMENT">\PageBlob512NotAligned\File513Bytes</file>
<file error="ERROR_BLOB_OR_FILE_SIZE_ALIGNMENT">\PageBlob512NotAligned\File10Bytes</file>
<file error="ERROR_BLOB_OR_FILE_SIZE_ALIGNMENT">\PageBlob512NotAligned\File500Bytes</file>
```


Ниже приведен пример *error.xml* для различных ошибок при копировании в блочные BLOB-объекты.

```xml
<file error="ERROR_CONTAINER_OR_SHARE_NAME_LENGTH">\ab</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\invalid dns name</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_LENGTH">\morethan63charactersfortestingmorethan63charactersfortestingmorethan63charactersfortestingmorethan63charactersfortestingmorethan63charactersfortestingmorethan63charactersfortesting</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\testdirectory-~!@#$%^&amp;()_+{}</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\test__doubleunderscore</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_IMPROPER_DASH">\-startingwith-hyphen</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\Starting with Capital</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\_startingwith_underscore</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_IMPROPER_DASH">\55555555--4444--3333--2222--111111111111</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_LENGTH">\1</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\11111111-_2222-_3333-_4444-_555555555555</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_IMPROPER_DASH">\test--doublehyphen</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTI5Ni3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTMwMS3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTMwMy3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_CONTROL">\InvalidUnicodeFiles\Ã.txt</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTMwNS3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTI5OS3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTMwMi3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTMwNC3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTI5OC3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTMwMC3vv70=</file>
<file error="ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL" name_encoding="Base64">XEludmFsaWRVbmljb2RlRmlsZXNcU3BjQ2hhci01NTI5Ny3vv70=</file>
```

Ниже приведен пример *error.xml* для различных ошибок при копировании в службу файлов Azure.

```xml
<file error="ERROR_BLOB_OR_FILE_SIZE_LIMIT">\AzFileMorethan1TB\AzFile1.2TB</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\testdirectory-~!@#$%^&amp;()_+{}</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_IMPROPER_DASH">\55555555--4444--3333--2222--111111111111</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_IMPROPER_DASH">\-startingwith-hyphen</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\11111111-_2222-_3333-_4444-_555555555555</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_IMPROPER_DASH">\test--doublehyphen</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_LENGTH">\ab</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\invalid dns name</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\test__doubleunderscore</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_LENGTH">\morethan63charactersfortestingmorethan63charactersfortestingmorethan63charactersfortestingmorethan63charactersfortestingmorethan63charactersfortestingmorethan63charactersfortesting</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\_startingwith_underscore</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_LENGTH">\1</file>
<file error="ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH">\Starting with Capital</file>
```

В каждом из перечисленных выше случаев устраните ошибки, прежде чем переходить к следующему шагу. Дополнительные сведения об ошибках, полученных при копировании данных в Data Box с помощью протоколов SMB или NFS, см. в статье [Устранение неполадок Data Box и Data Box Heavy](data-box-troubleshoot.md). Сведения об ошибках, полученных при копировании данных в Data Box через остальное, см. в статье [Устранение неполадок Data Box хранилища BLOB-объектов](data-box-troubleshoot-rest.md).

## <a name="inspect-bom-during-prepare-to-ship"></a>Проверка спецификации при подготовке к отправке

Во время подготовки к отправке создается список файлов, известный как ведомость материалов (BOM) или файл манифеста.

- Используйте этот файл для проверки фактических имен и количества файлов, которые были скопированы в Data Box.
- Используйте этот файл, чтобы проверить фактические размеры файлов.
- Убедитесь, что *crc64* соответствует ненулевой строке. <!--A null value for crc64 indicates that there was a reparse point error)-->

Дополнительные сведения об ошибках, полученных при подготовке к отправке, см. в статье [Устранение неполадок Data Box и Data Box Heavy](data-box-troubleshoot.md).

### <a name="bom-or-manifest-file"></a>Файл спецификации или манифеста

Файл BOM или manifest содержит список всех файлов, которые копируются на устройство Data Box. Файл спецификации содержит имена файлов и соответствующие размеры, а также контрольную сумму. Отдельный файл спецификации создается для блочных BLOB-объектов, страничных больших двоичных объектов, файлов Azure, для копирования через интерфейсы API-интерфейсов и для копирования на управляемые диски на Data Box. Файлы спецификации можно загрузить из локального веб-интерфейса устройства во время подготовки к отправке.

Эти файлы также находятся на устройстве Data Box и передаются в связанную учетную запись хранения в центре обработки данных Azure.

### <a name="bom-file-format"></a>Формат файла спецификации

Файл спецификации или файла манифеста имеет следующий общий формат:

`<file size = "file-size-in-bytes" crc64="cyclic-redundancy-check-string">\folder-path-on-data-box\name-of-file-copied.md</file>`

Ниже приведен пример манифеста, созданного при копировании данных в общий ресурс блочного BLOB-объекта на Data Box.

```
<file size="10923" crc64="0x51c78833c90e4e3f">\databox\media\data-box-deploy-copy-data\connect-shares-file-explorer1.png</file>
<file size="15308" crc64="0x091a8b2c7a3bcf0a">\databox\media\data-box-deploy-copy-data\get-share-credentials2.png</file>
<file size="53486" crc64="0x053da912fb45675f">\databox\media\data-box-deploy-copy-data\nfs-client-access.png</file>
<file size="6093" crc64="0xadb61d0d7c6d4deb">\databox\data-box-cable-options.md</file>
<file size="6499" crc64="0x080add29add367d9">\databox\data-box-deploy-copy-data-via-nfs.md</file>
<file size="11089" crc64="0xc3ce6b13a4fe3001">\databox\data-box-deploy-copy-data-via-rest.md</file>
<file size="7749" crc64="0xd2e346a4588e307a">\databox\data-box-deploy-ordered.md</file>
<file size="14275" crc64="0x260296e5d1b1608a">\databox\data-box-deploy-copy-data.md</file>
<file size="4077" crc64="0x2bb0a170225bceec">\databox\data-box-deploy-picked-up.md</file>
<file size="15447" crc64="0xcec0ca8527720b3c">\databox\data-box-portal-admin.md</file>
<file size="9126" crc64="0x820856b5a54321ad">\databox\data-box-overview.md</file>
<file size="10963" crc64="0x5e9a14f9f4784fd8">\databox\data-box-safety.md</file>
<file size="5941" crc64="0x8631d62fbc038760">\databox\data-box-security.md</file>
<file size="12536" crc64="0x8c8ff93e73d665ec">\databox\data-box-system-requirements-rest.md</file>
<file size="3220" crc64="0x7257a263c434839a">\databox\data-box-system-requirements.md</file>
```

Файлы BOM или manifest также копируются в учетную запись хранения Azure. Вы можете использовать файлы BOM или manifest, чтобы убедиться, что файлы, отправленные в Azure, соответствуют данным, скопированным в Data Box.

## <a name="review-copy-log-during-upload-to-azure"></a>Проверка журнала копирования во время отправки в Azure

Во время передачи данных в Azure создается журнал копирования.

### <a name="copy-log"></a>Журнал копирования

Для каждого обрабатываемого заказа Data Box служба создает журнал копирования в связанной учетной записи хранения. В журнале копирования содержится общее число отправленных файлов, а также количество файлов, которые были вычислены во время копирования данных из Data Box в учетную запись хранения Azure.

Циклическая проверка избыточности (CRC) выполняется во время отправки в Azure. Кркс из копии данных и после отправки данных. Несоответствие CRC указывает, что не удалось отправить соответствующие файлы.

По умолчанию журналы записываются в контейнер `copylog`. Журналы хранятся в соответствии со следующим соглашением об именовании:

`storage-account-name/databoxcopylog/ordername_device-serial-number_CopyLog_guid.xml`.

Путь к журналу копирования также отображается в колонке **Обзор** портала.

![Путь к файлу копирования журнала в колонке обзора после завершения](media/data-box-logs/copy-log-path-1.png)

### <a name="upload-completed-successfully"></a>Отправка успешно завершена 

В следующем примере описан общий формат журнала копирования для Data Boxной отправки, которая успешно завершена:

```
<?xml version="1.0"?>
-<CopyLog Summary="Summary">
<Status>Succeeded</Status>
<TotalFiles>45</TotalFiles>
<FilesErrored>0</FilesErrored>
</CopyLog>
```

### <a name="upload-completed-with-errors"></a>Отправка завершена с ошибками 

Отправка в Azure также может завершиться с ошибками.

![Путь к файлу копирования журнала в колонке обзора при завершении с ошибками](media/data-box-logs/copy-log-path-2.png)

Ниже приведен пример журнала копирования, в котором завершилась отправка с ошибками:

```xml
<ErroredEntity Path="iso\samsungssd.iso">
  <Category>UploadErrorCloudHttp</Category>
  <ErrorCode>409</ErrorCode>
  <ErrorMessage>The blob type is invalid for this operation.</ErrorMessage>
  <Type>File</Type>
</ErroredEntity><ErroredEntity Path="iso\iSCSI_Software_Target_33.iso">
  <Category>UploadErrorCloudHttp</Category>
  <ErrorCode>409</ErrorCode>
  <ErrorMessage>The blob type is invalid for this operation.</ErrorMessage>
  <Type>File</Type>
</ErroredEntity><CopyLog Summary="Summary">
  <Status>Failed</Status>
  <TotalFiles_Blobs>72</TotalFiles_Blobs>
  <FilesErrored>2</FilesErrored>
</CopyLog>
```
### <a name="upload-completed-with-warnings"></a>Отправка завершена с предупреждениями

Отправка в Azure завершается с предупреждениями, если в данных были имена контейнеров, BLOB-объектов и файлов, которые не соответствуют соглашениям об именовании Azure, а имена были изменены для отправки данных в Azure.

![Путь к файлу копирования журнала в колонке обзора при завершении с предупреждениями](media/data-box-logs/copy-log-path-3.png)

Ниже приведен пример журнала копирования, в котором контейнеры, которые не соответствуют соглашениям об именовании Azure, были переименованы во время передачи данных в Azure.

Новые уникальные имена для контейнеров находятся в формате, `DataBox-GUID` а данные для контейнера помещаются в новый переименованный контейнер. В журнале копирования указывается старое и новое имя контейнера для контейнера.

```xml
<ErroredEntity Path="New Folder">
   <Category>ContainerRenamed</Category>
   <ErrorCode>1</ErrorCode>
   <ErrorMessage>The original container/share/blob has been renamed to: DataBox-3fcd02de-bee6-471e-ac62-33d60317c576 :from: New Folder :because either the name has invalid character(s) or length is not supported</ErrorMessage>
  <Type>Container</Type>
</ErroredEntity>
```

Ниже приведен пример журнала копирования, в котором большие двоичные объекты или файлы, которые не соответствуют соглашениям об именовании Azure, были переименованы во время передачи данных в Azure. Новые имена больших двоичных объектов или файлов преобразуются в дайджест SHA256 относительного пути к контейнеру и передаются по пути на основе целевого типа. В качестве места назначения можно использовать блочные, страничные BLOB-объекты или файлы Azure.

`copylog`Указывает старый и новый большой двоичный объект или имя файла, а также путь в Azure.

```xml
<ErroredEntity Path="TesDir028b4ba9-2426-4e50-9ed1-8e89bf30d285\Ã">
  <Category>BlobRenamed</Category>
  <ErrorCode>1</ErrorCode>
  <ErrorMessage>The original container/share/blob has been renamed to: PageBlob/DataBox-0xcdc5c61692e5d63af53a3cb5473e5200915e17b294683968a286c0228054f10e :from: Ã :because either name has invalid character(s) or length is not supported</ErrorMessage>
  <Type>File</Type>
</ErroredEntity><ErroredEntity Path="TesDir9856b9ab-6acb-4bc3-8717-9a898bdb1f8c\Ã">
  <Category>BlobRenamed</Category>
  <ErrorCode>1</ErrorCode>
  <ErrorMessage>The original container/share/blob has been renamed to: AzureFile/DataBox-0xcdc5c61692e5d63af53a3cb5473e5200915e17b294683968a286c0228054f10e :from: Ã :because either name has invalid character(s) or length is not supported</ErrorMessage>
  <Type>File</Type>
</ErroredEntity><ErroredEntity Path="TesDirf92f6ca4-3828-4338-840b-398b967d810b\Ã">
  <Category>BlobRenamed</Category>
  <ErrorCode>1</ErrorCode>
  <ErrorMessage>The original container/share/blob has been renamed to: BlockBlob/DataBox-0xcdc5c61692e5d63af53a3cb5473e5200915e17b294683968a286c0228054f10e :from: Ã :because either name has invalid character(s) or length is not supported</ErrorMessage>
  <Type>File</Type>
</ErroredEntity>
```

## <a name="get-chain-of-custody-logs-after-data-erasure"></a>Получение цепочки журналов поставок после очистки данных

После удаления данных с Data Box дисков в соответствии с рекомендациями NIST SP 800-88 версии 1, можно получить цепочку журналов поставок. Эти журналы включают в себя журналы аудита и историю заказов. Файлы BOM или manifest также копируются вместе с журналами аудита.

### <a name="audit-logs"></a>Журналы аудита

Журналы аудита содержат сведения о включении и доступе к общим папкам на Data Box или Data Box Heavy, если они находятся за пределами центра обработки данных Azure. Эти журналы находятся по адресу: `storage-account/azuredatabox-chainofcustodylogs`

Ниже приведен пример журнала аудита из Data Box.

```
9/10/2018 8:23:01 PM : The operating system started at system time ‎2018‎-‎09‎-‎10T20:23:01.497758400Z.
9/10/2018 8:23:42 PM : An account was successfully logged on.
Subject:
    Security ID:        S-1-5-18
    Account Name:       WIN-DATABOXADMIN
    Account Domain: Workgroup
    Logon ID:       0x3E7
Logon Information:
    Logon Type:     3
    Restricted Admin Mode:  -
    Virtual Account:        No
    Elevated Token:     No
Impersonation Level:        Impersonation
New Logon:
    Security ID:        S-1-5-7
    Account Name:       ANONYMOUS LOGON
    Account Domain: NT AUTHORITY
    Logon ID:       0x775D5
    Linked Logon ID:    0x0
    Network Account Name:   -
    Network Account Domain: -
    Logon GUID:     {00000000-0000-0000-0000-000000000000}
Process Information:
    Process ID:     0x4
    Process Name:       
Network Information:
    Workstation Name:   -
    Source Network Address: -
    Source Port:        -
Detailed Authentication Information:
    Logon Process:      NfsSvr
    Authentication Package:MICROSOFT_AUTHENTICATION_PACKAGE_V1_0
    Transited Services: -
    Package Name (NTLM only):   -
    Key Length:     0
This event is generated when a logon session is created. It is generated on the computer that was accessed. 
The subject fields indicate the account on the local system which requested the logon. This is most commonly a service such as the Server service, or a local process such as Winlogon.exe or Services.exe. 
The logon type field indicates the kind of logon that occurred. The most common types are 2 (interactive) and 3 (network).
The New Logon fields indicate the account for whom the new logon was created, i.e. the account that was logged on.
The network fields indicate where a remote logon request originated. Workstation name is not always available and may be left blank in some cases.
The impersonation level field indicates the extent to which a process in the logon session can impersonate.
The authentication information fields provide detailed information about this specific logon request.
    - Logon GUID is a unique identifier that can be used to correlate this event with a KDC event.
    - Transited services indicate which intermediate services have participated in this logon request.
    - Package name indicates which sub-protocol was used among the NTLM protocols.
    - Key length indicates the length of the generated session key. This will be 0 if no session key was requested.
9/10/2018 8:25:58 PM : An account was successfully logged on.
```


## <a name="download-order-history"></a>Скачивание журнала заказов

Журнал заказов доступен в портал Azure. Если заказ завершен и очистка устройства (очистки данных с дисков) завершена, перейдите к заказу устройства и перейдите к **сведениям о заказе**. Доступен параметр **Скачать журнал заказов**. Дополнительные сведения см. в статье [История заказа загрузки](data-box-portal-admin.md#download-order-history).

При прокрутке журнала заказов отображается следующее:

- Сведения об отслеживании несущей для вашего устройства.
- События с действием *секурирасе* . Эти события соответствуют очисткиу данных на диске.
- Data Box ссылки на журналы. Отображаются пути для *журналов аудита*, *журналов копирования* и файлов *BOM* .

Ниже приведен пример журнала заказа из портал Azure.

```
-------------------------------
Microsoft Data Box Order Report
-------------------------------
Name                                               : gus-poland                              
StartTime(UTC)                              : 9/19/2018 8:49:23 AM +00:00                       
DeviceType                                     : DataBox                                           
-------------------
Data Box Activities
-------------------
Time(UTC)                 | Activity                       | Status          | Description

9/19/2018 8:49:26 AM      | OrderCreated                   | Completed       |
10/2/2018 7:32:53 AM      | DevicePrepared                 | Completed       |
10/3/2018 1:36:43 PM      | ShippingToCustomer             | InProgress      | Shipment picked up. Local Time : 10/3/2018 1:36:43 PM at AMSTERDAM-NLD                                                                                
10/4/2018 8:23:30 PM      | ShippingToCustomer             | InProgress      | Processed at AMSTERDAM-NLD. Local Time : 10/4/2018 8:23:30 PM at AMSTERDAM-NLD                                                                        
10/4/2018 11:43:34 PM     | ShippingToCustomer             | InProgress      | Departed Facility in AMSTERDAM-NLD. Local Time : 10/4/2018 11:43:34 PM at AMSTERDAM-NLD
10/5/2018 8:13:49 AM      | ShippingToCustomer             | InProgress      | Arrived at Delivery Facility in BRIGHTON-GBR. Local Time : 10/5/2018 8:13:49 AM at LAMBETH-GBR                                                         
10/5/2018 9:13:24 AM      | ShippingToCustomer             | InProgress      | With delivery courier. Local Time : 10/5/2018 9:13:24 AM at BRIGHTON-GBR                                                                               
10/5/2018 12:03:04 PM     | ShippingToCustomer             | Completed       | Delivered - Signed for by. Local Time : 10/5/2018 12:03:04 PM at BRIGHTON-GBR                                                                          
1/25/2019 3:19:25 PM      | ShippingToDataCenter           | InProgress      | Shipment picked up. Local Time : 1/25/2019 3:19:25 PM at BRIGHTON-GBR                                                                                       
1/25/2019 8:03:55 PM      | ShippingToDataCenter           | InProgress      | Processed at BRIGHTON-GBR. Local Time : 1/25/2019 8:03:55 PM at LAMBETH-GBR                                                                            
1/25/2019 8:04:58 PM      | ShippingToDataCenter           | InProgress      | Departed Facility in BRIGHTON-GBR. Local Time : 1/25/2019 8:04:58 PM at BRIGHTON-GBR                                                                    
1/25/2019 9:06:09 PM      | ShippingToDataCenter           | InProgress      | Arrived at Sort Facility LONDON-HEATHROW-GBR. Local Time : 1/25/2019 9:06:09 PM at LONDON-HEATHROW-GBR                                                
1/25/2019 9:48:54 PM      | ShippingToDataCenter           | InProgress      | Processed at LONDON-HEATHROW-GBR. Local Time : 1/25/2019 9:48:54 PM at LONDON-HEATHROW-GBR                                                            
1/25/2019 10:30:20 PM     | ShippingToDataCenter           | InProgress      | Departed Facility in LONDON-HEATHROW-GBR. Local Time : 1/25/2019 10:30:20 PM at LONDON-HEATHROW-GBR
1/28/2019 7:11:35 AM      | ShippingToDataCenter           | InProgress      | Arrived at Delivery Facility in AMSTERDAM-NLD. Local Time : 1/28/2019 7:11:35 AM at AMSTERDAM-NLD                                                     
1/28/2019 9:07:57 AM      | ShippingToDataCenter           | InProgress      | With delivery courier. Local Time : 1/28/2019 9:07:57 AM at AMSTERDAM-NLD                                                                             
1/28/2019 1:35:56 PM      | ShippingToDataCenter           | InProgress      | Scheduled for delivery. Local Time : 1/28/2019 1:35:56 PM at AMSTERDAM-NLD                                                                            
1/28/2019 2:57:48 PM      | ShippingToDataCenter           | Completed       | Delivered - Signed for by. Local Time : 1/28/2019 2:57:48 PM at AMSTERDAM-NLD
1/29/2019 2:18:43 PM      | PhysicalVerification           | Completed       |
1/29/2019 3:49:50 PM      | DeviceBoot                     | Completed       | Appliance booted up successfully.
1/29/2019 3:49:51 PM      | AnomalyDetection               | Completed       | No anomaly detected.
2/12/2019 10:37:03 PM     | DataCopy                       | Resumed         |
2/13/2019 12:05:15 AM     | DataCopy                       | Resumed         |
2/15/2019 7:07:34 PM      | DataCopy                       | Completed       | Copy Completed.
2/15/2019 7:47:32 PM      | SecureErase                    | Started         |
2/15/2019 8:01:10 PM      | SecureErase                    | Completed       | Azure Data Box:<Device-serial-no> has been sanitized according to NIST 800-88 Rev 1.
------------------
Data Box Log Links
------------------
Account Name         : gusacct
Copy Logs Path       : databoxcopylog/gus-poland_<Device-serial-no>_CopyLog_<GUID>.xml
Audit Logs Path      : azuredatabox-chainofcustodylogs\<GUID>\<Device-serial-no>
BOM Files Path       : azuredatabox-chainofcustodylogs\<GUID>\<Device-serial-no>
```

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте, как [устранять неполадки Data Box и Data Box Heavy](data-box-troubleshoot.md).
