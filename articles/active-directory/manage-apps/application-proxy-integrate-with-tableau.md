---
title: Azure Active Directory Application Proxy и Tableau | Документация Майкрософт
description: Сведения об использовании Azure Active Directory Application Proxy (Azure AD) для предоставления удаленного доступа к развертыванию Tableau.
services: active-directory
author: kenwith
manager: daveba
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 08/20/2018
ms.author: kenwith
ms.reviewer: japere
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6720a5ad963bc73e11ef7b46150e946521928c01
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99258631"
---
# <a name="azure-active-directory-application-proxy-and-tableau"></a>Azure Active Directory Application Proxy и Tableau 

Использование Azure Active Directory Application Proxy и Tableau упрощает использование прокси приложения для предоставления удаленного доступа к развертыванию Tableau. В этой статье описывается настройка сценария.  

## <a name="prerequisites"></a>Предварительные условия 

Сценарий в этой статье предполагает, что:

- [Tableau](https://onlinehelp.tableau.com/current/server/en-us/proxy.htm#azure) настроен; 

- [соединитель прокси приложения](application-proxy-add-on-premises-application.md) установлен. 

 
## <a name="enabling-application-proxy-for-tableau"></a>Включение прокси приложения для Tableau 

Прокси приложения поддерживает поток предоставления OAuth 2.0, который необходим для корректной работы Tableau. Это означает, что больше не требуется каких-либо особых действий для включения этого приложения. Чтобы это сделать, настройте приложение в соответствии с инструкциями, приведенными далее.


## <a name="publish-your-applications-in-azure"></a>Публикация приложений в Azure 

Чтобы опубликовать Tableau, вам необходимо опубликовать приложение на портале Azure.

Для:

- Подробные пошаговые инструкции по шагам 1–8 см. в статье [Публикация приложений с помощью прокси приложения Azure AD](application-proxy-add-on-premises-application.md). 
- Сведения о том, как найти значения Tableau для полей App Proxy, см. в документации по Tableau.  

**Чтобы опубликовать приложение, сделайте следующее**. 


1. Войдите в [портал Azure](https://portal.azure.com) в качестве администратора приложения. 

2. Выберите **Azure Active Directory > Корпоративные приложения**. 

3. Щелкните **Добавить** в верхней части колонки. 

4. Выберите **Локальное приложение**. 

5. Введите в обязательные поля сведения о новом приложении. Вам нужно настроить следующие параметры. 

    - **Внутренний URL-адрес**: это приложение должно использовать внутренний URL-адрес, которым является сам URL-адрес Tableau. Например, `https://adventure-works.tableau.com`. 

    - **Метод предварительной аутентификации**: Azure Active Directory (рекомендуется, но не обязательно). 

6. Щелкните **Добавить** в верхней части колонки. Когда приложение будет добавлено, откроется меню быстрого запуска. 

7. В меню быстрого запуска выберите **Назначить пользователя для тестирования**, а затем назначьте для приложения хотя бы одного пользователя. Убедитесь, что у тестовой учетной записи есть доступ к локальному приложению. 

8. Щелкните **Назначить**, чтобы сохранить назначение тестового пользователя. 

9. (Необязательно.) На странице управления приложением щелкните **Единый вход**. Выберите **Встроенная проверка подлинности Windows** в раскрывающемся меню и заполните обязательные поля на основе конфигурации Tableau. Щелкните **Сохранить**. 

 

## <a name="testing"></a>Тестирование 

Теперь приложение готово к тестированию. Перейдите по внешнему URL-адресу, который использовался для публикации Tableau, и войдите в систему как пользователь, назначенный для обоих приложений.



## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о прокси приложения Azure AD см. в статье [Как обеспечить безопасный удаленный доступ к локальным приложениям](application-proxy.md).

