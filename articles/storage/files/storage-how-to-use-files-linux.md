---
title: Использование файлов Azure в Linux | Документы Майкрософт
description: Узнайте, как подключить файловый ресурс Azure через протокол SMB в Linux. См. список предварительных требований. Ознакомьтесь с вопросами безопасности SMB на клиентах Linux.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 10/19/2019
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 5161d8e169a7eb9e757dfbfa71fa697880e1806e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98673693"
---
# <a name="use-azure-files-with-linux"></a>Использование Файлов Azure в Linux
[Файлы Azure](storage-files-introduction.md) — это простая в использовании облачная файловая система от Майкрософт. Файловые ресурсы Azure можно подключить в дистрибутивах Linux с помощью [SMB-клиента в ядре](https://wiki.samba.org/index.php/LinuxCIFS). В этой статье описаны два способа подключения файлового ресурса Azure: по запросу с помощью команды `mount` и при загрузке путем создания записи в `/etc/fstab`.

Рекомендуемый способ подключения файлового ресурса Azure в Linux — использование SMB 3,0. По умолчанию для файлов Azure требуется шифрование при передаче, которое поддерживается только SMB 3,0. Служба файлов Azure также поддерживает протокол SMB 2,1, который не поддерживает шифрование при передаче, но вы не можете подключать файловые ресурсы Azure с SMB 2,1 из другого региона или локальной среды Azure по соображениям безопасности. Если приложению специально не требуется SMB 2,1, есть небольшая причина использовать его, так как большинство популярных, недавно выпущенных дистрибутивов Linux, поддерживают SMB 3,0:  

| Дистрибутив Linux | SMB 2.1 <br>(подключение к виртуальным машинам в одном регионе) | SMB 3.0 <br>(подключение из локальной среды и между регионами) |
| --- | :---: | :---: |
| Ubuntu | 14.04 или более поздней версии | 16.04+ |
| Red Hat Enterprise Linux (RHEL) | 7 или выше | 7.5 или выше |
| CentOS | 7 или выше |  7.5 или выше |
| Debian | 8+ | 10+ |
| openSUSE | 13.2 или выше | 42.3 или выше |
| SUSE Linux Enterprise Server | 12+ | 12 с пакетом обновления 2+ |

Если вы используете дистрибутив Linux, не указанный в приведенной выше таблице, можно проверить, поддерживает ли дистрибутив Linux протокол SMB 3,0 с шифрованием, проверив версию ядра Linux. SMB 3,0 с шифрованием было добавлено в ядро Linux версии 4,11. `uname`Команда возвратит используемую версию ядра Linux:

```bash
uname -r
```

## <a name="prerequisites"></a>Предварительные условия
<a id="smb-client-reqs"></a>

* <a id="install-cifs-utils"></a>**Убедитесь, что пакет CIFS-utils установлен.**  
    Пакет cifs-utils можно установить с помощью диспетчера пакетов в дистрибутиве Linux по своему усмотрению. 

    В дистрибутивах **Ubuntu** и **на основе Debian** используйте диспетчер пакетов `apt`:

    ```bash
    sudo apt update
    sudo apt install cifs-utils
    ```

    В **Fedora**, **Red Hat Enterprise Linux 8 +** и **CentOS 8 +**, используйте `dnf` Диспетчер пакетов:

    ```bash
    sudo dnf install cifs-utils
    ```

    В более старых версиях **Red Hat Enterprise Linux** и **CentOS** используйте `yum` Диспетчер пакетов:

    ```bash
    sudo yum install cifs-utils 
    ```

    В **openSUSE** используйте диспетчер пакетов `zypper`:

    ```bash
    sudo zypper install cifs-utils
    ```

    В других дистрибутивах используйте соответствующий диспетчер пакетов или выполните [компиляцию из источника](https://wiki.samba.org/index.php/LinuxCIFS_utils#Download).

* **Самая последняя версия интерфейса командной строки Azure (CLI).** Дополнительные сведения об установке Azure CLI см. в статьях [установка Azure CLI](/cli/azure/install-azure-cli) и выбор операционной системы. Если вы предпочитаете использовать модуль Azure PowerShell в PowerShell 6 +, вы можете, однако, приведенные ниже инструкции представлены для Azure CLI.

* **Откройте порт 445**. Взаимодействие SMB выполняется через TCP-порт 445. Проверьте, чтобы брандмауэр не блокировал TCP-порты 445 с клиентского компьютера.  Замените `<your-resource-group>` , а `<your-storage-account>` затем выполните следующий скрипт:
    ```bash
    resourceGroupName="<your-resource-group>"
    storageAccountName="<your-storage-account>"
    
    # This command assumes you have logged in with az login
    httpEndpoint=$(az storage account show \
        --resource-group $resourceGroupName \
        --name $storageAccountName \
        --query "primaryEndpoints.file" | tr -d '"')
    smbPath=$(echo $httpEndpoint | cut -c7-$(expr length $httpEndpoint))
    fileHost=$(echo $smbPath | tr -d "/")

    nc -zvw3 $fileHost 445
    ```

    Если подключение установлено успешно, вы увидите примерно следующее:

    ```ouput
    Connection to <your-storage-account> 445 port [tcp/microsoft-ds] succeeded!
    ```

    Если вы не можете открыть порт 445 в корпоративной сети или заблокировать его через поставщика услуг Интернета, вы можете использовать VPN-подключение или ExpressRoute для обхода порта 445. Дополнительные сведения см. в статье [рекомендации по сетям для прямого доступа к общей папке Azure](storage-files-networking-overview.md).

## <a name="mounting-azure-file-share"></a>Подключение файлового ресурса Azure
Чтобы использовать общую папку Azure с дистрибутивом Linux, необходимо создать каталог, который будет использоваться в качестве точки подключения для файлового ресурса Azure. Точку подключения можно создать в любом месте в системе Linux, но это распространенное соглашение о создании этой точки в разделе/mnt. После точки подключения используйте `mount` команду для доступа к файловому ресурсу Azure.

При необходимости можно подключить одну общую папку Azure к нескольким точкам подключения.

### <a name="mount-the-azure-file-share-on-demand-with-mount"></a>Подключение файлового ресурса Azure по запросу с помощью `mount`
1. **Создайте папку для точки подключения**: Replace `<your-resource-group>` , `<your-storage-account>` и `<your-file-share>` с соответствующей информацией о вашей среде:

    ```bash
    resourceGroupName="<your-resource-group>"
    storageAccountName="<your-storage-account>"
    fileShareName="<your-file-share>"

    mntPath="/mnt/$storageAccountName/$fileShareName"

    sudo mkdir -p $mntPath
    ```

1. **Используйте команду mount, чтобы подключить файловый ресурс Azure**. В примере ниже локальные разрешения для файлов и папок Linux по умолчанию 0755, что означает чтение, запись и выполнение для владельца (на основе владельца файлов и каталогов Linux), чтение и выполнение для пользователей в группе владелец, а также чтение и выполнение для других компонентов в системе. Можно использовать `uid` `gid` Параметры подключения и, чтобы задать идентификатор пользователя и группу для подключения. Кроме того, можно использовать `dir_mode` и `file_mode` для задания пользовательских разрешений. Дополнительные сведения о настройке разрешений см. в разделе [Цифровая нотация UNIX](https://en.wikipedia.org/wiki/File_system_permissions#Numeric_notation) в Википедии. 

    ```bash
    # This command assumes you have logged in with az login
    httpEndpoint=$(az storage account show \
        --resource-group $resourceGroupName \
        --name $storageAccountName \
        --query "primaryEndpoints.file" | tr -d '"')
    smbPath=$(echo $httpEndpoint | cut -c7-$(expr length $httpEndpoint))$fileShareName

    storageAccountKey=$(az storage account keys list \
        --resource-group $resourceGroupName \
        --account-name $storageAccountName \
        --query "[0].value" | tr -d '"')

    sudo mount -t cifs $smbPath $mntPath -o vers=3.0,username=$storageAccountName,password=$storageAccountKey,serverino
    ```

    > [!Note]  
    > Приведенная выше команда подключения подключена к SMB 3,0. Если дистрибутив Linux не поддерживает протокол SMB 3,0 с шифрованием или если он поддерживает только SMB 2,1, вы можете подключить его только к виртуальной машине Azure в том же регионе, что и учетная запись хранения. Чтобы подключить файловый ресурс Azure в дистрибутиве Linux, который не поддерживает SMB 3,0 с шифрованием, необходимо [отключить шифрование при передаче для учетной записи хранения](../common/storage-require-secure-transfer.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).

Завершив работу с файловым ресурсом Azure, вы можете отключить его, выполнив команду `sudo umount $mntPath`.

### <a name="create-a-persistent-mount-point-for-the-azure-file-share-with-etcfstab"></a>Создание постоянной точки подключения для файлового ресурса Azure с помощью `/etc/fstab`
1. **Создать папку для точки подключения**. папка для точки подключения может быть создана в любом месте файловой системы, но ее можно создать в разделе/mnt. Например, следующая команда создает новый каталог, заменяет `<your-resource-group>` , `<your-storage-account>` и `<your-file-share>` с соответствующей информацией для вашей среды:

    ```bash
    resourceGroupName="<your-resource-group>"
    storageAccountName="<your-storage-account>"
    fileShareName="<your-file-share>"

    mntPath="/mnt/$storageAccountName/$fileShareName"

    sudo mkdir -p $mntPath
    ```

1. **Создайте файл учетных данных, в котором будет храниться имя пользователя (имя учетной записи хранения) и пароль (ключ учетной записи хранения) для имени файлового ресурса.** 

    ```bash
    if [ ! -d "/etc/smbcredentials" ]; then
        sudo mkdir "/etc/smbcredentials"
    fi

    storageAccountKey=$(az storage account keys list \
        --resource-group $resourceGroupName \
        --account-name $storageAccountName \
        --query "[0].value" | tr -d '"')
    
    smbCredentialFile="/etc/smbcredentials/$storageAccountName.cred"
    if [ ! -f $smbCredentialFile ]; then
        echo "username=$storageAccountName" | sudo tee $smbCredentialFile > /dev/null
        echo "password=$storageAccountKey" | sudo tee -a $smbCredentialFile > /dev/null
    else 
        echo "The credential file $smbCredentialFile already exists, and was not modified."
    fi
    ```

1. **Измените разрешения для файла учетных данных таким образом, чтобы только пользователь с правами root мог читать или изменять файл пароля.** Поскольку ключ учетной записи хранения является паролем управляющего администратора учетной записи хранения, необходимо установить разрешения для файла, чтобы только у пользователя с правами root был доступ к файлу, а пользователи с более низким статусом прав не могли получить ключ учетной записи хранения.   

    ```bash
    sudo chmod 600 $smbCredentialFile
    ```

1. **Используйте следующую команду, чтобы `/etc/fstab` Добавить следующую строку в**: в примере ниже локальные разрешения для файлов и папок Linux по умолчанию 0755, что означает чтение, запись и выполнение для владельца (на основе владельца файлов и каталогов Linux), чтение и выполнение для пользователей в группе владелец, а также чтение и выполнение для других компонентов в системе. Можно использовать `uid` `gid` Параметры подключения и, чтобы задать идентификатор пользователя и группу для подключения. Кроме того, можно использовать `dir_mode` и `file_mode` для задания пользовательских разрешений. Дополнительные сведения о настройке разрешений см. в разделе [Цифровая нотация UNIX](https://en.wikipedia.org/wiki/File_system_permissions#Numeric_notation) в Википедии.

    ```bash
    # This command assumes you have logged in with az login
    httpEndpoint=$(az storage account show \
        --resource-group $resourceGroupName \
        --name $storageAccountName \
        --query "primaryEndpoints.file" | tr -d '"')
    smbPath=$(echo $httpEndpoint | cut -c7-$(expr length $httpEndpoint))$fileShareName

    if [ -z "$(grep $smbPath\ $mntPath /etc/fstab)" ]; then
        echo "$smbPath $mntPath cifs nofail,vers=3.0,credentials=$smbCredentialFile,serverino" | sudo tee -a /etc/fstab > /dev/null
    else
        echo "/etc/fstab was not modified to avoid conflicting entries as this Azure file share was already present. You may want to double check /etc/fstab to ensure the configuration is as desired."
    fi

    sudo mount -a
    ```
    
    > [!Note]  
    > Приведенная выше команда подключения подключена к SMB 3,0. Если дистрибутив Linux не поддерживает протокол SMB 3,0 с шифрованием или если он поддерживает только SMB 2,1, вы можете подключить его только к виртуальной машине Azure в том же регионе, что и учетная запись хранения. Чтобы подключить файловый ресурс Azure в дистрибутиве Linux, который не поддерживает SMB 3,0 с шифрованием, необходимо [отключить шифрование при передаче для учетной записи хранения](../common/storage-require-secure-transfer.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).

### <a name="using-autofs-to-automatically-mount-the-azure-file-shares"></a>Использование функции autofs для автоматического подключения общих файловых ресурсов Azure

1. **Убедитесь, что пакет autofs установлен.**  

    Пакет autofs можно установить с помощью диспетчера пакетов в дистрибутиве Linux по своему усмотрению. 

    В дистрибутивах **Ubuntu** и **на основе Debian** используйте диспетчер пакетов `apt`:
    ```bash
    sudo apt update
    sudo apt install autofs
    ```
    В **Fedora**, **Red Hat Enterprise Linux 8 +** и **CentOS 8 +**, используйте `dnf` Диспетчер пакетов:
    ```bash
    sudo dnf install autofs
    ```
    В более старых версиях **Red Hat Enterprise Linux** и **CentOS** используйте `yum` Диспетчер пакетов:
    ```bash
    sudo yum install autofs 
    ```
    В **openSUSE** используйте диспетчер пакетов `zypper`:
    ```bash
    sudo zypper install autofs
    ```
2. **Создайте точку подключения для общих папок**:
   ```bash
    sudo mkdir /fileshares
    ```
3. **Создавайте новый файл пользовательской конфигурации autofs**
    ```bash
    sudo vi /etc/auto.fileshares
    ```
4. **Добавьте следующие записи в/ЕТК/АУТО.филешарес.**
   ```bash
   echo "$fileShareName -fstype=cifs,credentials=$smbCredentialFile :$smbPath"" > /etc/auto.fileshares
   ```
5. **Добавьте следующую запись в/ЕТК/АУТО.мастер.**
   ```bash
   /fileshares /etc/auto.fileshares --timeout=60
   ```
6. **Перезапустить autofs**
    ```bash
    sudo systemctl restart autofs
    ```
7.  **Доступ к папке, указанной для общей папки**
    ```bash
    cd /fileshares/$filesharename
    ```
## <a name="securing-linux"></a>Обеспечение безопасности Linux.
Чтобы подключить общую папку Azure в Linux, необходимо иметь доступ к порту 445. Многие организации блокируют порт 445 из-за угроз безопасности, присущих SMB 1. SMB 1, также известный как CIFS (Общая файловая система Интернета), — это устаревший протокол файловой системы, входящий в состав многих дистрибутивов Linux. SMB 1 — это устаревший, неэффективный и, самое главное, небезопасный протокол. Хорошая новость в том, что служба файлов Azure не поддерживает SMB 1 и начиная с версии ядра Linux 4,18, Linux позволяет отключить SMB 1. Мы [настоятельно рекомендуем](https://aka.ms/stopusingsmb1) отключить SMB 1 на клиентах Linux перед использованием файловых ресурсов SMB в рабочей среде.

Начиная с Linux ядра 4,18, модуль ядра SMB, вызываемый `cifs` по старым причинам, предоставляет новый параметр модуля (часто называемый *ParM* различными внешними документами) `disable_legacy_dialects` . Несмотря на то, что впервые появились в ядре Linux 4,18, некоторые поставщики отменяли это изменение на более старые ядра, которые они поддерживают. Для удобства в следующей таблице подробно описывается доступность этого параметра модуля в распространенных дистрибутивах Linux.

| Distribution | Можно отключить SMB 1 |
|--------------|-------------------|
| Ubuntu 14.04 — 16.04 | Нет |
| Ubuntu 18.04 | Да |
| Ubuntu 19.04 + | Да |
| Debian 8-9 | Нет |
| Debian 10 + | Да |
| Fedora 29 + | Да |
| CentOS 7 | Нет | 
| CentOS 8 + | Да |
| Red Hat Enterprise Linux 6. x-7. x | Нет |
| Red Hat Enterprise Linux 8 + | Да |
| openSUSE LEAP 15,0 | Нет |
| openSUSE LEAP 15.1 + | Да |
| openSUSE Тумблевид | Да |
| SUSE Linux Enterprise 11. x-12. x | Нет |
| SUSE Linux Enterprise 15 | Нет |
| SUSE Linux Enterprise 15,1 | Нет |

Проверить, поддерживает ли дистрибутив Linux параметр Module, можно с `disable_legacy_dialects` помощью следующей команды.

```bash
sudo modinfo -p cifs | grep disable_legacy_dialects
```

Эта команда должна вывести следующее сообщение:

```output
disable_legacy_dialects: To improve security it may be helpful to restrict the ability to override the default dialects (SMB2.1, SMB3 and SMB3.02) on mount with old dialects (CIFS/SMB1 and SMB2) since vers=1.0 (CIFS/SMB1) and vers=2.0 are weaker and less secure. Default: n/N/0 (bool)
```

Перед отключением SMB 1 необходимо убедиться, что модуль SMB в данный момент не загружен в систему (это происходит автоматически при подключении общего ресурса SMB). Это можно сделать с помощью следующей команды, которая не должна выводить Nothing, если SMB не загружена:

```bash
lsmod | grep cifs
```

Чтобы выгрузить модуль, сначала отключите все общие ресурсы SMB (с помощью `umount` команды, описанной выше). Вы можете выделить все подключенные общие папки SMB в системе с помощью следующей команды:

```bash
mount | grep cifs
```

После отключения всех файловых ресурсов SMB можно выгрузить модуль. Это можно сделать с помощью команды `modprobe` .

```bash
sudo modprobe -r cifs
```

Модуль с SMB 1 можно загрузить вручную с помощью `modprobe` команды:

```bash
sudo modprobe cifs disable_legacy_dialects=Y
```

Наконец, можно проверить, загружен ли модуль SMB с параметром, просмотрев загруженные параметры в `/sys/module/cifs/parameters` :

```bash
cat /sys/module/cifs/parameters/disable_legacy_dialects
```

Чтобы постоянно отключить SMB 1 в Ubuntu и дистрибутивах на основе Debian, необходимо создать новый файл (если у вас еще нет пользовательских параметров для других модулей), вызванных `/etc/modprobe.d/local.conf` с параметром. Это можно сделать с помощью следующей команды:

```bash
echo "options cifs disable_legacy_dialects=Y" | sudo tee -a /etc/modprobe.d/local.conf > /dev/null
```

Вы можете убедиться, что это работало, загрузив модуль SMB:

```bash
sudo modprobe cifs
cat /sys/module/cifs/parameters/disable_legacy_dialects
```

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о службе файлов Azure см. по следующим ссылкам.

* [Планирование развертывания службы файлов Azure](storage-files-planning.md)
* [Часто задаваемые вопросы](./storage-files-faq.md)
* [Устранение неполадок](storage-troubleshoot-linux-file-connection-problems.md)