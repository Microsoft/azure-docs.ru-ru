---
title: Предоставление серверу WebSocket доступа к шлюзу приложений
description: В этой статье содержатся сведения о том, как предоставить серверу WebSocket доступ к шлюзу приложений с помощью контроллера входящего трафика для кластеров AKS.
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/4/2019
ms.author: caya
ms.openlocfilehash: 68d4ff7e4617136e4c58ce672f34de56e46f0229
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85207793"
---
# <a name="expose-a-websocket-server-to-application-gateway"></a>Предоставление серверу WebSocket доступа к шлюзу приложений

Как описано в документации по шлюзу приложений версии 2, она [обеспечивает встроенную поддержку протоколов WebSocket и HTTP/2](features.md#websocket-and-http2-traffic). Обратите внимание, что для шлюза приложений и входящих в Kubernetes данных нет настраиваемых пользователем параметров, позволяющих выборочно включать или отключать поддержку WebSocket.

В Kubernetes Deployment YAML ниже показана минимальная конфигурация, используемая для развертывания сервера WebSocket, что аналогично развертыванию обычного веб-сервера.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: websocket-server
spec:
  selector:
    matchLabels:
      app: ws-app
  replicas: 2
  template:
    metadata:
      labels:
        app: ws-app
    spec:
      containers:
        - name: websocket-app
          imagePullPolicy: Always
          image: your-container-repo.azurecr.io/websockets-app
          ports:
            - containerPort: 8888
      imagePullSecrets:
        - name: azure-container-registry-credentials

---

apiVersion: v1
kind: Service
metadata:
  name: websocket-app-service
spec:
  selector:
    app: ws-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8888

---

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: websocket-repeater
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway
spec:
  rules:
    - host: ws.contoso.com
      http:
        paths:
          - backend:
              serviceName: websocket-app-service
              servicePort: 80
```

Учитывая, что все необходимые компоненты выполнены и у вас есть шлюз приложений, управляемый Kubernetes входящими в AKS, развертывание выше может привести к тому, что сервер WebSockets будет предоставляться через порт 80 общедоступного IP-адреса шлюза приложений и `ws.contoso.com` домена.

Следующая команда заключается в проверке развертывания сервера WebSocket:
```shell
curl -i -N -H "Connection: Upgrade" \
        -H "Upgrade: websocket" \
        -H "Origin: http://localhost" \
        -H "Host: ws.contoso.com" \
        -H "Sec-Websocket-Version: 13" \
        -H "Sec-WebSocket-Key: 123" \
        http://1.2.3.4:80/ws
```

## <a name="websocket-health-probes"></a>Зонды работоспособности WebSocket

Если в развертывании не определены явно проверки работоспособности, шлюз приложений попытается получить HTTP-запрос в конечной точке сервера WebSocket.
В зависимости от реализации сервера ([вот, что мы любим](https://github.com/gorilla/websocket/blob/master/examples/chat/main.go)) могут потребоваться определенные заголовки WebSocket (например, `Sec-Websocket-Version` ).
Так как шлюз приложений не добавляет заголовки WebSocket, скорее всего, запрос проверки работоспособности шлюза приложений с сервера WebSocket будет иметь значение `400 Bad Request` .
В результате шлюз приложений помечает модули Pod как неработоспособные, что в конечном итоге приведет к созданию `502 Bad Gateway` для потребителей сервера WebSocket.
Чтобы избежать этого, может потребоваться добавить обработчик HTTP GET для проверки работоспособности сервера ( `/health` например, который возвращает `200 OK` ).
