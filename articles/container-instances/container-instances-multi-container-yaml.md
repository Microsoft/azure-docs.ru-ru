---
title: Руководство. Развертывание многоконтейнерной группы с помощью YAML
description: Из этого руководства вы узнаете, как развернуть группу с несколькими контейнерами в службе "Экземпляры контейнеров Azure" с помощью YAML-файла и Azure CLI.
ms.topic: article
ms.date: 07/01/2020
ms.openlocfilehash: 74269440357ee2d7ae36661618a31293346fa712
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107771269"
---
# <a name="tutorial-deploy-a-multi-container-group-using-a-yaml-file"></a>Руководство по развертыванию многоконтейнерной группы с использованием YAML-файла

> [!div class="op_single_selector"]
> * [YAML](container-instances-multi-container-yaml.md)
> * [Resource Manager](container-instances-multi-container-group.md)
>

Служба "Экземпляры контейнеров Azure" поддерживает развертывание нескольких контейнеров в одном узле с использованием [группы контейнеров](container-instances-container-groups.md). Группу контейнеров удобно использовать при создании сопроводительного приложения для ведения журнала, мониторинга или любой другой задачи, для которой службе требуется дополнительный связанный процесс.

В этом руководстве объясняется, как выполнить простую конфигурацию сопроводительного приложения с двумя контейнерами, развернув [YAML-файл](container-instances-reference-yaml.md) с помощью Azure CLI. Файл YAML — это краткий формат для указания параметров экземпляра. Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Настройка YAML-файла.
> * Развертывание группы контейнеров
> * Просмотр журналов контейнеров.

> [!NOTE]
> Многоконтенерные группы сейчас ограничены контейнерами Linux.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

## <a name="configure-a-yaml-file"></a>Настройка YAML-файла.

Чтобы развернуть многоконтейнерную группу с помощью команды [az container create][az-container-create] в Azure CLI, вы должны указать конфигурацию группы контейнеров в YAML-файле. Затем передайте YAML-файл в качестве параметра в команду.

Сначала скопируйте следующий YAML-файл в новый файл с именем **deploy-aci.yaml**. В Azure Cloud Shell для создания файла в рабочей папке можно применить Visual Studio Code.

```
code deploy-aci.yaml
```

Этот YAML-файл определяет группу контейнеров с именем "myContainerGroup", имеющую два контейнера, общедоступный IP-адрес и два предоставленных порта. Контейнеры развертываются на основе общедоступных образов корпорации Майкрософт. Первый контейнер в группе запускает веб-приложение с выходом в Интернет. Второй контейнер, расширение, периодически отправляет HTTP-запросы веб-приложению, запущенному в первом контейнере через локальную сеть группы контейнеров.

```YAML
apiVersion: 2019-12-01
location: eastus
name: myContainerGroup
properties:
  containers:
  - name: aci-tutorial-app
    properties:
      image: mcr.microsoft.com/azuredocs/aci-helloworld:latest
      resources:
        requests:
          cpu: 1
          memoryInGb: 1.5
      ports:
      - port: 80
      - port: 8080
  - name: aci-tutorial-sidecar
    properties:
      image: mcr.microsoft.com/azuredocs/aci-tutorial-sidecar
      resources:
        requests:
          cpu: 1
          memoryInGb: 1.5
  osType: Linux
  ipAddress:
    type: Public
    ports:
    - protocol: tcp
      port: 80
    - protocol: tcp
      port: 8080
tags: {exampleTag: tutorial}
type: Microsoft.ContainerInstance/containerGroups
```

Чтобы использовать частный реестр образов контейнеров, добавьте свойство `imageRegistryCredentials` в группу контейнеров с актуальными значениями для вашей среды:

```YAML
  imageRegistryCredentials:
  - server: imageRegistryLoginServer
    username: imageRegistryUsername
    password: imageRegistryPassword
```

## <a name="deploy-the-container-group"></a>Развертывание группы контейнеров

Создайте группу ресурсов с помощью команды [az group create][az-group-create].

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Разверните группу контейнеров с помощью команды [az container create][az-container-create], передав YAML-файл как аргумент.

```azurecli-interactive
az container create --resource-group myResourceGroup --file deploy-aci.yaml
```

В течение нескольких секунд вы должны получить исходный ответ Azure.

## <a name="view-deployment-state"></a>Просмотр состояния развертывания

Чтобы просмотреть состояние развертывания, используйте следующую команду [az container show][az-container-show].

```azurecli-interactive
az container show --resource-group myResourceGroup --name myContainerGroup --output table
```

Если вы хотите просмотреть запущенное приложение, перейдите к его IP-адресу в своем браузере. Например, в этом примере выходных данных используется IP-адрес `52.168.26.124`:

```console
Name              ResourceGroup    Status    Image                                                                                               IP:ports              Network    CPU/Memory       OsType    Location
----------------  ---------------  --------  --------------------------------------------------------------------------------------------------  --------------------  ---------  ---------------  --------  ----------
myContainerGroup  danlep0318r      Running   mcr.microsoft.com/azuredocs/aci-tutorial-sidecar,mcr.microsoft.com/azuredocs/aci-helloworld:latest  20.42.26.114:80,8080  Public     1.0 core/1.5 gb  Linux     eastus
```

## <a name="view-container-logs"></a>Просмотр журналов контейнеров

Просмотрите выходные данные журнала контейнера с помощью команды [az container logs][az-container-logs]. Аргумент `--container-name` определяет контейнер, из которого извлекаются журналы. В этом примере указывается контейнер `aci-tutorial-app`.

```azurecli-interactive
az container logs --resource-group myResourceGroup --name myContainerGroup --container-name aci-tutorial-app
```

Выходные данные:

```console
listening on port 80
::1 - - [02/Jul/2020:23:17:48 +0000] "HEAD / HTTP/1.1" 200 1663 "-" "curl/7.54.0"
::1 - - [02/Jul/2020:23:17:51 +0000] "HEAD / HTTP/1.1" 200 1663 "-" "curl/7.54.0"
::1 - - [02/Jul/2020:23:17:54 +0000] "HEAD / HTTP/1.1" 200 1663 "-" "curl/7.54.0"
```

Чтобы просмотреть журналы для сопроводительного контейнера, выполните аналогичную команду с именем контейнера `aci-tutorial-sidecar`.

```azurecli-interactive
az container logs --resource-group myResourceGroup --name myContainerGroup --container-name aci-tutorial-sidecar
```

Выходные данные:

```console
Every 3s: curl -I http://localhost                          2020-07-02 20:36:41

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0  1663    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
HTTP/1.1 200 OK
X-Powered-By: Express
Accept-Ranges: bytes
Cache-Control: public, max-age=0
Last-Modified: Wed, 29 Nov 2017 06:40:40 GMT
ETag: W/"67f-16006818640"
Content-Type: text/html; charset=UTF-8
Content-Length: 1663
Date: Thu, 02 Jul 2020 20:36:41 GMT
Connection: keep-alive
```

Как видите, сопроводительное приложение периодически выполняет HTTP-запрос к основному веб-приложению через локальную сеть группы, чтобы убедиться, что оно работает. Пример сопроводительного приложения можно расширить, реализовав активацию оповещения при получении кода ответа HTTP, который отличается от `200 OK`.

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как применить YAML-файл для развертывания многоконтейнерной группы в службе "Экземпляры контейнеров Azure". Вы ознакомились с выполнением следующих задач:

> [!div class="checklist"]
> * Настройка YAML-файла для многоконтейнерной группы
> * Развертывание группы контейнеров
> * Просмотр журналов контейнеров.

Вы также можете определить многоконтейнерную группу с использованием [шаблона Resource Manager](container-instances-multi-container-group.md). Шаблон Resource Manager можно легко адаптировать под такие сценарии, в которых нужно развернуть дополнительные ресурсы служб Azure в группе контейнеров.

<!-- LINKS - External -->

<!-- LINKS - Internal -->
[aci-tutorial]: ./container-instances-tutorial-prepare-app.md
[az-container-create]: /cli/azure/container#az_container_create
[az-container-logs]: /cli/azure/container#az_container_logs
[az-container-show]: /cli/azure/container#az_container_show
[az-group-create]: /cli/azure/group#az_group_create
[az-deployment-group-create]: /cli/azure/deployment/group#az_deployment_group_create
[template-reference]: /azure/templates/microsoft.containerinstance/containergroups
