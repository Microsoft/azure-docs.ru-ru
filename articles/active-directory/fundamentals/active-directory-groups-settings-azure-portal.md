---
title: Изменение сведений о группе в Azure Active Directory | Документация Майкрософт
description: Инструкции по изменению сведений о группе с помощью Azure Active Directory.
services: active-directory
author: ajburnle
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: how-to
ms.date: 08/27/2018
ms.author: ajburnle
ms.reviewer: krbain
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: c346877d29c5fef03b45c895cbf8d462f90d0979
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92369137"
---
# <a name="edit-your-group-information-using-azure-active-directory"></a>Изменение сведений о группе с помощью Azure Active Directory

С помощью Azure Active Directory (Azure AD) можно изменить параметры группы, в том числе имя, описание или тип членства.

## <a name="to-edit-your-group-settings"></a>Изменение параметров группы
1. Войдите на [портал Azure](https://portal.azure.com) с учетной записью глобального администратора каталога.

2. Выберите **Azure Active Directory**, а затем щелкните **Группы**.

    Откроется страница **Группы — Все группы** со списком активных групп.

3. На странице **Группы — Все группы** введите максимальную часть имени группы в поле **Поиск**. В этой статье мы будем искать группу **Политика управления мобильными устройствами — Запад**.

    Результаты поиска будут отображаться под полем **поиска** по мере ввода.

    ![Страница "Все группы" с заполненным полем поиска](media/active-directory-groups-settings-azure-portal/search-for-specific-group.png)

4. Выберите группу **Политика управления мобильными устройствами — Запад**, а затем выберите **Свойства** в разделе **Управление**.

    ![Страница "Обзор группы" с выделенным параметром-членом и сведениями](media/active-directory-groups-settings-azure-portal/group-overview-blade.png)

5. Измените **Общие параметры** по необходимости, например:

    ![Параметры свойств для группы](media/active-directory-groups-settings-azure-portal/group-properties-settings.png)

    - **Имя группы.** Измените существующее имя группы.
    
    - **Описание группы**. Измените существующее описание группы.

    - **Тип группы.** Тип группы нельзя изменить после создания. Чтобы изменить **тип группы**, необходимо удалить группу и создать новую.
    
    - **Тип членства.** Измените тип членства. Дополнительные сведения о различных доступных типах членства см. в статьях [Создание базовой группы и добавление членов с помощью портала Azure Active Directory](active-directory-groups-create-azure-portal.md).
    
    - **Идентификатор объекта.** Невозможно изменить идентификатор объекта, но можно скопировать его и использовать в командах PowerShell для группы. Дополнительные сведения об использовании командлетов PowerShell см. в статье [Настройка параметров групп с помощью командлетов Azure Active Directory](../enterprise-users/groups-settings-v2-cmdlets.md).

## <a name="next-steps"></a>Дальнейшие действия
В следующих статьях содержатся дополнительные сведения об Azure Active Directory.

- [Просмотр групп и участников](active-directory-groups-view-azure-portal.md)

- [Создание простой группы и добавление участников](active-directory-groups-create-azure-portal.md)

- [Добавление или удаление участников из группы](active-directory-groups-members-azure-portal.md)

- [Управление динамическими правилами для пользователей в группе](../enterprise-users/groups-create-rule.md)

- [Управление членством в группе](active-directory-groups-membership-azure-portal.md)

- [Управление доступом к ресурсам с помощью групп](active-directory-manage-groups.md)

- [Связывание подписки Azure с Azure Active Directory или добавление ее в службу](active-directory-how-subscriptions-associated-directory.md)
