---
title: Скрытие корпоративного приложения от взаимодействия с пользователем в Azure AD
description: Сведения о том, как скрыть корпоративное приложение от взаимодействия с пользователем на панелях Azure Active Directory доступа или Microsoft 365.
services: active-directory
author: kenwith
manager: daveba
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 03/25/2020
ms.author: kenwith
ms.reviewer: kasimpso
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8469b48b92f3f9a645a0c05441e6c1943b02e16f
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99258886"
---
# <a name="hide-enterprise-applications-from-end-users-in-azure-active-directory"></a>Скрыть корпоративные приложения от конечных пользователей в Azure Active Directory

Инструкции по скрытию приложений с панели "MyApps" конечных пользователей или средства запуска Microsoft 365. Когда приложение скрыто, у пользователей по-прежнему есть к нему доступ. 

## <a name="prerequisites"></a>Предварительные условия

Для скрытия приложения на панели "MyApps" и Microsoft 365 средстве запуска требуются права администратора приложения.

Для скрытия всех Microsoft 365 приложений требуются права глобального администратора.


## <a name="hide-an-application-from-the-end-user"></a>Скрытие приложения от пользователя
Выполните следующие действия, чтобы скрыть приложение на панели MyApps и Microsoft 365 средства запуска приложений.

1.  Войдите на [портал Azure](https://portal.azure.com) как глобальный администратор каталога.
2.  Выберите **Azure Active Directory**.
3.  Выберите **Корпоративные приложения**. Откроется колонка **Корпоративные приложения — Все приложения**.
4.  В разделе **Тип приложения** выберите **Корпоративные приложения**, если этот параметр еще не выбран.
5.  Найдите приложение, которое нужно скрыть, а затем щелкните его.  Откроется колонка обзора приложения.
6.  Нажмите кнопку **Свойства**. 
7.  Для вопроса **Видно пользователям?** выберите **Нет**.
8.  Выберите команду **Сохранить**.

> [!NOTE]
> Эти инструкции относятся только к корпоративным приложениям.

## <a name="use-azure-ad-powershell-to-hide-an-application"></a>Использование Azure AD PowerShell для скрытия приложения

Чтобы скрыть приложение на панели "MyApps", можно вручную добавить тег Хидеапп к субъекту-службе для приложения. Выполните следующие команды [PowerShell AzureAD](/powershell/module/azuread/#service_principals) , чтобы задать для приложения значение **нет**. для свойства **Visible** . 

```PowerShell
Connect-AzureAD

$objectId = "<objectId>"
$servicePrincipal = Get-AzureADServicePrincipal -ObjectId $objectId
$tags = $servicePrincipal.tags
$tags += "HideApp"
Set-AzureADServicePrincipal -ObjectId $objectId -Tags $tags
```

## <a name="hide-microsoft-365-applications-from-the-myapps-panel"></a>Скрытие Microsoft 365 приложений на панели "MyApps"

Чтобы скрыть все Microsoft 365 приложения с панели "MyApps", выполните следующие действия. Приложения по-прежнему будут видны на портале Office 365.

1.  Войдите на [портал Azure](https://portal.azure.com) как глобальный администратор каталога.
2.  Выберите **Azure Active Directory**.
3.  Выберите **Пользователи**.
4.  Выберите **Параметры пользователя**.
5.  В разделе **корпоративных приложений** щелкните **Manage how end users launch and view their applications** (Управление запуском и просмотром приложений для пользователей).
6.  Для параметра **Пользователи могут видеть приложения Office 365 только на портале Office 365** выберите **Да**.
7.  Выберите команду **Сохранить**.

## <a name="next-steps"></a>Дальнейшие действия
* [Просмотр всех моих групп](../fundamentals/active-directory-groups-view-azure-portal.md)
* [Назначение корпоративному приложению пользователя или группы](assign-user-or-group-access-portal.md)
* [Удаление назначения пользователя или группы из корпоративного приложения](./assign-user-or-group-access-portal.md)
* [Изменение имени или логотипа корпоративного приложения](./add-application-portal-configure.md)
