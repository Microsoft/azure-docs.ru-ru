---
title: Резервное копирование и восстановление Oracle Database базы данных 19c на виртуальной машине Linux в Azure с помощью Azure Backup
description: Узнайте, как выполнять резервное копирование и восстановление базы данных Oracle Database 19c с помощью службы Azure Backup.
author: cro27
ms.service: virtual-machines
ms.subservice: oracle
ms.collection: linux
ms.topic: article
ms.date: 01/28/2021
ms.author: cholse
ms.reviewer: dbakevlar
ms.openlocfilehash: 90f86a198ad36c2961f77336092d863953ee45ba
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101673899"
---
# <a name="back-up-and-recover-an-oracle-database-19c-database-on-an-azure-linux-vm-using-azure-backup"></a>Резервное копирование и восстановление Oracle Database базы данных 19c на виртуальной машине Linux в Azure с помощью Azure Backup

В этой статье показано, как использовать Azure Backup для создания моментальных снимков дисков виртуальных машин, включая файлы базы данных и область быстрого восстановления. С помощью Azure Backup можно создать полные моментальные снимки диска, которые будут использоваться в качестве резервных, которые хранятся в [хранилище служб восстановления](../../../backup/backup-azure-recovery-services-vault-overview.md).  Azure Backup также предоставляет резервные копии с единообразным применением приложений, которые гарантируют, что для восстановления данных не требуются дополнительные исправления. Восстановление данных с согласованием на уровне приложений сокращает время восстановления, позволяя быстро вернуть приложение в рабочее состояние.

> [!div class="checklist"]
>
> * Резервное копирование базы данных с помощью резервного копирования с учетом приложений
> * Восстановление базы данных из точки восстановления
> * Восстановление виртуальной машины из точки восстановления

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../../includes/azure-cli-prepare-your-environment.md)]

Чтобы выполнить резервное копирование и восстановление, необходимо сначала создать виртуальную машину Linux с установленным экземпляром Oracle Database 19c. Образ Marketplace, который сейчас используется для создания виртуальной машины, — Oracle  **: Oracle-Database-19-3: Oracle-Database-19-0904: Latest**. Выполните действия, описанные в [кратком руководстве по созданию базы данных Oracle](./oracle-database-quick-create.md) , чтобы создать базу данных Oracle для работы с этим руководством.


## <a name="prepare-the-environment"></a>Подготовка среды

Чтобы подготовить среду, выполните следующие действия.

1. Подключитесь к виртуальной машине.
1. Подготовьте базу данных.

### <a name="connect-to-the-vm"></a>Подключение к виртуальной машине

1. Используйте следующую команду для создания сеанса Secure Shell (SSH) с виртуальной машиной. Замените сочетание IP-адреса и имени узла значением `<publicIpAddress>` вашей виртуальной машины.
    
   ```bash
   ssh azureuser@<publicIpAddress>
   ```
   
1. Переключитесь на *привилегированного* пользователя:

   ```bash
   sudo su -
   ```
    
1. Добавьте пользователя Oracle в файл */etc/sudoers* :

   ```bash
   echo "oracle   ALL=(ALL)      NOPASSWD: ALL" >> /etc/sudoers
   ```

### <a name="prepare-the-database"></a>Подготовка базы данных

В этом шаге предполагается, что у вас есть экземпляр Oracle (*Test*), который выполняется на виртуальной машине с именем *vmoracle19c*.

1. Переключитесь на пользователя *Oracle* .
 
   ```bash
    sudo su - oracle
    ```
    
2. Перед подключением необходимо задать переменную среды ORACLE_SID:
    
    ```bash
    export ORACLE_SID=test;
    ```

    Кроме того, необходимо добавить переменную ORACLE_SID в файл `.bashrc` пользователей `oracle` для последующих операций входа с помощью следующей команды:

    ```bash
    echo "export ORACLE_SID=test" >> ~oracle/.bashrc
    ```
    
3. Запустите прослушиватель Oracle, если он еще не запущен:

    ```output
    $ lsnrctl start
    ```

    Результат должен выглядеть следующим образом:

    ```bash
    LSNRCTL for Linux: Version 19.0.0.0.0 - Production on 18-SEP-2020 03:23:49

    Copyright (c) 1991, 2019, Oracle.  All rights reserved.

    Starting /u01/app/oracle/product/19.0.0/dbhome_1/bin/tnslsnr: please wait...

    TNSLSNR for Linux: Version 19.0.0.0.0 - Production
    System parameter file is /u01/app/oracle/product/19.0.0/dbhome_1/network/admin/listener.ora
    Log messages written to /u01/app/oracle/diag/tnslsnr/vmoracle19c/listener/alert/log.xml
    Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=vmoracle19c.eastus.cloudapp.azure.com)(PORT=1521)))
    Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1521)))

    Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=vmoracle19c.eastus.cloudapp.azure.com)(PORT=1521)))
    STATUS of the LISTENER
    ------------------------
    Alias                     LISTENER
    Version                   TNSLSNR for Linux: Version 19.0.0.0.0 - Production
    Start Date                18-SEP-2020 03:23:49
    Uptime                    0 days 0 hr. 0 min. 0 sec
    Trace Level               off
    Security                  ON: Local OS Authentication
    SNMP                      OFF
    Listener Parameter File   /u01/app/oracle/product/19.0.0/dbhome_1/network/admin/listener.ora
    Listener Log File         /u01/app/oracle/diag/tnslsnr/vmoracle19c/listener/alert/log.xml
    Listening Endpoints Summary...
     (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=vmoracle19c.eastus.cloudapp.azure.com)(PORT=1521)))
    (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1521)))
    The listener supports no services
    The command completed successfully
    ```

4.  Создайте расположение области быстрого восстановления:

    ```bash
    mkdir /u02/fast_recovery_area
    ```

5.  Подключение к базе данных:

    ```bash
    SQL> sqlplus / as sysdba
    ```

6.  Запустите базу данных, если она еще не запущена:

    ```bash
    SQL> startup
    ```

7.  Задайте переменные среды базы данных для области быстрого восстановления:

    ```bash
    SQL> alter system set db_recovery_file_dest_size=4096M scope=both;
    SQL> alter system set db_recovery_file_dest='/u02/fast_recovery_area' scope=both;
    ```
    
8.  Убедитесь, что база данных находится в режиме журнала архивации, чтобы включить оперативное резервное копирование.

    Сначала проверьте состояние архива журнала:

    ```bash
    SQL> SELECT log_mode FROM v$database;

    LOG_MODE
    ------------
    NOARCHIVELOG
    ```

    Если он находится в режиме НОАРЧИВЕЛОГ, выполните следующие команды:

    ```bash
    SQL> SHUTDOWN IMMEDIATE;
    SQL> STARTUP MOUNT;
    SQL> ALTER DATABASE ARCHIVELOG;
    SQL> ALTER DATABASE OPEN;
    SQL> ALTER SYSTEM SWITCH LOGFILE;
    ```

9.  Создайте таблицу для проверки операций резервного копирования и восстановления.

    ```bash
    SQL> create user scott identified by tiger quota 100M on users;
    SQL> grant create session, create table to scott;
    connect scott/tiger
    SQL> create table scott_table(col1 number, col2 varchar2(50));
    SQL> insert into scott_table VALUES(1,'Line 1');
    SQL> commit;
    SQL> quit
    ```

10. Настройте RMAN для резервного копирования в область быстрого восстановления, расположенную на диске виртуальной машины:

    ```bash
    $ rman target /
    RMAN> configure snapshot controlfile name to '/u02/fast_recovery_area/snapcf_ev.f';
    RMAN> configure channel 1 device type disk format '/u02/fast_recovery_area/%d/Full_%d_%U_%T_%s';
    RMAN> configure channel 2 device type disk format '/u02/fast_recovery_area/%d/Full_%d_%U_%T_%s'; 
    ```

11. Подтвердите сведения об изменении конфигурации:

    ```bash
    RMAN> show all;
    ```    

12.  Теперь запустите резервное копирование. Следующая команда выполнит полную резервную копию базы данных, включая архивные файлы журнала, в виде архива в сжатом формате:

     ```bash
     RMAN> backup as compressed backupset database plus archivelog;
     ```

## <a name="using-azure-backup-preview"></a>Использование Azure Backup (Предварительная версия)

Служба Azure Backup предоставляет простые, безопасные и экономичные решения для резервного копирования и восстановления данных из облака Microsoft Azure. Служба Azure Backup предоставляет независимые и изолированные резервные копии для защиты от случайного удаления исходных данных. Резервные копии хранятся в хранилище Служб восстановления со встроенным управлением точками восстановления. Конфигурация и масштабируемость просты, резервные копии оптимизированы, и вы можете легко восстанавливать их по мере необходимости.

Azure Backup служба предоставляет [платформу](../../../backup/backup-azure-linux-app-consistent.md) для обеспечения согласованности приложений во время резервного копирования виртуальных машин Windows и Linux для различных приложений, таких как Oracle, MySQL, Mongo DB и PostgreSQL. Это подразумевает вызов предварительного сценария (для замораживания приложений) перед созданием моментального снимка дисков и вызовом командлета POST-Script (команды для разморозки приложений) после завершения создания моментального снимка для возврата приложений в нормальный режим. В то время как образцы предварительных и посторонних сценариев предоставляются на сайте GitHub, вы отвечаете за создание и обслуживание этих сценариев.

Теперь Azure Backup предоставляет расширенную платформу предварительных сценариев и пост-Script Framework (**в настоящее время находится в предварительной версии**), где служба Azure Backup предоставляет Упакованные предварительные и посторонние скрипты для выбранных приложений. Azure Backup пользователям просто нужно присвоить имя приложению, а затем служба архивации виртуальных машин Azure автоматически вызовет соответствующие скрипты, предшествующие предварительной публикации. Упакованные предварительные и посторонние скрипты будут поддерживаться группой Azure Backup, поэтому пользователи могут быть уверены в поддержке, владении и допустимости этих сценариев. В настоящее время поддерживаемыми приложениями для расширенной платформы являются *Oracle* и *MySQL*.

В этом разделе вы будете использовать Azure Backup расширенную платформу для создания моментальных снимков, совместимых с приложениями, для работающей виртуальной машины и базы данных Oracle. База данных будет переведена в режим резервного копирования, что позволит выполнить транзакционно согласованное оперативное резервное копирование, в то время как Azure Backup создает моментальный снимок дисков виртуальной машины. Моментальный снимок будет полной копией хранилища, а не добавочной или копией моментального снимка записи, поэтому он является эффективным носителем для восстановления базы данных. Преимущество использования Azure Backupных моментальных снимков, согласованных с приложениями, заключается в том, что они чрезвычайно быстро зависят от размера базы данных, а моментальный снимок можно использовать для операций восстановления сразу после выполнения, не дожидаясь его передачи в хранилище служб восстановления.

Чтобы использовать Azure Backup для резервного копирования базы данных, выполните следующие действия.

1. Подготовьте среду для создания резервной копии, учитывающей приложения.
1. Настройка резервных копий, учитывающих приложения.
1. Активация резервной копии виртуальной машины в соответствии с приложениями.

### <a name="prepare-the-environment-for-an-application-consistent-backup"></a>Подготовка среды для резервного копирования с учетом приложений

1. Переключитесь на *привилегированного* пользователя:

   ```bash
   sudo su -
   ```

1. Создать нового пользователя резервной копии:

   ```bash
   useradd -G backupdba azbackup
   ```
   
2. Настройте среду пользователя резервного копирования:

   ```bash
   echo "export ORACLE_SID=test" >> ~azbackup/.bashrc
   echo export ORACLE_HOME=/u01/app/oracle/product/19.0.0/dbhome_1 >> ~azbackup/.bashrc
   echo export PATH='$ORACLE_HOME'/bin:'$PATH' >> ~azbackup/.bashrc
   ```
   
3. Настройте внешнюю проверку подлинности для нового пользователя резервной копии. Пользователь резервного копирования должен иметь доступ к базе данных с использованием внешней проверки подлинности, так что не требует пароля.

   Сначала переключитесь обратно на пользователя *Oracle* :

   ```bash
   su - oracle
   ```

   Войдите в базу данных с помощью sqlplus и проверьте параметры по умолчанию для внешней проверки подлинности:
   
   ```bash
   sqlplus / as sysdba
   SQL> show parameter os_authent_prefix
   SQL> show parameter remote_os_authent
   ```
   
   Результат должен выглядеть, как в следующем примере: 

   ```output
   NAME                                 TYPE        VALUE
   ------------------------------------ ----------- ------------------------------
   os_authent_prefix                    string      ops$
   remote_os_authent                    boolean     FALSE
   ```

   Теперь создайте пользователя базы данных *азбаккуп* с проверкой подлинности извне и предоставьте привилегию сисбаккуп:
   
   ```bash
   SQL> CREATE USER ops$azbackup IDENTIFIED EXTERNALLY;
   SQL> GRANT CREATE SESSION, ALTER SESSION, SYSBACKUP TO ops$azbackup;
   ```

   > [!IMPORTANT] 
   > Если при выполнении инструкции появляется сообщение об ошибке `ORA-46953: The password file is not in the 12.2 format.` `GRANT` , выполните следующие действия, чтобы выполнить миграцию файла орапвд в формат 12,2:
   >
   > 1. Выйдите из sqlplus.
   > 1. Переместите файл пароля, используя старый формат, в новое имя.
   > 1. Перенесите файл пароля.
   > 1. Удалите старый файл.
   > 1. Выполните следующую команду.
   >
   >    ```bash
   >    mv $ORACLE_HOME/dbs/orapwtest $ORACLE_HOME/dbs/orapwtest.tmp
   >    orapwd file=$ORACLE_HOME/dbs/orapwtest input_file=$ORACLE_HOME/dbs/orapwtest.tmp
   >    rm $ORACLE_HOME/dbs/orapwtest.tmp
   >    ```
   >
   > 1. Повторите `GRANT` операцию в sqlplus.
   >
   
4. Создайте хранимую процедуру для записи сообщений резервной копии в журнал предупреждений базы данных:

   ```bash
   sqlplus / as sysdba
   SQL> GRANT EXECUTE ON DBMS_SYSTEM TO SYSBACKUP;
   SQL> CREATE PROCEDURE sysbackup.azmessage(in_msg IN VARCHAR2)
   AS
     v_timestamp     VARCHAR2(32);
   BEGIN
     SELECT TO_CHAR(SYSDATE, 'YYYY-MM-DD HH24:MI:SS')
     INTO v_timestamp FROM DUAL;
     DBMS_OUTPUT.PUT_LINE(v_timestamp || ' - ' || in_msg);
     SYS.DBMS_SYSTEM.KSDWRT(SYS.DBMS_SYSTEM.ALERT_FILE, in_msg);
   END azmessage;
   /
   SQL> SHOW ERRORS
   SQL> QUIT
   ```
   
### <a name="set-up-application-consistent-backups"></a>Настройка резервных копий с постоянным применением приложений  

1. Переключитесь на *привилегированного* пользователя:

   ```bash
   sudo su -
   ```

2. Проверьте папку "etc/Azure". Если это не так, создайте рабочий каталог для резервного копирования с помощью приложений:

   ```bash
   if [ ! -d "/etc/azure" ]; then
      sudo mkdir /etc/azure
   fi
   ```

3. Проверьте наличие в папке "Рабочая нагрузка. conf". Если это не так, создайте файл в каталоге */etc/Azure* с именем *рабочей нагрузки. conf* со следующим содержимым, который должен начинаться с `[workload]` . Если файл уже существует, измените поля так, чтобы они совпадали со следующим содержимым. В противном случае следующая команда создаст файл и заполнит его содержимое:

   ```bash
   echo "[workload]
   workload_name = oracle
   command_path = /u01/app/oracle/product/19.0.0/dbhome_1/bin/
   timeout = 90
   linux_user = azbackup" > /etc/azure/workload.conf
   ```

### <a name="trigger-an-application-consistent-backup-of-the-vm"></a>Активация резервной копии виртуальной машины в соответствии с приложениями

# <a name="portal"></a>[Портал](#tab/azure-portal)

1.  В портал Azure перейдите к группе ресурсов **RG-Oracle** и щелкните виртуальную машину **vmoracle19c**.

2.  В колонке **резервное копирование** создайте новое **хранилище служб восстановления** в группе ресурсов **RG-Oracle** с именем **myVault**.
    Для параметра **выберите политику резервного копирования** используйте **(New) даилиполици**. Если вы хотите изменить периодичность резервного копирования или диапазон хранения, выберите **создать новую политику** .

    ![Страница добавления хранилищ служб восстановления](./media/oracle-backup-recovery/recovery-service-01.png)

3.  Чтобы продолжить, щелкните **включить резервное копирование**.

    > [!IMPORTANT]
    > После нажатия кнопки **Включить резервное копирование** процесс резервного копирования запустится только в запланированное время. Чтобы настроить немедленное резервное копирование, сделайте следующее:

4. На странице Группа ресурсов щелкните только что созданное хранилище служб восстановления **myVault**. Указание. может потребоваться обновить страницу, чтобы увидеть ее.

5.  Перейдите к колонке myVault, щелкните **Архивные элементы** и в столбце **Количество элементов архивации** выберите количество элементов архивации.

    ![Страница сведений о хранилище myVault в хранилищах служб восстановления](./media/oracle-backup-recovery/recovery-service-02.png)

6.  В колонке **Backup Items (Azure Virtual Machine)** (Элементы архивации (виртуальная машина Azure)) в правой части страницы щелкните многоточие (**...**) и выберите **Создать резервную копию**.

    ![Команда "Создать резервную копию" в хранилищах служб восстановления](./media/oracle-backup-recovery/recovery-service-03.png)

7.  Примите значение хранить резервную копию по умолчанию до и нажмите кнопку **ОК** . Дождитесь завершения процесса резервного копирования. 

    Чтобы просмотреть состояние задания резервного копирования, щелкните **задания резервного копирования**.

    ![Страница задания в хранилищах служб восстановления](./media/oracle-backup-recovery/recovery-service-04.png)

    Состояние задания резервного копирования отобразится на следующем рисунке:

    ![Страница задания в хранилищах служб восстановления с отображением состояния](./media/oracle-backup-recovery/recovery-service-05.png)
    
    Обратите внимание, что для выполнения моментального снимка требуется всего несколько секунд, что может занять некоторое время для его перемещения в хранилище, а задание резервного копирования не будет завершено до завершения перемещения.

8. Для согласованных с приложениями резервных копий устраните все ошибки в файле журнала. Файл журнала находится по адресу/Вар/лог/Азуре/Микрософт.Азуре.рековерисервицес.вмснапшотлинукс/екстенсион.лог.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Создайте хранилище служб восстановления.

   ```azurecli
   az backup vault create --location eastus --name myVault --resource-group rg-oracle
   ```

2. Включите защиту резервного копирования для виртуальной машины:

   ```azurecli
   az backup protection enable-for-vm \
      --resource-group rg-oracle \
      --vault-name myVault \
      --vm vmoracle19c \
      --policy-name DefaultPolicy
   ```

3. Запустите резервное копирование, чтобы запустить его сейчас, а не ждать, пока резервное копирование не будет активировано по расписанию по умолчанию (5 AM UTC): 

   ```azurecli
   az backup protection backup-now \
      --resource-group rg-oracle \
      --vault-name myVault \
      --backup-management-type AzureIaasVM \
      --container-name vmoracle19c \
      --item-name vmoracle19c 
   ```

   Ход выполнения задания резервного копирования можно отслеживать с помощью `az backup job list` и `az backup job show` .

---

## <a name="recovery"></a>Восстановление

Чтобы восстановить базу данных, выполните следующие действия.

1. Удалите файлы базы данных.
1. Создайте скрипт восстановления из хранилища служб восстановления.
1. Подключите точку восстановления.
1. Выполните восстановление.

### <a name="remove-the-database-files"></a>Удаление файлов базы данных 

Далее в этой статье вы узнаете, как выполнить тестирование процесса восстановления. Перед тестированием процесса восстановления вам нужно удалить файлы базы данных.

1.  Завершите работу экземпляра Oracle:

    ```bash
    sqlplus / as sysdba
    SQL> shutdown abort
    ORACLE instance shut down.
    ```

2.  Удалите файлы данных и резервные копии:

    ```bash
    sudo su - oracle
    cd /u02/oradata/TEST
    rm -f *.dbf
    cd /u02/fast_recovery_area
    rm -f *
    ```

### <a name="generate-a-restore-script-from-the-recovery-services-vault"></a>Создание скрипта восстановления из хранилища служб восстановления

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. В портал Azure найдите элемент хранилища служб восстановления *myVault* и выберите его.

    ![Элементы архивации хранилища myVault в хранилищах служб восстановления](./media/oracle-backup-recovery/recovery-service-06.png)

2. В колонке " **Обзор** " выберите **элементы архивации** и выберите **Виртуальная машина Azure**, для которой в списке должно быть провер количество элементов резервной копии.

    ![Число элементов архивации виртуальной машины Azure в хранилищах служб восстановления](./media/oracle-backup-recovery/recovery-service-07.png)

3. На странице элементы архивации (виртуальные машины Azure) **vmoracle19c** виртуальной машины. Нажмите кнопку с многоточием справа, чтобы открыть меню и выбрать пункт **Восстановление файлов**.

    ![Снимок экрана страницы восстановления файлов в хранилищах служб восстановления](./media/oracle-backup-recovery/recovery-service-08.png)

4. В области **Восстановление файлов (предварительная версия)** щелкните **Скачать скрипт**. Затем сохраните файл загрузки (. Копировать) в папку на клиентском компьютере. Для запуска скрипта создается пароль. Скопируйте пароль в файл для последующего использования. 

    ![Параметры сохранения файла скачивания сценария](./media/oracle-backup-recovery/recovery-service-09.png)

5. Скопируйте в виртуальную машину файл с расширением. Копировать.

    В следующем примере показано, как использовать команду безопасного копирования для перемещения файла на виртуальную машину. Вы также можете скопировать содержимое в буфер обмена, а затем вставить его в новый файл, установленный на виртуальной машине.

    > [!IMPORTANT]
    > Чтобы перейти к следующему примеру, вы должны обновить значения IP-адреса и папки. Значения должны соответствовать папке, в которой сохранен файл.
    >

    ```bash
    $ scp vmoracle19c_xxxxxx_xxxxxx_xxxxxx.py azureuser@<publicIpAddress>:/tmp
    ```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы получить список точек восстановления для виртуальной машины, используйте команду AZ Backup точек восстановления. В этом примере мы выбираем последнюю точку восстановления для виртуальной машины с именем myVM, защищенной в хранилище myRecoveryServicesVault:

```azurecli
   az backup recoverypoint list \
      --resource-group rg-oracle \
      --vault-name myVault \
      --backup-management-type AzureIaasVM \
      --container-name vmoracle19c \
      --item-name vmoracle19c \
      --query [0].name \
      --output tsv
```

Чтобы получить сценарий, который подключает точку восстановления к виртуальной машине, используйте команду az backup restore files mount-rp. В следующем примере мы получаем скрипт для виртуальной машины с именем myVM, защищенной в хранилище myRecoveryServicesVault.

Замените myRecoveryPointName именем точки восстановления, полученной в предыдущей команде:

```azurecli
   az backup restore files mount-rp \
      --resource-group rg-oracle \
      --vault-name myVault \
      --container-name vmoracle19c \
      --item-name vmoracle19c \
      --rp-name myRecoveryPointName
```

Сценарий скачивается, и отображается пароль, как показано в следующем примере:

```bash
   File downloaded: vmoracle19c_eus_4598131610710119312_456133188157_6931c635931f402eb543ee554e1cf06f102c6fc513d933.py. Use password c4487e40c760d29
```

Скопируйте в виртуальную машину файл с расширением. Копировать.

В следующем примере показано, как использовать команду безопасного копирования для перемещения файла на виртуальную машину. Вы также можете скопировать содержимое в буфер обмена, а затем вставить его в новый файл, установленный на виртуальной машине.

> [!IMPORTANT]
> Чтобы перейти к следующему примеру, вы должны обновить значения IP-адреса и папки. Значения должны соответствовать папке, в которой сохранен файл.
>

```bash
$ scp vmoracle19c_xxxxxx_xxxxxx_xxxxxx.py azureuser@<publicIpAddress>:/tmp
```
---

### <a name="mount-the-restore-point"></a>Подключение точки восстановления

1. Создайте точку подключения восстановления и скопируйте в нее скрипт.

    В следующем примере создайте каталог */RESTORE* для подключения к файлу, переместите файл в каталог и измените файл, чтобы он принадлежал привилегированному пользователю и был создан исполняемый файл.

    ```bash 
    ssh azureuser@<publicIpAddress>
    sudo su -
    mkdir /restore
    chmod 777 /restore
    cd /restore
    cp /tmp/vmoracle19c_xxxxxx_xxxxxx_xxxxxx.py /restore
    chmod 755 /restore/vmoracle19c_xxxxxx_xxxxxx_xxxxxx.py
    ```
    
    Теперь выполните сценарий, чтобы восстановить резервную копию. Вам будет предложено ввести пароль, созданный в портал Azure. 
  
   ```bash
    ./vmoracle19c_xxxxxx_xxxxxx_xxxxxx.py
    ```

    В следующем примере показано, что вы должны увидеть после выполнения предыдущего скрипта. Когда появится запрос продолжить, введите **Y**.

    ```output
    Microsoft Azure VM Backup - File Recovery
    ______________________________________________
    Please enter the password as shown on the portal to securely connect to the recovery point. : b1ad68e16dfafc6

    Connecting to recovery point using ISCSI service...

    Connection succeeded!

    Please wait while we attach volumes of the recovery point to this machine...

    ************ Volumes of the recovery point and their mount paths on this machine ************

    Sr.No.  |  Disk  |  Volume  |  MountPath

    1)  | /dev/sdc  |  /dev/sdc1  |  /restore/vmoracle19c-20201215123912/Volume1

    2)  | /dev/sdd  |  /dev/sdd1  |  /restore/vmoracle19c-20201215123912/Volume2

    3)  | /dev/sdd  |  /dev/sdd2  |  /restore/vmoracle19c-20201215123912/Volume3

    4)  | /dev/sdd  |  /dev/sdd15  |  /restore/vmoracle19c-20201215123912/Volume5

    The following partitions failed to mount since the OS couldn't identify the filesystem.

    ************ Volumes from unknown filesystem ************

    Sr.No.  |  Disk  |  Volume  |  Partition Type

    1)  | /dev/sdb  |  /dev/sdb14  |  BIOS Boot partition

    Please refer to '/restore/vmoracle19c-2020XXXXXXXXXX/Scripts/MicrosoftAzureBackupILRLogFile.log' for more details.

    ************ Open File Explorer to browse for files. ************

    After recovery, remove the disks and close the connection to the recovery point by clicking the 'Unmount Disks' button from the portal or by using the relevant unmount command in case of powershell or CLI.

    After unmounting disks, run the script with the parameter 'clean' to remove the mount paths of the recovery point from this machine.

    Please enter 'q/Q' to exit...
    ```

2. Будет подтвержден доступ к подключенным томам.

    Чтобы выйти, введите **q**, а затем найдите подключенные тома. Чтобы создать список добавленных томов, в командной строке введите **df-h**.
    
    ```
    [root@vmoracle19c restore]# df -h
    Filesystem      Size  Used Avail Use% Mounted on
    devtmpfs        3.8G     0  3.8G   0% /dev
    tmpfs           3.8G     0  3.8G   0% /dev/shm
    tmpfs           3.8G   17M  3.8G   1% /run
    tmpfs           3.8G     0  3.8G   0% /sys/fs/cgroup
    /dev/sdd2        30G  9.6G   18G  36% /
    /dev/sdb1       126G  736M  119G   1% /u02
    /dev/sda1       497M  199M  298M  41% /boot
    /dev/sda15      495M  9.7M  486M   2% /boot/efi
    tmpfs           771M     0  771M   0% /run/user/54322
    /dev/sdc1       126G  2.9G  117G   3% /restore/vmoracle19c-20201215123912/Volume1
    /dev/sdd1       497M  199M  298M  41% /restore/vmoracle19c-20201215123912/Volume2
    /dev/sdd2        30G  9.6G   18G  36% /restore/vmoracle19c-20201215123912/Volume3
    /dev/sdd15      495M  9.7M  486M   2% /restore/vmoracle19c-20201215123912/Volume5
    ```

### <a name="perform-recovery"></a>Выполнение восстановления

1. Скопируйте отсутствующие файлы резервной копии обратно в область быстрого восстановления:

    ```bash
    cd /restore/vmoracle19c-2020XXXXXXXXXX/Volume1/fast_recovery_area/TEST
    cp * /u02/fast_recovery_area/TEST
    cd /u02/fast_recovery_area/TEST
    chown -R oracle:oinstall *
    ```

2. Следующие команды используют RMAN для восстановления отсутствующих файлов данных и восстановления базы.

    ```bash
    sudo su - oracle
    rman target /
    RMAN> startup mount;
    RMAN> restore database;
    RMAN> recover database;
    RMAN> alter database open;
    ```
    
3. Убедитесь, что содержимое базы данных восстановлено полностью:

    ```bash
    RMAN> SELECT * FROM scott.scott_table;
    ```

4. Отключите точку восстановления.

   ```bash
   umount /restore/vmoracle19c-20210107110037/Volume*
   ```

    На портале Azure в колонке **Восстановление файлов (предварительная версия)** нажмите кнопку **Unmount Disks** (Отключить диски).

    ![Команда отключения дисков](./media/oracle-backup-recovery/recovery-service-10.png)
    
    Можно также отключить тома восстановления, запустив сценарий Python снова с параметром **-Clean** .

## <a name="restore-the-entire-vm"></a>Восстановление всей виртуальной машины

Вместо того чтобы восстанавливать удаленные файлы из хранилищ служб восстановления, можно восстановить всю виртуальную машину.

Чтобы восстановить всю виртуальную машину, выполните следующие действия.

1. Останавливаюте и удаляйте vmoracle19c.
1. Восстановите виртуальную машину.
1. Задайте общедоступный IP-адрес.
1. Подключитесь к виртуальной машине.
1. Запустите базу данных на этапе подключения и выполните восстановление.

### <a name="stop-and-delete-vmoracle19c"></a>Останавливаюте и удаляйте vmoracle19c

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. В портал Azure перейдите на виртуальную машину **vmoracle19c** и нажмите кнопку " **Закрыть**".

1. Если виртуальная машина больше не работает, выберите **Удалить** , а затем нажмите **кнопку Да**.

   ![Команда удаления хранилища](./media/oracle-backup-recovery/recover-vm-01.png)

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Останавливает и освобождает виртуальную машину:

    ```azurecli
    az vm deallocate --resource-group rg-oracle --name vmoracle19c
    ```

2. Удалите виртуальную машину. При появлении запроса введите "y":

    ```azurecli
    az vm delete --resource-group rg-oracle --name vmoracle19c
    ```

---

### <a name="recover-the-vm"></a>Восстановление виртуальной машины

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. Создайте учетную запись хранения для промежуточной подготовки в портал Azure.

   1. В портал Azure выберите **+ создать ресурс** и найдите и выберите **учетная запись хранения**.
    
      ![Снимок экрана, на котором показано, где можно создать ресурс.](./media/oracle-backup-recovery/storage-1.png)
    
    
   1. На странице Создание учетной записи хранения выберите существующую группу ресурсов **RG-Oracle**, присвойте имя учетной записи хранения **Оракресторе** и выберите **Storage v2 (Женералпурпосе v2)** для типа учетной записи. Измените репликацию на **локально избыточное хранилище (LRS)** и установите для параметра производительность значение **Standard**. Убедитесь, что в качестве расположения выбран тот же регион, что и для всех остальных ресурсов в группе ресурсов. 
    
      ![Страница добавления учетной записи хранения](./media/oracle-backup-recovery/recovery-storage-1.png)
   
   1. Щелкните Проверка + создать, а затем — создать.

2. В портал Azure найдите элемент хранилища служб восстановления *myVault* и щелкните его.

    ![Элементы архивации хранилища myVault в хранилищах служб восстановления](./media/oracle-backup-recovery/recovery-service-06.png)
    
3.  В колонке " **Обзор** " выберите **элементы архивации** и выберите **Виртуальная машина Azure**, для которой в списке должно быть провер количество элементов резервной копии.

    ![Число элементов архивации виртуальной машины Azure в хранилищах служб восстановления](./media/oracle-backup-recovery/recovery-service-07.png)

4.  На странице элементы архивации (виртуальные машины Azure) отобразится страница **vmoracle19c** виртуальной машины. Щелкните имя виртуальной машины.

    ![Страница восстановления виртуальной машины](./media/oracle-backup-recovery/recover-vm-02.png)

5.  В колонке **vmoracle19c** выберите точку восстановления с типом согласованности **согласованности приложений** и нажмите кнопку с многоточием (**...**) справа, чтобы открыть меню.  В меню щелкните **восстановить виртуальную машину**.

    ![Команда восстановления виртуальной машины](./media/oracle-backup-recovery/recover-vm-03.png)

6.  В колонке **восстановить виртуальную машину** выберите **создать** и **Создайте новую виртуальную машину**. Введите имя виртуальной машины **vmoracle19c** и выберите виртуальную машину **vmoracle19cVNET**. подсеть будет автоматически заполнена на основе выбора виртуальной сети. Для процесса восстановления виртуальной машины требуется учетная запись хранения Azure в той же группе ресурсов и регионе. Вы можете выбрать учетную запись хранения **ораресторе** , которая была настроена ранее.

    ![Значения восстановления конфигурации](./media/oracle-backup-recovery/recover-vm-04.png)

7.  Чтобы восстановить виртуальную машину, нажмите кнопку **Восстановление**.

8.  Чтобы просмотреть состояние процесса восстановления, щелкните **Задания**, а затем — **Задания архивации**.

    ![Команда состояния заданий резервного копирования](./media/oracle-backup-recovery/recover-vm-05.png)

    Щелкните операцию " **выполняется** восстановление", чтобы отобразить состояние процесса восстановления:

    ![Состояние процесса восстановления](./media/oracle-backup-recovery/recover-vm-06.png)

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы настроить учетную запись хранения и файловый ресурс, выполните следующие команды в Azure CLI.

1. Создайте учетную запись хранения в той же группе ресурсов и в том же расположении, что и виртуальная машина:

   ```azurecli
   az storage account create -n orarestore -g rg-oracle -l eastus --sku Standard_LRS
   ```

2. Получите список доступных точек восстановления. 

   ```azurecli
   az backup recoverypoint list \
      --resource-group rg-oracle \
      --vault-name myVault \
      --backup-management-type AzureIaasVM \
      --container-name vmoracle19c \
      --item-name vmoracle19c \
      --query [0].name \
      --output tsv
   ```

3. Восстановите точку восстановления в учетную запись хранения. Замените `<myRecoveryPointName>` точкой восстановления из списка, созданного на предыдущем шаге:

   ```azurecli
   az backup restore restore-disks \
      --resource-group rg-oracle \
      --vault-name myVault \
      --container-name vmoracle19c \
      --item-name vmoracle19c \
      --storage-account orarestore \
      --rp-name <myRecoveryPointName> \
      --target-resource-group rg-oracle
   ```

4. Получение сведений о задании восстановления. Следующая команда получает дополнительные сведения о запущенном восстановленном задании, включая его имя, которое требуется для получения URI шаблона. 

   ```azurecli
   az backup job list \
       --resource-group rg-oracle \
       --vault-name myVault \
       --output table
   ```

   Результат будет выглядеть примерно так `(Note down the name of the restore job)` :

   ```output
   Name                                  Operation        Status     Item Name    Start Time UTC                    Duration
   ------------------------------------  ---------------  ---------  -----------  --------------------------------  --------------
   c009747a-0d2e-4ac9-9632-f695bf874693  Restore          Completed  vmoracle19c  2021-01-10T21:46:07.506223+00:00  0:03:06.634177
   6b779c98-f57a-4db1-b829-9e8eab454a52  Backup           Completed  vmoracle19c  2021-01-07T10:11:15.784531+00:00  0:21:13.220616
   502bc7ae-d429-4f0f-b78e-51d41b7582fc  ConfigureBackup  Completed  vmoracle19c  2021-01-07T09:43:55.298755+00:00  0:00:30.839674
   ```

5. Получите сведения об универсальном коде ресурса (URI), который будет использоваться для повторного создания виртуальной машины. Замените имя задания восстановления из предыдущего шага на `<RestoreJobName>` .

    ```azurecli
      az backup job show \
        -v myVault \
        -g rg-oracle \
        -n <RestoreJobName> \
        --query properties.extendedInfo.propertyBag
    ```

   Выходные данные похожи на следующие:

   ```output
   {
   "Config Blob Container Name": "vmoracle19c-75aefd4b34c64dd39fdcd3db579783f2",
   "Config Blob Name": "config-vmoracle19c-c009747a-0d2e-4ac9-9632-f695bf874693.json",
   "Config Blob Uri": "https://orarestore.blob.core.windows.net/vmoracle19c-75aefd4b34c64dd39fdcd3db579783f2/config-vmoracle19c-c009747a-0d2e-4ac9-9632-f695bf874693.json",
   "Job Type": "Recover disks",
   "Recovery point time ": "1/7/2021 10:11:19 AM",
   "Target Storage Account Name": "orarestore",
   "Target resource group": "rg-oracle",
   "Template Blob Uri": "https://orarestore.blob.core.windows.net/vmoracle19c-75aefd4b34c64dd39fdcd3db579783f2/azuredeployc009747a-0d2e-4ac9-9632-f695bf874693.json"
   }
   ```

   Имя шаблона, расположенное в конце URI шаблона BLOB-объекта, который в этом примере имеет значение `azuredeployc009747a-0d2e-4ac9-9632-f695bf874693.json` , и имя контейнера BLOB-объектов, которое указано в `vmoracle19c-75aefd4b34c64dd39fdcd3db579783f2` списке. 

   Используйте следующие значения в следующей команде, чтобы назначить переменные при подготовке к созданию виртуальной машины. Ключ SAS создается для контейнера хранилища с 30-минутной длительностью.  


   ```azurecli
   expiretime=$(date -u -d "30 minutes" '+%Y-%m-%dT%H:%MZ')
   connection=$(az storage account show-connection-string \
    --resource-group rg-oracle \
    --name orarestore \
    --query connectionString)
   token=$(az storage blob generate-sas \
    --container-name <ContainerName> \
    --name <TemplateName> \
    --expiry $expiretime \
    --permissions r \
    --output tsv \
    --connection-string $connection)
   url=$(az storage blob url \
    --container-name <ContainerName> \
    --name <TemplateName> \
    --connection-string $connection \
    --output tsv)
   ```

   Теперь разверните шаблон, чтобы создать виртуальную машину.

   ```azurecli
   az deployment group create \
      --resource-group rg-oracle \
      --template-uri $url?$token
   ```

   Вам будет предложено ввести имя виртуальной машины.

---

### <a name="set-the-public-ip-address"></a>Установка общедоступного IP-адреса

После восстановления ВИРТУАЛЬНОЙ машины необходимо переназначить исходный IP-адрес новой виртуальной машине.

# <a name="portal"></a>[Портал](#tab/azure-portal)

1.  В портал Azure перейдите к виртуальной машине **vmoracle19c**. Вы заметите, что ему был назначен новый общедоступный IP-адрес и сетевой адаптер, аналогичный vmoracle19c-NIC-XXXXXXXXXXXX, но у него нет DNS-адреса. Когда исходная виртуальная машина удалила общедоступный IP-адрес и сетевой адаптер, они будут сохранены, и следующие шаги повторно присоединяют их к новой виртуальной машине.

    ![Список общедоступных IP-адресов](./media/oracle-backup-recovery/create-ip-01.png)

2.  Остановка виртуальной машины

    ![Создание IP-адреса](./media/oracle-backup-recovery/create-ip-02.png)

3.  К **сети**

    ![Связывание IP-адреса](./media/oracle-backup-recovery/create-ip-03.png)

4.  Щелкните **Подключить сетевой интерфейс**, выберите оригинальный сетевой адаптер * * vmoracle19cVMNic, с которым по-прежнему связан исходный общедоступный IP-адрес, и нажмите кнопку **ОК** .

    ![Выбор значений типа ресурса и сетевого интерфейса](./media/oracle-backup-recovery/create-ip-04.png)

5.  Теперь необходимо отключить сетевую карту, созданную с помощью операции восстановления виртуальной машины, так как она настроена в качестве основного интерфейса. Щелкните **Отключить сетевой интерфейс** и выберите новую сетевую карту, подобную **vmoracle19c-NIC-XXXXXXXXXXXX**, а затем нажмите кнопку **ОК** .

    ![Снимок экрана, на котором показано, где выбрать вариант отсоединить сетевой интерфейс.](./media/oracle-backup-recovery/create-ip-05.png)
    
    Теперь созданная виртуальная машина будет иметь исходную сетевую карту, связанную с исходным IP-адресом и правилами группы безопасности сети.
    
    ![Значение IP-адреса](./media/oracle-backup-recovery/create-ip-06.png)
    
6.  Вернитесь к **обзору** и нажмите кнопку **Пуск** . 

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Останавливает и освобождает виртуальную машину:

   ```azurecli
   az vm deallocate --resource-group rg-oracle --name vmoracle19c
   ```

2. Вывод списка текущих, восстановленных сетевых адаптеров виртуальных машин

   ```azurecli
   az vm nic list --resource-group rg-oracle --vm-name vmoracle19c
   ```

   Выходные данные будут выглядеть так, как показано на этой странице, в которой отображается имя созданного сетевого адаптера `vmoracle19cRestoredNICc2e8a8a4fc3f47259719d5523cd32dcf`

   ```output
   {
    "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx/resourceGroups/rg-oracle/providers/Microsoft.Network/networkInterfaces/vmoracle19cRestoredNICc2e8a8a4fc3f47259719d5523cd32dcf",
    "primary": true,
    "resourceGroup": "rg-oracle"
   }
   ```

3. Присоедините оригинальный сетевой адаптер, который должен иметь имя `<VMName>VMNic` , в данном случае `vmoracle19cVMNic` . Исходный общедоступный IP-адрес все еще подключен к этой сетевой карте и будет восстановлен на виртуальной машине при повторном подключении сетевого адаптера. 

   ```azurecli
   az vm nic add --nics vmoracle19cVMNic --resource-group rg-oracle --vm-name vmoracle19c
   ```

4. Отсоединить созданную для восстановления сетевую карту

   ```azurecli
   az vm nic remove --nics vmoracle19cRestoredNICc2e8a8a4fc3f47259719d5523cd32dcf --resource-group rg-oracle --vm-name vmoracle19c
   ```

5. Запустите виртуальную машину:

   ```azurecli
   az vm start --resource-group rg-oracle --name vmoracle19c
   ```

---

### <a name="connect-to-the-vm"></a>Подключение к виртуальной машине

Чтобы подключиться к виртуальной машине, используйте следующий скрипт:

```azurecli
ssh <publicIpAddress>
```

### <a name="start-the-database-to-mount-stage-and-perform-recovery"></a>Запуск базы данных на этапе подключения и выполнение восстановления

1. Возможно, экземпляр выполняется при попытке запуска базы данных при загрузке виртуальной машины. Однако база данных требует восстановления и, скорее всего, находится на этапе подключения, поэтому сначала выполняется подготовительное завершение работы.

    ```bash
    $ sudo su - oracle
    $ sqlplus / as sysdba
    SQL> shutdown immediate
    SQL> startup mount
    SQL> recover automatic database;
    SQL> alter database open;
    ```
    
1. Убедитесь, что содержимое базы данных восстановлено:

    ```bash
    SQL> select * from scott.scott_table;
    ```

Резервное копирование и восстановление Oracle Database базы данных 19c на виртуальной машине Linux в Azure теперь завершено.

## <a name="delete-the-vm"></a>Удаление виртуальной машины

Вы можете удалить ставшие ненужными группу ресурсов, виртуальную машину и все связанные с ней ресурсы, использовав следующую команду:

```azurecli
az group delete --name rg-oracle
```

## <a name="next-steps"></a>Дальнейшие действия

Изучите [руководство по созданию высокодоступных виртуальных машин](../../linux/create-cli-complete.md).

[Изучите примеры развертывания виртуальных машин с помощью интерфейса командной строки](https://github.com/Azure-Samples/azure-cli-samples/tree/master/virtual-machine).