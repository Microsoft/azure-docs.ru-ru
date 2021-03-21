---
title: Включение службы профилей пользователей SharePoint с помощью AD DS Azure | Документация Майкрософт
description: Узнайте, как настроить управляемый домен доменных служб Azure Active Directory для поддержки синхронизации профилей для SharePoint Server.
services: active-directory-ds
author: justinha
manager: daveba
ms.assetid: 938a5fbc-2dd1-4759-bcce-628a6e19ab9d
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 07/09/2020
ms.author: justinha
ms.openlocfilehash: 8fbde10ccf5a7f083f5fbddaadd6668d880a1fac
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96619833"
---
# <a name="configure-azure-active-directory-domain-services-to-support-user-profile-synchronization-for-sharepoint-server"></a>Настройка доменных служб Azure Active Directory для поддержки синхронизации профилей пользователей для SharePoint Server

Сервер SharePoint содержит службу для синхронизации профилей пользователей. Эта функция позволяет хранить профили пользователей в центральном расположении и быть доступными на нескольких сайтах и фермах SharePoint. Чтобы настроить службу профилей пользователей SharePoint Server, необходимо предоставить соответствующие разрешения в управляемом домене доменных служб Azure Active Directory (Azure AD DS). Дополнительные сведения см. [в статье Синхронизация профилей пользователей в SharePoint Server](/SharePoint/administration/user-profile-service-administration).

В этой статье показано, как настроить AD DS Azure, чтобы разрешить службу синхронизации профилей пользователей SharePoint Server.

## <a name="before-you-begin"></a>Перед началом

Для работы с этой статьей требуются следующие ресурсы и разрешения:

* Активная подписка Azure.
    * Если у вас еще нет подписки Azure, [создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Связанный с вашей подпиской клиент Azure Active Directory, синхронизированный с локальным или облачным каталогом.
    * Если потребуется, [создайте клиент Azure Active Directory][create-azure-ad-tenant] или [свяжите подписку Azure со своей учетной записью][associate-azure-ad-tenant].
* Управляемый домен доменных служб Azure Active Directory, включенный и настроенный в клиенте Azure AD.
    * При необходимости выполните инструкции из руководства по [созданию и настройке управляемого домена доменных служб Azure Active Directory][create-azure-ad-ds-instance].
* Виртуальная машина управления Windows Server, присоединенная к управляемому домену AD DS Azure.
    * При необходимости выполните инструкции из руководства по [созданию виртуальной машины управления][tutorial-create-management-vm].
* Учетная запись пользователя, входящая в группу *администраторов Azure AD DC* в клиенте Azure AD.
* Учетная запись службы SharePoint для службы синхронизации профилей пользователей.
    * При необходимости см. раздел [Планирование административных и служебных учетных записей в SharePoint Server][sharepoint-service-account].

## <a name="service-accounts-overview"></a>Общие сведения об учетных записях служб

В управляемом домене группа безопасности с именем *учетные записи службы AAD DC* существует как часть подразделения *пользователей* . Участникам этой группы безопасности делегируются следующие привилегии доступа:

- Привилегия **репликации изменений каталога** в корневом DSE.
- Привилегия **репликации изменений каталога** в контексте именования *конфигурации* ( `cn=configuration` контейнера).

Группа безопасности *учетных записей службы контроллера домена AAD* также является членом встроенной группы, которая является *предварительно совместимой с Windows 2000 доступом*.

При добавлении в эту группу безопасности учетной записи службы для службы синхронизации профилей пользователей SharePoint Server предоставляются необходимые привилегии для правильной работы.

## <a name="enable-support-for-sharepoint-server-user-profile-sync"></a>Включить поддержку для синхронизации профилей пользователей SharePoint Server

Учетной записи службы для SharePoint Server требуются достаточные права для репликации изменений в каталог и обеспечения правильной работы синхронизации профилей пользователей SharePoint Server. Чтобы предоставить эти привилегии, добавьте учетную запись службы, используемую для синхронизации профилей пользователей SharePoint, в группу *учетных записей службы контроллера домена AAD* .

На виртуальной машине Azure AD DS Management выполните следующие действия.

> [!NOTE]
> Чтобы изменить членство в группе в управляемом домене, необходимо войти в учетную запись пользователя, которая является членом группы *администраторов контроллера домена AAD* .

1. На начальном экране выберите **Администрирование**. Отобразится список доступных средств управления, которые были установлены в руководстве по [созданию виртуальной машины управления][tutorial-create-management-vm].
1. Чтобы управлять членством в группе, выберите **центр администрирования Active Directory** в списке средств администрирования.
1. В левой области выберите управляемый домен, например *aaddscontoso.com*. Отобразится список имеющихся подразделений и ресурсов.
1. Выберите подразделение **Пользователи** , а затем выберите группу безопасности *учетные записи службы контроллера домена AAD* .
1. Выберите **элементы**, а затем нажмите кнопку **Добавить...**.
1. Введите имя учетной записи службы SharePoint, а затем нажмите кнопку **ОК**. В следующем примере учетной записи службы SharePoint присваивается имя *Admin*:

    ![Добавление учетной записи службы SharePoint в группу безопасности учетных записей службы контроллера домена AAD](./media/deploy-sp-profile-sync/add-member-to-aad-dc-service-accounts-group.png)

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. [в статье Управление синхронизацией профилей пользователей в SharePoint Server](/SharePoint/administration/manage-profile-synchronization).

<!-- INTERNAL LINKS -->
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md
[tutorial-create-management-vm]: tutorial-create-management-vm.md

<!-- EXTERNAL LINKS -->
[sharepoint-service-account]: /sharepoint/security-for-sharepoint-server/plan-for-administrative-and-service-accounts