---
title: Поддерживаемые форматы содержимого
description: Сведения о форматах содержимого, поддерживаемых реестром контейнеров Azure, включая образы контейнеров, совместимые с DOCKER, Helmные диаграммы, изображения OCI и, например, артефакты OCI.
ms.topic: article
ms.date: 08/30/2019
ms.openlocfilehash: b2a54c65d149a27ed9eae85c3308d657ed3471a3
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100008338"
---
# <a name="content-formats-supported-in-azure-container-registry"></a>Форматы содержимого, поддерживаемые в Реестре контейнеров Azure

Используйте частный репозиторий в Реестре контейнеров Azure для управления одним из следующих форматов содержимого. 

## <a name="docker-compatible-container-images"></a>Образы контейнеров, совместимые с Docker

Поддерживаются следующие форматы образов контейнеров docker:

* [Манифест версии 2 образа Docker, схема 1](https://docs.docker.com/registry/spec/manifest-v2-1/)

* [Манифест образа DOCKER v2, схема 2](https://docs.docker.com/registry/spec/manifest-v2-2/) . включает списки манифестов, которые позволяют реестрам хранить [образы с несколькими архитектурами](push-multi-architecture-images.md) в одной `image:tag` ссылке.

## <a name="oci-images"></a>Изображения OCI

Реестр контейнеров Azure поддерживает образы, которые соответствуют [спецификации формата образа для инициативы Open Container (OCI)](https://github.com/opencontainers/image-spec/blob/master/spec.md), включая спецификацию необязательного [индекса изображений](https://github.com/opencontainers/image-spec/blob/master/image-index.md) . Форматы упаковки включают [Формат изображения в формате "только в единственном числе" (SIF)](https://github.com/sylabs/sif).

## <a name="oci-artifacts"></a>, Артефакты OCI

Реестр контейнеров Azure поддерживает [спецификацию распространения OCI](https://github.com/opencontainers/distribution-spec), независимые от поставщика, независимые от облака спецификации для хранения, совместного использования, защиты и развертывания образов контейнеров и других типов содержимого (артефактов). Спецификация позволяет реестру хранить широкий спектр артефактов в дополнение к образам контейнеров. Для отправки и извлечения артефактов используются инструменты, подходящие для артефакта. Пример см. в разделе [Отправка и извлечение артефакта OCI с помощью реестра контейнеров Azure](container-registry-oci-artifacts.md).

Дополнительные сведения о артефактах OCI см. в репозитории "реестр" в [хранилище "OCI" (Орас)](https://github.com/deislabs/oras) и репозитории [артефактов OCI](https://github.com/opencontainers/artifacts) на сайте GitHub.

## <a name="helm-charts"></a>Чарты Helm

Реестр контейнеров Azure может размещать репозитории для [диаграмм Helm](https://helm.sh/), формат упаковки, используемый для быстрого управления и развертывания приложений для Kubernetes. Поддерживается [клиент Helm](https://docs.helm.sh/using_helm/#installing-helm) версии 2 (2.11.0 или более поздней).

## <a name="next-steps"></a>Дальнейшие действия

* Узнайте, как [принудительно отправлять и извлекать](container-registry-get-started-docker-cli.md) образы с помощью Реестра контейнеров Azure.

* Используйте [задачи Реестра контейнеров Azure](container-registry-tasks-overview.md) для сборки и тестирования образов контейнера. 

* Используйте платформу [BuildKit Moby](https://github.com/moby/buildkit) для сборки и упаковки контейнеров в формате OCI.

* Настройте [репозиторий Helm](container-registry-helm-repos.md), размещенный в Реестре контейнеров Azure. 


