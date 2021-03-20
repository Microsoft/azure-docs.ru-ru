---
title: Удаление с помощью средства создания моментальных снимков с единообразием приложений Azure для Azure NetApp Files | Документация Майкрософт
description: Содержит инструкции по запуску команды Delete инструмента для создания моментальных снимков с единообразием приложений Azure, который можно использовать с Azure NetApp Files.
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
ms.openlocfilehash: 0e2e4beebedb93524da43c5a3fad750b0295f5cd
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97632912"
---
# <a name="delete-using-azure-application-consistent-snapshot-tool-preview"></a>Удаление с помощью средства создания моментальных снимков с единообразием приложений Azure (Предварительная версия)

В этой статье представлено руководство по запуску команды Delete инструмента для создания моментальных снимков с единообразием приложений Azure, который можно использовать с Azure NetApp Files.

## <a name="introduction"></a>Введение

Можно удалить моментальные снимки тома и записи каталога базы данных с помощью `azacsnap -c delete` команды.

> [!IMPORTANT]
> Моментальные снимки, созданные менее чем через 10 минут до выполнения этой команды, не должны быть удалены из-за возможных помех с репликацией моментальных снимков.

## <a name="command-options"></a>Параметры команды

`-c delete`Команда имеет следующие параметры.

- `--delete hana` При использовании с параметрами `--hanasid <SID>` и `--hanabackupid <HANA backup id>` удалит записи из каталога резервного копирования SAP Hana, соответствующего критериям.

- `--delete storage` При использовании с параметром `--snapshot <snapshot name>` будет удален моментальный снимок из серверной системы хранения.

- `--delete sync` При использовании с параметрами `--hanasid <SID>` и `--hanabackupid <HANA backup id>` получает имя моментального снимка хранилища из каталога резервного копирования для `<HANA backup id>` , а затем удаляет запись из каталога резервного копирования _и_ моментальный снимок с любого тома, содержащего именованный моментальный снимок.

- `--delete sync` При использовании с `--snapshot <snapshot name>` будет проверять наличие записей в каталоге резервного копирования для `<snapshot name>` , получает SAP HANA идентификатор резервного копирования и удаляет запись из каталога резервных копий _и_ моментальный снимок из любого тома, содержащего именованный моментальный снимок.

- `[--force]` используемых *Используйте с осторожностью*.  Эта операция приведет к принудительному удалению без запроса подтверждения.

- `[--configfile <config filename>]` — Необязательный параметр, который позволяет использовать пользовательские имена файлов конфигурации.

### <a name="delete-a-snapshot-using-sync-option"></a>Удаление моментального снимка с помощью `sync` параметра

```bash
azacsnap -c delete --delete sync --hanasid H80 --hanabackupid 157979797979
```

> [!NOTE]
> Проверяет наличие записей в каталоге резервного копирования для SAP HANA идентификатор резервной копии 157979797979, получает имя моментального снимка хранилища и удаляет запись в каталоге резервного копирования и моментальный снимок из всех томов, содержащих именованный моментальный снимок.

```bash
azacsnap -c delete --delete sync --snapshot hana_hourly.2020-01-22_2358
```

> [!NOTE]
> Проверяет наличие записей в каталоге резервного копирования для моментального снимка с именем hana_hourly .2020-01 -22 _2358, получает идентификатор резервной копии SAP HANA и удаляет запись в каталоге резервных копий и моментальный снимок из любого тома, содержащего именованный моментальный снимок.

### <a name="delete-a-snapshot-using-hana-option"></a>Удаление моментального снимка с помощью `hana` параметра

```bash
azacsnap -c delete --delete hana --hanasid H80 --hanabackupid 157979797979
```

> [!NOTE]
> Удаляет SAP HANA с ИДЕНТИФИКАТОРом резервной копии 157979797979 из каталога резервного копирования для SID H80.

### <a name="delete-a-snapshot-using-storage-option"></a>Удаление моментального снимка с помощью `storage` параметра

```bash
azacsnap -c delete --delete storage --snapshot hana_hourly.2020-01-22_2358
```

> [!NOTE]
> Удаляет моментальный снимок из всех томов, содержащих моментальный снимок с именем hana_hourly .2020-01 -22 _2358.

**Вывод с помощью `--delete storage` параметра**

Пользователю предлагается подтвердить удаление.

```bash
azacsnap -c delete --delete storage --snapshot azacsnap-hsr-ha.2020-07-02T221702.2535255Z
```

```output
Processing delete request for snapshot 'azacsnap-hsr-ha.2020-07-02T221702.2535255Z'.
Are you sure you want to permanently delete the snapshot 'azacsnap-hsr-ha.2020-07-02T221702.2535255Z' from all storage volumes.? (y/n) [n]: y
Snapshot deletion completed
```

Можно избежать подтверждения пользователя, используя необязательный `--force` параметр следующим образом:

```bash
azacsnap -c delete --delete storage --snapshot azacsnap-hsr-ha.2020-07-02T222201.4988618Z --force
```

```output
Processing delete request for snapshot 'azacsnap-hsr-ha.2020-07-02T222201.4988618Z'.
Snapshot deletion completed
```

## <a name="next-steps"></a>Дальнейшие действия

- [Получение сведений о снимке](azacsnap-cmd-ref-details.md)
- [Создание резервной копии](azacsnap-cmd-ref-backup.md)
