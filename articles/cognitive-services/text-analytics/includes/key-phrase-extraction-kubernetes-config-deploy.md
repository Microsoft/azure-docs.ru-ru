---
title: извлечение ключевых фраз конфигурации Kubernetes и шаги развертывания
titleSuffix: Azure Cognitive Services
description: извлечение ключевых фраз конфигурации Kubernetes и шаги развертывания
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: 27c78566877f27e80ae5ae27c5250f228c7ae676
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96017905"
---
### <a name="deploy-the-key-phrase-extraction-container-to-an-aks-cluster"></a>Развертывание контейнера извлечение ключевых фраз в кластере AKS

1. Откройте Azure CLI и войдите в Azure.

    ```azurecli
    az login
    ```

1. Войдите в кластер AKS. Замените `your-cluster-name` и `your-resource-group` соответствующими значениями.

    ```azurecli
    az aks get-credentials -n your-cluster-name -g -your-resource-group
    ```

    После выполнения этой команды будет выводится сообщение следующего вида:

    ```output
    Merged "your-cluster-name" as current context in /home/username/.kube/config
    ```

    > [!WARNING]
    > Если в учетной записи Azure доступно несколько подписок `az aks get-credentials` , а команда возвращает ошибку, то распространенной проблемой является то, что вы используете неправильную подписку. Задайте контекст сеанса Azure CLI, чтобы использовать ту же подписку, в которой были созданы ресурсы, и повторите попытку.
    > ```azurecli
    >  az account set -s subscription-id
    > ```

1. Откройте текстовый редактор по желанию. В этом примере используется Visual Studio Code.

    ```console
    code .
    ```

1. В текстовом редакторе создайте новый файл с именем *кэйфрасе. YAML* и вставьте в него следующий YAML. Обязательно замените `billing/value` и `apikey/value` собственными сведениями.

    ```yaml
    apiVersion: apps/v1beta1
    kind: Deployment
    metadata:
      name: keyphrase
    spec:
      template:
        metadata:
          labels:
            app: keyphrase-app
        spec:
          containers:
          - name: keyphrase
            image: mcr.microsoft.com/azure-cognitive-services/keyphrase
            ports:
            - containerPort: 5000
            resources:
              requests:
                memory: 2Gi
                cpu: 1
              limits:
                memory: 4Gi
                cpu: 1
            env:
            - name: EULA
              value: "accept"
            - name: billing
              value: # {ENDPOINT_URI}
            - name: apikey
              value: # {API_KEY}
     
    --- 
    apiVersion: v1
    kind: Service
    metadata:
      name: keyphrase
    spec:
      type: LoadBalancer
      ports:
      - port: 5000
      selector:
        app: keyphrase-app
    ```

1. Сохраните файл и закройте текстовый редактор.
1. Выполните команду Kubernetes `apply` с файлом *кэйфрасе. YAML* в качестве целевого объекта:

    ```console
    kubectl apply -f keyphrase.yaml
    ```

    После того как команда успешно применит конфигурацию развертывания, появится сообщение следующего вида:

    ```output
    deployment.apps "keyphrase" created
    service "keyphrase" created
    ```
1. Убедитесь, что модуль был развернут:

    ```console
    kubectl get pods
    ```

    Выходные данные для состояния выполнения Pod:

    ```output
    NAME                         READY     STATUS    RESTARTS   AGE
    keyphrase-5c9ccdf575-mf6k5   1/1       Running   0          1m
    ```

1. Убедитесь, что служба доступна, и получите IP-адрес.

    ```console
    kubectl get services
    ```

    Выходные данные для состояния выполнения службы *кэйфрасе* в Pod:

    ```output
    NAME         TYPE           CLUSTER-IP    EXTERNAL-IP      PORT(S)          AGE
    kubernetes   ClusterIP      10.0.0.1      <none>           443/TCP          2m
    keyphrase    LoadBalancer   10.0.100.64   168.61.156.180   5000:31234/TCP   2m
    ```
