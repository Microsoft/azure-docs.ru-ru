---
title: Поддерживаемые номера SKU виртуальных машин для Политики Azure
description: Статья, описывающая поддерживаемые номера SKU виртуальных машин (издатель, предложение образа и SKU образа), которые поддерживаются встроенными политиками Azure, предоставляемыми службой архивации.
ms.topic: conceptual
ms.date: 11/08/2019
ms.openlocfilehash: 4658d1beffe707682e173491edea1eac0db9c811
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "82183625"
---
# <a name="supported-vm-skus-for-azure-policy"></a>Поддерживаемые номера SKU виртуальных машин для Политики Azure

Azure Backup предоставляет встроенную политику (с помощью политики Azure), которую можно назначить **всем виртуальным машинам Azure в указанном расположении в рамках подписки или группы ресурсов**. Когда эта политика назначается данной области, все новые виртуальные машины, созданные в этой области, автоматически настраиваются для резервного копирования в **существующее хранилище в том же расположении и подписке**. В следующей таблице перечислены все SKU виртуальных машин, поддерживаемые этой политикой.

## <a name="supported-vms"></a>Поддерживаемые виртуальные машины *

**Имя политики:** Настройка резервного копирования на виртуальных машинах расположения в существующем центральном хранилище в том же расположении

Издатель образа | Предложение образа | Номер SKU образа.
--- | --- | ---
MicrosoftWindowsServer | WindowsServer | Windows Server 2008 R2 с пакетом обновления 1 (SP1) (2008-R2-SP1)
MicrosoftWindowsServer | WindowsServer | [смаллдиск] Windows Server 2008 R2 SP (2008-R2-SP1-смаллдиск)
MicrosoftWindowsServer | WindowsServer | Windows Server 2012 Datacenter (2012 — Datacenter)
MicrosoftWindowsServer | WindowsServer | [смаллдиск] Windows Server 2012 Datacenter (2012-Datacenter-смаллдиск)
MicrosoftWindowsServer | WindowsServer | Windows Server 2012 R2 Datacenter (2012-R2-Datacenter)
MicrosoftWindowsServer | WindowsServer | [смаллдиск] Windows Server 2012 R2 Datacenter (2012-R2-Datacenter-смаллдиск)
MicrosoftWindowsServer | WindowsServer | Windows Server 2016 Datacenter (2016 — Datacenter)
MicrosoftWindowsServer | WindowsServer | Windows Server 2016 Datacenter-Server Core (2016-Datacenter-Server-Core)
MicrosoftWindowsServer | WindowsServer | [смаллдиск] Windows Server 2016 Datacenter-Server Core (2016-Datacenter-Server-Core-смаллдиск)
MicrosoftWindowsServer | WindowsServer | [смаллдиск] Windows Server 2016 Datacenter (2016-Datacenter-смаллдиск)
MicrosoftWindowsServer | WindowsServer | Windows Server 2019 Datacenter Server Core с контейнерами (2016-Datacenter-with-Containers)
MicrosoftWindowsServer | WindowsServer | Windows Server 2016 удаленный рабочий стол узел сеансов 2016 (2016 — Datacenter-with-удаленных рабочих столов)
MicrosoftWindowsServer | WindowsServer | Windows Server 2019 Datacenter (2019 — Datacenter)
MicrosoftWindowsServer | WindowsServer | Windows Server 2019 Datacenter Server Core (2019 — Datacenter-Core)
MicrosoftWindowsServer | WindowsServer | [смаллдиск] Windows Server 2019 Datacenter Server Core (2019-Datacenter-Core-смаллдиск)
MicrosoftWindowsServer | WindowsServer | Windows Server 2019 Datacenter Server Core с контейнерами (2019-Datacenter-Core-with-Containers)
MicrosoftWindowsServer | WindowsServer | [смаллдиск] Windows Server 2019 Datacenter Server Core с контейнерами (2019-Datacenter-Core-with-Containers-смаллдиск)
MicrosoftWindowsServer | WindowsServer | [смаллдиск] Windows Server 2019 Datacenter (2019-Datacenter-смаллдиск)
MicrosoftWindowsServer | WindowsServer | Windows Server 2019 Datacenter с контейнерами (2019-Datacenter-with-Containers)
MicrosoftWindowsServer | WindowsServer | [смаллдиск] Windows Server 2019 Datacenter с контейнерами (2019-Datacenter-with-Containers-смаллдиск)
MicrosoftWindowsServer | WindowsServer | Windows Server 2019 Datacenter (zh-CN) (2019-Datacenter-zhcn)
MicrosoftWindowsServer | виндовссерверсемианнуал | Datacenter-Core-1709-смаллдиск
MicrosoftWindowsServer | виндовссерверсемианнуал | Datacenter-Core-1709-with-Containers-смаллдиск
MicrosoftWindowsServer | виндовссерверсемианнуал | Datacenter-Core-1803-with-Containers-смаллдиск
MicrosoftWindowsServerHPCPack | WindowsServerHPCPack | Все номера SKU образов
MicrosoftSQLServer | SQL2016SP1 — WS2016 | Все номера SKU образов
MicrosoftSQLServer | SQL2016-WS2016 | Все номера SKU образов
MicrosoftSQLServer | SQL2016SP1-WS2016-BYOL | Все номера SKU образов
MicrosoftSQLServer | SQL2012SP3 — WS2012R2 | Все номера SKU образов
MicrosoftSQLServer | SQL2016 — WS2012R2 | Все номера SKU образов
MicrosoftSQLServer | SQL2014SP2-WS2012R2 | Все номера SKU образов
MicrosoftSQLServer | SQL2012SP3-WS2012R2-BYOL | Все номера SKU образов
MicrosoftSQLServer | SQL2014SP1-WS2012R2-BYOL | Все номера SKU образов
MicrosoftSQLServer | SQL2014SP2-WS2012R2-BYOL | Все номера SKU образов
MicrosoftSQLServer | SQL2016-WS2012R2-BYOL | Все номера SKU образов
MicrosoftRServer | Млсервер — WS2016 | Все номера SKU образов
микрософтвисуалстудио | VisualStudio | Все номера SKU образов
микрософтвисуалстудио | Windows | Все номера SKU образов
микрософтдинамиксакс | Dynamics | Pre-req-AX7-OneBox-U8
microsoft-ads | Windows-Data-наука-VM | Все номера SKU образов
MicrosoftWindowsDesktop | Windows-10 | Все номера SKU образов
RedHat | RHEL | 6,7, 6,8, 6,9, 6,10, 7,2, 7,3, 7,4, 7,5, 7,6, 7,7
RedHat | RHEL-SAP-HANA | 6,7, 7,2, 7,3
SUSE | SLES | 12. X
SUSE | SLES-HPC | 12. X
SUSE | SLES-HPC-Priority | 12. X
SUSE | SLES — SAP | 12. X
SUSE | SLES-SAP-BYOS | 12. X
SUSE | SLES-Priority | 12. X
SUSE | SLES — BYOS | 12. X
SUSE | SLES — САПКАЛ | 12. X
SUSE | SLES-Standard | 12. X
Canonical | UbuntuServer | 14.04.0-LTS
Canonical | UbuntuServer | 14.04.1-LTS
Canonical | UbuntuServer | 14.04.2-LTS
Canonical | UbuntuServer | 14.04.3-LTS
Canonical | UbuntuServer | 14.04.4-LTS
Canonical | UbuntuServer | 14.04.5-DAILY-LTS
Canonical | UbuntuServer | 14.04.5-LTS
Canonical | UbuntuServer | 16.04-DAILY-LTS
Canonical | UbuntuServer | 16.04-LTS
Canonical | UbuntuServer | 16.04.0-LTS
Canonical | UbuntuServer | 18.04-DAILY-LTS
Canonical | UbuntuServer | 18.04-LTS
Oracle; | Oracle-Linux | 6,8, 6,9, 6,10, 7,3, 7,4, 7,5, 7,6
OpenLogic | CentOS | 6. X, 7. X
OpenLogic | CentOS — LVM | 6. X, 7. X
OpenLogic | CentOS — SRIOV | 6. X, 7. X
cloudera | cloudera-centos-os | 7. X
