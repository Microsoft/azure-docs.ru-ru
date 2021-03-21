---
title: Резервное копирование Oracle Database базы данных 19c на виртуальной машине Linux в Azure с помощью RMAN и службы хранилища Azure
description: Узнайте, как создать резервную копию базы данных 19c Oracle Database в облачном хранилище Azure.
author: cro27
ms.service: virtual-machines
ms.subservice: oracle
ms.collection: linux
ms.topic: article
ms.date: 01/28/2021
ms.author: cholse
ms.reviewer: dbakevlar
ms.openlocfilehash: a6ce5446bd6470ef7a829925646d486801b28ebc
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101670025"
---
# <a name="back-up-and-recover-an-oracle-database-19c-database-on-an-azure-linux-vm-using-azure-storage"></a>Резервное копирование и восстановление Oracle Database базы данных 19c на виртуальной машине Linux в Azure с помощью службы хранилища Azure

В этой статье показано использование службы хранилища Azure в качестве носителя для резервного копирования и восстановления базы данных Oracle, работающей на виртуальной машине Azure. Резервное копирование базы данных осуществляется с помощью Oracle RMAN в хранилище файлов Azure, подключенном к виртуальной машине по протоколу SMB. Использование службы хранилища Azure для резервного копирования носителей чрезвычайно экономично и производительно. Однако для очень больших баз данных Azure Backup предоставляет лучшее решение.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../../includes/azure-cli-prepare-your-environment.md)]

- Чтобы выполнить резервное копирование и восстановление, необходимо сначала создать виртуальную машину Linux с установленным экземпляром Oracle Database 19c. Образ Marketplace, который сейчас используется для создания виртуальной машины, — Oracle  **: Oracle-Database-19-3: Oracle-Database-19-0904: Latest**. Выполните действия, описанные в [кратком руководстве по созданию базы данных Oracle](./oracle-database-quick-create.md) , чтобы создать базу данных Oracle для работы с этим руководством.

## <a name="prepare-the-database-environment"></a>Подготовка среды базы данных

1. Используйте следующую команду для создания сеанса Secure Shell (SSH) с виртуальной машиной. Замените сочетание IP-адреса и имени узла значением `publicIpAddress` вашей виртуальной машины.
    
   ```bash
   ssh azureuser@<publicIpAddress>
   ```
   
2. Переключитесь на ***привилегированного*** пользователя:
 
   ```bash
   sudo su -
   ```
    
3. Добавьте пользователя Oracle в файл ***/etc/sudoers*** :

   ```bash
   echo "oracle   ALL=(ALL)      NOPASSWD: ALL" >> /etc/sudoers
   ```

4. В этом шаге предполагается, что у вас есть экземпляр Oracle (Test), который выполняется на виртуальной машине с именем *vmoracle19c*.

   Переключитесь на пользователя *Oracle* .

   ```bash
   sudo su - oracle
   ```
    
5. Перед подключением необходимо задать переменную среды ORACLE_SID:
    
    ```bash
    export ORACLE_SID=test;
    ```
   
   Кроме того, необходимо добавить переменную ORACLE_SID в файл `.bashrc` пользователей `oracle` для последующих операций входа с помощью следующей команды:

    ```bash
    echo "export ORACLE_SID=test" >> ~oracle/.bashrc
    ```
    
6. Запустите прослушиватель Oracle, если он еще не запущен:
    
   ```bash
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

7.  Создайте расположение области быстрого восстановления:

    ```bash
    mkdir /u02/fast_recovery_area
    ```

8.  Подключение к базе данных:

    ```bash
    sqlplus / as sysdba
    ```

9.  Запустите базу данных, если она еще не запущена:

    ```bash
    SQL> startup
    ```

10. Задайте переменные среды базы данных для области быстрого восстановления:

    ```bash
    SQL>  system set db_recovery_file_dest_size=4096M scope=both;
    SQL> alter system set db_recovery_file_dest='/u02/fast_recovery_area' scope=both;
    ```
    
11. Убедитесь, что база данных находится в режиме журнала архивации, чтобы включить оперативное резервное копирование.
    Сначала проверьте состояние архива журнала:

    ```bash
    SQL> SELECT log_mode FROM v$database;

    LOG_MODE
    ------------
    NOARCHIVELOG
    ```

    В режиме НОАРЧИВЕЛОГ выполните следующие команды в sqlplus:

    ```bash
    SQL> SHUTDOWN IMMEDIATE;
    SQL> STARTUP MOUNT;
    SQL> ALTER DATABASE ARCHIVELOG;
    SQL> ALTER DATABASE OPEN;
    SQL> ALTER SYSTEM SWITCH LOGFILE;
    ```

12. Создайте таблицу для проверки операций резервного копирования и восстановления.

    ```bash
    SQL> create user scott identified by tiger quota 100M on users;
    SQL> grant create session, create table to scott;
    SQL> connect scott/tiger
    SQL> create table scott_table(col1 number, col2 varchar2(50));
    SQL> insert into scott_table VALUES(1,'Line 1');
    SQL> commit;
    SQL> quit
    ```

## <a name="back-up-to-azure-files"></a>Резервное копирование в службу файлов Azure

Чтобы создать резервную копию в службе "файлы Azure", выполните следующие действия.

1. Настройка хранилища файлов Azure.
1. Подключите файловый ресурс службы хранилища Azure к виртуальной машине.
1. Архивируйте базу данных.
1. Восстановить и восстановить базу данных.

### <a name="set-up-azure-file-storage"></a>Настройка хранилища файлов Azure

На этом шаге выполняется резервное копирование базы данных Oracle с помощью диспетчера восстановления Oracle (RMAN) в хранилище файлов Azure. Файловые ресурсы Azure — это полностью управляемые общие файловые ресурсы, которые находятся в облаке. Доступ к ним можно получить с помощью протокола SMB или протокола сетевой файловой системы (NFS). На этом этапе описывается создание общей папки, использующей протокол SMB для подключения к виртуальной машине. Сведения о подключении с помощью NFS см. в статье [Подключение хранилища BLOB-объектов с помощью протокола nfs 3,0](../../../storage/blobs/network-file-system-protocol-support-how-to.md).

При подключении файлов Azure мы будем использовать `cache=none` для отключения кэширования данных файловых ресурсов. И для того, чтобы файлы, созданные в общей папке, принадлежали пользователю Oracle, задайте `uid=oracle` `gid=oinstall` также параметры и. 

# <a name="portal"></a>[Портал](#tab/azure-portal)

Сначала настройте учетную запись хранения.

1. Настройка хранилища файлов в портал Azure

    В портал Azure выберите ***+ создать ресурс** _, а затем найдите и выберите _ *_учетная запись хранения_* .*
    
    ![Снимок экрана, на котором показано, где создать ресурс и выбрать учетную запись хранения.](./media/oracle-backup-recovery/storage-1.png)
    
2. На странице Создание учетной записи хранения выберите существующую группу ресурсов ***RG-Oracle** _, присвойте имя учетной записи хранения _*_Oracbkup1_*_ и выберите _*_Storage v2 (Женералпурпосе v2)_*_ для типа учетной записи. Измените репликацию на _*_локально избыточное хранилище (LRS)_*_ и задайте для параметра производительность значение _ *_Standard_* *. Убедитесь, что в качестве расположения выбран тот же регион, что и для всех остальных ресурсов в группе ресурсов. 
    
    ![Снимок экрана, на котором показано, где выбрать существующую группу ресурсов.](./media/oracle-backup-recovery/file-storage-1.png)
   
   
3. Откройте вкладку ***Advanced** _ и в разделе файлы Azure задайте _*_большие файловые ресурсы_*_ , чтобы они были _включены_* *. Щелкните Проверка + создать, а затем — создать.
    
    ![Снимок экрана, на котором показано, где можно задать большие файловые ресурсы для включения.](./media/oracle-backup-recovery/file-storage-2.png)
    
    
4. После создания учетной записи хранения перейдите к ресурсу и выберите ***файловые ресурсы*** .
    
    ![Снимок экрана, на котором показано, где выбрать файловые ресурсы.](./media/oracle-backup-recovery/file-storage-3.png)
    
5. Щелкните ***+ общий файловый ресурс** _, а в колонке _*_новый файловый ресурс_*_ — имя общей папки _*_orabkup1_*_. Установите _*_квоту_*_ равным _*_10240_*_ гиб и проверьте, _*_оптимизирована ли транзакция_*_ как уровень. Квота отражает верхнюю границу, до которой может увеличиваться файловый ресурс. Так как мы используем хранилище уровня "Стандартный", ресурсы PAYG и не подготавливаются, поэтому установка этого параметра равным 10 Тиб не приведет к затратам, чем вы используете. Если стратегия резервного копирования требует большего объема хранилища, необходимо задать для квоты соответствующий уровень, чтобы вместить все резервные копии.   Завершив создание новой колонки общей папки, щелкните значок _ *_создать_*.
    
    ![Снимок экрана, на котором показано, куда добавить новый файловый ресурс.](./media/oracle-backup-recovery/file-storage-4.png)
    
    
6. При создании щелкните ***orabkup1*** на странице Параметры общей папки. 
    Щелкните вкладку ***подключить** _, чтобы открыть колонку подключение, а затем перейдите на вкладку _ *_Linux_**. Скопируйте команды, предоставленные для подключения общей папки по протоколу SMB. 
    
    ![Страница добавления учетной записи хранения](./media/oracle-backup-recovery/file-storage-5.png)

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы настроить учетную запись хранения и общую папку, выполните следующие команды в Azure CLI.

1. Создайте учетную запись хранения в той же группе ресурсов и в том же расположении, что и виртуальная машина:
   ```azurecli
   az storage account create -n orabackup1 -g rg-oracle -l eastus --sku Standard_LRS --enable-large-file-share
   ```

2. Создайте общую папку в учетной записи хранения с квотой 10 Тиб:

   ```azurecli
   az storage share create --account-name orabackup1 --name orabackup --quota 10240
   ```

3. Получите первичный ключ учетной записи хранения (key1), который требуется при подключении общей папки к виртуальной машине:

   ```azurecli
   az storage account keys list --resource-group rg-oracle --account-name orabackup1
   ```

---

### <a name="mount-the-azure-storage-file-share-to-your-vm"></a>Подключение общей папки службы хранилища Azure к виртуальной машине

1. Создайте точку подключения:

   ```bash
   sudo mkdir /mnt/orabackup
   ```

2. Настройка учетных данных:

   ```bash
   if [ ! -d "/etc/smbcredentials" ]; then
    sudo mkdir /etc/smbcredentials
   fi
   ```

   Замените `<Your Storage Account Key1>` ключ учетной записи хранения, полученный ранее, перед выполнением следующей команды:
   ```bash
   if [ ! -f "/etc/smbcredentials/orabackup1.cred" ]; then
     sudo bash -c 'echo "username=orabackup1" >> /etc/smbcredentials/orabackup1.cred'
     sudo bash -c 'echo "password=<Your Storage Account Key1>" >> /etc/smbcredentials/orabackup1.cred'
   fi
   ```

3. Измените разрешения для файла учетных данных:

   ```bash
   sudo chmod 600 /etc/smbcredentials/orabackup1.cred
   ```

4. Добавьте подключение к/etc/fstab:

   ```bash
   sudo bash -c 'echo "//orabackup1.file.core.windows.net/orabackup /mnt/orabackup cifs nofail,vers=3.0,credentials=/etc/smbcredentials/orabackup1.cred,dir_mode=0777,file_mode=0777,serverino,cache=none,uid=oracle,gid=oinstall" >> /etc/fstab'
   ```

5. Выполните команды, чтобы подключить хранилище файлов Azure с помощью протокола SMB.

   ```bash
   sudo mount -t cifs //orabackup1.file.core.windows.net/orabackup /mnt/orabackup -o vers=3.0,credentials=/etc/smbcredentials/orabackup1.cred,dir_mode=0777,file_mode=0777,serverino,cache=none,uid=oracle,gid=oinstall
   ```

   Если появляется сообщение об ошибке следующего вида:

   ```output
   mount: wrong fs type, bad option, bad superblock on //orabackup1.file.core.windows.net/orabackup 
   ```

   пакет CIFS может быть не установлен на узле Linux. 
   
   Чтобы проверить, установлены ли компоненты CIS, выполните следующую команду:

   ```bash
   sudo rpm -qa|grep cifs-utils
   ```

   Если команда не возвращает выходные данные, установите пакет CIFS с помощью следующей команды:

   ```bash 
   sudo yum install cifs-utils
   ```

   Теперь выполните команду подключения выше, чтобы подключить хранилище файлов Azure.

6. Убедитесь, что файловый ресурс подключен правильно, выполнив следующую команду:

   ```bash
   df -h
   ```

   Результат должен выглядеть следующим образом:

   ```output
   $ df -h
   Filesystem                                    Size  Used Avail Use% Mounted on
   devtmpfs                                      3.3G     0  3.3G   0% /dev
   tmpfs                                         3.3G     0  3.3G   0% /dev/shm
   tmpfs                                         3.3G   17M  3.3G   1% /run
   tmpfs                                         3.3G     0  3.3G   0% /sys/fs/cgroup
   /dev/sda2                                      30G  9.1G   19G  34% /
   /dev/sdc1                                      59G  2.7G   53G   5% /u02
   /dev/sda1                                     497M  199M  298M  41% /boot
   /dev/sda15                                    495M  9.7M  486M   2% /boot/efi
   tmpfs                                         671M     0  671M   0% /run/user/54321
   /dev/sdb1                                      14G  2.1G   11G  16% /mnt/resource
   tmpfs                                         671M     0  671M   0% /run/user/54322
   //orabackup1.file.core.windows.net/orabackup   10T     0   10T   0% /mnt/orabackup
   ```

### <a name="back-up-the-database"></a>Резервное копирование базы данных

В этом разделе мы будем использовать Диспетчер восстановления Oracle (RMAN) для создания полной резервной копии базы данных и архивирования журналов, а также для записи резервной копии в качестве резервного набора данных в общую папку Azure, подключенную ранее. 

1. Настройте RMAN для резервного копирования в точке подключения файлов Azure:

    ```bash
    $ rman target /
    RMAN> configure snapshot controlfile name to '/mnt/orabkup/snapcf_ev.f';
    RMAN> configure channel 1 device type disk format '/mnt/orabkup/%d/Full_%d_%U_%T_%s';
    RMAN> configure channel 2 device type disk format '/mnt/orabkup/%d/Full_%d_%U_%T_%s'; 
    ```

2. Так как стандартные файловые ресурсы Azure имеют максимальный размер файла, равный 1 Тиб, размер RMANных фрагментов будет ограничен 1 тиб. (Обратите внимание, что файловые ресурсы уровня "Премиум" имеют максимальный размер файла, равный 4 тиб. Дополнительные сведения см. в статье [целевые показатели масштабируемости и производительности службы файлов Azure](../../../storage/files/storage-files-scale-targets.md).

    ```bash
    RMAN> configure channel device type disk maxpiecesize 1000G;
    ```

3. Подтвердите сведения об изменении конфигурации:

    ```bash
    RMAN> show all;
    ```

4. Теперь запустите резервное копирование. Следующая команда выполнит полную резервную копию базы данных, включая архивные файлы журнала, в виде архива в сжатом формате:   

    ```bash
    RMAN> backup as compressed backupset database plus archivelog;
    ```

Вы создали резервную копию базы данных в сети с помощью Oracle RMAN, а также резервную копию, расположенную в хранилище файлов Azure. Этот метод имеет преимущество использования функций RMAN при хранении резервных копий в хранилище файлов Azure, где их можно получить из других виртуальных машин, если необходимо клонировать базу данных.  
    
Использование RMAN и хранилища файлов Azure для резервного копирования базы данных имеет множество преимуществ, время резервного копирования и восстановления связано с размером базы данных, поэтому для очень больших баз данных для этих операций может потребоваться значительное время.  Альтернативный механизм резервного копирования с использованием Azure Backup резервных копий виртуальных машин, совместимых с приложениями, использует технологию создания моментальных снимков для резервного копирования, что имеет преимущества очень быстрых резервных копий, независимо от размера базы данных. Интеграция с хранилищем служб восстановления обеспечивает безопасное хранение Oracle Database резервных копий в облачном хранилище Azure, доступных из других виртуальных машин и регионов Azure. 

### <a name="restore-and-recover-the-database"></a>Восстановить и восстановить базу данных

1.  Завершите работу экземпляра Oracle:

    ```bash
    sqlplus / as sysdba
    SQL> shutdown abort
    ORACLE instance shut down.
    ```

2.  Удалите файлы данных и резервные копии:

    ```bash
    cd /u02/oradata/TEST
    rm -f *.dbf
    ```

3. Следующие команды используют RMAN для восстановления отсутствующих файлов данных и восстановления базы.

    ```bash
    rman target /
    RMAN> startup mount;
    RMAN> restore database;
    RMAN> recover database;
    RMAN> alter database open;
    ```
    
4. Убедитесь, что содержимое базы данных восстановлено полностью:

    ```bash
    RMAN> SELECT * FROM scott.scott_table;
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