---
title: Общие сведения об агенте администратора Azure AD Connect (Azure AD Connect) | Документация Майкрософт
description: Сведения о средствах, используемых для синхронизации и мониторинга вашей локальной среды в Azure AD.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: overview
ms.date: 09/04/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: da4e1cfc68dff4ad2bc0552c6d35fe1230779306
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "91312964"
---
# <a name="what-is-the-azure-ad-connect-admin-agent"></a>Общие сведения об агенте администратора Azure AD Connect 
Агент администрирования Azure AD Connect — это новый компонент Azure Active Directory Connect, который можно установить на сервере Azure Active Directory Connect. Он используется для сбора из среды Active Directory конкретных данных, которые помогут специалисту службы поддержки Майкрософт устранить неполадки, когда вы открываете обращение в службу поддержки. 

>[!NOTE]
>По умолчанию агент администратора не установлен и не включен.  Вам следует установить этот агент для сбора данных, которые помогут решить обращения в службу поддержки.

При установке Агент администрирования Azure AD Connect ожидает выполнение определенных запросов данных от Azure Active Directory, получает запрошенные данные из среды синхронизации и отправляет их в Azure Active Directory, где они передаются специалисту службы поддержки Майкрософт. 

Информация, которую агент администрирования Azure AD Connect извлекает из вашей среды, нигде не сохраняется. Ее может видеть только специалист службы поддержки Майкрософт, который попытается исследовать и устранить проблемы, связанные с использованием Azure Active Directory Connect. Агент администрирования Azure AD Connect по умолчанию не устанавливается на сервере Azure AD Connect. 

## <a name="install-the-azure-ad-connect-administration-agent-on-the-azure-ad-connect-server"></a>Установка агента администрирования Azure AD Connect на сервере Azure AD Connect 

Предварительные требования:
1.    На сервере требуется установка Azure AD Connect
2.    На сервере требуется установка Azure AD Connect Health

![Агент администрирования](media/whatis-aadc-admin-agent/adminagent0.png)

Двоичные файлы агента администрирования Azure AD Connect размещаются на сервере Azure AD Connect. Чтобы установить этот агент, сделайте следующее:

1.    Откройте PowerShell с правами администратора.
2.    Перейдите к папке, в которой расположено приложение: cd C:\Program Files\Microsoft Azure Active Directory Connect\Tools
3.    Запустите ConfigureAdminAgent.ps1.

В ответ на приглашение введите учетные данные глобального администратора AAD. Это должны быть те же учетные данные, которые вы указали во время установки Azure AD Connect.

После установки агента вы увидите две новые программы в списке "Установка и удаление программ" раздела "Панель управления" на сервере. 

![Снимок экрана: список "Установка и удаление программ" с добавленными вами программами](media/whatis-aadc-admin-agent/adminagent1.png)

## <a name="what-data-in-my-sync-service-is-shown-to-the-microsoft-service-engineer"></a>Какие данные служба синхронизации передает специалисту службы поддержки Майкрософт? 
После открытия обращения в службу поддержки Майкрософт ее специалист может видеть данные пользователя в Active Directory, пространстве соединителя Active Directory на сервере Azure Active Directory Connect, пространстве соединителя Azure Active Directory на сервере Azure Active Directory Connect и метавселенной на сервере Azure Active Directory Connect. 

Специалист службы поддержки Майкрософт не может изменять данные в системе и не видит пароли. 

## <a name="what-if-i-dont-want-the-microsoft-support-engineer-to-access-my-data"></a>Что если я не хочу, чтобы специалист службы поддержки Майкрософт получал доступ к моим данным? 
Если после установки агента вы решите, что при обращении в службу поддержки специалист службы поддержки Майкрософт не должен получать доступ к вашим данным, вы можете отключить эту возможность, изменив файл конфигурации службы, как описано ниже. 

1. Откройте в блокноте файл **C:\Program Files\Microsoft Azure AD Connect Administration Agent\AzureADConnectAdministrationAgentService.exe.config**.
2. Отключите параметр **UserDataEnabled**, как показано ниже. Если параметр **UserDataEnabled** существует и для него задано значение True, измените его на False. Если этот параметр не существует, добавьте его, как показано ниже.    

    ```xml
    <appSettings>
      <add key="TraceFilename" value="ADAdministrationAgent.log" />
      <add key="UserDataEnabled" value="false" />
    </appSettings>
    ```

3. Сохраните файл конфигурации.
4. Перезапустите службу агента администрирования Azure AD Connect, как показано ниже

![Снимок экрана: перезапуск службы агента администрирования Azure Active Directory](media/whatis-aadc-admin-agent/adminagent2.png)

## <a name="next-steps"></a>Дальнейшие действия
Узнайте больше об [интеграции локальных удостоверений с Azure Active Directory](whatis-hybrid-identity.md).
