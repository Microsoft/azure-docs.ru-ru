---
title: Развертывание платформы SAP BusinessObjects BI в Azure для Linux | Документация Майкрософт
description: Развертывание и настройка платформы SAP BusinessObjects BI в Azure для Linux
services: virtual-machines-windows,virtual-network,storage,azure-netapp-files,azure-files,mysql
documentationcenter: saponazure
author: dennispadia
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 10/05/2020
ms.author: depadia
ms.openlocfilehash: b94e1f82409da3329eb6d978fa2ae0222928cd97
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102505942"
---
# <a name="sap-businessobjects-bi-platform-deployment-guide-for-linux-on-azure"></a>Руководство по развертыванию платформы SAP BusinessObjects BI для Linux в Azure

В этой статье описывается стратегия развертывания платформы SAP Боби в Azure для Linux. В этом примере настраиваются две виртуальные машины с SSD (цен. категория "Премиум")ными управляемыми дисками в качестве каталога установки. База данных Azure для MySQL используется для базы данных CMS, а Azure NetApp Files для сервера репозитория файлов совместно используется на обоих серверах. Веб-приложение Tomcat Java по умолчанию и приложение платформы BI устанавливаются вместе на обеих виртуальных машинах. Для балансировки нагрузки запроса пользователя используется шлюз приложений с собственными возможностями разгрузки TLS/SSL.

Этот тип архитектуры эффективен для небольших развертываний или нерабочей среды. В рабочей среде или масштабном развертывании можно использовать отдельные узлы для веб-приложений, а также несколько узлов Боби приложений, разрешающих серверу обрабатывать дополнительные сведения.

![Развертывание SAP Боби в Azure для Linux](media/businessobjects-deployment-guide/businessobjects-deployment-linux.png)

В этом примере используется следующая версия продукта и макет файловой системы.

- SAP BusinessObjects Platform 4,3
- SUSE Linux Enterprise Server 12 SP5
- База данных Azure для MySQL (версия: 8.0.15)
- Соединитель с API MySQL C — либмисклклиент (версия: 6.1.11)

| Файловая система        | Описание                                                                                                               | Размер (ГБ)             | Владелец  | Group  | Служба хранилища                    |
|--------------------|---------------------------------------------------------------------------------------------------------------------------|-----------------------|--------|--------|----------------------------|
| /usr/sap           | Файловая система для установки экземпляра SAP Боби, веб-приложения Tomcat по умолчанию и драйверов баз данных (при необходимости) | Рекомендации по выбору размера SAP | bl1adm | sapsys | Диск управляемого класса Premium — SSD |
| /уср/сап/фрсинпут  | Каталог подключения предназначен для общих файлов на всех узлах Боби, которые будут использоваться в качестве каталога репозитория входного файла.  | Бизнес-потребности         | bl1adm | sapsys | Azure NetApp Files         |
| /уср/сап/фрсаутпут | Каталог подключения предназначен для общих файлов на всех узлах Боби, которые будут использоваться в качестве каталога репозитория выходного файла. | Бизнес-потребности         | bl1adm | sapsys | Azure NetApp Files         |

## <a name="deploy-linux-virtual-machine-via-azure-portal"></a>Развертывание виртуальной машины Linux с помощью портал Azure

В этом разделе мы создадим две виртуальные машины (ВМ) с образом операционной системы Linux для платформы SAP Боби. Ниже приведены высокоуровневые действия по созданию виртуальных машин.

1. Создание [группы ресурсов](../../../azure-resource-manager/management/manage-resource-groups-portal.md#create-resource-groups)

2. Создайте [виртуальную сеть](../../../virtual-network/quick-create-portal.md#create-a-virtual-network).

   - Не используйте одну подсеть для всех служб Azure в развертывании платформы SAP BI. На основе архитектуры платформы SAP BI необходимо создать несколько подсетей. В этом развертывании мы создадим три подсети: подсеть приложения, подсеть хранилища файлов и подсеть шлюза приложений.
   - В Azure шлюз приложений и Azure NetApp Files всегда должны находиться в отдельной подсети. Дополнительные сведения см. в разделе [шлюз приложений Azure](../../../application-gateway/configuration-overview.md) и [рекомендации по Azure NetApp Files планировании сети](../../../azure-netapp-files/azure-netapp-files-network-topologies.md) .

3. Создайте группу доступности.

   - Для обеспечения избыточности каждого уровня в развертывании с несколькими экземплярами размещайте виртуальные машины для каждого уровня в группе доступности. Убедитесь, что группы доступности разделяются для каждого уровня в зависимости от архитектуры.

4. Создайте виртуальную машину 1 **(azusbosl1).**

   - Вы можете использовать пользовательский образ или выбрать образ из Azure Marketplace. См. статью [развертывание виртуальной машины из Azure Marketplace для SAP](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/virtual-machines/workloads/sap/deployment-guide.md#scenario-1-deploying-a-vm-from-the-azure-marketplace-for-sap) или [развертывание виртуальной машины с помощью ПОЛЬЗОВАТЕЛЬСКОГО образа для SAP](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/virtual-machines/workloads/sap/deployment-guide.md#scenario-2-deploying-a-vm-with-a-custom-image-for-sap) в зависимости от ваших нужд.

5. Создайте виртуальную машину 2 **(azusbosl2).**
6. Добавьте один SSD (цен. категория "Премиум") диск. Он будет использоваться в качестве каталога установки SAP Боби.

## <a name="provision-azure-netapp-files"></a>Инициализация Azure NetApp Files

Прежде чем продолжить установку для Azure NetApp Files, ознакомьтесь с [документацией по файлам Azure NetApp](../../../azure-netapp-files/azure-netapp-files-introduction.md).

Azure NetApp Files доступен в нескольких [регионах Azure](https://azure.microsoft.com/global-infrastructure/services/?products=netapp). Проверьте, предлагает ли выбранный регион Azure Azure NetApp Files.

Страница [доступность Azure NetApp Files по регионам Azure](https://azure.microsoft.com/global-infrastructure/services/?products=netapp&regions=all) используется для проверки доступности Azure NetApp Files по регионам.

Запросите подключение к Azure NetApp Files, перейдя на [Azure NetApp Files инструкции](../../../azure-netapp-files/azure-netapp-files-register.md) перед развертыванием Azure NetApp Files.

### <a name="deploy-azure-netapp-files-resources"></a>Развертывание ресурсов Azure NetApp Files

В следующих инструкциях предполагается, что вы уже развернули [виртуальную сеть Azure](../../../virtual-network/virtual-networks-overview.md). Azure NetApp Files ресурсы и виртуальные машины, на которых будут подключены Azure NetApp Files ресурсы, должны быть развернуты в одной виртуальной сети Azure или в одноранговых виртуальных сетях Azure.

1. Если вы еще не развернули ресурсы, запросите подключение [к Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-register.md).

2. Создайте учетную запись NetApp в выбранном регионе Azure, следуя инструкциям в разделе [Создание учетной записи NetApp](../../../azure-netapp-files/azure-netapp-files-create-netapp-account.md).

3. Настройте пул емкости Azure NetApp Files, следуя инструкциям в разделе [Настройка пула емкости Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-set-up-capacity-pool.md).

   - Архитектура платформы SAP BI, представленная в этой статье, использует один пул емкости Azure NetApp Files на уровне службы *Premium* . Для сервера репозитория файлов SAP BI в Azure рекомендуется использовать уровень Azure NetApp Files *Premium* или *Ultra* [Service](../../../azure-netapp-files/azure-netapp-files-service-levels.md).

4. Делегируйте подсеть Azure NetApp Files, как описано в инструкциях в разделе [Делегирование подсети для Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-delegate-subnet.md).

5. Разверните Azure NetApp Files тома, следуя инструкциям в разделе [Создание тома NFS для Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-create-volumes.md).

   Том использовании можно развернуть как NFSv3 и Нфсв 4.1, так как оба протокола поддерживаются для платформы SAP Боби. Разверните тома в соответствующей подсети Azure NetApp Files. IP-адреса томов NetApp для Azure назначаются автоматически.

Помните, что ресурсы Azure NetApp Files и виртуальные машины Azure должны находиться в одной виртуальной сети Azure или в одноранговых виртуальных сетях Azure. Например, азусбоби-фрсинпут, азусбоби-фрсаутпут — это имена томов и nfs://10.31.2.4/azusbobi-frsinput, nfs://10.31.2.4/azusbobi-frsoutput — пути к файлам для Azure NetApp Filesных томов.

- Volume азусбоби-фрсинпут (nfs://10.31.2.4/azusbobi-frsinput)
- Volume азусбоби-фрсаутпут (nfs://10.31.2.4/azusbobi-frsoutput)

### <a name="important-considerations"></a>Важные замечания

При создании Azure NetApp Files для сервера репозитория файлов платформы SAP Боби необходимо учитывать следующее.

1. Пул минимальных мощностей — 4 тебибитес (тиб).
2. Минимальный размер тома — 100 гибибайтах (гиб).
3. Azure NetApp Files и все виртуальные машины, на которых будут подключаться Azure NetApp Filesные тома, должны находиться в одной виртуальной сети Azure или в [одноранговой виртуальной сети](../../../virtual-network/virtual-network-peering-overview.md) в том же регионе. В настоящее время поддерживается доступ к Azure NetApp Files через пиринг виртуальных сетей в том же регионе. Доступ к NetApp Azure через Глобальный пиринг пока не поддерживается.
4. Выбранная виртуальная сеть должна иметь подсеть, делегированную Azure NetApp Files.
5. С помощью [политики экспорта](../../../azure-netapp-files/azure-netapp-files-configure-export-policy.md)Azure NetApp Files можно управлять разрешенными клиентами, типом доступа (чтение и запись, только чтение и т. д.).
6. Функция Azure NetApp Files еще не поддерживает зоны. Сейчас эта функция не развернута во всех зонах доступности в регионе Azure. Следует учитывать возможную задержку в некоторых регионах Azure.
7. Тома Azure NetApp Files можно развернуть в виде томов NFSv3 или NFSv4.1. Оба протокола поддерживаются для приложений платформы SAP BI.

## <a name="configure-file-systems-on-linux-servers"></a>Настройка файловых систем на серверах Linux

Во всех действиях этого раздела используются следующие префиксы.

**[A]**: шаг применяется ко всем узлам.

### <a name="format-and-mount-sap-file-system"></a>Форматирование и подключение файловой системы SAP

1. **[A]** список всех подключенных дисков

    ```bash
    sudo lsblk
    NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    sda      8:0    0   30G  0 disk
    ├─sda1   8:1    0    2M  0 part
    ├─sda2   8:2    0  512M  0 part /boot/efi
    ├─sda3   8:3    0    1G  0 part /boot
    └─sda4   8:4    0 28.5G  0 part /
    sdb      8:16   0   32G  0 disk
    └─sdb1   8:17   0   32G  0 part /mnt
    sdc      8:32   0  128G  0 disk
    sr0     11:0    1  628K  0 rom  
    # Premium SSD of 128 GB is attached to Virtual Machine, whose device name is sdc
    ```

2. **[A]** устройство блочного формата для/usr/SAP

    ```bash
    sudo mkfs.xfs /dev/sdc
    ```

3. **[A]** создайте каталог подключения.

    ```bash
    sudo mkdir -p /usr/sap
    ```

4. **[A]** Получение UUID блочного устройства

    ```bash
    sudo blkid

    #It will display information about block device. Copy UUID of the formatted block device

    /dev/sdc: UUID="0eb5f6f8-fa77-42a6-b22d-7a9472b4dd1b" TYPE="xfs"
    ```

5. **[A]** поддержание записи подключения файловой системы в/etc/fstab

    ```bash
    sudo echo "UUID=0eb5f6f8-fa77-42a6-b22d-7a9472b4dd1b /usr/sap xfs defaults,nofail 0 2" >> /etc/fstab
    ```

6. **[A]** подключить файловую систему

    ```bash
    sudo mount -a

    sudo df -h

    Filesystem                     Size  Used Avail Use% Mounted on
    devtmpfs                       7.9G  8.0K  7.9G   1% /dev
    tmpfs                          7.9G   82M  7.8G   2% /run
    tmpfs                          7.9G     0  7.9G   0% /sys/fs/cgroup
    /dev/sda4                       29G  1.8G   27G   6% /
    tmpfs                          1.6G     0  1.6G   0% /run/user/1000
    /dev/sda3                     1014M   87M  928M   9% /boot
    /dev/sda2                      512M  1.1M  511M   1% /boot/efi
    /dev/sdb1                       32G   49M   30G   1% /mnt
    /dev/sdc                       128G   29G  100G  23% /usr/sap
    ```

### <a name="mount-azure-netapp-files-volume"></a>Подключить Azure NetApp Files том

1. **[A]** Создайте каталоги подключения.

   ```bash
   sudo mkdir -p /usr/sap/frsinput
   sudo mkdir -p /usr/sap/frsoutput
   ```

2. **[A]** настройте клиентскую ОС для поддержки подключения нфсв 4.1 **(применимо только при использовании нфсв 4.1)** .

   Если вы используете Azure NetApp Files томов с протоколом Нфсв 4.1, выполните следующую настройку на всех виртуальных машинах, где необходимо подключить Azure NetApp Files Нфсв 4.1.

   **Проверка параметров домена NFS**

   Убедитесь, что домен настроен в качестве домена Azure NetApp Files по умолчанию,  **defaultv4iddomain.com** , а для сопоставления установлено значение **никто**.

   ```bash
   sudo cat /etc/idmapd.conf
   # Example
   [General]
   Domain = defaultv4iddomain.com
   [Mapping]
   Nobody-User = nobody
   Nobody-Group = nobody
   ```

   > [!Important]
   >
   > Не забудьте установить домен NFS в/ЕТК/идмапд.конф на виртуальной машине в соответствии с конфигурацией домена по умолчанию в Azure NetApp Files: **defaultv4iddomain.com**. В случае несоответствия между конфигурацией домена NFS-клиента (например, виртуальной машиной) и сервером NFS, т. е. конфигурацией Azure NetApp, разрешения на доступ к файлам на томах NetApp в Azure, подключенных к виртуальным машинам, будут отображаться как никто.

   Проверяет значение `nfs4_disable_idmapping`. Ему должно быть задано значение **Y**. Чтобы создать структуру каталогов, в которой находится параметр `nfs4_disable_idmapping`, выполните команду mount. Вы не сможете вручную создать каталог в /sys/modules, так как доступ зарезервирован для ядра или драйверов.

   ```bash
   # Check nfs4_disable_idmapping
   cat /sys/module/nfs/parameters/nfs4_disable_idmapping

   # If you need to set nfs4_disable_idmapping to Y
   mkdir /mnt/tmp
   mount -t nfs -o sec=sys,vers=4.1 10.31.2.4:/azusbobi-frsinput /mnt/tmp
   umount /mnt/tmp

   echo "Y" > /sys/module/nfs/parameters/nfs4_disable_idmapping

   # Make the configuration permanent
   echo "options nfs nfs4_disable_idmapping=Y" >> /etc/modprobe.d/nfs.conf
   ```

3. **[A]** Добавление записей подключения

   При использовании NFSv3

   ```bash
   sudo echo "10.31.2.4:/azusbobi-frsinput /usr/sap/frsinput  nfs   rw,hard,rsize=65536,wsize=65536,vers=3" >> /etc/fstab
   sudo echo "10.31.2.4:/azusbobi-frsoutput /usr/sap/frsoutput  nfs   rw,hard,rsize=65536,wsize=65536,vers=3" >> /etc/fstab
   ```

   При использовании Нфсв 4.1

   ```bash
   sudo echo "10.31.2.4:/azusbobi-frsinput /usr/sap/frsinput  nfs   rw,hard,rsize=65536,wsize=65536,vers=4.1,sec=sys" >> /etc/fstab
   sudo echo "10.31.2.4:/azusbobi-frsoutput /usr/sap/frsoutput  nfs   rw,hard,rsize=65536,wsize=65536,vers=4.1,sec=sys" >> /etc/fstab
   ```

4. **[A]** подключение томов NFS

   ```bash
   sudo mount -a

   sudo df -h

   Filesystem                     Size  Used Avail Use% Mounted on
   devtmpfs                       7.9G  8.0K  7.9G   1% /dev
   tmpfs                          7.9G   82M  7.8G   2% /run
   tmpfs                          7.9G     0  7.9G   0% /sys/fs/cgroup
   /dev/sda4                       29G  1.8G   27G   6% /
   tmpfs                          1.6G     0  1.6G   0% /run/user/1000
   /dev/sda3                     1014M   87M  928M   9% /boot
   /dev/sda2                      512M  1.1M  511M   1% /boot/efi
   /dev/sdb1                       32G   49M   30G   1% /mnt
   /dev/sdc                       128G   29G  100G  23% /usr/sap
   10.31.2.4:/azusbobi-frsinput   101T   18G  100T   1% /usr/sap/frsinput
   10.31.2.4:/azusbobi-frsoutput  100T  512K  100T   1% /usr/sap/frsoutput
   ```

## <a name="configure-cms-database---azure-database-for-mysql"></a>Настройка базы данных CMS — база данных Azure для MySQL

В этом разделе содержатся сведения о том, как подготавливать базу данных Azure для MySQL с помощью портал Azure. В нем также приводятся инструкции по созданию баз данных CMS и Audit для платформы SAP Боби и учетной записи пользователя для доступа к базе данных.

Рекомендации применимы только в том случае, если вы используете Azure DB для MySQL. Инструкции для других баз данных см. в документации по SAP или конкретной базе данных.

### <a name="create-an-azure-database-for-mysql"></a>Создание базы данных Azure для MySQL

Войдите в портал Azure и выполните действия, описанные в этом [кратком руководстве по началу работы с базой данных Azure для MySQL](../../../mysql/quickstart-create-mysql-server-database-using-azure-portal.md). Некоторые моменты, которые следует учитывать при подготовке базы данных Azure для MySQL.

1. Выберите тот же регион для базы данных Azure для MySQL, в котором работают серверы приложений платформы SAP BI.

2. Выберите поддерживаемую версию базы данных на основе [матрицы доступности продуктов (PAM) для SAP BI](https://support.sap.com/pam) , относящейся к версии SAP Боби. Следуйте тем же рекомендациям по совместимости, что и для MySQL AB в SAP PAM

3. В области "вычисления и хранение" выберите **настроить сервер** и выберите соответствующую ценовую категорию в зависимости от размера выходных данных.

4. Автоматическое **расширение хранилища** включено по умолчанию. Помните, что [хранилище](../../../mysql/concepts-pricing-tiers.md#storage) можно масштабировать, но не уменьшать.

5. По умолчанию **срок хранения резервной копии** составляет семь дней, но при необходимости его можно [настроить](../../../mysql/howto-restore-server-portal.md#set-backup-configuration) до 35 дней.

6. Резервное копирование базы данных Azure для MySQL по умолчанию является локально избыточным, поэтому если требуется резервное копирование сервера в геоизбыточном хранилище, выберите пункт **Геоизбыточное** из **резервных копий параметры избыточности**.

> [!NOTE]
> Изменение [параметров избыточности резервных копий](../../../mysql/concepts-backup.md#backup-redundancy-options) после создания сервера не поддерживается.

### <a name="configure-connection-security"></a>Настройка безопасности подключения

По умолчанию созданный сервер защищен брандмауэром и не является общедоступным. Чтобы предоставить доступ к виртуальной сети, в которой работают серверы приложений платформы SAP BI, выполните указанные ниже действия.  

1. Перейдите в раздел ресурсы сервера в портал Azure и выберите пункт **Безопасность подключения** в меню слева для ресурса сервера.
2. Выберите **Да** , чтобы **Разрешить доступ к службам Azure**.
3. В разделе правила ВИРТУАЛЬНОЙ сети выберите **Добавление существующей виртуальной сети**. Выберите виртуальную сеть и подсеть сервера приложений платформы SAP BI. Кроме того, необходимо предоставить доступ к полю перехода или другим серверам, с которых можно подключить [MySQL Workbench](../../../mysql/connect-workbench.md) к базе данных Azure для MySQL. MySQL Workbench будет использоваться для создания базы данных CMS и Audit
4. После добавления виртуальных сетей нажмите кнопку **сохранить**.

### <a name="create-cms-and-audit-database"></a>Создание CMS и базы данных аудита

1. Скачайте и установите MySQL Workbench с [веб-сайта MySQL](https://dev.mysql.com/downloads/workbench/). Убедитесь, что вы установили MySQL Workbench на сервере, который имеет доступ к базе данных Azure для MySQL.

2. Подключитесь к серверу с помощью MySQL Workbench. Выполните инструкции, описанные в этой [статье](../../../mysql/connect-workbench.md#get-connection-information). Если проверка подключения прошла успешно, вы получите следующее сообщение:

   ![Подключение к SQL Workbench](media/businessobjects-deployment-guide/businessobjects-sql-workbench.png)

3. На вкладке SQL-запрос выполните приведенный ниже запрос, чтобы создать схему для CMS и базы данных аудита.

   ```sql
   # Here cmsbl1 is the database name of CMS database. You can provide the name you want for CMS database.
   CREATE SCHEMA `cmsbl1` DEFAULT CHARACTER SET utf8;

   # auditbl1 is the database name of Audit database. You can provide the name you want for CMS database.
   CREATE SCHEMA `auditbl1` DEFAULT CHARACTER SET utf8;
   ```
   
4. Создание учетной записи пользователя для подключения к схеме

   ```sql
   # Create a user that can connect from any host, use the '%' wildcard as a host part
   CREATE USER 'cmsadmin'@'%' IDENTIFIED BY 'password';
   CREATE USER 'auditadmin'@'%' IDENTIFIED BY 'password';

   # Grant all privileges to a user account over a specific database:
   GRANT ALL PRIVILEGES ON cmsbl1.* TO 'cmsadmin'@'%' WITH GRANT OPTION;
   GRANT ALL PRIVILEGES ON auditbl1.* TO 'auditadmin'@'%' WITH GRANT OPTION;

   # Following any updates to the user privileges, be sure to save the changes by issuing the FLUSH PRIVILEGES
   FLUSH PRIVILEGES;
   ```

5. Проверка прав и ролей учетной записи пользователя MySQL

   ```sql
   USE sys;
   SHOW GRANTS for 'cmsadmin'@'%';
   +------------------------------------------------------------------------+
   | Grants for cmsadmin@%                                                  |
   +------------------------------------------------------------------------+
   | GRANT USAGE ON *.* TO `cmsadmin`@`%`                                   |
   | GRANT ALL PRIVILEGES ON `cmsbl1`.* TO `cmsadmin`@`%` WITH GRANT OPTION |
   +------------------------------------------------------------------------+

   USE sys;
   SHOW GRANTS FOR 'auditadmin'@'%';
   +----------------------------------------------------------------------------+
   | Grants for auditadmin@%                                                    |
   +----------------------------------------------------------------------------+
   | GRANT USAGE ON *.* TO `auditadmin`@`%`                                     |
   | GRANT ALL PRIVILEGES ON `auditbl1`.* TO `auditadmin`@`%` WITH GRANT OPTION |
   +----------------------------------------------------------------------------+
   ```

### <a name="install-mysql-c-api-connector-libmysqlclient-on-linux-server"></a>Установка соединителя C API MySQL (либмисклклиент) на сервере Linux

Для доступа к базе данных серверу приложений SAP Боби требуются клиент и драйверы базы данных. Соединитель API MySQL C для Linux должен использоваться для доступа к базам данных CMS и Audit. Подключение ODBC к базе данных CMS не поддерживается. В этом разделе приводятся инструкции по настройке соединителя C API MySQL в Linux.

1. Дополнительные сведения о драйверах, совместимых с базой данных Azure для MySQL, см. в статье [драйверы и инструменты управления MySQL, совместимые с базой данных Azure для MySQL](../../../mysql/concepts-compatibility.md) . Проверьте наличие драйвера **MySQL Connector/C (либмисклклиент)** в этой статье.

2. Обратитесь к этой [ссылке](https://downloads.mysql.com/archives/c-c/) , чтобы скачать драйверы.

3. Выберите операционную систему и скачайте пакет "RPM" общего компонента соединителя MySQL. В этом примере используется версия соединителя MySQL-Connector-c-Shared-6.1.11.

4. Установите соединитель во все экземпляры приложения SAP Боби.

   ```bash
   # Install rpm package
   SLES: sudo zypper install <package>.rpm
   RHEL: sudo yum install <package>.rpm
   ```

5. Проверьте путь к libmysqlclient.so.

   ```bash
   # Find the location of libmysqlclient.so file
   whereis libmysqlclient

   # sample output
   libmysqlclient: /usr/lib64/libmysqlclient.so
   ```

6. Задайте LD_LIBRARY_PATH, чтобы указать `/usr/lib64` каталог для учетной записи пользователя, которая будет использоваться для установки.

   ```bash
   # This configuration is for bash shell. If you are using any other shell for sidadm, kindly set environment variable accordingly.
   vi /home/bl1adm/.bashrc

   export LD_LIBRARY_PATH=/usr/lib64
   ```

## <a name="server-preparation"></a>Подготовка сервера

Во всех действиях этого раздела используются следующие префиксы.

**[A]**: шаг применяется ко всем узлам.

1. **[A]** на основе разновидности Linux (SLES или RHEL) необходимо задать параметры ядра и установить необходимые библиотеки. См. раздел **требования к системе** в [руководстве по установке платформы бизнес-аналитики для UNIX](https://help.sap.com/viewer/65018c09dbe04052b082e6fc4ab60030/4.3/en-US).

2. **[A]** убедитесь, что часовой пояс на компьютере установлен правильно. Дополнительные сведения о [требованиях UNIX и Linux](https://help.sap.com/viewer/65018c09dbe04052b082e6fc4ab60030/4.3/en-US/46b143336e041014910aba7db0e91070.html) см. в разделе руководства по установке.

3. **[A]** создайте учетную запись пользователя (**BL1** ADM) и группу (sapsys), в которой могут выполняться фоновые процессы программного обеспечения. Используйте эту учетную запись для выполнения установки и запуска программного обеспечения. Учетной записи не требуются привилегии root.

4. **[A]** задайте для среды учетной записи пользователя (**BL1** ADM) поддерживаемый языковой стандарт UTF-8 и убедитесь, что программное обеспечение консоли поддерживает наборы символов UTF-8. Чтобы убедиться, что операционная система использует правильный языковой стандарт, задайте для переменных среды LC_ALL и LANG предпочитаемый язык в пользовательской среде (**BL1** ADM).

   ```bash
   # This configuration is for bash shell. If you are using any other shell for sidadm, kindly set environment variable accordingly.
   vi /home/bl1adm/.bashrc

   export LANG=en_US.utf8
   export LC_ALL=en_US.utf8
   ```

5. **[A]** настройте учетную запись пользователя (**BL1** ADM).

   ```bash
   # Set ulimit for bl1adm to unlimited
   root@azusbosl1:~> ulimit -f unlimited bl1adm
   root@azusbosl1:~> ulimit -u unlimited bl1adm

   root@azusbosl1:~> su - bl1adm
   bl1adm@azusbosl1:~> ulimit -a

   core file size          (blocks, -c) unlimited
   data seg size           (kbytes, -d) unlimited
   scheduling priority             (-e) 0
   file size               (blocks, -f) unlimited
   pending signals                 (-i) 63936
   max locked memory       (kbytes, -l) 64
   max memory size         (kbytes, -m) unlimited
   open files                      (-n) 1024
   pipe size            (512 bytes, -p) 8
   POSIX message queues     (bytes, -q) 819200
   real-time priority              (-r) 0
   stack size              (kbytes, -s) 8192
   cpu time               (seconds, -t) unlimited
   max user processes              (-u) unlimited
   virtual memory          (kbytes, -v) unlimited
   file locks                      (-x) unlimited
   ```

6. Скачайте и извлеките носители для платформы SAP BusinessObjects BI из SAP Service Marketplace.

## <a name="installation"></a>Установка

Проверьте языковой стандарт для учетной записи пользователя **BL1** ADM на сервере.

```bash
bl1adm@azusbosl1:~> locale
LANG=en_US.utf8
LC_ALL=en_US.utf8
```

Перейдите к носителю SAP BusinessObjects BI Platform и выполните приведенную ниже команду с **BL1** ADM User-

```bash
./setup.sh -InstallDir /usr/sap/BL1
```

Следуйте указаниям по установке [платформы SAP Боби](https://help.sap.com/viewer/product/SAP_BUSINESSOBJECTS_BUSINESS_INTELLIGENCE_PLATFORM) для UNIX, относящимся к вашей версии. Некоторые моменты, которые следует учитывать при установке платформы SAP Боби.

- На экране **Настройка регистрации продукта** вы можете использовать временный ключ лицензии для решения SAP BUSINESSOBJECTS из sap Примечание [1288121](https://launchpad.support.sap.com/#/notes/1288121) или создать лицензионный ключ в SAP Service Marketplace.

- На экране **Выбор типа установки** выберите **полная** установка на первом сервере (azusbosl1), для другого сервера (azusbosl2) выберите **Пользовательский/развернуть** , чтобы развернуть существующую установку Боби.

- На экране **Выбор по умолчанию или существующей базы данных** выберите **настроить существующую базу данных**, в результате чего будет предложено выбрать CMS и базу данных аудита. Выберите **MySQL** для типа базы данных CMS и тип базы данных аудита.

  Если вы не хотите настраивать аудит во время установки, можно также выбрать вариант нет базы данных аудита.

- Выберите соответствующие параметры на **экране Выбор сервера веб-приложений Java** на основе архитектуры SAP Боби. В этом примере мы выбрали вариант 1, который устанавливает сервер Tomcat на той же платформе SAP Боби.

- Введите сведения о базе данных CMS в **базе данных настройки РЕПОЗИТОРИЯ CMS — MySQL**. Пример ввода сведений о базе данных CMS для установки Linux. База данных Azure для MySQL используется по умолчанию с портом 3306
  
  ![Развертывание SAP Боби в базе данных Linux — CMS](media/businessobjects-deployment-guide/businessobjects-deployment-linux-sql-cms.png)

- Используемых Введите сведения о базе данных аудита в окне **Настройка репозитория аудита база данных — MySQL**. Пример входных данных для аудита для установки Linux.

  ![Развертывание SAP Боби в Linux — база данных аудита](media/businessobjects-deployment-guide/businessobjects-deployment-linux-sql-audit.png)

- Следуйте инструкциям и введите необходимые входные данные для завершения установки.

Для развертывания с несколькими экземплярами запустите установку на втором узле (azusbosl2). На экране **Выбор типа установки** выберите **Пользовательский/развернуть** , который развернет существующую программу установки Боби.

В базе данных Azure для предложения MySQL шлюз используется для перенаправления подключений к экземплярам сервера. После установки подключения в клиенте MySQL отображается версия MySQL, установленная в шлюзе, а не фактическая версия, которая работает на вашем экземпляре сервера MySQL. Чтобы определить версию MySQL на экземпляре сервера, выполните команду `SELECT VERSION();` в командной строке MySQL. Так что в центральной консоли управления (CMC) вы увидите другую версию базы данных, которая по сути является версией, установленной на шлюзе. Для получения дополнительных сведений проверьте [Поддерживаемые версии сервера базы данных Azure для MySQL](../../../mysql/concepts-supported-versions.md) .

![Развертывание SAP Боби в Linux — параметры CMC](media/businessobjects-deployment-guide/businessobjects-deployment-linux-sql-cmc.png)

```sql
# Run direct query to the database using MySQL Workbench

select version();

+-----------+
| version() |
+-----------+
| 8.0.15    |
+-----------+
```

## <a name="post-installation"></a>После установки

### <a name="tomcat-clustering---session-replication"></a>Кластеризация Tomcat — репликация сеансов

Tomcat поддерживает кластеризацию двух или более серверов приложений для репликации и отработки отказа сеанса. Сеансы платформы SAP Боби сериализуются, сеанс пользователя может легко отключаться на другой экземпляр Tomcat даже в случае сбоя сервера приложений.

Например, если пользователь подключен к веб-серверу, который завершается ошибкой, когда пользователь найдет на иерархию папок в приложении SAP BI. В правильно настроенном кластере пользователь может продолжить навигацию по иерархии папок без перенаправления на страницу входа.

В SAP Note [2808640](https://launchpad.support.sap.com/#/notes/2808640)действия по настройке кластеризации Tomcat предоставляются с помощью многоадресной рассылки. Но в Azure многоадресная рассылка не поддерживается. Чтобы сделать кластер Tomcat работать в Azure, необходимо использовать [статикмембершипинтерцептор](https://tomcat.apache.org/tomcat-8.0-doc/config/cluster-interceptor.html#Static_Membership) (примечание SAP [2764907](https://launchpad.support.sap.com/#/notes/2764907)). Проверьте [кластеризацию Tomcat, используя статическое членство для платформы SAP BUSINESSOBJECTS BI](https://blogs.sap.com/2020/09/04/sap-on-azure-tomcat-clustering-using-static-membership-for-sap-businessobjects-bi-platform/) в блоге SAP, чтобы настроить кластер Tomcat в Azure.

### <a name="load-balancing-web-tier-of-sap-bi-platform"></a>Веб-уровень балансировки нагрузки платформы SAP BI

В развертывании с несколькими экземплярами SAP Боби серверы веб-приложений Java (веб-уровень) работают на двух или более узлах. Для равномерного распределения нагрузки пользователей между веб-серверами можно использовать балансировщик нагрузки между конечными пользователями и веб-серверами. В Azure можно использовать Azure Load Balancer или шлюз приложений Azure для управления трафиком к серверам веб-приложений. Сведения о каждом предложении описаны в следующем разделе.

#### <a name="azure-load-balancer-network-based-load-balancer"></a>Балансировщик нагрузки Azure (балансировщик нагрузки на основе сети)

[Azure Load Balancer](../../../load-balancer/load-balancer-overview.md) — это высокопроизводительная балансировщик нагрузки уровня 4 (TCP, UDP) с низкой задержкой, который распределяет трафик между работоспособными виртуальными машинами. Зонд работоспособности подсистемы балансировки нагрузки наблюдает за заданным портом на каждой ВИРТУАЛЬНОЙ машине и распределяет трафик только на рабочие виртуальные машины. Можно выбрать общедоступную подсистему балансировки нагрузки или внутреннюю подсистему балансировки нагрузки в зависимости от того, требуется ли доступ к платформе SAP BI из Интернета. Его зона избыточна, обеспечивая высокий уровень доступности в Зоны доступности.

Дополнительные сведения см. в разделе внутренний Load Balancer, где сервер веб-приложений работает на порте 8080, по умолчанию это HTTP-порт Tomcat, который будет отслеживаться проверкой работоспособности. Поэтому любой входящий запрос, поступающий от конечных пользователей, будет перенаправлен на серверы веб-приложений (azusbosl1 или azusbosl2) во внутреннем пуле. Балансировщик нагрузки не поддерживает завершение TLS/SSL (также известное как разгрузка TLS/SSL). Если вы используете балансировщик нагрузки Azure для распределения трафика между веб-серверами, рекомендуется использовать Load Balancer (цен. категория "Стандартный").

> [!NOTE]
> Если в серверный пул внутреннего (без общедоступного IP-адреса) Azure Load Balancer ценовой категории "Стандартный" помещаются виртуальные машины без общедоступных IP-адресов, у них не будет исходящего подключения к Интернету без дополнительной настройки, разрешающей маршрутизацию к общедоступным конечным точкам. Подробные сведения о такой настройке см. в статье [Подключение к общедоступной конечной точке для виртуальных машин с помощью Azure Load Balancer (цен. категория "Стандартный") в сценариях обеспечения высокого уровня доступности SAP](high-availability-guide-standard-load-balancer-outbound-connections.md).

![Azure Load Balancer балансировки трафика между веб-серверами](media/businessobjects-deployment-guide/businessobjects-deployment-load-balancer.png)

#### <a name="azure-application-gateway-web-application-load-balancer"></a>Шлюз приложений Azure (балансировщик нагрузки веб-приложений)

[Шлюз приложений Azure (АГВ)](../../../application-gateway/overview.md) предоставляет контроллер доставки приложений (ADC) в качестве службы, который используется, чтобы помочь приложению направить пользовательский трафик на один или несколько серверов веб-приложений. Она предлагает различные возможности балансировки нагрузки уровня 7, такие как разгрузка TLS/SSL, брандмауэр веб-приложения (WAF), сходство сеансов на основе файлов cookie и другие для ваших приложений.

В платформе SAP BI шлюз приложений направляет веб-трафик приложений в указанные ресурсы во внутреннем пуле — azusbosl1 или azusbos2. Вы назначаете прослушиватель для порта, создаете правила и добавляете ресурсы во внутренний пул. На рисунке ниже шлюз приложений с частным интерфейсным IP-адресом (10.31.3.20) выступает в качестве точки входа для пользователей, обрабатывает входящие подключения TLS/SSL (HTTPS-TCP/443), расшифровывает TLS/SSL и передает незашифрованный запрос (HTTP-TCP/8080) на серверы в серверном пуле. Используя встроенную функцию завершения TLS/SSL, нам нужно просто поддерживать один сертификат TLS/SSL в шлюзе приложений, что упрощает операции.

![Шлюз приложений для балансировки трафика между веб-серверами](media/businessobjects-deployment-guide/businessobjects-deployment-application-gateway.png)

Чтобы настроить шлюз приложений для веб-сервера SAP Боби, см. статью [Балансировка нагрузки на веб-серверах SAP Боби с помощью шлюза приложений Azure](https://blogs.sap.com/2020/09/17/sap-on-azure-load-balancing-web-application-servers-for-sap-bobi-using-azure-application-gateway/) в блоге SAP.

> [!NOTE]
> Мы рекомендуем использовать шлюз приложений Azure для балансировки нагрузки трафика на веб-сервер, так как он предоставляет функции, такие как разгрузка SSL, централизованное управление SSL для уменьшения количества операций шифрования и расшифровки на сервере, Round-Robin алгоритм для распределения трафика, возможности брандмауэра веб-приложения (WAF), высокий уровень доступности и т. д.

### <a name="sap-businessobjects-bi-platform---back-up-and-restore"></a>Платформа SAP BusinessObjects BI — резервное копирование и восстановление

Резервное копирование и восстановление — это процесс создания периодических копий данных и приложений в отдельном расположении. Поэтому его можно восстановить или восстановить в предыдущее состояние, если исходные данные или приложения будут утеряны или повреждены. Это также важный компонент стратегии аварийного восстановления бизнеса.

Чтобы разработать комплексную стратегию резервного копирования и восстановления для платформы SAP Боби, найдите компоненты, которые приводят к простою системы или нарушению работы приложения. В платформе SAP Боби резервное копирование следующих компонентов крайне важно для защиты приложения.

- Каталог установки SAP Боби (управляемые диски класса Premium)
- Сервер репозитория файлов (Azure NetApp Files или файлы Azure Premium)
- База данных CMS (база данных Azure для MySQL или база данных на виртуальной машине Azure)

В следующем разделе описывается реализация стратегии резервного копирования и восстановления для каждого компонента на платформе SAP Боби.

#### <a name="backup--restore-for-sap-bobi-installation-directory"></a>Резервное копирование & восстановление для каталога установки SAP Боби

В Azure самым простым способом резервного копирования серверов приложений и всех подключенных дисков является использование службы [Azure Backup](../../../backup/backup-overview.md) . Она предоставляет независимые и изолированные резервные копии для защиты непреднамеренного уничтожения данных на виртуальных машинах. Резервные копии хранятся в хранилище Служб восстановления со встроенным управлением точками восстановления. Конфигурация и масштабирование просты, резервное копирование оптимизировано и может быть легко восстановлено при необходимости.

В рамках процесса резервного копирования создается моментальный снимок, а данные передаются в хранилище службы восстановления без влияния на рабочие нагрузки. Моментальный снимок обеспечивает разный уровень согласованности, как описано в статье о [согласованности моментальных снимков](../../../backup/backup-azure-vms-introduction.md#snapshot-consistency) . Вы также можете создать резервную копию подмножества дисков данных в виртуальной машине, используя функции резервного копирования и восстановления выборочных дисков. Дополнительные сведения см. в документе [резервное копирование](../../../backup/backup-azure-vms-introduction.md) виртуальных машин Azure и [часто задаваемые вопросы. Резервное копирование виртуальных машин Azure](../../../backup/backup-azure-vm-backup-faq.yml).

#### <a name="backup--restore-for-file-repository-server"></a>Резервное копирование & восстановления для сервера репозитория файлов

Для **Azure NetApp Files** можно создать моментальные снимки по запросу и запланировать автоматическое создание моментального снимка с помощью политик моментальных снимков. Копии моментальных снимков предоставляют копию тома использовании на момент времени. Дополнительные сведения см. в статье [Управление моментальными снимками с помощью Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-manage-snapshots.md).

Служба архивации **файлов Azure** интегрирована со службой машинного [Azure Backup](../../../backup/backup-overview.md) , которая централизует функцию резервного копирования и восстановления вместе с резервным копированием виртуальных машин и упрощает работу над операциями. Дополнительные сведения см. в статье [резервное копирование файловых ресурсов Azure](../../../backup/azure-file-share-backup-overview.md) и [часто задаваемые вопросы. Резервное копирование файлов Azure](../../../backup/backup-azure-files-faq.md).

#### <a name="backup--restore-for-cms-database"></a>Восстановление & резервного копирования для базы данных CMS

База данных Azure MySQL — это предложение DBaaS в Azure, которое автоматически создает резервные копии сервера и сохраняет их в настроенном пользователем локально избыточном или геоизбыточном хранилище. База данных Azure MySQL создает резервные копии файлов данных и журнала транзакций. В зависимости от поддерживаемого максимального размера хранилища он либо принимает полные, либо разностные резервные копии (максимум 4 ТБ серверов хранения) или резервное копирование моментальных снимков (до 16 ТБ серверов хранения). Эти резервные копии позволяют восстановить сервер в любой момент времени в течение заданного срока хранения резервной копии. Срок хранения резервных копий по умолчанию составляет семь дней, при [необходимости его можно настроить](../../../mysql/howto-restore-server-portal.md#set-backup-configuration) до трех дней. Все резервные копии шифруются с помощью 256-битового шифрования AES.

Эти файлы резервных копий не предоставляются пользователем и не могут быть экспортированы. Эти резервные копии можно использовать только для операций восстановления в базе данных Azure для MySQL. [Mysqldump](../../../mysql/concepts-migrate-dump-restore.md) можно использовать для копирования базы данных. Дополнительные сведения см. [в статье резервное копирование и восстановление в базе данных Azure для MySQL](../../../mysql/concepts-backup.md).

Для базы данных, установленной на виртуальных машинах, можно использовать стандартные средства резервного копирования или [Azure Backup](../../../backup/sap-hana-db-about.md) для базы данных Hana. Кроме того, если службы и средства Azure не соответствуют требованиям, вы можете использовать другие средства резервного копирования или скрипты для создания резервных копий дисков.

## <a name="sap-businessobjects-bi-platform-reliability"></a>Надежность платформы SAP BusinessObjects BI

Платформа SAP BusinessObjects BI включает различные уровни, оптимизированные для конкретных задач и операций. Когда компонент на любом из уровней становится недоступным, приложение SAP Боби становится недоступным или некоторые функциональные возможности приложения не будут работать. Необходимо убедиться, что каждый уровень является надежным для поддержания работоспособности приложений без каких-либо нарушений бизнеса.

В этом разделе рассматриваются следующие варианты для SAP Боби Platform-

- **Высокий уровень доступности:** У высокодоступной платформы есть по крайней мере две возможности в регионе Azure, чтобы обеспечить работоспособность приложения, если один из серверов становится недоступным.
- **Аварийное восстановление:** Это процесс восстановления функциональности приложения, если есть серьезные потери, такие как весь регион Azure становятся недоступными из-за неестественного сбоя.

Реализация этого решения зависит от характера настройки системы в Azure. Поэтому клиент должен адаптировать решение о высоком уровне доступности и аварийном восстановлении в зависимости от бизнес-требований.

### <a name="high-availability"></a>Высокий уровень доступности

Высокий уровень доступности относится к набору технологий, которые позволяют минимизировать количество нарушений ИТ, обеспечивая непрерывность бизнес-процессов приложений и служб благодаря избыточности, отказоустойчивости или компонентам, защищенным с помощью отработки отказа, в одном центре обработки данных. В нашем случае центры обработки данных находятся в одном регионе Azure. В статье [архитектура высокого уровня доступности и сценарии для SAP](sap-high-availability-architecture-scenarios.md) представляют собой начальные сведения о различных методах обеспечения высокого уровня доступности и рекомендации, предлагаемые в Azure для приложений SAP, которые дополняют инструкции, приведенные в этом разделе.

В зависимости от результата изменения размера платформы SAP Боби необходимо разработать альбомную ориентацию и определить распределение компонентов бизнес-аналитики между виртуальными машинами и подсетями Azure. Уровень избыточности в распределенной архитектуре зависит от требуемой для бизнеса цели времени восстановления (RTO) и целевой точки восстановления (RPO). Платформа SAP Боби включает различные уровни и компоненты на каждом уровне, предназначенные для обеспечения избыточности. Поэтому, если один из компонентов завершается неудачно, в приложении SAP Боби остается немало сбоев. Например,

- Избыточные серверы приложений, такие как серверы приложений BI и веб-сервер
- Уникальные компоненты, такие как база данных CMS, сервер репозитория файлов, Load Balancer

В следующем разделе описывается достижение высокого уровня доступности для каждого компонента платформы SAP Боби.

#### <a name="high-availability-for-application-servers"></a>Высокий уровень доступности для серверов приложений

Для служб бизнес-аналитики и серверов приложений, независимо от того, установлены ли они отдельно или вместе, не требуется определенное решение для обеспечения высокой доступности. Вы можете добиться высокого уровня доступности за счет избыточности, настроив несколько экземпляров бизнес-аналитики и серверов на различных виртуальных машинах Azure.

Чтобы снизить влияние простоя из-за одного или нескольких событий, рекомендуется следовать рекомендациям ниже высокого уровня доступности для серверов приложений, работающих на нескольких виртуальных машинах.

- Используйте Зоны доступности для защиты сбоев центров обработки данных.
- Настройте несколько виртуальных машин в группе доступности для обеспечения избыточности.
- Используйте управляемые диски для виртуальных машин в группе доступности.
- Настройте каждый уровень приложения в виде отдельных групп доступности.

Дополнительные сведения см. [в подокне Управление доступностью виртуальных машин Linux](../../availability.md) .

#### <a name="high-availability-for-cms-database"></a>Высокий уровень доступности для базы данных CMS

Если вы используете службу "база данных Azure как услуга" (DBaaS) для базы данных CMS, по умолчанию предоставляется платформа высокого уровня доступности. Вам нужно просто выбрать регион и обеспечить высокую доступность, избыточность и устойчивость служб без необходимости настройки дополнительных компонентов. Дополнительные сведения о соглашении об уровне обслуживания для поддерживаемого предложения DBaaS в Azure см. в этой службе, а также [в базе данных Azure для MySQL](../../../mysql/concepts-high-availability.md) и [высокой доступности для базы данных SQL Azure](../../../azure-sql/database/high-availability-sla.md) .

Дополнительные сведения о развертывании СУБД для базы данных CMS см. в [руководствах по развертыванию СУБД для рабочей нагрузки SAP](dbms_guide_general.md), что позволяет получить представление о различных развертываниях СУБД и его подходе к повышению уровня доступности.

#### <a name="high-availability-for-file-repository-server"></a>Высокий уровень доступности для сервера репозитория файлов

Сервер репозитория файлов (FRS) относится к каталогам на диске, где хранятся такие содержимое, как отчеты, Вселенной и соединения. Он является общим для всех серверов приложений этой системы. Поэтому необходимо убедиться, что он имеет высокий уровень доступности.

В Azure вы можете выбрать [файлы Azure Premium](../../../storage/files/storage-files-introduction.md) или [Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-introduction.md) для общей папки, которая обеспечивает высокую доступность и надежность. Дополнительные сведения см. в разделе [избыточность](../../../storage/files/storage-files-planning.md#redundancy) для службы файлов Azure.

> [!NOTE]
> Протокол SMB для службы "файлы Azure" общедоступен, но поддержка протокола NFS для файлов Azure сейчас доступна в предварительной версии. Дополнительные сведения см. [в разделе Поддержка NFS 4,1 для службы файлов Azure теперь доступна в предварительной версии](https://azure.microsoft.com/en-us/blog/nfs-41-support-for-azure-files-is-now-in-preview/) .

Так как эта служба файлового ресурса недоступна во всех регионах, убедитесь, что вы ссылаетесь на сайты, [Доступные по регионам](https://azure.microsoft.com/en-us/global-infrastructure/services/) , чтобы найти актуальные сведения. Если служба недоступна в вашем регионе, можно создать сервер NFS, на котором можно предоставить общий доступ к файловой системе для приложения SAP Боби. Но также необходимо учитывать высокий уровень доступности.

#### <a name="high-availability-for-load-balancer"></a>Высокий уровень доступности для балансировщика нагрузки

Для распределения трафика между веб-сервером можно использовать Azure Load Balancer или шлюз приложений Azure. Избыточность любой подсистемы балансировки нагрузки может быть достигнута на основе номера SKU, выбранного для развертывания.

- Для Azure Load Balancer избыточность может быть достигнута путем настройки Load Balancer (цен. категория "Стандартный") интерфейсной части в качестве избыточной в виде зоны. Дополнительные сведения см. в разделе [Load Balancer (цен. Категория "Стандартный") и зоны доступности](../../../load-balancer/load-balancer-standard-availability-zones.md)
- Для шлюза приложений высокая доступность может быть достигнута в зависимости от типа уровня, выбранного во время развертывания.
  - номер SKU v1 поддерживает сценарии с высоким уровнем доступности при развертывании двух или более экземпляров. Azure распределяет эти экземпляры по доменам сбоя и обновления, чтобы избежать одновременного сбоя всех экземпляров. Так что с этим номером SKU можно добиться избыточности в пределах зоны.
  - v2 SKU автоматически гарантирует, что новые экземпляры будут распределяться между доменами сбоя и доменами обновления. Если вы выбрали избыточность между зонами, последние экземпляры также распределяются по зонам доступности, что обеспечивает устойчивость зон к сбоям. Дополнительные сведения см. в статье [Автомасштабирование и избыточность в зонах — шлюз приложений версии 2](../../../application-gateway/application-gateway-autoscaling-zone-redundant.md) .

#### <a name="reference-high-availability-architecture-for-sap-businessobjects-bi-platform"></a>Эталонная архитектура высокого уровня доступности для платформы SAP BusinessObjects BI

В приведенной ниже эталонной архитектуре описывается настройка платформы SAP Боби с помощью группы доступности, которая обеспечивает избыточность и доступность виртуальных машин в зоне. Архитектура демонстрирует использование различных служб Azure, таких как шлюз приложений Azure, Azure NetApp Files и база данных Azure для MySQL для Боби платформы SAP, которая обеспечивает встроенную избыточность, что сокращает сложность управления различными решениями высокого уровня доступности.

На приведенном ниже рисунке входящий трафик (HTTPS-TCP/443) сбалансирован с балансировкой нагрузки с помощью SKU шлюза приложений Azure версии 1, который обеспечивает высокую доступность при развертывании на двух или более экземплярах. Несколько экземпляров веб-сервера, серверов управления и серверов обработки развертываются на отдельных виртуальных машинах для обеспечения избыточности, и каждый уровень развертывается в отдельных группах доступности. Azure NetApp Files имеет встроенную избыточность в центре обработки данных, поэтому тома использовании для сервера репозитория файлов будут высокодоступными. База данных CMS подготовлена в базе данных Azure для MySQL (DBaaS), которая имеет высокий уровень доступности. Дополнительные сведения см. в разделе о [высокой доступности в базе данных Azure для MySQL](../../../mysql/concepts-high-availability.md) .

![Избыточность платформы SAP BusinessObjects BI с использованием групп доступности](media/businessobjects-deployment-guide/businessobjects-deployment-high-availability.png)

Приведенная выше архитектура позволяет получить представление о том, как можно выполнить развертывание SAP Боби в Azure. Но он не охватывает все возможные варианты настройки для платформы SAP Боби в Azure. Клиент может адаптировать свое развертывание в зависимости от бизнес-требований, выбрав различные продукты и службы для различных компонентов, таких как Load Balancer, сервер репозитория файлов и СУБД.

В нескольких регионах Azure Зоны доступности предлагаются, что означает, что он имеет независимый источник питания, охлаждение и сеть. Она позволяет клиенту развертывать приложения в двух или трех зонах доступности. Для заказчиков, желающих обеспечить высокий уровень доступности в AZs, можно развернуть платформу SAP Боби в разных зонах доступности, убедившись в том, что каждый компонент в приложении является избыточным в пределах зоны.

### <a name="disaster-recovery"></a>Аварийное восстановление

Инструкция в этом разделе посвящена стратегии обеспечения защиты от аварийного восстановления для платформы SAP Боби. Он дополняет приложение [аварийного восстановления для документа SAP](../../../site-recovery/site-recovery-sap.md) , которое представляет основные ресурсы для общего подхода к аварийному восстановлению SAP.

#### <a name="reference-disaster-recovery-architecture-for-sap-businessobjects-bi-platform"></a>Эталонная архитектура аварийного восстановления для платформы SAP BusinessObjects BI

Эта Эталонная архитектура выполняет развертывание нескольких экземпляров платформы SAP Боби с избыточными серверами приложений. Для аварийного восстановления необходимо выполнить отработку отказа для всех уровней в дополнительный регион. Каждый уровень использует другую стратегию для обеспечения защиты от аварийного восстановления.

![Аварийное восстановление платформы SAP BusinessObjects BI](media/businessobjects-deployment-guide/businessobjects-deployment-disaster-recovery.png)

#### <a name="load-balancer"></a>Подсистема балансировки нагрузки

Load Balancer используется для распределения трафика между серверами веб-приложений платформы SAP Боби. Чтобы получить аварийное восстановление для шлюза приложений Azure, реализуйте параллельную настройку шлюза приложений в дополнительном регионе.

#### <a name="virtual-machines-running-web-and-bi-application-servers"></a>Виртуальные машины с серверами приложений бизнес-аналитики

Azure Site Recovery службу можно использовать для репликации виртуальных машин, на которых выполняются веб-приложения и серверы приложений BI в дополнительном регионе. Она реплицирует серверы в дополнительном регионе, чтобы при сбоях и сбоях можно было легко выполнить отработку отказа в реплицированную среду и продолжить работу.

#### <a name="file-repository-servers"></a>Серверы репозитория файлов

- **Azure NetApp Files** предоставляет тома NFS и SMB, поэтому для репликации данных между регионами Azure можно использовать любое средство копирования на основе файлов. Дополнительные сведения о копировании тома использовании в другом регионе см. в разделе [часто задаваемые вопросы о Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-faqs.md#how-do-i-create-a-copy-of-an-azure-netapp-files-volume-in-another-azure-region)

  Вы можете использовать Azure NetApp Files межрегионовую репликацию, которая в настоящее время находится на [этапе предварительной версии](https://azure.microsoft.com/en-us/blog/azure-netapp-files-cross-region-replication-and-new-enhancements-in-preview/) , использующей технологию NetApp SnapMirror®. Таким образом, только измененные блоки отправляются по сети в сжатом и эффективном формате. Эта собственная технология позволяет сократить объем данных, необходимых для репликации в регионах, что экономит расходы на передачу данных. Кроме того, сокращается время репликации, что позволяет достичь меньшей цели точки восстановления (RPO). Дополнительные сведения см. в разделе [требования и рекомендации по использованию репликации между регионами](../../../azure-netapp-files/cross-region-replication-requirements-considerations.md) .

- **Файлы Azure Premium** поддерживают только локально избыточное (LRS) и хранилище, избыточное в виде зоны (ZRS). Для стратегии аварийного восстановления файлов Azure Premium можно использовать [AzCopy](../../../storage/common/storage-use-azcopy-v10.md) или [Azure PowerShell](/powershell/module/az.storage/) для копирования файлов в другую учетную запись хранения в другом регионе. Дополнительные сведения см. в статье [Аварийное восстановление и отработка отказа учетной записи хранения](../../../storage/common/storage-disaster-recovery-guidance.md) .

#### <a name="cms-database"></a>База данных CMS

База данных Azure для MySQL предоставляет несколько вариантов восстановления базы данных в случае аварии. Выберите подходящий вариант, который подходит для вашего бизнеса.

- Включите реплики чтения между регионами, чтобы повысить непрерывность бизнес-процессов и планирование аварийного восстановления. Можно выполнить репликацию с исходного сервера до пяти реплик. Реплики чтения обновляются асинхронно с помощью технологии репликации двоичных журналов MySQL. Реплики — это новые серверы, которыми вы управляете как обычными серверами службы "База данных Azure для MySQL". Дополнительные сведения о репликах чтения, доступных регионах, ограничениях и отработки отказа см. в [статье основные понятия чтения реплик](../../../mysql/concepts-read-replicas.md).

- Используйте функцию геовосстановления базы данных Azure для MySQL, которая восстанавливает сервер с помощью геоизбыточных резервных копий. Эти резервные копии доступны, даже если регион, в котором размещен сервер, находится в автономном режиме. Из этих копий можно выполнить восстановление в любом другом регионе и восстановить подключение сервера.

  > [!NOTE]
  > Геовосстановление возможно только в том случае, если сервер был подготовлен с геоизбыточным хранилищем резервных копий. Изменение **параметров избыточности резервных копий** после создания сервера не поддерживается. Дополнительные сведения см. в статье [избыточность резервной копии](../../../mysql/concepts-backup.md#backup-redundancy-options) .

Ниже приведены рекомендации по аварийному восстановлению каждого уровня, используемого в этом примере.

| Уровни платформы SAP Боби   | Рекомендация                                                                                           |
|---------------------------|----------------------------------------------------------------------------------------------------------|
| Шлюз приложений Azure | Параллельная установка шлюза приложений в дополнительном регионе                                                |
| Серверы веб-приложений   | Репликация с помощью Site Recovery                                                                         |
| Серверы приложений бизнес-аналитики    | Репликация с помощью Site Recovery                                                                         |
| Azure NetApp Files        | Средство копирования на основе файлов для репликации данных в дополнительный регион **или** использовании кросс-региона (Предварительная версия) |
| База данных Azure для MySQL  | Межрегионовые реплики чтения **или** восстановление резервной копии из геоизбыточных резервных копий.                             |

## <a name="next-steps"></a>Дальнейшие действия

- [Настройка аварийного восстановления для многоуровневого развертывания приложений SAP](../../../site-recovery/site-recovery-sap.md)
- [SAP NetWeaver на виртуальных машинах Windows. Руководство по планированию и внедрению](planning-guide.md)
- [Развертывание виртуальных машин Azure для SAP NetWeaver](deployment-guide.md)
- [SAP NetWeaver на виртуальных машинах Azure. Руководство по развертыванию СУБД SQL Server](./dbms_guide_general.md)
