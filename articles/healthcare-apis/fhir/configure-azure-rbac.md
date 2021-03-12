---
title: Настройка управления доступом на основе ролей Azure (Azure RBAC) для Azure API для FHIR
description: В этой статье описывается, как настроить Azure RBAC для плоскости данных Azure API for FHIR.
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: reference
ms.date: 03/15/2020
ms.author: matjazl
ms.reviewer: dseven
ms.openlocfilehash: fb57b689b77ec4cc6205c8885c2a2e062c469efb
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103019719"
---
# <a name="configure-azure-rbac-for-fhir"></a>Настройка Azure RBAC для FHIR 

В этой статье вы узнаете, как использовать [Управление доступом на основе ролей Azure (Azure RBAC)](../../role-based-access-control/index.yml) для назначения доступа к плоскости данных Azure API for FHIR. Azure RBAC — это предпочтительные методы назначения доступа к плоскости данных при управлении пользователями плоскости данных в Azure Active Directoryном клиенте, связанном с подпиской Azure. Если вы используете внешний клиент Azure Active Directory, см. [ссылку на локальное назначение RBAC](configure-local-rbac.md).

## <a name="confirm-azure-rbac-mode"></a>Подтверждение режима RBAC в Azure

Чтобы использовать Azure RBAC, необходимо настроить API Azure для FHIR на использование клиента подписки Azure для плоскости данных и не указывать идентификаторы объектов удостоверений. Чтобы проверить параметры, просмотрите колонку **Проверка подлинности** в API Azure для FHIR:

:::image type="content" source="media/rbac/confirm-azure-rbac-mode.png" alt-text="Подтверждение режима RBAC в Azure":::

Для **центра** должен быть задан клиент Azure Active Directory, связанный с подпиской, и в поле " **допустимые идентификаторы объектов**" не должно быть GUID. Вы также увидите, что флажок отключен и метка указывает, что для назначения ролей плоскости данных следует использовать Azure RBAC.

## <a name="assign-roles"></a>Назначение ролей

Чтобы предоставить пользователям, субъектам-службам или группам доступ к плоскости данных FHIR, щелкните **Управление доступом (IAM)**, а затем выберите **назначения ролей** и щелкните **+ добавить**:

:::image type="content" source="media/rbac/add-azure-rbac-role-assignment.png" alt-text="Добавление назначения роли Azure":::

В области Выбор **роли** найдите одну из встроенных ролей для плоскости данных FHIR:

:::image type="content" source="media/rbac/built-in-fhir-data-roles.png" alt-text="Встроенные роли данных FHIR":::

Доступные параметры:

* FHIR Data Reader: может читать (и искать) данные FHIR.
* FHIR Data Writer: может читать, записывать и обратимо удалять данные FHIR.
* Средство экспорта данных FHIR: может читать и экспортировать ( `$export` оператор) данные.
* Участник данных FHIR: может выполнять все операции с плоскостью данных.

Если эти роли недостаточно для ваших нужд, можно также [создать пользовательские роли](../../role-based-access-control/tutorial-custom-role-powershell.md).

В поле **Выбор** найдите пользователя, субъекта-службы или группу, которой нужно назначить роль.

## <a name="caching-behavior"></a>Поведение кэширования

API Azure для FHIR будет кэшировать решения в течение 5 минут. Если вы предоставите пользователю доступ к серверу FHIR, добавив его в список разрешенных идентификаторов объектов или удалите их из списка, вы должны пройти до пяти минут на изменение разрешений для распространения.

## <a name="next-steps"></a>Дальнейшие действия

Из этой статьи вы узнали, как назначать роли Azure для плоскости данных FHIR. Дополнительные сведения о дополнительных параметрах для API Azure для FHIR:
 
>[!div class="nextstepaction"]
>[Дополнительные параметры для API Azure для FHIR](azure-api-for-fhir-additional-settings.md)