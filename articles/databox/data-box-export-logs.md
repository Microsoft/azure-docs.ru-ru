---
title: Мониторинг и Azure Data Box журналов, Azure Data Box Heavy событий для экспорта заказов | Документация Майкрософт
description: Описание процесса трассировки и регистрации событий на различных стадиях Azure Data Box и Azure Data Box Heavy порядка экспорта.
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: article
ms.date: 07/10/2020
ms.author: alkohli
ms.openlocfilehash: 3a915ac8de83a5e183660ec4a3d05044eafff4a9
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94337514"
---
# <a name="tracking-and-event-logging-for-your-azure-data-box-and-azure-data-box-heavy-export-orders"></a>Отслеживание и ведение журнала событий для Azure Data Box и Azure Data Box Heavy экспорт заказов

Порядок экспорта Data Box или Data Box Heavy проходит следующие шаги: порядок, Настройка, копирование данных, возврат и очистки данных. В соответствии с каждым шагом в заказе можно выполнить несколько действий по управлению доступом к заказу, аудиту событий, отслеживанию порядка и интерпретации различных формируемых журналов.

В этой статье подробно описаны различные механизмы или средства, доступные для мониторинга и аудита экспортных заказов для Data Box или Data Box Heavy. Сведения в этой статье применимы как к, так Data Box и Data Box Heavy. В последующих разделах все ссылки на Data Box также применяются к Data Box Heavy.

В следующей таблице приведены сводные сведения о шагах порядка экспорта Data Box и средствах, доступных для трассировки и аудита заказа на каждом шаге.

| Стадия порядка экспорта Data Box       | Средство для отслеживания и аудита                                                                        |
|----------------------------|------------------------------------------------------------------------------------------------|
| Создание заказа               | [Настройка контроля доступа в порядке с помощью Azure RBAC](#set-up-access-control-on-the-order) <br> [Включить подробный журнал в указанном порядке](#enable-verbose-log-in-the-order)                                                    |
| Заказ обработан            | [Отслеживание порядка](#track-the-order) по <ul><li> портала Azure; </li><li> веб-сайта оператора доставки; </li><li>уведомления по электронной почте.</ul> |
| Настройка устройства              | [Журналы действий](#query-activity-logs-during-setup) доступа к учетным данным устройства              |
| Копирование данных с устройства        | [Проверка журналов копирования](#copy-log) <br> [Ознакомьтесь с подробными журналами](#verbose-log) перед копированием данных            |
| Удаление данных с устройства   | [Просмотр цепочки журналов поставок](#get-chain-of-custody-logs-after-data-erasure) , включая журналы аудита и журнал заказов                |


## <a name="set-up-access-control-on-the-order"></a>Настройка контроля доступа в заказе

Вы можете контролировать, кто имеет доступ к вашему заказу при первом создании заказа. Настройте роли Azure в различных областях для управления доступом к Data Boxму заказу. Роль Azure определяет тип доступа — доступ для чтения и записи, доступный только для чтения и для чтения и записи в подмножестве операций.

Для службы Azure Data Box можно определить две роли:

- **Data Box Reader** — доступ только для чтения к заказам, определенным в области. Они могут только просматривать сведения о заказе. Они не могут получить доступ к другим сведениям, связанным с учетными записями хранения, или изменить сведения о заказе, например адрес и т. д.
- **Участник Data Box** — может создать заказ для перемещения данных в указанную учетную запись хранения, *Если у них уже есть доступ на запись к учетной записи хранения*. Если у них нет доступа к учетной записи хранения, они даже не смогут создать Data Box порядок копирования данных в учетную запись. Эта роль не определяет никакие разрешения, связанные с учетной записью хранения, и не предоставляет доступ к учетным записям хранения.  

Чтобы ограничить доступ к заказу, можно:

- Назначение роли на уровне заказа. Пользователь обладает только такими разрешениями, которые определены ролями для взаимодействия только с определенным порядком Data Box и ничего другого.
- Назначение роли на уровне группы ресурсов пользователь имеет доступ ко всем Data Boxным заказам в группе ресурсов.

Дополнительные сведения о предлагаемом использовании Azure RBAC см. в статье рекомендации [по использованию Azure RBAC](../role-based-access-control/best-practices.md).

## <a name="enable-verbose-log-in-the-order"></a>Включить подробный журнал в указанном порядке

При размещении заказа на экспорт для Data Box можно включить сбор подробных журналов. Вот экран заказа, где можно включить подробный журнал:

![Выбор параметра экспорта](media/data-box-deploy-export-ordered/azure-data-box-export-order-export-option.png)

При выборе параметра **включить подробный журнал** создается подробный файл журнала при копировании данных из учетной записи хранения Azure. Этот журнал содержит список всех файлов, которые были успешно экспортированы.

Дополнительные сведения о порядке экспорта см. в разделе [создание порядка экспорта для Data Box](data-box-deploy-export-ordered.md)

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

- Каждый вход в Data Box регистрируется в режиме реального времени. Однако эти сведения доступны только в [цепочке журналов аудита поставок](#chain-of-custody-audit-logs) после успешного завершения заказа.

## <a name="view-logs-during-data-copy"></a>Просмотр журналов во время копирования данных

Перед копированием данных из Data Box можно скачать и проверить *Журнал копирования* и *подробный журнал* для данных, скопированных в Data Box. Эти журналы создаются при копировании данных из учетной записи хранения в Azure в Data Box. 

### <a name="copy-log"></a>Журнал копирования

Перед копированием данных из Data Box Скачайте журнал копирования на странице **подключение и копирование** .

Ниже приведен пример выходных данных *журнала копирования* в случае отсутствия ошибок и копирования всех файлов во время копирования данных из Azure в Data Box устройство.

```output
<CopyLog Summary="Summary">
  <Status>Succeeded</Status>
  <TotalFiles_Blobs>5521</TotalFiles_Blobs>
  <FilesErrored>0</FilesErrored>
</CopyLog>
``` 
    
Ниже приведен пример выходных данных, когда *Журнал копирования* содержит ошибки, а некоторые файлы не удалось скопировать из Azure.

```output
<ErroredEntity CloudFormat="AppendBlob" Path="export-ut-appendblob/wastorage.v140.3.0.2.nupkg">
  <Category>UploadErrorCloudHttp</Category>
  <ErrorCode>400</ErrorCode>
  <ErrorMessage>UnsupportBlobType</ErrorMessage>
  <Type>File</Type>
</ErroredEntity><ErroredEntity CloudFormat="AppendBlob" Path="export-ut-appendblob/xunit.console.Primary_2020-05-07_03-54-42-PM_27444.hcsml">
  <Category>UploadErrorCloudHttp</Category>
  <ErrorCode>400</ErrorCode>
  <ErrorMessage>UnsupportBlobType</ErrorMessage>
  <Type>File</Type>
</ErroredEntity><ErroredEntity CloudFormat="AppendBlob" Path="export-ut-appendblob/xunit.console.Primary_2020-05-07_03-54-42-PM_27444 (1).hcsml">
  <Category>UploadErrorCloudHttp</Category>
  <ErrorCode>400</ErrorCode>
  <ErrorMessage>UnsupportBlobType</ErrorMessage>
  <Type>File</Type>
</ErroredEntity><CopyLog Summary="Summary">
  <Status>Failed</Status>
  <TotalFiles_Blobs>4</TotalFiles_Blobs>
  <FilesErrored>3</FilesErrored>
</CopyLog>    
```

Доступны следующие варианты экспорта этих файлов: 

- Вы можете передать файлы, которые не удалось скопировать по сети. 
- Если размер данных превышает объем используемой емкости устройства, выполняется частичное копирование. Все файлы, которые не были скопированы, перечисляются в этом журнале. Этот журнал можно использовать в качестве входного XML-файла, чтобы создать новый заказ Data Box, а затем скопировать эти файлы.

### <a name="verbose-log"></a>Подробный журнал

*Подробный журнал* содержит список всех файлов, успешно экспортированных из учетной записи хранения Azure. Кроме того, журнал содержит размер файла и вычисление контрольной суммы.

Подробный журнал содержит сведения в следующем формате:

`<file size = "file-size-in-bytes" crc64="cyclic-redundancy-check-string">\folder-path-on-data-box\name-of-file-copied.md</file>`

Ниже приведен пример выходных данных подробного журнала. 

```powershell
  <File CloudFormat="BlockBlob" Path="validblobdata/test1.2.3.4" Size="1024" crc64="7573843669953104266">
  </File><File CloudFormat="BlockBlob" Path="validblobdata/helloEndWithDot..txt" Size="11" crc64="7320094093915972193">
  </File><File CloudFormat="BlockBlob" Path="validblobdata/test..txt" Size="12" crc64="17906086011702236012">
  </File><File CloudFormat="BlockBlob" Path="validblobdata/test1" Size="1024" crc64="7573843669953104266">
  </File><File CloudFormat="BlockBlob" Path="validblobdata/test1.2.3" Size="1024" crc64="7573843669953104266">
  </File><File CloudFormat="BlockBlob" Path="validblobdata/.......txt" Size="11" crc64="7320094093915972193">
  </File><File CloudFormat="BlockBlob" Path="validblobdata/copylogb08fa3095564421bb550d775fff143ed====..txt" Size="53638" crc64="1147139997367113454">
  </File><File CloudFormat="BlockBlob" Path="validblobdata/testmaxChars-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-123456790-12345679" Size="1024" crc64="7573843669953104266">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/file0" Size="0" crc64="0">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/file1" Size="0" crc64="0">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/file4096_000001" Size="4096" crc64="16969371397892565512">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/file4096_000000" Size="4096" crc64="16969371397892565512">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/64KB-Seed10.dat" Size="65536" crc64="10746682179555216785">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/LiveSiteReport_Oct.xlsx" Size="7028" crc64="6103506546789189963">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/NE_Oct_GeoReport.xlsx" Size="103197" crc64="13305485882546035852">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/64KB-Seed1.dat" Size="65536" crc64="3140622834011462581">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/1mbfiles-0-0" Size="1048576" crc64="16086591317856295272">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/file524288_000001" Size="524288" crc64="8908547729214703832">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/4mbfiles-0-0" Size="4194304" crc64="1339017920798612765">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/file524288_000000" Size="524288" crc64="8908547729214703832">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/8mbfiles-0-1" Size="8388608" crc64="3963298606737216548">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/1mbfiles-0-1" Size="1048576" crc64="11061759121415905887">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/XLS-10MB.xls" Size="1199104" crc64="2218419493992437463">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/8mbfiles-0-0" Size="8388608" crc64="1072783424245035917">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/4mbfiles-0-1" Size="4194304" crc64="9991307204216370812">
  </File><File CloudFormat="BlockBlob" Path="export-ut-container/VL_Piracy_Negtive10_TPNameAndGCS.xlsx" Size="12398699" crc64="13526033021067702820">
  </File>
  ``````

Подробные журналы также копируются в учетную запись хранения Azure. По умолчанию журналы записываются в контейнер `copylog`. Журналы хранятся в соответствии со следующим соглашением об именовании:

`storage-account-name/databoxcopylog/ordername_device-serial-number_CopyLog_guid.xml`.

Путь к журналу копирования также отображается в колонке **Обзор** портала.

<!-- add a screenshot-->

Эти журналы можно использовать, чтобы убедиться, что файлы, скопированные из Azure, соответствуют данным, скопированным на локальный сервер. 

Используйте подробный файл журнала:

- Для проверки фактических имен и количества файлов, скопированных из Data Box.
- Для проверки на соответствие фактическим размерам файлов.
- Чтобы убедиться, что *crc64* соответствует ненулевой строке. Циклическая проверка избыточности (CRC) выполняется во время экспорта из Azure. Кркс из экспорта и после копирования данных из Data Box на локальный сервер можно сравнивать. Несоответствие CRC указывает, что соответствующие файлы не удалось скопировать должным образом.


## <a name="get-chain-of-custody-logs-after-data-erasure"></a>Получение цепочки журналов поставок после очистки данных

После удаления данных с Data Box дисков в соответствии с рекомендациями NIST SP 800-88 версии 1, можно получить цепочку журналов поставок. Эти журналы включают цепочку журналов аудита поставок и истории заказов. Файлы BOM или manifest также копируются вместе с журналами аудита.

### <a name="chain-of-custody-audit-logs"></a>Цепочка журналов аудита поставок

Цепочка журналов аудита поставок содержит сведения о включении и доступе к общим папкам на Data Box или Data Box Heavy, когда она находится за пределами центра обработки данных Azure. Эти журналы находятся по адресу: `storage-account/azuredatabox-chainofcustodylogs`

Ниже приведен пример журнала аудита из Data Box.

```output
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
