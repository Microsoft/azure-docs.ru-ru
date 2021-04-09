---
title: Руководство. Применение пользовательского образа виртуальной машины в масштабируемом наборе с помощью Azure CLI
description: Сведения о создании пользовательского образа виртуальной машины, который можно использовать для развертывания масштабируемого набора виртуальных машин, с помощью Azure CLI
author: cynthn
ms.service: virtual-machine-scale-sets
ms.subservice: imaging
ms.topic: tutorial
ms.date: 05/01/2020
ms.author: cynthn
ms.custom: mvc, devx-track-azurecli
ms.reviewer: akjosh
ms.openlocfilehash: b12715e299f523d7ace56a72b0098b5d7ffac0ab
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98683060"
---
# <a name="tutorial-create-and-use-a-custom-image-for-virtual-machine-scale-sets-with-the-azure-cli"></a>Руководство по Создание и использование пользовательского образа для масштабируемых наборов виртуальных машин с помощью Azure CLI
Создавая масштабируемый набор, вы указываете образ для использования при развертывании экземпляров виртуальных машин. Чтобы сократить количество задач после развертывания экземпляров виртуальных машин, можно использовать пользовательский образ виртуальной машины. Этот образ содержит все необходимые установки или конфигурации приложения. Все экземпляры виртуальных машин, созданные в масштабируемом наборе, используют пользовательский образ виртуальной машины и готовы обслуживать трафик приложения. Из этого руководства вы узнаете, как выполнить следующие задачи:

> [!div class="checklist"]
> * Создание Общей коллекции образов.
> * Создание определения специализированного образа.
> * Создание версии образа
> * Создание масштабируемого набора на основе специализированного образа.
> * Предоставление общего доступа к коллекции образов.


[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.4.0 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="overview"></a>Обзор

[Коллекция общих образов](../virtual-machines/shared-image-galleries.md) упрощает обмен пользовательскими образами в вашей организации. Пользовательские образы похожи на образы магазина, однако их можно создавать самостоятельно. Пользовательские образы можно использовать для начальной загрузки конфигураций, например при предварительной загрузке приложений, конфигураций приложений и других конфигураций операционной системы. 

Общая коллекция образов позволяет предоставить другим пользователям общий доступ к пользовательским образам. Выберите образы, к которым нужно предоставить общий доступ, регионы, где они будут доступны, и пользователей, которым будет доступно совместное использование. 

## <a name="create-and-configure-a-source-vm"></a>Создание и настройка исходной виртуальной машины

Сначала создайте группу ресурсов с помощью команды [az group create](/cli/azure/group), а затем — виртуальную машину с помощью команды [az vm create](/cli/azure/vm). Позже эта виртуальная машина будет использована в качестве источника для образа. В следующем примере создается виртуальная машина с именем *myVM* в группе ресурсов *myResourceGroup*.

```azurecli-interactive
az group create --name myResourceGroup --location eastus

az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image ubuntults \
  --admin-username azureuser \
  --generate-ssh-keys
```

> [!IMPORTANT]
> **Идентификатор** виртуальной машины указан в выходных данных команды [az vm create](/cli/azure/vm). Скопируйте его в надежное место, так как он понадобиться нам позже.

Общедоступный IP-адрес виртуальной машины также указан в выходных данных команды [az vm create](/cli/azure/vm). Подключитесь по протоколу SSH к общедоступному IP-адресу виртуальной машины следующим образом.

```console
ssh azureuser@<publicIpAddress>
```

Чтобы настроить виртуальную машину, установите базовый веб-сервер. Когда экземпляр виртуальной машины в масштабируемом наборе будет развернут, у него будут все пакеты, необходимые для запуска веб-приложения. С помощью команды `apt-get` установите *NGINX*, как показано ниже.

```bash
sudo apt-get install -y nginx
```

По завершении введите `exit`, чтобы отключить SSH-подключение.

## <a name="create-an-image-gallery"></a>Создание коллекции образов 

Коллекция образов является основным ресурсом, который позволяет обмен изображениями. 

Допустимыми знаками для имени коллекции являются прописные или строчные буквы, цифры и точки. Имя коллекции не должно содержать дефисы.   Имена коллекций должны быть уникальным в пределах вашей подписки. 

Создайте коллекцию образов, используя команду [az sig create](/cli/azure/sig#az-sig-create). В следующем примере показано, как создать группу ресурсов с именем *myGalleryRG* в регионе *восточная часть США* и коллекцию с именем *myGallery*.

```azurecli-interactive
az group create --name myGalleryRG --location eastus
az sig create --resource-group myGalleryRG --gallery-name myGallery
```

## <a name="create-an-image-definition"></a>Создание определения образа

Образы можно объединять в логические группы с помощью определений образов. Определения образов используются для управления сведениями о версиях созданных в них образов. 

В имени определения образа можно использовать прописные и строчные буквы, цифры, точки и дефисы. 

Убедитесь, что используется определение образа подходящего типа. Если вы сделали виртуальную машину универсальной (с помощью Sysprep для Windows или waagent -deprovision для Linux), создайте универсальное определение образа с помощью `--os-state generalized`. Если вы хотите использовать виртуальную машину без удаления существующих учетных записей пользователей, создайте специализированное определение образа с помощью `--os-state specialized`.

Дополнительные сведения о значениях, которые можно указать для определения образа, см. в разделе [Определения образов](../virtual-machines/shared-image-galleries.md#image-definitions).

Создайте в коллекции определение образа, используя команду [az sig image-definition create](/cli/azure/sig/image-definition#az-sig-image-definition-create).

В этом примере определение образа имеет имя *myImageDefinition* и предназначено для [специализированного](../virtual-machines/shared-image-galleries.md#generalized-and-specialized-images) образа ОС Linux. Чтобы создать определение для образов с помощью ОС Windows, используйте `--os-type Windows`. 

```azurecli-interactive 
az sig image-definition create \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   --gallery-image-definition myImageDefinition \
   --publisher myPublisher \
   --offer myOffer \
   --sku mySKU \
   --os-type Linux \
   --os-state specialized
```

> [!IMPORTANT]
> **Идентификатор** определения образа указан в выходных данных команды. Скопируйте его в надежное место, так как он понадобиться нам позже.


## <a name="create-the-image-version"></a>Создание версии образа

Создайте версию образа на основе виртуальной машины, используя команду [az image gallery create-image-version](/cli/azure/sig/image-version#az-sig-image-version-create).  

Допустимыми знаками для имени версии образа являются цифры и точки. Числа должны быть в диапазоне 32-битного целого числа. Формат: *основной номер версии*.*дополнительный номер версии*.*исправление*.

В этом примере версия нашего образа — *1.0.0*, и мы собираемся создать 1 реплику в регионе *центрально-южная часть США* и 1 реплику в регионе *восточная часть США 2*. В числе регионов репликации должен быть регион, в котором находится исходная виртуальная машина.

Замените значение параметра `--managed-image` в этом примере на идентификатор вашей виртуальной машины из предыдущего шага.

```azurecli-interactive 
az sig image-version create \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   --gallery-image-definition myImageDefinition \
   --gallery-image-version 1.0.0 \
   --target-regions "southcentralus=1" "eastus=1" \
   --managed-image "/subscriptions/<Subscription ID>/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM"
```

> [!NOTE]
> Прежде чем использовать тот же управляемый образ для создания другой версии образа, необходимо дождаться завершения сборки и репликации версии образа.
>
> Вы также можете сохранить образ в хранилище класса Премиум, добавив `--storage-account-type  premium_lrs`, или [хранилище, избыточное между зонами](../storage/common/storage-redundancy.md), добавив `--storage-account-type  standard_zrs` при создании версии образа.
>




## <a name="create-a-scale-set-from-the-image"></a>Создание масштабируемого набора на основе образа
Создайте масштабируемый набор на основе специализированного образа, используя команду [`az vmss create`](/cli/azure/vmss#az-vmss-create). 

Создайте масштабируемый набор с помощью команды [`az vmss create`](/cli/azure/vmss#az-vmss-create), используя параметр --specialized, чтобы указать, что образ является специализированным. 

Используйте идентификатор определения образа в качестве значения параметра `--image`, чтобы создать экземпляры масштабируемого набора на основе последней доступной версии образа. Вы также можете создать экземпляры масштабируемого набора на основе определенной версии, указав идентификатор версии образа в параметре `--image`. 

Создайте масштабируемый набор с именем *myScaleSet* на основе последней версии созданного ранее образа *myImageDefinition*.

```azurecli
az group create --name myResourceGroup --location eastus
az vmss create \
   --resource-group myResourceGroup \
   --name myScaleSet \
   --image "/subscriptions/<Subscription ID>/resourceGroups/myGalleryRG/providers/Microsoft.Compute/galleries/myGallery/images/myImageDefinition" \
   --specialized
```

Создание и настройка всех ресурсов и виртуальных машин масштабируемого набора занимает несколько минут.


## <a name="test-your-scale-set"></a>Проверка масштабируемого набора
Чтобы разрешить передачу трафика в указанный масштабируемый набор и проверить работу веб-сервера, создайте правило подсистемы балансировки нагрузки с помощью команды [az network lb rule create](/cli/azure/network/lb/rule). В следующем примере создается правило *myLoadBalancerRuleWeb*, которое разрешает трафик через *TCP-порт* *80*.

```azurecli-interactive
az network lb rule create \
  --resource-group myResourceGroup \
  --name myLoadBalancerRuleWeb \
  --lb-name myScaleSetLB \
  --backend-pool-name myScaleSetLBBEPool \
  --backend-port 80 \
  --frontend-ip-name loadBalancerFrontEnd \
  --frontend-port 80 \
  --protocol tcp
```

Чтобы посмотреть, как работает масштабируемый набор, получите общедоступный IP-адрес подсистемы балансировки нагрузки с помощью команды [az network public-ip show](/cli/azure/network/public-ip). Следующий пример получает IP-адрес для *myScaleSetLBPublicIP*, созданного ранее вместе с масштабируемым набором.

```azurecli-interactive
az network public-ip show \
  --resource-group myResourceGroup \
  --name myScaleSetLBPublicIP \
  --query [ipAddress] \
  --output tsv
```

Введите общедоступный IP-адрес в веб-браузер. Отобразится веб-страница NGINX по умолчанию, как показано ниже.

![Nginx, запущенный из пользовательского образа виртуальной машины](media/tutorial-use-custom-image-cli/default-nginx-website.png)



## <a name="share-the-gallery"></a>Предоставление общего доступа к коллекции

Вы можете предоставить общий доступ к образам в разных подписках, используя механизм управления доступом на основе ролей Azure (Azure RBAC). Предоставлять общий доступ к образам можно на уровне коллекции, определения образа или версии образа. Любой пользователь с разрешениями на чтение версии образа, даже из другой подписки, сможет развернуть виртуальную машину с помощью версии образа.

Мы рекомендуем предоставлять общий доступ другим пользователям на уровне коллекции. Чтобы получить идентификатор объекта коллекции, используйте команду [az sig show](/cli/azure/sig#az-sig-show).

```azurecli-interactive
az sig show \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   --query id
```

Используйте идентификатор объекта в качестве области, а также адрес электронной почты и команду [az role assignment create](/cli/azure/role/assignment#az-role-assignment-create), чтобы предоставить пользователю доступ к общей коллекции образов. Замените `<email-address>` и `<gallery iD>` своими значениями.

```azurecli-interactive
az role assignment create \
   --role "Reader" \
   --assignee <email address> \
   --scope <gallery ID>
```

Дополнительные сведения о совместном использовании ресурсов с помощью Azure RBAC см. в статье [Добавление и удаление назначений ролей Azure с помощью Azure CLI](../role-based-access-control/role-assignments-cli.md).


## <a name="clean-up-resources"></a>Очистка ресурсов
Чтобы удалить масштабируемый набор и дополнительные ресурсы, удалите группу ресурсов и все входящие в нее ресурсы с помощью команды [az group delete](/cli/azure/group). При использовании параметра `--no-wait` управление возвращается в командную строку без ожидания завершения операции. Параметр `--yes` подтверждает, что вы хотите удалить ресурсы без дополнительного запроса.

```azurecli-interactive
az group delete --name myResourceGroup --no-wait --yes
```


## <a name="next-steps"></a>Дальнейшие действия
Из этого руководства вы узнали, как создавать и использовать пользовательский образ виртуальной машины для масштабируемых наборов с помощью Azure CLI, в частности, как выполнять такие задачи:

> [!div class="checklist"]
> * Создание Общей коллекции образов.
> * Создание определения специализированного образа.
> * Создание версии образа
> * Создание масштабируемого набора на основе специализированного образа.
> * Предоставление общего доступа к коллекции образов.

Перейдите к следующему руководству, чтобы узнать, как развертывать приложения в масштабируемый набор.

> [!div class="nextstepaction"]
> [Руководство по развертыванию приложений в масштабируемых наборах](tutorial-install-apps-cli.md)