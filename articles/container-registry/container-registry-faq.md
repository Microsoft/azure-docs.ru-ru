---
title: Часто задаваемые вопросы
description: Ответы на часто задаваемые вопросы, связанные со службой Реестра контейнеров Azure
author: sajayantony
ms.topic: article
ms.date: 03/15/2021
ms.author: sajaya
ms.openlocfilehash: 5550c53289228f154fab485b4b7bbff17555aad7
ms.sourcegitcommit: ed7376d919a66edcba3566efdee4bc3351c57eda
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105045745"
---
# <a name="frequently-asked-questions-about-azure-container-registry"></a>Часто задаваемые вопросы о Реестре контейнеров Azure

В этой статье рассматриваются часто задаваемые вопросы и известные проблемы Реестра контейнеров Azure.

Рекомендации по устранению неполадок в реестре см. в следующих разделах:
* [Устранение неполадок при входе в реестр](container-registry-troubleshoot-login.md)
* [Устранение проблем с сетью с помощью реестра](container-registry-troubleshoot-access.md)
* [Устранение проблем с производительностью реестра](container-registry-troubleshoot-performance.md)

## <a name="resource-management"></a>Управление ресурсами

- [Можно ли создать реестр контейнеров Azure с помощью шаблона Resource Manager?](#can-i-create-an-azure-container-registry-using-a-resource-manager-template)
- [Есть ли возможность проверки образов в ACR на уязвимости системы безопасности?](#is-there-security-vulnerability-scanning-for-images-in-acr)
- [Как настроить Kubernetes с помощью Реестра контейнеров Azure?](#how-do-i-configure-kubernetes-with-azure-container-registry)
- [Как получить учетные данные администратора для реестра контейнеров?](#how-do-i-get-admin-credentials-for-a-container-registry)
- [Как получить учетные данные администратора в шаблоне Resource Manager?](#how-do-i-get-admin-credentials-in-a-resource-manager-template)
- [Удаление репликации завершается сбоем с запрещенным состоянием, хотя репликация удаляется с помощью Azure CLI или Azure PowerShell](#delete-of-replication-fails-with-forbidden-status-although-the-replication-gets-deleted-using-the-azure-cli-or-azure-powershell)
- [Правила брандмауэра успешно обновляются, но не вступают в силу](#firewall-rules-are-updated-successfully-but-they-do-not-take-effect)

### <a name="can-i-create-an-azure-container-registry-using-a-resource-manager-template"></a>Можно ли создать реестр контейнеров Azure с помощью шаблона Resource Manager?

Да. Вот [шаблон](https://github.com/Azure/azure-quickstart-templates/tree/master/101-container-registry), который можно использовать для создания реестра.

### <a name="is-there-security-vulnerability-scanning-for-images-in-acr"></a>Есть ли возможность проверки образов в ACR на уязвимости системы безопасности?

Да. См. документацию из [Центра безопасности Azure](../security-center/defender-for-container-registries-introduction.md), [Twistlock](https://www.twistlock.com/2016/11/07/twistlock-supports-azure-container-registry/) и [Aqua](https://blog.aquasec.com/image-vulnerability-scanning-in-azure-container-registry).

### <a name="how-do-i-configure-kubernetes-with-azure-container-registry"></a>Как настроить Kubernetes с помощью Реестра контейнеров Azure?

См. документацию по [Kubernetes](https://kubernetes.io/docs/user-guide/images/#using-azure-container-registry-acr) и инструкции для [Службы Azure Kubernetes](../aks/cluster-container-registry-integration.md).

### <a name="how-do-i-get-admin-credentials-for-a-container-registry"></a>Как получить учетные данные администратора для реестра контейнеров?

> [!IMPORTANT]
> Учетная запись администратора предоставляет доступ к реестру одному пользователю. Она предназначена, главным образом, для тестирования. Не рекомендуется совместное использование учетных данных администратора несколькими пользователями. Для пользователей и субъектов-служб в сценариях автоматического входа рекомендуется использовать отдельные удостоверения. См. [общие сведения о проверке подлинности](container-registry-authentication.md).

Перед получением учетных данных администратора убедитесь в том, что администратор реестра включен.

Получение учетных данных с помощью Azure CLI:

```azurecli
az acr credential show -n myRegistry
```

С помощью Azure PowerShell:

```powershell
Invoke-AzureRmResourceAction -Action listCredentials -ResourceType Microsoft.ContainerRegistry/registries -ResourceGroupName myResourceGroup -ResourceName myRegistry
```

### <a name="how-do-i-get-admin-credentials-in-a-resource-manager-template"></a>Как получить учетные данные администратора в шаблоне Resource Manager?

> [!IMPORTANT]
> Учетная запись администратора предоставляет доступ к реестру одному пользователю. Она предназначена, главным образом, для тестирования. Не рекомендуется совместное использование учетных данных администратора несколькими пользователями. Для пользователей и субъектов-служб в сценариях автоматического входа рекомендуется использовать отдельные удостоверения. См. [общие сведения о проверке подлинности](container-registry-authentication.md).

Перед получением учетных данных администратора убедитесь в том, что администратор реестра включен.

Получение первого пароля:

```json
{
    "password": "[listCredentials(resourceId('Microsoft.ContainerRegistry/registries', 'myRegistry'), '2017-10-01').passwords[0].value]"
}
```

Получение второго пароля:

```json
{
    "password": "[listCredentials(resourceId('Microsoft.ContainerRegistry/registries', 'myRegistry'), '2017-10-01').passwords[1].value]"
}
```

### <a name="delete-of-replication-fails-with-forbidden-status-although-the-replication-gets-deleted-using-the-azure-cli-or-azure-powershell"></a>Удаление репликации завершается сбоем с запрещенным состоянием, хотя репликация удаляется с помощью Azure CLI или Azure PowerShell

Эта ошибка возникает, когда у пользователя есть разрешения на доступ к реестру, но нет разрешений для чтения подписки. Чтобы решить эту проблему, назначьте пользователю разрешения читателя для подписки:


```azurecli  
az role assignment create --role "Reader" --assignee user@contoso.com --scope /subscriptions/<subscription_id> 
```

### <a name="firewall-rules-are-updated-successfully-but-they-do-not-take-effect"></a>Правила брандмауэра успешно обновляются, но не вступают в силу

Распространение изменений, внесенных в правила брандмауэра, занимает некоторое время. После изменения параметров брандмауэра подождите несколько минут, прежде чем проверять это изменение.


## <a name="registry-operations"></a>Операции с реестром

- [Как получить доступ к API HTTP для реестра Docker версии 2?](#how-do-i-access-docker-registry-http-api-v2)
- [Как удалить все манифесты, на которые не ссылается ни один тег в репозитории?](#how-do-i-delete-all-manifests-that-are-not-referenced-by-any-tag-in-a-repository)
- [Почему израсходованная квота реестра не уменьшается после удаления образов?](#why-does-the-registry-quota-usage-not-reduce-after-deleting-images)
- [Как проверить изменения квоты хранилища?](#how-do-i-validate-storage-quota-changes)
- [Как пройти проверку подлинности в реестре при запуске CLI в контейнере?](#how-do-i-authenticate-with-my-registry-when-running-the-cli-in-a-container)
- [Как включить протокол TLS 1.2?](#how-to-enable-tls-12)
- [Поддерживает ли Реестр контейнеров Azure доверие содержимому?](#does-azure-container-registry-support-content-trust)
- [Как предоставить доступ к извлекаемым или передаваемым образам без разрешения на управление ресурсом реестра?](#how-do-i-grant-access-to-pull-or-push-images-without-permission-to-manage-the-registry-resource)
- [Как включить автоматическое помещение образов в карантин для реестра?](#how-do-i-enable-automatic-image-quarantine-for-a-registry)
- [Как включить анонимный доступ для извлечения?](#how-do-i-enable-anonymous-pull-access)
- [Разделы справки отправлять нераспространяемые слои в реестр?](#how-do-i-push-non-distributable-layers-to-a-registry)

### <a name="how-do-i-access-docker-registry-http-api-v2"></a>Как получить доступ к API HTTP для реестра Docker версии 2?

ACR поддерживает API HTTP для реестра Docker версии 2. Доступ к API можно получить по адресу `https://<your registry login server>/v2/`. Например, `https://mycontainerregistry.azurecr.io/v2/`.

### <a name="how-do-i-delete-all-manifests-that-are-not-referenced-by-any-tag-in-a-repository"></a>Как удалить все манифесты, на которые не ссылается ни один тег в репозитории?

Если вы используете bash:

```azurecli
az acr repository show-manifests -n myRegistry --repository myRepository --query "[?tags[0]==null].digest" -o tsv  | xargs -I% az acr repository delete -n myRegistry -t myRepository@%
```

Для PowerShell:

```azurecli
az acr repository show-manifests -n myRegistry --repository myRepository --query "[?tags[0]==null].digest" -o tsv | %{ az acr repository delete -n myRegistry -t myRepository@$_ }
```

Примечание. Вы можете добавить `-y` в команду удаления, чтобы пропустить подтверждение.

Дополнительные сведения см. в статье [Удаление образов контейнеров в службе "Реестр контейнеров Azure"](container-registry-delete.md).

### <a name="why-does-the-registry-quota-usage-not-reduce-after-deleting-images"></a>Почему израсходованная квота реестра не уменьшается после удаления образов?

Такая ситуация может возникнуть, если другие образы контейнеров по-прежнему ссылаются на базовые слои. Если удалить образ без ссылок, данные по использованию реестра будут обновлены через несколько минут.

### <a name="how-do-i-validate-storage-quota-changes"></a>Как проверить изменения квоты хранилища?

Создайте образ с уровнем 1 ГБ, используя приведенный ниже файл Docker. В результате образ будет иметь уровень, который не используется ни одним другим образом в реестре.

```dockerfile
FROM alpine
RUN dd if=/dev/urandom of=1GB.bin  bs=32M  count=32
RUN ls -lh 1GB.bin
```

Выполните сборку образа и отправьте его в реестр с помощью CLI Docker.

```bash
docker build -t myregistry.azurecr.io/1gb:latest .
docker push myregistry.azurecr.io/1gb:latest
```

Вы можете увидеть, что использование хранилища увеличилось, на портале Azure либо запросить данные по использованию с помощью интерфейса CLI.

```azurecli
az acr show-usage -n myregistry
```

Удалите образ с помощью Azure CLI или портала и проверьте обновленные данные по использованию через несколько минут.

```azurecli
az acr repository delete -n myregistry --image 1gb
```

### <a name="how-do-i-authenticate-with-my-registry-when-running-the-cli-in-a-container"></a>Как пройти проверку подлинности в реестре при запуске CLI в контейнере?

Необходимо запустить контейнер Azure CLI, подключив сокет Docker:

```bash
docker run -it -v /var/run/docker.sock:/var/run/docker.sock azuresdk/azure-cli-python:dev
```

В контейнере установите `docker`:

```bash
apk --update add docker
```

Затем пройдите проверку подлинности в реестре:

```azurecli
az acr login -n MyRegistry
```

### <a name="how-to-enable-tls-12"></a>Как включить протокол TLS 1.2?

Включите TLS 1.2 с помощью любого нового клиента Docker (версии не ниже 18.03.0). 

> [!IMPORTANT]
> С 13 января 2020 года Реестр контейнеров Azure будет требовать использовать протокол TLS версии 1.2 для всех безопасных подключений серверов и приложений. Поддержка протоколов TLS версии 1.0 и 1.1 будет прекращена.

### <a name="does-azure-container-registry-support-content-trust"></a>Поддерживает ли Реестр контейнеров Azure доверие содержимому?

Да, вы можете использовать доверенные образы в Реестре контейнеров Azure, так как служба [Docker Notary](https://docs.docker.com/notary/getting_started/) была интегрирована и ее можно включить. Подробные сведения см. в статье [Доверие содержимому в Реестре контейнеров Azure](container-registry-content-trust.md).


####  <a name="where-is-the-file-for-the-thumbprint-located"></a>Где находится файл для отпечатка?

По пути `~/.docker/trust/tuf/myregistry.azurecr.io/myrepository/metadata`:

* Открытые ключи и сертификаты всех ролей (за исключением ролей делегирования) хранятся в `root.json`.
* Открытые ключи и сертификаты роли делегирования хранятся в JSON-файле родительской роли (например, `targets.json` для роли `targets/releases`).

Рекомендуется проверить эти открытые ключи и сертификаты после общей проверки TUF, выполненной Docker и клиентом Notary.

### <a name="how-do-i-grant-access-to-pull-or-push-images-without-permission-to-manage-the-registry-resource"></a>Как предоставить доступ к извлекаемым или передаваемым образам без разрешения на управление ресурсом реестра?

ACR поддерживает [пользовательские роли](container-registry-roles.md), предоставляющие различные уровни разрешений. В частности, роли `AcrPull` и `AcrPush` позволяют пользователям получать и отправлять образы без разрешения на управление ресурсом реестра в Azure.

* Портал Azure: ваш реестр -> Управление доступом (IAM) -> Добавить (выберите роль `AcrPull` или `AcrPush`).
* Azure CLI: найдите идентификатор ресурса реестра, выполнив следующую команду:

  ```azurecli
  az acr show -n myRegistry
  ```
  
  Затем можно назначить пользователю роль `AcrPull` или `AcrPush` (в следующем примере используется `AcrPull`):

  ```azurecli
  az role assignment create --scope resource_id --role AcrPull --assignee user@example.com
  ```

  Или назначьте роль субъекту-службе, определяемому ИДЕНТИФИКАТОРом приложения:

  ```azurecli
  az role assignment create --scope resource_id --role AcrPull --assignee 00000000-0000-0000-0000-000000000000
  ```

После этого пользователь с ролью сможет проходить проверку подлинности и получать доступ к образам в реестре.

* Проверка подлинности в реестре:
    
  ```azurecli
  az acr login -n myRegistry 
  ```

* Вывод списка репозиториев:

  ```azurecli
  az acr repository list -n myRegistry
  ```

* Получение образа:

  ```bash
  docker pull myregistry.azurecr.io/hello-world
  ```

При использовании только роли `AcrPull` или `AcrPush` у пользователя нет разрешения на управление ресурсом реестра в Azure. Например, с помощью команды `az acr list` или `az acr show -n myRegistry` нельзя просмотреть реестр.

### <a name="how-do-i-enable-automatic-image-quarantine-for-a-registry"></a>Как включить автоматическое помещение образов в карантин для реестра?

В настоящее время помещение образов в карантин — это предварительная версия функции в ACR. Вы можете включить режим карантина для реестра, чтобы обычным пользователям были видны только те образы, которые успешно прошли проверку безопасности. Подробные сведения см. в [репозитории ACR в GitHub](https://github.com/Azure/acr/tree/master/docs/preview/quarantine).

### <a name="how-do-i-enable-anonymous-pull-access"></a>Как включить анонимный доступ для извлечения?

Настройка реестра контейнеров Azure для анонимного доступа (без проверки подлинности) в настоящее время является функцией предварительной версии, доступной на [уровнях служб](container-registry-skus.md)"Стандартный" и "Премиум". 

Чтобы включить анонимный доступ по запросу, обновите реестр с помощью Azure CLI (2.21.0 или более поздней версии) и передайте `--anonymous-pull-enabled` параметр в команду [AZ контроля](/cli/azure/acr#az_acr_update) доступа.

```azurecli
az acr update --name myregistry --anonymous-pull-enabled
``` 

Анонимный доступ по запросу можно отключить в любое время, задав `--anonymous-pull-enabled` для значение `false` .

> [!NOTE]
> * Прежде чем пытаться выполнить анонимную операцию извлечения, выполните команду, `docker logout` чтобы убедиться, что удалены все существующие учетные данные DOCKER.
> * Для клиентов без проверки подлинности доступны только операции плоскости данных.
> * Реестр может регулировать высокую частоту запросов без проверки подлинности.

> [!WARNING]
> Анонимный доступ в настоящее время применяется ко всем репозиториям в реестре. Если вы управляете доступом к репозиторию с помощью [маркеров уровня репозитория](container-registry-repository-scoped-permissions.md), имейте в виду, что все пользователи могут извлекать из этих репозиториев в реестре, для которого включено анонимное извлечение. Рекомендуется удалять маркеры, если включен анонимный доступ по запросу.

### <a name="how-do-i-push-non-distributable-layers-to-a-registry"></a>Разделы справки отправлять нераспространяемые слои в реестр?

Нераспространяемый слой в манифесте содержит параметр URL-адреса, из которого может быть получено содержимое. Некоторые возможные варианты использования для включения нераспространяемых слоев заключаются в ограниченном сетевом реестре, Air-гаппед реестра с ограниченным доступом или для реестров без подключения к Интернету.

Например, если у вас настроены правила NSG, чтобы виртуальная машина могла извлекать образы только из реестра контейнеров Azure, Docker выдаст сбои для внешних и нераспространяемых слоев. Например, образ Windows Server Core будет содержать ссылки на внешний слой в реестре контейнеров Azure в своем манифесте и не сможет запрашивать этот сценарий.

Чтобы разрешить принудительную отправку нераспространяемых слоев:

1. Измените `daemon.json` файл, который находится на `/etc/docker/` узлах Linux и в `C:\ProgramData\docker\config\daemon.json` Windows Server. Если файл был ранее пустым, добавьте следующее содержимое:

   ```json
   {
     "allow-nondistributable-artifacts": ["myregistry.azurecr.io"]
   }
   ```
   > [!NOTE]
   > Значение представляет собой массив адресов реестра, разделенных запятыми.

2. Сохраните и закройте файл.

3. Перезапустите DOCKER.

При отправке изображений в реестр в списке их нераспространяемые слои помещаются в раздел реестра.

> [!WARNING]
> Артефакты, не являющиеся распространяемыми, обычно имеют ограничения на то, как и где они могут распространяться и предоставляться для общего доступа. Используйте эту функцию только для отправки артефактов в частные реестры. Убедитесь, что вы соответствуете всем терминам, охватывающим распространение нераспространяемых артефактов.

## <a name="diagnostics-and-health-checks"></a>Диагностика и проверки работоспособности

- [Проверка работоспособности с помощью `az acr check-health`](#check-health-with-az-acr-check-health)
- [Команда docker pull завершается ошибкой: "net/http: запрос отменен во время ожидания подключения (превышено значение Client.Timeout во время ожидания заголовков)"](#docker-pull-fails-with-error-nethttp-request-canceled-while-waiting-for-connection-clienttimeout-exceeded-while-awaiting-headers)
- [Команда docker push выполняется успешно, но команда docker pull завершается ошибкой: "не авторизовано: требуется проверка подлинности"](#docker-push-succeeds-but-docker-pull-fails-with-error-unauthorized-authentication-required)
- [Команда `az acr login` выполняется успешно, но команды docker завершаются ошибкой: "не авторизовано: требуется проверка подлинности"](#az-acr-login-succeeds-but-docker-fails-with-error-unauthorized-authentication-required)
- [Включение и получение журналов отладки для управляющей программы Docker](#enable-and-get-the-debug-logs-of-the-docker-daemon)    
- [Новые разрешения пользователей могут вступать в силу не сразу после обновления](#new-user-permissions-may-not-be-effective-immediately-after-updating)
- [Данные для проверки подлинности предоставляются в неправильном формате при прямых вызовах REST API](#authentication-information-is-not-given-in-the-correct-format-on-direct-rest-api-calls)
- [Почему на портале Azure отображаются не все мои репозитории или теги?](#why-does-the-azure-portal-not-list-all-my-repositories-or-tags)
- [Почему порталу Azure не удается получить репозитории или теги?](#why-does-the-azure-portal-fail-to-fetch-repositories-or-tags)
- [Почему не удается выполнить запрос на вытягивание или отправку из-за неразрешенной операции?](#why-does-my-pull-or-push-request-fail-with-disallowed-operation)
- [Формат репозитория является недопустимым или не поддерживается](#repository-format-is-invalid-or-unsupported)
- [Как собрать данные трассировки HTTP в Windows?](#how-do-i-collect-http-traces-on-windows)

### <a name="check-health-with-az-acr-check-health"></a>Проверка работоспособности с помощью `az acr check-health`

Сведения об устранении распространенных проблем среды и реестра см. в статье [Проверка работоспособности реестра контейнеров Azure](container-registry-check-health.md).

### <a name="docker-pull-fails-with-error-nethttp-request-canceled-while-waiting-for-connection-clienttimeout-exceeded-while-awaiting-headers"></a>Команда docker pull завершается ошибкой: "net/http: запрос отменен во время ожидания подключения (превышено значение Client.Timeout во время ожидания заголовков)"

 - Если эта ошибка является временной, повторная попытка завершится успешно.
 - Если команда `docker pull` постоянно завершается сбоем, возможно, имеется проблема с управляющей программой Docker. Как правило, такую проблему можно устранить путем перезапуска управляющей программы Docker. 
 - Если эта проблема не пропадает после перезапуска управляющей программы Docker, она может быть связана с неполадками сетевого подключения на компьютере. Чтобы проверить общую работоспособность сети на компьютере, выполните приведенную ниже команду для проверки подключения к конечной точке. Минимальная версия `az acr`, содержащая эту команду проверки подключения, — 2.2.9. Если используется более старая версия, обновите Azure CLI.
 
  ```azurecli
  az acr check-health -n myRegistry
  ```

 - Все операции клиента Docker должны предусматривать механизм повторного выполнения.

### <a name="docker-pull-is-slow"></a>Команда pull Docker выполняется медленно
Используйте [это средство](http://www.azurespeed.com/Azure/Download) для проверки скорости скачивания по сети на компьютере. Если сетевое подключение компьютера работает медленно, попробуйте использовать виртуальную машину Azure в том же регионе, что и реестр. Обычно это позволяет повысить скорость сети.

### <a name="docker-push-is-slow"></a>Команда push Docker выполняется медленно
Используйте [это средство](http://www.azurespeed.com/Azure/Upload) для проверки скорости отправки по сети на компьютере. Если сетевое подключение компьютера работает медленно, попробуйте использовать виртуальную машину Azure в том же регионе, что и реестр. Обычно это позволяет повысить скорость сети.

### <a name="docker-push-succeeds-but-docker-pull-fails-with-error-unauthorized-authentication-required"></a>Команда docker push выполняется успешно, но команда docker pull завершается ошибкой: "не авторизовано: требуется проверка подлинности"

Эта ошибка может возникать при использовании версии Red Hat управляющей программы Docker, в которой параметр `--signature-verification` включен по умолчанию. Проверить параметры управляющей программы Docker для Red Hat Enterprise Linux (RHEL) или Fedora можно с помощью следующей команды:

```bash
grep OPTIONS /etc/sysconfig/docker
```

Например, Fedora 28 Server имеет следующие параметры управляющей программы Docker:

`OPTIONS='--selinux-enabled --log-driver=journald --live-restore'`

При отсутствии `--signature-verification=false` команда `docker pull` завершается ошибкой наподобие следующей:

```output
Trying to pull repository myregistry.azurecr.io/myimage ...
unauthorized: authentication required
```

Чтобы устранить эту ошибку, сделайте следующее:
1. Добавьте параметр `--signature-verification=false` в файл конфигурации управляющей программы Docker `/etc/sysconfig/docker`. Пример:
   
   `OPTIONS='--selinux-enabled --log-driver=journald --live-restore --signature-verification=false'`
   
2. Перезапустите службу управляющей программы Docker, выполнив следующую команду:
   
   ```bash
   sudo systemctl restart docker.service
   ```

Чтобы получить подробные сведения о `--signature-verification`, выполните команду `man dockerd`.

### <a name="az-acr-login-succeeds-but-docker-fails-with-error-unauthorized-authentication-required"></a>Команда az acr login выполняется успешно, но команды docker завершаются ошибкой: "не авторизовано: требуется проверка подлинности"

Убедитесь в том, что используется URL-адрес сервера в нижнем регистре, например `docker push myregistry.azurecr.io/myimage:latest`, даже если имя ресурса реестра содержит символы в верхнем или смешанном регистре, например `myRegistry`.

### <a name="enable-and-get-the-debug-logs-of-the-docker-daemon"></a>Включение и получение журналов отладки для управляющей программы Docker    

Запустите `dockerd` с параметром `debug`. Сначала создайте файл конфигурации управляющей программы Docker (`/etc/docker/daemon.json`), если его нет, и добавьте параметр `debug`:

```json
{    
    "debug": true    
}
```

Затем перезапустите управляющую программу. Например, в Ubuntu 14.04:

```bash
sudo service docker restart
```

Подробные сведения можно найти в [документации по Docker](https://docs.docker.com/engine/admin/#enable-debugging).    

 * Журналы могут создаваться в разных местах в зависимости от используемой системы. Например, в Ubuntu 14.04 это `/var/log/upstart/docker.log`.    
Подробные сведения см. в [документации по Docker](https://docs.docker.com/engine/admin/#read-the-logs).    

 * Для Docker для Windows журналы создаются в папке %LOCALAPPDATA%/docker/. Однако они пока могут содержать не всю отладочную информацию.    

   Для доступа к полному журналу управляющей программы могут потребоваться дополнительные действия:

    ```console
    docker run --privileged -it --rm -v /var/run/docker.sock:/var/run/docker.sock -v /usr/local/bin/docker:/usr/local/bin/docker alpine sh

    docker run --net=host --ipc=host --uts=host --pid=host -it --security-opt=seccomp=unconfined --privileged --rm -v /:/host alpine /bin/sh
    chroot /host
    ```
    Теперь у вас есть доступ ко всем файлам виртуальной машины, в которой выполняется `dockerd`. Журнал находится в файле `/var/log/docker.log`.

### <a name="new-user-permissions-may-not-be-effective-immediately-after-updating"></a>Новые разрешения пользователей могут вступать в силу не сразу после обновления

При предоставлении новых разрешений (ролей) субъекту-службе изменение может вступать в силу не сразу. Есть две возможные причины.

* Задержка назначения ролей Azure Active Directory. Обычно оно происходит быстро, но из-за задержки распространения может занимать несколько минут.
* Задержка применения разрешений на сервере токенов ACR. Может составлять до 10 минут. Чтобы устранить эту проблемы, можно выполнить команду `docker logout`, а затем снова пройти проверку подлинности с теми же учетными данными через одну минуту:

  ```bash
  docker logout myregistry.azurecr.io
  docker login myregistry.azurecr.io
  ```

В настоящее время ACR не поддерживает удаление домашней репликации пользователями. В качестве обходного пути можно включить в шаблон команду создания домашней репликации, но пропустить ее создание, добавив `"condition": false`, как показано ниже:

```json
{
    "name": "[concat(parameters('acrName'), '/', parameters('location'))]",
    "condition": false,
    "type": "Microsoft.ContainerRegistry/registries/replications",
    "apiVersion": "2017-10-01",
    "location": "[parameters('location')]",
    "properties": {},
    "dependsOn": [
        "[concat('Microsoft.ContainerRegistry/registries/', parameters('acrName'))]"
     ]
},
```

### <a name="authentication-information-is-not-given-in-the-correct-format-on-direct-rest-api-calls"></a>Данные для проверки подлинности предоставляются в неправильном формате при прямых вызовах REST API

Может возникнуть ошибка `InvalidAuthenticationInfo`, особенно при использовании средства `curl` с параметром `-L`, `--location` (для поддержки перенаправлений).
Например, получение большого двоичного объекта с помощью `curl` с параметром `-L` и обычной проверкой подлинности:

```bash
curl -L -H "Authorization: basic $credential" https://$registry.azurecr.io/v2/$repository/blobs/$digest
```

может привести к следующему ответу:

```xml
<?xml version="1.0" encoding="utf-8"?>
<Error><Code>InvalidAuthenticationInfo</Code><Message>Authentication information is not given in the correct format. Check the value of Authorization header.
RequestId:00000000-0000-0000-0000-000000000000
Time:2019-01-01T00:00:00.0000000Z</Message></Error>
```

Основная причина заключается в том, что некоторые реализации `curl` поддерживают перенаправления с заголовками из исходного запроса.

Чтобы устранить эту проблему, необходимо выполнять перенаправления вручную без заголовков. Выведите заголовки ответа с параметром `-D -` `curl`, а затем извлеките заголовок `Location`:

```bash
redirect_url=$(curl -s -D - -H "Authorization: basic $credential" https://$registry.azurecr.io/v2/$repository/blobs/$digest | grep "^Location: " | cut -d " " -f2 | tr -d '\r')
curl $redirect_url
```

### <a name="why-does-the-azure-portal-not-list-all-my-repositories-or-tags"></a>Почему на портале Azure отображаются не все мои репозитории или теги? 

В браузере Microsoft Edge или Internet Explorer отображается не более 100 репозиториев или тегов. Если в реестре более 100 репозиториев или тегов, для вывода их списка рекомендуется использовать браузер Firefox или Chrome.

### <a name="why-does-the-azure-portal-fail-to-fetch-repositories-or-tags"></a>Почему порталу Azure не удается получить репозитории или теги?

Возможно, браузеру не удается отправить на сервер запрос на получение репозиториев или тегов. Это может происходить по разным причинам:

* отсутствие подключения к сети;
* Брандмауэр
* блокировщики рекламы;
* ошибки DNS.

Обратитесь к администратору сети или проверьте конфигурацию сети и подключение. Попробуйте выполнить команду `az acr check-health -n yourRegistry` в Azure CLI, чтобы проверить, может ли среда подключиться к Реестру контейнеров. Кроме того, в браузере можно попробовать использовать режим инкогнито или частный сеанс, чтобы избежать проблем с устаревшим кэшем или файлами cookie.

### <a name="why-does-my-pull-or-push-request-fail-with-disallowed-operation"></a>Почему не удается выполнить запрос на вытягивание или отправку из-за неразрешенной операции?

Ниже приведены некоторые сценарии, в которых операции могут быть запрещены.
* Классические реестры больше не поддерживаются. Выполните обновление до поддерживаемого [уровня служб](./container-registry-skus.md) с помощью команды [az acr update](/cli/azure/acr#az-acr-update) или на портале Azure.
* Образ или репозиторий может быть заблокирован, поэтому его нельзя удалить или обновить. Для просмотра текущих атрибутов можно воспользоваться командой [az acr show repository](./container-registry-image-lock.md).
* Некоторые операции запрещены, если образ находится на карантине. Дополнительные сведения о [карантине](https://github.com/Azure/acr/tree/master/docs/preview/quarantine).
* Возможно, ваш реестр достиг [предельного размера хранилища](container-registry-skus.md#service-tier-features-and-limits).

### <a name="repository-format-is-invalid-or-unsupported"></a>Формат репозитория является недопустимым или не поддерживается

Если отображается сообщение об ошибке, например "неподдерживаемый формат репозитория", "Недопустимый формат" или "запрошенные данные не существуют", при указании имени репозитория в операциях репозитория Проверьте правильность написания имени и регистра. Допустимые имена репозитория могут содержать только буквы в нижнем регистре, точки, тире, символы подчеркивания и символы косой черты. 

Полные правила именования репозитория см. в статье [Спецификация распространения в рамках инициативы Open Container](https://github.com/docker/distribution/blob/master/docs/spec/api.md#overview).

### <a name="how-do-i-collect-http-traces-on-windows"></a>Как собрать данные трассировки HTTP в Windows?

#### <a name="prerequisites"></a>Предварительные требования

- Включите расшифровку HTTPS в fiddler: <https://docs.telerik.com/fiddler/Configure-Fiddler/Tasks/DecryptHTTPS>
- Включите в Docker использование прокси-сервера через пользовательский интерфейс Docker: <https://docs.docker.com/docker-for-windows/#proxies>
- По завершении отмените эти изменения.  Docker не будет работать, если эта функция включена и fiddler не запущен.

#### <a name="windows-containers"></a>Контейнеры Windows

Настройка адрес прокси-сервера Docker 127.0.0.1:8888.

#### <a name="linux-containers"></a>Контейнеры Linux

Определите IP-адрес виртуального коммутатора виртуальной машины Docker:

```powershell
(Get-NetIPAddress -InterfaceAlias "*Docker*" -AddressFamily IPv4).IPAddress
```

Задайте в качестве адреса прокси-сервера Docker выходные данные предыдущей команды и укажите порт 8888 (например, 10.0.75.1:8888).

## <a name="tasks"></a>Задания

- [Как массово отменить выполнения?](#how-do-i-batch-cancel-runs)
- [Как включить папку .git в команду az acr build?](#how-do-i-include-the-git-folder-in-az-acr-build-command)
- [Поддерживают ли задачи GitLab для исходных триггеров?](#does-tasks-support-gitlab-for-source-triggers)
- [Какая служба управления репозиторием GIT поддерживает задачи?](#what-git-repository-management-service-does-tasks-support)

### <a name="how-do-i-batch-cancel-runs"></a>Как массово отменить выполнения?

Приведенные ниже команды отменяют все выполняемые задачи в указанном реестре.

```azurecli
az acr task list-runs -r $myregistry --run-status Running --query '[].runId' -o tsv \
| xargs -I% az acr task cancel-run -r $myregistry --run-id %
```

### <a name="how-do-i-include-the-git-folder-in-az-acr-build-command"></a>Как включить папку .git в команду az acr build?

Если передать локальную папку с исходным кодом в команду `az acr build`, папка `.git` по умолчанию исключается из отправляемого пакета. Можно создать файл `.dockerignore` со следующим параметром. Он предписывает команде восстановить все файлы из папки `.git` в отправляемом пакете. 

`!.git/**`

Этот параметр также применяется к команде `az acr run`.

### <a name="does-tasks-support-gitlab-for-source-triggers"></a>Поддерживают ли задачи GitLab для исходных триггеров?

В настоящее время GitLab для исходных триггеров не поддерживается.

### <a name="what-git-repository-management-service-does-tasks-support"></a>Какая служба управления репозиторием GIT поддерживает задачи?

| Служба GIT | Исходный контекст | Ручная сборка | Автоматическая сборка с помощью триггера фиксации |
|---|---|---|---|
| GitHub | `https://github.com/user/myapp-repo.git#mybranch:myfolder` | Да | Да |
| Azure Repos | `https://dev.azure.com/user/myproject/_git/myapp-repo#mybranch:myfolder` | Да | Да |
| GitLab | `https://gitlab.com/user/myapp-repo.git#mybranch:myfolder` | Да | Нет |
| Bitbucket; | `https://user@bitbucket.org/user/mayapp-repo.git#mybranch:myfolder` | Да | Нет |

## <a name="run-error-message-troubleshooting"></a>Устранение ошибок выполнения

| Сообщение об ошибке | Руководство по устранению неполадок |
|---|---|
|Доступ к виртуальной машине не настроен, поэтому подписки не найдены|Это может произойти, если вы используете `az login --identity` в задаче ACR. Это временная ошибка, которая возникает, если назначение роли для управляемого удостоверения еще не было распространено. Подождите несколько секунд, а затем повторите попытку.|

## <a name="cicd-integration"></a>Интеграция средств непрерывной интеграции и доставки (CI/CD)

- [CircleCI](https://github.com/Azure/acr/blob/master/docs/integration/CircleCI.md)
- [GitHub Actions](https://github.com/Azure/acr/blob/master/docs/integration/github-actions/github-actions.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Дополнительные сведения](container-registry-intro.md) о Реестре контейнеров Azure.
