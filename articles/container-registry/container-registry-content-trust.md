---
title: Управление подписанными образами
description: Узнайте, как включить доверие к содержимому в Реестре контейнеров Azure, а также отправлять и извлекать подписанные образы. Отношение доверия содержимого реализует отношение доверия DOCKER и является функцией уровня служб Premium.
ms.topic: article
ms.date: 09/18/2020
ms.openlocfilehash: f44cea09521dc235ad0d555264b165c9a3842a14
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92148580"
---
# <a name="content-trust-in-azure-container-registry"></a>Доверие к содержимому в Реестре контейнеров Azure

Реестр контейнеров Azure реализует модель [доверия к содержимому][docker-content-trust] Docker, обеспечивая отправку и извлечение подписанных образов. Эта статья поможет приступить к реализации модели доверия к содержимому в ваших реестрах контейнеров.

> [!NOTE]
> Доверие к содержимому — это функция [уровня служб "Премиум"](container-registry-skus.md) в службе "Реестр контейнеров Azure".

## <a name="how-content-trust-works"></a>Как работает функция доверия к содержимому

Для любой распределенной системы, созданной с учетом требований безопасности, важно проверить как *источник*, так и *целостность* данных, поступающих в эту систему. Получателям данных нужна возможность проверить издателя (источник) данных, а также убедиться в том, что они не были изменены после публикации (целостность). 

Функция доверия к содержимому позволяет издателю **подписывать** образы, которые отправляются в реестр. Получатели образов (люди или системы, извлекающие эти образы из реестра) могут настроить свои клиенты для извлечения *только* подписанных образов. Когда получатель извлекает подписанный образ, Docker-клиент проверяет его целостность. Таким образом получателям гарантируется, что подписанные образы в вашем реестре были опубликованы именно вами и не были изменены после публикации.

### <a name="trusted-images"></a>Доверенные образы

Функция доверия к содержимому поддерживает **теги** образов в репозитории. В таких репозиториях могут находиться как подписанные, так и неподписанные теги. Например, вы подписали образы `myimage:stable` и `myimage:latest`, но не подписали `myimage:dev`.

### <a name="signing-keys"></a>Ключи подписывания

Управление функцией доверия к содержимому осуществляется с помощью набора ключей подписывания, которые связаны с конкретным репозиторием в реестре. Есть несколько типов таких ключей, с помощью которых Docker-клиенты и ваш реестр управляют доверием к тегам в репозитории. Если функция доверия к содержимому включена и интегрирована в конвейер публикации и получения в контейнере, следует управлять этими ключами с осмотрительностью. Подробные сведения см. в разделе [Управление ключами](#key-management) этой статьи, а также на странице [Manage keys for content trust][docker-manage-keys] (Управление ключами функции доверия к содержимому) документации Docker.

> [!TIP]
> Здесь представлен общий обзор модели доверия к содержимому в Docker. Более подробное описание см. в статье [Content trust in Docker][docker-content-trust] (Функция доверия к содержимому в Docker).

## <a name="enable-registry-content-trust"></a>Включение доверия к содержимому в реестре

Сначала необходимо включить функцию доверия к содержимому на уровне реестра. После этого клиенты (пользователи или службы) смогут отправлять подписанные образы в ваш реестр. Включение доверия не ограничивает использование реестра только получателями с этой функцией. Получатели без нее используют ваш реестр как обычно. Но, если у них эта функция включена, для них отображаются *только* подписанные образы в вашем реестре.

Сначала перейдите в реестр на портале Azure. В разделе **Политики** последовательно выберите **Доверие к содержимому** > **Включено** > **Сохранить**. Можно также выполнить команду [az acr config content-trust update][az-acr-config-content-trust-update] в Azure CLI.

![Снимок экрана, показывающий включение отношения доверия содержимого для реестра в портал Azure.][content-trust-01-portal]

## <a name="enable-client-content-trust"></a>Включение доверия к содержимому в клиенте

Чтобы работать с доверенными образами, функция доверия к содержимому в Docker-клиенте должна быть включена и у издателей, и у получателей. Издатель может подписывать образы, отправляемые в реестр с функцией доверия. А получатель, включая функцию доверия, ограничивает отображение образов в реестре только подписанными. В Docker-клиенте эта функция по умолчанию отключена, но ее можно включить для сеанса оболочки или для команды.

Чтобы включить доверие для сеанса оболочки, установите для переменной среды `DOCKER_CONTENT_TRUST` значение **1**. Пример для оболочки bash:

```bash
# Enable content trust for shell session
export DOCKER_CONTENT_TRUST=1
```

Если нужно включить или отключить доверие для одной команды, аргумент `--disable-content-trust` поддерживается несколькими командами Docker. Пример включения доверия для одной команды:

```bash
# Enable content trust for single command
docker build --disable-content-trust=false -t myacr.azurecr.io/myimage:v1 .
```

Если доверие уже включено для сеанса оболочки и нужно отключить его для одной команды, используйте следующий код:

```bash
# Disable content trust for single command
docker build --disable-content-trust -t myacr.azurecr.io/myimage:v1 .
```

## <a name="grant-image-signing-permissions"></a>Выдача разрешения на подписывание образов

Только пользователи или системы с разрешением на подписывание могут отправлять доверенные образы в ваш реестр. Чтобы выдать такое разрешение пользователю (или системе через субъект-службу), назначьте ему роль `AcrImageSigner` в Azure Active Directory в дополнение к роли `AcrPush` (или ее аналогу), необходимой для отправки образов в реестр. Дополнительные сведения см. в разделе [Роли и разрешения реестра контейнеров Azure](container-registry-roles.md).

> [!IMPORTANT]
> Вы не можете предоставить надежное разрешение на принудительную отправку изображений следующим учетным записям администратора: 
> * [учетная запись администратора](container-registry-authentication.md#admin-account) реестра контейнеров Azure
> * Учетная запись пользователя в Azure Active Directory с [ролью "Классический системный администратор](../role-based-access-control/rbac-and-directory-admin-roles.md#classic-subscription-administrator-roles)".

Далее описано, как назначить роль `AcrImageSigner` на портале Azure и в инфраструктуре Azure CLI.

### <a name="azure-portal"></a>Портал Azure

Перейдите в свой реестр на портале Azure и последовательно выберите **Управление доступом (IAM)**  > **Добавление назначения ролей**. В разделе **Добавление назначения ролей** выберите `AcrImageSigner` в поле **Роль**, затем одного или нескольких пользователей либо один или несколько субъектов-служб в поле **Выбрать** и выберите **Сохранить**.

В этом примере роли назначены две сущности `AcrImageSigner` : субъект-служба с именем "Service – Principal" и пользователь с именем "пользователь Azure".

![Предоставление разрешений на подпись образа записи контроля доступа в портал Azure][content-trust-02-portal]

### <a name="azure-cli"></a>Azure CLI

Чтобы выдать разрешение на подписывание с помощью Azure CLI, назначьте роль `AcrImageSigner` пользователю из вашего реестра. Команда имеет следующий формат.

```azurecli
az role assignment create --scope <registry ID> --role AcrImageSigner --assignee <user name>
```

Например, чтобы предоставить роли пользователя, не являющегося администратором, можно выполнить следующие команды в сеансе Azure CLI, прошедшем проверку подлинности. Измените значение `REGISTRY` в соответствии с именем реестра контейнеров Azure.

```bash
# Grant signing permissions to authenticated Azure CLI user
REGISTRY=myregistry
REGISTRY_ID=$(az acr show --name $REGISTRY --query id --output tsv)
```

```azurecli
az role assignment create --scope $REGISTRY_ID --role AcrImageSigner --assignee azureuser@contoso.com
```

Также можно выдать права на отправку образов в реестр [субъекту-службе](container-registry-auth-service-principal.md). Субъект-службу можно использовать для систем сборки и других автоматических систем, чтобы отправлять доверенные образы в ваш реестр. Действия аналогичны выдаче разрешения пользователю, только в значении `--assignee` указывается идентификатор субъекта-службы.

```azurecli
az role assignment create --scope $REGISTRY_ID --role AcrImageSigner --assignee <service principal ID>
```

В `<service principal ID>` можно указать **appId** или **objectId** субъекта-службы или одно из его имен **servicePrincipalNames**. Дополнительные сведения о работе с субъектами-службами и Реестром контейнеров Azure см. в статье [Аутентификация в реестре контейнеров Azure с помощью субъектов-служб](container-registry-auth-service-principal.md).

> [!IMPORTANT]
> После изменения роли выполните `az acr login` для обновления локального маркера идентификации для Azure CLI, чтобы новые роли вступили в действие. Сведения о проверке ролей для удостоверения см. в статье [Добавление и удаление назначений ролей Azure с помощью Azure CLI](../role-based-access-control/role-assignments-cli.md) и [Устранение неполадок в Azure RBAC](../role-based-access-control/troubleshooting.md).

## <a name="push-a-trusted-image"></a>Отправка доверенного образа

Чтобы отправить доверенный тег образа в реестр контейнеров, включите функцию доверия к содержимому и отправьте образ с помощью `docker push`. После первого выполнения принудительной отправки с подписанным тегом вам будет предложено создать парольную фразу как для корневого ключа подписывания, так и для ключа подписывания репозитория. Оба ключа создаются и хранятся локально на вашем компьютере.

```console
$ docker push myregistry.azurecr.io/myimage:v1
[...]
The push refers to repository [myregistry.azurecr.io/myimage]
ee83fc5847cb: Pushed
v1: digest: sha256:aca41a608e5eb015f1ec6755f490f3be26b48010b178e78c00eac21ffbe246f1 size: 524
Signing and pushing trust metadata
You are about to create a new root signing key passphrase. This passphrase
will be used to protect the most sensitive key in your signing system. Please
choose a long, complex passphrase and be careful to keep the password and the
key file itself secure and backed up. It is highly recommended that you use a
password manager to generate the passphrase and keep it safe. There will be no
way to recover this key. You can find the key in your config directory.
Enter passphrase for new root key with ID 4c6c56a:
Repeat passphrase for new root key with ID 4c6c56a:
Enter passphrase for new repository key with ID bcd6d98:
Repeat passphrase for new repository key with ID bcd6d98:
Finished initializing "myregistry.azurecr.io/myimage"
Successfully signed myregistry.azurecr.io/myimage:v1
```

После первого применения `docker push` с включенным доверием Docker-клиент будет использовать тот же корневой ключ для последующих отправок. Каждый раз при отправке в тот же репозиторий нужно вводить только ключ репозитория. При отправке в новый репозиторий создайте парольную фразу для нового ключа репозитория.

## <a name="pull-a-trusted-image"></a>Извлечение доверенного образа

Чтобы извлечь доверенный образ, включите функцию доверия к содержимому и выполните команду `docker pull` в обычном режиме. Для извлечения надежных образов обычным пользователям будет достаточно роли `AcrPull`. Дополнительные роли, например `AcrImageSigner`, не требуются. Получатели с включенным доверием могут извлекать только образы с подписанными тегами. Пример извлечения подписанного тега:

```console
$ docker pull myregistry.azurecr.io/myimage:signed
Pull (1 of 1): myregistry.azurecr.io/myimage:signed@sha256:0800d17e37fb4f8194495b1a188f121e5b54efb52b5d93dc9e0ed97fce49564b
sha256:0800d17e37fb4f8194495b1a188f121e5b54efb52b5d93dc9e0ed97fce49564b: Pulling from myimage
8e3ba11ec2a2: Pull complete
Digest: sha256:0800d17e37fb4f8194495b1a188f121e5b54efb52b5d93dc9e0ed97fce49564b
Status: Downloaded newer image for myregistry.azurecr.io/myimage@sha256:0800d17e37fb4f8194495b1a188f121e5b54efb52b5d93dc9e0ed97fce49564b
Tagging myregistry.azurecr.io/myimage@sha256:0800d17e37fb4f8194495b1a188f121e5b54efb52b5d93dc9e0ed97fce49564b as myregistry.azurecr.io/myimage:signed
```

Если клиент с включенным доверием содержимого пытается извлечь неподписанный тег, операция завершается ошибкой, как показано ниже.

```console
$ docker pull myregistry.azurecr.io/myimage:unsigned
Error: remote trust data does not exist
```

### <a name="behind-the-scenes"></a>Сопутствующие ресурсы

При выполнении `docker pull` клиент Docker использует ту же библиотеку, что и в [Notary CLI][docker-notary-cli], для запроса на сопоставление хэша извлекаемого тега с помощью SHA-256. После проверки подписей доверенных данных клиент выдает указание Docker Engine "извлечь по хэшу". При извлечении Engine использует контрольную сумму SHA-256 как адрес содержимого для запроса и проверки манифеста образа из реестра контейнеров Azure.

> [!NOTE]
> Реестр контейнеров Azure официально не поддерживает интерфейс командной строки Нотари, но совместим с API сервера Нотари, который входит в состав DOCKER Desktop. Сейчас рекомендуется Нотари версии **0.6.0** .

## <a name="key-management"></a>Управление ключами

Как указано в выходных данных `docker push`, наиболее важным при первой отправке доверенного образа является корневой ключ. Не забудьте создать для него резервную копию и храните ее в безопасном расположении. По умолчанию Docker-клиент хранит ключи подписывания здесь:

```sh
~/.docker/trust/private
```

Создайте резервную копию корневых ключей и ключей репозитория, поместив их в архив, и храните в безопасном расположении. Пример (bash):

```bash
umask 077; tar -zcvf docker_private_keys_backup.tar.gz ~/.docker/trust/private; umask 022
```

Другие ключи, наряду с корневыми ключами и ключами репозитория, созданными локально, создаются и хранятся в Реестре контейнеров Azure при отправке доверенного образа. Подробное описание различных типов ключей, а также советы по управлению ими в функции доверия к содержимому в Docker см. на странице [Manage keys for content trust][docker-manage-keys] (Управление ключами функции доверия к содержимому) документации Docker.

### <a name="lost-root-key"></a>Потеря корневого ключа

В случае потери корневого ключа вы теряете доступ к подписанным этим ключом тегам из любого репозитория. Реестр контейнеров Azure не восстанавливает доступ к таким тегам. Чтобы удалить все доверенные данные (подписи) в реестре, отключите, а затем снова включите доверие к содержимому для этого реестра.

> [!WARNING]
> При отключении и повторном включении доверия к содержимому в реестре **удаляются все доверенные данные всех подписанных тегов в каждом репозитории реестра**. Это действие невозможно отменить — Реестр контейнеров Azure не восстанавливает удаленные доверенные данные. Отключение доверия не приводит к удалению самих образов.

Чтобы отключить доверие к содержимому, перейдите в реестр на портале Azure. В разделе **Политики** последовательно выберите **Доверие к содержимому** > **Отключено** > **Сохранить**. Появится предупреждение о потере всех подписей в реестре. Выберите **ОК**, чтобы удалить все подписи в реестре без возможности восстановления.

![Отключение доверия к содержимому в реестре на портале Azure][content-trust-03-portal]

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о доверии содержимого, включая команды [доверия DOCKER](https://docs.docker.com/engine/reference/commandline/trust/) и [Делегирование доверия](https://docs.docker.com/engine/security/trust/trust_delegation/), см. в разделе " [доверие содержимого в DOCKER][docker-content-trust] ". В этой статье мы затронули лишь ключевые моменты обширной темы доверия к содержимому, которая более подробно рассматривается в документации Docker.

* Пример того, как применяется функция доверия к содержимому при создании и отправке образа Docker, см. в документации по [Azure Pipelines](/azure/devops/pipelines/build/content-trust).

<!-- IMAGES> -->
[content-trust-01-portal]: ./media/container-registry-content-trust/content-trust-01-portal.png
[content-trust-02-portal]: ./media/container-registry-content-trust/content-trust-02-portal.png
[content-trust-03-portal]: ./media/container-registry-content-trust/content-trust-03-portal.png

<!-- LINKS - external -->
[docker-content-trust]: https://docs.docker.com/engine/security/trust/content_trust
[docker-manage-keys]: https://docs.docker.com/engine/security/trust/trust_key_mng/
[docker-notary-cli]: https://docs.docker.com/notary/getting_started/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

<!-- LINKS - internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-config-content-trust-update]: /cli/azure/acr/config/content-trust#az-acr-config-content-trust-update
