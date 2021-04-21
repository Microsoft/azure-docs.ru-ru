---
title: 'Azure AD Connect: устранение неполадок с синхронизацией объектов | Документация Майкрософт'
description: В этой статье приводятся пошаговые инструкции по устранению неполадок, связанных с синхронизацией объектов, с помощью задач устранения неполадок.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: curtand
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 04/29/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 73d4239dd34f2a64aa7b3edbf88bad4348e01291
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "85356208"
---
# <a name="troubleshoot-object-synchronization-with-azure-ad-connect-sync"></a>Устранение неполадок синхронизации объектов с помощью службы синхронизации Azure AD Connect
В этой статье приводятся пошаговые инструкции по устранению неполадок, связанных с синхронизацией объектов, с помощью задач устранения неполадок. Ознакомиться с устранением неполадок в Azure Active Directory Connect (Azure AD) можно [в этом коротком видео](https://aka.ms/AADCTSVideo).

## <a name="troubleshooting-task"></a>Задачи по устранению неполадок
Чтобы устранить неполадки с синхронизацией объектов в развертывании Azure AD Connect версии 1.1.749.0 и выше, используйте задачу устранения неполадок в мастере. Если вы используете более ранние версии, выполните устранение неполадок вручную, как описано [здесь](tshoot-connect-object-not-syncing.md).

### <a name="run-the-troubleshooting-task-in-the-wizard"></a>Запуск задачи устранения неполадок в мастере
Чтобы запустить задачу устранения неполадок в мастере, выполните следующие действия:

1.  Откройте новый сеанс Windows PowerShell на сервере Azure AD Connect и запустите его от имени администратора.
2.  Запустите `Set-ExecutionPolicy RemoteSigned` или `Set-ExecutionPolicy Unrestricted`.
3.  Откройте мастер Azure AD Connect.
4.  Перейдите к странице "Дополнительные задачи", выберите "Устранение неполадок" и щелкните "Далее".
5.  На странице "Устранение неполадок" щелкните "Запуск", чтобы открыть меню устранения неполадок в PowerShell.
6.  В главном меню выберите Troubleshoot Object Synchronization (Устранение неполадок с синхронизацией объектов).
![Устранение неполадок синхронизации объектов](media/tshoot-connect-objectsync/objsynch11.png)

### <a name="troubleshooting-input-parameters"></a>Входные параметры для устранения неполадок
Следующие входные параметры необходимы для выполнения задачи по устранению неполадок.
1.  **Object Distinguished Name** (Различающееся имя объекта) — различающееся имя объекта, неполадки которого требуется устранить.
2.  **AD Connector Name** (Имя соединителя AD) — имя леса AD, где находится указанный выше объект.
3.  Учетные данные глобального администратора клиента Azure AD ![учетные данные глобального администратора](media/tshoot-connect-objectsync/objsynch1.png)

### <a name="understand-the-results-of-the-troubleshooting-task"></a>Изучение результатов задачи устранения неполадок
Задача устранения неполадок выполняет следующие проверки.

1.  Выявляет несоответствия в имени участника-пользователя, если объект синхронизирован с Azure Active Directory.
2.  Проверяет, фильтруется ли объект при фильтрации домена.
3.  Проверяет, фильтруется ли объект при фильтрации подразделения.
4.  Проверьте, не блокируется ли синхронизация объектов из-за связанного почтового ящика.
5. Проверьте, не является ли объект динамической группой рассылки, которая не должна синхронизироваться.

В оставшейся части этого раздела описаны возможные результаты, возвращаемые задачей. В каждом случае задача предоставляет анализ и рекомендуемые действия для устранения проблемы.

## <a name="detect-upn-mismatch-if-object-is-synced-to-azure-active-directory"></a>Выявление несоответствия в имени участника-пользователя при синхронизации объекта с Azure Active Directory
### <a name="upn-suffix-is-not-verified-with-azure-ad-tenant"></a>Суффикс имени участника-пользователя НЕ проверяется клиентом Azure AD
Если имя субъекта-пользователя (UserPrincipalName) и суффикс альтернативного имени пользователя не проверены клиентом Azure AD Tenant, тогда Azure Active Directory заменяет суффиксы имени участника-пользователя на стандартное доменное имя onmicrosoft.com.

![Azure AD заменяет UPN](media/tshoot-connect-objectsync/objsynch2.png)

### <a name="azure-ad-tenant-dirsync-feature-synchronizeupnformanagedusers-is-disabled"></a>Компонент DirSync клиента Azure AD SynchronizeUpnForManagedUsers отключен
Если компонент DirSync клиента Azure AD SynchronizeUpnForManagedUsers отключен, Azure Active Directory не позволяет выполнить обновление синхронизации атрибута UserPrincipalName и альтернативного имени пользователя для учетных записей лицензированных пользователей с помощью управляемой аутентификации.

![SynchronizeUpnForManagedUsers](media/tshoot-connect-objectsync/objsynch4.png)

## <a name="object-is-filtered-due-to-domain-filtering"></a>Фильтрование объекта при фильтрации домена
### <a name="domain-is-not-configured-to-sync"></a>Домен не настроен для синхронизации
Объект находится за пределами области, так как домен не настроен. В указанном ниже примере объект находится за пределами области, так как домен, к которому он принадлежит, фильтруется после синхронизации.

![Домен не настроен для синхронизации](media/tshoot-connect-objectsync/objsynch5.png)

### <a name="domain-is-configured-to-sync-but-is-missing-run-profilesrun-steps"></a>Домен настроен для синхронизации, но отсутствуют профили и шаги выполнения
Объект находится за пределами области, так как в домене отсутствуют профили и шаги выполнения. В указанном ниже примере объект находится за пределами области, так как в домене, к которому он принадлежит, отсутствуют шаги выполнения для профиля выполнения полного импорта.
![отсутствующие профили выполнения](media/tshoot-connect-objectsync/objsynch6.png)

## <a name="object-is-filtered-due-to-ou-filtering"></a>Фильтрование объекта при фильтрации подразделения
Объект находится за пределами области из-за настройки фильтрации подразделений. В следующем примере объект принадлежит к подразделению OU=NoSync,DC=bvtadwbackdc,DC=com.  Это подразделение не включено в область синхронизации.</br>

![OU](./media/tshoot-connect-objectsync/objsynch7.png)

## <a name="linked-mailbox-issue"></a>Проблема из-за связанного почтового ящика
Связанный почтовый ящик должен быть привязан к внешней главной учетной записи, расположенной в другом доверенному лесу учетных записей. Если внешняя главная учетная запись отсутствует, то Azure AD Connect не будет синхронизировать учетную запись пользователя, которая соответствует связанному почтовому ящику в лесу Exchange, с клиентом Azure AD.</br>
![Связанный почтовый ящик](./media/tshoot-connect-objectsync/objsynch12.png)

## <a name="dynamic-distribution-group-issue"></a>Проблема из-за динамической группы рассылки
Из-за различий между локальной средой Active Directory и Azure Active Directory служба Azure AD Connect не синхронизирует динамические группы рассылки с клиентом Azure AD.

![Динамическая группа рассылки](./media/tshoot-connect-objectsync/objsynch13.png)

## <a name="html-report"></a>Отчет HTML
Помимо выполнения анализа объекта задача устранения неполадок также создает отчет HTML со всеми сведениями об объекте. Этот отчет HTML можно использовать совместно с группой поддержки для дальнейшего устранения неполадок, если потребуется.

![Отчет HTML](media/tshoot-connect-objectsync/objsynch8.png)

## <a name="next-steps"></a>Дальнейшие действия
Узнайте больше об [интеграции локальных удостоверений с Azure Active Directory](whatis-hybrid-identity.md).
