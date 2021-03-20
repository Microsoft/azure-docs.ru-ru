---
title: Образы с несколькими архитектурами в реестре
description: Использование реестра контейнеров Azure для создания, импорта, хранения и развертывания многоархитектурных образов (с несколькими архитектурами)
ms.topic: article
ms.date: 02/07/2021
ms.custom: ''
ms.openlocfilehash: f8467cd3108ae4faea9ecb4c9d9ae339f476c311
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100012339"
---
# <a name="multi-architecture-images-in-your-azure-container-registry"></a>Образы с несколькими архитектурами в реестре контейнеров Azure

В этой статье представлены *многоархитектурные* образы (с *несколькими* архитектурами), а также способы создания, хранения и использования функций реестра контейнеров Azure. 

Многоуровневый образ — это тип образа контейнера, который может сочетать варианты для различных архитектур и иногда для разных операционных систем. При запуске образа с поддержкой нескольких архитектур клиенты контейнера автоматически выбирают вариант образа, соответствующий вашей операционной системе и архитектуре.

## <a name="manifests-and-manifest-lists"></a>Манифесты и списки манифестов

Многоуровневые образы основаны на манифестах образов и списках манифестов.

### <a name="manifest"></a>манифеста

Каждый образ контейнера представлен [манифестом](container-registry-concepts.md#manifest). Манифест — это JSON-файл, который однозначно определяет изображение, ссылаясь на его слои и соответствующие размеры. 

Базовый манифест `hello-world` образа Linux выглядит следующим образом:

  ```json
  {
    "schemaVersion": 2,
    "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
    "config": {
        "mediaType": "application/vnd.docker.container.image.v1+json",
        "size": 1510,
        "digest": "sha256:fbf289e99eb9bca977dae136fbe2a82b6b7d4c372474c9235adc1741675f587e"
      },
    "layers": [
        {
          "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
          "size": 977,
          "digest": "sha256:2c930d010525941c1d56ec53b97bd057a67ae1865eebf042686d2a2d18271ced"
        }
      ]
  }
  ```
    
Просмотреть манифест в реестре контейнеров Azure можно с помощью портал Azure или таких средств, как команда [AZ запись в репозитории: Показать-манифесты](/cli/azure/acr/repository#az_acr_repository_show_manifests) в Azure CLI.

### <a name="manifest-list"></a>Список манифестов

*Список манифестов* для образа с несколькими сборками (известно больше как [индекс изображения](https://github.com/opencontainers/image-spec/blob/master/image-index.md) для изображений OCI) — это коллекция (индекс) изображений, которую можно создать, указав одно или несколько имен изображений. Он содержит подробные сведения о каждом из таких образов, как поддерживаемая ОС, архитектура, размер и дайджест манифеста. Список манифестов можно использовать так же, как имя образа в `docker pull` `docker run` командах и. 

`docker`CLI управляет манифестами и списками манифестов с помощью команды [DOCKER manifest](https://docs.docker.com/engine/reference/commandline/manifest/) .

> [!NOTE]
> В настоящее время `docker manifest` команда и подкоманды экспериментальны. Дополнительные сведения об использовании экспериментальных команд см. в документации по DOCKER.

Список манифестов можно просмотреть с помощью `docker manifest inspect` команды. Ниже приведены выходные данные для образа с несколькими сайтами `mcr.microsoft.com/mcr/hello-world:latest` , в котором есть три манифеста: два для архитектуры ОС Linux и одна для архитектуры Windows.
```json
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.docker.distribution.manifest.list.v2+json",
  "manifests": [
    {
      "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
      "size": 524,
      "digest": "sha256:83c7f9c92844bbbb5d0a101b22f7c2a7949e40f8ea90c8b3bc396879d95e899a",
      "platform": {
        "architecture": "amd64",
        "os": "linux"
      }
    },
    {
      "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
      "size": 525,
      "digest": "sha256:873612c5503f3f1674f315c67089dee577d8cc6afc18565e0b4183ae355fb343",
      "platform": {
        "architecture": "arm64",
        "os": "linux"
      }
    },
    {
      "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
      "size": 1124,
      "digest": "sha256:b791ad98d505abb8c9618868fc43c74aa94d08f1d7afe37d19647c0030905cae",
      "platform": {
        "architecture": "amd64",
        "os": "windows",
        "os.version": "10.0.17763.1697"
      }
    }
  ]
}
```

Если список манифестов с несколькими сайтами хранится в реестре контейнеров Azure, то список манифестов можно также просмотреть с помощью портал Azure или с помощью таких средств, как AZ записей в репозитории для записи записей [контроля доступа](/cli/azure/acr/repository#az_acr_repository_how_manifests) .

## <a name="import-a-multi-arch-image"></a>Импорт образа из нескольких ссылок 

Существующий образ с несколькими сайтами можно импортировать в реестр контейнеров Azure с помощью команды [AZ запись контроля](/cli/azure/acr#az_acr_import) доступа. Синтаксис импорта изображения такой же, как и для образа с одной архитектурой. Как и при импорте образа с одной архитектурой, импорт образа с несколькими структурами не использует команды DOCKER. 

Дополнительные сведения см. [в разделе Импорт образов контейнеров в реестр контейнеров](container-registry-import-images.md).

## <a name="push-a-multi-arch-image"></a>Отправка образа с несколькими сайтами

При наличии рабочих процессов сборки для создания образов контейнеров для различных архитектур выполните следующие действия, чтобы отправить образ с несколькими сайтами в реестр контейнеров Azure.

1. Пометьте и отправьте каждый образ, зависящий от архитектуры, в реестр контейнеров. В следующем примере предполагается наличие двух архитектур Linux: arm64 и AMD64. 

   ```console
   docker tag myimage:arm64 \
     myregistry.azurecr.io/multi-arch-samples/myimage:arm64

   docker push myregistry.azurecr.io/multi-arch-samples/myimage:arm64
 
   docker tag myimage:amd64 \
     myregistry.azurecr.io/multi-arch-samples/myimage:amd64

   docker push myregistry.azurecr.io/multi-arch-samples/myimage:amd64
   ``` 

1. Выполните команду `docker manifest create` , чтобы создать список манифестов, чтобы объединить предыдущие изображения в образ с несколькими сайтами.

   ```console
   docker manifest create myregistry.azurecr.io/multi-arch-samples/myimage:multi \
    myregistry.azurecr.io/multi-arch-samples/myimage:arm64 \
    myregistry.azurecr.io/multi-arch-samples/myimage:amd64
   ```

1. Отправьте манифест в реестр контейнеров с помощью `docker manifest push` :

   ```console
   docker manifest push myregistry.azurecr.io/multi-arch-samples/myimage:multi
   ```

1. Используйте `docker manifest inspect` команду, чтобы просмотреть список манифестов. Пример выходных данных команды приведен в предыдущем разделе.

После передачи манифеста с несколькими сайтами в реестр работа с образом с несколькими сайтами будет выполняться так же, как и с одноархитектурным образом. Например, извлекать образ с помощью `docker pull` и использовать команды [AZ контроля доступа к репозиторию](/cli/azure/acr/repository#az_acr_repository) для просмотра тегов, манифестов и других свойств изображения.

## <a name="build-and-push-a-multi-arch-image"></a>Создание и отправка образа с несколькими сборками

Используя функции [задач контроля](container-registry-tasks-overview.md)учетных записей, вы можете собрать и отправить образ с несколькими сайтами в реестр контейнеров Azure. Например, определите [многошаговую задачу](container-registry-tasks-multi-step.md) в файле YAML, который создает образ с несколькими сайтами Linux.

В следующем примере предполагается, что у вас есть отдельные файлы dockerfile для двух архитектур, arm64 и AMD64. Он создает и отправляет образы, относящиеся к архитектуре, а затем создается и отправляет манифест с несколькими структурами, содержащий `latest` тег:

```yml
version: v1.1.0

steps:
- build: -t {{.Run.Registry}}/multi-arch-samples/myimage:{{.Run.ID}}-amd64 -f dockerfile.arm64 . 
- build: -t {{.Run.Registry}}/multi-arch-samples/myyimage:{{.Run.ID}}-arm64 -f dockerfile.amd64 . 
- push: 
    - {{.Run.Registry}}/multi-arch-samples/myimage:{{.Run.ID}}-arm64
    - {{.Run.Registry}}/multi-arch-samples/myimage:{{.Run.ID}}-amd64
- cmd: >
    docker manifest create
    {{.Run.Registry}}/multi-arch-samples/myimage:latest
    {{.Run.Registry}}/multi-arch-samples/myimage:{{.Run.ID}}-arm64
    {{.Run.Registry}}/multi-arch-samples/myimage:{{.Run.ID}}-amd64
- cmd: docker manifest push --purge {{.Run.Registry}}/multi-arch-samples/myimage:latest
- cmd: docker manifest inspect {{.Run.Registry}}/multi-arch-samples/myimage:latest
```

## <a name="next-steps"></a>Дальнейшие действия

* Используйте [Azure pipelines](/azure/devops/pipelines/get-started/what-is-azure-pipelines.md) для создания образов контейнеров для различных архитектур.
* Дополнительные сведения о создании образов с несколькими платформами с помощью экспериментального подключаемого модуля DOCKER [буилдкс](https://docs.docker.com/buildx/working-with-buildx/) .

<!-- LINKS - external -->
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-windows]: https://docs.docker.com/docker-for-windows/
