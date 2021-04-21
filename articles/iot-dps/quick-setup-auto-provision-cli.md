---
title: Краткое руководство. Настройка Подготовки устройств к добавлению в Центр Интернета вещей Azure с помощью Azure CLI
description: Краткое руководство. Настройка Службы подготовки устройств к добавлению в Центр Интернета вещей Azure с помощью Azure CLI
author: wesmc7777
ms.author: wesmc
ms.date: 11/08/2019
ms.topic: quickstart
ms.service: iot-dps
services: iot-dps
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 3d52a83c8c0920c4d85aa5b4b6b89fd8d36e5fea
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107774977"
---
# <a name="quickstart-set-up-the-iot-hub-device-provisioning-service-with-azure-cli"></a>Краткое руководство. Настройка службы подготовки устройств к добавлению в Центр Интернета вещей с помощью Azure CLI

Azure CLI используется для создания ресурсов Azure и управления ими из командной строки или с помощью скриптов. В этом кратком руководстве описано создание центра Интернета вещей и службы "Подготовка устройств к добавлению в Центр Интернета вещей" с помощью Azure CLI, а также связывание этих двух служб. 

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

> [!IMPORTANT]
> Центр Интернета вещей и служба подготовки, которые вы создадите в рамках этого краткого руководства, будут общедоступными как конечные точки DNS, поэтому не используйте конфиденциальную информацию при изменении имен этих ресурсов.
>

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]


## <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group#az_group_create). Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими. 

В следующем примере создается группа ресурсов с именем *my-sample-resource-group* в расположении *westus*.

```azurecli-interactive 
az group create --name my-sample-resource-group --location westus
```

> [!TIP]
> В примере создается группа ресурсов в расположении западной части США. Список доступных расположений можно просмотреть, выполнив команду `az account list-locations -o table`.
>
>

## <a name="create-an-iot-hub"></a>Создание Центра Интернета вещей

Создайте Центр Интернета вещей с помощью команды [az iot hub create](/cli/azure/iot/hub#az_iot_hub_create).

В следующем примере создается Центр Интернета вещей с именем *my-sample-hub* в расположении *westus*. Имя центра Интернета вещей должно быть глобально уникальным в Azure. Для этого можно добавить уникальный префикс или суффикс к имени примера или же вообще выбрать новое имя. Убедитесь, что это имя соответствует соглашению об именовании: оно должно быть длиной от 3 до 50 символов и может содержать только буквы верхнего или нижнего регистра, цифры или дефисы (-). 

```azurecli-interactive 
az iot hub create --name my-sample-hub --resource-group my-sample-resource-group --location westus
```

## <a name="create-a-device-provisioning-service"></a>Создание службы подготовки устройств

Создайте службу подготовки устройств с помощью команды [az iot dps create](/cli/azure/iot/dps#az_iot_dps_create). 

В следующем примере создается служба подготовки устройств с именем *my-sample-dps* в расположении *westus*. Кроме того, нужно будет выбрать глобально уникальное имя для службы подготовки. Убедитесь, что это имя соответствует соглашению об именовании для Службы подготовки устройств к добавлению в Центр Интернета вещей: оно должно быть длиной от 3 до 64 символов и может содержать только буквы верхнего или нижнего регистра, цифры или дефисы.

```azurecli-interactive 
az iot dps create --name my-sample-dps --resource-group my-sample-resource-group --location westus
```

> [!TIP]
> В примере создается служба подготовки устройств в расположении "Западная часть США". Чтобы просмотреть список доступных расположений, выполните команду `az provider show --namespace Microsoft.Devices --query "resourceTypes[?resourceType=='ProvisioningServices'].locations | [0]" --out table`, или перейдите на страницу [Состояние Azure](https://azure.microsoft.com/status/) и введите в строке поиска "Служба подготовки устройств". В командах расположения можно указать, используя одно слово или несколько. Например: westus, West US, WEST US и т. д. Обратите внимание, что регистр не учитывается. Если для указания расположения вы используете несколько слов, укажите значение поиска в кавычках, например `--location "West US"`.
>

## <a name="get-the-connection-string-for-the-iot-hub"></a>Получение строки подключения для Центра Интернета вещей

Необходимо связать строку подключения Центра Интернета вещей со службой подготовки устройств. Используйте команду [az iot hub show-connection-string](/cli/azure/iot/hub#az_iot_hub_show_connection_string), чтобы получить строку подключения. Ее выходные данные потребуются, чтобы задать переменную, необходимую для связывания двух ресурсов. 

В следующем примере переменной *hubConnectionString* задается значение строки подключения для первичного ключа политики *iothubowner* в центре (параметр `--policy-name` можно использовать для указания другой политики). Замените *my-sample-hub* на уникальное имя центра Интернета вещей, которое вы выбрали ранее. Для извлечения строки подключения из выходных данных команды в Azure CLI в ней используются параметры [query](/cli/azure/query-azure-cli) и [output](/cli/azure/format-output-azure-cli#tsv-output-format).

```azurecli-interactive 
hubConnectionString=$(az iot hub show-connection-string --name my-sample-hub --key primary --query connectionString -o tsv)
```

Чтобы получить строку подключения, можно использовать команду `echo`.

```azurecli-interactive 
echo $hubConnectionString
```

> [!NOTE]
> Эти две команды можно использовать для узла, где работает Bash.
> 
> Если вы используете локальную оболочку Windows или CMD либо узел PowerShell, измените команды, чтобы использовать правильный синтаксис для этой среды.
>
> Если вы используете Azure Cloud Shell, убедитесь, что в раскрывающемся списке со средами в левой части окна оболочки указано **Bash**.
>

## <a name="link-the-iot-hub-and-the-provisioning-service"></a>Связывание Центра Интернета вещей со службой подготовки устройств

Свяжите Центр Интернета вещей со службой подготовки устройств с помощью команды [az iot dps linked-hub create](/cli/azure/iot/dps/linked-hub#az_iot_dps_linked_hub_create). 

В следующем примере связываются центр Интернета вещей с именем *my-sample-hub* в расположении *westus* и служба подготовки устройств с именем *my-sample-dps*. Замените эти имена на уникальные имена центра Интернета вещей и службы подготовки устройств, которые вы выбрали ранее. В команде используется строка подключения для центра Интернета вещей, сохраненная в переменной *hubConnectionString* на предыдущем шаге.

```azurecli-interactive 
az iot dps linked-hub create --dps-name my-sample-dps --resource-group my-sample-resource-group --connection-string $hubConnectionString --location westus
```

Выполнение этой команды может занять несколько минут.

## <a name="verify-the-provisioning-service"></a>Проверка службы подготовки устройств

Получите сведения о службе подготовки устройств, выполнив команду [az iot dps show](/cli/azure/iot/dps#az_iot_dps_show).

В следующем примере возвращаются сведения о службе подготовки устройств с именем *my-sample-dps*. Замените это имя на собственное имя службы подготовки устройств.

```azurecli-interactive
az iot dps show --name my-sample-dps
```
Связанный Центр Интернета вещей отображается в коллекции *properties.iotHubs*.

![Проверка службы подготовки устройств](./media/quick-setup-auto-provision-cli/verify-provisioning-service.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

Другие краткие руководства в этой коллекции созданы на основе этого документа. Если вы планируете продолжать работу с этими руководствами по быстрому запуску или обычными руководствами, не удаляйте созданные ресурсы. Если же вы не планируете продолжать работу, вы можете использовать команды ниже, чтобы удалить службу подготовки устройств, Центр Интернета вещей или группу ресурсов со всеми связанными ресурсами. Замените имена ресурсов, указанные ниже, именами собственных ресурсов.

Чтобы удалить службу подготовки устройств, выполните команду [az iot dps delete](/cli/azure/iot/dps#az_iot_dps_delete):

```azurecli-interactive
az iot dps delete --name my-sample-dps --resource-group my-sample-resource-group
```
Чтобы удалить Центр Интернета вещей, выполните команду [az iot hub delete](/cli/azure/iot/hub#az_iot_hub_delete):

```azurecli-interactive
az iot hub delete --name my-sample-hub --resource-group my-sample-resource-group
```

Чтобы удалить группу ресурсов со всеми связанными ресурсами, выполните команду [az group delete](/cli/azure/group#az_group_delete):

```azurecli-interactive
az group delete --name my-sample-resource-group
```

## <a name="next-steps"></a>Дальнейшие действия

Вы развернули центр Интернета вещей и экземпляр службы подготовки устройств, а затем связали эти два ресурса. Чтобы узнать, как подготовить виртуальное устройство, см. краткое руководство по созданию имитированного устройства.

> [!div class="nextstepaction"]
> [Краткое руководство. Подготовка имитированного устройства TPM с помощью пакета SDK Интернета вещей Azure для C](./quick-create-simulated-device.md)
