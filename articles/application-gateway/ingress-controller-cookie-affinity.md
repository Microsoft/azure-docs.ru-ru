---
title: Включение сходства на основе файлов cookie с помощью шлюза приложений
description: В этой статье содержатся сведения о том, как включить сходство на основе файлов cookie с шлюзом приложений.
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/4/2019
ms.author: caya
ms.openlocfilehash: a7806cf9518090539ba540a9a522af1aae2691f0
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93397422"
---
# <a name="enable-cookie-based-affinity-with-an-application-gateway"></a>Включение сходства на основе файлов cookie с шлюзом приложений
Как описано в [документации по шлюзу приложений Azure](./application-gateway-components.md#http-settings), шлюз приложений поддерживает сопоставление на основе файлов cookie. Это означает, что он может направить последующий трафик из сеанса пользователя на тот же сервер для обработки.

## <a name="example"></a>Пример
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: guestbook
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway
    appgw.ingress.kubernetes.io/cookie-based-affinity: "true"
spec:
  rules:
  - http:
      paths:
      - backend:
          serviceName: frontend
          servicePort: 80
```