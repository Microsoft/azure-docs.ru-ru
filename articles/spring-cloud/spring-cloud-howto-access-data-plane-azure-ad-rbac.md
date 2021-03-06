---
title: Доступ к плоскости данных с помощью Azure AD, RBAC
description: Как получить доступ к плоскости данных с помощью управления доступом на основе ролей Azure Active Directory.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: how-to
ms.date: 02/04/2021
ms.custom: devx-track-java
ms.openlocfilehash: 2909d40fbc7cb5b310ea17f17a6e683ce47638ce
ms.sourcegitcommit: f7eda3db606407f94c6dc6c3316e0651ee5ca37c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102220138"
---
# <a name="access-the-data-plane-with-azure-active-directory-and-role-based-access-control"></a>Доступ к плоскости данных с помощью Azure Active Directory и управления доступом на основе ролей

В этой статье объясняется, как получить доступ к конечным точкам сервера настройки Azure весны в облаке и реестру служб с помощью управления доступом на основе ролей (Azure Active Directory) (AAD).

## <a name="assign-role-to-aad-usergroup-msi-or-service-principal"></a>Назначение роли пользователю или группе AAD, MSI или субъекту-службе

Чтобы использовать AAD и RBAC, необходимо назначить роль *читателя облачных данных Azure весны* для пользователя, группы или субъекта-службы с помощью следующей процедуры:

1. Перейдите на страницу обзора службы экземпляра службы.

2. Щелкните **Управление доступом (IAM)** , чтобы открыть колонку "Управление доступом".

3. Нажмите кнопку " **Добавить** " и **добавьте назначения ролей** (для добавления может потребоваться авторизация).

4. Найдите и выберите *Azure Весна облачного модуля чтения данных* в разделе **роль**.
5. Назначьте доступ `User, group, or service principal` или в `User assigned managed identity` соответствии с типом пользователя. Найдите и выберите пользователь.  
6. Щелкните `Save`.

   ![назначить роль](media/access-data-plane-aad-rbac/assign-data-reader-role.png)

## <a name="access-data-plane"></a>Доступ к плоскости данных

После назначения пользователю AAD роли *читателя облачных данных Azure весны* клиенты могут войти в Azure CLI с учетной записью пользователя, субъекта-службы или управляемого удостоверения.  Чтобы получить маркер доступа, см. раздел [Проверка Подлинности Azure CLI](https://docs.microsoft.com/cli/azure/authenticate-azure-cli) .  Затем используйте следующие команды.

```azurecli
az login
az account get-access-token
```

В настоящее время CLI поддерживает конечные точки по умолчанию сервера конфигурации и реестра служб. Дополнительные сведения см. в разделе [конечные точки, готовые для рабочей среды](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready-endpoints). 

Клиенты также могут получить полный список поддерживаемых конечных точек сервера конфигурации и реестра служб путем доступа к конечным точкам.
* *https://SERVICE_NAME.Root_Endpoint/eureka/actuator/*
* *https://SERVICE_NAME.Root_Endpoint/config/actuator/* 

Токен доступа в заголовке предоставляет авторизацию. Поддерживается только метод GET.

Например, получите доступ к конечной точке, например, *https://SERVICE_NAME.Root_Endpoint/eureka/actuator/health* чтобы просмотреть состояние работоспособности Еурека.

Ниже показаны различные корневые конечные точки в соответствии с различными типами облаков.

| Cloud          | Корневая конечная точка              |
| -------------- | -------------------------- |
| Общие         | svc.azuremicroservices.io  |
| Mooncake/Китай | svc.microservices.azure.cn |

Если ответ не *прошел проверку* подлинности 401, проверьте, была ли роль назначена.  Для вступления роли в силу или проверки того, что срок действия маркера доступа не истек, займет несколько минут.

## <a name="next-steps"></a>Дальнейшие действия
* [Проверка подлинности Azure CLI](https://docs.microsoft.com/cli/azure/authenticate-azure-cli)
* [Конечные точки, готовые для производства](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready-endpoints)

## <a name="see-also"></a>См. также раздел
* [Создание ролей и разрешений](spring-cloud-howto-permissions.md)