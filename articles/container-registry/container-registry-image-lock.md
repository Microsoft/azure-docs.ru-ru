---
title: Блокировка образов
description: Задайте атрибуты для образа или репозитория контейнера, чтобы их нельзя было удалить или перезазаписать в реестре контейнеров Azure.
ms.topic: article
ms.date: 09/30/2019
ms.openlocfilehash: da84767523bb6d948b71b1c1ad2ddaffb628354a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "77659702"
---
# <a name="lock-a-container-image-in-an-azure-container-registry"></a>Блокировка образа контейнера в реестре контейнеров Azure

В реестре контейнеров Azure можно заблокировать версию образа или репозиторий, чтобы его нельзя было удалить или обновить. Чтобы заблокировать образ или репозиторий, обновите его атрибуты с помощью команды Azure CLI [AZ записей в репозитории обновления][az-acr-repository-update]. 

Для работы с этой статьей необходимо запустить Azure CLI в Azure Cloud Shell или локально (рекомендуется версия 2.0.55 или более поздняя). Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][azure-cli].

> [!IMPORTANT]
> Эта статья не применяется для блокировки всего реестра, например с помощью **параметров > locks** в портал Azure или `az lock` командах в Azure CLI. Блокировка ресурса реестра не мешает создавать, обновлять или удалять данные в репозиториях. Блокировка реестра влияет только на операции управления, такие как добавление или удаление репликации, или удаление самого реестра. Дополнительные сведения см. в статьях [Блокировка ресурсов для предотвращения непредвиденных изменений](../azure-resource-manager/management/lock-resources.md).

## <a name="scenarios"></a>Сценарии

По умолчанию изображение с тегами в реестре контейнеров Azure является *изменяемым*, поэтому с соответствующими разрешениями можно многократно обновлять и отправлять в реестр образ с тем же тегом. Образы контейнеров также можно [Удалить](container-registry-delete.md) по мере необходимости. Такое поведение полезно при разработке образов и необходимости сохранять размер реестра.

Однако при развертывании образа контейнера в рабочей среде может потребоваться *неизменяемый* образ контейнера. Неизменяемый образ — это тот, который нельзя случайно удалить или перезаписать.

См. раздел [рекомендации по отстановке тегов и управления версиями](container-registry-image-tag-version.md) для стратегий добавления тегов и образов версий в реестр.

Используйте команду [AZ запись в репозитории обновления][az-acr-repository-update] , чтобы задать атрибуты репозитория, чтобы можно было:

* Блокировка версии образа или всего репозитория

* Защитите версию или репозиторий образа от удаления, но разрешите обновления

* Запретить операции чтения (извлечения) для версии образа или всего репозитория

Примеры см. в следующих разделах. 

## <a name="lock-an-image-or-repository"></a>Блокировка образа или репозитория 

### <a name="show-the-current-repository-attributes"></a>Отображение текущих атрибутов репозитория
Чтобы просмотреть текущие атрибуты репозитория, выполните следующую команду AZ запись в [репозитории][az-acr-repository-show] :

```azurecli
az acr repository show \
    --name myregistry --repository myrepo \
    --output jsonc
```

### <a name="show-the-current-image-attributes"></a>Отображение атрибутов текущего изображения
Чтобы просмотреть текущие атрибуты тега, выполните следующую команду AZ запись в [репозитории][az-acr-repository-show] :

```azurecli
az acr repository show \
    --name myregistry --image image:tag \
    --output jsonc
```

### <a name="lock-an-image-by-tag"></a>Заблокировать образ по тегу

Чтобы заблокировать изображение *myrepo/MyImage: Tag* в *myregistry*, выполните следующую команду [AZ запись репозитория Update][az-acr-repository-update] :

```azurecli
az acr repository update \
    --name myregistry --image myrepo/myimage:tag \
    --write-enabled false
```

### <a name="lock-an-image-by-manifest-digest"></a>Блокировка образа с помощью дайджеста манифеста

Чтобы заблокировать изображение *myrepo/MyImage* , определяемое дайджестом манифеста (хэш SHA-256, представленный в виде `sha256:...` ), выполните следующую команду. (Чтобы найти дайджест манифеста, связанный с одним или несколькими тегами изображений, выполните команду AZ запись в [репозитории "отобразить-манифесты][az-acr-repository-show-manifests] ".)

```azurecli
az acr repository update \
    --name myregistry --image myrepo/myimage@sha256:123456abcdefg \
    --write-enabled false
```

### <a name="lock-a-repository"></a>Блокировка репозитория

Чтобы заблокировать репозиторий *myrepo/MyImage* и все образы в нем, выполните следующую команду:

```azurecli
az acr repository update \
    --name myregistry --repository myrepo/myimage \
    --write-enabled false
```

## <a name="protect-an-image-or-repository-from-deletion"></a>Защита образа или репозитория от удаления

### <a name="protect-an-image-from-deletion"></a>Защита образа от удаления

Чтобы разрешить обновление изображения *myrepo/MyImage: Tag* , но не удалить его, выполните следующую команду:

```azurecli
az acr repository update \
    --name myregistry --image myrepo/myimage:tag \
    --delete-enabled false --write-enabled true
```

### <a name="protect-a-repository-from-deletion"></a>Защита репозитория от удаления

Следующая команда задает репозиторий *myrepo/MyImage* , поэтому его нельзя удалить. Отдельные образы по-прежнему могут быть обновлены или удалены.

```azurecli
az acr repository update \
    --name myregistry --repository myrepo/myimage \
    --delete-enabled false --write-enabled true
```

## <a name="prevent-read-operations-on-an-image-or-repository"></a>Запрет операций чтения для образа или репозитория

Чтобы запретить операции чтения (извлечения) в образе *myrepo/MyImage: Tag* , выполните следующую команду:

```azurecli
az acr repository update \
    --name myregistry --image myrepo/myimage:tag \
    --read-enabled false
```

Чтобы запретить операции чтения для всех образов в репозитории *myrepo/MyImage* , выполните следующую команду:

```azurecli
az acr repository update \
    --name myregistry --repository myrepo/myimage \
    --read-enabled false
```

## <a name="unlock-an-image-or-repository"></a>Разблокирование образа или репозитория

Чтобы восстановить поведение по умолчанию для изображения *myrepo/MyImage: Tag* , чтобы его можно было удалить и обновить, выполните следующую команду:

```azurecli
az acr repository update \
    --name myregistry --image myrepo/myimage:tag \
    --delete-enabled true --write-enabled true
```

Чтобы восстановить поведение по умолчанию репозитория *myrepo/MyImage* и всех образов, чтобы их можно было удалить и обновить, выполните следующую команду:

```azurecli
az acr repository update \
    --name myregistry --repository myrepo/myimage \
    --delete-enabled true --write-enabled true
```

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы узнали об использовании команды AZ запись в [репозитории Update][az-acr-repository-update] для предотвращения удаления или обновления версий изображений в репозитории. Дополнительные сведения о настройке дополнительных атрибутов см. в справочнике по командам [AZ контроля доступа на обновление репозитория][az-acr-repository-update] .

Чтобы просмотреть атрибуты, заданные для версии или репозитория образа, используйте команду [AZ запись в репозитории Показать][az-acr-repository-show] .

Дополнительные сведения об операциях удаления см. [в разделе Удаление образов контейнеров в реестре контейнеров Azure][container-registry-delete].

<!-- LINKS - Internal -->
[az-acr-repository-update]: /cli/azure/acr/repository#az-acr-repository-update
[az-acr-repository-show]: /cli/azure/acr/repository#az-acr-repository-show
[az-acr-repository-show-manifests]: /cli/azure/acr/repository#az-acr-repository-show-manifests
[azure-cli]: /cli/azure/install-azure-cli
[container-registry-delete]: container-registry-delete.md

