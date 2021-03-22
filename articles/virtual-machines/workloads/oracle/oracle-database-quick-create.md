---
title: Создание базы данных Oracle на виртуальной машине Azure | Документация Майкрософт
description: Быстрое создание и запуск базы данных Oracle 12c в среде Azure.
author: dbakevlar
ms.service: virtual-machines
ms.subservice: oracle
ms.collection: linux
ms.topic: quickstart
ms.date: 10/05/2020
ms.author: kegorman
ms.openlocfilehash: 8964248bb23b2b615c7e73e26d730fbd79b4e9e7
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102184463"
---
# <a name="create-an-oracle-database-in-an-azure-vm"></a>Создание базы данных Oracle на виртуальной машине Azure

В этом руководстве объясняется, как с помощью Azure CLI развернуть виртуальную машину Azure, используя [образ из коллекции Oracle Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/Oracle.OracleDatabase12102EnterpriseEdition?tab=Overview), чтобы создать базу данных Oracle 19c. После развертывания сервера вы подключитесь к нему по протоколу SSH для настройки базы данных Oracle. 

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

Если вы решили установить и использовать CLI локально, для выполнения инструкций в этом руководстве вам понадобится Azure CLI 2.0.4 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0]( /cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group). Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими. 

В приведенном ниже примере создается группа ресурсов с именем *rg-oracle* в расположении *eastus*.

```azurecli-interactive
az group create --name rg-oracle --location eastus
```

## <a name="create-virtual-machine"></a>Создание виртуальной машины

Чтобы создать виртуальную машину, выполните команду [az vm create](/cli/azure/vm). 

В следующем примере создается виртуальная машина `vmoracle19c`. Также создаются ключи SSH, если они не существуют в расположении ключей по умолчанию. Чтобы использовать определенный набор ключей, используйте параметр `--ssh-key-value`.  

```azurecli-interactive 
az vm create ^
    --resource-group rg-oracle ^
    --name vmoracle19c ^
    --image Oracle:oracle-database-19-3:oracle-database-19-0904:latest ^
    --size Standard_DS2_v2 ^
    --admin-username azureuser ^
    --generate-ssh-keys ^
    --public-ip-address-allocation static ^
    --public-ip-address-dns-name vmoracle19c

```

После создания виртуальной машины в Azure CLI отображается информация следующего вида. Обратите внимание на значение `publicIpAddress`. Этот адрес используется для доступа к виртуальной машине.

```output
{
  "fqdns": "",
  "id": "/subscriptions/{snip}/resourceGroups/rg-oracle/providers/Microsoft.Compute/virtualMachines/vmoracle19c",
  "location": "eastus",
  "macAddress": "00-0D-3A-36-2F-56",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "13.64.104.241",
  "resourceGroup": "rg-oracle"
}
```
## <a name="create-and-attach-a-new-disk-for-oracle-datafiles-and-fra"></a>Создание и подключение нового диска для файлов данных Oracle и FRA

```bash
az vm disk attach --name oradata01 --new --resource-group rg-oracle --size-gb 128 --sku StandardSSD_LRS --vm-name vmoracle19c
```

## <a name="open-ports-for-connectivity"></a>Открытие портов для подключения
При выполнении этой задачи нужно настроить для использования несколько внешних конечных точек для прослушивателя базы данных и EM Express путем настройки группы безопасности сети Azure, защищающей виртуальную машину. 

1. Чтобы открыть конечную точку для удаленного доступа к базе данных Oracle, создайте правило группы безопасности сети, как описано ниже:
   ```bash
   az network nsg rule create ^
       --resource-group rg-oracle ^
       --nsg-name vmoracle19cNSG ^
       --name allow-oracle ^
       --protocol tcp ^
       --priority 1001 ^
       --destination-port-range 1521
   ```
2. Чтобы открыть конечную точку для удаленного доступа к Oracle EM Express, создайте правило группы безопасности сети с помощью команды az network nsg rule create:
   ```bash
   az network nsg rule create ^
       --resource-group rg-oracle ^
       --nsg-name vmoracle19cNSG ^
       --name allow-oracle-EM ^
       --protocol tcp ^
       --priority 1002 ^
       --destination-port-range 5502
   ```
3. При необходимости получите общедоступный IP-адрес виртуальной машины еще раз с помощью команды az network public-ip show:

   ```bash
   az network public-ip show ^
       --resource-group rg-oracle ^
       --name vmoracle19cPublicIP ^
       --query [ipAddress] ^
       --output tsv
   ```

## <a name="prepare-the-vm-environment"></a>Подготовка среды виртуальных машин

1. Подключение к виртуальной машине

   Используйте следующую команду для создания сеанса SSH с виртуальной машиной. Замените IP-адрес общедоступным IP-адресом виртуальной машины (значение `publicIpAddress`).

   ```bash
   ssh azureuser@<publicIpAddress>
   ```

2. Переключитесь на роль привилегированного пользователя.

   ```bash
   sudo su -
   ```

3. Проверьте наличие последнего созданного дискового устройства. Мы отформатируем его для хранения файлов Oracle.

   ```bash
   ls -alt /dev/sd*|head -1
   ```

   Результат должен быть аналогичен следующему:
   ```output
   brw-rw----. 1 root disk 8, 16 Dec  8 22:57 /dev/sdc
   ```

4. Отформатируйте устройство. 
   Выполните на устройстве команду parted с правами привилегированного пользователя. 
   
   Сначала создайте метку диска:
   ```bash
   parted /dev/sdc mklabel gpt
   ```
   Затем создайте основной раздел, занимающий весь диск:
   ```bash
   parted -a optimal /dev/sdc mkpart primary 0GB 64GB   
   ```
   Наконец, просмотрите сведения об устройстве, выведя его метаданные:
   ```bash
   parted /dev/sdc print
   ```
   Результат должен выглядеть следующим образом:
   ```bash
   # parted /dev/sdc print
   Model: Msft Virtual Disk (scsi)
   Disk /dev/sdc: 68.7GB
   Sector size (logical/physical): 512B/4096B
   Partition Table: gpt
   Disk Flags:
   Number  Start   End     Size    File system  Name     Flags
    1      1049kB  64.0GB  64.0GB  ext4         primary
   ```

5. Создайте в разделе устройства файловую систему.

   ```bash
   mkfs -t ext4 /dev/sdc1
   ```

6. Создание точки подключения
   ```bash
   mkdir /u02
   ```

7. Подключение диска

   ```bash
   mount /dev/sdc1 /u02
   ```

8. Измените разрешения в точке подключения.

   ```bash
   chmod 777 /u02
   ```

9. Добавьте подключение к файлу /etc/fstab. 

   ```bash
   echo "/dev/sdc1               /u02                    ext4    defaults        0 0" >> /etc/fstab
   ```
   
10. Измените файл ***/etc/hosts***, указав общедоступный IP-адрес и имя узла.

    Измените ***общедоступный IP-адрес и имя виртуальной машины*** в соответствии с фактическими значениями:
  
    ```bash
    echo "<Public IP> <VMname>.eastus.cloudapp.azure.com <VMname>" >> /etc/hosts
    ```
11. Обновите файл имени узла.
    
    С помощью приведенной ниже команды добавьте доменное имя виртуальной машины в файл **/etc/hostname**. Предполагается, что вы создали группу ресурсов и виртуальную машину в регионе **eastus**:
    
    ```bash
    sed -i 's/$/\.eastus\.cloudapp\.azure\.com &/' /etc/hostname
    ```

12. Откройте порты брандмауэра.
    
    Так как в образе Marketplace по умолчанию включено средство SELinux, необходимо открыть брандмауэр, чтобы трафик поступал на порт 1521, ожидающий передачи данных из базы данных, и порт 5502 для Enterprise Manager Express. Выполните следующие команды с правами привилегированного пользователя:

    ```bash
    firewall-cmd --zone=public --add-port=1521/tcp --permanent
    firewall-cmd --zone=public --add-port=5502/tcp --permanent
    firewall-cmd --reload
    ```
   

## <a name="create-the-database"></a>Создание базы данных

Программное обеспечение Oracle уже установлено в образе Marketplace. Создайте пример базы данных, как описано ниже. 

1.  Переключитесь на роль пользователя **oracle**:

    ```bash
    sudo su - oracle
    ```
2. Запуск прослушивателя базы данных

   ```bash
   lsnrctl start
   ```
   Выход аналогичен приведенному ниже:
  
   ```output
   LSNRCTL for Linux: Version 19.0.0.0.0 - Production on 20-OCT-2020 01:58:18

   Copyright (c) 1991, 2019, Oracle.  All rights reserved.

   Starting /u01/app/oracle/product/19.0.0/dbhome_1/bin/tnslsnr: please wait...

   TNSLSNR for Linux: Version 19.0.0.0.0 - Production
   Log messages written to /u01/app/oracle/diag/tnslsnr/vmoracle19c/listener/alert/log.xml
   Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=vmoracle19c.eastus.cloudapp.azure.com)(PORT=1521)))

   Connecting to (ADDRESS=(PROTOCOL=tcp)(HOST=)(PORT=1521))
   STATUS of the LISTENER
   ------------------------
   Alias                     LISTENER
   Version                   TNSLSNR for Linux: Version 19.0.0.0.0 - Production
   Start Date                20-OCT-2020 01:58:18
   Uptime                    0 days 0 hr. 0 min. 0 sec
   Trace Level               off
   Security                  ON: Local OS Authentication
   SNMP                      OFF
   Listener Log File         /u01/app/oracle/diag/tnslsnr/vmoracle19c/listener/alert/log.xml
   Listening Endpoints Summary...
     (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=vmoracle19c.eastus.cloudapp.azure.com)(PORT=1521)))
   The listener supports no services
   The command completed successfully
   ```
3. Создайте каталог данных для файлов данных Oracle:

   ```bash
   mkdir /u02/oradata
   ```
    

3.  Запустите помощник по созданию базы данных:

    ```bash
    dbca -silent \
       -createDatabase \
       -templateName General_Purpose.dbc \
       -gdbname test \
       -sid test \
       -responseFile NO_VALUE \
       -characterSet AL32UTF8 \
       -sysPassword OraPasswd1 \
       -systemPassword OraPasswd1 \
       -createAsContainerDatabase false \
       -databaseType MULTIPURPOSE \
       -automaticMemoryManagement false \
       -storageType FS \
       -datafileDestination "/u02/oradata/" \
       -ignorePreReqs
    ```

    Создание базы данных занимает несколько минут.

    Выходные данные будут выглядеть примерно так:

    ```output
        Prepare for db operation
       10% complete
       Copying database files
       40% complete
       Creating and starting Oracle instance
       42% complete
       46% complete
       50% complete
       54% complete
       60% complete
       Completing Database Creation
       66% complete
       69% complete
       70% complete
       Executing Post Configuration Actions
       100% complete
       Database creation complete. For details check the logfiles at: /u01/app/oracle/cfgtoollogs/dbca/test.
       Database Information:
       Global Database Name:test
       System Identifier(SID):test
       Look at the log file "/u01/app/oracle/cfgtoollogs/dbca/test/test.log" for further details.
    ```

4. Задайте переменные Oracle.

    Перед подключением необходимо задать переменную среды *ORACLE_SID*:

    ```bash
        export ORACLE_SID=test
    ```

    Кроме того, необходимо добавить переменную ORACLE_SID в файл `.bashrc` пользователей `oracle` для последующих операций входа с помощью следующей команды:

    ```bash
    echo "export ORACLE_SID=test" >> ~oracle/.bashrc
    ```

## <a name="oracle-em-express-connectivity"></a>Подключение к Oracle EM Express

Мы включим в Oracle EM Express инструмент управления графическим интерфейсом пользователя, чтобы была возможность просматривать базу данных. Для подключения к Oracle EM Express сначала необходимо настроить порт в Oracle. 

1. Подключитесь к базе данных с помощью SQL Plus.

    ```bash
    sqlplus sys as sysdba
    ```

2. После подключения задайте порт 5502 для EM Express.

    ```bash
    exec DBMS_XDB_CONFIG.SETHTTPSPORT(5502);
    ```

3.  Подключите EM Express с помощью браузера. Убедитесь, что браузер совместим с EM Express (также необходимо установить Flash). 

    ```https
    https://<VM ip address or hostname>:5502/em
    ```

    Вы можете войти с помощью учетной записи **SYS** и установить флажок **as sysdba**. Используйте пароль **OraPasswd1**, заданный во время установки. 

    ![Снимок экрана со страницей входа Oracle OEM Express](./media/oracle-quick-start/oracle_oem_express_login.png)

## <a name="automate-database-startup-and-shutdown"></a>Автоматизация запуска и завершения работы базы данных

По умолчанию база данных Oracle не запускается автоматически при перезапуске виртуальной машины. Чтобы база данных Oracle запускалась автоматически, войдите с правами привилегированного пользователя. Затем создайте и обновите некоторые системные файлы.

1. Войдите с правами привилегированного пользователя.

    ```bash
    sudo su -
    ```

2.  Выполните следующую команду, чтобы изменить автоматический флаг запуска с `N` на `Y` в файле `/etc/oratab`:

    ```bash
    sed -i 's/:N/:Y/' /etc/oratab
    ```

3.  Создайте файл с именем `/etc/init.d/dbora` и вставьте в него следующее содержимое:

    ```bash
    #!/bin/sh
    # chkconfig: 345 99 10
    # Description: Oracle auto start-stop script.
    #
    # Set ORA_HOME to be equivalent to $ORACLE_HOME.
    ORA_HOME=/u01/app/oracle/product/19.0.0/dbhome_1
    ORA_OWNER=oracle

    case "$1" in
    'start')
        # Start the Oracle databases:
        # The following command assumes that the Oracle sign-in
        # will not prompt the user for any values.
        # Remove "&" if you don't want startup as a background process.
        su - $ORA_OWNER -c "$ORA_HOME/bin/dbstart $ORA_HOME" &
        touch /var/lock/subsys/dbora
        ;;

    'stop')
        # Stop the Oracle databases:
        # The following command assumes that the Oracle sign-in
        # will not prompt the user for any values.
        su - $ORA_OWNER -c "$ORA_HOME/bin/dbshut $ORA_HOME" &
        rm -f /var/lock/subsys/dbora
        ;;
    esac
    ```

4.  Измените разрешения для файлов с помощью команды *chmod*:

    ```bash
    chgrp dba /etc/init.d/dbora
    chmod 750 /etc/init.d/dbora
    ```

5.  Создайте символьные ссылки для запуска и завершения работы:

    ```bash
    ln -s /etc/init.d/dbora /etc/rc.d/rc0.d/K01dbora
    ln -s /etc/init.d/dbora /etc/rc.d/rc3.d/S99dbora
    ln -s /etc/init.d/dbora /etc/rc.d/rc5.d/S99dbora
    ```

6.  Чтобы проверить изменения, перезапустите виртуальную машину.

    ```bash
    reboot
    ```

## <a name="clean-up-resources"></a>Очистка ресурсов

Когда вы завершите изучение первой базы данных Oracle в Azure, вы можете удалить ненужную виртуальную машину, группу ресурсов и все связанные с ней ресурсы с помощью команды [az group delete](/cli/azure/group).

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как защитить базу данных в Azure с помощью [стратегий резервного копирования Oracle](./oracle-database-backup-strategies.md).

Узнайте о других [решениях Oracle в Azure](./oracle-overview.md). 

Ознакомьтесь с руководством по [установке и настройке Oracle ASM](configure-oracle-asm.md).
