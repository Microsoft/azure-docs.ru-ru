---
title: Развертывание виртуальных машин на устройстве Azure Stack ребра Pro с помощью шаблонов
description: В этой статье описывается создание виртуальных машин и управление ими на устройстве Azure Stack пограничной Pro с помощью шаблонов.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 02/22/2021
ms.author: alkohli
ms.openlocfilehash: 2922e9620f558e072cfc1264f7e661efd11a90fb
ms.sourcegitcommit: 5bbc00673bd5b86b1ab2b7a31a4b4b066087e8ed
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102437014"
---
# <a name="deploy-vms-on-your-azure-stack-edge-pro-gpu-device-via-templates"></a>Развертывание виртуальных машин на устройстве с Azure Stack ребра Pro GPU с помощью шаблонов

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

В этом учебнике описано, как создать виртуальную машину на устройстве Azure Stack пограничной Pro и управлять ею с помощью шаблонов. Эти шаблоны являются файлами нотация объектов JavaScript (JSON), которые определяют инфраструктуру и конфигурацию для виртуальной машины. В этих шаблонах указываются развертываемые ресурсы и свойства этих ресурсов.

Шаблоны являются гибкими в разных средах, так как они могут принимать параметры в качестве входных данных в среде выполнения из файла. Стандартная структура именования предназначена `TemplateName.json` для шаблона и `TemplateName.parameters.json` для файла параметров. Дополнительные сведения о шаблонах ARM см. в [разделе что такое Azure Resource Manager Templates?](../azure-resource-manager/templates/overview.md).

В этом учебнике мы будем использовать предварительно написанные примеры шаблонов для создания ресурсов. Вам не потребуется изменять файл шаблона, и вы можете изменить только файлы, `.parameters.json` чтобы настроить развертывание на компьютере. 

## <a name="vm-deployment-workflow"></a>Рабочий процесс развертывания виртуальной машины

Для развертывания Azure Stack пограничных виртуальных машинах Pro на нескольких устройствах можно использовать один виртуальный жесткий диск Sysprep для полного резерва, один и тот же шаблон для развертывания, а также внести незначительные изменения в параметры для каждого расположения развертывания (эти изменения могут быть произведены вручную, так как мы делаем это с помощью программного обеспечения). 

Общая сводка по рабочему процессу развертывания с использованием шаблонов выглядит следующим образом:

1. **Настройка необходимых компонентов** . Существуют три типа необходимых компонентов: устройство, клиент и виртуальная машина.

    1. **Предварительные требования для устройства**

        1. Подключение к Azure Resource Manager на устройстве.
        2. Включение вычислений через локальный пользовательский интерфейс.

    2. **Требования к клиенту**

        1. Скачайте шаблоны виртуальных машин и связанные файлы на клиенте.
        1. При необходимости настройте TLS 1,2 на клиенте.
        1. [Скачайте и установите обозреватель службы хранилища Microsoft Azure](https://go.microsoft.com/fwlink/?LinkId=708343&clcid=0x409) на клиенте.

    3. **Необходимые компоненты для виртуальной машины**

        1. Создайте группу ресурсов в расположении устройства, которая будет содержать все ресурсы виртуальной машины.
        1. Создайте учетную запись хранения для отправки виртуального жесткого диска, используемого для создания образа виртуальной машины.
        1. Добавьте URI учетной записи локального хранилища в DNS или файл hosts на клиенте, обращающемся к устройству.
        1. Установите сертификат хранилища BLOB-объектов на устройстве и на локальном клиенте, обращающемся к устройству. При необходимости установите сертификат хранилища BLOB-объектов на Обозреватель службы хранилища.
        1. Создайте и отправьте виртуальный жесткий диск в созданную ранее учетную запись хранения.

2. **Создание виртуальной машины из шаблонов**

    1. Создайте образ виртуальной машины, используя `CreateImage.parameters.json` параметры файл и `CreateImage.json` шаблон развертывания.
    1. Создайте виртуальную машину с ранее созданными ресурсами с помощью `CreateVM.parameters.json` файла параметров и  `CreateVM.json` шаблона развертывания.

## <a name="device-prerequisites"></a>Предварительные требования для устройства

Настройте эти компоненты на устройстве Azure Stack пограничной Pro.

[!INCLUDE [azure-stack-edge-gateway-deploy-virtual-machine-prerequisites](../../includes/azure-stack-edge-gateway-deploy-virtual-machine-prerequisites.md)]

## <a name="client-prerequisites"></a>Требования к клиенту

Настройте эти компоненты на клиенте, которые будут использоваться для доступа к устройству Azure Stack ребра Pro.

1. [Скачайте обозреватель службы хранилища](https://azure.microsoft.com/features/storage-explorer/) , если вы используете его для отправки виртуального жесткого диска. Кроме того, можно скачать AzCopy для отправки виртуального жесткого диска. При работе с предыдущими версиями AzCopy может потребоваться настроить TLS 1,2 на клиентском компьютере. 
1. [Скачайте шаблоны виртуальных машин и файлы параметров](https://aka.ms/ase-vm-templates) на клиентский компьютер. Распакуйте его в каталог, который будет использоваться в качестве рабочего каталога.


## <a name="vm-prerequisites"></a>Необходимые компоненты для виртуальной машины

Настройте эти компоненты, чтобы создать ресурсы, необходимые для создания виртуальной машины. 

    
### <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте группу ресурсов Azure с помощью командлета [New-AzureRmResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Группа ресурсов — это логический контейнер, в который развертываются и управляются ресурсы Azure, такие как учетная запись хранения, диск, управляемый диск.

> [!IMPORTANT]
> Все ресурсы создаются в том же расположении, что и устройство, а для расположения задано значение **дбелокал**.

```powershell
New-AzureRmResourceGroup -Name <Resource group name> -Location DBELocal
```

Результат выполнения команды показан ниже.

```powershell
PS C:\windows\system32> New-AzureRmResourceGroup -Name myasegpurgvm -Location DBELocal

ResourceGroupName : myasegpurgvm
Location          : dbelocal
ProvisioningState : Succeeded
Tags              :
ResourceId        : /subscriptions/DDF9FC44-E990-42F8-9A91-5A6A5CC472DB/resourceGroups/myasegpurgvm

PS C:\windows\system32>
```

### <a name="create-a-storage-account"></a>Создание учетной записи хранения

Создайте новую учетную запись хранения с помощью группы ресурсов, созданной на предыдущем шаге. Эта учетная запись представляет собой **локальную учетную запись хранения** , которая будет использоваться для отправки образа виртуального диска для виртуальной машины.

```powershell
New-AzureRmStorageAccount -Name <Storage account name> -ResourceGroupName <Resource group name> -Location DBELocal -SkuName Standard_LRS
```

> [!NOTE]
> Только локальные учетные записи хранения, такие как локально избыточное хранилище (Standard_LRS или Premium_LRS), можно создавать с помощью Azure Resource Manager. Чтобы создать многоуровневые учетные записи хранения, см. действия в разделе [Добавление и подключение к учетным записям хранения на Azure Stack пограничных Pro](azure-stack-edge-j-series-deploy-add-storage-accounts.md).

Результат выполнения команды показан ниже.

```powershell
PS C:\windows\system32> New-AzureRmStorageAccount -Name myasegpusavm -ResourceGroupName myasegpurgvm -Location DBELocal -SkuName Standard_LRS

StorageAccountName ResourceGroupName Location SkuName     Kind    AccessTier CreationTime
------------------ ----------------- -------- -------     ----    ---------- ------------
myasegpusavm       myasegpurgvm      DBELocal StandardLRS Storage            7/29/2020 10:11:16 PM

PS C:\windows\system32>
```

Чтобы получить ключ учетной записи хранения, выполните `Get-AzureRmStorageAccountKey` команду. Ниже показан пример выходных данных этой команды.

```powershell
PS C:\windows\system32> Get-AzureRmStorageAccountKey

cmdlet Get-AzureRmStorageAccountKey at command pipeline position 1
Supply values for the following parameters:
(Type !? for Help.)
ResourceGroupName: myasegpurgvm
Name: myasegpusavm

KeyName    Value                                                  Permissions   
-------     -----                                                   --
key1 GsCm7QriXurqfqx211oKdfQ1C9Hyu5ZutP6Xl0dqlNNhxLxDesDej591M8y7ykSPN4fY9vmVpgc4ftkwAO7KQ== 11 
key2 7vnVMJUwJXlxkXXOyVO4NfqbW5e/5hZ+VOs+C/h/ReeoszeV+qoyuBitgnWjiDPNdH4+lSm1/ZjvoBWsQ1klqQ== ll
```

### <a name="add-blob-uri-to-hosts-file"></a>Добавление URI BLOB-объекта в файл hosts.

Убедитесь, что вы уже добавили URI большого двоичного объекта в файл hosts для клиента, который используется для подключения к хранилищу BLOB-объектов. **Запустите Блокнот от имени администратора** и добавьте следующую запись для URI большого двоичного объекта в `C:\windows\system32\drivers\etc\hosts` :

`<Device IP> <storage account name>.blob.<Device name>.<DNS domain>`

В типичной среде DNS настроена таким образом, что все учетные записи хранения будут указывать на устройство Azure Stack пограничной Pro с `*.blob.devicename.domainname.com` записью.

### <a name="optional-install-certificates"></a>Используемых Установка сертификатов

Пропустите этот шаг, если подключение будет осуществляться через Обозреватель службы хранилища по *протоколу HTTP*. Если используется *протокол HTTPS*, необходимо установить соответствующие сертификаты в обозреватель службы хранилища. В этом случае установите сертификат конечной точки большого двоичного объекта. Дополнительные сведения см. в статье Создание и передача сертификатов в оснастке [Управление сертификатами](azure-stack-edge-gpu-manage-certificates.md). 

### <a name="create-and-upload-a-vhd"></a>Создание и передача виртуального жесткого диска

Убедитесь, что у вас есть образ виртуального диска, который можно использовать для отправки на более позднем этапе. Выполните действия, описанные в разделе [Создание образа виртуальной машины](azure-stack-edge-gpu-create-virtual-machine-image.md). 

Скопируйте все образы дисков, которые будут использоваться, в страничные BLOB-объекты в локальной учетной записи хранения, созданной на предыдущих шагах. Для [отправки виртуального жесткого диска в учетную запись хранения](azure-stack-edge-gpu-deploy-virtual-machine-powershell.md#upload-a-vhd) , созданную на предыдущих шагах, можно использовать такие средства, как [Обозреватель службы хранилища](https://azure.microsoft.com/features/storage-explorer/) или AzCopy. 

### <a name="use-storage-explorer-for-upload"></a>Использовать Обозреватель службы хранилища для отправки

1. Откройте Обозреватель службы хранилища. Перейдите в раздел **Edit (изменить** ) и убедитесь, что для приложения задано значение **Target Azure Stack API**.

    ![Задайте для параметра target значение Azure Stack API](media/azure-stack-edge-gpu-deploy-virtual-machine-templates/set-target-apis-1.png)

2. Установите сертификат клиента в формате PEM. Перейдите к разделу **Edit > SSL certificates > Import Certificates**. Укажите сертификат клиента.

    ![Импорт сертификата конечной точки хранилища BLOB-объектов](media/azure-stack-edge-gpu-deploy-virtual-machine-templates/import-blob-storage-endpoint-certificate-1.png)

    - Если вы используете сертификаты, создаваемые устройством, скачайте и преобразуйте сертификат конечной точки хранилища BLOB `.cer` -объектов в `.pem` Формат. Выполните следующую команду. 
    
        ```powershell
        PS C:\windows\system32> Certutil -encode 'C:\myasegpu1_Blob storage (1).cer' .\blobstoragecert.pem
        Input Length = 1380
        Output Length = 1954
        CertUtil: -encode command completed successfully.
        ```
    - Если вы используете собственный сертификат, используйте корневой сертификат цепочки подписывания в `.pem` формате.

3. После импорта сертификата перезапустите Обозреватель службы хранилища, чтобы изменения вступили в силу.

    ![Перезапуск Обозревателя службы хранилища](media/azure-stack-edge-gpu-deploy-virtual-machine-templates/restart-storage-explorer-1.png)

4. В левой области щелкните правой кнопкой мыши элемент **учетные записи хранения** и выберите **подключиться к службе хранилища Azure**. 

    ![Подключение к службе хранилища Azure 1](media/azure-stack-edge-gpu-deploy-virtual-machine-templates/connect-azure-storage-1.png)

5. Выберите параметр **Использовать имя и ключ учетной записи хранения**. Выберите **Далее**.

    ![Подключение к службе хранилища Azure 2](media/azure-stack-edge-gpu-deploy-virtual-machine-templates/connect-azure-storage-2.png)

6. В поле **подключение с именем и ключом** укажите **Отображаемое имя**, **имя учетной записи хранения**, **ключ учетной записи** хранения Azure. Выберите **другой** домен хранилища и укажите `<device name>.<DNS domain>` строку подключения. Если сертификат не был установлен в Обозреватель службы хранилища, установите флажок **использовать HTTP** . Выберите **Далее**.

    ![Подключение с помощью имени и ключа](media/azure-stack-edge-gpu-deploy-virtual-machine-templates/connect-name-key-1.png)

7. Проверьте **сводку подключения** и нажмите кнопку **подключить**.

8. Учетная запись хранения появится на левой панели. Выберите и разверните учетную запись хранения. Выберите **контейнеры больших двоичных объектов**, щелкните правой кнопкой мыши и выберите **создать контейнер BLOB-объектов**. Укажите имя для контейнера больших двоичных объектов.

9. Выберите только что созданный контейнер и в правой области выберите **отправить > отправить файлы**. 

    ![Отправить VHD-файл 1](media/azure-stack-edge-gpu-deploy-virtual-machine-templates/upload-vhd-file-1.png)

10. Перейдите к виртуальному жесткому диску, который необходимо отправить в **выбранных файлах**, и укажите его. Выберите **Тип большого двоичного** объекта **страничный BLOB-объект** и щелкните **Отправить**.

    ![Отправить VHD-файл 2](media/azure-stack-edge-gpu-deploy-virtual-machine-templates/upload-vhd-file-2.png)

11. После загрузки виртуального жесткого диска в контейнер больших двоичных объектов выберите виртуальный жесткий диск, щелкните его правой кнопкой мыши и выберите пункт **Свойства**. 

    ![Отправить VHD-файл 3](media/azure-stack-edge-gpu-deploy-virtual-machine-templates/upload-vhd-file-3.png)

12. Скопируйте и сохраните **URI**, который будет использоваться на последующих шагах.

    ![Копировать URI](media/azure-stack-edge-gpu-deploy-virtual-machine-templates/copy-uri-1.png)


## <a name="create-image-for-your-vm"></a>Создание образа для виртуальной машины

Чтобы создать образ для виртуальной машины, измените `CreateImage.parameters.json` файл параметров, а затем разверните шаблон `CreateImage.json` , который использует этот файл параметров.


### <a name="edit-parameters-file"></a>Изменение файла параметров

Файл `CreateImage.parameters.json` принимает следующие параметры: 

```json
"parameters": {
        "osType": {
              "value": "<Operating system corresponding to the VHD you upload can be Windows or Linux>"
        },
        "imageName": {
            "value": "<Name for the VM image>"
        },
        "imageUri": {
              "value": "<Path to the VHD that you uploaded in the Storage account>"
        },
    }
```

Измените файл, `CreateImage.parameters.json` включив в него следующие значения для устройства Azure Stack ребра Pro:

1. Укажите тип операционной системы, соответствующий загружаемому виртуальному жесткому диску. Типом операционной системы может быть Windows или Linux.

    ```json
    "parameters": {
            "osType": {
              "value": "Windows"
            },
    ```

2. Измените URI образа на URI образа, отправленного на предыдущем шаге:

   ```json
   "imageUri": {
       "value": "https://myasegpusavm.blob.myasegpu1.wdshcsso.com/windows/WindowsServer2016Datacenter.vhd"
       },
   ```

   Если вы используете *http* с обозреватель службы хранилища, измените универсальный код ресурса (URI) на *http* .

3. Укажите уникальное имя образа. Этот образ используется для создания виртуальной машины в последующих шагах. 

   Ниже приведен пример JSON, который используется в этой статье.

    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
        "contentVersion": "1.0.0.0",
      "parameters": {
        "osType": {
          "value": "Linux"
        },
        "imageName": {
          "value": "myaselinuximg"
        },
        "imageUri": {
          "value": "https://sa2.blob.myasegpuvm.wdshcsso.com/con1/ubuntu18.04waagent.vhd"
        }
      }
    }
    ```

5. Сохраните файл параметров.


### <a name="deploy-template"></a>Развертывание шаблона 

Разверните шаблон `CreateImage.json` . Этот шаблон развертывает ресурсы образа, которые будут использоваться для создания виртуальных машин на следующем шаге.

> [!NOTE]
> При развертывании шаблона при возникновении ошибки проверки подлинности учетные данные Azure для этого сеанса могли быть просрочены. Повторно выполните `login-AzureRM` команду, чтобы снова подключиться к Azure Resource Manager на устройстве Azure Stack ребра Pro.

1. Выполните следующую команду: 
    
    ```powershell
    $templateFile = "Path to CreateImage.json"
    $templateParameterFile = "Path to CreateImage.parameters.json"
    $RGName = "<Name of your resource group>"
    New-AzureRmResourceGroupDeployment `
        -ResourceGroupName $RGName `
        -TemplateFile $templateFile `
        -TemplateParameterFile $templateParameterFile `
        -Name "<Name for your deployment>"
    ```
    Эта команда развертывает ресурс изображения. Чтобы запросить ресурс, выполните следующую команду:

    ```powershell
    Get-AzureRmImage -ResourceGroupName <Resource Group Name> -name <Image Name>
    ``` 
    Ниже приведен пример выходных данных успешно созданного образа.
    
    ```powershell
    PS C:\WINDOWS\system32> login-AzureRMAccount -EnvironmentName aztest -TenantId c0257de7-538f-415c-993a-1b87a031879d
    
    Account               SubscriptionName              TenantId                             Environment
    -------               ----------------              --------                             -----------
    EdgeArmUser@localhost Default Provider Subscription c0257de7-538f-415c-993a-1b87a031879d aztest
    
   PS C:\WINDOWS\system32> $templateFile = "C:\12-09-2020\CreateImage\CreateImage.json"
    PS C:\WINDOWS\system32> $templateParameterFile = "C:\12-09-2020\CreateImage\CreateImage.parameters.json"
    PS C:\WINDOWS\system32> $RGName = "rg2"
    PS C:\WINDOWS\system32> New-AzureRmResourceGroupDeployment -ResourceGroupName $RGName -TemplateFile $templateFile -TemplateParameterFile $templateParameterFile -Name "deployment4"
        
    DeploymentName          : deployment4
    ResourceGroupName       : rg2
    ProvisioningState       : Succeeded
    Timestamp               : 12/10/2020 7:06:57 PM
    Mode                    : Incremental
    TemplateLink            :
    Parameters              :
                              Name             Type                       Value
                              ===============  =========================  ==========
                              osType           String                     Linux
                              imageName        String                     myaselinuximg
                              imageUri         String
                              https://sa2.blob.myasegpuvm.wdshcsso.com/con1/ubuntu18.04waagent.vhd
    
    Outputs                 :
    DeploymentDebugLogLevel :    
    PS C:\WINDOWS\system32>
    ```
    
## <a name="create-vm"></a>Создание виртуальной машины

### <a name="edit-parameters-file-to-create-vm"></a>Изменение файла параметров для создания виртуальной машины
 
Чтобы создать виртуальную машину, используйте файл параметров `CreateVM.parameters.json`. Он принимает следующие параметры.
    
```json
"vmName": {
            "value": "<Name for your VM>"
        },
        "adminUsername": {
            "value": "<Username to log into the VM>"
        },
        "Password": {
            "value": "<Password to log into the VM>"
        },
        "imageName": {
            "value": "<Name for your image>"
        },
        "vmSize": {
            "value": "<A supported size for your VM>"
        },
        "vnetName": {
            "value": "<Name for the virtual network, use ASEVNET>"
        },
        "subnetName": {
            "value": "<Name for the subnet, use ASEVNETsubNet>"
        },
        "vnetRG": {
            "value": "<Resource group for Vnet, use ASERG>"
        },
        "nicName": {
            "value": "<Name for the network interface>"
        },
        "privateIPAddress": {
            "value": "<Private IP address, enter a static IP in the subnet created earlier or leave empty to assign DHCP>"
        },
        "IPConfigName": {
            "value": "<Name for the ipconfig associated with the network interface>"
        }
```    

Назначьте соответствующие параметры в `CreateVM.parameters.json` для устройства Azure Stack ребра Pro.

1. Укажите уникальное имя, имя сетевого интерфейса и имя ipconfig. 
1. Введите имя пользователя, пароль и поддерживаемый размер виртуальной машины.
1. При включении сетевого интерфейса для вычислений виртуальный коммутатор и виртуальная сеть были автоматически созданы на этом сетевом интерфейсе. Вы можете запросить существующую виртуальную сеть, чтобы получить имя виртуальной сети, ее имя и имя группы ресурсов виртуальной сети.

    Выполните следующую команду:

    ```powershell
    Get-AzureRmVirtualNetwork
    ```
    Пример выходных данных:
    
    ```powershell
    
    PS C:\WINDOWS\system32> Get-AzureRmVirtualNetwork
    
    Name                   : ASEVNET
    ResourceGroupName      : ASERG
    Location               : dbelocal
    Id                     : /subscriptions/947b3cfd-7a1b-4a90-7cc5-e52caf221332/resourceGroups/ASERG/providers/Microsoft
                             .Network/virtualNetworks/ASEVNET
    Etag                   : W/"990b306d-18b6-41ea-a456-b275efe21105"
    ResourceGuid           : f8309d81-19e9-42fc-b4ed-d573f00e61ed
    ProvisioningState      : Succeeded
    Tags                   :
    AddressSpace           : {
                               "AddressPrefixes": [
                                 "10.57.48.0/21"
                               ]
                             }
    DhcpOptions            : null
    Subnets                : [
                               {
                                 "Name": "ASEVNETsubNet",
                                 "Etag": "W/\"990b306d-18b6-41ea-a456-b275efe21105\"",
                                 "Id": "/subscriptions/947b3cfd-7a1b-4a90-7cc5-e52caf221332/resourceGroups/ASERG/provider
                             s/Microsoft.Network/virtualNetworks/ASEVNET/subnets/ASEVNETsubNet",
                                 "AddressPrefix": "10.57.48.0/21",
                                 "IpConfigurations": [],
                                 "ResourceNavigationLinks": [],
                                 "ServiceEndpoints": [],
                                 "ProvisioningState": "Succeeded"
                               }
                             ]
    VirtualNetworkPeerings : []
    EnableDDoSProtection   : false
    EnableVmProtection     : false
    
    PS C:\WINDOWS\system32>
    ```

    Используйте АСЕВНЕТ для имени виртуальной сети, Асевнетсубнет для Name Subnet и АСЕРГ для имени группы ресурсов виртуальной сети.
    
1. Теперь вам потребуется статический IP-адрес для назначения виртуальной машине, которая находится в сети подсети, определенной выше. Замените **PrivateIPAddress** на этот адрес в файле параметров. Чтобы виртуальная машина получила IP-адрес от локального сервера DCHP, оставьте `privateIPAddress` значение пустым.  
    
    ```json
    "privateIPAddress": {
                "value": "5.5.153.200"
            },
    ```
    
4. Сохраните файл параметров.

    Ниже приведен пример JSON, который используется в этой статье.
    
    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
          "vmName": {
              "value": "VM1"
          },
          "adminUsername": {
              "value": "Administrator"
          },
          "Password": {
              "value": "Password1"
          },
        "imageName": {
          "value": "myaselinuximg"
        },
        "vmSize": {
          "value": "Standard_NC4as_T4_v3"
        },
        "vnetName": {
          "value": "ASEVNET"
        },
        "subnetName": {
          "value": "ASEVNETsubNet"
        },
        "vnetRG": {
          "value": "aserg"
        },
        "nicName": {
          "value": "nic5"
        },
        "privateIPAddress": {
          "value": ""
        },
        "IPConfigName": {
          "value": "ipconfig5"
        }
      }
    }
    ```      

### <a name="deploy-template-to-create-vm"></a>Развертывание шаблона для создания виртуальной машины

Разверните шаблон создания виртуальной машины `CreateVM.json` . Этот шаблон создает сетевой интерфейс из существующей виртуальной сети и создает виртуальную машину из развернутого образа.

1. Выполните следующую команду: 
    
    ```powershell
    Command:
        
        $templateFile = "<Path to CreateVM.json>"
        $templateParameterFile = "<Path to CreateVM.parameters.json>"
        $RGName = "<Resource group name>"
             
        New-AzureRmResourceGroupDeployment `
            -ResourceGroupName $RGName `
            -TemplateFile $templateFile `
            -TemplateParameterFile $templateParameterFile `
            -Name "<DeploymentName>"
    ```   

    Создание виртуальной машины займет 15-20 минут. Ниже приведен пример выходных данных успешно созданной виртуальной машины.
    
    ```powershell
    PS C:\WINDOWS\system32> $templateFile = "C:\12-09-2020\CreateVM\CreateVM.json"
    PS C:\WINDOWS\system32> $templateParameterFile = "C:\12-09-2020\CreateVM\CreateVM.parameters.json"
    PS C:\WINDOWS\system32> $RGName = "rg2"
    PS C:\WINDOWS\system32> New-AzureRmResourceGroupDeployment -ResourceGroupName $RGName -TemplateFile $templateFile -TemplateParameterFile $templateParameterFile -Name "Deployment6"
       
    DeploymentName          : Deployment6
    ResourceGroupName       : rg2
    ProvisioningState       : Succeeded
    Timestamp               : 12/10/2020 7:51:28 PM
    Mode                    : Incremental
    TemplateLink            :
    Parameters              :
                              Name             Type                       Value
                              ===============  =========================  ==========
                              vmName           String                     VM1
                              adminUsername    String                     Administrator
                              password         String                     Password1
                              imageName        String                     myaselinuximg
                              vmSize           String                     Standard_NC4as_T4_v3
                              vnetName         String                     ASEVNET
                              vnetRG           String                     aserg
                              subnetName       String                     ASEVNETsubNet
                              nicName          String                     nic5
                              ipConfigName     String                     ipconfig5
                              privateIPAddress  String
    
    Outputs                 :
    DeploymentDebugLogLevel :
    
    PS C:\WINDOWS\system32
    ```   

    Можно также выполнить `New-AzureRmResourceGroupDeployment` команду асинхронно с `–AsJob` параметром. Ниже приведен пример выходных данных при выполнении командлета в фоновом режиме. Затем можно запросить состояние задания, созданного с помощью `Get-Job` командлета.

    ```powershell   
    PS C:\WINDOWS\system32> New-AzureRmResourceGroupDeployment `
    >>     -ResourceGroupName $RGName `
    >>     -TemplateFile $templateFile `
    >>     -TemplateParameterFile $templateParameterFile `
    >>     -Name "Deployment2" `
    >>     -AsJob
     
    Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
    --     ----            -------------   -----         -----------     --------             -------
    2      Long Running... AzureLongRun... Running       True            localhost            New-AzureRmResourceGro...
     
    PS C:\WINDOWS\system32> Get-Job -Id 2
     
    Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
    --     ----            -------------   -----         -----------     --------             -------
    ```

7. Убедитесь, что виртуальная машина успешно подготовлена. Выполните следующую команду:

    `Get-AzureRmVm`


## <a name="connect-to-a-vm"></a>Подключение к виртуальной машине

В зависимости от того, была создана виртуальная машина Windows или Linux, действия для подключения могут отличаться.

### <a name="connect-to-windows-vm"></a>Подключение к виртуальной машине Windows

Выполните следующие действия, чтобы подключиться к виртуальной машине Windows.

[!INCLUDE [azure-stack-edge-gateway-connect-vm](../../includes/azure-stack-edge-gateway-connect-virtual-machine-windows.md)]

### <a name="connect-to-linux-vm"></a>Подключение к виртуальной машине Linux

Выполните следующие действия, чтобы подключиться к виртуальной машине Linux.

[!INCLUDE [azure-stack-edge-gateway-connect-vm](../../includes/azure-stack-edge-gateway-connect-virtual-machine-linux.md)]



## <a name="next-steps"></a>Дальнейшие действия

[Командлеты Azure Resource Manager](/powershell/module/azurerm.resources/?view=azurermps-6.13.0&preserve-view=true)