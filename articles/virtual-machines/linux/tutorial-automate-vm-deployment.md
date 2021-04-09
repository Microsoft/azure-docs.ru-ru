---
title: Учебник. Настройка виртуальной машины Linux с помощью cloud-init в Azure
description: В этом руководстве описано, как использовать пакет cloud-init и Key Vault для настройки виртуальных машин Linux при первой загрузке в Azure
services: virtual-machines-linux
documentationcenter: virtual-machines
author: cynthn
ms.service: virtual-machines
ms.collection: linux
ms.topic: tutorial
ms.workload: infrastructure
ms.date: 09/12/2019
ms.author: cynthn
ms.custom: mvc, devx-track-js, devx-track-azurecli
ms.openlocfilehash: 6dd10039ba5c71d3a787761914b111e7828c47d3
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102552615"
---
# <a name="tutorial---how-to-use-cloud-init-to-customize-a-linux-virtual-machine-in-azure-on-first-boot"></a>Руководство. Настройка виртуальной машины Linux при первой загрузке с помощью cloud-init в Azure

В рамках предыдущего руководства вы узнали, как применить протокол SSH к виртуальной машине и установить nginx. Для быстрого и согласованного создания виртуальных машин, как правило, применяются средства автоматизации. Наиболее распространенный подход к настройке виртуальной машины при первой загрузке — использование [cloud-init](https://cloudinit.readthedocs.io). Из этого руководства вы узнаете, как выполнить следующие задачи:

> [!div class="checklist"]
> * создать файл конфигурации cloud-init;
> * создать виртуальную машину, использующую файл конфигурации cloud-init;
> * просмотреть выполняющееся приложение Node.js после создания виртуальной машины;
> * обеспечить безопасное хранение сертификатов в хранилище ключей;
> * автоматизировать безопасные развертывания nginx с помощью файла конфигурации cloud-init.

Если вы решили установить и использовать интерфейс командной строки локально, то для работы с этим руководством вам понадобится Azure CLI 2.0.30 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0]( /cli/azure/install-azure-cli).

## <a name="cloud-init-overview"></a>Обзор cloud-Init
[Пакет cloud-init](https://cloudinit.readthedocs.io) — широко используемое средство, используемое для настройки виртуальной машины Linux при ее первой загрузке. Вы можете использовать cloud-init для установки пакетов, записи файлов или настройки пользователей и параметров безопасности. Так как cloud-init выполняется при начальной загрузке, для применения вашей конфигурации не требуются какие-либо дополнительные действия или обязательные агенты.

Кроме того, cloud-init работает с разными дистрибутивами. Например, для установки пакета не используется **apt-get install** или **yum install**. Вместо этого можно определить список пакетов для установки. Файл cloud-init автоматически использует собственный инструмент управления пакетами из выбранного дистрибутива.

Мы и наши партнеры работаем над тем, чтобы сценарии cloud-init были добавлены в образы, предоставляемые для Azure. Подробные сведения о поддержке cloud-init для каждого дистрибутива см. в статье [Поддержка cloud-init для виртуальных машин в Azure](using-cloud-init.md).


## <a name="create-cloud-init-config-file"></a>Создание файла конфигурации cloud-init
Чтобы посмотреть, как работает cloud-init, создайте виртуальную машину для установки сервера NGINX и запуска простого приложения Node.js Hello World. Приведенная ниже конфигурация cloud-init устанавливает требуемые пакеты, создает приложение Node.js, а затем инициализирует и запускает это приложение.

В командной строке Bash или в Cloud Shell создайте файл с именем *cloud-init.txt* и добавьте в него следующую конфигурацию. Например, введите `sensible-editor cloud-init.txt`, чтобы создать файл и просмотреть список доступных редакторов. Убедитесь, что весь файл cloud-init скопирован правильно, особенно первая строка:

```yaml
#cloud-config
package_upgrade: true
packages:
  - nginx
  - nodejs
  - npm
write_files:
  - owner: www-data:www-data
    path: /etc/nginx/sites-available/default
    content: |
      server {
        listen 80;
        location / {
          proxy_pass http://localhost:3000;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection keep-alive;
          proxy_set_header Host $host;
          proxy_cache_bypass $http_upgrade;
        }
      }
  - owner: azureuser:azureuser
    path: /home/azureuser/myapp/index.js
    content: |
      var express = require('express')
      var app = express()
      var os = require('os');
      app.get('/', function (req, res) {
        res.send('Hello World from host ' + os.hostname() + '!')
      })
      app.listen(3000, function () {
        console.log('Hello world app listening on port 3000!')
      })
runcmd:
  - service nginx restart
  - cd "/home/azureuser/myapp"
  - npm init
  - npm install express -y
  - nodejs index.js
```

Чтобы узнать больше о параметрах конфигурации cloud-init, ознакомьтесь с [примерами конфигурации cloud-init](https://cloudinit.readthedocs.io/en/latest/topics/examples.html).

## <a name="create-virtual-machine"></a>Создание виртуальной машины
Прежде чем создать виртуальную машину, выполните команду [az group create](/cli/azure/group#az-group-create), чтобы создать группу ресурсов. В следующем примере создается группа ресурсов с именем *myResourceGroupAutomate* в расположении *eastus*.

```azurecli-interactive
az group create --name myResourceGroupAutomate --location eastus
```

Теперь создайте виртуальную машину командой [az vm create](/cli/azure/vm#az-vm-create). Используйте параметр `--custom-data`, чтобы передать файл конфигурации cloud-init. Укажите полный путь к конфигурации *cloud-init.txt*, если этот файл сохранен вне текущего рабочего каталога. В следующем примере создается виртуальная машина с именем *myVM*.

```azurecli-interactive
az vm create \
    --resource-group myResourceGroupAutomate \
    --name myAutomatedVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys \
    --custom-data cloud-init.txt
```

На создание виртуальной машины, установку пакетов и запуск приложения может потребоваться несколько минут. Некоторые фоновые задачи продолжают работу после возврата к командной строке в Azure CLI. Прежде чем вы получите доступ к приложению, может пройти несколько минут. Когда виртуальная машина будет создана, запишите значение `publicIpAddress`, отображаемое Azure CLI. Это адрес для доступа к приложению Node.js через веб-браузер.

Чтобы разрешить передачу веб-трафика для виртуальной машины, откройте порт 80 для Интернета командой [az vm open-port](/cli/azure/vm#az-vm-open-port).

```azurecli-interactive
az vm open-port --port 80 --resource-group myResourceGroupAutomate --name myAutomatedVM
```

## <a name="test-web-app"></a>Тестирование веб-приложения
Теперь можно открыть веб-браузер и ввести в адресной строке *http:\/\/\<publicIpAddress>* . Укажите собственный общедоступный IP-адрес, настроенный при создании виртуальной машины. Отобразится ваше приложение Node.js, как показано ниже:

![Просмотр работающего сайта NGINX](./media/tutorial-automate-vm-deployment/nginx.png)


## <a name="inject-certificates-from-key-vault"></a>Внедрение сертификатов из Key Vault
В этом дополнительном разделе описывается, как безопасно хранить сертификаты в Azure Key Vault и внедрять их во время развертывания виртуальных машин. Вместо того, чтобы использовать пользовательский образ со встроенными сертификатами, можно использовать этот процесс, чтобы обеспечить внедрение наиболее актуальных сертификатов в виртуальную машину при ее первом запуске. При этом сертификат никогда не покидает платформу Azure и не предоставляется в сценарии, журнале командной строки или шаблоне.

Azure Key Vault защищает криптографические ключи и секреты, в том числе сертификаты и пароли. Key Vault помогает оптимизировать управление ключами и позволяет поддерживать контроль над ключами, которые предоставляют доступ к вашим данным и шифруют их. В этом сценарии вводятся некоторые понятия Key Vault, относящиеся к созданию и использованию сертификата. Впрочем, это не все общие сведения о том, как использовать Key Vault.

Далее описывается, как:

- создать Azure Key Vault;
- создать или передать сертификат в Key Vault;
- создать секрет из сертификата для внедрения в виртуальную машину;
- создать виртуальную машину и внедрить сертификат.

### <a name="create-an-azure-key-vault"></a>создать Azure Key Vault;
Сначала создайте Key Vault командой [az keyvault create](/cli/azure/keyvault#az-keyvault-create) и включите его использование при развертывании виртуальной машины. У каждого Key Vault должно быть уникальное имя в нижнем регистре. Замените `mykeyvault` в следующем примере собственным уникальным именем Key Vault.

```azurecli-interactive
keyvault_name=mykeyvault
az keyvault create \
    --resource-group myResourceGroupAutomate \
    --name $keyvault_name \
    --enabled-for-deployment
```

### <a name="generate-certificate-and-store-in-key-vault"></a>Создание сертификата и его сохранение в Key Vault
Для использования в рабочей среде следует импортировать действительный сертификат, подписанный доверенным поставщиком, выполнив команду [az keyvault certificate import](/cli/azure/keyvault/certificate#az-keyvault-certificate-import). В следующем примере в этом руководстве показано, как можно командой [az keyvault certificate create](/cli/azure/keyvault/certificate#az-keyvault-certificate-create) создать самозаверяющий сертификат, использующий политику сертификата по умолчанию.

```azurecli-interactive
az keyvault certificate create \
    --vault-name $keyvault_name \
    --name mycert \
    --policy "$(az keyvault certificate get-default-policy --output json)"
```


### <a name="prepare-certificate-for-use-with-vm"></a>Подготовка сертификата для использования с виртуальной машиной
Чтобы использовать сертификата во время создания виртуальной машины, получите идентификатор сертификата, выполнив команду [az keyvault secret list-versions](/cli/azure/keyvault/secret#az-keyvault-secret-list-versions). Виртуальной машине требуется сертификат в определенном формате, чтобы вставить его во время загрузки. Поэтому преобразуйте сертификат с помощью команды [az vm secret format](/cli/azure/vm). Следующий пример присваивает переменным результаты этих команд, чтобы их было удобно использовать в дальнейшем.

```azurecli-interactive
secret=$(az keyvault secret list-versions \
          --vault-name $keyvault_name \
          --name mycert \
          --query "[?attributes.enabled].id" --output tsv)
vm_secret=$(az vm secret format --secret "$secret" --output json)
```


### <a name="create-cloud-init-config-to-secure-nginx"></a>Создание конфигурации cloud-init для защиты сервера NGINX
При создании виртуальной машины сертификаты и ключи хранятся в защищенном каталоге */var/lib/waagent/* . Чтобы автоматизировать добавление сертификата в виртуальную машину и настройку сервера NGINX, используйте обновленную конфигурацию cloud-init из предыдущего примера.

Создайте файл *cloud-init-secured.txt* и вставьте в него приведенную ниже конфигурацию. Если вы используете Cloud Shell, создайте файл конфигурации cloud-init в этой оболочке, а не на локальном компьютере. Например, введите `sensible-editor cloud-init-secured.txt`, чтобы создать файл и просмотреть список доступных редакторов. Убедитесь, что весь файл cloud-init скопирован правильно, особенно первая строка:

```yaml
#cloud-config
package_upgrade: true
packages:
  - nginx
  - nodejs
  - npm
write_files:
  - owner: www-data:www-data
    path: /etc/nginx/sites-available/default
    content: |
      server {
        listen 80;
        listen 443 ssl;
        ssl_certificate /etc/nginx/ssl/mycert.cert;
        ssl_certificate_key /etc/nginx/ssl/mycert.prv;
        location / {
          proxy_pass http://localhost:3000;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection keep-alive;
          proxy_set_header Host $host;
          proxy_cache_bypass $http_upgrade;
        }
      }
  - owner: azureuser:azureuser
    path: /home/azureuser/myapp/index.js
    content: |
      var express = require('express')
      var app = express()
      var os = require('os');
      app.get('/', function (req, res) {
        res.send('Hello World from host ' + os.hostname() + '!')
      })
      app.listen(3000, function () {
        console.log('Hello world app listening on port 3000!')
      })
runcmd:
  - secretsname=$(find /var/lib/waagent/ -name "*.prv" | cut -c -57)
  - mkdir /etc/nginx/ssl
  - cp $secretsname.crt /etc/nginx/ssl/mycert.cert
  - cp $secretsname.prv /etc/nginx/ssl/mycert.prv
  - service nginx restart
  - cd "/home/azureuser/myapp"
  - npm init
  - npm install express -y
  - nodejs index.js
```

### <a name="create-secure-vm"></a>Создание безопасной виртуальной машины
Теперь создайте виртуальную машину командой [az vm create](/cli/azure/vm#az-vm-create). Данные сертификата внедряются из Key Vault с помощью параметра `--secrets`. Как и в предыдущем примере, можно передать файл конфигурации cloud-init с помощью параметра `--custom-data`.

```azurecli-interactive
az vm create \
    --resource-group myResourceGroupAutomate \
    --name myVMWithCerts \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys \
    --custom-data cloud-init-secured.txt \
    --secrets "$vm_secret"
```

На создание виртуальной машины, установку пакетов и запуск приложения может потребоваться несколько минут. Некоторые фоновые задачи продолжают работу после возврата к командной строке в Azure CLI. Прежде чем вы получите доступ к приложению, может пройти несколько минут. Когда виртуальная машина будет создана, запишите значение `publicIpAddress`, отображаемое Azure CLI. Это адрес для доступа к приложению Node.js через веб-браузер.

Чтобы разрешить безопасную передачу веб-трафика для виртуальной машины, откройте порт 443 для Интернета командой [az vm open-port](/cli/azure/vm#az-vm-open-port).

```azurecli-interactive
az vm open-port \
    --resource-group myResourceGroupAutomate \
    --name myVMWithCerts \
    --port 443
```

### <a name="test-secure-web-app"></a>Тестирование защищенного веб-приложения
Теперь можно открыть веб-браузер и ввести в адресной строке *https:\/\/\<publicIpAddress>* . Укажите собственный общедоступный IP-адрес, как показано в выходных данных при создании предыдущей виртуальной машины. Если используется самозаверяющий сертификат, ответьте согласием на предупреждение системы безопасности.

![Принятие предупреждения о безопасности веб-браузера](./media/tutorial-automate-vm-deployment/browser-warning.png)

На экране отобразятся защищенный сайт NGINX и приложение Node.js, как в показано следующем примере.

![Просмотр работающего защищенного сайта NGINX](./media/tutorial-automate-vm-deployment/secured-nginx.png)


## <a name="next-steps"></a>Дальнейшие действия
В рамках этого руководства вы настроили виртуальные машины при первой загрузке с помощью файла конфигурации cloud-init. Вы ознакомились с выполнением следующих задач:

> [!div class="checklist"]
> * создать файл конфигурации cloud-init;
> * создать виртуальную машину, использующую файл конфигурации cloud-init;
> * просмотреть выполняющееся приложение Node.js после создания виртуальной машины;
> * обеспечить безопасное хранение сертификатов в хранилище ключей;
> * автоматизировать безопасные развертывания nginx с помощью файла конфигурации cloud-init.

Перейдите к следующему руководству, чтобы научиться создавать пользовательские образы виртуальных машин.

> [!div class="nextstepaction"]
> [Создание пользовательских образов виртуальной машины](./tutorial-custom-images.md)
