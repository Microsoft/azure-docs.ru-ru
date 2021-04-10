---
title: Руководство по защите веб-сервера Linux с помощью TLS/SSL-сертификатов в Azure
description: В этом руководстве описано, как использовать Azure CLI для защиты виртуальной машины Linux, запущенной на веб-сервере NGINX, с помощью SSL-сертификатов, хранящихся в Azure Key Vault.
services: virtual-machines
documentationcenter: virtual-machines
author: cynthn
manager: gwallace
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines
ms.collection: linux
ms.topic: tutorial
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 04/30/2018
ms.author: cynthn
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 3f762597ad81dfaba907115cbcf6074d81ec2fa4
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102549589"
---
# <a name="tutorial-secure-a-web-server-on-a-linux-virtual-machine-in-azure-with-tlsssl-certificates-stored-in-key-vault"></a>Руководство по защите веб-сервера на виртуальной машине Linux в Azure с помощью TLS/SSL-сертификатов, хранимых в Key Vault
Чтобы защитить веб-серверы, для шифрования веб-трафика можно использовать протокол TLS, который ранее назывался протоколом SSL. TLS/SSL-сертификаты могут храниться в Azure Key Vault и разрешать безопасное развертывание сертификатов на виртуальных машинах Linux в Azure. Из этого руководства вы узнаете, как выполнить следующие задачи:

> [!div class="checklist"]
> * создать Azure Key Vault;
> * создать или передать сертификат в Key Vault;
> * создание виртуальной машины и установка веб-сервера NGINX;
> * внедрение сертификата в виртуальную машину и настройка NGINX с помощью TLS-привязки.

При работе с этим руководством используется интерфейс командной строки (CLI) в [Azure Cloud Shell](../../cloud-shell/overview.md), который всегда обновлен до последней версии. Чтобы открыть Cloud Shell, выберите **Попробовать** в верхнем углу любого блока кода.

Если вы решили установить и использовать интерфейс командной строки локально, то для работы с этим руководством вам понадобится Azure CLI 2.0.30 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0]( /cli/azure/install-azure-cli).


## <a name="overview"></a>Обзор
Azure Key Vault защищает криптографические ключи и секреты, в том числе сертификаты и пароли. Key Vault помогает оптимизировать управление сертификатами и позволяет контролировать ключи, которые предоставляют доступ к этим сертификатам. Можно создать самозаверяющий сертификат в Key Vault или передать существующий доверенный сертификат.

Вместо того чтобы использовать образ виртуальной машины, включающий встроенные сертификаты, можно внедрить сертификаты в работающую виртуальную машину. Этот процесс гарантирует установку на веб-сервер самых последних сертификатов во время развертывания. Если вы обновляете или заменяете сертификат, вам не нужно создавать новый пользовательский образ виртуальной машины. Последние сертификаты внедряются автоматически при создании дополнительных виртуальных машин. При этом сертификат никогда не покидает платформу Azure и не отображается в скрипте, журнале командной строки или шаблоне.


## <a name="create-an-azure-key-vault"></a>создать Azure Key Vault;
Прежде чем создать Key Vault и сертификаты, выполните командлет [az group create](/cli/azure/group), чтобы создать группу ресурсов. В следующем примере создается группа ресурсов с именем *myResourceGroupSecureWeb* в расположении *eastus*:

```azurecli-interactive 
az group create --name myResourceGroupSecureWeb --location eastus
```

Затем создайте Key Vault с помощью [az keyvault create](/cli/azure/keyvault) и включите его использование при развертывании виртуальной машины. У каждого хранилища ключей должно быть уникальное имя в нижнем регистре. Замените *\<mykeyvault>* в следующем примере собственным уникальным именем Key Vault.

```azurecli-interactive 
keyvault_name=<mykeyvault>
az keyvault create \
    --resource-group myResourceGroupSecureWeb \
    --name $keyvault_name \
    --enabled-for-deployment
```

## <a name="generate-a-certificate-and-store-in-key-vault"></a>Создание сертификата и его сохранение в Key Vault
Для использования в рабочей среде следует импортировать действительный сертификат, подписанный доверенным поставщиком, выполнив команду [az keyvault certificate import](/cli/azure/keyvault/certificate). В следующем примере в этом руководстве показано, как можно командой [az keyvault certificate create](/cli/azure/keyvault/certificate) создать самозаверяющий сертификат, использующий политику сертификата по умолчанию.

```azurecli-interactive 
az keyvault certificate create \
    --vault-name $keyvault_name \
    --name mycert \
    --policy "$(az keyvault certificate get-default-policy)"
```

### <a name="prepare-a-certificate-for-use-with-a-vm"></a>Подготовка сертификата для использования с виртуальной машиной
Чтобы использовать сертификата во время создания виртуальной машины, получите идентификатор сертификата, выполнив команду [az keyvault secret list-versions](/cli/azure/keyvault/secret). Преобразуйте сертификат, выполнив команду [az vm secret format](/cli/azure/vm/secret#az-vm-secret-format). Следующий пример присваивает переменным результаты этих команд, чтобы их было удобно использовать в дальнейшем.

```azurecli-interactive 
secret=$(az keyvault secret list-versions \
          --vault-name $keyvault_name \
          --name mycert \
          --query "[?attributes.enabled].id" --output tsv)
vm_secret=$(az vm secret format --secrets "$secret" -g myResourceGroupSecureWeb --keyvault $keyvault_name)
```

### <a name="create-a-cloud-init-config-to-secure-nginx"></a>Создание конфигурации cloud-init для защиты сервера NGINX
[Пакет cloud-init](https://cloudinit.readthedocs.io) — широко используемое средство, используемое для настройки виртуальной машины Linux при ее первой загрузке. Вы можете использовать cloud-init для установки пакетов, записи файлов или настройки пользователей и параметров безопасности. Так как cloud-init выполняется при начальной загрузке, для применения вашей конфигурации не требуются какие-либо дополнительные действия или обязательные агенты.

При создании виртуальной машины сертификаты и ключи хранятся в защищенном каталоге */var/lib/waagent/* . Используйте cloud-init, чтобы автоматизировать добавление сертификата виртуальной машины и настройку веб-сервера. В этом примере вы установите и настроите веб-сервер NGINX. Этот же процесс можно использовать для установки и настройки Apache. 

Создайте файл *cloud-init-web-server.txt* и вставьте в него приведенную ниже конфигурацию:

```yaml
#cloud-config
package_upgrade: true
packages:
  - nginx
write_files:
  - owner: www-data:www-data
  - path: /etc/nginx/sites-available/default
    content: |
      server {
        listen 443 ssl;
        ssl_certificate /etc/nginx/ssl/mycert.cert;
        ssl_certificate_key /etc/nginx/ssl/mycert.prv;
      }
runcmd:
  - secretsname=$(find /var/lib/waagent/ -name "*.prv" | cut -c -57)
  - mkdir /etc/nginx/ssl
  - cp $secretsname.crt /etc/nginx/ssl/mycert.cert
  - cp $secretsname.prv /etc/nginx/ssl/mycert.prv
  - service nginx restart
```

### <a name="create-a-secure-vm"></a>Создание защищенной виртуальной машины
Теперь создайте виртуальную машину командой [az vm create](/cli/azure/vm). Данные сертификата внедряются из Key Vault с помощью параметра `--secrets`. Передайте конфигурацию cloud-init с параметром `--custom-data`:

```azurecli-interactive 
az vm create \
    --resource-group myResourceGroupSecureWeb \
    --name myVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys \
    --custom-data cloud-init-web-server.txt \
    --secrets "$vm_secret"
```

На создание виртуальной машины, установку пакетов и запуск приложения может потребоваться несколько минут. Когда виртуальная машина будет создана, запишите значение `publicIpAddress`, отображаемое Azure CLI. Этот адрес используется для доступа к сайту в веб-браузере.

Чтобы разрешить безопасную передачу веб-трафика для виртуальной машины, откройте порт 443 для Интернета командой [az vm open-port](/cli/azure/vm).

```azurecli-interactive 
az vm open-port \
    --resource-group myResourceGroupSecureWeb \
    --name myVM \
    --port 443
```


### <a name="test-the-secure-web-app"></a>Тестирование защищенного веб-приложения
Теперь можно открыть веб-браузер и ввести в адресной строке *https:\/\/\<publicIpAddress>* . Укажите собственный общедоступный IP-адрес, настроенный при создании виртуальной машины. Если используется самозаверяющий сертификат, ответьте согласием на предупреждение системы безопасности.

![Принятие предупреждения о безопасности веб-браузера](./media/tutorial-secure-web-server/browser-warning.png)

На экране отобразится защищенный веб-сайт NGINX, как в показано следующем примере:

![Просмотр работающего защищенного сайта NGINX](./media/tutorial-secure-web-server/secured-nginx.png)


## <a name="next-steps"></a>Дальнейшие действия

Следуя указаниям этого учебника, вы защитили веб-сервер NGINX TLS/SSL-сертификатом, который хранится в Azure Key Vault. Вы ознакомились с выполнением следующих задач:

> [!div class="checklist"]
> * создать Azure Key Vault;
> * создать или передать сертификат в Key Vault;
> * создание виртуальной машины и установка веб-сервера NGINX;
> * внедрение сертификата в виртуальную машину и настройка NGINX с помощью TLS-привязки.

Чтобы увидеть предварительно созданные примеры скриптов виртуальной машины, перейдите по ссылке ниже.

> [!div class="nextstepaction"]
> [Примеры сценариев для виртуальной машины Linux](https://github.com/Azure-Samples/azure-cli-samples/tree/master/virtual-machine)