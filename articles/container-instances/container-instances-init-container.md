---
title: Запуск контейнеров init
description: Выполните команду init Containers в службе "экземпляры контейнеров Azure", чтобы выполнить задачи установки в группе контейнеров до запуска контейнеров приложений.
ms.topic: article
ms.date: 06/01/2020
ms.openlocfilehash: 5a729263ee632eb9227694ec8684eb6889c6324b
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85954287"
---
# <a name="run-an-init-container-for-setup-tasks-in-a-container-group"></a>Запуск контейнера init для задач установки в группе контейнеров

Служба "экземпляры контейнеров Azure" поддерживает *контейнеры init* в группе контейнеров. Контейнеры init выполняются до завершения до запуска контейнера приложения или контейнеров. Как и в случае с [контейнерами Kubernetes init](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/), используйте один или несколько контейнеров init для выполнения логики инициализации для контейнеров приложений, таких как Настройка учетных записей, выполнение сценариев установки или настройка баз данных.

В этой статье показано, как использовать шаблон Azure Resource Manager для настройки группы контейнеров с помощью контейнера init.

## <a name="things-to-know"></a>Полезная информация

* **Версия API** — для развертывания контейнеров init требуется по крайней мере API экземпляров контейнеров Azure версии 2019-12-01. Развертывание с помощью `initContainers` свойства в [файле YAML](container-instances-multi-container-yaml.md) или [шаблоне диспетчер ресурсов](container-instances-multi-container-group.md).
* **Порядок выполнения** контейнеров-init выполняется в порядке, указанном в шаблоне, и перед другими контейнерами. По умолчанию можно указать не более 59 контейнеров init для каждой группы контейнеров. В группе должен быть хотя бы один контейнер без инициализации.
* **Среда размещения** — контейнеры инициализации выполняются на том же оборудовании, что и остальные контейнеры в группе.
* **Ресурсы** — не указываются ресурсы для контейнеров init. Им предоставляются общие ресурсы, такие как ЦП и память, доступные группе контейнеров. Во время выполнения контейнера инициализации другие контейнеры не выполняются в группе.
* **Поддерживаемые свойства** — контейнеры init могут использовать свойства группы, такие как тома, секреты и управляемые удостоверения. Однако они не могут использовать порты или IP-адреса, если они настроены для группы контейнеров. 
* **Политика перезапуска** — каждый контейнер инициализации должен успешно выйти, прежде чем следующий контейнер в группе начнет работу. Если контейнер init не завершает работу успешно, действие перезапуска зависит от [политики перезапуска](container-instances-restart-policy.md) , настроенной для группы.

    |Политика в группе  |Политика в init  |
    |---------|---------|
    |Всегда     |OnFailure (При сбое)         |
    |OnFailure (При сбое)     |OnFailure (При сбое)         |
    |Никогда     |Никогда         |
* **Расходы** . в группе контейнеров взимается плата за первое развертывание контейнера init.

## <a name="resource-manager-template-example"></a>Пример шаблона Resource Manager

Для начала скопируйте следующий код JSON в новый файл с именем `azuredeploy.json`. Шаблон настраивает группу контейнеров с одним контейнером init и двумя контейнерами приложений:

* Контейнер *init1* запускает образ [Бусибокс](https://hub.docker.com/_/busybox) из DOCKER Hub. Он находится в течение 60 секунд, а затем записывает строку командной строки в файл в [томе emptyDir](container-instances-volume-emptydir.md).
* Контейнеры приложений запускают `aci-wordcount` образ контейнера Майкрософт:
    * Контейнер *Гамлет* запускает приложение WordCount в конфигурации по умолчанию, считая частоты слов в Шекспир Play *Гамлет*.
    * Контейнер приложения *Джульетта* считывает строку командной строки из тома емптдир, чтобы запустить приложение WordCount, а не Шекспир *тексте пьесы Ромео и Джульетта*.

Дополнительные сведения и примеры использования `aci-wordcount` образа см. в разделе [Установка переменных среды в экземплярах контейнеров](container-instances-environment-variables.md).

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
      "containerGroupName": {
        "type": "string",
        "defaultValue": "myContainerGroup",
        "metadata": {
          "description": "Container Group name."
        }
      }
    },
    "resources": [
        {
            "name": "[parameters('containerGroupName')]",
            "type": "Microsoft.ContainerInstance/containerGroups",
            "apiVersion": "2019-12-01",
            "location": "[resourceGroup().location]",
            "properties": {
                "sku": "Standard",
                "initContainers": [
                {
                    "name": "init1",
                    "properties": {
                        "image": "busybox",
                        "environmentVariables": [],
                        "volumeMounts": [
                            {
                                "name": "emptydir1",
                                "mountPath": "/mnt/emptydir"
                            }
                        ],
                         "command": [
                            "/bin/sh",
                            "-c",
                            "sleep 60; echo python wordcount.py http://shakespeare.mit.edu/romeo_juliet/full.html > /mnt/emptydir/command_line.txt"
                        ]
                    }
                }
            ], 
                "containers": [
                    {
                        "name": "hamlet",
                        "properties": {
                            "image": "mcr.microsoft.com/azuredocs/aci-wordcount",
                            "volumeMounts": [
                            {
                                "name": "emptydir1",
                                "mountPath": "/mnt/emptydir"
                            }
                        ],
                            "environmentVariables": [
                             {
                                "name": "NumWords",
                                "value": "3"
                             },
                             {  "name": "MinLength",
                                "value": "5"
                             }
                            ],
                            "resources": {
                                "requests": {
                                    "memoryInGB": 1.0,
                                    "cpu": 1
                                }
                            }
                        }
                    },
                    {
                        "name": "juliet",
                        "properties": {
                            "image": "mcr.microsoft.com/azuredocs/aci-wordcount",
                            "volumeMounts": [
                            {
                                "name": "emptydir1",
                                "mountPath": "/mnt/emptydir"
                            }
                            ],
                            "command": [
                                "/bin/sh",
                                "-c",
                                "$(cat /mnt/emptydir/command_line.txt)"
                            ],
                            "environmentVariables": [
                             {
                                "name": "NumWords",
                                "value": "3"
                             },
                             {  "name": "MinLength",
                                "value": "5"
                             }
                            ],
                            "resources": {
                                "requests": {
                                    "memoryInGB": 1.0,
                                    "cpu": 1
                                }
                            }
                        }
                    }
                ],
                "restartPolicy": "OnFailure",
                "osType": "Linux",
                "volumes": [
                    {
                        "name": "emptydir1",
                        "emptyDir": {}
                    }
                ]           
            },
            "tags": {}
        }
    ]
}
```

## <a name="deploy-the-template"></a>Развертывание шаблона

Создайте группу ресурсов с помощью команды [az group create][az-group-create].

```azurecli
az group create --name myResourceGroup --location eastus
```

Разверните шаблон с помощью команды [AZ Deployment Group Create][az-deployment-group-create] .

```azurecli
az deployment group create \
  --resource-group myResourceGroup \
  --template-file azuredeploy.json
```

В группе с контейнером init время развертывания увеличивается из-за времени, необходимого для завершения контейнера или контейнеров инициализации.


## <a name="view-container-logs"></a>Просмотр журналов контейнеров

Чтобы убедиться, что контейнер инициализации успешно запущен, просмотрите выходные данные контейнеров приложений с помощью команды [AZ Container Logs][az-container-logs] . Аргумент `--container-name` определяет контейнер, из которого извлекаются журналы. В этом примере извлекаются журналы для контейнеров *Гамлет* и *Джульетта* , в которых отображаются различные выходные данные команды:

```azurecli
az container logs \
  --resource-group myResourceGroup \
  --name myContainerGroup \
  --container-name hamlet
```

Выходные данные:

```console
[('HAMLET', 386), ('HORATIO', 127), ('CLAUDIUS', 120)]
```

```azurecli
az container logs \
  --resource-group myResourceGroup \
  --name myContainerGroup \
  --container-name juliet
```

Выходные данные:

```console
[('ROMEO', 177), ('JULIET', 134), ('CAPULET', 119)]
```

## <a name="next-steps"></a>Дальнейшие действия

Контейнеры инициализации помогают выполнять задачи установки и инициализации для контейнеров приложений. Дополнительные сведения о выполнении контейнеров на основе задач см. [в статье выполнение контейнерных задач с помощью политик перезапуска](container-instances-restart-policy.md).

Служба "экземпляры контейнеров Azure" предоставляет другие возможности изменения поведения контейнеров приложений. Примеры приведены ниже.

* [Установка переменных среды в экземплярах контейнеров](container-instances-environment-variables.md)
* [Установка командной строки в экземпляре контейнера для переопределения операции командной строки по умолчанию](container-instances-start-command.md)


[az-group-create]: /cli/azure/group#az-group-create
[az-deployment-group-create]: /cli/azure/deployment/group#az-deployment-group-create
[az-container-logs]: /cli/azure/container#az-container-logs
