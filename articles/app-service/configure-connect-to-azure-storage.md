---
title: Добавление службы хранилища Azure (контейнер)
description: Узнайте, как подключить настраиваемую сетевую папку в контейнерном приложении в службе приложений Azure. Совместное использование файлов между приложениями, удаленное управление статическим содержимым и локальный доступ и т. д.
author: msangapu-msft
ms.topic: article
ms.date: 7/01/2019
ms.author: msangapu
zone_pivot_groups: app-service-containers-windows-linux
ms.openlocfilehash: c3e5ca30b7f0f00b6d647f5fdaeb772900996adc
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101720245"
---
# <a name="access-azure-storage-preview-as-a-network-share-from-a-container-in-app-service"></a>Доступ к хранилищу Azure (Предварительная версия) в качестве сетевой папки из контейнера в службе приложений

::: zone pivot="container-windows"

В этом руководство показано, как подключить файлы службы хранилища Azure в качестве сетевой папки к контейнеру Windows в службе приложений. Поддерживаются только общие папки службы [файлов Azure](../storage/files/storage-how-to-use-files-cli.md) и [файлы Premium](../storage/files/storage-how-to-create-file-share.md) . Преимущества включают защищенное содержимое, его переносимость, доступ к нескольким приложениям и несколько методов для передачи.

> [!NOTE]
>Служба хранилища Azure в службе приложений доступна **в режиме предварительной версии** и **не поддерживается** в **рабочих сценариях**.

::: zone-end

::: zone pivot="container-linux"

В этом руководство показано, как подключить службу хранилища Azure к службе приложений контейнеров Linux. К преимуществам относятся защищенное содержимое, переносимость содержимого, постоянное хранилище, доступ к нескольким приложениям и несколько методов передачи.

> [!NOTE]
>Служба хранилища Azure в службе приложений доступна **в предварительной версии** для службы приложений на платформе Linux и веб-приложение для контейнеров. Он **не поддерживается** в **рабочих сценариях**.

::: zone-end

## <a name="prerequisites"></a>Предварительные условия

::: zone pivot="container-windows"

- [Существующее приложение контейнера Windows в службе приложений Azure](quickstart-custom-container.md)
- [Создание файлового ресурса Azure](../storage/files/storage-how-to-use-files-cli.md)
- [Отправка файлов в общую папку Azure](../storage/files/storage-how-to-create-file-share.md)

::: zone-end

::: zone pivot="container-linux"

- Существующая [служба приложений в приложении Linux](index.yml).
- [Учетная запись хранения Azure](../storage/common/storage-account-create.md?tabs=azure-cli) ;
- [Общая папка и каталог Azure](../storage/files/storage-how-to-use-files-cli.md).

::: zone-end

> [!NOTE]
> Служба "файлы Azure" не является хранилищем по умолчанию и оплачивается отдельно, но не входит в состав веб-приложения. Не поддерживает использование конфигурации брандмауэра из-за ограничений инфраструктуры.
>

## <a name="limitations"></a>Ограничения

::: zone pivot="container-windows"

- В настоящее время служба хранилища Azure в службе приложений **не поддерживается** для использования собственных сценариев (неконтейнерные приложения Windows).
- Служба хранилища Azure в службе приложений **не поддерживает** использование конфигурации **брандмауэра хранилища** из-за ограничений инфраструктуры.
- Служба хранилища Azure с помощью службы приложений позволяет указать до **пяти** точек подключения для каждого приложения.
- Служба хранилища Azure, подключенная к приложению, недоступна через конечные точки FTP или FTPs службы приложений. Или воспользуйтесь [обозревателем хранилищ Azure](https://azure.microsoft.com/features/storage-explorer/).

::: zone-end

::: zone pivot="container-linux"

- Служба хранилища Azure в службе приложений поддерживает подключение **контейнеров файлов Azure** (чтение и запись) и **контейнеров больших двоичных объектов Azure** (только для чтения).
- Служба хранилища Azure в службе приложений позволяет указать до **пяти** точек подключения для каждого приложения.
- Служба хранилища Azure, подключенная к приложению, недоступна через конечные точки FTP или FTPs службы приложений. Или воспользуйтесь [обозревателем хранилищ Azure](https://azure.microsoft.com/features/storage-explorer/).

::: zone-end

## <a name="link-storage-to-your-app"></a>Связывание хранилища с приложением

::: zone pivot="container-windows"

После создания [учетной записи хранения Azure, общей папки и каталога](#prerequisites)вы можете настроить приложение в службе хранилища Azure.

Чтобы подключить файловый ресурс Azure к каталогу в приложении службы приложений, используйте [`az webapp config storage-account add`](/cli/azure/webapp/config/storage-account#az-webapp-config-storage-account-add) команду. Тип хранилища должен быть AzureFiles.

```azurecli
az webapp config storage-account add --resource-group <group-name> --name <app-name> --custom-id <custom-id> --storage-type AzureFiles --share-name <share-name> --account-name <storage-account-name> --access-key "<access-key>" --mount-path <mount-path-directory of form c:<directory name> >
```

Это необходимо сделать для всех каталогов, которые необходимо связать с общим ресурсом службы файлов Azure.

::: zone-end

::: zone pivot="container-linux"

После создания [учетной записи хранения Azure, общей папки и каталога](#prerequisites)вы можете настроить приложение в службе хранилища Azure.

Чтобы подключить учетную запись хранения к каталогу в приложении службы приложений, используйте [`az webapp config storage-account add`](/cli/azure/webapp/config/storage-account#az-webapp-config-storage-account-add) команду. Тип хранилища может быть AzureBlob или AzureFiles. В этом примере используется AzureFiles. Параметр пути подключения соответствует папке в контейнере, которую необходимо подключить к службе хранилища Azure. Если задать для него значение "/", весь контейнер будет подключен к службе хранилища Azure.


> [!CAUTION]
> Каталог, указанный в качестве пути подключения в веб-приложении, должен быть пустым. Любое содержимое, хранящееся в этом каталоге, будет удалено при добавлении внешнего подключения. Если в существующем приложении выполняется перенос файлов, перед началом работы создайте резервную копию приложения и его содержимого.
>

```azurecli
az webapp config storage-account add --resource-group <group-name> --name <app-name> --custom-id <custom-id> --storage-type AzureFiles --share-name <share-name> --account-name <storage-account-name> --access-key "<access-key>" --mount-path <mount-path-directory>
```

Для всех остальных каталогов, которые вы хотите связать с учетной записью хранения, следует сделать то же самое.

::: zone-end

## <a name="verify-linked-storage"></a>Проверка связанного хранилища

После того как общая папка будет связана с приложением, вы можете проверить ее, выполнив следующую команду:

```azurecli
az webapp config storage-account list --resource-group <resource-group> --name <app-name>
```

## <a name="next-steps"></a>Дальнейшие действия

::: zone pivot="container-windows"

- [Перенос пользовательского программного обеспечения в службу приложений Azure с помощью пользовательского контейнера](tutorial-custom-container.md?pivots=container-windows).

::: zone-end

::: zone pivot="container-linux"

- [Настройка пользовательского контейнера](configure-custom-container.md?pivots=platform-linux).

::: zone-end