---
title: Краткое руководство. Создание масштабируемого набора виртуальных машин с помощью Azure CLI
description: Начните работу с развертыванием. Узнайте, как быстро создать масштабируемый набор виртуальных машин с помощью Azure CLI
author: ju-shim
ms.author: jushiman
ms.topic: quickstart
ms.service: virtual-machine-scale-sets
ms.date: 03/27/2018
ms.reviewer: mimckitt
ms.custom: mimckitt, devx-track-azurecli
ms.openlocfilehash: d65df03bc23704fb4d4c037655cc4b84ccc4e576
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105934654"
---
# <a name="quickstart-create-a-virtual-machine-scale-set-with-the-azure-cli"></a>Краткое руководство. Создание масштабируемого набора виртуальных машин с помощью Azure CLI
Масштабируемый набор виртуальных машин позволяет развернуть набор виртуальных машин с поддержкой автомасштабирования и управлять этим набором. Вы можете вручную изменить число виртуальных машин в масштабируемом наборе или определить правила для автоматического масштабирования в зависимости от использования ЦП, объема памяти или сетевого трафика. После этого Azure Load Balancer будет распределять трафик между экземплярами виртуальных машин в масштабируемом наборе. При работе с этим кратким руководством вы создадите масштабируемый набор виртуальных машин и развернете пример приложения с помощью шаблона Azure CLI.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0.29 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена. 


## <a name="create-a-scale-set"></a>Создание масштабируемого набора
Прежде чем создать масштабируемый набор, выполните команду [az group create](/cli/azure/group) для создания группы ресурсов. В следующем примере создается группа ресурсов с именем *myResourceGroup* в расположении *eastus*.

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Создайте масштабируемый набор виртуальных машин с помощью команды [az vmss create](/cli/azure/vmss). В следующем примере создается масштабируемый набор с именем *myScaleSet*, который настроен на автоматическое обновление при применении изменений. Также создаются ключи SSH, если они не существуют в *~/.ssh/id_rsa*. Эти ключи SSH используются для входа на экземпляры виртуальных машин. Чтобы вместо создания ключей SSH применить существующий набор, используйте параметр `--ssh-key-value` и укажите расположение ключей.

```azurecli-interactive
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --admin-username azureuser \
  --generate-ssh-keys
```

Создание и настройка всех ресурсов и виртуальных машин масштабируемого набора занимает несколько минут.


## <a name="deploy-sample-application"></a>Разверните пример приложения
Для проверки масштабируемого набора установите базовое веб-приложение. Расширение пользовательского скрипта Azure используется для скачивания и запуска скрипта, который устанавливает приложение на экземплярах виртуальных машин. Это расширение можно использовать для настройки после развертывания, установки программного обеспечения и других задач настройки или управления. Дополнительные сведения см. в статье [Расширение Custom Script в ОС Windows](../virtual-machines/extensions/custom-script-linux.md).

Используйте расширение пользовательских скриптов для установки базового веб-сервера NGINX. Примените расширение пользовательского скрипта, которое устанавливает NGINX, с помощью команды [az vmss extension set](/cli/azure/vmss/extension), как показано ниже.

```azurecli-interactive
az vmss extension set \
  --publisher Microsoft.Azure.Extensions \
  --version 2.0 \
  --name CustomScript \
  --resource-group myResourceGroup \
  --vmss-name myScaleSet \
  --settings '{"fileUris":["https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/automate_nginx.sh"],"commandToExecute":"./automate_nginx.sh"}'
```


## <a name="allow-traffic-to-application"></a>Разрешение передачи трафика в приложение
При создании масштабируемого набора была автоматически развернута служба Azure Load Balancer. Load Balancer распределяет трафик между экземплярами виртуальных машин в масштабируемом наборе. Чтобы разрешить передачу трафика в пример веб-приложения, создайте правило подсистемы балансировки нагрузки с помощью команды [az network lb rule create](/cli/azure/network/lb/rule). В следующем примере создается правило *myLoadBalancerRuleWeb*.

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


## <a name="test-your-scale-set"></a>Проверка масштабируемого набора
Чтобы увидеть, как работает масштабируемый набор, откройте пример веб-приложения в браузере. Получите общедоступный IP-адрес балансировщика нагрузки с помощью команды [az network public-ip show](/cli/azure/network/public-ip). Следующий пример получает IP-адрес для *myScaleSetLBPublicIP*, созданного ранее вместе с масштабируемым набором.

```azurecli-interactive
az network public-ip show \
  --resource-group myResourceGroup \
  --name myScaleSetLBPublicIP \
  --query '[ipAddress]' \
  --output tsv
```

Введите в браузере общедоступный IP-адрес подсистемы балансировки нагрузки. Подсистема балансировки нагрузки передаст запрос на один из экземпляров виртуальной машины, как показано в следующем примере:

![Веб-страница NGINX по умолчанию](media/virtual-machine-scale-sets-create-cli/running-nginx-site.png)


## <a name="clean-up-resources"></a>Очистка ресурсов
Вы можете удалить ненужную группу ресурсов, масштабируемый набор и все связанные ресурсы с помощью команды [az group delete](/cli/azure/group), как показано ниже. При использовании параметра `--no-wait` управление возвращается в командную строку без ожидания завершения операции. Параметр `--yes` подтверждает, что вы хотите удалить ресурсы без дополнительного запроса.

```azurecli-interactive
az group delete --name myResourceGroup --yes --no-wait
```


## <a name="next-steps"></a>Дальнейшие действия
С помощью этого краткого руководства вы создали простой масштабируемый набор и с помощью расширения пользовательского скрипта установили базовый веб-сервер NGINX на экземплярах виртуальных машин. Чтобы получить дополнительные сведения, продолжите роботу с руководством по созданию масштабируемых наборов виртуальных машин Azure и управлению ими.

> [!div class="nextstepaction"]
> [Создание масштабируемых наборов виртуальных машин Azure и управление ими](tutorial-create-and-manage-cli.md)
