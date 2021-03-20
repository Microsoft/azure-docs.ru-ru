---
title: Подключить том gitRepo к группе контейнеров
description: Сведения о подключении тома gitRepo для клонирования репозитория в экземпляры контейнеров
ms.topic: article
ms.date: 06/15/2018
ms.openlocfilehash: 5f361b0bbd4a02f754e47c8321a664af05036601
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86260720"
---
# <a name="mount-a-gitrepo-volume-in-azure-container-instances"></a>Подключение тома gitRepo в службе "Экземпляры контейнеров Azure"

Узнайте, как подключить том *gitRepo*, чтобы клонировать репозиторий в экземпляры контейнеров.

> [!NOTE]
> Подключение тома *gitRepo* сейчас поддерживается только для контейнеров Linux. Пока мы работаем над переносом всех компонентов в контейнеры Windows, в [обзоре](container-instances-overview.md#linux-and-windows-containers)можно найти различия в текущих платформах.

## <a name="gitrepo-volume"></a>Том gitRepo

Том *gitRepo* подключает каталог и клонирует в него указанный репозиторий Git при запуске контейнера. Используя том *gitRepo* в экземплярах контейнера, можно избежать добавления кода для выполнения этой задачи в приложениях.

После подключения тома *gitRepo* задайте три свойства, чтобы настроить том.

| Свойство | Обязательно | Описание |
| -------- | -------- | ----------- |
| `repository` | Да | Полный URL-адрес (включая `http://` или `https://`) репозитория Git, который будет клонирован.|
| `directory` | Нет | Каталог, в который необходимо клонировать репозиторий. Значение пути не должно содержать знак `..` или начинаться с него.  Если указан знак `.`, репозиторий клонируется в каталог тома. В противном случае репозиторий Git клонируется в подкаталог с указанным именем в каталоге томов. |
| `revision` | Нет | Зафиксированный хэш версии, которая будет клонирована. Если не указан, клонируется версия `HEAD`. |

## <a name="mount-gitrepo-volume-azure-cli"></a>Подключение тома gitRepo: Azure CLI

Чтобы подключить том gitRepo при развертывании экземпляров контейнера с помощью [Azure CLI](/cli/azure), укажите параметры `--gitrepo-url` и `--gitrepo-mount-path` в команде [az container create][az-container-create]. При необходимости можно указать каталог тома, в который будет осуществляться клонирование (`--gitrepo-dir`) и хэш фиксации редакции, которая будет клонирована (`--gitrepo-revision`).

В этом примере команда копирует пример приложения Microsoft [ACI-HelloWorld][aci-helloworld] в `/mnt/aci-helloworld` в экземпляре контейнера:

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \
    --name hellogitrepo \
    --image mcr.microsoft.com/azuredocs/aci-helloworld \
    --dns-name-label aci-demo \
    --ports 80 \
    --gitrepo-url https://github.com/Azure-Samples/aci-helloworld \
    --gitrepo-mount-path /mnt/aci-helloworld
```

Чтобы убедиться, что том gitRepo подключен, запустите оболочку в контейнере с помощью команды [az container exec][az-container-exec] и выведите список каталогов.

```azurecli
az container exec --resource-group myResourceGroup --name hellogitrepo --exec-command /bin/sh
```

```output
/usr/src/app # ls -l /mnt/aci-helloworld/
total 16
-rw-r--r--    1 root     root           144 Apr 16 16:35 Dockerfile
-rw-r--r--    1 root     root          1162 Apr 16 16:35 LICENSE
-rw-r--r--    1 root     root          1237 Apr 16 16:35 README.md
drwxr-xr-x    2 root     root          4096 Apr 16 16:35 app
```

## <a name="mount-gitrepo-volume-resource-manager"></a>Подключение тома gitRepo: Resource Manager

Чтобы подключить том gitRepo при развертывании экземпляров контейнера с помощью [шаблона Azure Resource Manager](/azure/templates/microsoft.containerinstance/containergroups), в шаблоне сначала нужно заполнить массив `volumes` в разделе `properties` группы контейнеров. Затем для каждого контейнера в группе контейнеров, в которой нужно подключить том *gitRepo*, нужно заполнить массив `volumeMounts` в разделе `properties` определения контейнера.

Например, следующий шаблон Resource Manager создает группу контейнеров, состоящую из одного контейнера. Контейнер клонирует два репозитория GitHub, указанных блоками тома *gitRepo*. Второй том содержит дополнительные свойства, указывающие каталог для клонирования и зафиксированный хэш определенной версии.

<!-- https://github.com/Azure/azure-docs-json-samples/blob/master/container-instances/aci-deploy-volume-gitrepo.json -->
[!code-json[volume-gitrepo](~/azure-docs-json-samples/container-instances/aci-deploy-volume-gitrepo.json)]

В результате структура каталогов двух клонированных репозиториев, определенных в предыдущем шаблоне, выглядит следующим образом.

```
/mnt/repo1/aci-helloworld
/mnt/repo2/my-custom-clone-directory
```

Пример развертывания экземпляра контейнера с помощью шаблона Azure Resource Manager см. в статье [Развертывание группы контейнеров](container-instances-multi-container-group.md).

## <a name="private-git-repo-authentication"></a>Проверка подлинности в частном репозитории Git

Чтобы подключить том gitRepo как частный репозиторий Git, укажите учетные данные в URL-адресе репозитория. Как правило, учетные данные представлены в виде имени пользователя и личного маркера доступа (PAT), который предоставляет доступ к репозиторию.

Например, параметр Azure CLI `--gitrepo-url` для частного репозитория GitHub будет похож на следующий (где "gituser" — это имя пользователя GitHub, а "abcdef1234fdsa4321abcdef" — ​​это личный маркер доступа пользователя):

```console
--gitrepo-url https://gituser:abcdef1234fdsa4321abcdef@github.com/GitUser/some-private-repository
```

Для репозитория Git Azure Repos укажите любое имя пользователя (вы можете использовать "azurereposuser", как в следующем примере) в сочетании с действительным PAT.

```console
--gitrepo-url https://azurereposuser:abcdef1234fdsa4321abcdef@dev.azure.com/your-org/_git/some-private-repository
```

Дополнительные сведения о личных маркерах доступа для GitHub и Azure Repos:

GitHub: [создание личных маркеров доступа для командной строки][pat-github]

Azure Repos: [создание личных маркеров доступа для аутентификации доступа][pat-repos]

## <a name="next-steps"></a>Дальнейшие действия

Сведения о подключении других типов томов в службе "Экземпляры контейнеров Azure" см. в следующих статьях:

* [Mount an Azure file share in Azure Container Instances](container-instances-volume-azure-files.md) (Подключение файлового ресурса Azure с помощью службы "Экземпляры контейнеров Azure")
* [Подключение тома emptyDir в службе "Экземпляры контейнеров Azure"](container-instances-volume-emptydir.md)
* [Подключение тома secret в службе "Экземпляры контейнеров Azure"](container-instances-volume-secret.md)

<!-- LINKS - External -->
[aci-helloworld]: https://github.com/Azure-Samples/aci-helloworld
[pat-github]: https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/
[pat-repos]: /azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate

<!-- LINKS - Internal -->
[az-container-create]: /cli/azure/container#az-container-create
[az-container-exec]: /cli/azure/container#az-container-exec
