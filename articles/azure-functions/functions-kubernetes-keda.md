---
title: Функции Azure в Kubernetes с KEDA
description: Узнайте, как выполнять функции Azure в Kubernetes в облаке или локальной среде с помощью KEDA — управляемого событиями автоматического масштабирования на основе Kubernetes.
author: jeffhollan
ms.topic: conceptual
ms.date: 11/18/2019
ms.author: jehollan
ms.openlocfilehash: 525635ef40437fe308c52e2d5aba2c97ed8f20e7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92927538"
---
# <a name="azure-functions-on-kubernetes-with-keda"></a>Функции Azure в Kubernetes с KEDA

Среда выполнения Функций Azure обеспечивает гибкость при размещении в соответствии с вашими требованиями.  [KEDA](https://keda.sh) (управляемое событиями автоматическое масштабирование на основе Kubernetes) легко интегрируется со средой выполнения Функций Azure и инструментами для обеспечения масштабирования на основе событий в Kubernetes.

## <a name="how-kubernetes-based-functions-work"></a>Принципы работы функций на основе Kubernetes

Служба "Функции Azure" состоит из двух ключевых компонентов: среды выполнения и контроллера масштабирования.  Среда выполнения Функций запускает и выполняет код.  Среда выполнения включает в себя логику для активации, ведения журнала и управления выполнением функций.  Среда выполнения Функций Azure может быть запущена *где угодно*.  Второй компонент — контроллер масштабирования.  Контроллер масштабирования отслеживает скорость событий, предназначенных для функции, и заранее масштабирует количество экземпляров, в которых выполняется приложение.  Дополнительные сведения см. в статье [Масштабирование и размещение Функций Azure](functions-scale.md).

Функции на основе Kubernetes предоставляют среду выполнения Функций в [контейнере Docker](functions-create-function-linux-custom-image.md) с масштабированием на основе событий через KEDA.  KEDA может уменьшать горизонтальный масштаб до 0 экземпляров (при отсутствии событий) или увеличивать его до *n* экземпляров. Это достигается за счет предоставления пользовательских метрик для средства автомасштабирования Kubernetes (средство горизонтального автомасштабирования pod).  Использование контейнеров Функций с KEDA позволяет реплицировать возможности бессерверных функций в любом кластере Kubernetes.  Эти функции также можно развернуть с помощью компонента [виртуальных узлов Azure Kubernetes Services (AKS)](../aks/virtual-nodes-cli.md) для бессерверной инфраструктуры.

## <a name="managing-keda-and-functions-in-kubernetes"></a>Управление KEDA и функциями в Kubernetes

Для запуска Функций в кластере Kubernetes необходимо установить компонент KEDA. Этот компонент можно установить с помощью [Azure Functions Core Tools](functions-run-local.md).

### <a name="installing-with-helm"></a>Установка с помощью Helm

Существует несколько способов установки KEDA в любом кластере Kubernetes, включая Helm.  Варианты развертывания описаны на [сайте KEDA](https://keda.sh/docs/1.4/deploy/).

## <a name="deploying-a-function-app-to-kubernetes"></a>Развертывание приложения-функции в Kubernetes

Вы можете развернуть любое приложение-функцию в кластере Kubernetes с KEDA.  Так как функции выполняются в контейнере Docker, вашему проекту требуется `Dockerfile`.  Если у вас его еще нет, можно добавить Dockerfile, выполнив следующую команду в корне проекта Функций:

> [!NOTE]
> Основные средства автоматически создают Dockerfile для функций Azure, написанных на .NET, Node, Python или PowerShell. Для приложений функций, написанных на Java, Dockerfile необходимо создать вручную. Используйте [список образов](https://github.com/Azure/azure-functions-docker) функций Azure, чтобы найти правильный образ для создания функции Azure.

```cli
func init --docker-only
```

Чтобы создать образ и развернуть функции в Kubernetes, выполните следующую команду:

> [!NOTE]
> Core Tools будут использовать Docker CLI для создания и публикации образа. Убедитесь, что Docker уже установлен и подключен к вашей учетной записи с помощью `docker login`.

```cli
func kubernetes deploy --name <name-of-function-deployment> --registry <container-registry-username>
```

> Замените `<name-of-function-deployment>` на имя приложения-функции.

Команда deploy выполняет ряд действий:
1. Созданный ранее Dockerfile используется для создания локального образа для приложения-функции.
2. Локальный образ помечается и передается в реестр контейнеров, где пользователь вошел в систему.
3. Манифест создается и применяется к кластеру, который определяет `Deployment` ресурс Kubernetes, `ScaledObject` ресурс и `Secrets` , который включает переменные среды, импортированные из `local.settings.json` файла.

### <a name="deploying-a-function-app-from-a-private-registry"></a>Развертывание приложения-функции из частного реестра

Приведенный выше поток подходит и для частных реестров.  Если вы извлекаете образ контейнера из частного реестра, включите флаг `--pull-secret`, который ссылается на секрет Kubernetes, содержащий учетные данные частного реестра при запуске `func kubernetes deploy`.

## <a name="removing-a-function-app-from-kubernetes"></a>Удаление приложения-функции из Kubernetes

После развертывания можно удалить функцию, удалив связанный `Deployment`, `ScaledObject`, созданный `Secrets`.

```cli
kubectl delete deploy <name-of-function-deployment>
kubectl delete ScaledObject <name-of-function-deployment>
kubectl delete secret <name-of-function-deployment>
```

## <a name="uninstalling-keda-from-kubernetes"></a>Удаление KEDA из Kubernetes

Действия по удалению KEDA описаны [на сайте KEDA](https://keda.sh/docs/1.4/deploy/).

## <a name="supported-triggers-in-keda"></a>Поддерживаемые триггеры в KEDA

KEDA поддерживает следующие триггеры Функций Azure:

* [Очереди хранилища Azure](functions-bindings-storage-queue.md)
* [Очереди служебной шины Azure](functions-bindings-service-bus.md)
* [Центр событий Azure/Центр Интернета вещей](functions-bindings-event-hubs.md)
* [Apache Kafka](https://github.com/azure/azure-functions-kafka-extension)
* [Очередь RabbitMQ](https://github.com/azure/azure-functions-rabbitmq-extension)

### <a name="http-trigger-support"></a>Поддержка триггеров HTTP

Вы можете использовать Функции Azure, которые предоставляют триггеры HTTP, но KEDA не управляет ими напрямую.  Вы можете использовать триггер Prometheus KEDA, чтобы [масштабировать Функции Azure HTTP от 1 до *n* экземпляров](https://dev.to/anirudhgarg_99/scale-up-and-down-a-http-triggered-function-app-in-kubernetes-using-keda-4m42).

## <a name="next-steps"></a>Next Steps
Для получения дополнительных сведений см. следующие ресурсы:

* [Создание функции с помощью пользовательского образа](functions-create-function-linux-custom-image.md)
* [Как программировать и тестировать функции Azure в локальной среде](functions-develop-local.md)
* [Как работает план потребления Функций Azure](functions-scale.md)