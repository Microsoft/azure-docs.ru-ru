---
title: Установка IoT Edge в Kubernetes | Документация Майкрософт
description: Узнайте, как установить IoT Edge в Kubernetes с помощью локальной среды кластера разработки
author: kgremban
manager: philmea
ms.author: veyalla
ms.date: 04/26/2019
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: d5fc7b6b480056b56a2776cebd0fa87a5b96f9f0
ms.sourcegitcommit: 5f32f03eeb892bf0d023b23bd709e642d1812696
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/12/2021
ms.locfileid: "103201674"
---
# <a name="how-to-install-iot-edge-on-kubernetes-preview"></a>Установка IoT Edge в Kubernetes (Предварительная версия)

[!INCLUDE [iot-edge-version-all-supported](../../includes/iot-edge-version-all-supported.md)]

IoT Edge можно интегрировать с Kubernetes, используя его как устойчивый высокодоступный уровень инфраструктуры. Ниже показано, где эта поддержка подходит на высоком уровне IoT Edge решения:

![K8S, введение](./media/how-to-install-iot-edge-kubernetes/kubernetes-model.png)

>[!TIP]
>Хорошей моделью для этой интеграции является обдумывание Kubernetes как другой операционной среды, IoT Edge приложения могут работать в дополнение к Linux и Windows.

## <a name="architecture"></a>Architecture 
В Kubernetes для развертываний пограничных рабочих нагрузок IoT Edge предоставляет *настраиваемое определение ресурса* (CRD). IoT Edge агент предполагает роль  *контроллера CRD* , который согласовывает требуемое облачное состояние и состояние локального кластера.

Время существования модуля управляется планировщиком Kubernetes, который поддерживает доступность модуля и выбирает их размещение. IoT Edge управляет платформой приложений пограничной среды, последовательно выполняя согласованность требуемого состояния, указанного в центре Интернета вещей, с состоянием в пограничном кластере. Модель приложения по-прежнему является знакомой моделью, основанной на IoT Edge модулях и маршрутах. Контроллер IoT Edge Agent выполняет *автоматическую* модель приложения IOT Edge перевода в собственные конструкции Kubernetes, такие как модули Pod, развертывания, службы и т. д.

Ниже приведена схема архитектуры высокого уровня.

![kubernetesная дуга](./media/how-to-install-iot-edge-kubernetes/publicpreview-refresh-kubernetes.png)

Каждый компонент развертывания пограничной области ограничен пространством имен Kubernetes, характерным для устройства, что делает возможным совместное использование одних и тех же ресурсов кластера между несколькими пограничными устройствами и их развертываниями.

>[!NOTE]
>IoT Edge в Kubernetes доступна в [общедоступной предварительной версии](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="tutorials-and-references"></a>Учебники и справочники 

Дополнительные сведения, в том числе подробные учебники и ссылки, см. [IOT EDGE на мини-сайте документации по Kubernetes Preview](https://aka.ms/edgek8sdoc) .
