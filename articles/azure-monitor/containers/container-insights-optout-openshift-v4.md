---
title: Как отключить мониторинг кластера Azure и Red Hat OpenShift v4 | Документация Майкрософт
description: В этой статье описывается, как можно отключить мониторинг кластера Azure Red Hat OpenShift и Red Hat OpenShift версии 4 с помощью аналитики контейнера.
ms.topic: conceptual
ms.date: 04/24/2020
ms.openlocfilehash: 09ca05a25ce9bb02b8a3d515acf060e2e9e7e8c2
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101731805"
---
# <a name="how-to-stop-monitoring-your-azure-and-red-hat-openshift-v4-cluster"></a>Как отключить мониторинг кластера Azure и Red Hat OpenShift v4

После включения мониторинга в кластере Azure Red Hat OpenShift и Red Hat OpenShift версии 4. x вы можете отключить мониторинг кластера с помощью аналитики контейнера, если вы решите, что вы больше не хотите его отслеживать. В этой статье показано, как это сделать.  

## <a name="how-to-stop-monitoring-using-helm"></a>Как прерывать мониторинг с помощью Helm

1. Чтобы сначала выяснить, что в кластере установлен выпуск Helm для Container Insights, выполните следующую команду Helm.

    ```
    helm list
    ```

    Результат должен выглядеть так:

    ```
    NAME                            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
    azmon-containers-release-1      default         3               2020-04-21 15:27:24.1201959 -0700 PDT   deployed        azuremonitor-containers-2.7.0   7.0.0-1
    ```

    *Азмон-Containers — Release-1* представляет выпуск Helm диаграммы для аналитики контейнера.

2. Чтобы удалить выпуск диаграммы, выполните следующую команду Helm.

    `helm delete <releaseName>`

    Пример

    `helm delete azmon-containers-release-1`

    Выпуск будет удален из кластера. Проверить можно, выполнив `helm list` команду:

    ```
    NAME                            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
    ```

Изменение конфигурации может занять несколько минут. Так как Helm отслеживает выпуски даже после их удаления, можно выполнить аудит журнала кластера и даже отменить удаление выпуска с помощью `helm rollback` .

## <a name="next-steps"></a>Дальнейшие действия

Если Рабочая область Log Analytics создана только для поддержки мониторинга кластера и больше не нужна, ее необходимо удалить вручную. Если вы не знакомы с удалением рабочей области, см. статью [удаление log Analytics рабочей области Azure](../logs/delete-workspace.md).