---
title: Развертывание рабочих нагрузок Azure IoT Edge (предварительная версия)
services: azure-arc
ms.service: azure-arc
ms.date: 02/19/2021
ms.topic: article
author: mlearned
ms.author: mlearned
description: Развертывание рабочих нагрузок Azure IoT Edge
keywords: Kubernetes, Arc, Azure, K8s, контейнеры
ms.openlocfilehash: c352455b69360df0b26f5aac38fc40ccb30fb9de
ms.sourcegitcommit: b4647f06c0953435af3cb24baaf6d15a5a761a9c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/02/2021
ms.locfileid: "101650419"
---
# <a name="deploy-azure-iot-edge-workloads-preview"></a>Развертывание рабочих нагрузок Azure IoT Edge (предварительная версия)

## <a name="overview"></a>Обзор

В Azure Arc и Azure IoT Edge легко дополнять возможности друг друга. 

Служба "Дуга Azure" предоставляет механизмы для операторов кластера для настройки базовых компонентов кластера и применения политик кластера. 

Azure IoT Edge позволяет операторам приложений удаленно развертывать рабочие нагрузки и управлять ими в нужном масштабе с помощью удобных облачных и двунаправленных коммуникаций. 

На схеме ниже показано отношение "Дуга Azure" и "Azure IoT Edge".

![Настройка IoT Arc](./media/edge-arc.png)

## <a name="pre-requisites"></a>Предварительные требования

* [Зарегистрируйте устройства IoT Edge ](../../iot-edge/quickstart-linux.md#register-an-iot-edge-device) и [разверните имитируемый модуль датчика температуры](../../iot-edge/quickstart-linux.md#deploy-a-module). Обратите внимание на строку подключения устройства для *Values. YAML* , упомянутого ниже.

* Для развертывания Kubernetes с помощью оператора Flux в Azure Arc используйте [поддержку IoT Edge](https://aka.ms/edgek8sdoc).

* Скачайте файл [*Values. YAML*](https://github.com/Azure/iotedge/blob/preview/iiot/kubernetes/charts/edge-kubernetes/values.yaml) для диаграммы IOT Edge Helm и замените `deviceConnectionString` заполнитель в конце файла на строку подключения, записанную ранее. При необходимости задайте другие поддерживаемые параметры установки диаграммы. Создайте пространство имен для рабочей нагрузки IoT Edge и создайте в ней секрет:

  ```
  $ kubectl create ns iotedge

  $ kubectl create secret generic dcs --from-file=fully-qualified-path-to-values.yaml --namespace iotedge
  ```

  Также можно настроить удаленно с помощью [примера конфигурации кластера](./tutorial-use-gitops-connected-cluster.md).

## <a name="connect-a-cluster"></a>Подключение кластера

Используйте `az` расширение Azure CLI `connectedk8s` для подключения кластера Kubernetes к службе "Дуга" Azure.

  ```
  az connectedk8s connect --name AzureArcIotEdge --resource-group AzureArcTest
  ```

## <a name="create-a-configuration-for-iot-edge"></a>Создание конфигурации для IoT Edge

[Пример репозитория Git](https://github.com/veyalla/edgearc) указывает на диаграмму IOT Edge Helm и ссылается на секрет, созданный в разделе Предварительные требования.

Используйте `az` расширение Azure CLI `k8s-configuration` , чтобы создать конфигурацию, связывающую подключенный кластер с репозиторием Git:

  ```
  az k8s-configuration create --name iotedge --cluster-name AzureArcIotEdge --resource-group AzureArcTest --operator-instance-name iotedge --operator-namespace azure-arc-iot-edge --enable-helm-operator --helm-operator-chart-version 0.6.0 --helm-operator-chart-values "--set helm.versions=v3" --repository-url "git://github.com/veyalla/edgearc.git" --cluster-scoped
  ```

Через несколько минут вы увидите модули рабочей нагрузки IoT Edge, развернутые в `iotedge` пространстве имен кластера. 

Просмотрите `SimulatedTemperatureSensor` журналы Pod в этом пространстве имен, чтобы увидеть создаваемые образцы значений. Вы также можете просматривать сообщения, которые поступают в Центр Интернета вещей, используя [расширение Azure IoT Hub Toolkit для Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-toolkit).

## <a name="cleanup"></a>Очистка

Удалите конфигурацию с помощью:

```
az k8s-configuration delete -g AzureArcTest --cluster-name AzureArcIotEdge --name iotedge
```

## <a name="next-steps"></a>Дальнейшие действия

[Использование Политики Azure для управления конфигураций кластера](./use-azure-policy.md)
