---
title: Сведения о SAP HANA резервного копирования базы данных на виртуальных машинах Azure
description: В этой статье вы узнаете, как выполнять резервное копирование SAP HANA баз данных, работающих на виртуальных машинах Azure.
ms.topic: conceptual
ms.date: 12/11/2019
ms.openlocfilehash: efb9c3f786e429df404e261f053a9c9a9b032e11
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96296460"
---
# <a name="about-sap-hana-database-backup-in-azure-vms"></a>Сведения о SAP HANA резервного копирования базы данных на виртуальных машинах Azure

SAP HANA базы данных — это критически важные рабочие нагрузки, для которых требуется низкая Целевая точка восстановления (RPO) и целевое время восстановления (RTO). Теперь вы можете [выполнять резервное копирование SAP HANA баз данных, работающих на виртуальных машинах Azure](./tutorial-backup-sap-hana-db.md) , с помощью [Azure Backup](./backup-overview.md).

Azure Backup [backint) создание сертифицировано](https://www.sap.com/dmc/exp/2013_09_adpd/enEN/#/d/solutions?id=8f3fd455-a2d7-4086-aa28-51d8870acaa5) SAP, чтобы обеспечить собственную поддержку резервного копирования, используя собственные API-интерфейсы SAP HANA. Это предложение от Azure Backup соответствует мантру Azure Backup резервным копиям **нулевой инфраструктуры** , что устраняет необходимость в развертывании и управлении инфраструктурой резервного копирования. Теперь вы можете легко выполнять резервное копирование и восстановление SAP HANA баз данных, работающих на виртуальных машинах Azure ([виртуальные машины серии M](../virtual-machines/m-series.md) также поддерживаются сейчас), и использовать возможности управления предприятием, предоставляемые Azure Backup.

## <a name="added-value"></a>Добавленное значение

Использование Azure Backup для резервного копирования и восстановления SAP HANA баз данных дает следующие преимущества.

* **15-минутная цель точки восстановления (RPO)**. Теперь возможно восстановление критических данных до 15 минут.
* Восстановление до **точки во времени с одним щелчком**. восстановить рабочие данные на альтернативные серверы Hana стало просто. Создание цепочки резервных копий и каталогов для выполнения восстановления осуществляется в Azure в фоновом режиме.
* **Долгосрочное хранение**: для обеспечения строгого соответствия требованиям и аудита. Храните резервные копии в течение нескольких лет в зависимости от длительности хранения, за пределами которых точки восстановления автоматически удаляются встроенной функцией управления жизненным циклом.
* **Управление резервным копированием из Azure**. Используйте возможности управления и мониторинга Azure Backup для улучшения процесса управления. Azure CLI также поддерживается.

Чтобы просмотреть сценарии резервного копирования и восстановления, которые поддерживаются сегодня, см. [таблицу поддержки сценариев SAP HANA](./sap-hana-backup-support-matrix.md#scenario-support).

## <a name="backup-architecture"></a>Архитектура службы Backup

![Схема архитектуры резервного копирования](./media/sap-hana-db-about/backup-architecture.png)

* Процесс резервного копирования начинается с [создания хранилища служб восстановления](./tutorial-backup-sap-hana-db.md#create-a-recovery-services-vault) в Azure. Это хранилище будет использоваться для хранения резервных копий и точек восстановления, созданных с течением времени.
* Виртуальная машина Azure с SAP HANA Server зарегистрирована в хранилище, и будут [обнаружены](./tutorial-backup-sap-hana-db.md#discover-the-databases)базы данных для резервного копирования. Чтобы служба Azure Backup обнаружила базы данных, необходимо запустить [сценарий](https://aka.ms/scriptforpermsonhana) на сервере Hana в качестве привилегированного пользователя.
* Этот скрипт создает пользователя **азуревлбаккуфанаусер** DB и соответствующий ключ с тем же именем в **hdbuserstore**. Сведения о том, что делает сценарий, см. в разделе  [что такое сценарий предварительной регистрации](tutorial-backup-sap-hana-db.md#what-the-pre-registration-script-does) .
* Azure Backup служба теперь устанавливает **подключаемый модуль Azure Backup для Hana** на зарегистрированном SAP HANA сервере.
* Пользователь **азуревлбаккуфанаусер** DB, созданный сценарием предрегистрации, используется **подключаемым модулем Azure Backup для Hana** для выполнения всех операций резервного копирования и восстановления. При попытке настроить резервное копирование для SAP HANA баз данных без выполнения этого скрипта может появиться следующая ошибка: **усереррорханаскриптнотрун**.
* Чтобы [настроить резервное копирование](./tutorial-backup-sap-hana-db.md#configure-backup) для обнаруженных баз данных, выберите необходимую политику архивации и включите резервное копирование.

* После настройки резервного копирования Azure Backup служба настраивает следующие параметры Backint) создание на уровне базы данных на защищенном сервере SAP HANA.
  *  [catalog_backup_using_backint:true];
  *  [enable_accumulated_catalog_backup:false];
  *  [parallel_data_backup_backint_channels:1];
  *  [log_backup_timeout_s:900)];
  *  [backint_response_timeout:7200].

>[!NOTE]
>Убедитесь, что эти параметры *отсутствуют* на уровне узла. Параметры уровня узла будут переопределять эти параметры и могут вызвать непредвиденное поведение.
>

* **Подключаемый модуль Azure Backup для Hana** поддерживает все расписания резервного копирования и сведения о политике. Он запускает запланированные резервные копии и взаимодействует с **модулем резервного копирования Hana** через API-интерфейсы backint) создание.
* **Модуль резервного копирования Hana** возвращает поток backint) создание с данными для резервного копирования.
* Все запланированные резервные копии и резервные копии по запросу (запущенные с портал Azure), которые являются либо полными, либо разностными, инициируются **подключаемым модулем Azure Backup для Hana**. Однако резервные копии журналов управляются и активируются самим **модулем резервного копирования Hana** .
* Azure Backup для SAP HANA, является сертифицированным решением Backint) создание, не зависит от базовых дисков или типов виртуальных машин. Резервная копия выполняется потоками, созданными HANA.

## <a name="using-azure-vm-backup-with-azure-sap-hana-backup"></a>Резервное копирование виртуальных машин Azure с помощью Azure SAP HANA Backup

В дополнение к использованию резервной копии SAP HANA в Azure, которая обеспечивает резервное копирование и восстановление на уровне базы данных, можно использовать решение резервного копирования виртуальных машин Azure для резервного копирования ОПЕРАЦИОННЫХ систем и дисков, не относящихся к базам данных.

[Backint) создание сертифицированное решение резервного копирования Azure SAP HANA](#backup-architecture) можно использовать для резервного копирования и восстановления базы данных.

[Резервное копирование виртуальных машин Azure](backup-azure-vms-introduction.md) можно использовать для резервного копирования операционной системы и других дисков, не относящихся к базам данных. Резервная копия виртуальной машины создается каждый день и копирует все диски (за исключением **дисков ОС с ускоритель записи (WA)** и **Ultra Disks**). Так как резервное копирование базы данных выполняется с помощью решения Azure SAP HANA Backup, можно создать резервную копию файлов только для ОС и дисков, отличных от баз данных, с помощью функции [выборочного резервного копирования и восстановления виртуальных машин Azure](selective-disk-backup-restore.md) .

Чтобы восстановить виртуальную машину, работающую SAP HANA, выполните следующие действия.

* [Восстановите новую виртуальную машину из резервной копии виртуальной машины Azure](backup-azure-arm-restore-vms.md) из последней точки восстановления. Или создайте новую пустую виртуальную машину и подключите диски из последней точки восстановления.
* Если диски WA исключены, они не восстанавливаются. В этом случае создайте пустые диски WA и область журнала.
* После установки всех остальных конфигураций (таких как IP-адрес, имя системы и т. д.) виртуальная машина получает данные из Azure Backup.
* Теперь восстановите базу данных в виртуальной машине из [резервной копии Azure SAP HANA DB](sap-hana-db-restore.md#restore-to-a-point-in-time-or-to-a-recovery-point) до нужной точки во времени.

## <a name="next-steps"></a>Дальнейшие действия

* Узнайте, как [восстановить базу данных SAP Hana, работающую на виртуальной машине Azure](./sap-hana-db-restore.md) .
* Узнайте, как [управлять базами данных SAP HANA, резервное копирование которых выполняется с помощью Azure Backup](./sap-hana-db-manage.md).
