---
title: Восстановление задания экспорта инструмента импорта и экспорта Azure версии 1 | Документация Майкрософт
description: Узнайте, как восстановить задание экспорта, созданное и выполняемое с помощью службы импорта и экспорта Azure.
author: alkohli
services: storage
ms.service: storage
ms.topic: how-to
ms.date: 01/19/2021
ms.author: alkohli
ms.subservice: common
ms.openlocfilehash: d84f26b2764a103a9b504c1480e88b58fed3c201
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98706725"
---
# <a name="repairing-an-export-job"></a>Исправление задания экспорта
После завершения задания экспорта вы можете локально выполнить следующие задачи с помощью инструмента импорта и экспорта Microsoft Azure:  
  
1.  загрузить любые файлы, которые служба импорта и экспорта Azure не смогла экспортировать;  
  
2.  проверить правильность экспорта файлов с диска.  
  
Для использования этой возможности требуется подключение к службе хранилища Azure.  
  
Команда для исправления задания импорта: **RepairExport**.

## <a name="repairexport-parameters"></a>Параметры RepairExport

Для команды **RepairExport** можно определить следующие параметры.  
  
|Параметр|Описание|  
|---------------|-----------------|  
|**/r:<файл_восстановления\>**|Обязательный. Путь к файлу восстановления для отслеживания восстановления и возобновления прерванной операции восстановления. Каждый диск должен быть связан с одним и только одним файлом восстановления. При запуске восстановления для данного диска путь передается в файл восстановления, который еще не существует. Чтобы возобновить прерванную операцию восстановления, необходимо передать имя существующего файла восстановления. Всегда указывайте файл восстановления, соответствующий целевому диску.|  
|**/logdir:<каталог_журнала\>**|Необязательный параметр. Каталог журналов. В этот каталог записываются файлы подробного журнала. Если каталог журнала не указан, вместо него будет использоваться текущий каталог.|  
|**/d:<целевой_каталог\>**|Обязательный. Каталог, который нужно проверить и восстановить. Этот каталог обычно является корневым каталогом экспортируемого диска, но также может быть общей сетевой папкой, содержащей копию экспортированных файлов.|  
|**/BK: <ключ BitLocker\>**|Необязательный параметр. Укажите ключ BitLocker, если необходимо, чтобы средство разблокировало зашифрованное расположение, где хранятся экспортированные файлы.|  
|**/SN: <StorageAccountName\>**|Обязательный. Имя учетной записи хранения для задания экспорта.|  
|**/SK: <StorageAccountKey\>**|**Требуется** только в том случае, если не указан SAS контейнера. Ключ учетной записи хранения для задания экспорта.|  
|**/csas: <Контаинерсас\>**|**Требуется** только в том случае, если не указан ключ учетной записи хранения. Контейнер SAS для доступа к большим двоичным объектам, связанным с заданием экспорта.|  
|**/CopyLogFile:<файл_журнала\>**|Обязательный. Путь к файлу журнала копирования диска. Этот файл создается службой импорта и экспорта Windows Azure. Его можно скачать из хранилища BLOB-объектов, связанного с заданием. Файл журнала копирования содержит сведения о сбое больших двоичных объектов или восстанавливаемых файлов.|  
|**/ ManifestFile:<файл_манифеста_диска\>**|Необязательный параметр. Путь к файлу манифеста экспортируемого диска. Этот файл создается службой импорта и экспорта Microsoft Azure и хранится на диске экспорта. При необходимости в большом двоичном объекте в учетной записи хранения, связанной с заданием.<br /><br /> Содержимое файлов на экспортируемом диске проверяется с помощью MD5-хэшей, содержащихся в этом файле. Все поврежденные файлы будут скачаны и перезаписаны в целевые каталоги.|  
  
## <a name="using-repairexport-mode-to-correct-failed-exports"></a>Использование режима RepairExport для исправления проблем, возникших при экспорте  
С помощью инструмента импорта и экспорта Azure вы можете скачать файлы, которые не удалось экспортировать. Файл журнала копирования будет содержать список файлов, которые не удалось экспортировать.  
  
Среди прочего, возможны следующие причины проблем при экспорте:  
  
-   повреждение дисков;  
  
-   изменение ключа учетной записи хранения во время передачи.  
  
Чтобы запустить средство в режиме **RepairExport**, сначала необходимо подключить диск, содержащий экспортированные файлы, к компьютеру. Затем запустите инструмент импорта и экспорта Azure, указав путь к этому диску с помощью параметра `/d`. Также необходимо указать путь к загруженному файлу журнала копирования диска. Приведенный ниже пример командной строки запускает средство для восстановления всех файлов, которые не удалось экспортировать:  
  
```  
WAImportExport.exe RepairExport /r:C:\WAImportExport\9WM35C3U.rep /d:G:\ /sn:bobmediaaccount /sk:VkGbrUqBWLYJ6zg1m29VOTrxpBgdNOlp+kp0C9MEdx3GELxmBw4hK94f7KysbbeKLDksg7VoN1W/a5UuM2zNgQ== /CopyLogFile:C:\WAImportExport\9WM35C3U.log  
```  
  
В следующем примере показан файл журнала копирования, в котором показано, что не удалось экспортировать один блок в BLOB-объекте:  
  
```xml
<?xml version="1.0" encoding="utf-8"?>  
<DriveLog>  
  <DriveId>9WM35C2V</DriveId>  
  <Blob Status="CompletedWithErrors">  
    <BlobPath>pictures/wild/desert.jpg</BlobPath>  
    <FilePath>\pictures\wild\desert.jpg</FilePath>  
    <LastModified>2012-09-18T23:47:08Z</LastModified>  
    <Length>163840</Length>  
    <BlockList>  
      <Block Offset="65536" Length="65536" Id="AQAAAA==" Status="Failed" />  
    </BlockList>  
  </Blob>  
  <Status>CompletedWithErrors</Status>  
</DriveLog>  
```  
  
Файл журнала копирования указывает на сбой при загрузке службой импорта и экспорта Windows Azure одного из блоков большого двоичного объекта в файл на экспортируемом диске. Остальные компоненты файла загружены успешно, и длина файла установлена правильно. В этом случае средство откроет файл на диске, загрузит блок из учетной записи хранения и запишет этот блок длиной 65536 в файл, начиная со смещения 65536.  
  
## <a name="using-repairexport-to-validate-drive-contents"></a>Использование RepairExport для проверки содержимого диска  
Вы также можете использовать службу импорта и экспорта Azure с параметром **RepairExport**, чтобы проверить правильность содержимого на диске. Файл манифеста на каждом экспортируемом диске содержит MD5-хэши всего содержимого диска.  
  
Служба импорта и экспорта Azure может в процессе экспорта сохранять эти файлы манифеста в учетную запись хранения. Расположение файлов манифеста можно получить после завершения задания с помощью команды [получения задания](/rest/api/storageimportexport/jobs). Дополнительные сведения о формате файла манифеста диска см. в разделе [Формат файла манифеста службы импорта и экспорта](/previous-versions/azure/storage/common/storage-import-export-file-format-metadata-and-properties).  
  
В следующем примере показано, как запустить инструмент импорта и экспорта Azure с параметрами **/ManifestFile** и **/CopyLogFile**.  
  
```  
WAImportExport.exe RepairExport /r:C:\WAImportExport\9WM35C3U.rep /d:G:\ /sn:bobmediaaccount /sk:VkGbrUqBWLYJ6zg1m29VOTrxpBgdNOlp+kp0C9MEdx3GELxmBw4hK94f7KysbbeKLDksg7VoN1W/a5UuM2zNgQ== /CopyLogFile:C:\WAImportExport\9WM35C3U.log /ManifestFile:G:\9WM35C3U.manifest  
```  
  
В следующем примере показан файл манифеста:  
  
```xml
<?xml version="1.0" encoding="utf-8"?>  
<DriveManifest Version="2011-10-01">  
  <Drive>  
    <DriveId>9WM35C3U</DriveId>  
    <ClientCreator>Windows Azure Import/Export service</ClientCreator>  
    <BlobList>
      <Blob>  
        <BlobPath>pictures/city/redmond.jpg</BlobPath>  
        <FilePath>\pictures\city\redmond.jpg</FilePath>  
        <Length>15360</Length>  
        <PageRangeList>  
          <PageRange Offset="0" Length="3584" Hash="72FC55ED9AFDD40A0C8D5C4193208416" />  
          <PageRange Offset="3584" Length="3584" Hash="68B28A561B73D1DA769D4C24AA427DB8" />  
          <PageRange Offset="7168" Length="512" Hash="F521DF2F50C46BC5F9EA9FB787A23EED" />  
        </PageRangeList>  
        <PropertiesPath Hash="E72A22EA959566066AD89E3B49020C0A">\pictures\city\redmond.jpg.properties</PropertiesPath>  
      </Blob>  
      <Blob>  
        <BlobPath>pictures/wild/canyon.jpg</BlobPath>  
        <FilePath>\pictures\wild\canyon.jpg</FilePath>  
        <Length>10884</Length>  
        <BlockList>  
          <Block Offset="0" Length="2721" Id="AAAAAA==" Hash="263DC9C4B99C2177769C5EBE04787037" />  
          <Block Offset="2721" Length="2721" Id="AQAAAA==" Hash="0C52BAE2CC20EFEC15CC1E3045517AA6" />  
          <Block Offset="5442" Length="2721" Id="AgAAAA==" Hash="73D1CB62CB426230C34C9F57B7148F10" />  
          <Block Offset="8163" Length="2721" Id="AwAAAA==" Hash="11210E665C5F8E7E4F136D053B243E6A" />  
        </BlockList>  
        <PropertiesPath Hash="81D7F81B2C29F10D6E123D386C3A4D5A">\pictures\wild\canyon.jpg.properties</PropertiesPath>  
      </Blob> 
    </BlobList>  
 </Drive>  
</DriveManifest>  
``` 
  
Завершив процесс восстановления, средство считывает каждый файл, указанный в файле манифеста, и проверяет целостность этих файлов с помощью MD5-хэшей. Если использовать указанный выше манифест, будут проверены следующие компоненты.  

```  
G:\pictures\city\redmond.jpg, offset 0, length 3584  
  
G:\pictures\city\redmond.jpg, offset 3584, length 3584  
  
G:\pictures\city\redmond.jpg, offset 7168, length 3584  
  
G:\pictures\city\redmond.jpg.properties  
  
G:\pictures\wild\canyon.jpg, offset 0, length 2721  
  
G:\pictures\wild\canyon.jpg, offset 2721, length 2721  
  
G:\pictures\wild\canyon.jpg, offset 5442, length 2721  
  
G:\pictures\wild\canyon.jpg, offset 8163, length 2721  
  
G:\pictures\wild\canyon.jpg.properties  
```

Средство загрузит заново любой компонент, который не прошел проверку, и запишет его в тот же файл на диске.  
  
## <a name="next-steps"></a>Дальнейшие действия
 
* [Настройка средства импорта и экспорта Azure](storage-import-export-tool-setup-v1.md)   
* [Подготовка жестких дисков для задания импорта](storage-import-export-data-to-blobs.md#step-1-prepare-the-drives)   
* [Просмотр состояния задания с помощью файлов журнала копирования](storage-import-export-tool-reviewing-job-status-v1.md)   
* [Подготовка задания импорта](storage-import-export-tool-repairing-an-import-job-v1.md)