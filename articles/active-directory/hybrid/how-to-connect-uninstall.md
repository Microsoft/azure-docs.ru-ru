---
title: Удаление Azure AD Connect
description: В этом документе описано, как удалить Azure AD Connect.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 12/09/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: f5eb537a70c69745c8067ffb71cfb895a0875945
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96934189"
---
# <a name="uninstall-azure-ad-connect"></a>Удаление Azure AD Connect

В этом документе описано, как правильно удалить Azure AD Connect.

## <a name="uninstall-azure-ad-connect-from-the-server"></a>Удаление Azure AD Connect с сервера
Первое, что нужно сделать, это удалить Azure AD Connect с сервера, на котором она выполняется.  Выполните указанные ниже действия.

 1. На сервере, на котором работает Azure AD Connect, перейдите в **Панель управления**.
 2. Щелкните **удалить программу** 
  ![ Удаление программы](media/how-to-connect-uninstall/uninstall-1.png)</br>
 
 3. Выберите **Azure AD Connect**.
 ![Выберите Azure AD Connect](media/how-to-connect-uninstall/uninstall-2.png)</br>
 
 4. При появлении запроса нажмите кнопку **Да** для подтверждения.
 5. Это подтверждение отобразит экран Azure AD Connect.  Щелкните **Удалить**.
 ![Удалить](media/how-to-connect-uninstall/uninstall-3.png)</br>
 
 6. После завершения этого действия нажмите кнопку **выход**.
 7. ![Выход](media/how-to-connect-uninstall/uninstall-4.png)</br>
 
 8. В **панели управления** щелкните **Обновить** , и все компоненты должны быть удалены.


## <a name="next-steps"></a>Дальнейшие действия

- Узнайте больше об [интеграции локальных удостоверений с Azure Active Directory](whatis-hybrid-identity.md).
- [Установка Azure AD Connect с помощью имеющейся базы данных ADSync](how-to-connect-install-existing-database.md)
- [Установка Azure AD Connect с использованием делегированных разрешений администратора SQL](how-to-connect-install-sql-delegation.md)

