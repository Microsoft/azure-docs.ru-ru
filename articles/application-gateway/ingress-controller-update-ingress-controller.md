---
title: Обновление входящего контроллера с помощью Helm
description: В этой статье содержатся сведения об обновлении входящего шлюза приложений с помощью Helm.
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/4/2019
ms.author: caya
ms.openlocfilehash: f20302a4993da1754255254ce6d69c000750d4ab
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "84806781"
---
# <a name="how-to-upgrade-application-gateway-ingress-controller-using-helm"></a>How to upgrade Application Gateway Ingress Controller using Helm (Обновление контроллера входящего трафика Шлюза приложений с помощью Helm) 

Контроллер входящего трафика шлюза приложений Azure для Kubernetes (АГИК) можно обновить с помощью репозитория Helm, размещенного в службе хранилища Azure.

Прежде чем начать процедуру обновления, убедитесь, что добавлен необходимый репозиторий:

- Просмотрите добавленные в настоящее время репозитории Helm с помощью:

    ```bash
    helm repo list
    ```

- Добавьте репозиторий АГИК с:

    ```bash
    helm repo add \
        application-gateway-kubernetes-ingress \
        https://appgwingress.blob.core.windows.net/ingress-azure-helm-package/
    ```

## <a name="upgrade"></a>Обновление

1. Обновите репозиторий АГИК Helm, чтобы получить последний выпуск:

    ```bash
    helm repo update
    ```

1. Просмотр доступных версий `application-gateway-kubernetes-ingress` диаграммы:

    ``` bash
    helm search -l application-gateway-kubernetes-ingress
    ```

    Пример ответа:

    ```bash
    NAME                                                    CHART VERSION   APP VERSION     DESCRIPTION
    application-gateway-kubernetes-ingress/ingress-azure    0.7.0-rc1       0.7.0-rc1       Use Azure Application Gateway as the ingress for an Azure...
    application-gateway-kubernetes-ingress/ingress-azure    0.6.0           0.6.0           Use Azure Application Gateway as the ingress for an Azure...
    ```

    Последняя доступная версия из приведенного выше списка: `0.7.0-rc1`

1. Просмотр установленных на данный момент диаграмм Helm:

    ```bash
    helm list
    ```

    Пример ответа:

    ```bash
    NAME            REVISION        UPDATED                         STATUS  CHART                   APP VERSION     NAMESPACE
    odd-billygoat   22              Fri Jun 21 15:56:06 2019        FAILED  ingress-azure-0.7.0-rc1 0.7.0-rc1       default
    ```

    Установка диаграммы Helm из образца ответа выше имеет имя `odd-billygoat` . Мы будем использовать это имя для остальных команд. Действительное имя развертывания, скорее всего, будет отличаться.

1. Обновите развертывание Helm до новой версии:

    ```bash
    helm upgrade \
        odd-billygoat \
        application-gateway-kubernetes-ingress/ingress-azure \
        --version 0.9.0-rc2
    ```

## <a name="rollback"></a>Откат

Если развертывание Helm завершится сбоем, можно вернуться к предыдущему выпуску.

1. Получите последний известный рабочий номер выпуска:

    ```bash
    helm history odd-billygoat
    ```

    Пример результатов выполнения:

    ```bash
    REVISION        UPDATED                         STATUS          CHART                   DESCRIPTION
    1               Mon Jun 17 13:49:42 2019        DEPLOYED        ingress-azure-0.6.0     Install complete
    2               Fri Jun 21 15:56:06 2019        FAILED          ingress-azure-xx        xxxx
    ```

    В примере выходных данных `helm history` команды он выглядит как Последнее успешное развертывание нашей `odd-billygoat` редакции. `1`

1. Откат к последней успешной редакции:

    ```bash
    helm rollback odd-billygoat 1
    ```
