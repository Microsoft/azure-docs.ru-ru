---
title: Получение сведений с помощью средства создания моментальных снимков для приложений Azure для Azure NetApp Files | Документация Майкрософт
description: Содержит инструкции по запуску команды Details в средстве создания моментальных снимков с помощью приложений Azure, которые можно использовать с Azure NetApp Files.
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
ms.openlocfilehash: 43d358f7050dcc831f9f2bf439fba6e688ee633b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97632903"
---
# <a name="obtain-details-using-azure-application-consistent-snapshot-tool-preview"></a>Получение сведений с помощью средства создания моментальных снимков для приложений Azure (Предварительная версия)

В этой статье представлено руководство по выполнению команды Details в средстве создания моментальных снимков с помощью приложений Azure, которые можно использовать с Azure NetApp Files.

## <a name="introduction"></a>Введение

`azacsnap -c details`Команда выводит сведения о моментальных снимках (имя тома, имя моментального снимка, время создания, комментарии, размер моментального снимка), хранящиеся на всех томах в файле конфигурации.  Выходные данные консоли можно вставлять в электронную таблицу для дальнейшего анализа.

Из-за информации, хранящейся в внутренних системах, выходные данные команды немного отличаются при запуске в **больших экземплярах Azure** и **Azure NetApp Files**.

## <a name="command-options"></a>Параметры команды

`-c details`Команда имеет следующие параметры.

- `--details snapshots` Содержит список основных сведений о моментальных снимках для каждого настроенного тома. Эту команду можно выполнить на сервере-источнике или на сервере в расположении аварийного восстановления. Команда предоставляет следующие сведения, разделенные каждым томом, содержащим моментальные снимки:
  - Размер всего моментальных снимков в томе.
  - для каждого снимка на этом томе указываются следующие сведения:
  - имя моментального снимка;
  - время создания снимка;
  - размер снимка;
  - частота создания снимка;

- `--details replication` Содержит основные сведения о состоянии репликации с рабочего сайта на сайт аварийного восстановления. Команда предоставляет выходные данные, позволяющие отслеживать выполнение репликации.  Он показывает размер реплицируемых элементов. Он также предоставляет рекомендации, если репликация занимает слишком много времени или если ссылка не работает.

- `[--configfile <config filename>]` — Необязательный параметр, который позволяет использовать пользовательские имена файлов конфигурации.

### <a name="output-of-the-azacsnap--c-details---details-snapshots-command"></a>Выходные данные `azacsnap -c details --details snapshots` команды

Приведенный ниже пример был выполнен в **крупном экземпляре Azure** , и выходные данные были обрезаны для краткости.  Также имейте в виду, что некоторые строки могут быть перенесены в соответствии с выходными данными.

```bash
azacsnap -c details --details snapshots
```

```output
List snapshot details called with snapshotFilter ''
#, Volume, Snapshot, Create Time, HANA Backup ID, Snapshot Size
#1, hana_data_h31_azsollabbl20a31_mnt00001_t210_vol, hana_hourly.2020-06-17T113043.1586971Z, "Wed Jun 17 11:31:14 2020", "HANA Backup ID: 1592393444174, 702.6MB
#2, hana_data_h31_azsollabbl20a31_mnt00001_t210_vol, shoasnap-hsr-ha.2020-07-02_2200.5, "Thu Jul 02 22:01:37 2020", "Backup ID: 1593727205201, 342.3MB
#3, hana_data_h31_azsollabbl20a31_mnt00001_t210_vol, azacsnap-hsr-ha.2020-07-02T220201.5332158Z, "Thu Jul 02 22:03:34 2020", "HANA Backup ID: 1593727322533|azacsnap version: 5.0 Preview (20200617.75879)", 3.27MB
#4, hana_data_h31_azsollabbl20a31_mnt00001_t210_vol, shoasnap-hsr-ha.2020-07-02_2205.4, "Thu Jul 02 22:06:36 2020", "Backup ID: 1593727504776, 3.14MB
#5, hana_data_h31_azsollabbl20a31_mnt00001_t210_vol, azacsnap-hsr-ha.2020-07-02T220702.3283669Z, "Thu Jul 02 22:08:37 2020", "HANA Backup ID: 1593727623339|azacsnap version: 5.0 Preview (20200617.75879)", 3.50MB
#6, hana_data_h31_azsollabbl20a31_mnt00001_t210_vol, shoasnap-hsr-ha.2020-07-02_2210.3, "Thu Jul 02 22:11:37 2020", "Backup ID: 1593727805216, 2.85MB
#7, hana_data_h31_azsollabbl20a31_mnt00001_t210_vol, azacsnap-hsr-ha.2020-07-02T221201.7013700Z, "Thu Jul 02 22:13:36 2020", "HANA Backup ID: 1593727922724|azacsnap version: 5.0 Preview (20200617.75879)", 3.34MB
#8, hana_data_h31_azsollabbl20a31_mnt00001_t210_vol, shoasnap-hsr-ha.2020-07-02_2215.2, "Thu Jul 02 22:16:36 2020", "Backup ID: 1593728104772, 2.73MB
#9, hana_data_h31_azsollabbl20a31_mnt00001_t210_vol, azacsnap-hsr-ha.2020-07-02T221702.2535255Z, "Thu Jul 02 22:18:35 2020", "HANA Backup ID: 1593728223274|azacsnap version: 5.0 Preview (20200617.75879)", 3.39MB
#10, hana_data_h31_azsollabbl20a31_mnt00001_t210_vol, shoasnap-hsr-ha.2020-07-02_2220.1, "Thu Jul 02 22:21:37 2020", "Backup ID: 1593728405346, 3.29MB
#11, hana_data_h31_azsollabbl20a31_mnt00001_t210_vol, azacsnap-hsr-ha.2020-07-02T222201.4988618Z, "Thu Jul 02 22:23:36 2020", "HANA Backup ID: 1593728522505|azacsnap version: 5.0 Preview (20200617.75879)", 3.68MB
#12, hana_data_h31_azsollabbl20a31_mnt00001_t210_vol, shoasnap-hsr-ha.2020-07-02_2225.0, "Thu Jul 02 22:26:37 2020", "Backup ID: 1593728705321, 2.80MB
#13, hana_data_h31_azsollabbl20a31_mnt00001_t210_vol, azacsnap-hsr-ha.2020-07-02T222702.0521995Z, "Thu Jul 02 22:28:37 2020", "HANA Backup ID: 1593728823058|azacsnap version: 5.0 Preview (20200617.75879)", 1.04MB
, hana_data_h31_azsollabbl20a31_mnt00001_t210_vol, , , Size used by Snapshots, 1.05GB
#1, hana_shared_h31_azsollabbl20a31_t210_vol, hana_hourly.2020-06-17T113043.1586971Z, "Wed Jun 17 11:31:12 2020", "HANA Backup ID: 1592393444174, 2.55GB
#2, hana_shared_h31_azsollabbl20a31_t210_vol, shoasnap-hsr-ha.2020-07-02_2200.5, "Thu Jul 02 22:01:37 2020", "Backup ID: 1593727205201, 4.30MB
#3, hana_shared_h31_azsollabbl20a31_t210_vol, azacsnap-hsr-ha.2020-07-02T220201.5332158Z, "Thu Jul 02 22:03:36 2020", "HANA Backup ID: 1593727322533|azacsnap version: 5.0 Preview (20200617.75879)", 8.04MB
#4, hana_shared_h31_azsollabbl20a31_t210_vol, shoasnap-hsr-ha.2020-07-02_2205.4, "Thu Jul 02 22:06:37 2020", "Backup ID: 1593727504776, 8.12MB
#5, hana_shared_h31_azsollabbl20a31_t210_vol, azacsnap-hsr-ha.2020-07-02T220702.3283669Z, "Thu Jul 02 22:08:35 2020", "HANA Backup ID: 1593727623339|azacsnap version: 5.0 Preview (20200617.75879)", 4.28MB
#6, hana_shared_h31_azsollabbl20a31_t210_vol, shoasnap-hsr-ha.2020-07-02_2210.3, "Thu Jul 02 22:11:37 2020", "Backup ID: 1593727805216, 4.33MB
#7, hana_shared_h31_azsollabbl20a31_t210_vol, azacsnap-hsr-ha.2020-07-02T221201.7013700Z, "Thu Jul 02 22:13:34 2020", "HANA Backup ID: 1593727922724|azacsnap version: 5.0 Preview (20200617.75879)", 4.31MB
#8, hana_shared_h31_azsollabbl20a31_t210_vol, shoasnap-hsr-ha.2020-07-02_2215.2, "Thu Jul 02 22:16:37 2020", "Backup ID: 1593728104772, 4.30MB
#9, hana_shared_h31_azsollabbl20a31_t210_vol, azacsnap-hsr-ha.2020-07-02T221702.2535255Z, "Thu Jul 02 22:18:37 2020", "HANA Backup ID: 1593728223274|azacsnap version: 5.0 Preview (20200617.75879)", 7.84MB
#10, hana_shared_h31_azsollabbl20a31_t210_vol, shoasnap-hsr-ha.2020-07-02_2220.1, "Thu Jul 02 22:21:37 2020", "Backup ID: 1593728405346, 8.09MB
#11, hana_shared_h31_azsollabbl20a31_t210_vol, azacsnap-hsr-ha.2020-07-02T222201.4988618Z, "Thu Jul 02 22:23:34 2020", "HANA Backup ID: 1593728522505|azacsnap version: 5.0 Preview (20200617.75879)", 4.34MB
#12, hana_shared_h31_azsollabbl20a31_t210_vol, shoasnap-hsr-ha.2020-07-02_2225.0, "Thu Jul 02 22:26:37 2020", "Backup ID: 1593728705321, 4.31MB
#13, hana_shared_h31_azsollabbl20a31_t210_vol, azacsnap-hsr-ha.2020-07-02T222702.0521995Z, "Thu Jul 02 22:28:35 2020", "HANA Backup ID: 1593728823058|azacsnap version: 5.0 Preview (20200617.75879)", 4.31MB
, hana_shared_h31_azsollabbl20a31_t210_vol, , , Size used by Snapshots, 2.62GB
#1, hana_log_backups_h31_azsollabbl20a31_t210_vol, azacsnap_vesangam_other_test.2020-06-17T113215.4462696Z, "Wed Jun 17 11:32:43 2020", "HANA Log-Backups, 156KB
#2, hana_log_backups_h31_azsollabbl20a31_t210_vol, azacsnap_vesangam_other_test2.2020-06-17T114205.1041364Z, "Wed Jun 17 11:42:35 2020", "HANA Log-Backups, 1.34MB
, hana_log_backups_h31_azsollabbl20a31_t210_vol, , , Size used by Snapshots, 1.49MB
```

> [!NOTE]
> В этом примере показаны выходные данные для моментальных снимков, выполняемые с использованием предыдущей версии (v 4.3), а также моментальные снимки, созданные последней версией (5,0).

### <a name="output-of-the-azacsnap--c-details---details-replication-command"></a>Выходные данные `azacsnap -c details --details replication` команды

Эта команда проверяет состояние репликации хранилища с первичного сайта на расположение DR и *должно* выполняться на **сервере сайта аварийного восстановления**. Эта команда проверяет **только** тома в файле конфигурации.

> [!NOTE]
> Этот параметр доступен только в системах **крупных экземпляров Azure** (Хли).

```bash
azacsnap -c details --details replication
```

```output
Getting replication details for HLI systems
Volume, Link status, Current Replication Activity, Latest Snapshot Replicated, Size of Latest Snapshot Replicated, Current Lag Time between snapshots (HH:MM:SS)
hana_data_h80_mnt00001_t250_xdp, Active, Idle, snapmirror.d4b34832-8922-11e9-9b18-00a098f706fa_2151036590.2020-11-02_031000, 7.53MB, 00h 06m 02s
hana_shared_h80_t250_xdp, Active, Idle, snapmirror.d4b34832-8922-11e9-9b18-00a098f706fa_2151036594.2020-11-02_031000, 1.64MB, 00h 06m 04s
hana_log_backups_h80_t250_xdp, Active, Idle, snapmirror.d4b34832-8922-11e9-9b18-00a098f706fa_2151036635.2020-11-02_031500, 2.32KB, 00h 01m 04s
```

В следующем примере состояние репликации разорвано, и в этом сценарии активация аварийного восстановления, скорее всего, приведет к неполным данным на сайте аварийного восстановления.

> [!NOTE]
> Обратите внимание на предупреждение в следующем примере о том, что репликация для двух томов нарушена.

```bash
azacsnap -c details --details replication
```

```output
Getting replication details for HLI systems
Volume, Link status, Current Replication Activity, Latest Snapshot Replicated, Size of Latest Snapshot Replicated, Current Lag Time between snapshots (HH:MM:SS)
hana_data_h80_sapprdhdb80_mnt00001_t020_xdp, Broken-Off - Contact Microsoft Operations immediately!, Idle, snapmirror.d4b34832-8922-11e9-9b18-00a098f706fa_2151036523.2020-03-12_052000, 2.47MB, 99h 99m 99s  WARNING: replication has not occurred for more than 30 minutes!
hana_log_backups_h80_sapprdhdb80_t020_xdp, Broken-Off - Contact Microsoft Operations immediately!, Idle, snapmirror.d4b34832-8922-11e9-9b18-00a098f706fa_2151036539.2020-03-12_052400, 196.7KB, 99h 99m 99s  WARNING: replication has not occurred for more than 30 minutes!
hana_shared_h80_sapprdhdb80_t020_xdp, Active, Idle, snapmirror.d4b34832-8922-11e9-9b18-00a098f706fa_2151036524.2020-04-02_020000, 2.94MB, 00h 04m 55s
```

В этом примере имеется успешная репликация между основным сайтом и сайтом аварийного восстановления, поэтому эти системы готовы поддерживать сценарий аварийного восстановления.

```bash
azacsnap -c details --details replication
```

```output
Getting replication details for HLI systems
Volume, Link status, Current Replication Activity, Latest Snapshot Replicated, Size of Latest Snapshot Replicated, Current Lag Time between snapshots (HH:MM:SS)
hana_data_h80_sapprdhdb80_mnt00001_t020_xdp, Active, Idle, snapmirror.21215d07-2653-11e8-8e4c-00a098af659c_2157387233.2019-04-09_055000, 106.8MB, 00h 09m 45s
hana_log_backups_h80_sapprdhdb80_t020_xdp, Active, Idle, snapmirror.21215d07-2653-11e8-8e4c-00a098af659c_2157387278.2019-04-09_055700, 75.57MB, 00h 02m 45s
hana_shared_h80_sapprdhdb80_t020_xdp, Active, Idle, snapmirror.d4b34832-8922-11e9-9b18-00a098f706fa_2151036524.2020-04-02_020000, 2.94MB, 00h 04m 55s
```

## <a name="next-steps"></a>Дальнейшие действия

- [Создание резервной копии](azacsnap-cmd-ref-backup.md)
- [Удаление моментальных снимков](azacsnap-cmd-ref-delete.md)
