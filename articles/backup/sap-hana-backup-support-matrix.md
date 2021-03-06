---
title: Матрица поддержки резервного копирования SAP HANA
description: В этой статье вы узнаете о поддерживаемых сценариях и ограничениях при использовании Azure Backup для резервного копирования SAP HANA баз данных на виртуальных машинах Azure.
ms.topic: conceptual
ms.date: 11/7/2019
ms.custom: references_regions
ms.openlocfilehash: cbf910a0291e90965c9698a8b2a43c587cbfe0b8
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101707240"
---
# <a name="support-matrix-for-backup-of-sap-hana-databases-on-azure-vms"></a>Матрица поддержки для резервного копирования баз данных SAP HANA на виртуальных машинах Azure

Azure Backup поддерживает резервное копирование баз данных SAP HANA в Azure. В этой статье собраны сведения о поддерживаемых сценариях и ограничениях при использовании службы архивации Azure для резервного копирования баз данных SAP HANA на виртуальных машинах Azure.

> [!NOTE]
> Теперь для частоты резервного копирования журналов можно указать минимальное значение в 15 минут. Резервные копии журналов начнут создаваться только после успешного завершения полного резервного копирования базы данных.

## <a name="scenario-support"></a>Поддержка сценария

| **Сценарий**               | **Поддерживаемые конфигурации**                                | **Неподдерживаемые конфигурации**                              |
| -------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Топология**               | Только SAP HANA, выполняемые на виртуальных машинах Linux в Azure                    | Крупные экземпляры HANA                                   |
| **Регионы**                   | **Общедоступная версия**<br> **Америка** — центральная часть США, восточная часть США 2, восточная часть США, центрально-северная часть США, центрально-южная часть США, западная часть США 2, центрально-западная часть США, западная часть США, Центральная Канада, Восточная Канада, Южная Бразилия. <br> **Азиатско-Тихоокеанский регион** — Центральная Австралия, Центральная Австралия 2, Восточная Австралия, Юго-Восточная Австралия, Восточная Япония, Западная Япония, Республика Корея, центральный регион, Республика Корея, южный регион, Восточная Азия, Юго-Восточная Азия, Центральная Индия, Южная Индия, Западная Индия, Восточный Китай, Северный Китай, Восточный Китай 2, Северный Китай 2. <br> **Европа** — Западная Европа, Северная Европа, Центральная часть франции, южная часть Соединенного Королевства, западная часть Соединенного Королевства, Северная Германия, Центрально-Западная Германия, Северная Швейцария, западная Швейцария, Центральная Северная Швейцария, Восточная Норвежская, Норвегия-Запад <br> **Африка и Ближний Восток** — Северная часть ЮАР, Западная часть ЮАР, Северная часть ОАЭ, Центральная часть ОАЭ.  <BR>  **Регионы Azure для государственных организаций** | Южная Франция, Центральная Германия, Северо-Восточная Германия, US Gov (Айова). |
| **Версии ОС**            | SLES 12 с пакетом обновления 2 (SP2), SP3, SP4 и SP5; SLES 15 с SP0, SP1, SP2 <br><br>  RHEL 7,4, 7,6, 7,7, 8,1 & 8,2                |                                             |
| **Версии HANA**          | SDC для HANA 1. x, MDC в HANA 2. x SPS04, SPS05 Rev <= 53 (Проверка сценариев с поддержкой шифрования также)      |                                                            |
| **Развертывания HANA**       | SAP HANA на одной виртуальной машине Azure — только масштабирование. <br><br> Для развертываний с высоким уровнем доступности оба узла на двух разных компьютерах рассматриваются как отдельные узлы с отдельными цепочками данных.               | Горизонтальное масштабирование <br><br> В развертываниях с высоким уровнем доступности резервное копирование не всегда автоматически переключается на дополнительный узел при отработке отказа. Резервное копирование следует настраивать отдельно для каждого узла.                                           |
| **Экземпляры HANA**         | Один экземпляр SAP HANA на одной виртуальной машине Azure — только масштабирование. | Несколько экземпляров SAP HANA на одной виртуальной машине. В каждый момент времени можно защитить только один из этих экземпляров.                  |
| **Типы баз данных HANA**    | Контейнер отдельной базы данных на версии 1.x, контейнер с несколькими базами данных на версии 2.x. | Контейнер с несколькими базами данных на версии HANA 1.x.                                              |
| **Размер базы данных HANA**     | Базы данных HANA размером <= 8 ТБ (это не размер памяти системы HANA)               |                                                              |
| **Типы резервного копирования**           | Полные, разностные, добавочные и резервные копии журналов                          |  Моментальные снимки                                       |
| **Типы восстановления**          | Сведения о поддерживаемых типах восстановления см. в заметке SAP HANA номер [1642148](https://launchpad.support.sap.com/#/notes/1642148). |                                                              |
| **Ограничения резервного копирования**          | До 8 ТБ полного размера резервной копии на экземпляр SAP HANA (мягкое ограничение)         |                                                              |
| **Особые конфигурации** |                                                              | SAP HANA и Dynamic Tiering <br>  Клонирование через LaMa        |

------

>[!NOTE]
>Azure Backup не выполняет автоматический переход на летнее время при резервном копировании базы данных SAP HANA на виртуальной машине Azure.
>
>Измените политику вручную, если это необходимо.

> [!NOTE]
> Теперь вы можете [отслеживать задания резервного копирования и восстановления](./sap-hana-db-manage.md#monitor-manual-backup-jobs-in-the-portal) (на том же компьютере), запущенные из собственных клиентов HANA (SAP HANA Studio или панели администраторов) в портал Azure.

## <a name="next-steps"></a>Дальнейшие действия

* Сведения о [резервном копировании SAP HANA баз данных, работающих на виртуальных машинах Azure](./backup-azure-sap-hana-database.md)
* Узнайте, как [восстановить базы данных SAP HANA, запущенные на виртуальных машинах Azure](./sap-hana-db-restore.md).
* Узнайте, как [управлять базами данных SAP HANA, резервное копирование которых выполняется с помощью Azure Backup](sap-hana-db-manage.md).
* Узнайте, как [устранять распространенные проблемы при резервном копировании баз данных SAP HANA](./backup-azure-sap-hana-database-troubleshoot.md).
