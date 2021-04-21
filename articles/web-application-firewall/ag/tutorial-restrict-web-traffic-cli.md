---
title: Включение брандмауэра веб-приложения — Azure CLI
description: Узнайте, как ограничить веб-трафик с помощью брандмауэра веб-приложения и Azure CLI.
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.date: 03/29/2021
ms.author: victorh
ms.topic: how-to
ms.custom: devx-track-azurecli
ms.openlocfilehash: 390fdd4d9e9d0bc62589484ab0c4ba7468bcaf4b
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107773105"
---
# <a name="enable-web-application-firewall-using-the-azure-cli"></a>Включение брандмауэра веб-приложения с помощью Azure CLI

Вы можете ограничить трафик в шлюзе приложений с помощью [брандмауэра веб-приложения](ag-overview.md) (WAF). Для защиты приложения WAF использует правила [OWASP](https://www.owasp.org/index.php/Category:OWASP_ModSecurity_Core_Rule_Set_Project). Эти правила включают защиту от атак, например от внедрения кода SQL, межсайтовых скриптов и захватов сеанса.

Вы узнаете, как выполнять следующие задачи:

 * Настройка сети
 * Создание шлюза приложений с включенным WAF.
 * создавать масштабируемый набор виртуальных машин;
 * Создание учетной записи хранения и настройка диагностики.

![Пример брандмауэра веб-приложений](../media/tutorial-restrict-web-traffic-cli/scenario-waf.png)

При необходимости эти инструкции можно выполнить с помощью [Azure PowerShell](tutorial-restrict-web-traffic-powershell.md).

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0.4 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Группа ресурсов — это логический контейнер, в котором происходит развертывание ресурсов Azure и управление ими. Создайте группу ресурсов Azure с именем *myResourceGroupAG*, используя команду [az group create](/cli/azure/group#az_group_create).

```azurecli-interactive
az group create --name myResourceGroupAG --location eastus
```

## <a name="create-network-resources"></a>Создание сетевых ресурсов

Виртуальная сеть и подсети используются для предоставления сетевого подключения к шлюзу приложений и его связанным ресурсам. Создайте виртуальную сеть с именем *myVNet* и подсеть с именем *myAGSubnet*. Затем создайте общедоступный IP-адрес *myAGPublicIPAddress*.

```azurecli-interactive
az network vnet create \
  --name myVNet \
  --resource-group myResourceGroupAG \
  --location eastus \
  --address-prefix 10.0.0.0/16 \
  --subnet-name myBackendSubnet \
  --subnet-prefix 10.0.1.0/24

az network vnet subnet create \
  --name myAGSubnet \
  --resource-group myResourceGroupAG \
  --vnet-name myVNet \
  --address-prefix 10.0.2.0/24

az network public-ip create \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress \
  --allocation-method Static \
  --sku Standard
```

## <a name="create-an-application-gateway-with-a-waf"></a>Создание шлюза приложений с WAF

Выполните команду [az network application-gateway create](/cli/azure/network/application-gateway), чтобы создать шлюз приложений *myAppGateway*. При создании шлюза приложений с помощью Azure CLI укажите такие сведения о конфигурации, как емкость, номер SKU и параметры HTTP. Шлюз приложений назначен на адреса *myAGSubnet* и *myAGPublicIPAddress*.

```azurecli-interactive
az network application-gateway create \
  --name myAppGateway \
  --location eastus \
  --resource-group myResourceGroupAG \
  --vnet-name myVNet \
  --subnet myAGSubnet \
  --capacity 2 \
  --sku WAF_v2 \
  --http-settings-cookie-based-affinity Disabled \
  --frontend-port 80 \
  --http-settings-port 80 \
  --http-settings-protocol Http \
  --public-ip-address myAGPublicIPAddress

az network application-gateway waf-config set \
  --enabled true \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --firewall-mode Detection \
  --rule-set-version 3.0
```

Создание шлюза приложений может занять несколько минут. Когда шлюз приложений будет создан, вы увидите такие новые функции шлюза:

- *appGatewayBackendPool* — шлюз приложений должен иметь по крайней мере один внутренний пул адресов.
- *appGatewayBackendHttpSettings* — указывает, что для обмена данными используются порт 80 и протокол HTTP.
- *appGatewayHttpListener* — прослушиватель по умолчанию, связанный с *appGatewayBackendPool*.
- *appGatewayFrontendIP* — назначает адрес *myAGPublicIPAddress* для прослушивателя *appGatewayHttpListener*.
- *rule1* — правило маршрутизации по умолчанию, связанное с прослушивателем *appGatewayHttpListener*.

## <a name="create-a-virtual-machine-scale-set"></a>создавать масштабируемый набор виртуальных машин;

В этом примере вы создаете масштабируемый набор виртуальных машин, чтобы предоставить два сервера для внутреннего пула в шлюзе приложений. Виртуальные машины в масштабируемом наборе связаны с подсетью *myBackendSubnet*. Чтобы создать масштабируемый набор, выполните команду [az vmss create](/cli/azure/vmss#az_vmss_create).

Перед выполнением замените \<username> и \<password> своими значениями.

```azurecli-interactive
az vmss create \
  --name myvmss \
  --resource-group myResourceGroupAG \
  --image UbuntuLTS \
  --admin-username <username> \
  --admin-password <password> \
  --instance-count 2 \
  --vnet-name myVNet \
  --subnet myBackendSubnet \
  --vm-sku Standard_DS2 \
  --upgrade-policy-mode Automatic \
  --app-gateway myAppGateway \
  --backend-pool-name appGatewayBackendPool
```

### <a name="install-nginx"></a>Установка nginx

```azurecli-interactive
az vmss extension set \
  --publisher Microsoft.Azure.Extensions \
  --version 2.0 \
  --name CustomScript \
  --resource-group myResourceGroupAG \
  --vmss-name myvmss \
  --settings '{ "fileUris": ["https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/application-gateway/iis/install_nginx.sh"],"commandToExecute": "./install_nginx.sh" }'
```

## <a name="create-a-storage-account-and-configure-diagnostics"></a>Создание учетной записи хранения и настройка диагностики.

В этой статье шлюз приложений использует учетную запись хранения, чтобы хранить данные для выявления и предотвращения угроз. Для записи данных можно также использовать журналы Azure Monitor или концентратор событий. 

### <a name="create-a-storage-account"></a>Создание учетной записи хранения

Создайте учетную запись хранения с именем *myagstore1* с помощью команды [az storage account create](/cli/azure/storage/account#az_storage_account_create).

```azurecli-interactive
az storage account create \
  --name myagstore1 \
  --resource-group myResourceGroupAG \
  --location eastus \
  --sku Standard_LRS \
  --encryption-services blob
```

### <a name="configure-diagnostics"></a>Настройка диагностики

Настройте диагностику для записи данных в журналы ApplicationGatewayAccessLog, ApplicationGatewayPerformanceLog и ApplicationGatewayFirewallLog. Замените `<subscriptionId>` своим идентификатором подписки, а затем настройте диагностику с помощью команды [az monitor diagnostic-settings create](/cli/azure/monitor/diagnostic-settings#az_monitor_diagnostic_settings_create).

```azurecli-interactive
appgwid=$(az network application-gateway show --name myAppGateway --resource-group myResourceGroupAG --query id -o tsv)

storeid=$(az storage account show --name myagstore1 --resource-group myResourceGroupAG --query id -o tsv)

az monitor diagnostic-settings create --name appgwdiag --resource $appgwid \
  --logs '[ { "category": "ApplicationGatewayAccessLog", "enabled": true, "retentionPolicy": { "days": 30, "enabled": true } }, { "category": "ApplicationGatewayPerformanceLog", "enabled": true, "retentionPolicy": { "days": 30, "enabled": true } }, { "category": "ApplicationGatewayFirewallLog", "enabled": true, "retentionPolicy": { "days": 30, "enabled": true } } ]' \
  --storage-account $storeid
```

## <a name="test-the-application-gateway"></a>Тестирование шлюза приложений

Чтобы получить общедоступный IP-адрес шлюза приложений, используйте команду [az network public-ip show](/cli/azure/network/public-ip#az_network_public_ip_show). Скопируйте общедоступный IP-адрес и вставьте его в адресную строку браузера.

```azurecli-interactive
az network public-ip show \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress \
  --query [ipAddress] \
  --output tsv
```

![Тестирование базового URL-адреса в шлюзе приложений](../media/tutorial-restrict-web-traffic-cli/application-gateway-nginxtest.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

При необходимости вы можете удалить группу ресурсов, шлюз приложений и все связанные ресурсы.

```azurecli-interactive
az group delete --name myResourceGroupAG 
```

## <a name="next-steps"></a>Дальнейшие действия

[Настройка правил брандмауэра веб-приложения](application-gateway-customize-waf-rules-portal.md)
