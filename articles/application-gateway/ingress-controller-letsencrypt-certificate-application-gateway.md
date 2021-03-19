---
title: Использование сертификатов LetsEncrypt.org с шлюзом приложений
description: В этой статье содержатся сведения о том, как получить сертификат от LetsEncrypt.org и использовать его в шлюзе приложений для кластеров AKS.
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/4/2019
ms.author: caya
ms.openlocfilehash: df8722e8160538daa1535711092790dbb2405097
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "84807029"
---
# <a name="use-certificates-with-letsencryptorg-on-application-gateway-for-aks-clusters"></a>Использование сертификатов с LetsEncrypt.org в шлюзе приложений для кластеров AKS

В этом разделе вы настраиваете AKS для использования [LetsEncrypt.org](https://letsencrypt.org/) и автоматически получаете сертификат TLS/SSL для вашего домена. Сертификат будет установлен в шлюзе приложений, который будет выполнять завершение SSL/TLS для кластера AKS. Описанная здесь программа установки использует надстройку [CERT-Manager](https://github.com/jetstack/cert-manager) Kubernetes, которая автоматизирует создание и управление сертификатами.

Выполните следующие действия, чтобы установить [Диспетчер сертификатов](https://docs.cert-manager.io) в имеющемся кластере AKS.

1. Диаграмма Helm

    Выполните следующий скрипт, чтобы установить `cert-manager` диаграмму Helm. В результате:

    - Создание нового `cert-manager` пространства имен на AKS
    - Создайте следующий КРДС: сертификат, запрос, Клустериссуер, издатель, заказ
    - Установка диаграммы диспетчера сертификатов (из [docs.CERT-Manager.IO)](https://docs.cert-manager.io/en/latest/getting-started/install/kubernetes.html#steps)

    ```bash
    #!/bin/bash

    # Install the CustomResourceDefinition resources separately
    kubectl apply -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.8/deploy/manifests/00-crds.yaml

    # Create the namespace for cert-manager
    kubectl create namespace cert-manager

    # Label the cert-manager namespace to disable resource validation
    kubectl label namespace cert-manager certmanager.k8s.io/disable-validation=true

    # Add the Jetstack Helm repository
    helm repo add jetstack https://charts.jetstack.io

    # Update your local Helm chart repository cache
    helm repo update

    # Install the cert-manager Helm chart
    helm install \
      --name cert-manager \
      --namespace cert-manager \
      --version v0.8.0 \
      jetstack/cert-manager
    ```

2. Ресурс Клустериссуер

    Создайте `ClusterIssuer` ресурс. Он необходим `cert-manager` для представления `Lets Encrypt` центра сертификации, в котором будут получены подписанные сертификаты.

    При использовании ресурса, не являющегося пространством имен `ClusterIssuer` , диспетчер сертификатов выдает сертификаты, которые можно использовать из нескольких пространств имен. `Let’s Encrypt` использует протокол ACME, чтобы проверить, что вы управляете заданным доменным именем и выдаете сертификат. Дополнительные сведения о настройке `ClusterIssuer` свойств см. [здесь](https://docs.cert-manager.io/en/latest/tasks/issuers/index.html). `ClusterIssuer` предложит `cert-manager` выдавать сертификаты с помощью `Lets Encrypt` промежуточной среды, используемой для тестирования (корневой сертификат отсутствует в магазинах доверия браузера или клиента).

    Тип запроса по умолчанию в YAML ниже — `http01` . Другие проблемы задокументированы в [типах letsencrypt.org-Challenge](https://letsencrypt.org/docs/challenge-types/)

    > [!IMPORTANT] 
    > Обновите `<YOUR.EMAIL@ADDRESS>` YAML ниже

    ```bash
    #!/bin/bash
    kubectl apply -f - <<EOF
    apiVersion: certmanager.k8s.io/v1alpha1
    kind: ClusterIssuer
    metadata:
    name: letsencrypt-staging
    spec:
    acme:
        # You must replace this email address with your own.
        # Let's Encrypt will use this to contact you about expiring
        # certificates, and issues related to your account.
        email: <YOUR.EMAIL@ADDRESS>
        # ACME server URL for Let’s Encrypt’s staging environment.
        # The staging environment will not issue trusted certificates but is
        # used to ensure that the verification process is working properly
        # before moving to production
        server: https://acme-staging-v02.api.letsencrypt.org/directory
        privateKeySecretRef:
        # Secret resource used to store the account's private key.
        name: example-issuer-account-key
        # Enable the HTTP-01 challenge provider
        # you prove ownership of a domain by ensuring that a particular
        # file is present at the domain
        http01: {}
    EOF
    ```

3. Развертывание приложения

    Создайте ресурс входящего трафика, чтобы предоставить приложению доступ к нему с `guestbook` помощью шлюза приложений с сертификатом.

    Убедитесь, что шлюз приложений имеет общедоступную IP-конфигурацию с DNS-именем (либо с помощью домена по умолчанию `azure.com` , либо подготавливает `Azure DNS Zone` службу, и назначьте собственный пользовательский домен).
    Обратите внимание на заметку `certmanager.k8s.io/cluster-issuer: letsencrypt-staging` , которая сообщает менеджеру CERT о необходимости обрабатывать ресурс с тегами.

    > [!IMPORTANT] 
    > Обновите `<PLACEHOLDERS.COM>` YAML ниже с помощью собственного домена (или шлюза приложений), например "KH-AKS-Ingress.westeurope.cloudapp.Azure.com".

    ```bash
    kubectl apply -f - <<EOF
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
    name: guestbook-letsencrypt-staging
    annotations:
        kubernetes.io/ingress.class: azure/application-gateway
        certmanager.k8s.io/cluster-issuer: letsencrypt-staging
    spec:
    tls:
    - hosts:
        - <PLACEHOLDERS.COM>
        secretName: guestbook-secret-name
    rules:
    - host: <PLACEHOLDERS.COM>
        http:
        paths:
        - backend:
            serviceName: frontend
            servicePort: 80
    EOF
    ```

    Через несколько секунд вы сможете получить доступ к `guestbook` службе через URL-адрес HTTPS шлюза приложений, используя автоматически выданный **промежуточный** `Lets Encrypt` сертификат.
    Браузер может предупредить о недопустимом центре сертификации. Промежуточный сертификат выдается `CN=Fake LE Intermediate X1` . Это означает, что система работала должным образом и вы готовы к рабочему сертификату.

4. Рабочий сертификат

    После успешной установки промежуточного сертификата можно переключиться на рабочий сервер ACME:
    1. Замените промежуточную аннотацию для ресурса входящих данных на: `certmanager.k8s.io/cluster-issuer: letsencrypt-prod`
    1. Удалите существующую промежуточную среду, `ClusterIssuer` созданную на предыдущем шаге, и создайте новую, заменив сервер ACME с КЛУСТЕРИССУЕР YAML выше на `https://acme-v02.api.letsencrypt.org/directory`

5. Истечение срока действия сертификата и его продление

    До `Lets Encrypt` истечения срока действия сертификата `cert-manager` будет автоматически обновлять сертификат в хранилище секретов Kubernetes. На этом этапе контроллер входящего трафика шлюза приложений применит обновленный секрет, указанный в входящих ресурсах, который используется для настройки шлюза приложений.
