---
title: Устранение неполадок при входе в реестр
description: Симптомы, причины и способы устранения распространенных проблем при входе в реестр контейнеров Azure
ms.topic: article
ms.date: 08/11/2020
ms.openlocfilehash: 5deb1717cf3886d8ea9c021d92afa358946b16dc
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99052084"
---
# <a name="troubleshoot-registry-login"></a>Устранение неполадок при входе в реестр

Эта статья поможет вам устранить неполадки, которые могут возникнуть при входе в реестр контейнеров Azure. 

## <a name="symptoms"></a>Симптомы

Может включать один или несколько из следующих элементов:

* Не удается войти в реестр с помощью `docker login` , `az acr login` или обоих
* Не удалось войти в реестр, вы получаете сообщение об ошибке `unauthorized: authentication required` или `unauthorized: Application not registered with AAD`
* Не удалось войти в реестр, вы получаете Azure CLI ошибку `Could not connect to the registry login server`
* Не удается отправить или извлечь образы, и вы получаете сообщение об ошибке DOCKER `unauthorized: authentication required`
* Не удалось получить доступ к реестру из службы Kubernetes Azure, Azure DevOps или другой службы Azure.
* Не удается получить доступ к реестру. Вы получаете сообщение об ошибке. `Error response from daemon: login attempt failed with status: 403 Forbidden` см. [раздел Устранение неполадок в сети](container-registry-troubleshoot-access.md) .
* Не удается получить доступ или просмотреть параметры реестра в портал Azure или управлять реестром с помощью Azure CLI

## <a name="causes"></a>Причины

* Неправильно настроен DOCKER в вашей среде — [решение](#check-docker-configuration)
* Реестр не существует, или указано неверное имя — [решение](#specify-correct-registry-name)
* Учетные данные реестра не являются допустимыми — [решение](#confirm-credentials-to-access-registry)
* Учетные данные не имеют прав для принудительного выполнения операций Push, [](#confirm-credentials-are-authorized-to-access-registry) pull или Azure Resource Manager.
* Срок действия учетных данных истек — [решение](#check-that-credentials-arent-expired)

## <a name="further-diagnosis"></a>Дальнейшая диагностика 

Выполните команду [AZ контроля доступа проверки на работоспособность](/cli/azure/acr#az-acr-check-health) , чтобы получить дополнительные сведения о работоспособности среды реестра и при необходимости доступ к целевому реестру. Например, диагностика ошибок конфигурации DOCKER или Azure Active Directory проблем входа. 

Примеры команд см. [в статье Проверка работоспособности реестра контейнеров Azure](container-registry-check-health.md) . Если выводятся сообщения об ошибках, см. рекомендации по [ссылке на ошибку](container-registry-health-error-reference.md) и следующим разделам.

Если при работе с реестром вих Azure Kubernetes возникли проблемы, выполните команду [AZ AKS Check-контроля](/cli/azure/aks#az_aks_check_acr) доступа, чтобы убедиться, что реестр доступен из кластера AKS.

> [!NOTE]
> Некоторые ошибки проверки подлинности или авторизации также могут возникать при наличии брандмауэров или конфигураций сети, препятствующих доступу к реестру. См. [раздел Устранение неполадок в сети с помощью реестра](container-registry-troubleshoot-access.md).

## <a name="potential-solutions"></a>Возможные решения

### <a name="check-docker-configuration"></a>Проверка конфигурации DOCKER

Для большинства потоков проверки подлинности реестра контейнеров Azure требуется локальная установка DOCKER, поэтому вы можете пройти проверку подлинности в реестре для таких операций, как отправка и извлечение образов. Убедитесь, что клиент и управляющая программа DOCKER CLI (подсистема DOCKER) работают в вашей среде. Требуется клиент DOCKER версии 18,03 или более поздней.

Связанные ссылки

* [Общие сведения об аутентификации](container-registry-authentication.md#authentication-options)
* [Реестр контейнеров: вопросы и ответы](container-registry-faq.md)

### <a name="specify-correct-registry-name"></a>Укажите правильное имя реестра

При использовании `docker login` Укажите полное имя сервера входа в реестр, например *myregistry.azurecr.IO*. Убедитесь, что используются только строчные буквы. Пример

```console
docker login myregistry.azurecr.io
```

При использовании команды [AZ контроля учетных записей](/cli/azure/acr#az-acr-login) с удостоверением Azure Active Directory сначала [Войдите в Azure CLI](/cli/azure/authenticate-azure-cli), а затем укажите имя ресурса Azure реестра. Имя ресурса — это имя, указанное при создании реестра, например *myregistry* (без суффикса домена). Пример

```azurecli
az acr login --name myregistry
```

Связанные ссылки

* [Команда az acr login выполняется успешно, но команды docker завершаются ошибкой: "не авторизовано: требуется проверка подлинности"](container-registry-faq.md#az-acr-login-succeeds-but-docker-fails-with-error-unauthorized-authentication-required )

### <a name="confirm-credentials-to-access-registry"></a>Подтверждение учетных данных для доступа к реестру

Проверьте правильность учетных данных, используемых для сценария или предоставленных владельцем реестра. Некоторые возможные проблемы:

* При использовании субъекта-службы Active Directory убедитесь, что в клиенте Active Directory используются правильные учетные данные:
  * Имя пользователя — идентификатор приложения субъекта-службы (также называемый *идентификатором клиента*)
  * Пароль — пароль субъекта-службы (также называемый *секретом клиента*)
* Если для доступа к реестру используется служба Azure, например Azure Kubernetes Service или Azure DevOps, проверьте конфигурацию реестра для службы. 
* При запуске `az acr login` с `--expose-token` параметром, который позволяет войти в реестр без использования управляющей программы DOCKER, убедитесь, что вы выполнили проверку подлинности с помощью имени пользователя `00000000-0000-0000-0000-000000000000` .
* Если реестр настроен для [анонимного доступа по запросу](container-registry-faq.md#how-do-i-enable-anonymous-pull-access), существующие учетные данные DOCKER, хранящиеся в предыдущем имени входа DOCKER, могут препятствовать анонимному доступу. Выполните, `docker logout` прежде чем пытаться выполнить анонимную операцию извлечения реестра.

Связанные ссылки

* [Общие сведения об аутентификации](container-registry-authentication.md#authentication-options)
* [Отдельный вход с помощью Azure AD](container-registry-authentication.md#individual-login-with-azure-ad)
* [Вход с использованием субъекта-службы](container-registry-auth-service-principal.md)
* [Вход с управляемым удостоверением](container-registry-authentication-managed-identity.md)
* [Вход с использованием маркера области репозитория](container-registry-repository-scoped-permissions.md)
* [Вход с учетной записью администратора](container-registry-authentication.md#admin-account)
* [Коды ошибок проверки подлинности и авторизации Azure AD](../active-directory/develop/reference-aadsts-error-codes.md)
* [AZ запись контроля](/cli/azure/acr#az-acr-login) доступа, Справочник

### <a name="confirm-credentials-are-authorized-to-access-registry"></a>Подтвердите, что учетные данные имеют права доступа к реестру.

Подтвердите разрешения реестра, связанные с учетными данными, например `AcrPull` роль Azure для извлечения образов из реестра или `AcrPush` роль для push-образов. 

Для доступа к реестру на портале или управлении реестром с помощью Azure CLI требуется по крайней мере `Reader` роль или аналогичные разрешения для выполнения операций Azure Resource Manager.

Если ваши разрешения недавно были изменены, чтобы разрешить доступ к реестру на портале, вам может потребоваться использовать режиме инкогнито или частный сеанс в браузере, чтобы избежать использования устаревшего кэша браузера или файлов cookie.

Вы или владелец реестра должны иметь достаточные привилегии в подписке для добавления или удаления назначений ролей.

Связанные ссылки

* [Роли и разрешения Azure. Реестр контейнеров Azure](container-registry-roles.md)
* [Вход с использованием маркера области репозитория](container-registry-repository-scoped-permissions.md)
* [Добавление и удаление назначений ролей Azure с помощью портала Azure](../role-based-access-control/role-assignments-portal.md)
* [Создание приложения Azure AD и субъекта-службы, который может получить доступ к ресурсам, с помощью портала](../active-directory/develop/howto-create-service-principal-portal.md)
* [Создание секрета приложения](../active-directory/develop/howto-create-service-principal-portal.md#option-2-create-a-new-application-secret)
* [Коды проверки подлинности и авторизации Azure AD](../active-directory/develop/reference-aadsts-error-codes.md)

### <a name="check-that-credentials-arent-expired"></a>Убедитесь, что срок действия учетных данных не истек

Срок действия маркеров и Active Directory учетных данных может истечь через определенные периоды, предотвращая доступ к реестру. Чтобы обеспечить доступ, может потребоваться сброс или повторное создание учетных данных.

* Если используется индивидуальное удостоверение AD, управляемое удостоверение или субъект-служба для входа в реестр, срок действия маркера AD истекает через 3 часа. Снова войдите в реестр.  
* Если используется субъект-служба AD с секретом клиента с истекшим сроком действия, владелец или администратор учетной записи должен сбросить учетные данные или создать новый субъект-службу.
* При использовании [маркера уровня репозитория](container-registry-repository-scoped-permissions.md)владельцу реестра может потребоваться сбросить пароль или создать новый маркер.

Связанные ссылки

* [Сброс учетных данных субъекта-службы](/cli/azure/ad/sp/credential#az-ad-sp-credential-reset)
* [Повторное создание паролей токенов](container-registry-repository-scoped-permissions.md#regenerate-token-passwords)
* [Отдельный вход с помощью Azure AD](container-registry-authentication.md#individual-login-with-azure-ad)

## <a name="advanced-troubleshooting"></a>Расширенный поиск проблем

Если в реестре включен [Сбор журналов ресурсов](container-registry-diagnostics-audit-logs.md) , проверьте журнал контаинтеррегистрилогиневентс. В этом журнале хранятся события и состояние проверки подлинности, включая входящий идентификатор и IP-адрес. Запросите журнал о [сбоях проверки подлинности в реестре](container-registry-diagnostics-audit-logs.md#registry-authentication-failures). 

Связанные ссылки

* [Журналы для диагностической оценки и аудита](container-registry-diagnostics-audit-logs.md)
* [Реестр контейнеров: вопросы и ответы](container-registry-faq.md)
* [Рекомендации по использованию реестра контейнеров Azure](container-registry-best-practices.md)

## <a name="next-steps"></a>Дальнейшие действия

Если вы не решите проблему здесь, см. следующие параметры.

* Другие разделы по устранению неполадок в реестре:
  * [Устранение проблем с сетью с помощью реестра](container-registry-troubleshoot-access.md)
  * [Устранение проблем с производительностью реестра](container-registry-troubleshoot-performance.md)
* Варианты [поддержки сообщества](https://azure.microsoft.com/support/community/)
* [Майкрософт: вопросы и ответы](/answers/products/)
* [Откройте запрос в службу поддержки](https://azure.microsoft.com/support/create-ticket/) . на основе предоставленных сведений можно выполнить быструю диагностику ошибок проверки подлинности в реестре.