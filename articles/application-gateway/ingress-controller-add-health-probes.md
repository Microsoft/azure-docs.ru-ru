---
title: Добавление проб работоспособности в модули AKS Pod
description: В этой статье содержатся сведения о том, как добавить зонды работоспособности (готовность и/или Live) в AKS Pod с помощью шлюза приложений.
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/4/2019
ms.author: caya
ms.openlocfilehash: 8c8b8b0090877db7abc8fae0e44f928e8b10dcf5
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "84808003"
---
# <a name="add-health-probes-to-your-service"></a>Добавление проб работоспособности в службу
По умолчанию входной контроллер выполняет инициализацию HTTP GET для предоставляемых модулей Pod.
Свойства зонда можно настроить, добавив [проверку готовности или динамической проверки](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/) в `deployment` / `pod` спецификацию.

## <a name="with-readinessprobe-or-livenessprobe"></a>С `readinessProbe` или `livenessProbe`
```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: aspnetapp
spec:
  replicas: 3
  template:
    metadata:
      labels:
        service: site
    spec:
      containers:
      - name: aspnetapp
        image: mcr.microsoft.com/dotnet/core/samples:aspnetapp
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
        readinessProbe:
          httpGet:
            path: /
            port: 80
          periodSeconds: 3
          timeoutSeconds: 1
```

Справочник по API Kubernetes:
* [Зонды контейнеров](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes)
* [Действие HttpGet](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#httpgetaction-v1-core)

> [!NOTE]
> * `readinessProbe` и `livenessProbe` поддерживаются при настройке с помощью `httpGet` .
> * Проверка на портах, отличных от показанных в модуле Pod, сейчас не поддерживается.
> * `HttpHeaders`, `InitialDelaySeconds` , `SuccessThreshold` не поддерживаются.

##  <a name="without-readinessprobe-or-livenessprobe"></a>Без `readinessProbe` или `livenessProbe`
Если указанные выше проверки не предоставлены, то контроллер входящего трафика предполагала, что служба доступна в `Path` указанном для `backend-path-prefix` аннотации или `path` указании в `ingress` определении службы.

## <a name="default-values-for-health-probe"></a>Значения по умолчанию для проверки работоспособности
Для любого свойства, которое не может быть определено с помощью проверки готовности или жизни, устанавливаются значения по умолчанию.

| Свойство проверки шлюза приложений | Значение по умолчанию |
|-|-|
| `Path` | / |
| `Host` | localhost |
| `Protocol` | HTTP |
| `Timeout` | 30 |
| `Interval` | 30 |
| `UnhealthyThreshold` | 3 |