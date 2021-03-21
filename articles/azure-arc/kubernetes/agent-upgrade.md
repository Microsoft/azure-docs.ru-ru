---
title: Обновление агентов Kubernetes с включенной службой Arc Azure
services: azure-arc
ms.service: azure-arc
ms.date: 03/03/2021
ms.topic: article
author: shashankbarsin
ms.author: shasb
description: Обновление агента управления для Kubernetes с поддержкой дуги Azure
keywords: Kubernetes, Arc, Azure, K8s, контейнеры, агент, обновление
ms.openlocfilehash: 172654f655e594c295a8807b417c32d7849bd4b1
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102121921"
---
# <a name="upgrading-azure-arc-enabled-kubernetes-agents"></a>Обновление агентов Kubernetes с включенной службой Arc Azure

Служба "Дуга Azure" с поддержкой Kubernetes обеспечивает автоматическое обновление и возможность ручного обновления для своих агентов. Если вы используете параметр отключить автоматическое обновление и полагаетесь на обновление вручную, политика поддержки версий применима к агентам Arc и базовому кластеру Kubernetes.

## <a name="toggle-auto-upgrade-on-or-off-when-connecting-cluster-to-azure-arc"></a>Включение или отключение автоматического обновления при подключении кластера к службе "Дуга Azure"

Служба "Дуга Azure" с поддержкой Kubernetes предоставляет агенты с готовыми возможностями автоматического обновления.

Следующая команда подключает кластер к службе "Дуга Azure" с **включенным** автообновлением:

```console
az connectedk8s connect --name AzureArcTest1 --resource-group AzureArcTest
```

При включенном автоматическом обновлении агент опрашивает Azure каждый час на доступность более новой версии агентов. Если агент находит доступную более новую версию, он запускает обновление диаграммы Helm для агентов Arc Azure.

Чтобы отказаться от автоматического обновления, укажите `--disable-auto-upgrade` параметр при подключении кластера к службе "Дуга Azure". Следующая команда подключает кластер к службе "Дуга Azure" с **отключенным** автообновлением:

```console
az connectedk8s connect --name AzureArcTest1 --resource-group AzureArcTest --disable-auto-upgrade
```

> [!TIP]
> Если вы планируете отключить автоматическое обновление, обратитесь к [политике поддержки версий](#version-support-policy) для Kubernetes Azure с поддержкой ARC.

## <a name="toggle-auto-upgrade-onoff-after-connecting-cluster-to-azure-arc"></a>Включение или отключение автоматического обновления после подключения кластера к службе "Дуга Azure"

После подключения кластера к службе "Дуга Azure" можно переключить функцию автоматического обновления с помощью `az connectedk8s update` команды, как показано ниже:

```console
az connectedk8s update --name AzureArcTest1 --resource-group AzureArcTest --auto-upgrade false
```

## <a name="manually-upgrade-agents"></a>Обновление агентов вручную

Если автоматическое обновление для агентов отключено, можно вручную инициировать обновления для этих агентов с помощью `az connectedk8s upgrade` команды, как показано ниже.

```console
az connectedk8s upgrade -g AzureArcTest1 -n AzureArcTest --agent-version 1.0.1
```

Для управления версиями агентов служба "Дуга Azure", включенная в Kubernetes, соответствует стандартной [схеме семантического управления версиями](https://semver.org/) `MAJOR.MINOR.PATCH` . 

Каждое число в версии указывает на общую совместимость с предыдущей версией:

* **Основные версии** изменяются при наличии несовместимых обновлений API или обратной совместимости, которые могут быть разорваны.
* **Дополнительные версии** изменяются при обратной совместимости функциональных изменений с другими вспомогательными выпусками.
* Изменения **версий исправления** при исправлении ошибок, совместимых с обратной совместимостью.

## <a name="version-support-policy"></a>Политика поддержки версий

При создании проблем с поддержкой службы Arc Azure Kubernetes рекомендует следующую политику поддержки версий:

* У агентов Kubernetes, включенных в дугу Azure, есть окно поддержки "N-2", где "N" — это последний дополнительный выпуск агентов. 
  * Например, если служба ARC в Azure Kubernetes вводит 0.28. a, версии 0.28. a, 0.28. b, 0.27. c, 0.27. d, 0.26. e и 0.26. f поддерживаются в службе "Дуга Azure".

* Кластеры Kubernetes, подключающиеся к службе "Дуга Azure", имеют окно поддержки "N-2", где "N" — это последний стабильный дополнительный выпуск [вышестоящего Kubernetes](https://github.com/kubernetes/kubernetes/releases). 
  * Например, если Kubernetes вводит значение 1,20. a, в настоящее время версии 1,20. a, 1,20. b, 1.19. c, 1.19. d, 18E. e и 18E. f поддерживаются.

### <a name="how-often-are-minor-version-releases-of-azure-arc-enabled-kubernetes-available"></a>Как часто доступны дополнительные выпуски версии Kubernetes Azure с поддержкой Arc?

Одна младшая версия Kubernetes агентов Azure с поддержкой Arc выпускается примерно раз в месяц.

### <a name="what-happens-if-im-using-an-agent-version-or-a-kubernetes-version-outside-the-official-support-window"></a>Что произойдет, если я использую версию агента или версию Kubernetes за пределами официального окна поддержки?

"Вне поддержки" означает, что версии, которые вы используете, выходят за пределы "N-2" поддерживаемых версий агентов и вышестоящего кластера Kubernetes. Чтобы продолжить устранение проблемы с поддержкой, вам будет предложено обновить кластер и агенты до поддерживаемой версии.

## <a name="next-steps"></a>Дальнейшие действия

* Пошаговое руководство по [подключению кластера Kubernetes к службе "Дуга Azure](./connect-cluster.md)".
* У вас уже есть кластер Kubernetes, подключенный к службе "Дуга Azure"? [Создайте конфигурации в кластере Kubernetes с включенной Arc](./use-gitops-connected-cluster.md).
* Узнайте, как [использовать политику Azure для применения конфигураций в масштабе](./use-azure-policy.md).