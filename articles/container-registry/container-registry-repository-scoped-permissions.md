---
title: Разрешения на репозитории в Реестре контейнеров Azure
description: Создание маркера с разрешениями, ограниченными конкретными репозиториями в реестре уровня "Премиум" для извлечения или отправки изображений или выполнения других действий.
ms.topic: article
ms.date: 02/04/2021
ms.openlocfilehash: ceec69d746f77ea7a23bc70d029c8b3736e7f292
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99988265"
---
# <a name="create-a-token-with-repository-scoped-permissions"></a>Создание токена с разрешениями уровня репозитория

В этой статье описывается создание маркеров и сопоставлений областей для управления доступом к конкретным репозиториям в реестре контейнеров. Создавая токены, владелец реестра может предоставлять пользователям или службам ограниченный по времени и области доступ к репозиториям, чтобы получать или отправлять образы или выполнять другие действия. Токен предоставляет более детализированные разрешения, чем другие [варианты проверки подлинности](container-registry-authentication.md) в реестре, в которых областью действия разрешений является весь реестр. 

Ниже приведены сценарии создания токена.

* Разрешение устройствам Интернета вещей с отдельными токенами получить образ из репозитория.
* Предоставление внешней организации разрешений для определенного репозитория. 
* Ограничение доступа к репозиторию для разных групп пользователей в организации. Например, вы можете предоставить доступ на запись и чтение тем разработчикам, которые создают образы, предназначенные для конкретных репозиториев, и доступ на чтение тем командам, которые выполняют развертывание из этих репозиториев.

Эта возможность доступна на уровне **Премиум** службы реестра контейнеров. Ознакомиться с информацией об уровнях служб реестра и ограничениях можно в статье [Уровни служб Реестра контейнеров Azure](container-registry-skus.md).

> [!IMPORTANT]
> Сейчас эта функция доступна в предварительной версии, и применяются [некоторые ограничения](#preview-limitations). Предварительные версии предоставляются при условии, что вы принимаете [дополнительные условия использования][terms-of-use]. Некоторые аспекты этой функции могут быть изменены до выхода общедоступной версии.

## <a name="preview-limitations"></a>Ограничения предварительной версии

* Сейчас вы не можете назначить разрешения уровня репозитория для удостоверения Azure Active Directory, например субъекта-службы или управляемого удостоверения.
* Вы не можете создать карту области в реестре, включенном для [анонимного доступа на извлечение](container-registry-faq.md#how-do-i-enable-anonymous-pull-access).

## <a name="concepts"></a>Основные понятия

Чтобы настроить разрешения на уровне репозитория, создайте *токен* со связанной *картой области*. 

* **Токен** вместе с созданным паролем позволяет пользователю пройти проверку подлинности в реестре. Вы можете задать дату истечения срока действия пароля токена или отключить токен в любое время.  

  После проверки подлинности с помощью токена пользователь или служба могут выполнить одно *действие* или несколько в рамках области, охватывающей один репозиторий или несколько.

  |Действие  |Описание  | Пример |
  |---------|---------|--------|
  |`content/delete`    | Удаление данных из репозитория  | Удаление репозитория или манифеста |
  |`content/read`     |  Чтение данных из репозитория |  Извлечение артефакта |
  |`content/write`     |  Запись данных в репозиторий     | Использование с `content/read` для отправки артефакта |
  |`metadata/read`    | Чтение метаданных из репозитория   | Перечисление тегов или манифестов |
  |`metadata/write`     |  Запись метаданных в репозиторий  | Включение или отключение операций чтения, записи или удаления |

* Разрешения для репозитория групп **карт области**, которые применяются к токену или могут повторно применяться к другим токенам. Каждый токен связан с одной картой области. 

   Карта области предоставляет следующие возможности.

    * Настройка нескольких токенов с одинаковыми разрешениями для набора репозиториев
    * Обновление разрешений токена при добавлении или удалении действий репозитория в карте области или применении другой карты области 

  Реестр контейнеров Azure также предоставляет несколько определенных системой схем областей, которые можно применять при создании маркеров. Разрешения схем области, определяемые системой, применяются ко всем репозиториям в реестре.

На следующем рисунке показана связь между токенами и картами области. 

![Токены реестра и карты области](media/container-registry-repository-scoped-permissions/token-scope-map-concepts.png)

## <a name="prerequisites"></a>Предварительные требования

* В примерах команд **Azure CLI** -Azure CLI в этой статье требуется Azure CLI версии 2.17.0 или более поздней. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0](/cli/azure/install-azure-cli).
* **Docker** — чтобы проверить подлинность в реестре для извлечения или отправки образов, требуется локальная установка Docker. На сайте Docker предоставляются инструкции по установке для систем [macOS](https://docs.docker.com/docker-for-mac/), [Windows](https://docs.docker.com/docker-for-windows/) и [Linux](https://docs.docker.com/engine/installation/#supported-platforms).
* **Реестр контейнеров** — если у вас его нет, создайте реестр контейнеров уровня "Премиум" в подписке Azure или обновите существующий реестр. Это можно сделать на [портале Azure](container-registry-get-started-portal.md) или с помощью [Azure CLI](container-registry-get-started-azure-cli.md). 

## <a name="create-token---cli"></a>Создание токена — интерфейс командной строки

### <a name="create-token-and-specify-repositories"></a>Создание токена и указание репозиториев

Создайте токен с помощью команды [az acr token create][az-acr-token-create]. При создании токена можно указать один репозиторий или несколько и связанные действия в каждом из них. Репозитории не обязательно должны находиться в реестре. Сведения о создании токена путем указания существующей схемы области см. в [следующем разделе](#create-token-and-specify-scope-map).

Следующий пример создает токен в реестре *myregistry* со следующими разрешениями для репозитория `samples/hello-world`: `content/write` и `content/read`. По умолчанию эта команда задает для состояния токена по умолчанию значение `enabled`, но вы можете в любое время изменить состояние на `disabled`.

```azurecli
az acr token create --name MyToken --registry myregistry \
  --repository samples/hello-world \
  content/write content/read
```

В выходных данных отображаются сведения о токене. По умолчанию создаются два пароля, срок действия которых не истекает, но при необходимости можно задать дату окончания срока действия. Рекомендуется сохранить эти пароли в надежном месте для последующего использования при проверке подлинности. Повторно получить эти пароли невозможно, но можно создать новые.

```console
{
  "creationDate": "2020-01-18T00:15:34.066221+00:00",
  "credentials": {
    "certificates": [],
    "passwords": [
      {
        "creationTime": "2020-01-18T00:15:52.837651+00:00",
        "expiry": null,
        "name": "password1",
        "value": "uH54BxxxxK7KOxxxxRbr26dAs8JXxxxx"
      },
      {
        "creationTime": "2020-01-18T00:15:52.837651+00:00",
        "expiry": null,
        "name": "password2",
        "value": "kPX6Or/xxxxLXpqowxxxxkA0idwLtmxxxx"
      }
    ],
    "username": "MyToken"
  },
  "id": "/subscriptions/xxxxxxxx-adbd-4cb4-c864-xxxxxxxxxxxx/resourceGroups/myresourcegroup/providers/Microsoft.ContainerRegistry/registries/myregistry/tokens/MyToken",
  "name": "MyToken",
  "objectId": null,
  "provisioningState": "Succeeded",
  "resourceGroup": "myresourcegroup",
  "scopeMapId": "/subscriptions/xxxxxxxx-adbd-4cb4-c864-xxxxxxxxxxxx/resourceGroups/myresourcegroup/providers/Microsoft.ContainerRegistry/registries/myregistry/scopeMaps/MyToken-scope-map",
  "status": "enabled",
  "type": "Microsoft.ContainerRegistry/registries/tokens"
```

> [!NOTE]
> Дополнительные сведения о повторном создании паролей токенов и сроках действия см. в разделе [Повторное создание паролей токенов](#regenerate-token-passwords) далее в этой статье.

Выходные данные содержат сведения о карте области, созданной командой. Карту области, которая здесь называется `MyToken-scope-map`, можно использовать, чтобы применить те же действия репозитория к другим токенам. Или можно обновить карту области позже, чтобы изменить разрешения для связанных токенов.

### <a name="create-token-and-specify-scope-map"></a>Создание токена и указание карты области

Альтернативный способ создания токена заключается в указании существующей карты области. Если у вас еще нет карты области, сначала создайте ее, указав репозитории и связанные действия. Затем укажите эту карту области при создании токена. 

Чтобы создать карту области, используйте команду [az acr scope-map create][az-acr-scope-map-create]. Следующая команда создает карту области с теми же разрешениями в репозитории `samples/hello-world`, который использовался ранее. 

```azurecli
az acr scope-map create --name MyScopeMap --registry myregistry \
  --repository samples/hello-world \
  content/write content/read \
  --description "Sample scope map"
```

Выполните команду [az acr token create][az-acr-token-create], чтобы создать токен, указав карту области *MyScopeMap*. Как и в предыдущем примере, команда устанавливает для состояния токена по умолчанию значение `enabled`.

```azurecli
az acr token create --name MyToken \
  --registry myregistry \
  --scope-map MyScopeMap
```

В выходных данных отображаются сведения о токене. По умолчанию создаются два пароля. Рекомендуется сохранить эти пароли в надежном месте для последующего использования при проверке подлинности. Повторно получить эти пароли невозможно, но можно создать новые.

> [!NOTE]
> Дополнительные сведения о повторном создании паролей токенов и сроках действия см. в разделе [Повторное создание паролей токенов](#regenerate-token-passwords) далее в этой статье.

## <a name="create-token---portal"></a>Создание токена — портал

Для создания токенов и карт областей можно использовать портал Azure. Как и в случае с командой `az acr token create` интерфейса командной строки, можно применить существующую карту области или создать карту области при создании токена, указав один репозиторий или несколько и связанные действия. Репозитории не обязательно должны находиться в реестре. 

Следующий пример создает токен и карту области со следующими разрешениями для репозитория `samples/hello-world`: `content/write` и `content/read`.

1. На портале перейдите к нужному реестру контейнеров.
1. В разделе **разрешения репозитория** выберите **токены (Предварительная версия) > + добавить**.

      :::image type="content" source="media/container-registry-repository-scoped-permissions/portal-token-add.png" alt-text="Создание токена на портале":::
1. Введите имя токена.
1. В разделе **Карта области** выберите **Создать**.
1. Настройте карту области.
    1. Введите имя и описание для карты области. 
    1. В разделе **Репозитории** введите `samples/hello-world` и в разделе **Разрешения** выберите `content/read` и `content/write`. Затем выберите **+ Добавить**.  

        :::image type="content" source="media/container-registry-repository-scoped-permissions/portal-scope-map-add.png" alt-text="Создание карты области на портале":::

    1. После добавления репозиториев и разрешений выберите **Добавить**, чтобы добавить карту области.
1. Примите **Состояние** токена по умолчанию **Включено**, а затем выберите **Создать**.

После проверки и создания токена сведения о нем отображаются на экране **Токены**.

### <a name="add-token-password"></a>Добавление пароля токена

Чтобы использовать токен, созданный на портале, необходимо создать пароль. Можно создать один или два пароля и задать дату окончания срока действия для каждого из них. 

1. На портале перейдите к нужному реестру контейнеров.
1. В разделе **разрешения репозитория** выберите **токены (Предварительная версия)** и выберите маркер.
1. В сведениях о токене выберите **password1** или **password2** и щелкните значок "Создать".
1. На экране пароля при необходимости задайте дату окончания срока действия пароля и выберите **Создать**. Рекомендуется задать дату окончания срока действия.
1. После создания пароля скопируйте и сохраните его в надежном месте. После закрытия экрана вы не сможете повторно получить созданный пароль, но сможете создать новый.

    :::image type="content" source="media/container-registry-repository-scoped-permissions/portal-token-password.png" alt-text="Создание пароля токена на портале":::

## <a name="authenticate-with-token"></a>Проверка подлинности с помощью токена

Когда пользователь или служба используют токен для проверки подлинности в целевом реестре, они предоставляют имя пользователя и один из созданных паролей. 

Метод проверки подлинности зависит от настроенных действий, связанных с токеном.

|Действие  |Процедура проверки подлинности  |
  |---------|---------|
  |`content/delete`    | `az acr repository delete` в Azure CLI<br/><br/>Например, `az acr repository delete --name myregistry --repository myrepo --username MyToken --password xxxxxxxxxx`.|
  |`content/read`     |  `docker login`<br/><br/>`az acr login` в Azure CLI<br/><br/>Например, `az acr login --name myregistry --username MyToken --password xxxxxxxxxx`.  |
  |`content/write`     |  `docker login`<br/><br/>`az acr login` в Azure CLI     |
  |`metadata/read`    | `az acr repository show`<br/><br/>`az acr repository show-tags`<br/><br/>`az acr repository show-manifests` в Azure CLI   |
  |`metadata/write`     |  `az acr repository untag`<br/><br/>`az acr repository update` в Azure CLI |

## <a name="examples-use-token"></a>Примеры: использование токена

В следующих примерах токен, созданный ранее в этой статье, используется для выполнения общих операций с репозиторием: отправка и извлечение образов, удаление образов и перечисление тегов репозитория. Первоначально токен был настроен с разрешениями на передачу (действия `content/write` и `content/read`) для репозитория `samples/hello-world`.

### <a name="pull-and-tag-test-images"></a>Извлечение тестовых образов и пометка их тегами

В следующих примерах вы запрашиваете общедоступные `hello-world` `nginx` образы из реестра контейнеров Майкрософт и размечать их для реестра и репозитория.

```bash
docker pull mcr.microsoft.com/hello-world
docker pull mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
docker tag mcr.microsoft.com/hello-world myregistry.azurecr.io/samples/hello-world:v1
docker tag mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine myregistry.azurecr.io/samples/nginx:v1
```

### <a name="authenticate-using-token"></a>Проверка подлинности с помощью токена

Запустите `docker login` или `az acr login` для проверки подлинности в реестре для отправки или извлечения изображений. Укажите имя маркера в качестве имени пользователя и укажите один из его паролей. Токен должен иметь состояние `Enabled`.

Следующий пример приведен в формате для оболочки Bash и предоставляет значения с помощью переменных среды.

```bash
TOKEN_NAME=MyToken
TOKEN_PWD=<token password>

echo $TOKEN_PWD | docker login --username $TOKEN_NAME --password-stdin myregistry.azurecr.io
```

Выходные данные должны указывать на успешную проверку подлинности.

```console
Login Succeeded
```

### <a name="push-images-to-registry"></a>Отправка образов в реестр

После успешного входа попытайтесь отправить помеченные тегами образы в реестр. Так как у токена есть разрешения на отправку образов в репозиторий `samples/hello-world`, следующая операция отправки выполняется успешно.

```bash
docker push myregistry.azurecr.io/samples/hello-world:v1
```

У этого токена нет разрешений для репозитория `samples/nginx`, поэтому следующая попытка отправки завершается ошибкой, аналогичной `requested access to the resource is denied`.

```bash
docker push myregistry.azurecr.io/samples/nginx:v1
```

### <a name="update-token-permissions"></a>Обновить разрешения маркера

Чтобы изменить разрешения токена, обновите разрешения в соответствующей карте области. Обновленная карта области применяется сразу для всех связанных токенов. 

Например, обновите `MyToken-scope-map` с помощью действий `content/write` и `content/read` для репозитория `samples/ngnx` и удалите действие `content/write` для репозитория `samples/hello-world`.  

Чтобы использовать Azure CLI, выполните команду [az acr scope-map update][az-acr-scope-map-update] для обновления карты области.

```azurecli
az acr scope-map update \
  --name MyScopeMap \
  --registry myregistry \
  --add-repository samples/nginx content/write content/read \
  --remove-repository samples/hello-world content/write 
```

На портале Azure выполните следующие действия.

1. Перейдите к реестру контейнеров.
1. В разделе **разрешения репозитория** выберите карты **области (Предварительная версия)** и выберите карту области для обновления.
1. В разделе **Репозитории** введите `samples/nginx` и в разделе **Разрешения** выберите `content/read` и `content/write`. Затем выберите **+ Добавить**.
1. В разделе **Репозитории** выберите `samples/hello-world` и в разделе **Разрешения** отмените выбор `content/write`. Затем нажмите кнопку **Save** (Сохранить).

После обновления карты области следующая операция отправки выполняется успешно.

```bash
docker push myregistry.azurecr.io/samples/nginx:v1
```

Так как карта области имеет разрешение `content/read` только для репозитория `samples/hello-world`, попытка отправки в репозиторий `samples/hello-world` теперь завершается ошибкой.
 
```bash
docker push myregistry.azurecr.io/samples/hello-world:v1
```

Извлечение образов из обоих репозиториев выполняется успешно, так как карта области предоставляет разрешения `content/read` для обоих репозиториев.

```bash
docker pull myregistry.azurecr.io/samples/nginx:v1
docker pull myregistry.azurecr.io/samples/hello-world:v1
```
### <a name="delete-images"></a>Удаление изображений

Обновите карту области, добавив действие `content/delete` в репозиторий `nginx`. Это действие позволяет удалять образы в репозитории или удалить весь репозиторий.

Для краткости мы приводим только команду [az acr scope-map update][az-acr-scope-map-update] для обновления карты области.

```azurecli
az acr scope-map update \
  --name MyScopeMap \
  --registry myregistry \
  --add-repository samples/nginx content/delete
``` 

Чтобы обновить карту области с помощью портала, см. [предыдущий раздел](#update-token-permissions).

Чтобы удалить репозиторий `samples/nginx`, используйте следующую команду [az acr repository delete][az-acr-repository-delete]. Чтобы удалить образы или репозитории, передайте имя и пароль маркера в команду. В следующем примере используются переменные среды, созданные ранее в этой статье.

```azurecli
az acr repository delete \
  --name myregistry --repository samples/nginx \
  --username $TOKEN_NAME --password $TOKEN_PWD
```

### <a name="show-repo-tags"></a>Отображение тегов репозитория 

Обновите карту области, добавив действие `metadata/read` в репозиторий `hello-world`. Это действие позволяет считывать данные манифестов и тегов в репозитории.

Для краткости мы приводим только команду [az acr scope-map update][az-acr-scope-map-update] для обновления карты области.

```azurecli
az acr scope-map update \
  --name MyScopeMap \
  --registry myregistry \
  --add-repository samples/hello-world metadata/read 
```  

Чтобы обновить карту области с помощью портала, см. [предыдущий раздел](#update-token-permissions).

Чтобы считать метаданные в репозитории `samples/hello-world`, выполните команду [az acr repository show-manifests][az-acr-repository-show-manifests] или [az acr repository show-tags][az-acr-repository-show-tags]. 

Для чтения метаданных передайте имя и пароль маркера в любую команду. В следующем примере используются переменные среды, созданные ранее в этой статье.

```azurecli
az acr repository show-tags \
  --name myregistry --repository samples/hello-world \
  --username $TOKEN_NAME --password $TOKEN_PWD
```

Образец вывода:

```console
[
  "v1"
]
```

## <a name="manage-tokens-and-scope-maps"></a>Управление токенами и картами области

### <a name="list-scope-maps"></a>Список карт области

Для перечисления всех карт области, настроенных в реестре, воспользуйтесь командой [az acr scope-map list][az-acr-scope-map-list] или экраном **Карты области (предварительная версия)** на портале. Пример:

```azurecli
az acr scope-map list \
  --registry myregistry --output table
```

Выходные данные состоят из трех определяемых системой сопоставлений областей и других схем области, созданных вами. Маркеры можно настроить с помощью любой из этих карт области.

```
NAME                 TYPE           CREATION DATE         DESCRIPTION
-------------------  -------------  --------------------  ------------------------------------------------------------
_repositories_admin  SystemDefined  2020-01-20T09:44:24Z  Can perform all read, write and delete operations on the ...
_repositories_pull   SystemDefined  2020-01-20T09:44:24Z  Can pull any repository of the registry
_repositories_push   SystemDefined  2020-01-20T09:44:24Z  Can push to any repository of the registry
MyScopeMap           UserDefined    2019-11-15T21:17:34Z  Sample scope map
```

### <a name="show-token-details"></a>Отображение сведений о токене

Чтобы просмотреть сведения о токене, такие как его состояние и даты истечения срока действия пароля, выполните команду [az acr token show][az-acr-token-show] или выберите токен на экране **Токены (предварительная версия)** портала. Пример:

```azurecli
az acr scope-map show \
  --name MyScopeMap --registry myregistry
```

Для перечисления всех токенов, настроенных в реестре, воспользуйтесь командой [az acr token list][az-acr-token-list] или экраном **Токены (предварительная версия)** на портале. Пример:

```azurecli
az acr token list --registry myregistry --output table
```

### <a name="regenerate-token-passwords"></a>Повторное создание паролей токенов

Если вы не создавали пароль маркера или хотите создать новые пароли, выполните команду AZ запись [контроля учетных данных токена][az-acr-token-credential-generate] . 

Следующий пример создает новое значение для password1 для токена *MyToken* с периодом действия 30 дней. Пароль сохраняется в переменной среды `TOKEN_PWD`. Этот пример приведен в формате для оболочки Bash.

```azurecli
TOKEN_PWD=$(az acr token credential generate \
  --name MyToken --registry myregistry --expiration-in-days 30 \
  --password1 --query 'passwords[0].value' --output tsv)
```

Чтобы использовать портал Azure для создания пароля токена, см. действия, описанные в разделе [Создание токена — портал](#create-token---portal) выше.

### <a name="update-token-with-new-scope-map"></a>Обновление токена с использованием новой карты области

Если вы хотите обновить токен с использованием другой карты области, выполните команду [az acr token update][az-acr-token-update] и укажите новую карту области. Пример:

```azurecli
az acr token update --name MyToken --registry myregistry \
  --scope-map MyNewScopeMap
```

На экране **Токены (предварительная версия)** портала выберите токен и в разделе **Карта области** выберите другую карту области.

> [!TIP]
> После обновления токена с использованием новой карты области может потребоваться создать новые пароли токена. Используйте команду [az acr token credential generate][az-acr-token-credential-generate] или повторно создайте пароль токена на портале Azure.

## <a name="disable-or-delete-token"></a>Отключение или удаление токена

Вам может потребоваться временно отключить использование учетных данных токена для пользователя или службы. 

С помощью Azure CLI выполните команду [az acr token update][az-acr-token-update], чтобы задать для `status` значение `disabled`.

```azurecli
az acr token update --name MyToken --registry myregistry \
  --status disabled
```

На экране **Токены (предварительная версия)** портала выберите токен и выберите значение **Отключено** в поле **Состояние**.

Чтобы удалить токен и окончательно сделать невозможным доступ для всех пользователей, использующих его учетные данные, выполните команду [az acr token delete][az-acr-token-delete]. 

```azurecli
az acr token delete --name MyToken --registry myregistry
```

На экране **Токены (предварительная версия)** портала выберите токен и затем **Отменить**.

## <a name="next-steps"></a>Дальнейшие действия

* Чтобы управлять картами области и токенами, используйте дополнительные команды в группах команд [az acr scope-map][az-acr-scope-map] и [az acr token][az-acr-token].
* Сведения о других вариантах проверки подлинности в Реестре контейнеров Azure, в том числе с использованием удостоверения Azure Active Directory, субъекта-службы или учетной записи администратора, см. в [обзоре проверки подлинности](container-registry-authentication.md).


<!-- LINKS - External -->
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

<!-- LINKS - Internal -->
[az-acr-login]: /cli/azure/acr#az-acr-login
[az-acr-repository]: /cli/azure/acr/repository/
[az-acr-repository-show-tags]: /cli/azure/acr/repository/#az-acr-repository-show-tags
[az-acr-repository-show-manifests]: /cli/azure/acr/repository/#az-acr-repository-show-manifests
[az-acr-repository-delete]: /cli/azure/acr/repository/#az-acr-repository-delete
[az-acr-scope-map]: /cli/azure/acr/scope-map/
[az-acr-scope-map-create]: /cli/azure/acr/scope-map/#az-acr-scope-map-create
[az-acr-scope-map-list]: /cli/azure/acr/scope-map/#az-acr-scope-map-show
[az-acr-scope-map-show]: /cli/azure/acr/scope-map/#az-acr-scope-map-list
[az-acr-scope-map-update]: /cli/azure/acr/scope-map/#az-acr-scope-map-update
[az-acr-scope-map-list]: /cli/azure/acr/scope-map/#az-acr-scope-map-list
[az-acr-token]: /cli/azure/acr/token/
[az-acr-token-show]: /cli/azure/acr/token/#az-acr-token-show
[az-acr-token-list]: /cli/azure/acr/token/#az-acr-token-list
[az-acr-token-delete]: /cli/azure/acr/token/#az-acr-token-delete
[az-acr-token-create]: /cli/azure/acr/token/#az-acr-token-create
[az-acr-token-update]: /cli/azure/acr/token/#az-acr-token-update
[az-acr-token-credential-generate]: /cli/azure/acr/token/credential/#az-acr-token-credential-generate
