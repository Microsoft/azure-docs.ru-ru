---
title: 'Синхронизация Azure AD Connect: изменение пароля учетной записи AD DS | Документация Майкрософт'
description: В этом разделе описывается, как обновить Azure AD Connect после изменения пароля учетной записи AD DS.
services: active-directory
keywords: учетная запись AD DS, учетная запись Active Directory, пароль
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 76b19162-8b16-4960-9e22-bd64e6675ecc
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 07/12/2017
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4077146292db1266d5dbc51cc577f952b2bff191
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "85357517"
---
# <a name="changing-the-ad-ds-account-password"></a>Изменение пароля учетной записи AD DS
Учетная запись AD DS — это учетная запись пользователя, которая используется службой Azure AD Connect для взаимодействия с локальным каталогом Active Directory. В случае изменения пароля учетной записи AD DS необходимо также обновить пароль в службе синхронизации Azure AD Connect. В противном случае служба синхронизации больше не сможет правильно синхронизировать данные с локальным каталогом Active Directory. При этом будут возникать следующие ошибки:

* В Synchronization Service Manager любая операция импорта или экспорта с использованием локального каталога AD завершается ошибкой **no-start-credentials**.

* В средстве просмотра событий Windows журнал событий приложения содержит ошибку с **идентификатором события 6000** и сообщением об ошибке **Не удалось выполнить управляющий агент "contoso.com", так как учетные данные были недопустимыми**.


## <a name="how-to-update-the-synchronization-service-with-new-password-for-ad-ds-account"></a>Как обновить пароль в службе синхронизации в случае изменения пароля учетной записи AD DS
Чтобы обновить пароль в службе синхронизации, выполните следующие действия:

1. Запустите Synchronization Service Manager ("Запуск" → "Служба синхронизации").
</br>![Диспетчер службы синхронизации](./media/how-to-connect-sync-change-addsacct-pass/startmenu.png)  

2. Перейдите на вкладку **Соединители**.

3. Выберите **соединитель AD**, соответствующий учетной записи AD DS, для которой был изменен пароль.

4. В разделе **Действия** выберите **Свойства**.

5. Во всплывающем диалоговом окне выберите **Подключиться к лесу Active Directory**.

6. В текстовом поле **Пароль** введите новый пароль учетной записи AD DS.

7. Нажмите кнопку **ОК**, чтобы сохранить новый пароль и закрыть всплывающее диалоговое окно.

8. Перезапустите службу синхронизации Azure AD Connect в диспетчере управления службами Windows. Это гарантирует, что все ссылки на старый пароль будут удалены из кэша памяти.

## <a name="next-steps"></a>Дальнейшие действия
**Обзорные статьи**

* [Синхронизация Azure AD Connect: общие сведений о синхронизации и ее настройка](how-to-connect-sync-whatis.md)

* [Интеграция локальных удостоверений с Azure Active Directory](whatis-hybrid-identity.md)
