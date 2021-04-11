---
title: Настройка групп доступности для SQL Server на виртуальных машинах RHEL в Azure — виртуальные машины Linux | Документация Майкрософт
description: Сведения о настройке высокого уровня доступности в среде кластера RHEL и настройке STONITH
ms.service: virtual-machines-sql
ms.topic: tutorial
author: VanMSFT
ms.author: vanto
ms.reviewer: jroth
ms.date: 06/25/2020
ms.openlocfilehash: 889df16191104ca5b335d067abe0f3d1311c4603
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2021
ms.locfileid: "106450112"
---
# <a name="tutorial-configure-availability-groups-for-sql-server-on-rhel-virtual-machines-in-azure"></a>Руководство по Настройка групп доступности для SQL Server на виртуальных машинах RHEL в Azure 
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

> [!NOTE]
> В этом учебнике мы используем SQL Server 2017 с RHEL 7.6, но для настройки высокого уровня доступности можно использовать SQL Server 2019 в RHEL 7 или RHEL 8. Команды для настройки ресурсов кластера pacemake и группы доступности в RHEL 8 изменились. Дополнительные сведения о правильных командах вы можете получить в разделе [Создание ресурса группы доступности](/sql/linux/sql-server-linux-availability-group-cluster-rhel#create-availability-group-resource) и в информационных ресурсах по RHEL 8.

В этом руководстве описано следующее:

> [!div class="checklist"]
> - Создание новой группы ресурсов, группы доступности и виртуальных машин Linux
> - Включение высокого уровня доступности
> - Создание кластера Pacemaker
> - Настройка агента ограждения путем создания устройства STONITH
> - Установка SQL Server и средств mssql на RHEL
> - Настройка группы доступности Always On SQL Server
> - Настройка ресурсов группы доступности в кластере Pacemaker
> - Проверка отработки отказа и агента ограждения

В этом учебнике для развертывания ресурсов в Azure используется Azure CLI.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0.30 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

При наличии нескольких подписок [установите подписку](/cli/azure/manage-azure-subscriptions-azure-cli), для которой вы хотите развернуть эти ресурсы.

Выполните следующую команду, чтобы создать группу ресурсов `<resourceGroupName>` в регионе. Замените `<resourceGroupName>` именем по своему выбору. В этом руководстве мы используем `East US 2`. Дополнительные сведения см. [Краткое руководство. Создание виртуальной машины Linux с помощью Azure CLI](../../../application-gateway/quick-create-cli.md).

```azurecli-interactive
az group create --name <resourceGroupName> --location eastus2
```

## <a name="create-an-availability-set"></a>"Создать группу доступности"

Следующим шагом является создание группы доступности. Выполните следующую команду в Azure Cloud Shell и замените `<resourceGroupName>` именем группы ресурсов. Выберите имя для `<availabilitySetName>`.

```azurecli-interactive
az vm availability-set create \
    --resource-group <resourceGroupName> \
    --name <availabilitySetName> \
    --platform-fault-domain-count 2 \
    --platform-update-domain-count 2
```

После выполнения команды должны появиться следующие результаты:

```output
{
  "id": "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Compute/availabilitySets/<availabilitySetName>",
  "location": "eastus2",
  "name": "<availabilitySetName>",
  "platformFaultDomainCount": 2,
  "platformUpdateDomainCount": 2,
  "proximityPlacementGroup": null,
  "resourceGroup": "<resourceGroupName>",
  "sku": {
    "capacity": null,
    "name": "Aligned",
    "tier": null
  },
  "statuses": null,
  "tags": {},
  "type": "Microsoft.Compute/availabilitySets",
  "virtualMachines": []
}
```

## <a name="create-rhel-vms-inside-the-availability-set"></a>Создание виртуальных машин RHEL в группе доступности

> [!WARNING]
> При выборе образа RHEL с оплатой по мере использования (PAYG) и настройке высокого уровня доступности (HA) может понадобиться регистрация подписки. Это может привести к тому, что вы заплатите за подписку дважды, так как с вас будет взиматься плата за подписку RHEL в Microsoft Azure для виртуальных машин, а также за подписку на Red Hat. Дополнительные сведения см. в разделе https://access.redhat.com/solutions/2458541.
>
> Используйте образ RHEL с высоким уровнем доступности при создании виртуальной машины Azure, чтобы избежать "двойной оплаты". Образы, предлагаемые как образы RHEL с высоким уровнем доступности, являются также образами PAYG с предварительно включенным репозиторием с высоким уровнем доступности.

1. Получите список образов виртуальных машин, которые предлагают RHEL с высоким уровнем доступности:

    ```azurecli-interactive
    az vm image list --all --offer "RHEL-HA"
    ```

    Вы увидите следующие результаты:

    ```output
    [
      {
    "offer": "RHEL-HA",
    "publisher": "RedHat",
    "sku": "7.4",
    "urn": "RedHat:RHEL-HA:7.4:7.4.2019062021",
    "version": "7.4.2019062021"
       },
       {
    "offer": "RHEL-HA",
    "publisher": "RedHat",
    "sku": "7.5",
    "urn": "RedHat:RHEL-HA:7.5:7.5.2019062021",
    "version": "7.5.2019062021"
        },
        {
    "offer": "RHEL-HA",
    "publisher": "RedHat",
    "sku": "7.6",
    "urn": "RedHat:RHEL-HA:7.6:7.6.2019062019",
    "version": "7.6.2019062019"
         },
         {
    "offer": "RHEL-HA",
    "publisher": "RedHat",
    "sku": "8.0",
    "urn": "RedHat:RHEL-HA:8.0:8.0.2020021914",
    "version": "8.0.2020021914"
         },
         {
    "offer": "RHEL-HA",
    "publisher": "RedHat",
    "sku": "8.1",
    "urn": "RedHat:RHEL-HA:8.1:8.1.2020021914",
    "version": "8.1.2020021914"
          },
          {
    "offer": "RHEL-HA",
    "publisher": "RedHat",
    "sku": "80-gen2",
    "urn": "RedHat:RHEL-HA:80-gen2:8.0.2020021915",
    "version": "8.0.2020021915"
           },
           {
    "offer": "RHEL-HA",
    "publisher": "RedHat",
    "sku": "81_gen2",
    "urn": "RedHat:RHEL-HA:81_gen2:8.1.2020021915",
    "version": "8.1.2020021915"
           }
    ]
    ```

    Для работы с этим руководством мы выбираем в качестве примера образ `RedHat:RHEL-HA:7.6:7.6.2019062019` (для RHEL 7) или `RedHat:RHEL-HA:8.1:8.1.2020021914` (для RHEL 8).
    
    Можно также выбрать вариант с предварительно установленной ОС SQL Server 2019 в образах RHEL8-HA. Чтобы получить список этих образов, выполните следующую команду:  
    
    ```azurecli-interactive
    az vm image list --all --offer "sql2019-rhel8"
    ```

    Вы увидите следующие результаты:

    ```output
    [
      {
    "offer": "sql2019-rhel8",
    "publisher": "MicrosoftSQLServer",
    "sku": "enterprise",
    "urn": "MicrosoftSQLServer:sql2019-rhel8:enterprise:15.0.200317",
    "version": "15.0.200317"
       },
       }
    "offer": "sql2019-rhel8",
    "publisher": "MicrosoftSQLServer",
    "sku": "enterprise",
    "urn": "MicrosoftSQLServer:sql2019-rhel8:enterprise:15.0.200512",
    "version": "15.0.200512"
       },
       {
    "offer": "sql2019-rhel8",
    "publisher": "MicrosoftSQLServer",
    "sku": "sqldev",
    "urn": "MicrosoftSQLServer:sql2019-rhel8:sqldev:15.0.200317",
    "version": "15.0.200317"
       },
       {
    "offer": "sql2019-rhel8",
    "publisher": "MicrosoftSQLServer",
    "sku": "sqldev",
    "urn": "MicrosoftSQLServer:sql2019-rhel8:sqldev:15.0.200512",
    "version": "15.0.200512"
       },
       {
    "offer": "sql2019-rhel8",
    "publisher": "MicrosoftSQLServer",
    "sku": "standard",
    "urn": "MicrosoftSQLServer:sql2019-rhel8:standard:15.0.200317",
    "version": "15.0.200317"
       },
       {
    "offer": "sql2019-rhel8",
    "publisher": "MicrosoftSQLServer",
    "sku": "standard",
    "urn": "MicrosoftSQLServer:sql2019-rhel8:standard:15.0.200512",
    "version": "15.0.200512"
       }
    ]
    ```

    Если для создания виртуальных машин вы используете один из указанных выше образов, то в нем есть предварительно установленная версия SQL Server 2019. В этом случае пропустите раздел [Установка SQL Server и средств MSSQL](#install-sql-server-and-mssql-tools) далее в этой статье.
    
    
    > [!IMPORTANT]
    > Для настройки группы доступности имена компьютеров должны содержать менее 15 символов. Имя пользователя не может содержать прописных букв, а длина пароля должна составлять более 12 символов.

1. Нам нужно создать 3 виртуальные машины в группе доступности. Замените следующее в приведенной ниже команде:

    - `<resourceGroupName>`
    - `<VM-basename>`
    - `<availabilitySetName>`
    - `<VM-Size>` — пример "Standard_D16_v3"
    - `<username>`
    - `<adminPassword>`

    ```azurecli-interactive
    for i in `seq 1 3`; do
           az vm create \
             --resource-group <resourceGroupName> \
             --name <VM-basename>$i \
             --availability-set <availabilitySetName> \
             --size "<VM-Size>"  \
             --image "RedHat:RHEL-HA:7.6:7.6.2019062019" \
             --admin-username "<username>" \
             --admin-password "<adminPassword>" \
             --authentication-type all \
             --generate-ssh-keys
    done
    ```

Приведенная выше команда создает виртуальные машины и стандартную виртуальную сеть для них. Дополнительные сведения о различных конфигурациях см. в статье [az vm create](/cli/azure/vm) (Создание виртуальных машин Azure).

После завершения выполнения команды для каждой виртуальной машины вы должны получить результаты, аналогичные приведенным ниже.

```output
{
  "fqdns": "",
  "id": "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Compute/virtualMachines/<VM1>",
  "location": "eastus2",
  "macAddress": "<Some MAC address>",
  "powerState": "VM running",
  "privateIpAddress": "<IP1>",
  "publicIpAddress": "",
  "resourceGroup": "<resourceGroupName>",
  "zones": ""
}
```

> [!IMPORTANT]
> Стандартный образ, созданный с помощью указанной выше команды, создает стандартный диск ОС с 32 ГБ. При такой установке по умолчанию у вас может закончиться место. Чтобы создать диск ОС с 128 ГБ, в качестве примера можно использовать следующий параметр, добавленный в приведенную выше команду `az vm create`: `--os-disk-size-gb 128`.
>
> Затем вы можете [настроить Диспетчер логических дисков (LVM)](/previous-versions/azure/virtual-machines/linux/configure-lvm), если необходимо расширить соответствующие тома папки для размещения установки.

### <a name="test-connection-to-the-created-vms"></a>Проверка подключения к созданным виртуальным машинам

Подключитесь к VM1 или другим виртуальным машинам, используя следующую команду в Azure Cloud Shell. Если вы не можете найти IP-адрес виртуальной машины, следуйте указаниям в статье [Quickstart for Bash in Azure Cloud Shell](../../../cloud-shell/quickstart.md#ssh-into-your-linux-vm) (Краткое руководство по использованию Bash в Azure Cloud Shell).

```azurecli-interactive
ssh <username>@publicipaddress
```

Если установка прошла успешно, отобразятся следующие выходные данные представляющие терминал Linux:

```output
[<username>@<VM1> ~]$
```

Введите `exit` чтобы выйти из сеанса SSH.

## <a name="enable-high-availability"></a>Включение высокой доступности

> [!IMPORTANT]
> Для выполнения этой части руководства необходимо иметь подписку на RHEL и дополнительный компонент для высокого уровня доступности. Если вы используете образ, рекомендованный в предыдущем разделе, вам не нужно регистрировать другую подписку.
 
Подключитесь к каждому узлу виртуальной машины и следуйте руководству, приведенному ниже, чтобы включить высокий уровень доступности. Дополнительные сведения см. в разделе [Включение подписки высокой доступности для RHEL](/sql/linux/sql-server-linux-availability-group-cluster-rhel#enable-the-high-availability-subscription-for-rhel).

> [!TIP]
> Будет проще, если вы откроете сеанс SSH на каждой виртуальной машине одновременно, так как на протяжении всей статьи на каждой виртуальной машине должны выполняться одни и те же команды.
>
> Если вы копируете и вставляете несколько команд `sudo` и появляется запрос на введение пароля, дополнительные команды не будут выполняться. Выполните каждую команду по отдельности.


1. На каждой виртуальной машине выполните приведенные ниже команды, чтобы открыть порты брандмауэра Pacemaker.

    ```bash
    sudo firewall-cmd --permanent --add-service=high-availability
    sudo firewall-cmd --reload
    ```

1. Обновите и установите пакеты Pacemaker на всех узлах с помощью следующих команд:

    > [!NOTE]
    > **nmap** устанавливается как часть блока команд в качестве средства для поиска доступных IP-адресов в сети. Вы можете не устанавливать **nmap**, но оно пригодится в этом руководстве далее.

    ```bash
    sudo yum update -y
    sudo yum install -y pacemaker pcs fence-agents-all resource-agents fence-agents-azure-arm nmap
    sudo reboot
    ```

1. Задайте пароль для пользователя по умолчанию, который создается при установке пакетов Pacemaker. Используйте на всех узлах один и тот же пароль.

    ```bash
    sudo passwd hacluster
    ```

1. Используйте следующую команду, чтобы открыть файл "hosts" и настроить разрешение имен узлов. Дополнительные сведения по настройке файла "hosts", см. в статье [Настройка группы доступности Always On SQL Server для обеспечения высокой доступности в Linux](/sql/linux/sql-server-linux-availability-group-configure-ha#prerequisites).

    ```
    sudo vi /etc/hosts
    ```

    В редакторе **vi** введите `i`, чтобы вставить текст, а в пустую строку добавьте **частный IP-адрес** соответствующей виртуальной машины. Затем рядом с IP-адресом добавьте после пробела имя виртуальной машины. Каждая строка должна иметь отдельную запись.

    ```output
    <IP1> <VM1>
    <IP2> <VM2>
    <IP3> <VM3>
    ```

    > [!IMPORTANT]
    > Рекомендуется использовать **частный IP-адрес**, указанный выше. Использование общедоступного IP-адреса в этой конфигурации приведет к сбоям в настройке, и мы не рекомендуем подвергать вашу виртуальную машину воздействию внешних сетей.

    Чтобы выйти из редактора **vi**, сначала нажмите клавишу **Esc**, а затем введите команду `:wq` для записи файла и выхода.

## <a name="create-the-pacemaker-cluster"></a>Создание кластера Pacemaker

В этом разделе мы включим и запустим службу pcsd, а затем настроим кластер. Ресурсы кластера для SQL Server на Linux не создаются автоматически. Ресурсы Pacemaker необходимо включить и создать вручную. Дополнительные сведения см. в статье [Настройка экземпляра отказоустойчивого кластера RHEL для SQL Server](/sql/linux/sql-server-linux-shared-disk-cluster-red-hat-7-configure#install-and-configure-pacemaker-on-each-cluster-node)

### <a name="enable-and-start-pcsd-service-and-pacemaker"></a>Включение и запуск службы pcsd и Pacemaker

1. Выполните команды на всех узлах. Эти команды позволяют узлам повторно присоединяться к кластеру после перезагрузки.

    ```bash
    sudo systemctl enable pcsd
    sudo systemctl start pcsd
    sudo systemctl enable pacemaker
    ``` 

1. Удалите любые существующие конфигурации кластера со всех узлов. Выполните следующую команду:

    ```bash
    sudo pcs cluster destroy 
    sudo systemctl enable pacemaker 
    ```

1. Выполните следующие команды на первичном узле, чтобы настроить кластер.

    - Вам будет предложено ввести пароль при запуске `pcs cluster auth` команды для проверки подлинности узлов кластера. Введите пароль пользователя **hacluster**, созданный раньше.

    **RHEL 7**

    ```bash
    sudo pcs cluster auth <VM1> <VM2> <VM3> -u hacluster
    sudo pcs cluster setup --name az-hacluster <VM1> <VM2> <VM3> --token 30000
    sudo pcs cluster start --all
    sudo pcs cluster enable --all
    ```

    **RHEL8**

    Для RHEL 8 необходимо проверять подлинность узлов отдельно. При появлении запроса введите имя пользователя и пароль для **hacluster** вручную.

    ```bash
    sudo pcs host auth <node1> <node2> <node3>
    sudo pcs cluster setup <clusterName> <node1> <node2> <node3>
    sudo pcs cluster start --all
    sudo pcs cluster enable --all
    ```

1. Запустите следующую команду, чтобы убедиться, что все узлы подключены к сети.

    ```bash
    sudo pcs status
    ```

   **RHEL 7** 
   
    Если все узлы подключены, вы увидите результат, аналогичный приведенному ниже.

    ```output
    Cluster name: az-hacluster
     
    WARNINGS:
    No stonith devices and stonith-enabled is not false
     
    Stack: corosync
    Current DC: <VM2> (version 1.1.19-8.el7_6.5-c3c624ea3d) - partition with quorum
    Last updated: Fri Aug 23 18:27:57 2019
    Last change: Fri Aug 23 18:27:56 2019 by hacluster via crmd on <VM2>
     
    3 nodes configured
    0 resources configured
     
    Online: [ <VM1> <VM2> <VM3> ]
     
    No resources
     
     
    Daemon Status:
          corosync: active/enabled
          pacemaker: active/enabled
          pcsd: active/enabled
    ```
   
   **RHEL 8** 
   
    ```output
    Cluster name: az-hacluster
     
    WARNINGS:
    No stonith devices and stonith-enabled is not false
     
    Cluster Summary:
    * Stack: corosync
    * Current DC: <VM2> (version 1.1.19-8.el7_6.5-c3c624ea3d) - partition with quorum
    * Last updated: Fri Aug 23 18:27:57 2019
    * Last change: Fri Aug 23 18:27:56 2019 by hacluster via crmd on <VM2>
    * 3 nodes configured
    * 0 resource instances configured
     
   Node List:
    * Online: [ <VM1> <VM2> <VM3> ]
   
   Full List of Resources:
   * No resources
     
   Daemon Status:
          corosync: active/enabled
          pacemaker: active/enabled
          pcsd: active/enabled
    
    ```
    
1. Установите для ожидаемых голосов в динамическом кластере значение "3". Эта команда влияет только на динамический кластер и не изменяет файлы конфигурации.

    На всех узлах установите ожидаемые голоса с помощью следующей команды:

    ```bash
    sudo pcs quorum expected-votes 3
    ```

## <a name="configure-the-fencing-agent"></a>Настройка агента ограждения

Устройство STONITH предоставляет агент ограждения. Приведенные ниже инструкции изменены для данного руководства. Дополнительные сведения см. в разделе [Создание устройства STONITH](../../../virtual-machines/workloads/sap/high-availability-guide-rhel-pacemaker.md#create-stonith-device).
 
[Проверьте версию агента ограждения Azure, чтобы убедиться, что она обновлена](../../../virtual-machines/workloads/sap/high-availability-guide-rhel-pacemaker.md#cluster-installation). Используйте следующую команду:

```bash
sudo yum info fence-agents-azure-arm
```

В приведенном ниже примере вы увидите похожие выходные данные.

```output
Loaded plugins: langpacks, product-id, search-disabled-repos, subscription-manager
Installed Packages
Name        : fence-agents-azure-arm
Arch        : x86_64
Version     : 4.2.1
Release     : 11.el7_6.8
Size        : 28 k
Repo        : installed
From repo   : rhel-ha-for-rhel-7-server-eus-rhui-rpms
Summary     : Fence agent for Azure Resource Manager
URL         : https://github.com/ClusterLabs/fence-agents
License     : GPLv2+ and LGPLv2+
Description : The fence-agents-azure-arm package contains a fence agent for Azure instances.
```

### <a name="register-a-new-application-in-azure-active-directory"></a>Зарегистрируйте новое приложение в Azure Active Directory
 
 1. Перейдите на сайт https://portal.azure.com.
 2. Откройте колонку [Azure Active Directory](https://ms.portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Properties). Перейдите в колонку "Свойства" и запишите идентификатор каталога. В этом `tenant ID`
 3. Щелкните [**Регистрация приложений**](https://ms.portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade)
 4. Щелкните **Новая регистрация**
 5. Введите **имя**, например `<resourceGroupName>-app`, выберите **Accounts in this organization directory only** (Учетные записи только из этого каталога организации)
 6. Выберите тип приложения **Веб**, введите URL-адрес входа (например, http://localhost) ) и нажмите кнопку "Добавить". URL-адрес входа не используется и может быть любым допустимым URL-адресом. После этого щелкните **Зарегистрировать**.
 7. Выберите **Сертификаты и секреты**, а затем щелкните **Новый секрет клиента**.
 8. Введите описание нового ключа (секрета клиента), выберите **Срок действия не ограничен** и нажмите кнопку **Добавить**
 9. Запишите значение секрета. Он используется в качестве пароля субъекта-службы.
10. Щелкните **Обзор**. Запишите идентификатор приложения. Он используется в качестве имени пользователя (идентификатор входа в следующих шагах) субъекта-службы
 
### <a name="create-a-custom-role-for-the-fence-agent"></a>Создание пользовательской роли для агента ограждения

Следуйте инструкциям из руководства по [созданию настраиваемой Azure с помощью Azure CLI](../../../role-based-access-control/tutorial-custom-role-cli.md#create-a-custom-role).

Файл json должен выглядеть следующим образом:

- Замените `<username>` именем по своему выбору. Это позволяет избежать дублирования при создании определения роли.
- Замените `<subscriptionId>` идентификатором своей подписки Azure.

```json
{
  "Name": "Linux Fence Agent Role-<username>",
  "Id": null,
  "IsCustom": true,
  "Description": "Allows to power-off and start virtual machines",
  "Actions": [
    "Microsoft.Compute/*/read",
    "Microsoft.Compute/virtualMachines/powerOff/action",
    "Microsoft.Compute/virtualMachines/start/action"
  ],
  "NotActions": [
  ],
  "AssignableScopes": [
    "/subscriptions/<subscriptionId>"
  ]
}
```

Чтобы добавить роль, выполните следующую команду:

- Замените `<filename>` на имя файла.
- Если команда выполняется по пути, отличному от пути папки, в которую сохранен файл — включите путь к папке файла в команду.

```bash
az role definition create --role-definition "<filename>.json"
```

Вы должны увидеть следующий результат.

```output
{
  "assignableScopes": [
    "/subscriptions/<subscriptionId>"
  ],
  "description": "Allows to power-off and start virtual machines",
  "id": "/subscriptions/<subscriptionId>/providers/Microsoft.Authorization/roleDefinitions/<roleNameId>",
  "name": "<roleNameId>",
  "permissions": [
    {
      "actions": [
        "Microsoft.Compute/*/read",
        "Microsoft.Compute/virtualMachines/powerOff/action",
        "Microsoft.Compute/virtualMachines/start/action"
      ],
      "dataActions": [],
      "notActions": [],
      "notDataActions": []
    }
  ],
  "roleName": "Linux Fence Agent Role-<username>",
  "roleType": "CustomRole",
  "type": "Microsoft.Authorization/roleDefinitions"
}
```

### <a name="assign-the-custom-role-to-the-service-principal"></a>Назначение пользовательской роли субъекту-службе

Назначьте субъекту-службе пользовательскую роль `Linux Fence Agent Role-<username>` созданную в последнем шаге. Больше не используйте роль владельца!
 
1. Перейдите на сайт https://portal.azure.com.
2. Откройте колонку [Все ресурсы](https://ms.portal.azure.com/#blade/HubsExtension/BrowseAll)
3. Выберите виртуальную машину первого узла кластера.
4. Щелкните **Управление доступом (IAM)**
5. Щелкните **Добавить назначение ролей**
6. Выберите роль `Linux Fence Agent Role-<username>` из списка **Роль**
7. В списке **Выбор** введите имя созданного ранее приложения `<resourceGroupName>-app`
8. Щелкните **Сохранить**.
9. Повторите предыдущие шаги для всех узлов кластера.

### <a name="create-the-stonith-devices"></a>Создание устройств STONITH

Выполните следующие команды на узле 1:

- Замените `<ApplicationID>` значением идентификатора из регистрации приложения.
- Замените `<servicePrincipalPassword>` значением из секрета клиента.
- Замените `<resourceGroupName>` группой ресурсов из подписки, используемой в этом учебнике.
- Замените `<tenantID>` и `<subscriptionId>` из подписки Azure.

```bash
sudo pcs property set stonith-timeout=900
sudo pcs stonith create rsc_st_azure fence_azure_arm login="<ApplicationID>" passwd="<servicePrincipalPassword>" resourceGroup="<resourceGroupName>" tenantId="<tenantID>" subscriptionId="<subscriptionId>" power_timeout=240 pcmk_reboot_timeout=900
```

Поскольку мы уже добавили правило к брандмауэру, чтобы разрешить службу HA (`--add-service=high-availability`), вам не нужно открывать следующие порты брандмауэра на всех узлах: 2224, 3121, 21064, 5405. Однако, если вы испытываете какие-либо проблемы с подключением с HA, используйте следующую команду, чтобы открыть порты, связанные с HA.

> [!TIP]
> Опционально вы можете добавить все порты в этом учебнике сразу, чтобы сэкономить время. В соответствующих разделах ниже объясняются порты, которые нужно открыть. Если вы хотите добавить все порты сейчас, добавьте дополнительные порты: 1433 и 5022.

```bash
sudo firewall-cmd --zone=public --add-port=2224/tcp --add-port=3121/tcp --add-port=21064/tcp --add-port=5405/tcp --permanent
sudo firewall-cmd --reload
```

## <a name="install-sql-server-and-mssql-tools"></a>Установка SQL Server и средств mssql

> [!NOTE]
> Если вы создали виртуальные машины с предварительно установленной версией SQL Server 2019 (из образа RHEL8-HA), можете пропустить описанные ниже шаги по установке SQL Server и средств MSSQL и сразу перейти к разделу **Настройка группы доступности**, когда настроите пароль SA на всех виртуальных машинах, выполнив на них команду `sudo /opt/mssql/bin/mssql-conf set-sa-password`.

Используйте раздел ниже, чтобы установить на виртуальных машинах SQL Server и средства mssql. Вы можете выбрать любой из приведенных ниже примеров для установки SQL Server 2017 на RHEL 7 или SQL Server 2019 на RHEL 8. Выполните каждое из описанных действий на всех узлах. Дополнительные сведения см. в разделе [Установка SQL Server на виртуальной машине Red Hat](/sql/linux/quickstart-install-connect-red-hat).


### <a name="installing-sql-server-on-the-vms"></a>Установка SQL Server на виртуальных машинах

Для установки SQL Server используются следующие команды:

**RHEL 7 с SQL Server 2017** 

```bash
sudo curl -o /etc/yum.repos.d/mssql-server.repo https://packages.microsoft.com/config/rhel/7/mssql-server-2017.repo
sudo yum install -y mssql-server
sudo /opt/mssql/bin/mssql-conf setup
sudo yum install mssql-server-ha
```

**RHEL 8 с SQL Server 2019** 

```bash
sudo curl -o /etc/yum.repos.d/mssql-server.repo https://packages.microsoft.com/config/rhel/8/mssql-server-2019.repo
sudo yum install -y mssql-server
sudo /opt/mssql/bin/mssql-conf setup
sudo yum install mssql-server-ha
```
### <a name="open-firewall-port-1433-for-remote-connections"></a>Открытие порта 1433 брандмауэра для удаленных подключений

Для удаленного подключения на виртуальной машине необходимо открыть порт 1433. Используйте следующие команды, чтобы открыть порт 1433 в брандмауэре каждой виртуальной машины:

```bash
sudo firewall-cmd --zone=public --add-port=1433/tcp --permanent
sudo firewall-cmd --reload
```

### <a name="installing-sql-server-command-line-tools"></a>Установка программ командной строки SQL Server

Для установки программ командной строки SQL Server используются следующие команды. Дополнительные сведения см. в разделе [Установка программ командной строки SQL Server](/sql/linux/quickstart-install-connect-red-hat#tools).

**RHEL 7** 

```bash
sudo curl -o /etc/yum.repos.d/msprod.repo https://packages.microsoft.com/config/rhel/7/prod.repo
sudo yum install -y mssql-tools unixODBC-devel
```

**RHEL 8** 

```bash
sudo curl -o /etc/yum.repos.d/msprod.repo https://packages.microsoft.com/config/rhel/8/prod.repo
sudo yum install -y mssql-tools unixODBC-devel
```
 
> [!NOTE] 
> Для удобства добавьте /opt/mssql-tools/bin/ к переменной среды PATH. Это позволит запускать программы, не указывая полный путь. Выполните следующие команды, чтобы изменить переменную среды PATH как для сеансов входа в систему, так и для интерактивных сеансов и сеансов без входа в систему.</br></br>
`echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bash_profile`</br>
`echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc`</br>
`source ~/.bashrc`


### <a name="check-the-status-of-the-sql-server"></a>Проверка состояния SQL Server.

Закончив настройку, вы можете проверить состояние SQL Server и убедиться, что он работает:

```bash
systemctl status mssql-server --no-pager
```

Вы должны увидеть следующий результат.

```output
● mssql-server.service - Microsoft SQL Server Database Engine
   Loaded: loaded (/usr/lib/systemd/system/mssql-server.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2019-12-05 17:30:55 UTC; 20min ago
     Docs: https://docs.microsoft.com/en-us/sql/linux
 Main PID: 11612 (sqlservr)
   CGroup: /system.slice/mssql-server.service
           ├─11612 /opt/mssql/bin/sqlservr
           └─11640 /opt/mssql/bin/sqlservr
```

## <a name="configure-an-availability-group"></a>Настройка группы доступности

Используйте следующие шаги для настройки группы доступности Always On SQL Server для виртуальных машин. Дополнительные сведения см. в разделе [Настройка группы доступности Always On SQL Server для обеспечения высокого уровня доступности в Linux](/sql/linux/sql-server-linux-availability-group-configure-ha)

### <a name="enable-always-on-availability-groups-and-restart-mssql-server"></a>Включение групп доступности AlwaysOn и перезапуск mssql-server

Включите группы доступности Always On на каждом узле с экземпляром SQL Server. Затем перезапустите mssql-server. Выполните следующий скрипт:

```
sudo /opt/mssql/bin/mssql-conf set hadr.hadrenabled 1
sudo systemctl restart mssql-server
```

### <a name="create-a-certificate"></a>Создание сертификата

В настоящее время мы не поддерживаем аутентификацию домена приложения на конечной точке группы доступности. Поэтому для шифрования конечной точки группы доступности необходимо использовать сертификат.

1. Подключитесь ко **всем узлам** с помощью SQL Server Management Studio (SSMS) или SQL CMD. Чтобы включить сеанс AlwaysOn_health и создать главный ключ, выполните следующие команды:

    > [!IMPORTANT]
    > при удаленном подключении к экземпляру SQL Server в брандмауэре необходимо открыть порт 1433. Кроме того, для каждой виртуальной машины в группе безопасности сети необходимо разрешить входящие соединение к порту 1433. Дополнительные сведения о создании правила безопасности для входящего трафика см. в разделе [Create a security rule](../../../virtual-network/manage-network-security-group.md#create-a-security-rule) (Создание правила безопасности).

    - Замените `<Master_Key_Password>` собственным паролем.


    ```sql
    ALTER EVENT SESSION  AlwaysOn_health ON SERVER WITH (STARTUP_STATE=ON);
    GO
    CREATE MASTER KEY ENCRYPTION BY PASSWORD = '<Master_Key_Password>';
    ```

 
1. Подключитесь к первичной реплике с помощью SSMS или SQL CMD. Приведенные ниже команды создадут сертификат в `/var/opt/mssql/data/dbm_certificate.cer` и закрытый ключ в `var/opt/mssql/data/dbm_certificate.pvk` на первичной реплике SQL Server:

    - Замените `<Private_Key_Password>` собственным паролем.
    
    ```sql
    CREATE CERTIFICATE dbm_certificate WITH SUBJECT = 'dbm';
    GO
    
    BACKUP CERTIFICATE dbm_certificate
       TO FILE = '/var/opt/mssql/data/dbm_certificate.cer'
       WITH PRIVATE KEY (
               FILE = '/var/opt/mssql/data/dbm_certificate.pvk',
               ENCRYPTION BY PASSWORD = '<Private_Key_Password>'
           );
    GO
    ```

Выйдите из сеанса SQL CMD, выполнив команду `exit` и вернитесь в сеанс SSH.
 
### <a name="copy-the-certificate-to-the-secondary-replicas-and-create-the-certificates-on-the-server"></a>Копирование сертификата во вторичные реплики и создание сертификатов на сервере

1. Скопируйте два созданные файлы в одно и то же место на всех серверах, где будут размещаться реплики доступности.
 
    Выполните следующую команду `scp` на сервере-источнике, чтобы скопировать сертификат на целевые серверы:

    - Замените `<username>` и `<VM2>` на имена пользователя и целевой виртуальной машины, которые вы используете.
    - Выполните эту команду для всех вторичных реплик.

    > [!NOTE]
    > Вам не нужно запускать `sudo -i`, который предоставляет корневую среду. Вы можете просто запустить команду `sudo` перед каждой командой, как мы делали это ранее в этом учебнике.

    ```bash
    # The below command allows you to run commands in the root environment
    sudo -i
    ```

    ```bash
    scp /var/opt/mssql/data/dbm_certificate.* <username>@<VM2>:/home/<username>
    ```

1. Выполните следующую команду на целевом сервере:

    - Замените `<username>` именем пользователя.
    - Команда `mv` перемещает файлы или каталоги из одного места в другое.
    - Команда `chown` используется для изменения владельца и группы файлов, каталогов или ссылок.
    - Выполните эти команды для всех вторичных реплик.

    ```bash
    sudo -i
    mv /home/<username>/dbm_certificate.* /var/opt/mssql/data/
    cd /var/opt/mssql/data
    chown mssql:mssql dbm_certificate.*
    ```

1. Следующий сценарий Transact-SQL создает сертификат из резервной копии, созданной в первичной реплике SQL Server. Обновите сценарий, задав надежные пароли. Для расшифровки используется тот же пароль, что и при создании PVK-файла в предыдущем шаге. Выполните следующий скрипт используя SQL CMD или SSMS на всех серверах-получателях, чтобы создать сертификат:

    ```sql
    CREATE CERTIFICATE dbm_certificate
        FROM FILE = '/var/opt/mssql/data/dbm_certificate.cer'
        WITH PRIVATE KEY (
        FILE = '/var/opt/mssql/data/dbm_certificate.pvk',
        DECRYPTION BY PASSWORD = '<Private_Key_Password>'
                );
    GO
    ```

### <a name="create-the-database-mirroring-endpoints-on-all-replicas"></a>Создайте конечные точки на всех репликах зеркального отображения базы данных

Выполните следующий скрипт для всех экземпляров SQL Server с помощью SQL CMD или SSMS:

```sql
CREATE ENDPOINT [Hadr_endpoint]
    AS TCP (LISTENER_PORT = 5022)
    FOR DATABASE_MIRRORING (
    ROLE = ALL,
    AUTHENTICATION = CERTIFICATE dbm_certificate,
ENCRYPTION = REQUIRED ALGORITHM AES
);
GO

ALTER ENDPOINT [Hadr_endpoint] STATE = STARTED;
GO
```

### <a name="create-the-availability-group"></a>Создание группы доступности

Подключитесь к экземпляру SQL Server, на котором находится первичная реплика, используя SQL CMD или SSMS. Выполните следующую команду, чтобы создать группу доступности:

- Замените `ag1` на желаемое имя Группы доступности.
- Замените значения `<VM1>`, `<VM2>`, и `<VM3>` на имена экземпляров SQL Server, где размещаются реплики.

```sql
CREATE AVAILABILITY GROUP [ag1]
     WITH (DB_FAILOVER = ON, CLUSTER_TYPE = EXTERNAL)
     FOR REPLICA ON
         N'<VM1>'
          WITH (
             ENDPOINT_URL = N'tcp://<VM1>:5022',
             AVAILABILITY_MODE = SYNCHRONOUS_COMMIT,
             FAILOVER_MODE = EXTERNAL,
             SEEDING_MODE = AUTOMATIC
             ),
         N'<VM2>'
          WITH (
             ENDPOINT_URL = N'tcp://<VM2>:5022',
             AVAILABILITY_MODE = SYNCHRONOUS_COMMIT,
             FAILOVER_MODE = EXTERNAL,
             SEEDING_MODE = AUTOMATIC
             ),
         N'<VM3>'
         WITH(
            ENDPOINT_URL = N'tcp://<VM3>:5022',
            AVAILABILITY_MODE = SYNCHRONOUS_COMMIT,
            FAILOVER_MODE = EXTERNAL,
            SEEDING_MODE = AUTOMATIC
            );
GO

ALTER AVAILABILITY GROUP [ag1] GRANT CREATE ANY DATABASE;
GO
```

### <a name="create-a-sql-server-login-for-pacemaker"></a>Создание учетных данных SQL Server для Pacemaker

На всех экземплярах SQL Server создайте учетные данные SQL Server для Pacemaker. Следующий Transact-SQL создает имя для входа.

- Замените `<password>` собственным надежным паролем.

```sql
USE [master]
GO

CREATE LOGIN [pacemakerLogin] with PASSWORD= N'<password>';
GO

ALTER SERVER ROLE [sysadmin] ADD MEMBER [pacemakerLogin];
GO
```

Сохраните учетные данные, используемые для учетных данных SQL Server, на всех экземплярах SQL Server. 

1. Создание файла:

    ```bash
    sudo vi /var/opt/mssql/secrets/passwd
    ```

1. Добавьте следующие 2 строки в файл:

    ```bash
    pacemakerLogin
    <password>
    ```

    Чтобы выйти из редактора **vi**, сначала нажмите клавишу **Esc**, а затем введите команду `:wq` для записи файла и выхода.

1. Сделайте файл доступным для чтения только по корневому каталогу:

    ```bash
    sudo chown root:root /var/opt/mssql/secrets/passwd
    sudo chmod 400 /var/opt/mssql/secrets/passwd
    ```

### <a name="join-secondary-replicas-to-the-availability-group"></a>Присоединение вторичных реплик к группе доступности

1. Вам нужно открыть порт 5022 на брандмауэре для всех серверов, чтобы присоединить вторичные реплики к группе доступности. Выполните следующую команду в сеансе SSH:

    ```bash
    sudo firewall-cmd --zone=public --add-port=5022/tcp --permanent
    sudo firewall-cmd --reload
    ```

1. Выполните следующие команды на вторичных репликах, чтобы присоединить их к группе доступности:

    ```sql
    ALTER AVAILABILITY GROUP [ag1] JOIN WITH (CLUSTER_TYPE = EXTERNAL);
    GO

    ALTER AVAILABILITY GROUP [ag1] GRANT CREATE ANY DATABASE;
    GO
    ```

1. Выполните следующий сценарий Transact-SQL на первичной реплике и каждой вторичной реплике:

    ```sql
    GRANT ALTER, CONTROL, VIEW DEFINITION ON AVAILABILITY GROUP::ag1 TO pacemakerLogin;
    GO
    
    GRANT VIEW SERVER STATE TO pacemakerLogin;
    GO
    ```

1. После объединения вторичных реплик их можно увидеть в обозревателе объектов SSMS, развернув узел **высокого уровня доступности Always On**:

    ![Снимок экрана: первичная и вторичные реплики доступности](./media/rhel-high-availability-stonith-tutorial/availability-group-joined.png)

### <a name="add-a-database-to-the-availability-group"></a>Добавление базы данных в группу доступности

Мы будем следовать разделу [Добавление базы данных в группу доступности](/sql/linux/sql-server-linux-availability-group-configure-ha#add-a-database-to-the-availability-group).

В рамках этого шага используются приведенные ниже команды Transact-SQL. Выполните эти команды в первичной реплике:

```sql
CREATE DATABASE [db1]; -- creates a database named db1
GO

ALTER DATABASE [db1] SET RECOVERY FULL; -- set the database in full recovery mode
GO

BACKUP DATABASE [db1] -- backs up the database to disk
   TO DISK = N'/var/opt/mssql/data/db1.bak';
GO

ALTER AVAILABILITY GROUP [ag1] ADD DATABASE [db1]; -- adds the database db1 to the AG
GO
```

### <a name="verify-that-the-database-is-created-on-the-secondary-servers"></a>Убедитесь, что база данных создана на вторичных серверах.

На каждой вторичной реплике SQL Server выполните следующий запрос, чтобы убедиться, что база данных "db1" создана и находится в состоянии "СИНХРОНИЗОВАНО":

```
SELECT * FROM sys.databases WHERE name = 'db1';
GO
SELECT DB_NAME(database_id) AS 'database', synchronization_state_desc FROM sys.dm_hadr_database_replica_states;
```

Если в `synchronization_state_desc` указано "СИНХРОНИЗИРОВАНО" для `db1`, это означает, что реплики синхронизированы. Получатели показывают `db1` в первичной реплике.

## <a name="create-availability-group-resources-in-the-pacemaker-cluster"></a>Создание ресурсов группы доступности в кластере Pacemaker

Мы будем следовать руководству по [созданию ресурсов группы доступности в кластере Pacemaker](/sql/linux/sql-server-linux-create-availability-group#create-the-availability-group-resources-in-the-pacemaker-cluster-external-only).

> [!NOTE]
> Эта статья содержит ссылки на термин slave (ведомый). Корпорация Майкрософт больше не использует его. Когда этот термин будет удален из программного обеспечения, мы удалим его из статьи.

### <a name="create-the-ag-cluster-resource"></a>Создание кластерного ресурса группы доступности

1. Используйте одну из следующих команд в зависимости от выбранной ранее среды, чтобы создать ресурс `ag_cluster` в группе доступности `ag1`.

    **RHEL 7**
  
    ```bash
    sudo pcs resource create ag_cluster ocf:mssql:ag ag_name=ag1 meta failure-timeout=30s master notify=true
    ```

    **RHEL 8**
  
    ```bash
    sudo pcs resource create ag_cluster ocf:mssql:ag ag_name=ag1 meta failure-timeout=30s promotable notify=true
    ```

2. Проверьте ресурсы и убедитесь, что они подключены к сети, прежде чем продолжать, используя следующую команду:

    ```bash
    sudo pcs resource
    ```

    Вы должны увидеть следующий результат.
    
    **RHEL 7** 
    
    ```output
    [<username>@VM1 ~]$ sudo pcs resource
    Master/Slave Set: ag_cluster-master [ag_cluster]
    Masters: [ <VM1> ]
    Slaves: [ <VM2> <VM3> ]
    ```
    
    **RHEL 8** 
    
    ```output
    [<username>@VM1 ~]$ sudo pcs resource
    * Clone Set: ag_cluster-clone [ag_cluster] (promotable):
    * ag_cluster             (ocf::mssql:ag) :            Slave VMrhel3 (Monitoring) 
    * ag_cluster             (ocf::mssql:ag) :            Master VMrhel1 (Monitoring)
    * ag_cluster             (ocf::mssql:ag) :            Slave VMrhel2 (Monitoring)
    ```


### <a name="create-a-virtual-ip-resource"></a>Создание ресурса виртуального IP-адреса

1. Для создания ресурса виртуального IP-адреса, используйте доступный статический IP-адрес из вашей сети. Его можно найти с помощью программы командной строки `nmap`.

    ```bash
    nmap -sP <IPRange>
    # For example: nmap -sP 10.0.0.*
    # The above will scan for all IP addresses that are already occupied in the 10.0.0.x space.
    ```

2. Установите для свойства **stonith-включен** значение false

    ```bash
    sudo pcs property set stonith-enabled=false
    ```

3. Создайте ресурс виртуального IP-адреса с помощью указанной ниже команды.

    - Замените приведенное ниже значение `<availableIP>` неиспользованным IP-адресом.

    ```bash
    sudo pcs resource create virtualip ocf:heartbeat:IPaddr2 ip=<availableIP>
    ```

### <a name="add-constraints"></a>Добавление ограничений

1. Чтобы ресурс IP-адреса и ресурс группы доступности выполнялись в одном узле, необходимо настроить ограничение совместного размещения. Выполните следующую команду:

   **RHEL 7**
  
    ```bash
    sudo pcs constraint colocation add virtualip ag_cluster-master INFINITY with-rsc-role=Master
    ```

   **RHEL 8**
   
    ```bash
     sudo pcs constraint colocation add virtualip with master ag_cluster-clone INFINITY with-rsc-role=Master
    ```
  
2. Создайте ограничение очередности, чтобы ресурс группы доступности запускался до ресурса IP-адреса. Ограничение совместного размещения предполагает такое ограничение, поэтому его необходимо задать явно.

   **RHEL 7**
   
    ```bash
    sudo pcs constraint order promote ag_cluster-master then start virtualip
    ```

   **RHEL 8**
   
    ```bash
    sudo pcs constraint order promote ag_cluster-clone then start virtualip
    ```
  
3. Чтобы проверить ограничения, выполните следующую команду:

    ```bash
    sudo pcs constraint list --full
    ```

    Вы должны увидеть следующий результат.
    
    **RHEL 7**

    ```
    Location Constraints:
    Ordering Constraints:
          promote ag_cluster-master then start virtualip (kind:Mandatory) (id:order-ag_cluster-master-virtualip-mandatory)
    Colocation Constraints:
          virtualip with ag_cluster-master (score:INFINITY) (with-rsc-role:Master) (id:colocation-virtualip-ag_cluster-master-INFINITY)
    Ticket Constraints:
    ```
    
    **RHEL 8**
    
    ```output
    Location Constraints:
    Ordering Constraints:
            promote ag_cluster-clone then start virtualip (kind:Mandatory) (id:order-ag_cluster-clone-virtualip-mandatory)
    Colocation Constraints:
            virtualip with ag_cluster-clone (score:INFINITY) (with-rsc-role:Master) (id:colocation-virtualip-ag_cluster-clone-INFINITY)
    Ticket Constraints:
    ```

### <a name="re-enable-stonith"></a>Повторное включение stonith

Мы готовы к тестированию. Повторно включите stonith в кластере, выполнив следующую команду на узле 1:

```bash
sudo pcs property set stonith-enabled=true
```

### <a name="check-cluster-status"></a>Проверка состояния кластера

Вы можете проверить состояние кластерных ресурсов, используя следующую команду:

```output
[<username>@VM1 ~]$ sudo pcs status
Cluster name: az-hacluster
Stack: corosync
Current DC: <VM3> (version 1.1.19-8.el7_6.5-c3c624ea3d) - partition with quorum
Last updated: Sat Dec  7 00:18:38 2019
Last change: Sat Dec  7 00:18:02 2019 by root via cibadmin on VM1

3 nodes configured
5 resources configured

Online: [ <VM1> <VM2> <VM3> ]

Full list of resources:

 Master/Slave Set: ag_cluster-master [ag_cluster]
     Masters: [ <VM2> ]
     Slaves: [ <VM1> <VM3> ]
 virtualip      (ocf::heartbeat:IPaddr2):       Started <VM2>
 rsc_st_azure   (stonith:fence_azure_arm):      Started <VM1>

Daemon Status:
  corosync: active/enabled
  pacemaker: active/enabled
  pcsd: active/enabled
```

## <a name="test-failover"></a>Тестовая отработка отказа

Чтобы убедиться, что настройка прошла успешно, мы протестируем отработку отказа. Дополнительные сведения см. в разделе [Отработка отказа для группы доступности Always On на Linux](/sql/linux/sql-server-linux-availability-group-failover-ha).

1. Выполните следующую команду, чтобы вручную выполнить отработку отказа первичной реплики в `<VM2>`. Замените `<VM2>` значением имени вашего сервера.

   **RHEL 7**
   
    ```bash
    sudo pcs resource move ag_cluster-master <VM2> --master
    ```

   **RHEL 8**
   
    ```bash
    sudo pcs resource move ag_cluster-clone <VM2> --master
    ```

   Кроме того, вы можете указать дополнительный параметр, чтобы временное ограничение, созданное для перемещения ресурса на нужный узел, отключалось автоматически, не требуя выполнения шагов 2 и 3, приведенных ниже.

   **RHEL 7**

    ```bash
    sudo pcs resource move ag_cluster-master <VM2> --master lifetime=30S
    ```

   **RHEL 8**

    ```bash
    sudo pcs resource move ag_cluster-clone <VM2> --master lifetime=30S
    ```

   Другим вариантом автоматизации шагов 2 и 3, описанных ниже, с помощью которых можно снять временное ограничение в самой команде перемещения ресурса, является объединение нескольких команд в одну строку. 

   **RHEL 7**

    ```bash
    sudo pcs resource move ag_cluster-master <VM2> --master && sleep 30 && pcs resource clear ag_cluster-master
    ```

   **RHEL 8**

    ```bash
    sudo pcs resource move ag_cluster-clone <VM2> --master && sleep 30 && pcs resource clear ag_cluster-clone
    ```
    
2. Если вы еще раз проверите свои ограничения, то увидите, что из-за ручной отработки отказа было добавлено еще одно ограничение:
    
    **RHEL 7**
    
    ```output
    [<username>@VM1 ~]$ sudo pcs constraint list --full
    Location Constraints:
          Resource: ag_cluster-master
            Enabled on: VM2 (score:INFINITY) (role: Master) (id:cli-prefer-ag_cluster-master)
    Ordering Constraints:
            promote ag_cluster-master then start virtualip (kind:Mandatory) (id:order-ag_cluster-master-virtualip-mandatory)
    Colocation Constraints:
            virtualip with ag_cluster-master (score:INFINITY) (with-rsc-role:Master) (id:colocation-virtualip-ag_cluster-master-INFINITY)
    Ticket Constraints:
    ```

    **RHEL 8**
    
    ```output
    [<username>@VM1 ~]$ sudo pcs constraint list --full
    Location Constraints:
          Resource: ag_cluster-master
            Enabled on: VM2 (score:INFINITY) (role: Master) (id:cli-prefer-ag_cluster-clone)
    Ordering Constraints:
            promote ag_cluster-clone then start virtualip (kind:Mandatory) (id:order-ag_cluster-clone-virtualip-mandatory)
    Colocation Constraints:
            virtualip with ag_cluster-clone (score:INFINITY) (with-rsc-role:Master) (id:colocation-virtualip-ag_cluster-clone-INFINITY)
    Ticket Constraints:
    ```
    
3. Удалите ограничение c идентификатором `cli-prefer-ag_cluster-master`, используя следующую команду:

    **RHEL 7**
    
    ```bash
    sudo pcs constraint remove cli-prefer-ag_cluster-master
    ```

    **RHEL 8**
    
    ```bash
    sudo pcs constraint remove cli-prefer-ag_cluster-clone
    ```

1. Проверьте кластерные ресурсы с помощью команды `sudo pcs resource`, теперь основным экземпляром должна быть `<VM2>`.

    ```output
    [<username>@<VM1> ~]$ sudo pcs resource
    Master/Slave Set: ag_cluster-master [ag_cluster]
         ag_cluster (ocf::mssql:ag):        FAILED <VM1> (Monitoring)
         Masters: [ <VM2> ]
         Slaves: [ <VM3> ]
    virtualip      (ocf::heartbeat:IPaddr2):       Started <VM2>
    [<username>@<VM1> ~]$ sudo pcs resource
    Master/Slave Set: ag_cluster-master [ag_cluster]
         Masters: [ <VM2> ]
         Slaves: [ <VM1> <VM3> ]
    virtualip      (ocf::heartbeat:IPaddr2):       Started <VM2>
    ```

## <a name="test-fencing"></a>Тестирование ограждения

Вы можете протестировать STONITH, выполнив следующую команду. Попробуйте выполнить указанную ниже команду из `<VM1>` для `<VM3>`.

```bash
sudo pcs stonith fence <VM3> --debug
```

> [!NOTE]
> По умолчанию, действие ограждения приводит к тому, что узел отключается, а затем включается. Если вы хотите перевести узел в автономный режим, используйте в команде параметр `--off`.

Вы должны увидеть следующий результат:

```output
[<username>@<VM1> ~]$ sudo pcs stonith fence <VM3> --debug
Running: stonith_admin -B <VM3>
Return Value: 0
--Debug Output Start--
--Debug Output End--
 
Node: <VM3> fenced
```
Дополнительные сведения см. в статье [5.12. Testing a fence device](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/high_availability_add-on_reference/s1-stonithtest-haar) (Тестирование устройства ограждения).

## <a name="next-steps"></a>Дальнейшие действия

Чтобы использовать прослушиватель группы доступности для экземпляров SQL Server, нужно создать и настроить подсистему балансировки нагрузки.

> [!div class="nextstepaction"]
> [Руководство. Настройка прослушивателя группы доступности для SQL Server на виртуальных машинах RHEL в Azure](rhel-high-availability-listener-tutorial.md)
