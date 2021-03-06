---
title: Восстановление баз данных SAP HANA на виртуальных машинах Azure
description: Из этой статьи вы узнаете, как восстановить SAP HANA базы данных, работающие на виртуальных машинах Azure. Можно также использовать восстановление между регионами для восстановления баз данных в дополнительный регион.
ms.topic: conceptual
ms.date: 11/7/2019
ms.openlocfilehash: c502b7741acd343baefe5e2bf8b95cfc02e46688
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96021679"
---
# <a name="restore-sap-hana-databases-on-azure-vms"></a>Восстановление баз данных SAP HANA на виртуальных машинах Azure

В этой статье описывается, как восстановить SAP HANA базы данных, работающие на виртуальной машине Azure, в которой служба Azure Backup выполнила резервное копирование в хранилище служб восстановления. Восстановление можно использовать для создания копий данных для сценариев разработки и тестирования или для возврата к предыдущему состоянию.

Дополнительные сведения о резервном копировании баз данных SAP HANA см. в статье [резервное копирование SAP HANA баз данных на виртуальных машинах Azure](./backup-azure-sap-hana-database.md).

## <a name="restore-to-a-point-in-time-or-to-a-recovery-point"></a>Восстановление до точки во времени или до точки восстановления

В Azure Backup можно выполнить восстановление баз данных SAP HANA, запущенных на виртуальных машинах Azure, следующим образом.

* Восстановление до состояния на определенную дату или время (с точностью до секунд) с помощью резервных копий журналов. На основе выбранного времени Azure Backup автоматически определяет соответствующие полные или разностные резервные копии и цепочку резервных копий журналов, необходимых для восстановления.

* Восстановление конкретной полной или разностной резервной копии по определенной точке восстановления.

## <a name="prerequisites"></a>Предварительные условия

Перед восстановлением базы данных обратите внимание на следующее:

* Базу данных можно восстановить только в экземпляре SAP HANA, который находится в том же регионе.

* Целевой экземпляр должен быть зарегистрирован в том же хранилище, что и источник.

* Azure Backup не может обнаруживать два разных экземпляра SAP HANA на одной виртуальной машине. Поэтому восстановление данных из одного экземпляра в другой на одной виртуальной машине невозможно.

* Чтобы убедиться, что целевой экземпляр SAP HANA готов к восстановлению, проверьте состояние **готовности к резервному копированию** :

  1. Откройте хранилище, в котором зарегистрирован целевой экземпляр SAP HANA.

  1. На панели мониторинга хранилища в разделе **Приступая к работе** выберите пункт **резервное копирование**.

      ![Резервное копирование на панели мониторинга хранилища](media/sap-hana-db-restore/getting-started-backup.png)

  1. В поле **резервное копирование** в разделе **что вы хотите создать резервную копию?** выберите **SAP HANA на виртуальной машине Azure**.

      ![Выбор пункта "SAP HANA в виртуальной машине Azure"](media/sap-hana-db-restore/sap-hana-backup.png)

  1. В разделе **Обнаружение баз данных в виртуальных машинах** выберите **Просмотреть сведения**.

      ![Подробнее](media/sap-hana-db-restore/view-details.png)

  1. Проверьте **готовность резервного копирования** целевой виртуальной машины.

      ![Защищенные серверы](media/sap-hana-db-restore/protected-servers.png)

* Дополнительные сведения о типах восстановления, поддерживаемых SAP HANA, см. в SAP HANA Примечание [1642148](https://launchpad.support.sap.com/#/notes/1642148)

## <a name="restore-a-database"></a>Восстановление базы данных

Для восстановления необходимы следующие разрешения.

* Разрешения **оператора Backup** в хранилище, в котором выполняется восстановление.
* Доступ **участника (с правами на запись)** к исходной виртуальной машине, резервное копирование которой выполняется.
* Доступ **участника (записи**) к целевой виртуальной машине:
  * Если выполняется восстановление на ту же виртуальную машину, это исходная виртуальная машина.
  * Если выполняется восстановление в альтернативное расположение, это новая целевая виртуальная машина.

1. Откройте хранилище, в котором зарегистрирована SAP HANA база данных, подлежащей восстановлению.

1. На панели мониторинга хранилища в разделе **защищенные элементы** выберите **элементы архивации** .

    ![Архивные элементы](media/sap-hana-db-restore/backup-items.png)

1. В разделе **элементы архивации** в поле **тип управления архивацией** выберите **SAP HANA на виртуальной машине Azure** .

    ![Тип управления резервным копированием](media/sap-hana-db-restore/backup-management-type.png)

1. Выберите базу данных для восстановления

    ![База данных для восстановления](media/sap-hana-db-restore/database-to-restore.png)

1. Просмотрите меню базы данных. Он предоставляет сведения о резервной копии базы данных, включая:

    * Самые старые и последние точки восстановления

    * Состояние резервного копирования журнала за последние 24 и 72 часов для базы данных

    ![Меню базы данных](media/sap-hana-db-restore/database-menu.png)

1. Выбор **восстановления базы данных**

1. В разделе **Конфигурация восстановления** укажите, где (или как) следует восстанавливать данные:

    * **Альтернативное расположение**. Восстановите базу данных в альтернативное расположение и сохранив исходную базу данных.

    * **Перезаписать базу** данных. Восстановите данные в том же экземпляре SAP Hana, что и исходный источник. В этом варианте перезаписывается исходная база данных.

      ![Восстановить конфигурацию](media/sap-hana-db-restore/restore-configuration.png)

### <a name="restore-to-alternate-location"></a>Восстановление в альтернативном расположении

1. В меню **восстановить конфигурацию** в разделе **место для восстановления** выберите **альтернативное расположение**.

    ![Восстановление в альтернативном расположении](media/sap-hana-db-restore/restore-alternate-location.png)

1. Выберите SAP HANA имя узла и имя экземпляра, для которого требуется восстановить базу данных.
1. Убедитесь, что целевой экземпляр SAP HANA готов к восстановлению, убедившись в **готовности к резервному копированию.** Дополнительные сведения см. в [разделе Предварительные требования](#prerequisites) .
1. В поле **Имя восстановленной базы данных** введите имя целевой базы данных.

    > [!NOTE]
    > Восстановление отдельная база данных контейнера (SDC) должно удовлетворять этим [проверкам](backup-azure-sap-hana-database-troubleshoot.md#single-container-database-sdc-restore).

1. Если применимо, выберите **Перезаписать, если база данных с таким именем уже существует в выбранном экземпляре Hana**.
1. Щелкните **ОК**.

    ![Конфигурация восстановления — последний экран](media/sap-hana-db-restore/restore-configuration-last.png)

1. В окне **Выбор точки восстановления** выберите **журналы (на момент времени)** для [восстановления до определенной точки во времени](#restore-to-a-specific-point-in-time). Или выберите **полную & разностную резервную копию** для [восстановления до определенной точки восстановления](#restore-to-a-specific-recovery-point).

### <a name="restore-and-overwrite"></a>Восстановление и перезапись

1. В меню **восстановить конфигурацию** в разделе **место для восстановления** выберите **перезаписать базу данных**  >  **ОК**.

    ![Перезапись базы данных](media/sap-hana-db-restore/overwrite-db.png)

1. В окне **Выбор точки восстановления** выберите **журналы (на момент времени)** для [восстановления до определенной точки во времени](#restore-to-a-specific-point-in-time). Или выберите **полную & разностную резервную копию** для [восстановления до определенной точки восстановления](#restore-to-a-specific-recovery-point).

### <a name="restore-as-files"></a>Восстановление в виде файлов

Чтобы восстановить данные резервной копии в виде файлов вместо базы данных, выберите **восстановить как файлы**. Когда файлы будут скопированы в указанное расположение, их можно будет использовать на любом компьютере SAP HANA, где их нужно восстановить в качестве базы данных. Так как эти файлы можно переместить на любой компьютер, теперь вы можете восстанавливать данные в разных подписках и регионах.

1. В меню **восстановить конфигурацию** в разделе **где и как выполнить восстановление** выберите **восстановить как файлы**.
1. Выберите имя сервера **узла** или Hana, для которого необходимо восстановить файлы резервной копии.
1. В **целевом пути на сервере** введите путь к папке на сервере, выбранном на шаге 2. Это расположение, в котором служба будет выводить дамп всех необходимых файлов резервных копий.

    Файлы, которые выводятся в дампе:

    * файлы резервной копии базы данных;
    * файлы каталога;
    * файлы метаданных JSON (для каждого задействованного файла резервной копии).

    Как правило, использование пути к общей сетевой папке или пути к подключенной общей папке Azure, если он указан в качестве пути назначения, упрощает доступ к этим файлам с других компьютеров в той же сети или с подключенной к ним общей папкой Azure.

    >[!NOTE]
    >Чтобы восстановить файлы резервной копии базы данных в общей папке Azure, подключенной к целевой зарегистрированной виртуальной машине, убедитесь, что у привилегированной учетной записи есть разрешения на чтение и запись в общей папке Azure.

    ![Выбор пути назначения](media/sap-hana-db-restore/restore-as-files.png)

1. Выберите **точку восстановления** , в соответствии с которой будут восстановлены все файлы и папки резервных копий.

    ![Выбор точки восстановления](media/sap-hana-db-restore/select-restore-point.png)

1. Все файлы резервных копий, связанные с выбранной точкой восстановления, записываются в целевой путь.
1. В зависимости от типа выбранной точки восстановления (**на определенный момент времени** или **полное и разностное восстановления**) вы увидите одну или несколько папок, созданных в целевом расположении. Одна из папок с именем `Data_<date and time of restore>` содержит полные и разностные резервные копии, а другая папка с именем `Log` — резервные копии журналов.
1. Переместите эти восстановленные файлы на сервер SAP HANA, где они будут восстановлены в качестве базы данных.
1. Затем выполните следующие действия:
    1. Задайте разрешения для папки или каталога, в которых хранятся файлы резервных копий, с помощью следующей команды.

        ```bash
        chown -R <SID>adm:sapsys <directory>
        ```

    1. Выполните следующий набор команд как `<SID>adm`.

        ```bash
        su - <sid>adm
        ```

    1. Создайте файл каталога для восстановления. Извлеките значение **BackupId** из файла метаданных JSON для полной резервной копии. Оно будет использоваться далее в операции восстановления. Убедитесь, что полные резервные копии и резервные копии журналов находятся в разных папках, и удалите файлы каталогов и файлы метаданных JSON в этих папках.

        ```bash
        hdbbackupdiag --generate --dataDir <DataFileDir> --logDirs <LogFilesDir> -d <PathToPlaceCatalogFile>
        ```

        В приведенной выше команде:

        * `<DataFileDir>` — папка, содержащая полные резервные копии;
        * `<LogFilesDir>` — папка, содержащая резервные копии журналов;
        * `<PathToPlaceCatalogFile>` — папка, в которой должен быть размещен созданный файл каталога.

    1. Выполните восстановление с использованием созданного файла каталога: в HANA Studio или с запросом на восстановление HDBSQL. Запросы HDBSQL перечислены ниже.

    * Восстановление на определенный момент времени:

        Если вы создаете новую восстановленную базу данных, выполните команду HDBSQL, чтобы создать новую базу данных `<DatabaseName>`, а затем прервите работу базы данных для восстановления. Но если восстанавливается только существующая база данных, выполните команду HDBSQL, чтобы прервать работу базы данных.

        Затем выполните следующую команду для восстановления базы данных:

        ```hdbsql
        RECOVER DATABASE FOR <DatabaseName> UNTIL TIMESTAMP '<TimeStamp>' CLEAR LOG USING SOURCE '<DatabaseName@HostName>'  USING CATALOG PATH ('<PathToGeneratedCatalogInStep3>') USING LOG PATH (' <LogFileDir>') USING DATA PATH ('<DataFileDir>') USING BACKUP_ID <BackupIdFromJsonFile> CHECK ACCESS USING FILE
        ```

        * `<DatabaseName>` — имя новой или существующей базы данных, которую необходимо восстановить.
        * `<Timestamp>` — точная метка времени для восстановления до точки во времени.
        * `<DatabaseName@HostName>` — имя базы данных, резервная копия которой используется для восстановления, и имя **узла** или сервера SAP HANA, на котором находится эта база данных. Параметр `USING SOURCE <DatabaseName@HostName>` определяет, что резервная копия данных (используемая для восстановления) относится к базе данных с другим идентификатором безопасности или именем, отличающимся от целевого компьютера SAP HANA. Поэтому для восстановления на том же сервере HANA, с которого выполняется резервное копирование, не требуется указывать.
        * `<PathToGeneratedCatalogInStep3>` — Путь к файлу каталога, созданному на **шаге C**
        * `<DataFileDir>` — папка, содержащая полные резервные копии.
        * `<LogFilesDir>` — папка, содержащая резервные копии журналов.
        * `<BackupIdFromJsonFile>`— **BackupId** , извлеченный на **шаге C** .

    * Восстановление до определенной полной или разностной резервной копии:

        Если вы создаете новую восстановленную базу данных, выполните команду HDBSQL, чтобы создать новую базу данных `<DatabaseName>`, а затем прервите работу базы данных для восстановления. Но если восстанавливается только существующая база данных, выполните команду HDBSQL, чтобы прервать работу базы данных:

        ```hdbsql
        RECOVER DATA FOR <DatabaseName> USING BACKUP_ID <BackupIdFromJsonFile> USING SOURCE '<DatabaseName@HostName>'  USING CATALOG PATH ('<PathToGeneratedCatalogInStep3>') USING DATA PATH ('<DataFileDir>')  CLEAR LOG
        ```

        * `<DatabaseName>` — имя новой или существующей базы данных, которую необходимо восстановить.
        * `<Timestamp>` — точная метка времени для восстановления до точки во времени.
        * `<DatabaseName@HostName>` — имя базы данных, резервная копия которой используется для восстановления, и имя **узла** или сервера SAP HANA, на котором находится эта база данных. Параметр `USING SOURCE <DatabaseName@HostName>` определяет, что резервная копия данных (используемая для восстановления) относится к базе данных с другим идентификатором безопасности или именем, отличающимся от целевого компьютера SAP HANA. Поэтому для восстановления на том же сервере HANA, с которого выполняется резервное копирование, его не нужно указывать.
        * `<PathToGeneratedCatalogInStep3>`— путь к файлу каталога, созданному на **шаге C** .
        * `<DataFileDir>` — папка, содержащая полные резервные копии.
        * `<LogFilesDir>` — папка, содержащая резервные копии журналов.
        * `<BackupIdFromJsonFile>`— **BackupId** , извлеченный на **шаге C** .

### <a name="restore-to-a-specific-point-in-time"></a>Восстановление данных до определенной точки во времени

Если вы выбрали **Logs (Point in Time)** (Журналы (восстановление до точки во времени)) как тип восстановления, выполните следующие действия:

1. Выберите точку восстановления в графе журнала и нажмите кнопку **ОК** , чтобы выбрать точку восстановления.

    ![Точка восстановления](media/sap-hana-db-restore/restore-point.png)

1. Чтобы запустить задание восстановления, в меню **Восстановление** щелкните **Восстановить**.

    ![Выбор восстановления](media/sap-hana-db-restore/restore-restore.png)

1. Отслеживать ход восстановления в области **уведомления** или отслеживать его, выбрав пункт **восстановить задания** в меню База данных.

    ![Восстановление успешно запущено](media/sap-hana-db-restore/restore-triggered.png)

### <a name="restore-to-a-specific-recovery-point"></a>Восстановление до определенной точки восстановления

Если вы выбрали **Full & Differential** (Полная и разностная) как тип восстановления, выполните следующие действия:

1. Выберите точку восстановления из списка и нажмите кнопку **ОК** , чтобы выбрать точку восстановления.

    ![Восстановление определенной точки восстановления](media/sap-hana-db-restore/specific-recovery-point.png)

1. Чтобы запустить задание восстановления, в меню **Восстановление** щелкните **Восстановить**.

    ![Запустить задание восстановления](media/sap-hana-db-restore/restore-specific.png)

1. Отслеживать ход восстановления в области **уведомления** или отслеживать его, выбрав пункт **восстановить задания** в меню База данных.

    ![Ход восстановления](media/sap-hana-db-restore/restore-progress.png)

    > [!NOTE]
    > В случае восстановления в контейнере нескольких баз данных (MDC) после того, как системная база данных будет восстановлена на целевом экземпляре, необходимо снова запустить сценарий предварительной регистрации. Только после этого будут выполнены восстановления баз данных клиентов. Дополнительные сведения см. в разделе [Устранение неполадок — MDC Restore](backup-azure-sap-hana-database-troubleshoot.md#multiple-container-database-mdc-restore).

## <a name="cross-region-restore"></a>Восстановление между регионами

В качестве одного из вариантов восстановления. восстановление между регионами (КРР) позволяет восстанавливать SAP HANA базы данных, размещенные на виртуальных машинах Azure, в дополнительном регионе, который является парным регионом Azure.

Чтобы перейти к функции в предварительной версии, ознакомьтесь с [разделом перед началом](./backup-create-rs-vault.md#set-cross-region-restore).

Чтобы узнать, включен ли КРР, следуйте инструкциям в разделе [Настройка восстановления между регионами](backup-create-rs-vault.md#configure-cross-region-restore) .

### <a name="view-backup-items-in-secondary-region"></a>Просмотр элементов резервного копирования в дополнительном регионе

Если КРР включен, можно просмотреть элементы резервного копирования в дополнительном регионе.

1. На портале перейдите к   >  **элементу резервное копирование** хранилища служб восстановления.
1. Выберите **дополнительный регион** , чтобы просмотреть элементы в дополнительном регионе.

>[!NOTE]
>В списке будут показаны только типы управления резервным копированием, поддерживающие функцию КРР. В настоящее время допускается только восстановление данных вторичного региона в дополнительный регион.

![Элементы резервного копирования в дополнительном регионе](./media/sap-hana-db-restore/backup-items-secondary-region.png)

![Базы данных в дополнительном регионе](./media/sap-hana-db-restore/databases-secondary-region.png)

### <a name="restore-in-secondary-region"></a>Восстановление в дополнительном регионе

Пользовательский интерфейс восстановления в дополнительном регионе будет аналогичен пользовательскому интерфейсу восстановления в основном регионе. При настройке сведений в области восстановление конфигурации для настройки восстановления вам будет предложено указать только дополнительные параметры региона.

![Где и как выполнить восстановление](./media/sap-hana-db-restore/restore-secondary-region.png)

>[!NOTE]
>Виртуальную сеть в дополнительном регионе необходимо назначить уникальной, и ее нельзя использовать для других виртуальных машин в этой группе ресурсов.

![Идет активация уведомления о состоянии восстановления](./media/backup-azure-arm-restore-vms/restorenotifications.png)

>[!NOTE]
>
>* После запуска восстановления и на этапе пересылки данных задание восстановления нельзя отменить.
>* Роли Azure, необходимые для восстановления в дополнительном регионе, совпадают с ролями в основном регионе.

### <a name="monitoring-secondary-region-restore-jobs"></a>Мониторинг заданий восстановления в дополнительном регионе

1. На портале выберите   >  **задания резервного копирования** хранилища служб восстановления.
1. Выберите **дополнительный регион** , чтобы просмотреть элементы в дополнительном регионе.

    ![Задания резервного копирования отфильтрованы](./media/sap-hana-db-restore/backup-jobs-secondary-region.png)

## <a name="next-steps"></a>Дальнейшие действия

* [Узнайте, как](sap-hana-db-manage.md) управлять SAP HANA базами данных, резервное копирование которых осуществляется с помощью Azure Backup
