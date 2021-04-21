---
title: Настройка Oracle ASM в виртуальной машине Linux в Azure | Документация Майкрософт
description: Быстрая подготовка и запуск Oracle ASM в среде Azure.
author: dbakevlar
ms.service: virtual-machines
ms.subservice: oracle
ms.collection: linux
ms.topic: article
ms.date: 08/02/2018
ms.author: kegorman
ms.openlocfilehash: b180a0f148db5222a288958724ad248acd18b565
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101673856"
---
# <a name="set-up-oracle-asm-on-an-azure-linux-virtual-machine"></a>Настройка Oracle ASM в виртуальной машине Linux в Azure  

Виртуальные машины Azure предоставляют полностью настраиваемую и гибкую вычислительную среду. В этом руководстве описано развертывание базовой виртуальной машины Azure, а также установка и настройка Oracle ASM.  Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Создание виртуальной машины базы данных Oracle и подключение к ней.
> * Установка и настройка Oracle ASM.
> * Установка и настройка Oracle Grid Infrastructure.
> * Инициализация установки Oracle ASM.
> * Создание базы данных Oracle под управлением ASM.


Если вы решили установить и использовать интерфейс командной строки локально, для работы с этим руководством вам понадобится Azure CLI 2.0.4 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0]( /cli/azure/install-azure-cli). 

## <a name="prepare-the-environment"></a>Подготовка среды

### <a name="create-a-resource-group"></a>Создание группы ресурсов

Чтобы создать группу ресурсов, используйте команду [az group create](/cli/azure/group). Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими. В этом примере создается группа ресурсов с именем *myResourceGroup* в регионе *eastus*.

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

### <a name="create-a-vm"></a>Создание виртуальной машины

Чтобы создать виртуальную машину на основе образа базы данных Oracle и настроить ее для использования Oracle ASM, выполните команду [az vm create](/cli/azure/vm). 

В следующем примере создается виртуальная машина с именем myVM размера Standard_DS2_v2, к которой подключено четыре диска данных по 50 ГБ каждый. Кроме того, создаются ключи SSH, если они не существуют в расположении ключей по умолчанию.  Чтобы использовать определенный набор ключей, используйте параметр `--ssh-key-value`.  

   ```azurecli-interactive
   az vm create --resource-group myResourceGroup \
    --name myVM \
    --image Oracle:Oracle-Database-Ee:12.1.0.2:latest \
    --size Standard_DS2_v2 \
    --generate-ssh-keys \
    --data-disk-sizes-gb 50 50 50 50
   ```

После создания виртуальной машины в Azure CLI отображается информация следующего вида. Обратите внимание на значение `publicIpAddress`. Этот адрес используется для доступа к виртуальной машине.

   ```output
   {
     "fqdns": "",
     "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
     "location": "eastus",
     "macAddress": "00-0D-3A-36-2F-56",
     "powerState": "VM running",
     "privateIpAddress": "10.0.0.4",
     "publicIpAddress": "13.64.104.241",
     "resourceGroup": "myResourceGroup"
   }
   ```

### <a name="connect-to-the-vm"></a>Подключение к виртуальной машине

Чтобы создать сеанс SSH с виртуальной машиной и настроить дополнительные параметры, используйте следующую команду. Замените IP-адрес общедоступным IP-адресом виртуальной машины (значение `publicIpAddress`).

```bash
ssh <publicIpAddress>
```

## <a name="install-oracle-asm"></a>Установка Oracle ASM

Чтобы установить ASM, сделайте следующее: 

Дополнительные сведения об установке Oracle ASM см. в статье [Oracle ASMLib Downloads for Oracle Linux 6](https://www.oracle.com/technetwork/server-storage/linux/asmlib/ol6-1709075.html) (Скачиваемые компоненты Oracle ASMLib для Oracle Linux 6).  

1. Необходимо войти в систему как привилегированный пользователь, чтобы продолжить установку ASM:

   ```bash
   sudo su -
   ```
   
2. Выполните эти дополнительные команды для установки компонентов Oracle ASM:

   ```bash
    yum list | grep oracleasm 
    yum -y install kmod-oracleasm.x86_64 
    yum -y install oracleasm-support.x86_64 
    wget https://download.oracle.com/otn_software/asmlib/oracleasmlib-2.0.12-1.el6.x86_64.rpm 
    yum -y install oracleasmlib-2.0.12-1.el6.x86_64.rpm 
    rm -f oracleasmlib-2.0.12-1.el6.x86_64.rpm
   ```

3. Убедитесь, что служба Oracle ASM установлена:

   ```bash
   rpm -qa |grep oracleasm
   ```

    Выходные данные этой команды должны содержать следующие компоненты:

    ```bash
   oracleasm-support-2.1.10-4.el6.x86_64
   kmod-oracleasm-2.0.8-15.el6_9.x86_64
   oracleasmlib-2.0.12-1.el6.x86_64
    ```

4. Для правильной работы ASM требуется конкретный пользователь и роли. Следующие команды создают необходимые учетные записи пользователей и группы: 

   ```bash
    groupadd -g 54345 asmadmin 
    groupadd -g 54346 asmdba 
    groupadd -g 54347 asmoper 
    useradd -u 3000 -g oinstall -G dba,asmadmin,asmdba,asmoper grid 
    usermod -g oinstall -G dba,asmdba,asmadmin oracle
   ```

5. Убедитесь, что пользователи и группы созданы правильно:

   ```bash
   id grid
   ```

    Выходные данные этой команды должны содержать следующих пользователей и группы:

    ```bash
    uid=3000(grid) gid=54321(oinstall) groups=54321(oinstall),54322(dba),54345(asmadmin),54346(asmdba),54347(asmoper)
    ```
 
6. Создайте папку для пользователя *grid* и смените владельца:

   ```bash
   mkdir /u01/app/grid 
   chown grid:oinstall /u01/app/grid
   ```

## <a name="set-up-oracle-asm"></a>Настройка Oracle ASM

В этом руководстве пользователем по умолчанию является *grid*, а группой по умолчанию — *asmadmin*. Убедитесь, что пользователь *oracle* входит в группу asmadmin. Чтобы настроить установку Oracle ASM, сделайте следующее:

1. При настройке драйвера библиотеки Oracle ASM нужно определить пользователя (grid) и группу по умолчанию (asmadmin), настроить запуск и сканирование дисков во время запуска (выберите y). Необходимо указать сведения в ответ на запросы, используя следующую команду:

   ```bash
   /usr/sbin/oracleasm configure -i
   ```

   Выходные данные этой команды должны выглядеть следующим образом (указывать сведения в ответ на запросы больше не потребуется).

    ```output
   Configuring the Oracle ASM library driver.

   This will configure the on-boot properties of the Oracle ASM library
   driver. The following questions will determine whether the driver is
   loaded on boot and what permissions it will have. The current values
   will be shown in brackets ('[]'). Hitting <ENTER> without typing an
   answer will keep that current value. Ctrl-C will abort.

   Default user to own the driver interface []: grid
   Default group to own the driver interface []: asmadmin
   Start Oracle ASM library driver on boot (y/n) [n]: y
   Scan for Oracle ASM disks on boot (y/n) [y]: y
   Writing Oracle ASM library driver configuration: done
   ```

2. Просмотрите конфигурацию диска:

   ```bash
   cat /proc/partitions
   ```

   Результат этой команды должен выглядеть примерно как перечисление доступных дисков:

   ```output
   8       16   14680064 sdb
   8       17   14678976 sdb1
   8        0   52428800 sda
   8        1     512000 sda1
   8        2   51915776 sda2
   8       48   52428800 sdd
   8       64   52428800 sde
   8       80   52428800 sdf
   8       32   52428800 sdc
   11       0       1152 sr0
   ```

3. Отформатируйте диск */dev/sdc*, выполнив следующую команду и указав следующие сведения в ответ на запрос:
   - *n* — новый раздел;
   - *p* — основной раздел;
   - *1*— выбор первого раздела;
   - нажмите клавишу `enter` для выбора первого цилиндра по умолчанию;
   - нажмите клавишу `enter` для выбора последнего цилиндра по умолчанию;
   - нажмите клавишу *w* для записи изменений в таблицу разделов.  

   ```bash
   fdisk /dev/sdc
   ```
   
   В соответствии с указанными выше сведениями выходные данные команды `fdisk` должны выглядеть следующим образом:

   ```output
   Device contains not a valid DOS partition table, or Sun, SGI or OSF disklabel
   Building a new DOS disklabel with disk identifier 0xf865c6ca.
   Changes will remain in memory only, until you decide to write them.
   After that, of course, the previous content won't be recoverable.

   Warning: invalid flag 0x0000 of partition table 4 will be corrected by w(rite)

   The device presents a logical sector size that is smaller than
   the physical sector size. Aligning to a physical sector (or optimal
   I/O) size boundary is recommended, or performance may be impacted.

   WARNING: DOS-compatible mode is deprecated. It's strongly recommended to
           switch off the mode (command 'c') and change display units to
           sectors (command 'u').

   Command (m for help): n
   Command action
     e   extended
     p   primary partition (1-4)
   p
   Partition number (1-4): 1
   First cylinder (1-6527, default 1):
   Using default value 1
   Last cylinder, +cylinders or +size{K,M,G} (1-6527, default 6527):
   Using default value 6527

   Command (m for help): w
   The partition table has been altered!

   Calling ioctl() to re-read partition table.
   Syncing disks.
   ```

4. Повторите предыдущую команду `fdisk` для `/dev/sdd`, `/dev/sde` и `/dev/sdf`.

5. Проверьте конфигурацию диска:

   ```bash
   cat /proc/partitions
   ```

   Выходные данные команды должны выглядеть следующим образом:

   ```output
   major minor  #blocks  name

     8       16   14680064 sdb
     8       17   14678976 sdb1
     8       32   52428800 sdc
     8       33   52428096 sdc1
     8       48   52428800 sdd
     8       49   52428096 sdd1
     8       64   52428800 sde
     8       65   52428096 sde1
     8       80   52428800 sdf
     8       81   52428096 sdf1
     8        0   52428800 sda
     8        1     512000 sda1
     8        2   51915776 sda2
     11       0    1048575 sr0
   ```

6. Проверьте состояние службы Oracle ASM и запустите ее:

   ```bash
   service oracleasm status 
   service oracleasm start
   ```

   Выходные данные команды должны выглядеть следующим образом:

   ```output
   Checking if ASM is loaded: no
   Checking if /dev/oracleasm is mounted: no
   Initializing the Oracle ASMLib driver:                     [  OK  ]
   Scanning the system for Oracle ASMLib disks:               [  OK  ]
   ```

7. Создайте диски Oracle ASM:

   ```bash
   service oracleasm createdisk ASMSP /dev/sdc1 
   service oracleasm createdisk DATA /dev/sdd1 
   service oracleasm createdisk DATA1 /dev/sde1 
   service oracleasm createdisk FRA /dev/sdf1
   ```

   Выходные данные команды должны выглядеть следующим образом:

   ```output
   Marking disk "ASMSP" as an ASM disk:                       [  OK  ]
   Marking disk "DATA" as an ASM disk:                        [  OK  ]
   Marking disk "DATA1" as an ASM disk:                       [  OK  ]
   Marking disk "FRA" as an ASM disk:                         [  OK  ]
   ```

8. Выведите список дисков Oracle ASM:

   ```bash
   service oracleasm listdisks
   ```

   Выходные данные команды должны содержать следующие диски Oracle ASM:

   ```output
    ASMSP
    DATA
    DATA1
    FRA
   ```

9. Измените пароли для пользователей root, oracle и grid. **Запишите эти новые пароли**, так как они пригодятся во время установки позже.

   ```bash
   passwd oracle 
   passwd grid 
   passwd root
   ```

10. Измените разрешение папки:

    ```bash
    chmod -R 775 /opt 
    chown grid:oinstall /opt 
    chown oracle:oinstall /dev/sdc1 
    chown oracle:oinstall /dev/sdd1 
    chown oracle:oinstall /dev/sde1 
    chown oracle:oinstall /dev/sdf1 
    chmod 600 /dev/sdc1 
    chmod 600 /dev/sdd1 
    chmod 600 /dev/sde1 
    chmod 600 /dev/sdf1
    ```

## <a name="download-and-prepare-oracle-grid-infrastructure"></a>Скачивание и подготовка Oracle Grid Infrastructure

Чтобы скачать и подготовить программное обеспечение Oracle Grid Infrastructure, сделайте следующее:

1. Скачайте Oracle Grid Infrastructure со [страницы скачивания Oracle ASM](https://www.oracle.com/database/technologies/oracle19c-linux-downloads.html). 

   Под заголовком **Oracle Database 12c Release 1 Grid Infrastructure (12.1.0.2.0) для Linux x86–64** должно быть два ZIP-файла для скачивания.

2. Скачав ZIP-файлы на клиентском компьютере, вы можете скопировать файлы на виртуальную машину по протоколу SCP:

   ```bash
   scp *.zip <publicIpAddress>:.
   ```

3. Подключитесь к виртуальной машине Oracle в Azure по протоколу SSH, чтобы переместить ZIP-файлы в папку /opt. Затем измените владельца файлов:

   ```bash
   ssh <publicIPAddress>
   sudo mv ./*.zip /opt
   cd /opt
   sudo chown grid:oinstall linuxamd64_12102_grid_1of2.zip
   sudo chown grid:oinstall linuxamd64_12102_grid_2of2.zip
   ```

4. Распакуйте файлы. (Установите инструмент Linux для распаковки, если он еще не установлен.)

   ```bash
   sudo yum install unzip
   sudo unzip linuxamd64_12102_grid_1of2.zip
   sudo unzip linuxamd64_12102_grid_2of2.zip
   ```

5. Измените разрешение:

   ```bash
   sudo chown -R grid:oinstall /opt/grid
   ```

6. Обновите настроенную область буфера. Для установки компонентов Oracle Grid требуется по крайней мере 6,8 ГБ области буфера. Размер файла подкачки по умолчанию для образов Oracle Linux в Azure всего 2048 МБ. Вам нужно увеличить значение `ResourceDisk.SwapSizeMB` в файле `/etc/waagent.conf` и перезапустить службу WALinuxAgent, чтобы изменения вступили в силу. Так как это файл только для чтения, необходимо изменить разрешения, чтобы включить доступ для записи.

   ```bash
   sudo chmod 777 /etc/waagent.conf  
   vi /etc/waagent.conf
   ```

   Найдите параметр `ResourceDisk.SwapSizeMB` и измените его значение на **8192**. Необходимо будет нажать `insert`, чтобы перейти в режим вставки, ввести значение **8192** и нажать клавишу `esc` для возврата в режим команд. Чтобы записать изменения и закрыть файл, введите `:wq` и нажмите клавишу `enter`.
   
   > [!NOTE]
   > Для достижения оптимальной производительности и настройки области буфера мы настоятельно рекомендуем использовать `WALinuxAgent`, чтобы он всегда создавался на локальном временном диске. Дополнительные сведения см. в статье [How to add a swap file in Linux Azure virtual machines](https://support.microsoft.com/en-us/help/4010058/how-to-add-a-swap-file-in-linux-azure-virtual-machines) (Как добавить файл подкачки на виртуальной машине Linux Azure).

## <a name="prepare-your-local-client-and-vm-to-run-x11"></a>Подготовка локального клиента и виртуальной машины для запуска x11
Для установки и настройки Oracle ASM требуется графический интерфейс. Для упрощения установки мы используем протокол x11. Если вы используете клиентскую систему (Mac или Linux) с возможностями X11, вы можете пропустить эту установку и настройку, предназначенную для компьютеров с Windows. 

1. Скачайте [PuTTY](https://www.putty.org/) и [Xming](https://xming.en.softonic.com/) на компьютер с Windows. Прежде чем продолжить, необходимо будет установить оба приложения со значениями по умолчанию.

2. После установки PuTTY откройте командную строку, перейдите в папку PuTTY (например, C:\Program Files\PuTTY) и запустите файл `puttygen.exe` для создания ключа.

3. В генераторе ключей PuTTY сделайте следующее:
   
   1. Чтобы создать ключ, нажмите кнопку `Generate`.
   2. Скопируйте содержимое ключа (CTRL+C).
   3. Нажмите кнопку `Save private key`.
   4. Пропустите предупреждение о защите ключа с помощью парольной фразы, а затем нажмите кнопку `OK`.

   ![Снимок экрана генератора ключей PuTTY](./media/oracle-asm/puttykeygen.png)

4. В виртуальной машине выполните следующие команды:

   ```bash
   sudo su - grid
   mkdir .ssh 
   cd .ssh
   ```

5. Создайте файл с именем `authorized_keys`. Вставьте содержимое ключа в этот файл и сохраните файл.

   > [!NOTE]
   > Ключ должен содержать строку `ssh-rsa`. Кроме того, содержимое ключа должно быть одной строкой текста.
   >  

6. В клиентской системе запустите PuTTY. В панели **Категория** перейдите к пункту **Подключение** > **SSH** > **Аутентификация**. В поле **Private key file for authentication (Файл закрытого ключа для проверки подлинности)** выберите созданный ранее ключ.

   ![Снимок экрана параметров аутентификации SSH](./media/oracle-asm/setprivatekey.png)

7. В области **Категория** выберите **Подключение** > **SSH** > **X11**. Установите флажок **Enable X11 forwarding** (Включить перенаправление X11).

   ![Снимок экрана параметров перенаправления X11 SSH](./media/oracle-asm/enablex11.png)

8. В области **Категория** выберите **Сеанс**. Введите значение `<publicIPaddress>` виртуальной машины Oracle ASM в диалоговом окне имени узла, укажите новое имя `Saved Session` и нажмите кнопку `Save`.  После сохранения щелкните `open` для подключения к виртуальной машине Oracle ASM.  При первом подключении выводится предупреждение о том, что удаленная система не кэшируется в реестре. Щелкните `yes`, чтобы добавить ее и продолжить.

   ![Снимок экрана параметров сеанса PuTTY](./media/oracle-asm/puttysession.png)

## <a name="install-oracle-grid-infrastructure"></a>Установка Oracle Grid Infrastructure

Чтобы установить Oracle Grid Infrastructure, сделайте следующее:

1. Выполните вход в качестве пользователя **grid**. Вы сможете войти, не вводя пароль. 

   > [!NOTE]
   > Если вы используете Windows, перед установкой запустите Xming.

   ```bash
   cd /opt/grid
   ./runInstaller
   ```

   Откроется установщик Oracle Grid Infrastructure 12c Release 1. Запуск установщика может занять несколько минут.

2. На странице **выбора варианта установки** выберите **Install and Configure Oracle Grid Infrastructure for a Standalone Server** (Установить и настроить Oracle Grid Infrastructure для автономного сервера).

   ![Снимок экрана со страницей выбора варианта установки в установщике](./media/oracle-asm/install01.png)

3. На странице **выбора языка продукта** выберите **английский** или другой нужный язык.  Нажмите кнопку `next`.

4. На странице **Create ASM Disk Group** (Создание группы дисков ASM) сделайте следующее:
   - Введите имя группы дисков.
   - В разделе **Redundancy** (Избыточность) выберите **External** (Внешняя).
   - В списке **Allocation Unit Size** (Размер единицы распределения) выберите значение **4**.
   - В разделе **Add Disks** (Добавление дисков) выберите **ORCLASMSP**.
   - Нажмите кнопку `next`.

5. На странице **указания пароля ASM** установите переключатель **Use same passwords for these accounts** (Использовать одинаковые пароли для этих учетных записей) и введите пароль.

   ![Снимок экрана со страницей указания пароля ASM в установщике](./media/oracle-asm/install04.png)

6. На странице **указания параметров управления** предусмотрен параметр настройки EM Cloud Control. Мы пропустим его. Щелкните `next`, чтобы продолжить. 

7. На странице **привилегированных групп ОС** используйте параметры по умолчанию. Щелкните `next`, чтобы продолжить.

8. На странице **указания расположения установки** используйте параметры по умолчанию. Щелкните `next`, чтобы продолжить.

9. На странице **создания инвентаризации** смените каталог инвентаризации на `/u01/app/grid/oraInventory`. Щелкните `next`, чтобы продолжить.

   ![Снимок экрана со страницей создания инвентаризации в установщике](./media/oracle-asm/install08.png)

10. На странице **Root script execution configurationRoot script execution configuration** (Конфигурация выполнения корневого сценария) установите флажок **Automatically run configuration scripts** (Запускать сценарии настройки автоматически). Затем установите флажок **Use "root" user credential** (Использовать учетные данные привилегированного пользователя) и введите пароль привилегированного пользователя.

    ![Снимок экрана со страницей настройки выполнения корневого сценария в установщике](./media/oracle-asm/install09.png)

11. На странице **выполнения предварительных проверок** установка завершится ошибкой. Это ожидаемое поведение. Выберите `Fix & Check Again`.

12. В диалоговом окне **Fixup Script** (Сценарий исправления) нажмите кнопку `OK`.

13. На странице **Сводка** просмотрите выбранные параметры и нажмите кнопку `Install`.

    ![Снимок экрана со страницей сводки в установщике](./media/oracle-asm/install12.png)

14. Появится диалоговое окно с предупреждением о том, что сценарии настройки может выполнять только привилегированный пользователь. Щелкните `Yes`, чтобы продолжить.

15. На странице **завершения** нажмите кнопку `Close` для завершения установки.

## <a name="set-up-your-oracle-asm-installation"></a>Настройка установки Oracle ASM

Чтобы настроить установку Oracle ASM, сделайте следующее:

1. Убедитесь, что в сеансе X11 по-прежнему используется пользователь **grid**. Может потребоваться нажать кнопку `enter`, чтобы восстановить работу терминала. Затем запустите помощник по настройке Oracle ASM:

   ```bash
   cd /u01/app/grid/product/12.1.0/grid/bin
   ./asmca
   ```

   Откроется Oracle ASM Configuration Assistant.

2. В диалоговом окне **Configure ASM: Disk Groups** (Настройка ASM: группы дисков) нажмите кнопку `Create` и `Show Advanced Options`.

3. В диалоговом окне **Create Disk Group** (Создание группы дисков) выполните следующие действия:

   - Введите имя группы дисков **DATA**.
   - В разделе **Select Member Disks** (Выбор дисков-участников) выберите **ORCL_DATA** и **ORCL_DATA1**.
   - В списке **Allocation Unit Size** (Размер единицы распределения) выберите значение **4**.
   - Щелкните `ok` для создания группы дисков.
   - Щелкните `ok`, чтобы закрыть окно подтверждения.

   ![Снимок экрана с диалоговым окном "Create Disk Group" (Создание группы дисков)](./media/oracle-asm/asm02.png)

4. В диалоговом окне **Configure ASM: Disk Groups** (Настройка ASM: группы дисков) нажмите кнопку `Create` и `Show Advanced Options`.

5. В диалоговом окне **Create Disk Group** (Создание группы дисков) выполните следующие действия:

   - Введите имя группы дисков **FRA**.
   - В разделе **Redundancy** (Избыточность) выберите **External (none)** (Внешняя(нет)).
   - В разделе **Select Member Disks** (Выбор дисков-участников) выберите **ORCL_FRA**.
   - В списке **Allocation Unit Size** (Размер единицы распределения) выберите значение **4**.
   - Щелкните `ok` для создания группы дисков.
   - Щелкните `ok`, чтобы закрыть окно подтверждения.

   ![Снимок экрана: диалоговое окно "Создание группы дисков" с выделенным параметром "Внешний" (нет).](./media/oracle-asm/asm04.png)

6. Нажмите кнопку **Выйти**, чтобы закрыть ASM Configuration Assistant.

   ![Снимок экрана с диалоговым окном "Configure ASM: Disk Groups" (Настройка ASM: группы дисков) с кнопкой "Выйти"](./media/oracle-asm/asm05.png)

## <a name="create-the-database"></a>Создание базы данных

Программное обеспечение базы данных Oracle уже установлено в образе Azure Marketplace. Чтобы создать базу данных, сделайте следующее:

1. Переключитесь на суперпользователя Oracle и инициализируйте прослушиватель для ведения журнала:

   ```bash
   su - oracle
   cd /u01/app/oracle/product/12.1.0/dbhome_1/bin
   ./dbca
   ```

   Откроется Database Configuration Assistant.

2. На странице **операций с базой данных** щелкните `Create Database`.

3. На странице **Creation Mode** (Режим создания) сделайте следующее:

   - Введите имя для базы данных.
   - Выберите **Automatic Storage Management (ASM)** в качестве **типа хранилища**.
   - В качестве **расположения файлов базы данных** используйте расположение ASM по умолчанию.
   - В качестве **области быстрого восстановления** используйте расположение ASM по умолчанию.
   - Введите **пароль администратора** и **подтвердите его**.
   - Убедитесь, что выбран параметр `create as container database`.
   - Введите значение `pluggable database name`.

4. На странице **Сводка** просмотрите выбранные параметры и нажмите кнопку `Finish`, чтобы создать базу данных.

   ![Снимок экрана со страницей сводки](./media/oracle-asm/createdb03.png)

5. База данных создана. На странице **завершения** вы можете разблокировать дополнительные учетные записи для использования этой базы данных и изменить пароли. Для этого выберите параметр **управления паролями**. В противном случае щелкните `close`.

## <a name="delete-the-vm"></a>Удаление виртуальной машины

Вы успешно настроили Oracle ASM на основе образа база данных Oracle из Azure Marketplace.  Вы можете удалить ставшие ненужными группу ресурсов, виртуальную машину и все связанные с ней ресурсы, использовав следующую команду:

```azurecli
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

[Реализация Oracle Data Guard на виртуальной машине Azure под управлением Linux](configure-oracle-dataguard.md)

[Реализация Oracle Golden Gate на виртуальной машине Azure под управлением Linux](Configure-oracle-golden-gate.md)

Ознакомьтесь со статьей [Разработка базы данных Oracle и ее внедрение в Azure](oracle-design.md)
