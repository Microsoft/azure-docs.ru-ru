---
title: Azure COMPUTE — диагностическое расширение Linux 4,0
description: Как настроить диагностическое расширение Azure Linux (LAD) 4,0 для сбора метрик и ведения журнала событий из виртуальных машин Linux, работающих в Azure.
ms.topic: article
ms.service: virtual-machines
ms.subservice: extensions
author: amjads1
ms.author: amjads
ms.collection: linux
ms.date: 02/05/2021
ms.openlocfilehash: 4c4851ab28e5da74e7f1fa36f087ecfdabb1c638
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102560129"
---
# <a name="use-linux-diagnostic-extension-40-to-monitor-metrics-and-logs"></a>Использование диагностического расширения Linux 4,0 для мониторинга метрик и журналов

В этом документе описывается версия 4,0 и более новой версии диагностического расширения Linux.

> [!IMPORTANT]
> Сведения о версии 3. * см. в  [этом документе](./diagnostics-linux-v3.md). Сведения о версии 2.3 и более ранних см. в [этом документе](/previous-versions/azure/virtual-machines/linux/classic/diagnostic-extension-v2).

## <a name="introduction"></a>Введение

Диагностическое расширение Linux помогает отслеживать работоспособность виртуальных машин Linux, работающих на платформе Microsoft Azure. Оно предоставляет следующие возможности.

* Собирает метрики производительности системы из виртуальных машин и сохраняет их в отдельной таблице в назначенной учетной записи хранения.
* Извлекает события из системного журнала и сохраняет их в отдельной таблице в назначенной учетной записи хранения.
* Позволяет пользователям настраивать метрики данных, которые будут собираться и передаваться.
* Позволяет пользователям настраивать средства обслуживания системного журнала и уровни серьезности для событий, которые собираются и передаются.
* Позволяет пользователям передавать определенные файлы журналов в отдельную таблицу хранилища.
* Поддерживает отправку метрик и событий журнала в произвольные конечные точки EventHub и BLOB-объекты в формате JSON в назначенной учетной записи хранения.

Это расширение работает с обеими моделями развертывания Azure.

## <a name="installing-the-extension-in-your-vm"></a>Установка расширения в виртуальной машине

Это расширение можно включить с помощью командлетов Azure PowerShell, скриптов Azure CLI, шаблонов ARM или портала Azure. Дополнительные сведения см. в статье [Обзор расширений](features-linux.md).

>[!NOTE]
>Некоторые компоненты расширения виртуальной машины диагностики также поставляются в [расширение log Analytics VM](./oms-linux.md). Из-за этой архитектуры могут возникать конфликты, если оба расширения создаются в одном шаблоне ARM. Чтобы избежать этих конфликтов во время установки, используйте [ `dependsOn` директиву](../../azure-resource-manager/templates/define-resource-dependency.md#dependson) , чтобы убедиться, что расширения установлены последовательно. Расширения можно устанавливать в любом порядке.

Эти инструкции по установке и [загружаемый образец конфигурации](https://raw.githubusercontent.com/Azure/azure-linux-extensions/master/Diagnostic/tests/lad_2_3_compatible_portal_pub_settings.json) для LAD 4,0 можно настроить следующим образом:

* сбор и хранение тех же метрик, которые были предоставлены LAD 2,3, 3 *;
* Отправка метрик в Azure Monitor приемника вместе с обычным приемником в службе хранилища Azure, новое в LAD 4,0
* Запишите полезный набор метрик файловой системы, как это было предоставлено LAD 3,0;
* сбор событий по умолчанию системного журнала, которые были доступны в LAD 2.3;
* обеспечение построения диаграмм и вывода оповещений на основе метрик виртуальных машин на портале Azure.

Доступная для скачивания конфигурация — всего лишь пример. Измените ее в соответствии со своими потребностями.

### <a name="supported-linux-distributions"></a>Поддерживаемые дистрибутивы Linux

Диагностическое расширение Linux поддерживает следующие дистрибутивы и версии. Этот список дистрибутивов и версий применяется только к образам поставщиков Linux, одобренных Azure. Сторонние образы BYOL и BYOS, например устройства, обычно не поддерживаются диагностическим расширением Linux.

Если для дистрибутива перечислены только основные версии, например Debian 7, то поддерживаются также все дополнительные версии этого дистрибутива. Если указана конкретная дополнительная версия, то поддерживается только эта конкретная версия. Если к ней добавлен знак "+", значит поддерживаются еще и все дополнительные версии, которые больше указанной.

Поддерживаемые дистрибутивы и версии.

- Ubuntu 18.04, 16.04, 14.04
- CentOS 7, 6.5+
- Oracle Linux 7, 6.4+
- OpenSUSE 13.1+
- SUSE Linux Enterprise Server 12
- Debian 9, 8, 7
- RHEL 7, 6.7+

### <a name="prerequisites"></a>Предварительные требования

* **Агент Linux для Azure 2.2.0 или более поздней версии**. Большинство образов в коллекции виртуальных машин Azure на базе Linux включает версию 2.2.7 или более позднюю. Чтобы проверить версию, установленную в виртуальной машине, выполните команду `/usr/sbin/waagent -version`. Если в виртуальной машине установлена более старая версия гостевого агента, выполните [эти инструкции](./update-linux-agent.md) по ее обновлению.
* **Azure CLI**. [Установите среду Azure CLI](/cli/azure/install-azure-cli) на компьютере.
* Команда wget, если у вас ее еще нет: Выполните `sudo apt-get install wget`.
* Существующая подписка Azure и существующая учетная запись хранения общего назначения для хранения данных.  Учетные записи хранения общего назначения поддерживают хранилище таблиц, которое требуется.  Учетная запись хранилища BLOB-объектов не будет работать.
* Python 2;

### <a name="python-requirement"></a>Требование к Python

Для расширения диагностики Linux требуется Python 2. Если ваша виртуальная машина использует дистрибутив, которая по умолчанию не включает Python 2, ее необходимо установить. Приведенные ниже примеры команд будут устанавливать Python 2 на разных дистрибутивов.    

 - Red Hat, CentOS, Oracle: `yum install -y python2`
 - Ubuntu, Debian: `apt-get install -y python2`
 - SUSE: `zypper install -y python2`

Исполняемый файл python2 должен иметь псевдоним для *Python*. Ниже приведен один метод, который можно использовать для задания этого псевдонима:

1. Выполните следующую команду, чтобы удалить все существующие псевдонимы.
 
    ```
    sudo update-alternatives --remove-all python
    ```

2. Выполните следующую команду, чтобы создать псевдоним.

    ```
    sudo update-alternatives --install /usr/bin/python python /usr/bin/python2 1
    ```

### <a name="sample-installation"></a>Пример установки

> [!NOTE]
> Для любого из примеров заполните правильные значения переменных в первом разделе перед выполнением. 

Пример конфигурации, скачиваемый в этих примерах, собирает набор стандартных данных и отправляет их в хранилище таблиц. URL-адрес и содержимое примера конфигурации могут меняться. В большинстве случаев вам нужно скачать копию JSON-файла параметров портала и изменить его под свои потребности, а затем передать свою версию файла конфигурации в шаблоны или средства автоматизации, которые используются в вашей среде, а не скачивать его каждый раз по этому URL-адресу.

> [!NOTE]
> Чтобы включить новый приемник Azure Monitor, виртуальным машинам необходимо включить удостоверение системы для создания маркера проверки подлинности MSI. Это можно сделать во время создания виртуальной машины или после ее создания. Действия по включению удостоверения, назначенного системой, с помощью портала, интерфейса командной строки, PowerShell и диспетчера ресурсов.  перечислены [здесь](../../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)подробно. 

#### <a name="azure-cli-sample"></a>Пример Azure CLI

```azurecli
# Set your Azure VM diagnostic variables correctly below
my_resource_group=<your_azure_resource_group_name_containing_your_azure_linux_vm>
my_linux_vm=<your_azure_linux_vm_name>
my_diagnostic_storage_account=<your_azure_storage_account_for_storing_vm_diagnostic_data>

# Should login to Azure first before anything else
az login

# Select the subscription containing the storage account
az account set --subscription <your_azure_subscription_id>

# Enable System Assigned Identity to the existing VM
az vm identity assign -g $my_resource_group -n $my_linux_vm

# Download the sample Public settings. (You could also use curl or any web browser)
wget https://raw.githubusercontent.com/Azure/azure-linux-extensions/master/Diagnostic/tests/lad_2_3_compatible_portal_pub_settings.json -O portal_public_settings.json

# Build the VM resource ID. Replace storage account name and resource ID in the public settings.
my_vm_resource_id=$(az vm show -g $my_resource_group -n $my_linux_vm --query "id" -o tsv)
sed -i "s#__DIAGNOSTIC_STORAGE_ACCOUNT__#$my_diagnostic_storage_account#g" portal_public_settings.json
sed -i "s#__VM_RESOURCE_ID__#$my_vm_resource_id#g" portal_public_settings.json

# Build the protected settings (storage account SAS token)
my_diagnostic_storage_account_sastoken=$(az storage account generate-sas --account-name $my_diagnostic_storage_account --expiry 2037-12-31T23:59:00Z --permissions wlacu --resource-types co --services bt -o tsv)
my_lad_protected_settings="{'storageAccountName': '$my_diagnostic_storage_account', 'storageAccountSasToken': '$my_diagnostic_storage_account_sastoken'}"

# Finally tell Azure to install and enable the extension
az vm extension set --publisher Microsoft.Azure.Diagnostics --name LinuxDiagnostic --version 4.0 --resource-group $my_resource_group --vm-name $my_linux_vm --protected-settings "${my_lad_protected_settings}" --settings portal_public_settings.json
```
#### <a name="azure-cli-sample-for-installing-lad-40-extension-on-the-virtual-machine-scale-set-instance"></a>Azure CLI пример установки расширения LAD 4,0 в экземпляре масштабируемого набора виртуальных машин

```azurecli
#Set your Azure VMSS diagnostic variables correctly below
$my_resource_group=<your_azure_resource_group_name_containing_your_azure_linux_vm>
$my_linux_vmss=<your_azure_linux_vmss_name>
$my_diagnostic_storage_account=<your_azure_storage_account_for_storing_vm_diagnostic_data>

# Should login to Azure first before anything else
az login

# Select the subscription containing the storage account
az account set --subscription <your_azure_subscription_id>

# Enable System Assigned Identity to the existing VMSS
az vmss identity assign -g $my_resource_group -n $my_linux_vmss

# Download the sample Public settings. (You could also use curl or any web browser)
wget https://raw.githubusercontent.com/Azure/azure-linux-extensions/master/Diagnostic/tests/lad_2_3_compatible_portal_pub_settings.json -O portal_public_settings.json

# Build the VMSS resource ID. Replace storage account name and resource ID in the public settings.
$my_vmss_resource_id=$(az vmss show -g $my_resource_group -n $my_linux_vmss --query "id" -o tsv)
sed -i "s#__DIAGNOSTIC_STORAGE_ACCOUNT__#$my_diagnostic_storage_account#g" portal_public_settings.json
sed -i "s#__VM_RESOURCE_ID__#$my_vmss_resource_id#g" portal_public_settings.json

# Build the protected settings (storage account SAS token)
$my_diagnostic_storage_account_sastoken=$(az storage account generate-sas --account-name $my_diagnostic_storage_account --expiry 2037-12-31T23:59:00Z --permissions wlacu --resource-types co --services bt -o tsv)
$my_lad_protected_settings="{'storageAccountName': '$my_diagnostic_storage_account', 'storageAccountSasToken': '$my_diagnostic_storage_account_sastoken'}"

# Finally tell Azure to install and enable the extension
az vmss extension set --publisher Microsoft.Azure.Diagnostics --name LinuxDiagnostic --version 4.0 --resource-group $my_resource_group --vmss-name $my_linux_vmss --protected-settings "${my_lad_protected_settings}" --settings portal_public_settings.json
```

#### <a name="powershell-sample"></a>Пример для PowerShell

```powershell
$storageAccountName = "yourStorageAccountName"
$storageAccountResourceGroup = "yourStorageAccountResourceGroupName"
$vmName = "yourVMName"
$VMresourceGroup = "yourVMResourceGroupName"

# Get the VM object
$vm = Get-AzVM -Name $vmName -ResourceGroupName $VMresourceGroup

# Enable System Assigned Identity on an existing VM
Update-AzVM -ResourceGroupName $VMresourceGroup -VM $vm -IdentityType SystemAssigned

# Get the public settings template from GitHub and update the templated values for storage account and resource ID
$publicSettings = (Invoke-WebRequest -Uri https://raw.githubusercontent.com/Azure/azure-linux-extensions/master/Diagnostic/tests/lad_2_3_compatible_portal_pub_settings.json).Content
$publicSettings = $publicSettings.Replace('__DIAGNOSTIC_STORAGE_ACCOUNT__', $storageAccountName)
$publicSettings = $publicSettings.Replace('__VM_RESOURCE_ID__', $vm.Id)

# If you have your own customized public settings, you can inline those rather than using the template above: $publicSettings = '{"ladCfg":  { ... },}'

# Generate a SAS token for the agent to use to authenticate with the storage account
$sasToken = New-AzStorageAccountSASToken -Service Blob,Table -ResourceType Service,Container,Object -Permission "racwdlup" -Context (Get-AzStorageAccount -ResourceGroupName $storageAccountResourceGroup -AccountName $storageAccountName).Context -ExpiryTime $([System.DateTime]::Now.AddYears(10))

# Build the protected settings (storage account SAS token)
$protectedSettings="{'storageAccountName': '$storageAccountName', 'storageAccountSasToken': '$sasToken'}"

# Finally install the extension with the settings built above
Set-AzVMExtension -ResourceGroupName $VMresourceGroup -VMName $vmName -Location $vm.Location -ExtensionType LinuxDiagnostic -Publisher Microsoft.Azure.Diagnostics -Name LinuxDiagnostic -SettingString $publicSettings -ProtectedSettingString $protectedSettings -TypeHandlerVersion 4.0 
```

### <a name="updating-the-extension-settings"></a>Изменение параметров расширения

Изменив закрытые или общедоступные параметры, разверните их в виртуальных машинах, выполнив ту же команду. Если в параметрах есть изменения, они будут переданы в расширение. Расширение LAD повторно загрузит конфигурацию и перезапустится.

### <a name="migration-from-previous-versions-of-the-extension"></a>Переход с предыдущих версий расширения

Последняя версия расширения — **4,0, которая в настоящее время находится в общедоступной предварительной** версии. **Старые версии 3. x по-прежнему поддерживаются, тогда как версии 2. x являются устаревшими, начиная с 31 июля 2018**.

> [!IMPORTANT]
> Чтобы выполнить миграцию из 3. x в новую версию расширения, необходимо удалить старое расширение, а затем установить версию 4 расширения (с обновленной конфигурацией для назначения удостоверений системы и приемников для отправки метрик в приемник Azure Monitor.)

Рекомендации

* При установке расширения включите автоматическое обновление дополнительного номера версии.
  * На виртуальных машинах классической модели развертывания укажите "4. *" в качестве версии, если вы устанавливаете расширение с помощью Azure XPLAT CLI или PowerShell.
  * В виртуальных машинах на основе модели развертывания с помощью Azure Resource Manager включите в шаблон развертывания виртуальных машин параметр ""autoUpgradeMinorVersion": true".
* Может использовать ту же учетную запись хранения для LAD 4,0, что и LAD 3. *. 

## <a name="protected-settings"></a>Защищенные параметры

Этот набор данных конфигурации включает в себя конфиденциальные сведения, которые не должны быть общедоступными, например учетные данные для доступа к хранилищу. Эти параметры передаются в хранилище и сохраняются расширением в зашифрованной форме.

```json
{
    "storageAccountName" : "the storage account to receive data",
    "storageAccountEndPoint": "the hostname suffix for the cloud for this account",
    "storageAccountSasToken": "SAS access token",
    "mdsdHttpProxy": "HTTP proxy settings",
    "sinksConfig": { ... }
}
```

Имя | Значение
---- | -----
storageAccountName | Имя учетной записи хранения, в которую расширение записывает данные.
storageAccountEndPoint | Конечная точка, определяющая облако, в котором находится учетная запись хранения (необязательно). Если этот параметр отсутствует, LAD по умолчанию размещается в общедоступном облаке Azure (`https://core.windows.net`). Чтобы использовать учетную запись хранения в Azure для Германии, Azure для государственных организаций или Azure для Китая, задайте соответствующее значение.
storageAccountSasToken | [Токен SAS учетной записи](https://azure.microsoft.com/blog/sas-update-account-sas-now-supports-all-storage-services/) для служб BLOB-объектов и таблиц (`ss='bt'`), применимый к контейнерам и объектам (`srt='co'`), который предоставляет разрешения на добавление, создание, перечисление, обновление и запись (`sp='acluw'`). *Не* включайте начальный вопросительный знак (?).
mdsdHttpProxy | Сведения о прокси-сервере HTTP, необходимые для обеспечения подключения расширения к указанной учетной записи хранения и конечной точке (необязательно).
sinksConfig | Сведения об альтернативных местах назначения, в которые могут доставляться метрики и события (необязательно). Подробные сведения о каждом приемнике данных, поддерживаемом расширением, рассматриваются в приведенных ниже разделах.

Чтобы получить маркер SAS в шаблоне Resource Manager, используйте функцию **listAccountSas**. Пример такого шаблона см. в разделе [Пример функции list](../../azure-resource-manager/templates/template-functions-resource.md#list-example).

Вы можете легко создать необходимый токен SAS на портале Azure.

1. Выберите учетную запись хранения общего назначения, в которую расширение будет записывать данные.
1. В левом меню в разделе "Параметры" выберите пункт "Подписанный URL-адрес".
1. Заполните соответствующие разделы, как описано выше.
1. Нажмите кнопку "Создать SAS".

:::image type="content" source="./media/diagnostics-linux/make_sas.png" alt-text="На снимке экрана показана страница подписанного URL-доступа с созданием S.":::

Скопируйте созданный адрес SAS в поле storageAccountSasToken; удалите начальный вопросительный знак ("?").

### <a name="sinksconfig"></a>sinksConfig

```json
"sinksConfig": {
    "sink": [
        {
            "name": "sinkname",
            "type": "sinktype",
            ...
        },
        ...
    ]
},
```

В этом необязательном разделе определяются дополнительные места назначения, в которые расширение отправляет собранные данные. Массив sink содержит по одному объекту для каждого дополнительного приемника данных. Атрибут type определяет другие атрибуты объекта.

Элемент | Значение
------- | -----
name | Строка, используемая для указания этого приемника в других местах конфигурации расширения.
type | Тип определяемого приемника. Определяет другие значения (при их наличии) для экземпляров этого типа.

Версия 4,0 диагностического расширения Linux поддерживает два типа приемника: EventHub и JsonBlob.

#### <a name="the-eventhub-sink"></a>Приемник EventHub

```json
"sink": [
    {
        "name": "sinkname",
        "type": "EventHub",
        "sasURL": "https SAS URL"
    },
    ...
]
```

Запись sasURL содержит полный URL-адрес, включая токен SAS, для концентратора событий, в которым должны публиковаться данные. Для LAD требуется политика именования SAS, включающая утверждение отправки. Пример:

* Создание пространства имен Центров событий с именем `contosohub`
* Создание концентратора событий в пространстве имен с именем `syslogmsgs`
* Создание общей политики доступа для концентратора событий с именем `writer`, включающей утверждение отправки

Если вы успешно создали адрес SAS в полночь (UTC) 1 января 2018 г., значение sasURL может быть следующим:

```https
https://contosohub.servicebus.windows.net/syslogmsgs?sr=contosohub.servicebus.windows.net%2fsyslogmsgs&sig=xxxxxxxxxxxxxxxxxxxxxxxxx&se=1514764800&skn=writer
```

Дополнительные сведения о создании маркеров SAS для Центров событий и получении сведений о них см. на [этой веб-странице](/rest/api/eventhub/generate-sas-token#powershell).

#### <a name="the-jsonblob-sink"></a>Приемник JsonBlob

```json
"sink": [
    {
        "name": "sinkname",
        "type": "JsonBlob"
    },
    ...
]
```

Данные, передаваемые в приемник JsonBlob, хранятся в BLOB-объектах в хранилище Azure. Каждый экземпляр LAD каждый час создает BLOB-объект для каждого имени приемника. Каждый BLOB-объект всегда содержит синтаксически правильный массив JSON объекта. Новые записи добавляются в массив атомарным образом. BLOB-объекты хранятся в контейнере с тем же именем, что и приемник. К именам приемников JsonBlob применяются правила службы хранилища Azure для имен контейнеров BLOB-объектов: от 3 до 63 букв, цифр или дефисов в кодировке ASCII.

## <a name="public-settings"></a>Общедоступные параметры

Эта структура содержит различные блоки параметров, которые определяют то, какие сведения собираются расширением. Каждый параметр (кроме Ладкфг) является необязательным. Если вы укажете метрику или коллекцию syslog в `ladCfg` , необходимо также указать `StorageAccount` . необходимо указать элемент sinksConfig, чтобы включить приемник Azure Monitor для метрик из LAD 4,0

```json
{
    "ladCfg":  { ... },
    "fileLogs": { ... },
    "StorageAccount": "the storage account to receive data",
    "sinksConfig": { ... },
    "mdsdHttpProxy" : ""
}
```

Элемент | Значение
------- | -----
StorageAccount | Имя учетной записи хранения, в которую расширение записывает данные. Должно совпадать с именем, указанным в [защищенных параметрах](#protected-settings).
mdsdHttpProxy | То же значение, что и в [защищенных параметрах](#protected-settings) (необязательно). Если задано значение защищенного параметра, оно определяет значение общедоступного параметра. Параметры прокси-сервера, которые содержат секретные данные, такие как пароль, следует указывать в [защищенных параметрах](#protected-settings).

Остальные элементы подробно описываются в следующих разделах.

### <a name="ladcfg"></a>ladCfg

```json
"ladCfg": {
    "diagnosticMonitorConfiguration": {
        "eventVolume": "Medium",
        "metrics": { ... },
        "performanceCounters": { ... },
        "syslogEvents": { ... }
    },
    "sampleRateInSeconds": 15
}
```

Эта структура управляет сбором метрик и журналов для доставки в службу метрик Azure и другим приемникам данных. Необходимо указать `performanceCounters`, `syslogEvents` или и то, и другое. Требуется задать структуру `metrics`.

Если вы не хотите включать системный журнал или коллекцию метрик, можно просто указать пустую структуру для элемента Ладкфг, как показано ниже. 

```json
"ladCfg": {
    "diagnosticMonitorConfiguration": {}
    }
```

Элемент | Значение
------- | -----
eventVolume | Определяет количество разделов, создаваемых в таблице хранилища (необязательно). Должно иметь значение `"Large"`, `"Medium"` или `"Small"`. Если не задано, по умолчанию используется значение `"Medium"`.
sampleRateInSeconds | Интервал по умолчанию между сбором необработанных (не объединенных) метрик (необязательно). Минимальная поддерживаемая частота выборки — 15 секунд. Если не задано, по умолчанию используется значение `15`.

#### <a name="metrics"></a>Метрики

```json
"metrics": {
    "resourceId": "/subscriptions/...",
    "metricAggregation" : [
        { "scheduledTransferPeriod" : "PT1H" },
        { "scheduledTransferPeriod" : "PT5M" }
    ]
}
```

Элемент | Значение
------- | -----
resourceId | Идентификатор ресурса Azure Resource Manager виртуальной машины или масштабируемого набора виртуальных машин, к которой относится виртуальная машина. Этот параметр также необходимо указывать, если в конфигурации используется какой-либо приемник JsonBlob.
scheduledTransferPeriod | Частота, с которой совокупные метрики должны вычисляться и передаваться в службу метрик Azure, выраженный в виде интервала IS 8601. Наименьший период передачи составляет 60 секунд, то есть PT1M. Необходимо указать по крайней мере одно значение scheduledTransferPeriod.

Выборка метрик, указанных в разделе performanceCounters, производится каждые 15 секунд или с частотой, явно определенной для счетчика. Если задано несколько частот scheduledTransferPeriod (как в примере), каждое агрегирование производится независимо.

#### <a name="performancecounters"></a>performanceCounters

```json
"performanceCounters": {
    "sinks": "",
    "performanceCounterConfiguration": [
        {
            "type": "builtin",
            "class": "Processor",
            "counter": "PercentIdleTime",
            "counterSpecifier": "/builtin/Processor/PercentIdleTime",
            "condition": "IsAggregate=TRUE",
            "sampleRate": "PT15S",
            "unit": "Percent",
            "annotation": [
                {
                    "displayName" : "Aggregate CPU %idle time",
                    "locale" : "en-us"
                }
            ]
        }
    ]
}
```

Этот необязательный раздел управляет сбором метрик. Необработанные выборки собираются для каждого периода [scheduledTransferPeriod](#metrics) для вычисления следующих значений:

* mean
* minimum
* maximum
* последнее собранное значение;
* количество необработанных выборок для вычисления агрегата.

Элемент | Значение
------- | -----
sinks | Разделенный запятыми список имен приемников, в которые LAD отправляет агрегированные результаты метрик (необязательно). Все агрегированные метрики публикуются в каждом приемнике из списка. См. [sinksConfig](#sinksconfig). Например, `"EHsink1, myjsonsink"`.
type | Определяет фактический поставщик метрики.
class | Вместе с элементом counter определяет конкретную метрику в пространстве имен поставщика.
Счетчик | Вместе с элементом class определяет конкретную метрику в пространстве имен поставщика.
counterSpecifier | Определяет конкретную метрику в пространстве имен метрик Azure.
condition | Выбирает определенный экземпляр объекта, к которому применяется метрика, или совокупность всех экземпляров этого объекта (необязательно). Дополнительные сведения см. в определениях метрики `builtin`.
sampleRate | Интервал IS 8601, который задает частоту, с которой собираются необработанные выборки данной метрики. Если этот элемент не задан, интервал сбора определяется значением [sampleRateInSeconds](#ladcfg). Минимальная поддерживаемая частота выборки — 15 секунд (PT15S).
unit | Должна быть одна из следующих строк: "Count", "Bytes", "Seconds", "Percent", "CountPerSecond", "BytesPerSecond", "Millisecond". Определяет единицу измерения для метрики. Потребители собранных данных ожидают, что значения собранных данных выражены в этой единице. LAD игнорирует это поле.
displayName | Метка (на языке, заданном с помощью соответствующего параметра языкового стандарта), которая добавляется к данным в метриках Azure. LAD игнорирует это поле.

counterSpecifier — это произвольный идентификатор. Потребители метрик, например функция построения диаграмм и вывода оповещений на портале Azure, используют counterSpecifier в качестве ключа, который определяет метрику или экземпляр метрики. Для метрик `builtin` рекомендуется использовать значения counterSpecifier, начинающиеся с `/builtin/`. При сборе определенного экземпляра метрики рекомендуется присоединять идентификатор экземпляра к значению counterSpecifier. Некоторые примеры.

* `/builtin/Processor/PercentIdleTime` — среднее время простоя всех виртуальных ЦП;
* `/builtin/Disk/FreeSpace(/mnt)` — свободное пространство для файловой системы /mnt;
* `/builtin/Disk/FreeSpace` — средний объем свободного пространства для всех подключенных файловых систем.

Ни расширение LAD, ни портал Azure не требуют соответствия значения counterSpecifier какому-либо шаблону. При формировании значений counterSpecifier соблюдайте единообразие.

При указании `performanceCounters` расширение LAD всегда записывает данные в таблицу в службе хранилища Azure. Те же самые данные можно записывать в BLOB-объекты JSON и Центры событий, однако отключить сохранение данных в таблице невозможно. Все экземпляры диагностического расширения, для которых настроено использование одного и того же имени учетной записи хранения и одной и той же конечной точки, добавляют метрики и журналы в одну и ту же таблицу. Если слишком много виртуальных машин осуществляют запись в один раздел таблицы, Azure может регулировать запись в него. Посредством параметра eventVolume записи распределяются по 1 (Small), 10 (Medium) или 100 (Large) разным разделам. Как правило, чтобы избежать регулирования трафика, достаточно значения Medium. Функция метрик Azure на портале Azure использует данные в этой таблице для создания графов или инициации оповещений. Имя таблицы образуется путем сцепления следующих строк:

* `WADMetrics`
* scheduledTransferPeriod для статистических значений, хранящихся в таблице;
* `P10DV2S`
* даты в формате ГГГГММДД, которая меняется каждые 10 дней.

Примеры: `WADMetricsPT1HP10DV2S20170410` и `WADMetricsPT1MP10DV2S20170609`.

#### <a name="syslogevents"></a>syslogEvents

```json
"syslogEvents": {
    "sinks": "",
    "syslogEventConfiguration": {
        "facilityName1": "minSeverity",
        "facilityName2": "minSeverity",
        ...
    }
}
```

Этот необязательный раздел управляет сбором событий из системного журнала. Если он отсутствует, события системного журнала не собираются.

Коллекция syslogEventConfiguration содержит по одной записи для каждого субъекта системного журнала, представляющего интерес. Если для определенного субъекта minSeverity имеет значение NONE или если субъект отсутствует в элементе, события от этого субъекта не собираются.

Элемент | Значение
------- | -----
sinks | Разделенный запятыми список имен приемников, в которых публикуются отдельные события журнала. Все события журнала, соответствующие ограничениям в syslogEventConfiguration, публикуются в каждом приемнике из списка. Пример "EHforsyslog"
facilityName | Имя субъекта системного журнала (например, "LOG\_USER" или "LOG\_LOCAL0"). Полный список см. в разделе "facility" на [странице man syslog](http://man7.org/linux/man-pages/man3/syslog.3.html).
minSeverity | Уровень серьезности системного журнала (например, "LOG\_ERR" или "LOG\_INFO"). Полный список см. в разделе "level" на [странице man syslog](http://man7.org/linux/man-pages/man3/syslog.3.html). Расширение собирает события, отправляемые субъекту, с указанным или более высоким уровнем.

При указании `syslogEvents` расширение LAD всегда записывает данные в таблицу в службе хранилища Azure. Те же самые данные можно записывать в BLOB-объекты JSON и Центры событий, однако отключить сохранение данных в таблице невозможно. Секционирование этой таблицы осуществляется так же, как таблицы `performanceCounters`. Имя таблицы образуется путем сцепления следующих строк:

* `LinuxSyslog`
* даты в формате ГГГГММДД, которая меняется каждые 10 дней.

Примеры: `LinuxSyslog20170410` и `LinuxSyslog20170609`.

### <a name="sinksconfig"></a>sinksConfig

Этот необязательный раздел управляет включением отправки метрик в приемник Azure Monitor в дополнение к учетной записи хранения и колонке метрик гостевой виртуальной машины по умолчанию.

> [!NOTE]
> Для этого на виртуальных машинах или VMSS необходимо включить удостоверение системы. Это можно сделать с помощью портала, интерфейса командной строки, PowerShell и диспетчера ресурсов. [Здесь](../../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)подробно описаны действия. Инструкции по включению этого также перечислены в примерах установки AZ CLI, PowerShell и т. д. 

```json
  "sinksConfig": {
    "sink": [
      {
        "name": "AzMonSink",
        "type": "AzMonSink",
        "AzureMonitor": {}
      }
    ]
  },
```


### <a name="filelogs"></a>fileLogs

Управляет записью файлов журналов. LAD собирает новые текстовые строки по мере их записи в файл и записывает их в строки таблицы и все указанные приемники (JsonBlob или EventHub).

> [!NOTE]
> Филелогс фиксируются подкомпонентом LAD с именем `omsagent` . Чтобы получить Филелогс, необходимо убедиться, что `omsagent` пользователь имеет разрешения на чтение для указанных файлов, а также разрешения на выполнение для всех каталогов в пути к этому файлу. Это можно проверить, запустив `sudo su omsagent -c 'cat /path/to/file'` после установки LAD.

```json
"fileLogs": [
    {
        "file": "/var/log/mydaemonlog",
        "table": "MyDaemonEvents",
        "sinks": ""
    }
]
```

Элемент | Значение
------- | -----
файл | Полный путь к файлу журнала, подлежащему отслеживанию и записи, и его имя. Путь и имя должны указывать на один файл; они не могут указывать на каталог или содержать подстановочные знаки. Учетная запись пользователя "omsagent" должна иметь доступ на чтение к пути к файлу.
table | Таблица службы хранилища Azure в назначенной учетной записи хранения (как указано в защищенной конфигурации), в которую записываются новые строки из заключительного фрагмента файла (необязательно).
sinks | Разделенный запятыми список имен дополнительных приемников, в которые отправляются строки из журнала (необязательно).

Необходимо указать либо table, либо sinks, либо оба эти элемента.

## <a name="metrics-supported-by-the-builtin-provider"></a>Метрики, поддерживаемые встроенным поставщиком

> [!NOTE]
> Метрики по умолчанию, поддерживаемые LAD, объединяются во всех файловых системах, дисках и именах. Для нестатистических метрик можно использовать новые метрики приемника Azure Monitor.

Встроенный поставщик метрик является источником метрик, которые наиболее интересны широкому кругу пользователей. Эти метрики делятся на пять основных классов:

* Процессор
* Память
* Сеть
* Файловая система
* Диск

### <a name="builtin-metrics-for-the-processor-class"></a>Встроенные метрики для класса "Процессор"

Класс метрик "Процессор" предоставляет сведения об использовании процессоров в виртуальной машине. В результате статистической обработки процентных значений получается среднее значение по всем ЦП. Если виртуальной машине выделено два виртуальных ЦП и один процессор был загружен на 100 %, а другой бездействовал 100 % времени, счетчик PercentIdleTime покажет значение 50. Если в течение этого же периода каждый процессор был загружен на 50 %, значением счетчика тоже будет 50. Если виртуальной машине выделено четыре виртуальных ЦП и один процессор был загружен на 100 %, а другие бездействовали, счетчик PercentIdleTime покажет значение 75.

Счетчик | Значение
------- | -------
PercentIdleTime | Процент периода статистической обработки, в течение которого процессоры выполняли цикл простоя ядра.
PercentProcessorTime | Процент времени, который был затрачен на выполнение всех потоков, кроме бездействующего.
PercentIOWaitTime | Процент времени, затраченный на ожидание завершения операций ввода-вывода.
PercentInterruptTime | Процент времени, затраченный на выполнение аппаратных и программных прерываний и отложенных вызовов процедур.
PercentUserTime | Процент времени за вычетом времени бездействия в течение периода статистической обработки, затраченный на выполнение операций пользователя с приоритетом, превышающим обычный.
PercentNiceTime | Процент времени, не являющегося временем бездействия, затраченный на выполнение операций с пониженным приоритетом (nice).
PercentPrivilegedTime | Процент времени, не являющегося временем бездействия, затраченный на выполнение операций в привилегированном режиме (режиме ядра).

Сумма показаний первых четырех счетчиков должна составлять 100 %. Сумма показаний последних трех счетчиков также равна 100 %; они представляют собой разбиение суммы счетчиков PercentProcessorTime, PercentIOWaitTime и PercentInterruptTime.

### <a name="builtin-metrics-for-the-memory-class"></a>Встроенные метрики для класса "Память"

Класс метрик "Память" предоставляет сведения об использовании памяти, подкачке и перестановке.

Счетчик | Значение
------- | -------
AvailableMemory | Доступный объем физической памяти в МиБ.
PercentAvailableMemory | Доступный объем физической памяти в процентах от общего объема памяти.
UsedMemory | Используемый объем физической памяти (МиБ).
PercentUsedMemory | Используемый объем физической памяти в процентах от общего объема памяти.
PagesPerSec | Общее количество операций подкачки (чтения и записи).
PagesReadPerSec | Число страниц, считанных из резервного хранилища (файла подкачки, файла программы, сопоставленного файла и т. д.).
PagesWrittenPerSec | Число страниц, записанных в резервное хранилище (файл подкачки, сопоставленный файл и т. д.).
AvailableSwap | Размер неиспользуемой области подкачки (МиБ).
PercentAvailableSwap | Размер неиспользуемой области подкачки в процентах от общего размера области подкачки.
UsedSwap | Размер используемой области подкачки (МиБ).
PercentUsedSwap | Размер используемой области подкачки в процентах от общего размера области подкачки.

Этот класс метрик имеет только один экземпляр. Атрибут condition не имеет полезных значений, и его следует опускать.

### <a name="builtin-metrics-for-the-network-class"></a>Встроенные метрики для класса "Сеть"

Класс метрик "сеть" предоставляет сведения об активности сети в отдельном сетевом интерфейсе с момента загрузки. LAD не предоставляет метрики пропускной способности, которые можно получить из метрик узла.

Счетчик | Значение
------- | -------
BytesTransmitted | Общее количество байт, отправленное с момента загрузки.
BytesReceived | Общее количество байт, полученное с момента загрузки.
BytesTotal | Общее количество байт, отправленное или полученное с момента загрузки.
PacketsTransmitted | Общее количество пакетов, отправленных с момента загрузки.
PacketsReceived | Общее количество пакетов, полученных с момента загрузки.
TotalRxErrors | Количество ошибок приема с момента загрузки.
TotalTxErrors | Количество ошибок передачи с момента загрузки.
TotalCollisions | Число конфликтов, о которых сообщили сетевые порты с момента загрузки.

### <a name="builtin-metrics-for-the-filesystem-class"></a>Встроенные метрики для класса "Файловая система"

Класс метрик "Файловая система" предоставляет сведения об использовании файловой системы. Абсолютные и процентные значения предоставляются так, как они отображались бы для обычного (непривилегированного) пользователя.

Счетчик | Значение
------- | -------
FreeSpace | Доступное дисковое пространство в байтах.
UsedSpace | Используемое дисковое пространство в байтах.
PercentFreeSpace | Процент свободного пространства.
PercentUsedSpace | Процент используемого пространства.
PercentFreeInodes | Процент неиспользуемых inode.
PercentUsedInodes | Процент выделенных (используемых) inode по всем файловым системам.
BytesReadPerSecond | Число прочитанных байт за секунду.
BytesWrittenPerSecond | Число записанных байт за секунду.
Байт/с | Число прочитанных или записанных байт за секунду.
ReadsPerSecond | Число операций чтения за секунду.
WritesPerSecond | Число операций записи за секунду.
TransfersPerSecond | Число операций чтения или записи за секунду.

### <a name="builtin-metrics-for-the-disk-class"></a>Встроенные метрики для класса "Диск"

Класс метрик "Диск" предоставляет сведения об использовании дискового устройства. Эти статистические данные относятся ко всему диску. Если на устройстве несколько файловых систем, предоставляются совокупные значения счетчиков по всем файловым системам.

Счетчик | Значение
------- | -------
ReadsPerSecond | Число операций чтения за секунду.
WritesPerSecond | Число операций записи за секунду.
TransfersPerSecond | Общее число операций за секунду.
AverageReadTime | Среднее число секунд на операцию чтения.
AverageWriteTime | Среднее число секунд на операцию записи.
AverageTransferTime | Среднее число секунд на операцию.
AverageDiskQueueLength | Среднее число операций с диском, помещенных в очередь.
ReadBytesPerSecond | Количество прочитанных байт за секунду.
WriteBytesPerSecond | Количество записанных байт за секунду.
Байт/с | Количество прочитанных или записанных байт за секунду.

## <a name="installing-and-configuring-lad-40"></a>Установка и настройка LAD 4,0

### <a name="azure-cli"></a>Azure CLI

Если защищенные параметры находятся в файле ProtectedSettings.jsи сведения о общедоступной конфигурации включены в PublicSettings.js, выполните следующую команду:

```azurecli
az vm extension set --publisher Microsoft.Azure.Diagnostics --name LinuxDiagnostic --version 4.0 --resource-group <resource_group_name> --vm-name <vm_name> --protected-settings ProtectedSettings.json --settings PublicSettings.json
```

В команде предполагается, что используется режим управления ресурсами Azure Azure CLI. Чтобы настроить LAD для виртуальных машин на основе классической модели развертывания (ASM), переключитесь в режим asm (`azure config mode asm`) и не указывайте в команде имя группы ресурсов. Дополнительные сведения см. в [документации по кроссплатформенному интерфейсу командной строки](/cli/azure/authenticate-azure-cli).

### <a name="powershell"></a>PowerShell

Если защищенные параметры находятся в `$protectedSettings` переменной, а сведения об общедоступной конфигурации находятся в `$publicSettings` переменной, выполните следующую команду:

```powershell
Set-AzVMExtension -ResourceGroupName <resource_group_name> -VMName <vm_name> -Location <vm_location> -ExtensionType LinuxDiagnostic -Publisher Microsoft.Azure.Diagnostics -Name LinuxDiagnostic -SettingString $publicSettings -ProtectedSettingString $protectedSettings -TypeHandlerVersion 4.0
```

## <a name="an-example-lad-40-configuration"></a>Пример конфигурации LAD 4,0

На основе предыдущих определений ниже приведен пример конфигурации расширения LAD 4,0 с пояснениями. Чтобы применить этот пример к своему случаю, следует использовать собственное имя учетной записи хранения, токен SAS учетной записи и токены SAS EventHub.

> [!NOTE]
> В зависимости от того, используется ли Azure CLI или PowerShell для установки LAD, метод предоставления общедоступных и защищенных параметров будет отличаться. При использовании Azure CLI сохраните следующие параметры в ProtectedSettings.jsи PublicSettings.jsв для использования с приведенной выше командой Sample. При использовании PowerShell сохраните параметры в `$protectedSettings` и `$publicSettings` , выполнив `$protectedSettings = '{ ... }'` .

### <a name="protected-settings"></a>Защищенные параметры

Эти защищенные параметры настраивают:

* учетную запись хранения;
* соответствующий токен SAS учетной записи;
* несколько приемников (JsonBlob или EventHubs с токенами SAS).

```json
{
  "storageAccountName": "yourdiagstgacct",
  "storageAccountSasToken": "sv=xxxx-xx-xx&ss=bt&srt=co&sp=wlacu&st=yyyy-yy-yyT21%3A22%3A00Z&se=zzzz-zz-zzT21%3A22%3A00Z&sig=fake_signature",
  "sinksConfig": {
    "sink": [
      {
        "name": "SyslogJsonBlob",
        "type": "JsonBlob"
      },
      {
        "name": "FilelogJsonBlob",
        "type": "JsonBlob"
      },
      {
        "name": "LinuxCpuJsonBlob",
        "type": "JsonBlob"
      },
      {
        "name": "MyJsonMetricsBlob",
        "type": "JsonBlob"
      },
      {
        "name": "LinuxCpuEventHub",
        "type": "EventHub",
        "sasURL": "https://youreventhubnamespace.servicebus.windows.net/youreventhubpublisher?sr=https%3a%2f%2fyoureventhubnamespace.servicebus.windows.net%2fyoureventhubpublisher%2f&sig=fake_signature&se=1808096361&skn=yourehpolicy"
      },
      {
        "name": "MyMetricEventHub",
        "type": "EventHub",
        "sasURL": "https://youreventhubnamespace.servicebus.windows.net/youreventhubpublisher?sr=https%3a%2f%2fyoureventhubnamespace.servicebus.windows.net%2fyoureventhubpublisher%2f&sig=yourehpolicy&skn=yourehpolicy"
      },
      {
        "name": "LoggingEventHub",
        "type": "EventHub",
        "sasURL": "https://youreventhubnamespace.servicebus.windows.net/youreventhubpublisher?sr=https%3a%2f%2fyoureventhubnamespace.servicebus.windows.net%2fyoureventhubpublisher%2f&sig=yourehpolicy&se=1808096361&skn=yourehpolicy"
      }
    ]
  }
}
```

### <a name="public-settings"></a>Общедоступные параметры

Эти общедоступные параметры обеспечивают выполнение расширением LAD следующих действий:

* отправка метрик процента процессорного времени и используемого дискового пространства в таблицу `WADMetrics*`;
* отправка сообщений из субъекта системного журнала user и сведений об уровне серьезности в таблицу `LinuxSyslog*`;
* отправка строк, добавленных к файлу `/var/log/myladtestlog`, в таблицу `MyLadTestLog`.

В каждом случае данные также передаются в следующие места:

* хранилище BLOB-объектов Azure (имя контейнера, определенное в приемнике JsonBlob);
* конечную точку EventHubs (указанную в приемнике EventHubs).

```json
{
  "StorageAccount": "yourdiagstgacct",
  "ladCfg": {
    "sampleRateInSeconds": 15,
    "diagnosticMonitorConfiguration": {
      "performanceCounters": {
        "sinks": "MyMetricEventHub,MyJsonMetricsBlob",
        "performanceCounterConfiguration": [
          {
            "unit": "Percent",
            "type": "builtin",
            "counter": "PercentProcessorTime",
            "counterSpecifier": "/builtin/Processor/PercentProcessorTime",
            "annotation": [
              {
                "locale": "en-us",
                "displayName": "Aggregate CPU %utilization"
              }
            ],
            "condition": "IsAggregate=TRUE",
            "class": "Processor"
          },
          {
            "unit": "Bytes",
            "type": "builtin",
            "counter": "UsedSpace",
            "counterSpecifier": "/builtin/FileSystem/UsedSpace",
            "annotation": [
              {
                "locale": "en-us",
                "displayName": "Used disk space on /"
              }
            ],
            "condition": "Name=\"/\"",
            "class": "Filesystem"
          }
        ]
      },
      "metrics": {
        "metricAggregation": [
          {
            "scheduledTransferPeriod": "PT1H"
          },
          {
            "scheduledTransferPeriod": "PT1M"
          }
        ],
        "resourceId": "/subscriptions/your_azure_subscription_id/resourceGroups/your_resource_group_name/providers/Microsoft.Compute/virtualMachines/your_vm_name"
      },
      "eventVolume": "Large",
      "syslogEvents": {
        "sinks": "SyslogJsonBlob,LoggingEventHub",
        "syslogEventConfiguration": {
          "LOG_USER": "LOG_INFO"
        }
      }
    }
  },
  "sinksConfig": {
    "sink": [
      {
        "name": "AzMonSink",
        "type": "AzMonSink",
        "AzureMonitor": {}
      }
    ]
  },
  "fileLogs": [
    {
      "file": "/var/log/myladtestlog",
      "table": "MyLadTestLog",
      "sinks": "FilelogJsonBlob,LoggingEventHub"
    }
  ]
}
```

Идентификатор `resourceId` в конфигурации должен соответствовать идентификатору виртуальной машины или масштабируемого набора виртуальных машин.

* Функции построения диаграмм и вывода оповещений на основе метрик платформы Azure известен идентификатор resourceId виртуальной машины, с которой вы работаете. Предполагается, что она находит данные для виртуальной машины, используя ключ поиска resourceId.
* При использовании автоматического масштабирования Azure значение resourceId в конфигурации автоматического масштабирования должно соответствовать значению resourceId, используемому расширением LAD.
* Идентификатор resourceId встроен в имена JsonBlob, которые записываются расширением LAD.

## <a name="view-your-data"></a>Просмотр данных

Для просмотра данных производительности или настройки оповещений используйте портал Azure:

:::image type="content" source="./media/diagnostics-linux/graph_metrics.png" alt-text="Снимок экрана показывает портал Azure с выбранным дисковым пространством для выбранной метрики и результирующей диаграммой.":::

Данные `performanceCounters` всегда хранятся в таблице службы хранилища Azure. Интерфейсы API службы хранилища Azure доступны для множества языков и платформ.

Данные, отправляемые в приемники JsonBlob, сохраняются в BLOB-объектах в учетной записи хранения, указанной в [защищенных параметрах](#protected-settings). Вы можете получать данные BLOB-объектов, используя любые интерфейсы API хранилища BLOB-объектов Azure.

Для доступа к данным в службе хранилища Azure также можно использовать следующие средства пользовательского интерфейса:

* Обозреватель сервера Visual Studio.
* [На снимке экрана показаны контейнеры и таблицы в обозреватель службы хранилища Azure.](https://azurestorageexplorer.codeplex.com/ "Обозреватель службы хранилища Azure").

На этом моментальном снимке сеанса обозревателя хранилищ Microsoft Azure показаны таблицы и контейнеры службы хранилища Azure, созданные в результате правильной настройки расширения LAD 3.0 в тестовой виртуальной машине. Это изображение не соответствует в точности [образцу конфигурации LAD 3.0](#an-example-lad-40-configuration).

:::image type="content" source="./media/diagnostics-linux/stg_explorer.png" alt-text="На снимке экрана показан Обозреватель службы хранилища Azure.":::

Сведения о получении сообщений, публикуемых в конечной точке EventHubs, см. в соответствующей [документации по EventHubs](../../event-hubs/event-hubs-about.md).

## <a name="next-steps"></a>Дальнейшие действия

* Создайте оповещения метрик в [Azure Monitor](../../azure-monitor/alerts/alerts-classic-portal.md) для собираемых метрик.
* Создайте [диаграммы мониторинга](../../azure-monitor/data-platform.md) для метрик.
* Узнайте, как [создать масштабируемый набор виртуальных машин](../linux/tutorial-create-vmss.md), используя метрики для управления автоматическим масштабированием.