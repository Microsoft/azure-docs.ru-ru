---
title: Создание образов виртуальных машин Azure Linux с помощью пакета Pack
description: Сведения об использовании Packer для создания образов виртуальных машин Linux в Azure
author: cynthn
ms.service: virtual-machines
ms.subservice: imaging
ms.topic: how-to
ms.workload: infrastructure
ms.date: 05/07/2019
ms.author: cynthn
ms.collection: linux
ms.openlocfilehash: 4d85106cb78b5f4799a78d18463d83594f54dbac
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102556746"
---
# <a name="how-to-use-packer-to-create-linux-virtual-machine-images-in-azure"></a>Создание образов виртуальных машин Linux в Azure с помощью Packer
Каждая виртуальная машина в Azure создается из образа, определяющего дистрибутив Linux и версию операционной системы. Образы могут содержать предварительно установленные приложения и конфигурации. Azure Marketplace предоставляет большое количество образов Майкрософт и сторонних разработчиков для наиболее распространенных операционных систем и приложений. Кроме того, вы можете создать собственные настраиваемые образы, отвечающие конкретным потребностям. В этой статье описывается определение и создание пользовательских образов в Azure с использованием средства с открытым кодом [Packer](https://www.packer.io/).

> [!NOTE]
> В Azure теперь есть служба "Конструктор образов Azure" (предварительная версия), которая используется для определения и создания собственных пользовательских образов. Так как Конструктор образов Azure создан на основе Packer, вы можете даже использовать имеющиеся скрипты подготовки оболочки Packer. Чтобы приступить к работе с построителем образов Azure, см. статью [Создание виртуальной машины Linux с помощью Azure Image Builder](image-builder.md).


## <a name="create-azure-resource-group"></a>Создание группы ресурсов Azure
В процессе сборки исходной виртуальной машины Packer создает временные ресурсы Azure. Чтобы сохранить эту исходную виртуальную машину для использования в качестве образа, необходимо определить группу ресурсов. Выходные данные процесса сборки Packer хранятся в этой группе ресурсов.

Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group). В следующем примере создается группа ресурсов с именем *myResourceGroup* в расположении *eastus*.

```azurecli
az group create -n myResourceGroup -l eastus
```


## <a name="create-azure-credentials"></a>Создание учетных данных Azure
Packer выполняет проверку подлинности с помощью субъекта-службы Azure. Субъект-служба Azure является удостоверением безопасности, которое можно использовать с приложениями, службами и средствами автоматизации, такими как Packer. Вы можете определять разрешения на то, какие операции может выполнять субъект-служба в Azure, и управлять ими.

Создайте субъект-службу с помощью командлета [az ad sp create-for-rbac](/cli/azure/ad/sp) и выведите учетные данные, необходимые для Packer:

```azurecli
az ad sp create-for-rbac --query "{ client_id: appId, client_secret: password, tenant_id: tenant }"
```

Ниже приведен пример выходных данных предыдущей команды.

```output
{
    "client_id": "f5b6a5cf-fbdf-4a9f-b3b8-3c2cd00225a4",
    "client_secret": "0e760437-bf34-4aad-9f8d-870be799c55d",
    "tenant_id": "72f988bf-86f1-41af-91ab-2d7cd011db47"
}
```

Для проверки подлинности в Azure также необходимо получить идентификатор подписки Azure с помощью команды [az account show](/cli/azure/account):

```azurecli
az account show --query "{ subscription_id: id }"
```

Используйте выходные данные этих двух команд на следующем шаге.


## <a name="define-packer-template"></a>Определение шаблона Packer
Чтобы собрать образы, создайте шаблон в формате JSON. В шаблоне определите средства разработки и подготовки, выполняющие процесс сборки. Packer использует [средство подготовки для Azure](https://www.packer.io/docs/builders/azure.html), которое позволяет определить ресурсы Azure, такие как учетные данные субъекта-службы, созданные на предыдущем шаге.

Создайте файл с именем *ubuntu.json* и вставьте следующее содержимое. Введите свои значения следующим образом:

| Параметр                           | Где можно получить |
|-------------------------------------|----------------------------------------------------|
| *client_id*                         | Первая строка выходных данных из `az ad sp` создает команду *appId* |
| *client_secret*                     | Вторая строка выходных данных из `az ad sp` создает команду *password* |
| *tenant_id*                         | Третья строка выходных данных из `az ad sp` создает команду *tenant* |
| *subscription_id*                   | Выходные данные команды `az account show` |
| *managed_image_resource_group_name* | Имя группы ресурсов, созданной на первом шаге |
| *managed_image_name*                | Имя создаваемого образа управляемого диска |


```json
{
  "builders": [{
    "type": "azure-arm",

    "client_id": "f5b6a5cf-fbdf-4a9f-b3b8-3c2cd00225a4",
    "client_secret": "0e760437-bf34-4aad-9f8d-870be799c55d",
    "tenant_id": "72f988bf-86f1-41af-91ab-2d7cd011db47",
    "subscription_id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx",

    "managed_image_resource_group_name": "myResourceGroup",
    "managed_image_name": "myPackerImage",

    "os_type": "Linux",
    "image_publisher": "Canonical",
    "image_offer": "UbuntuServer",
    "image_sku": "16.04-LTS",

    "azure_tags": {
        "dept": "Engineering",
        "task": "Image deployment"
    },

    "location": "East US",
    "vm_size": "Standard_DS2_v2"
  }],
  "provisioners": [{
    "execute_command": "chmod +x {{ .Path }}; {{ .Vars }} sudo -E sh '{{ .Path }}'",
    "inline": [
      "apt-get update",
      "apt-get upgrade -y",
      "apt-get -y install nginx",

      "/usr/sbin/waagent -force -deprovision+user && export HISTSIZE=0 && sync"
    ],
    "inline_shebang": "/bin/sh -x",
    "type": "shell"
  }]
}
```

Этот шаблон создает образ Ubuntu 16.04 LTS, устанавливает NGINX, а затем отзывает виртуальную машину.

> [!NOTE]
> Если вы расширите этот шаблон, чтобы подготовить учетные данные пользователя, настройте команду средства подготовки, которая отзывает агент Azure для чтения `-deprovision` вместо `deprovision+user`.
> Флаг `+user` удаляет все учетные записи пользователей из исходной виртуальной машины.


## <a name="build-packer-image"></a>Создание образа Packer
Если средство Packer еще не установлено на локальном компьютере, [следуйте инструкциям по его установке](https://www.packer.io/docs/install).

Создайте образ, указав файл шаблона Packer следующим образом:

```bash
./packer build ubuntu.json
```

Ниже приведен пример выходных данных предыдущей команды.

```output
azure-arm output will be in this color.

==> azure-arm: Running builder ...
    azure-arm: Creating Azure Resource Manager (ARM) client ...
==> azure-arm: Creating resource group ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-swtxmqm7ly’
==> azure-arm:  -> Location          : ‘East US’
==> azure-arm:  -> Tags              :
==> azure-arm:  ->> dept : Engineering
==> azure-arm:  ->> task : Image deployment
==> azure-arm: Validating deployment template ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-swtxmqm7ly’
==> azure-arm:  -> DeploymentName    : ‘pkrdpswtxmqm7ly’
==> azure-arm: Deploying deployment template ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-swtxmqm7ly’
==> azure-arm:  -> DeploymentName    : ‘pkrdpswtxmqm7ly’
==> azure-arm: Getting the VM’s IP address ...
==> azure-arm:  -> ResourceGroupName   : ‘packer-Resource-Group-swtxmqm7ly’
==> azure-arm:  -> PublicIPAddressName : ‘packerPublicIP’
==> azure-arm:  -> NicName             : ‘packerNic’
==> azure-arm:  -> Network Connection  : ‘PublicEndpoint’
==> azure-arm:  -> IP Address          : ‘40.76.218.147’
==> azure-arm: Waiting for SSH to become available...
==> azure-arm: Connected to SSH!
==> azure-arm: Provisioning with shell script: /var/folders/h1/ymh5bdx15wgdn5hvgj1wc0zh0000gn/T/packer-shell868574263
    azure-arm: WARNING! The waagent service will be stopped.
    azure-arm: WARNING! Cached DHCP leases will be deleted.
    azure-arm: WARNING! root password will be disabled. You will not be able to login as root.
    azure-arm: WARNING! /etc/resolvconf/resolv.conf.d/tail and /etc/resolvconf/resolv.conf.d/original will be deleted.
    azure-arm: WARNING! packer account and entire home directory will be deleted.
==> azure-arm: Querying the machine’s properties ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-swtxmqm7ly’
==> azure-arm:  -> ComputeName       : ‘pkrvmswtxmqm7ly’
==> azure-arm:  -> Managed OS Disk   : ‘/subscriptions/guid/resourceGroups/packer-Resource-Group-swtxmqm7ly/providers/Microsoft.Compute/disks/osdisk’
==> azure-arm: Powering off machine ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-swtxmqm7ly’
==> azure-arm:  -> ComputeName       : ‘pkrvmswtxmqm7ly’
==> azure-arm: Capturing image ...
==> azure-arm:  -> Compute ResourceGroupName : ‘packer-Resource-Group-swtxmqm7ly’
==> azure-arm:  -> Compute Name              : ‘pkrvmswtxmqm7ly’
==> azure-arm:  -> Compute Location          : ‘East US’
==> azure-arm:  -> Image ResourceGroupName   : ‘myResourceGroup’
==> azure-arm:  -> Image Name                : ‘myPackerImage’
==> azure-arm:  -> Image Location            : ‘eastus’
==> azure-arm: Deleting resource group ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-swtxmqm7ly’
==> azure-arm: Deleting the temporary OS disk ...
==> azure-arm:  -> OS Disk : skipping, managed disk was used...
Build ‘azure-arm’ finished.

==> Builds finished. The artifacts of successful builds are:
--> azure-arm: Azure.ResourceManagement.VMImage:

ManagedImageResourceGroupName: myResourceGroup
ManagedImageName: myPackerImage
ManagedImageLocation: eastus
```

Через несколько минут Packer создаст виртуальную машину, запустит средства подготовки и очистит развертывание.


## <a name="create-vm-from-azure-image"></a>Создание виртуальной машины на основе образа Azure
Теперь можно создать виртуальную машину из образа с помощью команды [az vm create](/cli/azure/vm). Укажите образ, созданный с помощью параметра `--image`. В следующем примере создаются виртуальная машина с именем *myVM* из образа *myPackerImage* и ключи SSH, если они еще не существуют.

```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image myPackerImage \
    --admin-username azureuser \
    --generate-ssh-keys
```

Если вы хотите создать виртуальные машины не в группе ресурсов или регионе, содержащем образ Packer, укажите идентификатор образа, а не его имя. Идентификатор образа можно получить, выполнив команду [az image show](/cli/azure/image#az-image-show).

Создание виртуальной машины занимает несколько минут. Когда виртуальная машина создана, запишите значение `publicIpAddress`, отображаемое в Azure CLI. Это адрес для доступа к сайту NGINX через веб-браузер.

Чтобы разрешить передачу веб-трафика для виртуальной машины, откройте порт 80 для Интернета командой [az vm open-port](/cli/azure/vm).

```azurecli
az vm open-port \
    --resource-group myResourceGroup \
    --name myVM \
    --port 80
```

## <a name="test-vm-and-nginx"></a>Тестирование виртуальной машины и NGINX
Теперь можно открыть веб-браузер и ввести в адресной строке `http://publicIpAddress`. Укажите собственный общедоступный IP-адрес, настроенный при создании виртуальной машины. Отобразится страница NGINX по умолчанию, как показано ниже.

![Сайт NGINX по умолчанию](./media/build-image-with-packer/nginx.png) 


## <a name="next-steps"></a>Дальнейшие действия
С помощью [Конструктора образов Azure](image-builder.md) можно также использовать имеющиеся скрипты подготовки Packer.
