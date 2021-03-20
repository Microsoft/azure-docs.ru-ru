---
title: Подключить том emptyDir к группе контейнеров
description: Узнайте, как подключить том emptyDir для обмена данными между контейнерами в группе контейнеров службы "Экземпляры контейнеров Azure"
ms.topic: article
ms.date: 01/31/2020
ms.openlocfilehash: 64a3c83008f163167528a5e5987fe2316942d5bc
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "77117737"
---
# <a name="mount-an-emptydir-volume-in-azure-container-instances"></a>Подключение тома emptyDir в службе "Экземпляры контейнеров Azure"

Узнайте, как подключить том *emptyDir* для обмена данными между контейнерами в группе контейнеров службы "Экземпляры контейнеров Azure". Используйте тома *emptyDir* как временные кэши для контейнерных рабочих нагрузок.

> [!NOTE]
> Сейчас подключение тома *emptyDir* поддерживается только для контейнеров Linux. Пока мы работаем над переносом всех компонентов в контейнеры Windows, в [обзоре](container-instances-overview.md#linux-and-windows-containers)можно найти различия в текущих платформах.

## <a name="emptydir-volume"></a>Том emptyDir

Том *emptyDir* предоставляет доступный для записи каталог каждому контейнеру в группе контейнеров. Контейнеры в группе могут считывать и записывать одни и те же файлы в томе. Также их можно подключать с использованием одинаковых или разных путей в каждом контейнере.

Некоторые примеры использования тома *emptyDir*:

* область временных файлов;
* контрольные точки во время длительных задач;
* хранение данных, полученных сопроводительным контейнером и обслуживаемых контейнером приложения.

В случае сбоев контейнеров данные в томе *emptyDir* сохраняются. Но при перезапуске контейнеров сохранность данных в томе *emptyDir* не гарантируется. Если вы останавливаете группу контейнеров, том *emptyDir* не сохраняется.

Максимальный размер тома *emptyDir* для Linux составляет 50 ГБ.

## <a name="mount-an-emptydir-volume"></a>Подключение тома emptyDir

Чтобы подключить том emptyDir в экземпляре контейнера, можно выполнить развертывание с помощью [шаблона Azure Resource Manager](/azure/templates/microsoft.containerinstance/containergroups), [файла YAML](container-instances-reference-yaml.md)или других программных методов для развертывания группы контейнеров.

Сначала заполните `volumes` массив в разделе файла группы контейнеров `properties` . Затем для каждого контейнера в группе контейнеров, в которой нужно подключить том *emptyDir*, заполните массив `volumeMounts` в разделе `properties` определения контейнера.

Например, при помощи следующего шаблона Resource Manager создается группа из двух контейнеров, в каждом из которых подключается том *emptyDir*:

<!-- https://github.com/Azure/azure-docs-json-samples/blob/master/container-instances/aci-deploy-volume-emptydir.json -->
[!code-json[volume-emptydir](~/azure-docs-json-samples/container-instances/aci-deploy-volume-emptydir.json)]

Примеры развертывания группы контейнеров см. в статье [развертывание многоконтейнерной группы с помощью шаблона диспетчер ресурсов](container-instances-multi-container-group.md) и [развертывание многоконтейнерной группы с помощью файла YAML](container-instances-multi-container-yaml.md).

## <a name="next-steps"></a>Дальнейшие действия

Сведения о подключении других типов томов в службе "Экземпляры контейнеров Azure" см. в следующих статьях:

* [Mount an Azure file share in Azure Container Instances](container-instances-volume-azure-files.md) (Подключение файлового ресурса Azure с помощью службы "Экземпляры контейнеров Azure")
* [Подключение тома gitRepo в службе "Экземпляры контейнеров Azure"](container-instances-volume-gitrepo.md)
* [Подключение тома secret в службе "Экземпляры контейнеров Azure"](container-instances-volume-secret.md)