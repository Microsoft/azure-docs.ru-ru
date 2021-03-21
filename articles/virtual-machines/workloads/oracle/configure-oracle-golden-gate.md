---
title: Реализация Oracle Golden Gate на виртуальной машине Azure под управлением Linux | Документация Майкрософт
description: Быстрое создание и запуск Oracle Golden Gate в среде Azure.
author: dbakevlar
ms.service: virtual-machines
ms.subservice: oracle
ms.collection: linux
ms.topic: article
ms.date: 08/02/2018
ms.author: kegorman
ms.openlocfilehash: fee6770108cbcc2334b2d8fb3ccc3b1e923772d9
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101673246"
---
# <a name="implement-oracle-golden-gate-on-an-azure-linux-vm"></a>Реализация Oracle Golden Gate на виртуальной машине Azure под управлением Linux 

Azure CLI используется для создания ресурсов Azure и управления ими из командной строки или с помощью скриптов. В этом руководстве описывается, как с помощью Azure CLI развернуть базу данных Oracle 12c, используя образ из коллекции Azure Marketplace. 

В этом документе демонстрируется пошаговое создание, установка и настройка Oracle Golden Gate на виртуальной машине Azure. В этом руководстве две виртуальные машины настраиваются в группе доступности в одном регионе. Этот же учебник можно использовать для настройки шлюза Ораклеголден для виртуальных машин в разных Зоны доступности в одном регионе Azure или для установки виртуальных машин в двух разных регионах.

Перед началом работы убедитесь, что вы установили Azure CLI. Дополнительные сведения см. в [руководстве по установке Azure CLI 2.0](/cli/azure/install-azure-cli).

## <a name="prepare-the-environment"></a>Подготовка среды

Чтобы выполнить установку Oracle Golden Gate, вам необходимо создать две виртуальные машины Azure в одной и той же группе доступности. Образ Marketplace, используемый для создания виртуальных машин, — **Oracle: Oracle-Database-ee: 12.1.0.2: Latest**.

Кроме того, необходимо уметь работать с редактором Unix и иметь базовое представление об x11 (X Windows).

Ниже приводится сводка конфигурации среды.
> 
> |  | **Первичный сайт** | **Сайт репликации** |
> | --- | --- | --- |
> | **Версия Oracle** |Версия 2 Oracle 12c — (12.1.0.2) |Версия 2 Oracle 12c — (12.1.0.2)|
> | **Имя компьютера** |myVM1 |myVM2 |
> | **Операционная система** |Oracle Linux 6.x |Oracle Linux 6.x |
> | **ИД безопасности Oracle** |CDB1 |CDB1 |
> | **Схема репликации** |TEST|TEST |
> | **Владелец/репликация Golden Gate** |C##GGADMIN |REPUSER |
> | **Процесс Golden Gate** |EXTORA |REPORA|


### <a name="sign-in-to-azure"></a>Вход в Azure 

Войдите в подписку Azure, используя команду [az login](/cli/azure/reference-index). Затем выполните инструкции на экране.

```azurecli
az login
```

### <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group). Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими. 

В следующем примере создается группа ресурсов с именем `myResourceGroup` в расположении именем `westus`.

```azurecli
az group create --name myResourceGroup --location westus
```

### <a name="create-an-availability-set"></a>"Создать группу доступности"

Следующий шаг необязателен, но мы рекомендуем его выполнить. Дополнительные сведения см. в статье [Рекомендации по группам доступности Azure для виртуальных машин Windows](/previous-versions/azure/virtual-machines/windows/infrastructure-example).

```azurecli
az vm availability-set create \
    --resource-group myResourceGroup \
    --name myAvailabilitySet \
    --platform-fault-domain-count 2 \
    --platform-update-domain-count 2
```

### <a name="create-a-virtual-machine"></a>Создание виртуальной машины

Создайте виртуальную машину с помощью команды [az vm create](/cli/azure/vm). 

В следующем примере создаются две виртуальные машины — `myVM1` и `myVM2`, а также ключи SSH, если они еще не существуют в расположении ключей по умолчанию. Чтобы использовать определенный набор ключей, используйте параметр `--ssh-key-value`.

#### <a name="create-myvm1-primary"></a>Создайте myVM1 (основная):

```azurecli
az vm create \
     --resource-group myResourceGroup \
     --name myVM1 \
     --availability-set myAvailabilitySet \
     --image Oracle:Oracle-Database-Ee:12.1.0.2:latest \
     --size Standard_DS1_v2  \
     --generate-ssh-keys \
```

После создания виртуальной машины в Azure CLI отображается информация следующего вида. (Запишите значение `publicIpAddress`. Этот адрес используется для доступа к виртуальной машине.)

```output
{
  "fqdns": "",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "westus",
  "macAddress": "00-0D-3A-36-2F-56",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "13.64.104.241",
  "resourceGroup": "myResourceGroup"
}
```

#### <a name="create-myvm2-replicate"></a>Создайте myVM2 (репликация):

```azurecli
az vm create \
     --resource-group myResourceGroup \
     --name myVM2 \
     --availability-set myAvailabilitySet \
     --image Oracle:Oracle-Database-Ee:12.1.0.2:latest \
     --size Standard_DS1_v2  \
     --generate-ssh-keys \
```

Запишите `publicIpAddress` после его создания.

### <a name="open-the-tcp-port-for-connectivity"></a>Открытие TCP-порта для возможности подключения

Следующий шаг — настройка внешних конечных точек, после чего вы сможете получить удаленный доступ к базе данных Oracle. Чтобы настроить внешние конечные точки, выполните следующие команды.

#### <a name="open-the-port-for-myvm1"></a>Откройте порт для myVM1:

```azurecli
az network nsg rule create --resource-group myResourceGroup\
    --nsg-name myVm1NSG --name allow-oracle\
    --protocol tcp --direction inbound --priority 999 \
    --source-address-prefix '*' --source-port-range '*' \
    --destination-address-prefix '*' --destination-port-range 1521 --access allow
```

Результат должен выглядеть следующим образом:

```output
{
  "access": "Allow",
  "description": null,
  "destinationAddressPrefix": "*",
  "destinationPortRange": "1521",
  "direction": "Inbound",
  "etag": "W/\"bd77dcae-e5fd-4bd6-a632-26045b646414\"",
  "id": "/subscriptions/<subscription-id>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myVmNSG/securityRules/allow-oracle",
  "name": "allow-oracle",
  "priority": 999,
  "protocol": "Tcp",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "sourceAddressPrefix": "*",
  "sourcePortRange": "*"
}
```

#### <a name="open-the-port-for-myvm2"></a>Откройте порт для myVM2:

```azurecli
az network nsg rule create --resource-group myResourceGroup\
    --nsg-name myVm2NSG --name allow-oracle\
    --protocol tcp --direction inbound --priority 999 \
    --source-address-prefix '*' --source-port-range '*' \
    --destination-address-prefix '*' --destination-port-range 1521 --access allow
```

### <a name="connect-to-the-virtual-machine"></a>Подключение к виртуальной машине

Используйте следующую команду для создания сеанса SSH с виртуальной машиной. Замените IP-адрес общедоступным IP-адресом виртуальной машины (значение `publicIpAddress`).

```bash
ssh <publicIpAddress>
```

### <a name="create-the-database-on-myvm1-primary"></a>Создание базы данных на myVM1 (основная)

Программное обеспечение Oracle уже установлено в образе Marketplace, поэтому следующим шагом является установка базы данных. 

Запустите программное обеспечение с правами суперпользователя oracle:

```bash
sudo su - oracle
```

Создание базы данных

```bash
$ dbca -silent \
   -createDatabase \
   -templateName General_Purpose.dbc \
   -gdbname cdb1 \
   -sid cdb1 \
   -responseFile NO_VALUE \
   -characterSet AL32UTF8 \
   -sysPassword OraPasswd1 \
   -systemPassword OraPasswd1 \
   -createAsContainerDatabase true \
   -numberOfPDBs 1 \
   -pdbName pdb1 \
   -pdbAdminPassword OraPasswd1 \
   -databaseType MULTIPURPOSE \
   -automaticMemoryManagement false \
   -storageType FS \
   -ignorePreReqs
```

Результат должен выглядеть следующим образом:

```output
Copying database files
1% complete
2% complete
8% complete
13% complete
19% complete
27% complete
Creating and starting Oracle instance
29% complete
32% complete
33% complete
34% complete
38% complete
42% complete
43% complete
45% complete
Completing Database Creation
48% complete
51% complete
53% complete
62% complete
70% complete
72% complete
Creating Pluggable Databases
78% complete
100% complete
Look at the log file "/u01/app/oracle/cfgtoollogs/dbca/cdb1/cdb1.log" for more details.
```

Задайте переменные ORACLE_SID и ORACLE_HOME.

```bash
$ ORACLE_HOME=/u01/app/oracle/product/12.1.0/dbhome_1; export ORACLE_HOME
$ ORACLE_SID=cdb1; export ORACLE_SID
$ LD_LIBRARY_PATH=ORACLE_HOME/lib; export LD_LIBRARY_PATH
```

При необходимости можно добавить ORACLE_HOME и ORACLE_SID в файл BASHRC, чтобы эти параметры сохранились для последующих входов в систему:

```bash
# add oracle home
export ORACLE_HOME=/u01/app/oracle/product/12.1.0/dbhome_1
# add oracle sid
export ORACLE_SID=cdb1
# add Oracle library path
export LD_LIBRARY_PATH=$ORACLE_HOME/lib
```

### <a name="start-oracle-listener"></a>Запуск прослушивателя Oracle

```bash
$ lsnrctl start
```

### <a name="create-the-database-on-myvm2-replicate"></a>Создание базы данных на myVM2 (репликация)

```bash
sudo su - oracle
```

Создание базы данных

```bash
$ dbca -silent \
   -createDatabase \
   -templateName General_Purpose.dbc \
   -gdbname cdb1 \
   -sid cdb1 \
   -responseFile NO_VALUE \
   -characterSet AL32UTF8 \
   -sysPassword OraPasswd1 \
   -systemPassword OraPasswd1 \
   -createAsContainerDatabase true \
   -numberOfPDBs 1 \
   -pdbName pdb1 \
   -pdbAdminPassword OraPasswd1 \
   -databaseType MULTIPURPOSE \
   -automaticMemoryManagement false \
   -storageType FS \
   -ignorePreReqs
```

Задайте переменные ORACLE_SID и ORACLE_HOME.

```bash
$ ORACLE_HOME=/u01/app/oracle/product/12.1.0/dbhome_1; export ORACLE_HOME
$ ORACLE_SID=cdb1; export ORACLE_SID
$ LD_LIBRARY_PATH=ORACLE_HOME/lib; export LD_LIBRARY_PATH
```

При необходимости можно добавить ORACLE_HOME и ORACLE_SID в файл BASHRC, чтобы эти параметры сохранились для последующих входов в систему.

```bash
# add oracle home
export ORACLE_HOME=/u01/app/oracle/product/12.1.0/dbhome_1
# add oracle sid
export ORACLE_SID=cdb1
# add Oracle library path
export LD_LIBRARY_PATH=$ORACLE_HOME/lib
```

### <a name="start-oracle-listener"></a>Запуск прослушивателя Oracle

```bash
$ sudo su - oracle
$ lsnrctl start
```

## <a name="configure-golden-gate"></a>Настройка Golden Gate 
Чтобы настроить Golden Gate, выполните действия в этом разделе.

### <a name="enable-archive-log-mode-on-myvm1-primary"></a>Включение режима журнала архивирования на myVM1 (основная)

```bash
$ sqlplus / as sysdba
SQL> SELECT log_mode FROM v$database;

LOG_MODE
------------
NOARCHIVELOG

SQL> SHUTDOWN IMMEDIATE;
SQL> STARTUP MOUNT;
SQL> ALTER DATABASE ARCHIVELOG;
SQL> ALTER DATABASE OPEN;
```
Включите принудительное ведение журнала и убедитесь, что есть хотя бы один файл журнала.

```bash
SQL> ALTER DATABASE FORCE LOGGING;
SQL> ALTER SYSTEM SWITCH LOGFILE;
SQL> ALTER SYSTEM set enable_goldengate_replication=true;
SQL> ALTER PLUGGABLE DATABASE PDB1 OPEN;
SQL> ALTER SESSION SET CONTAINER=PDB1;
SQL> ALTER DATABASE ADD SUPPLEMENTAL LOG DATA;
SQL> EXIT;
```

### <a name="download-golden-gate-software"></a>Загрузка программного обеспечения Golden Gate
Чтобы загрузить и подготовить программное обеспечение Oracle Golden Gate, сделайте следующее:

1. Загрузите файл **fbo_ggs_Linux_x64_shiphome.zip** из [страницы скачивания Oracle Golden Gate](https://www.oracle.com/technetwork/middleware/goldengate/downloads/index.html). Под заголовком загрузки **12.x.x.x Oracle GoldenGate для Oracle Linux x86-64** должен быть набор ZIP-файлов, которые нужно загрузить.

2. Загрузив ZIP-файлы на клиентском компьютере, скопируйте файлы на виртуальную машину по протоколу SCP.

   ```bash
   $ scp fbo_ggs_Linux_x64_shiphome.zip <publicIpAddress>:<folder>
   ```

3. Переместите ZIP-файлы в папку **/OPT** . Затем измените владельца файлов следующим образом:

   ```bash
   $ sudo su -
   # mv <folder>/*.zip /opt
   ```

4. Распакуйте файлы (установите служебную программу Linux для распаковки, если она еще не установлена):

   ```bash
   # yum install unzip
   # cd /opt
   # unzip fbo_ggs_Linux_x64_shiphome.zip
   ```

5. Измените разрешение:

   ```bash
   # chown -R oracle:oinstall /opt/fbo_ggs_Linux_x64_shiphome
   ```

### <a name="prepare-the-client-and-vm-to-run-x11-for-windows-clients-only"></a>Подготовка клиента и виртуальной машины для запуска X11 (только для клиентов Windows)
Этот параметр является необязательным. Если вы используете клиента Linux или уже установили x11, этот шаг можно пропустить.

1. Скачайте PuTTY и Xming на компьютер с Windows:

   * [Скачать PuTTY](https://www.putty.org/).
   * [Скачать Xming](https://xming.en.softonic.com/).

2. Установив PuTTY в папке PuTTY (например, C:\Program Files\PuTTY), запустите файл puttygen.exe (генератор ключей PuTTY).

3. В генераторе ключей PuTTY сделайте следующее:

   - Чтобы создать ключ, нажмите кнопку **Создать**.
   - Скопируйте содержимое ключа (**CTRL+C**).
   - Нажмите кнопку **Save private key** (Сохранить закрытый ключ).
   - Игнорируйте предупреждение, появившееся на экране, и нажмите кнопку **ОК**.

   ![Снимок экрана со страницей генератора ключей PuTTY](./media/oracle-golden-gate/puttykeygen.png)

4. В виртуальной машине выполните следующие команды:

   ```bash
   # sudo su - oracle
   $ mkdir .ssh (if not already created)
   $ cd .ssh
   ```

5. Создайте файл с именем **authorized_keys**. Вставьте содержимое ключа в этот файл и сохраните файл.

   > [!NOTE]
   > Ключ должен содержать строку `ssh-rsa`. Кроме того, содержимое ключа должно быть одной строкой текста.
   >  

6. Запустите PuTTY. В области **Категория** выберите **Подключение**  >  **SSH**  >  **AUTH**. В поле **файл закрытого ключа для проверки подлинности** перейдите к разделу, созданному ранее.

   ![Снимок экрана со страницей настройки закрытого ключа](./media/oracle-golden-gate/setprivatekey.png)

7. В области **Категория** выберите **Подключение** > **SSH** > **X11**. Установите флажок **Enable X11 forwarding** (Включить перенаправление X11).

   ![Снимок экрана со страницей включения X11](./media/oracle-golden-gate/enablex11.png)

8. В области **Категория** выберите **Сеанс**. Введите сведения об узле, а затем нажмите кнопку **Открыть**.

   ![Снимок экрана со страницей "Сеанс"](./media/oracle-golden-gate/puttysession.png)

### <a name="install-golden-gate-software"></a>Установка программного обеспечения Golden Gate

Чтобы установить Oracle Golden Gate, сделайте следующее:

1. Выполните вход в качестве пользователя oracle. (Вы можете выполнить вход без запроса пароля.) Перед началом установки убедитесь, что Xming выполняется.

   ```bash
   $ cd /opt/fbo_ggs_Linux_x64_shiphome/Disk1
   $ ./runInstaller
   ```

2. Выберите Oracle GoldenGate for Oracle Database 12c (Oracle GoldenGate для базы данных Oracle 12c). Затем нажмите кнопку **Далее** , чтобы продолжить.

   ![Снимок экрана со страницей выбора установки в установщике](./media/oracle-golden-gate/golden_gate_install_01.png)

3. Измените расположение программного обеспечения. Установите флажок **Start Manager** (Запустить диспетчер) и введите расположение базы данных. Нажмите кнопку **Далее**, чтобы продолжить.

   ![Снимок экрана со страницей выбора установки](./media/oracle-golden-gate/golden_gate_install_02.png)

4. Перейдите в каталог инвентаризации, а затем выберите **Далее**, чтобы продолжить.

   ![Снимок экрана: страница "Выбор установки", на которой показан каталог установки.](./media/oracle-golden-gate/golden_gate_install_03.png)

5. На экране **Сводка** выберите **Установить**, чтобы продолжить.

   ![Снимок экрана, на котором показаны страница "Выбор установки" и кнопка "установить".](./media/oracle-golden-gate/golden_gate_install_04.png)

6. Может потребоваться выполнить скрипт в качестве корневого. В этом случае откройте отдельный сеанс, подключитесь к виртуальной машине по протоколу SSH, используйте sudo, чтобы сделать скрипт корневым, а затем выполните его. Чтобы продолжить, нажмите кнопку **ОК**.

   ![Снимок экрана, на котором показано расположение скрипта и выполнение скрипта настройки.](./media/oracle-golden-gate/golden_gate_install_05.png)

7. По завершении установки нажмите кнопку **Закрыть**, чтобы завершить процесс.

   ![Снимок экрана: страница "Выбор установки", на которой отображается кнопка "Закрыть".](./media/oracle-golden-gate/golden_gate_install_06.png)

### <a name="set-up-service-on-myvm1-primary"></a>Настройка службы на myVM1 (основная)

1. Создайте или обновите файл tnsnames.ora:

   ```bash
   $ cd $ORACLE_HOME/network/admin
   $ vi tnsnames.ora

   cdb1=
    (DESCRIPTION=
      (ADDRESS=
        (PROTOCOL=TCP)
        (HOST=localhost)
        (PORT=1521)
      )
      (CONNECT_DATA=
        (SERVER=dedicated)
        (SERVICE_NAME=cdb1)
      )
    )

   pdb1=
    (DESCRIPTION=
      (ADDRESS=
        (PROTOCOL=TCP)
        (HOST=localhost)
        (PORT=1521)
      )
      (CONNECT_DATA=
        (SERVER=dedicated)
        (SERVICE_NAME=pdb1)
      )
    )
   ```

2. Создайте учетные записи владельца и пользователя Golden Gate.

   > [!NOTE]
   > Учетная запись владельца должна иметь префикс C##.
   >

    ```bash
    $ sqlplus / as sysdba
    SQL> CREATE USER C##GGADMIN identified by ggadmin;
    SQL> EXEC dbms_goldengate_auth.grant_admin_privilege('C##GGADMIN',container=>'ALL');
    SQL> GRANT DBA to C##GGADMIN container=all;
    SQL> connect C##GGADMIN/ggadmin
    SQL> ALTER SESSION SET CONTAINER=PDB1;
    SQL> EXIT;
    ```

3. Создайте тестовую учетную запись пользователя Golden Gate:

   ```bash
   $ cd /u01/app/oracle/product/12.1.0/oggcore_1
   $ sqlplus system/OraPasswd1@pdb1
   SQL> CREATE USER test identified by test DEFAULT TABLESPACE USERS TEMPORARY TABLESPACE TEMP;
   SQL> GRANT connect, resource, dba TO test;
   SQL> ALTER USER test QUOTA 100M on USERS;
   SQL> connect test/test@pdb1
   SQL> @demo_ora_create
   SQL> @demo_ora_insert
   SQL> EXIT;
   ```

4. Настройте файл параметров извлечения.

   Запустите интерфейс командной строки Golden Gate (ggsci):

   ```bash
   $ sudo su - oracle
   $ cd /u01/app/oracle/product/12.1.0/oggcore_1
   $ ./ggsci
   GGSCI> DBLOGIN USERID test@pdb1, PASSWORD test
   Successfully logged into database  pdb1
   GGSCI>  ADD SCHEMATRANDATA pdb1.test
   2017-05-23 15:44:25  INFO    OGG-01788  SCHEMATRANDATA has been added on schema test.
   2017-05-23 15:44:25  INFO    OGG-01976  SCHEMATRANDATA for scheduling columns has been added on schema test.

   GGSCI> EDIT PARAMS EXTORA
   ```

5. Добавьте следующий файл параметров EXTRACT (с помощью команды vi). Нажмите клавишу ESC, ":wq!" чтобы сохранить файл. 

   ```bash
   EXTRACT EXTORA
   USERID C##GGADMIN, PASSWORD ggadmin
   RMTHOST 10.0.0.5, MGRPORT 7809
   RMTTRAIL ./dirdat/rt  
   DDL INCLUDE MAPPED
   DDLOPTIONS REPORT 
   LOGALLSUPCOLS
   UPDATERECORDFORMAT COMPACT
   TABLE pdb1.test.TCUSTMER;
   TABLE pdb1.test.TCUSTORD;
   ```

6. Зарегистрируйте извлечение, интегрированное с помощью EXTRACT:

   ```bash
   $ cd /u01/app/oracle/product/12.1.0/oggcore_1
   $ ./ggsci

   GGSCI> dblogin userid C##GGADMIN, password ggadmin
   Successfully logged into database CDB$ROOT.

   GGSCI> REGISTER EXTRACT EXTORA DATABASE CONTAINER(pdb1)

   2017-05-23 15:58:34  INFO    OGG-02003  Extract EXTORA successfully registered with database at SCN 1821260.

   GGSCI> exit
   ```

7. Настройте контрольные точки извлечения и запустите извлечение в режиме реального времени:

   ```bash
   $ ./ggsci
   GGSCI>  ADD EXTRACT EXTORA, INTEGRATED TRANLOG, BEGIN NOW
   EXTRACT (Integrated) added.

   GGSCI>  ADD RMTTRAIL ./dirdat/rt, EXTRACT EXTORA, MEGABYTES 10
   RMTTRAIL added.

   GGSCI>  START EXTRACT EXTORA

   Sending START request to MANAGER ...
   EXTRACT EXTORA starting

   GGSCI > info all

   Program     Status      Group       Lag at Chkpt  Time Since Chkpt

   MANAGER     RUNNING
   EXTRACT     RUNNING     EXTORA      00:00:11      00:00:04
   ```

   На этом шаге найдите начальный сайт SCN, который будет использоваться позже в другом разделе:

   ```bash
   $ sqlplus / as sysdba
   SQL> alter session set container = pdb1;
   SQL> SELECT current_scn from v$database;
   CURRENT_SCN
   -----------
      1857887
   SQL> EXIT;
   ```

   ```bash
   $ ./ggsci
   GGSCI> EDIT PARAMS INITEXT
   ```

   ```bash
   EXTRACT INITEXT
   USERID C##GGADMIN, PASSWORD ggadmin
   RMTHOST 10.0.0.5, MGRPORT 7809
   RMTTASK REPLICAT, GROUP INITREP
   TABLE pdb1.test.*, SQLPREDICATE 'AS OF SCN 1857887'; 
   ```

   ```bash
   GGSCI> ADD EXTRACT INITEXT, SOURCEISTABLE
   ```

### <a name="set-up-service-on-myvm2-replicate"></a>Настройка службы на myVM2 (репликация)


1. Создайте или обновите файл tnsnames.ora:

   ```bash
   $ cd $ORACLE_HOME/network/admin
   $ vi tnsnames.ora

   cdb1=
    (DESCRIPTION=
      (ADDRESS=
        (PROTOCOL=TCP)
        (HOST=localhost)
        (PORT=1521)
      )
      (CONNECT_DATA=
        (SERVER=dedicated)
        (SERVICE_NAME=cdb1)
      )
    )

   pdb1=
    (DESCRIPTION=
      (ADDRESS=
        (PROTOCOL=TCP)
        (HOST=localhost)
        (PORT=1521)
      )
      (CONNECT_DATA=
        (SERVER=dedicated)
        (SERVICE_NAME=pdb1)
      )
    )
   ```

2. Создайте учетную запись репликации:

   ```bash
   $ sqlplus / as sysdba
   SQL> alter session set container = pdb1;
   SQL> create user repuser identified by rep_pass container=current;
   SQL> grant dba to repuser;
   SQL> exec dbms_goldengate_auth.grant_admin_privilege('REPUSER',container=>'PDB1');
   SQL> connect repuser/rep_pass@pdb1 
   SQL> EXIT;
   ```

3. Создайте тестовую учетную запись пользователя Golden Gate:

   ```bash
   $ cd /u01/app/oracle/product/12.1.0/oggcore_1
   $ sqlplus system/OraPasswd1@pdb1
   SQL> CREATE USER test identified by test DEFAULT TABLESPACE USERS TEMPORARY TABLESPACE TEMP;
   SQL> GRANT connect, resource, dba TO test;
   SQL> ALTER USER test QUOTA 100M on USERS;
   SQL> connect test/test@pdb1
   SQL> @demo_ora_create
   SQL> EXIT;
   ```

4. Файл параметров REPLICAT для репликации изменений: 

   ```bash
   $ cd /u01/app/oracle/product/12.1.0/oggcore_1
   $ ./ggsci
   GGSCI> EDIT PARAMS REPORA  
   ```

   Содержимое файла параметров REPORA:

   ```bash
   REPLICAT REPORA
   ASSUMETARGETDEFS
   DISCARDFILE ./dirrpt/repora.dsc, PURGE, MEGABYTES 100
   DDL INCLUDE MAPPED
   DDLOPTIONS REPORT
   DBOPTIONS INTEGRATEDPARAMS(parallelism 6)
   USERID repuser@pdb1, PASSWORD rep_pass
   MAP pdb1.test.*, TARGET pdb1.test.*;
   ```

5. Настройка репликации контрольной точки:

   ```bash
   GGSCI> ADD REPLICAT REPORA, INTEGRATED, EXTTRAIL ./dirdat/rt
   GGSCI> EDIT PARAMS INITREP

   ```

   ```bash
   REPLICAT INITREP
   ASSUMETARGETDEFS
   DISCARDFILE ./dirrpt/tcustmer.dsc, APPEND
   USERID repuser@pdb1, PASSWORD rep_pass
   MAP pdb1.test.*, TARGET pdb1.test.*;   
   ```

   ```bash
   GGSCI> ADD REPLICAT INITREP, SPECIALRUN
   ```

### <a name="set-up-the-replication-myvm1-and-myvm2"></a>Настройка репликации (myVM1 и myVM2)

#### <a name="1-set-up-the-replication-on-myvm2-replicate"></a>1. Настройка репликации в myVM2 (репликация)

  ```bash
  $ cd /u01/app/oracle/product/12.1.0/oggcore_1
  $ ./ggsci
  GGSCI> EDIT PARAMS MGR
  ```

Обновите файл с использованием следующего содержимого:

  ```bash
  PORT 7809
  ACCESSRULE, PROG *, IPADDR *, ALLOW
  ```

Затем перезапустите службу диспетчера:

  ```bash
  GGSCI> STOP MGR
  GGSCI> START MGR
  GGSCI> EXIT
  ```

#### <a name="2-set-up-the-replication-on-myvm1-primary"></a>2. Настройка репликации в myVM1 (основная)

Запустите начальную нагрузку и проверьте наличие ошибок:

```bash
$ cd /u01/app/oracle/product/12.1.0/oggcore_1
$ ./ggsci
GGSCI> START EXTRACT INITEXT
GGSCI> VIEW REPORT INITEXT
```

#### <a name="3-set-up-the-replication-on-myvm2-replicate"></a>3. Настройка репликации в myVM2 (репликация)

Измените номер SCN на полученный ранее номер:

  ```bash
  $ cd /u01/app/oracle/product/12.1.0/oggcore_1
  $ ./ggsci
  START REPLICAT REPORA, AFTERCSN 1857887
  ```

Репликация началась и вы можете протестировать ее, вставив новые записи в таблицы TEST.


### <a name="view-job-status-and-troubleshooting"></a>Просмотр состояния задания и устранение неполадок

#### <a name="view-reports"></a>Просмотр отчетов
Чтобы просмотреть отчеты на myVM1, выполните следующие команды:

  ```bash
  GGSCI> VIEW REPORT EXTORA 
  ```
 
Чтобы просмотреть отчеты на myVM2, выполните следующие команды:

  ```bash
  GGSCI> VIEW REPORT REPORA
  ```

#### <a name="view-status-and-history"></a>Просмотр сведений о состоянии и журналов
Чтобы просмотреть сведения о состоянии и журналы на myVM1, выполните следующие команды:

  ```bash
  GGSCI> dblogin userid c##ggadmin, password ggadmin 
  GGSCI> INFO EXTRACT EXTORA, DETAIL
  ```

Чтобы просмотреть сведения о состоянии и журналы на myVM2, выполните следующие команды:

  ```bash
  GGSCI> dblogin userid repuser@pdb1 password rep_pass 
  GGSCI> INFO REP REPORA, DETAIL
  ```
Теперь вы завершили установку и настройку Golden Gate на Oracle Linux.


## <a name="delete-the-virtual-machine"></a>Удаление виртуальной машины

Вы можете удалить ставшие ненужными группу ресурсов, виртуальную машину и все связанные с ней ресурсы, использовав следующую команду.

```azurecli
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

Изучите [руководство по созданию высокодоступных виртуальных машин](../../linux/create-cli-complete.md).

[Изучите примеры развертывания виртуальных машин с помощью интерфейса командной строки](https://github.com/Azure-Samples/azure-cli-samples/tree/master/virtual-machine).