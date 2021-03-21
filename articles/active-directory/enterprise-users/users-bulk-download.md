---
title: Скачайте список пользователей на портале Azure Active Directory | Документация Майкрософт
description: Вы также скачиваете записи пользователей в центре администрирования Azure в Azure Active Directory.
services: active-directory
author: curtand
ms.author: curtand
manager: daveba
ms.date: 01/04/2021
ms.topic: how-to
ms.service: active-directory
ms.subservice: enterprise-users
ms.workload: identity
ms.custom: it-pro
ms.reviewer: krbain
ms.collection: M365-identity-device-management
ms.openlocfilehash: 57e3a059a5dd846250ba162513ef118e084c4b87
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97861596"
---
# <a name="download-a-list-of-users-in-azure-active-directory-portal"></a>Скачивание списка пользователей на портале Azure Active Directory

Azure Active Directory (Azure AD) поддерживает операции многопользовательского импорта (создания).

## <a name="required-permissions"></a>Необходимые разрешения

Чтобы скачать список пользователей из Центра администрирования Azure AD, необходимо войти в систему с использованием учетной записи пользователя, которому в Azure AD назначена одна или несколько ролей администратора на уровне организации (минимальная необходимая роль — администратор). Роли "Приглашающий гостей" и "Разработчик приложений" не считаются ролями администратора.

## <a name="to-download-a-list-of-users"></a>Загрузка списка пользователей

1. [Войдите в организацию Azure AD](https://aad.portal.azure.com) с помощью учетной записи администратора пользователя в Организации.
2. Перейдите к разделу Azure Active Directory > "Пользователи". Затем выберите пользователей, которых нужно включить в список для скачивания. Для этого установите флажки в полях в столбце слева рядом с каждым пользователем. Примечание. Сейчас нельзя выбрать всех пользователей для экспорта. Каждого из них нужно выбирать отдельно.
3. В Azure AD выберите **Пользователи**  >  **скачать пользователей**.
4. На странице **скачать пользователей** выберите **начать** , чтобы получить CSV-файл со списком свойств профиля пользователя. При наличии ошибок можно загрузить и просмотреть файл результатов на странице Результаты массовой операции. Файл содержит причину каждой ошибки.

   ![Выберите, куда нужно загрузить список пользователей.](./media/users-bulk-download/bulk-download.png)

   В скачиваемом файле будет содержаться отфильтрованный список пользователей.

   В него включены следующие атрибуты пользователей:

   - userPrincipalName
   - displayName
   - surname
   - mail
   - givenName
   - objectId
   - userType
   - jobTitle
   - department
   - AccountEnabled
   - usageLocation
   - streetAddress
   - Состояние
   - country
   - physicalDeliveryOfficeName
   - city
   - postalCode
   - TelephoneNumber
   - mobile
   - authenticationAlternativePhoneNumber;
   - authenticationEmail;
   - alternateEmailAddress;
   - ageGroup
   - consentProvidedForMinor
   - legalAgeGroupClassification

## <a name="check-status"></a>Проверка состояния

Состояние ожидающих выполнения незавершенных запросов можно просмотреть на странице **результатов групповой операции** .

[![Проверьте состояние на странице результатов с массовыми операциями.](./media/users-bulk-download/bulk-center.png)](./media/users-bulk-download/bulk-center.png#lightbox)

## <a name="bulk-download-service-limits"></a>Ограничения службы пакетного скачивания

Каждое групповое действие по созданию списка пользователей может выполняться в течение одного часа. Это позволяет создавать и скачивать список по крайней мере 500 000 пользователей.

## <a name="next-steps"></a>Дальнейшие действия

- [Групповое добавление пользователей](users-bulk-add.md)
- [Массовое удаление пользователей](users-bulk-delete.md)
- [Массовое восстановление пользователей](users-bulk-restore.md)
