---
title: Предоставление службы AKS по протоколу HTTP или HTTPS с помощью шлюза приложений
description: В этой статье содержатся сведения о том, как предоставить службу AKS по протоколу HTTP или HTTPS с помощью шлюза приложений.
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/4/2019
ms.author: caya
ms.openlocfilehash: 3b816ddc0eccf8c406cfed37d6bfc594e27d3629
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85850350"
---
# <a name="expose-an-aks-service-over-http-or-https-using-application-gateway"></a>Предоставление службы AKS по протоколу HTTP или HTTPS с помощью шлюза приложений 

Эти учебники помогут продемонстрировать использование [Kubernetesных ресурсов](https://kubernetes.io/docs/concepts/services-networking/ingress/) для предоставления примера службы Kubernetes через [шлюз приложений Azure](https://azure.microsoft.com/services/application-gateway/) по протоколу HTTP или HTTPS.

## <a name="prerequisites"></a>Предварительные требования

- Установленная `ingress-azure` Диаграмма Helm.
  - [**Развертывание нуля**](ingress-controller-install-new.md). Если вы начинаете с нуля, ознакомьтесь с инструкциями по установке, в которых описаны действия по развертыванию кластера AKS с помощью шлюза приложений и установке контроллера входящего трафика шлюза приложений в кластере AKS.
  - [**Развертывание серия статей Brownfield**](ingress-controller-install-existing.md). при наличии существующего кластера AKS и шлюза приложений ознакомьтесь с этими инструкциями, чтобы установить контроллер входящего трафика шлюза приложений в кластере AKS.
- Если вы хотите использовать протокол HTTPS для этого приложения, вам потребуется сертификат x509 и его закрытый ключ.

## <a name="deploy-guestbook-application"></a>Развертывание `guestbook` приложения

Приложение гостевой книги — это каноническое приложение Kubernetes, которое объединяет интерфейс веб-интерфейса, серверную часть и базу данных Redis. По умолчанию `guestbook` предоставляет свое приложение через службу с именем `frontend` по порту `80` . Без Kubernetesного ресурса Служба недоступна из-за пределов кластера AKS. Для доступа к приложению по протоколам HTTP и HTTPS мы будем использовать входящие ресурсы приложения и программы установки.

Следуйте приведенным ниже инструкциям, чтобы развернуть приложение гостевой книги.

1. Загрузить `guestbook-all-in-one.yaml` отсюда [](https://raw.githubusercontent.com/kubernetes/examples/master/guestbook/all-in-one/guestbook-all-in-one.yaml)
1. Разверните `guestbook-all-in-one.yaml` в кластер AKS, выполнив

  ```bash
  kubectl apply -f guestbook-all-in-one.yaml
  ```

Теперь `guestbook` приложение развернуто.

## <a name="expose-services-over-http"></a>Предоставление служб по протоколу HTTP

Чтобы предоставить доступ к приложению гостевой книги, мы будем использовать следующий ресурс входящих данных:

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: guestbook
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway
spec:
  rules:
  - http:
      paths:
      - backend:
          serviceName: frontend
          servicePort: 80
```

Этот входящий трафик будет предоставлять `frontend` службу `guestbook-all-in-one` развертывания в качестве серверной части шлюза приложений по умолчанию.

Сохраните указанный выше ресурс как `ing-guestbook.yaml` .

1. `ing-guestbook.yaml`Выполните развертывание, выполнив команду.

    ```bash
    kubectl apply -f ing-guestbook.yaml
    ```

1. Проверьте состояние развертывания в журнале входящего контроллера.

Теперь `guestbook` приложение должно быть доступно. Это можно проверить, посетив общедоступный адрес шлюза приложений.

## <a name="expose-services-over-https"></a>Предоставление служб по протоколу HTTPS

### <a name="without-specified-hostname"></a>Без указанного имени узла

Без указания имени узла служба гостевой службы будет доступна для всех имен узлов, указывающих на шлюз приложений.

1. Перед развертыванием входящего трафика необходимо создать секрет kubernetes для размещения сертификата и закрытого ключа. Вы можете создать секрет kubernetes, выполнив

    ```bash
    kubectl create secret tls <guestbook-secret-name> --key <path-to-key> --cert <path-to-cert>
    ```

1. Определите следующий входной параметр. В списке входящих данных укажите имя секрета в `secretName` разделе.

    ```yaml
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: guestbook
      annotations:
        kubernetes.io/ingress.class: azure/application-gateway
    spec:
      tls:
        - secretName: <guestbook-secret-name>
      rules:
      - http:
          paths:
          - backend:
              serviceName: frontend
              servicePort: 80
    ```

    > [!NOTE] 
    > Замените `<guestbook-secret-name>` в указанном выше ресурсе с именем секрета. Сохраните указанный выше ресурс входящих данных в имени файла `ing-guestbook-tls.yaml` .

1. Выполните развертывание с помощью команды with-гостевой-TLS. YAML, выполнив

    ```bash
    kubectl apply -f ing-guestbook-tls.yaml
    ```

1. Проверьте состояние развертывания в журнале входящего контроллера.

Теперь `guestbook` приложение будет доступно как по протоколам HTTP, так и по протоколу HTTPS.

### <a name="with-specified-hostname"></a>С указанным именем узла

Можно также указать имя узла для входящего трафика для мультиплексирования конфигураций и служб TLS.
Указав имя узла, служба гостевой службы будет доступна только на указанном узле.

1. Определите следующий входной параметр.
    В списке входящих данных укажите имя секрета в `secretName` разделе, а затем замените имя узла в `hosts` разделе соответствующим образом.

    ```yaml
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: guestbook
      annotations:
        kubernetes.io/ingress.class: azure/application-gateway
    spec:
      tls:
        - hosts:
          - <guestbook.contoso.com>
          secretName: <guestbook-secret-name>
      rules:
      - host: <guestbook.contoso.com>
        http:
          paths:
          - backend:
              serviceName: frontend
              servicePort: 80
    ```

1. Развертывание `ing-guestbook-tls-sni.yaml` с запуском

    ```bash
    kubectl apply -f ing-guestbook-tls-sni.yaml
    ```

1. Проверьте состояние развертывания в журнале входящего контроллера.

Теперь `guestbook` приложение будет доступно только для HTTP и HTTPS на указанном узле ( `<guestbook.contoso.com>` в этом примере).

## <a name="integrate-with-other-services"></a>Интеграция с другими службами

Следующий входящий трафик позволит добавить дополнительные пути в этот входящий трафик и перенаправить эти пути в другие службы:

```yaml
apiVersion: extensions/v1beta1
  kind: Ingress
  metadata:
    name: guestbook
    annotations:
      kubernetes.io/ingress.class: azure/application-gateway
  spec:
    rules:
    - http:
        paths:
        - path: </other/*>
          backend:
            serviceName: <other-service>
            servicePort: 80
        - backend:
            serviceName: frontend
            servicePort: 80
```
