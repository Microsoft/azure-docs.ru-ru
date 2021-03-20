---
title: Подключить секретный том к группе контейнеров
description: Узнайте, как подключить том secret для хранения конфиденциальной информации, чтобы обеспечить доступ экземплярам контейнеров
ms.topic: article
ms.date: 07/02/2020
ms.openlocfilehash: ea82ba5994feaf102d4622eada284df431e004d0
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86169567"
---
# <a name="mount-a-secret-volume-in-azure-container-instances"></a>Подключение тома secret в службе "Экземпляры контейнеров Azure"

Используйте том *secret*, чтобы предоставить конфиденциальную информацию контейнерам в группе контейнеров. В томе *secret* в файлах хранятся секреты, к которым у контейнеров в группе контейнеров есть доступ. С помощью секретов в томе *secret* можно избежать добавления конфиденциальных данных, например ключей SSH или учетных данных базы данных, в коде приложения.

* После развертывания с секретами в группе контейнеров секретный том доступен *только для чтения*.
* Все тома secret поддерживает [tmpfs][tmpfs] — файловая система, сохраняющая данные в ОЗУ. Их содержимое никогда не записывается в хранилище для долговременного хранения данных.

> [!NOTE]
> Тома *secret* сейчас ограничены контейнерами Linux. Сведения о том, как передавать переменные безопасной среды для контейнеров Windows и Linux см. в статье [Настройка переменных среды](container-instances-environment-variables.md). Пока мы работаем над переносом всех компонентов в контейнеры Windows, в [обзоре](container-instances-overview.md#linux-and-windows-containers)можно найти различия в текущих платформах.

## <a name="mount-secret-volume---azure-cli"></a>Подключение тома secret в Azure CLI

Чтобы развернуть контейнер с одним или несколькими секретами с помощью Azure CLI, добавьте параметры `--secrets` и `--secrets-mount-path` в команду [az container create][az-container-create]. В этом примере размонтируется *секретный* том, состоящий из двух файлов, содержащих секреты "mysecret1" и "mysecret2" `/mnt/secrets` :

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \
    --name secret-volume-demo \
    --image mcr.microsoft.com/azuredocs/aci-helloworld \
    --secrets mysecret1="My first secret FOO" mysecret2="My second secret BAR" \
    --secrets-mount-path /mnt/secrets
```

В следующих выходных данных [az container exec][az-container-exec] показано открытие оболочки в выполняемом контейнере, список файлов в томе secret, а затем их содержимое:

```azurecli
az container exec \
  --resource-group myResourceGroup \
  --name secret-volume-demo --exec-command "/bin/sh"
```

```output
/usr/src/app # ls /mnt/secrets
mysecret1
mysecret2
/usr/src/app # cat /mnt/secrets/mysecret1
My first secret FOO
/usr/src/app # cat /mnt/secrets/mysecret2
My second secret BAR
/usr/src/app # exit
Bye.
```

## <a name="mount-secret-volume---yaml"></a>Подключение тома secret с помощью YAML

Вы также можете развернуть группы контейнеров с помощью Azure CLI и [шаблона YAML](container-instances-multi-container-yaml.md). При развертывании группы контейнеров, состоящей из нескольких контейнеров, рекомендуется использовать шаблон YAML.

При развертывании с помощью шаблона YAML значения секретов в шаблоне должны быть в **кодировке Base64**. Тем не менее значения секретов отображаются в виде обычного текста в файлах в контейнере.

Следующий шаблон YAML определяет группу контейнеров с одним контейнером, где том *secret* подключается к `/mnt/secrets`. Секретный том содержит два файла, содержащие секреты "mysecret1" и "mysecret2".

```yaml
apiVersion: '2019-12-01'
location: eastus
name: secret-volume-demo
properties:
  containers:
  - name: aci-tutorial-app
    properties:
      environmentVariables: []
      image: mcr.microsoft.com/azuredocs/aci-helloworld:latest
      ports: []
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
      volumeMounts:
      - mountPath: /mnt/secrets
        name: secretvolume1
  osType: Linux
  restartPolicy: Always
  volumes:
  - name: secretvolume1
    secret:
      mysecret1: TXkgZmlyc3Qgc2VjcmV0IEZPTwo=
      mysecret2: TXkgc2Vjb25kIHNlY3JldCBCQVIK
tags: {}
type: Microsoft.ContainerInstance/containerGroups
```

Для развертывания с помощью шаблона YAML сохраните предыдущий YAML в файл с именем `deploy-aci.yaml`, а затем выполните команду [az container create][az-container-create] с параметром `--file`:

```azurecli-interactive
# Deploy with YAML template
az container create \
  --resource-group myResourceGroup \
  --file deploy-aci.yaml
```

## <a name="mount-secret-volume---resource-manager"></a>Подключение тома secret с помощью Resource Manager

Помимо развертывания с помощью CLI и шаблона YAML группу контейнеров также можно развернуть с помощью [шаблона Azure Resource Manager](/azure/templates/microsoft.containerinstance/containergroups).

Сначала заполните массив `volumes` в разделе `properties` группы контейнеров шаблона. При развертывании с помощью шаблона Resource Manager значения секретов в шаблоне должны быть в **кодировке Base64**. Тем не менее значения секретов отображаются в виде обычного текста в файлах в контейнере.

Затем для каждого контейнера в группе контейнеров, в которой нужно подключить том *secret*, заполните массив `volumeMounts` в разделе `properties` определения контейнера.

Следующий шаблон Resource Manager определяет группу контейнеров с одним контейнером, где том *secret* подключается к `/mnt/secrets`. Том secret имеет два секрета, "mysecret1" и "mysecret2".

<!-- https://github.com/Azure/azure-docs-json-samples/blob/master/container-instances/aci-deploy-volume-secret.json -->
[!code-json[volume-secret](~/azure-docs-json-samples/container-instances/aci-deploy-volume-secret.json)]

Чтобы выполнить развертывание с помощью шаблона диспетчер ресурсов, сохраните предыдущий JSON в файл с именем `deploy-aci.json` , а затем выполните команду [AZ Deploy Group Create][az-deployment-group-create] с `--template-file` параметром:

```azurecli-interactive
# Deploy with Resource Manager template
az deployment group create \
  --resource-group myResourceGroup \
  --template-file deploy-aci.json
```

## <a name="next-steps"></a>Дальнейшие действия

### <a name="volumes"></a>Тома

Сведения о подключении других типов томов в службе "Экземпляры контейнеров Azure" см. в следующих статьях:

* [Mount an Azure file share in Azure Container Instances](container-instances-volume-azure-files.md) (Подключение файлового ресурса Azure с помощью службы "Экземпляры контейнеров Azure")
* [Подключение тома emptyDir в службе "Экземпляры контейнеров Azure"](container-instances-volume-emptydir.md)
* [Подключение тома gitRepo в службе "Экземпляры контейнеров Azure"](container-instances-volume-gitrepo.md)

### <a name="secure-environment-variables"></a>Защита переменных среды

Другой способ предоставления конфиденциальных данных в контейнеры (включая контейнеры Windows) — воспользоваться [защищенными переменными среды](container-instances-environment-variables.md#secure-values).

<!-- LINKS - External -->
[tmpfs]: https://wikipedia.org/wiki/Tmpfs

<!-- LINKS - Internal -->
[az-container-create]: /cli/azure/container#az-container-create
[az-container-exec]: /cli/azure/container#az-container-exec
[az-deployment-group-create]: /cli/azure/deployment/group#az-deployment-group-create
