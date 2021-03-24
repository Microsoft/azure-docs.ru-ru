---
title: Восстановление с помощью средства создания моментальных снимков с единообразием приложений Azure для Azure NetApp Files | Документация Майкрософт
description: Содержит инструкции по запуску команды Restore для средства создания моментальных снимков с единообразием приложений Azure, которые можно использовать с Azure NetApp Files.
services: azure-netapp-files
documentationcenter: ''
author: Phil-Jensen
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 12/14/2020
ms.author: phjensen
ms.openlocfilehash: 793b4da8fcf46ba4d5618f8ada86f9c3c8026ffd
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104865270"
---
# <a name="restore-using-azure-application-consistent-snapshot-tool-preview"></a>Восстановление с помощью инструмента моментального снимка совместимости приложений Azure (Предварительная версия)

В этой статье представлено руководство по запуску команды Restore в средстве создания моментальных снимков с помощью приложений Azure, которые можно использовать с Azure NetApp Files.

## <a name="introduction"></a>Введение

Восстановление тома из моментального снимка выполняется с помощью `azacsnap -c restore` команды.

> [!IMPORTANT]
> Это не выполняет восстановление базы данных, а только восстановление томов, как описано для каждого из параметров ниже.

## <a name="command-options"></a>Параметры команды

`-c restore`Команда имеет следующие параметры.

- `--restore snaptovol` Создает новый том на основе последнего моментального снимка на целевом томе. Эта команда создает новый "клонированный" том на основе настроенного целевого тома, используя последний моментальный снимок тома в качестве основы для создания нового тома.  Эта команда не прерывает репликацию хранилища с первичной на вторичную. Вместо этого на сайте аварийного восстановления создаются клоны последнего доступного моментального снимка, и рекомендуется точки подключения файловая система клонированных томов. Эта команда должна выполняться в системе крупных экземпляров Azure **в регионе аварийного восстановления** (то есть в целевой системе с отработкой отказа).

- `--restore revertvolume` Возврат целевого тома к предыдущему состоянию на основе последнего моментального снимка.  Использование этой команды в рамках отработки отказа аварийного восстановления в парный регион аварийного восстановления. Эта команда **останавливает** репликацию хранилища с первичного сайта на вторичный сайт и отменяет их до последнего доступного моментального снимка на томах аварийного восстановления вместе с рекомендуемой файловой системой точки подключения для восстановленных томов аварийного восстановления. Эта команда должна выполняться в системе крупных экземпляров Azure **в регионе аварийного восстановления** (то есть в целевой системе с отработкой отказа).
    > [!NOTE]
    > Подкоманда ( `--restore revertvolume` ) доступна только для крупных экземпляров Azure и недоступна для Azure NetApp Files.
- `--dbsid <SAP HANA SID>` — Это SAP HANA SID, выбираемый из файла конфигурации для применения команд восстановления тома.

- `[--configfile <config filename>]` — Необязательный параметр, который позволяет использовать пользовательские имена файлов конфигурации.

## <a name="perform-a-test-dr-failover-azacsnap--c-restore---restore-snaptovol"></a>Выполнение тестовой отработки отказа аварийного восстановления `azacsnap -c restore --restore snaptovol`

Эта команда похожа на полную команду отработки отказа АВАРИЙного восстановления ( `--restore restorevolume` ), а не на прерывание репликации между первичным сайтом и сайтом аварийного восстановления. создается том клонирования, который обеспечивает восстановление последнего моментального снимка на сайте аварийного восстановления. Эти клонированные тома затем могут использоваться клиентом для тестирования аварийного восстановления без выполнения полной отработки отказа своей среды HANA, которая прерывает соглашение о репликации между первичным сайтом и сайтом аварийного восстановления.

- Можно проверить несколько различных точек восстановления таким образом, каждая из которых имеет собственную точку восстановления.
- Клон обозначается меткой времени в момент выполнения команды и представляет последние данные и другой моментальный снимок, доступный при запуске.

> [!IMPORTANT]
> Эта операция применяется только к крупному экземпляру Azure.
>
> - При выполнении этой команды требуется контактный адрес электронной почты для операций, лиаисе до удаления клонов через 4 недели.
> - При каждом выполнении этой команды будет создан новый клон, который должен быть удален операциями Майкрософт после выполнения теста.
> - Все созданные тома клонов будут автоматически удалены через 4 недели.

Файл конфигурации (например, `DR.json` ) должен содержать только тома аварийного восстановления, а не рабочие тома, в противном случае для рабочих томов могут быть созданы клоны.

### <a name="output-of-the-azacsnap--c-restore---restore-snaptovol-command-for-single-node-scenario"></a>Выходные данные `azacsnap -c restore --restore snaptovol` команды (для сценария Single-Node)

```output
> azacsnap --configfile DR.json -c restore --restore snaptovol --dbsid H80
* This program is designed for those customers who have previously installed the
  Production HANA instance in the Disaster Recovery Location either as a
  stand-alone instance or as part of a multi-purpose environment.
* This program should be executed from the Disaster Recovery location otherwise
  unintended consequences may result.
* This program is intended to allow the customer to simulate a Disaster Recovery
  failover without actually requiring a failover and subsequent failback.
* Any other restore points must be handled by Microsoft Operations.
* As part of the process, a clone is created of the each of the 'data' and 'other'
  volumes per the configuration file.

Do you wish to continue? (y/n) [n]: y

About to create clones of volumes based on the latest snapshot, these will be
kept for 4 weeks before being automatically deleted by Microsoft Operations.
Enter an email address to contact when deleting clones: <b>person@nowhere.com</b>
Checking state of HLI volumes for SID 'PEW'
Configured volumes (Data and Other) are not ready to clone, will retry in 00:00:10 seconds
Configured volumes (Data and Other) are not ready to clone, will retry in 00:00:10 seconds
Configured volumes (Data and Other) are not ready to clone, will retry in 00:00:10 seconds
Configured volumes (Data and Other) are not ready to clone, will retry in 00:00:10 seconds
Configured volumes (Data and Other) are not ready to clone, will retry in 00:00:10 seconds
Configured volumes (Data and Other) are not ready to clone, will retry in 00:00:10 seconds
Configured volumes (Data and Other) are not ready to clone, will retry in 00:00:10 seconds
Displaying Mount Points by Volume as follows:
10.50.251.34:/hana_data_h80_sapprdhdb80_mnt00001_t020_xdp_rwclone_20200916_0256  /hana/data/H80/mnt00001 nfs  rw,bg,hard,timeo=600,vers=4,rsize=1048576,wsize=1048576,intr,noatime,lock 0 0
10.50.251.36:/hana_log_backups_h80_sapprdhdb80_t020_xdp_rwclone_20200916_0256  /hana/log_backups/H80/01 nfs  rw,bg,hard,timeo=600,vers=4,rsize=1048576,wsize=1048576,intr,noatime,lock 0 0
*******************  HANA Test DR Restore Steps  ******************************
* Complete the following steps to recover your HANA database:           *
* 1. Ensure ALL the target mount points exist to mount the snapshot clones.    *
*    e.g. mkdir /hana/logbackups/H99_SOURCE                                    *
* 2. Add Mount Point Details from 'Displaying Mount Points by Volume' as       *
*    output above into /etc/fstab of DR Server.                                *
* 3. Mount newly added filesystems.                                            *
* 4. Perform HANA Snapshot Recovery using HANA Studio.                         *
********************************************************************************
*  These snapshot copies (clones) are kept for 4 weeks before                  *
*  being automatically removed.                                                *
*  Please contact Microsoft Operations to delete them earlier.                 *
********************************************************************************
```

> [!IMPORTANT]
> Выходные данные "Отображение точек подключения по томам" отличаются в различных сценариях.

## <a name="perform-full-dr-failover-azacsnap--c-restore---restore-revertvolume"></a>Выполнение полной отработки отказа аварийного восстановления `azacsnap -c restore --restore revertvolume`

Эта команда **останавливает** репликацию хранилища с первичного сайта на вторичный сайт, восстанавливает последний моментальный снимок на томах аварийного восстановления и предоставляет точки подключения для томов аварийного восстановления.

Эта команда должна выполняться на сервере аварийного восстановления с помощью файла конфигурации (например, `DR.json` ) только с томами аварийного восстановления.

Выполните отработку отказа на сайт аварийного восстановления, выполнив команду `azacsnap -c restore --restore revertvolume` .  Для этой команды требуется добавить идентификатор безопасности в качестве параметра. Это идентификатор безопасности экземпляра HANA, который необходимо восстановить на сайте аварийного восстановления.

> [!IMPORTANT]
> Эту команду следует выполнять только в том случае, если вы планируете выполнить упражнение аварийного восстановления или проверку. Эта команда прерывает репликацию. Для повторного включения репликации необходимо обратиться в Microsoft Operations.

На высоком уровне ниже приведены шаги для выполнения отработки отказа аварийного восстановления.

- Необходимо завершить работу экземпляра HANA на **первичном** сайте. Это действие требуется только в том случае, если вы действительно выполняете отработку отказа на сайт аварийного восстановления, чтобы избежать несогласованности данных.
- Завершите работу экземпляра HANA на узле аварийного восстановления для рабочего идентификатора безопасности.
- Выполните команду `azacsnap -c restore --restore revertvolume` на узле аварийного восстановления с ИД безопасности для восстановления.
  - Команда разбивает канал репликации хранилища с основного сайта на сайт аварийного восстановления.
  - Команда восстанавливает тома "Data" и "Other" в соответствии с настройками.  Как правило, эта операция выполняется для томов `/hana/data` файловой системы и `/hana/logbackups` .  `/hana/shared`Файловая система не восстанавливается, а вместо нее использует существующий `/hana/shared` идентификатор SID в расположении Dr.
  - Подключение `/hana/data` томов и `/hana/logbackups` — Убедитесь, что они добавлены в `/etc/fstab` файл.
- Восстановите моментальный снимок HANA СИСТЕМДБ. HANA Studio показывает только последний моментальный снимок HANA, доступный в моментальном снимке хранилища, восстановленном в ходе выполнения команды моментального снимка `azacsnap -c restore --restore revertvolume` .
- Восстановите базу данных клиента.
- Запустите экземпляр HANA на сайте аварийного восстановления для рабочего идентификатора безопасности (например, в этом случае — H80).
- Выполните любое тестирование базы данных.

## <a name="next-steps"></a>Дальнейшие действия

- [Создание резервной копии](azacsnap-cmd-ref-backup.md)
- [Получение сведений о снимке](azacsnap-cmd-ref-details.md)
