---
title: План установки Azure AD Connect и Azure AD Connect Health. | Документы Майкрософт
description: Этот документ содержит общие сведения о параметрах установки и путях, доступных для установки Connect Health и Azure AD Connect.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 09/18/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: ae575aa6544a174a70eb8ea4749566e8660280e2
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94873273"
---
# <a name="azure-ad-connect-and-azure-ad-connect-health-installation-roadmap"></a>План установки Azure AD Connect и Azure AD Connect Health

## <a name="install-azure-ad-connect"></a>Установка Azure AD Connect

> [!IMPORTANT]
> Мы поддерживаем изменение или использование служб синхронизации Azure AD Connect только в контексте официально задокументированных действий. Любое из этих действий может привести к несогласованному или неподдерживаемому состоянию Azure AD Connect синхронизации. В результате корпорация Майкрософт не может предоставить техническую поддержку для таких развертываний.

Вы можете загрузить Azure AD Connect из [Центра загрузки Майкрософт](https://go.microsoft.com/fwlink/?LinkId=615771).

| Решение | Сценарий |
| --- | --- |
| Перед началом работы: [оборудование и предварительные требования](how-to-connect-install-prerequisites.md) |<li>Шаги, которые нужно выполнить до установки Azure AD Connect.</li> |
| [Стандартные параметры](how-to-connect-install-express.md) |<li>Рекомендуемый вариант при наличии AD с одним лесом.</li> <li>Пользователи входят в систему с одним и тем же паролем, используя синхронизацию паролей.</li> |
| [Настраиваемые параметры](how-to-connect-install-custom.md) |<li>Используется при наличии нескольких лесов. Поддерживает многие локальные [топологии](plan-connect-topologies.md).</li> <li>Настройка режима входа (например, с помощью сквозной аутентификации, AD FS для федерации или стороннего поставщика удостоверений).</li> <li>Настройка функций синхронизации, таких как фильтрация и обратная запись.</li> |
| [Обновление из DirSync](how-to-dirsync-upgrade-get-started.md) |<li>Используется при наличии уже работающего сервера DirSync.</li> |
| [Переход с Azure AD Sync на Azure AD Connect](how-to-upgrade-previous-version.md) |<li>Есть несколько разных способов на выбор.</li> |

[После установки](how-to-connect-post-installation.md) следует проверить правильность работы системы и назначить пользователям лицензии.

### <a name="next-steps-to-install-azure-ad-connect"></a>Следующие шаги по установке Azure AD Connect
|Раздел |Ссылка|  
| --- | --- |
|Загрузка Azure AD Connect | [Загрузка Azure AD Connect](https://go.microsoft.com/fwlink/?LinkId=615771)|
|Установка с помощью стандартных параметров | [Экспресс-установка Azure AD Connect](./how-to-connect-install-express.md)|
|Установка с помощью настроенных параметров | [Выборочная установка Azure AD Connect](./how-to-connect-install-custom.md)|
|Обновление из DirSync | [Azure AD Connect: обновление DirSync](./how-to-dirsync-upgrade-get-started.md)|
|После установки | [Проверка установки и назначение лицензий ](how-to-connect-post-installation.md)|

### <a name="learn-more-about-install-azure-ad-connect"></a>Дополнительные сведения об установке Azure AD Connect
Также требуется подготовиться к [рабочим](./how-to-connect-sync-staging-server.md) вопросам. Может потребоваться наличие резервного сервера, на который можно будет легко перейти в случае [аварии](how-to-connect-sync-staging-server.md#disaster-recovery). Если планируется часто изменять конфигурацию, следует предусмотреть сервер для [промежуточного режима](how-to-connect-sync-staging-server.md) .

|Раздел |Ссылка|  
| --- | --- |
|Поддерживаемые топологии | [Топологии для Azure AD Connect](plan-connect-topologies.md)|
|Принципы проектирования | [Azure AD Connect концепции проектирования](plan-connect-design-concepts.md)|
|Учетные записи, используемые для установки | [Azure AD Connect: учетные записи и разрешения](reference-connect-accounts-permissions.md)|
|Операционное планирование | [Службы синхронизации Azure AD Connect: рабочие задачи и рекомендации](./how-to-connect-sync-staging-server.md)|
|Параметры входа пользователя | [Azure AD Connect параметры входа пользователя](plan-connect-user-signin.md)|

## <a name="configure-sync-features"></a>Настройка функций синхронизации
Azure AD Connect поставляется с несколькими функциями, которые можно при необходимости включить или они включены по умолчанию. В некоторых сценариях и топологиях может потребоваться дополнительная конфигурация некоторых функций.

[Фильтрация](how-to-connect-sync-configure-filtering.md) используется, если требуется ограничить количество объектов, синхронизируемых с Azure AD. По умолчанию все пользователи, контакты, группы и компьютеры с ОС Windows 10 синхронизируются. Фильтрацию можно изменить в зависимости от доменов, подразделений и атрибутов.

[Синхронизация хэша паролей](how-to-connect-password-hash-synchronization.md) подразумевает синхронизацию хэша паролей из Active Directory в Azure AD. Пользователи могут использовать один и тот же пароль локально и в облаке, но управление им осуществляется только из одного расположения. Так как при синхронизации паролей в качестве главного источника используется локальная служба Active Directory, вы также можете применить собственную политику паролей.

[Компонент обратной записи паролей](../authentication/tutorial-enable-sspr.md) позволит вашим пользователям изменять и сбрасывать пароли в облаке, а также применять вашу локальную политику паролей.

[Обратная запись устройств](how-to-connect-device-writeback.md) позволит записать устройство, зарегистрированное в Azure AD, обратно в локальную Active Directory, чтобы его можно было использовать для условного доступа.

Функция [предотвращения случайного удаления](how-to-connect-sync-feature-prevent-accidental-deletes.md) включена по умолчанию и защищает облачный каталог от множества одновременных удалений. По умолчанию она позволяет сделать 500 удалений за сеанс. Этот параметр можно изменить в зависимости от размера вашей организации.

[Автоматическое обновление](how-to-connect-install-automatic-upgrade.md) , которое включено по умолчанию для установок со стандартными параметрами, гарантирует использование последней версии Azure AD Connect.

### <a name="next-steps-to-configure-sync-features"></a>Дальнейшие действия по настройке функций синхронизации
|Раздел |Ссылка|  
| --- | --- |
|Настройка фильтрации | [Синхронизация Azure AD Connect: Настройка фильтрации](how-to-connect-sync-configure-filtering.md)|
|Синхронизация хэша паролей | [Синхронизации хэша паролей](how-to-connect-password-hash-synchronization.md)|
|Сквозная проверка подлинности | [Сквозная проверка подлинности](how-to-connect-pta.md)
|Компонент обратной записи паролей | [Начало работы с управлением паролями](../authentication/tutorial-enable-sspr.md)|
|Обратная запись устройств | [Включение обратной записи устройств в службе Azure AD Connect](how-to-connect-device-writeback.md)|
|Предотвращение случайного удаления | [Синхронизация Azure AD Connect: предотвращение случайного удаления](how-to-connect-sync-feature-prevent-accidental-deletes.md)|
|Автоматическое обновление | [Azure AD Connect выполняет следующие функции: Автоматическое обновление](how-to-connect-install-automatic-upgrade.md)|

## <a name="customize-azure-ad-connect-sync"></a>Настройка синхронизации Azure AD Connect
Синхронизация Azure AD Connect поставляется с конфигурацией по умолчанию, ориентированной на работу с большинством заказчиков и топологий. Тем не менее, возникают ситуации, когда конфигурация по умолчанию не будет работать и ее необходимо скорректировать. Поддерживается внесение изменений, документированных в этом разделе и связанных статьях.

Если вы не работали ранее с топологией синхронизации, вам необходимо ознакомиться с основными сведениями и используемыми терминами, описанными в [технических концепциях](how-to-connect-sync-technical-concepts.md). Azure AD Connect является следующим этапом развития MIIS2003, ILM2007 и FIM2010. Даже если некоторые элементы идентичны, многое также изменилось.

[Конфигурация, используемая по умолчанию](concept-azure-ad-connect-sync-default-configuration.md) , может включать несколько лесов. В таких топологиях объект пользователя может быть представлен как контакт в другом лесу. У пользователя также может быть связанный почтовый ящик в другом лесу ресурсов. Поведение конфигурации по умолчанию описано в разделе [пользователей и контактов](concept-azure-ad-connect-sync-user-and-contacts.md).

Модель конфигурации в синхронизации называется [декларативной подготовкой](concept-azure-ad-connect-sync-declarative-provisioning-expressions.md). Потоки дополнительных атрибутов используют [функции](reference-connect-sync-functions-reference.md) для выражения преобразований атрибутов. Всю конфигурацию можно просмотреть и изучить с помощью средств, поставляемых вместе с Azure AD Connect. Если в конфигурацию необходимо внести изменения, убедитесь, что вы придерживаетесь [рекомендаций](how-to-connect-sync-best-practices-changing-default-configuration.md) , упрощающих переход на новые выпуски.

### <a name="next-steps-to-customize-azure-ad-connect-sync"></a>Следующие действия по настройке синхронизации Azure AD Connect
|Раздел |Ссылка|  
| --- | --- |
|Все статьи о синхронизации Azure AD Connect | [Службы синхронизации Azure AD Connect](how-to-connect-sync-whatis.md)|
|технических концепциях | [Синхронизация Azure AD Connect: технические понятия](how-to-connect-sync-technical-concepts.md)|
|Общие сведения о конфигурации по умолчанию | [Службы синхронизации Azure AD Connect: общие сведения о конфигурации по умолчанию](concept-azure-ad-connect-sync-default-configuration.md)|
|Общее представление о пользователях и контактах | [Синхронизация Azure AD Connect: общее представление о пользователях и контактах](concept-azure-ad-connect-sync-user-and-contacts.md)|
|декларативной подготовкой | [Azure AD Connect синхронизации: Общие сведения о выражениях декларативной подготовки](concept-azure-ad-connect-sync-declarative-provisioning-expressions.md)|
|Изменение конфигурации по умолчанию | [Рекомендации по изменению конфигурации по умолчанию](how-to-connect-sync-best-practices-changing-default-configuration.md)|

## <a name="configure-federation-features"></a>Настройка функций федерации

Служба Azure AD Connect предоставляет несколько функций, которые упрощают федерацию с Azure AD с помощью AD FS и управление доверием федерации. Она поддерживает AD FS в Windows Server 2012 R2 или более поздней версии.

[Обновите сертификат TLS/SSL AD FS фермы](how-to-connect-fed-ssl-update.md) , даже если вы не используете Azure AD Connect для управления доверием федерации.

[Добавьте сервер AD FS](how-to-connect-fed-management.md#addadfsserver) в ферму, чтобы расширить ее в соответствии с потребностями.

[Восстановите отношение доверия](how-to-connect-fed-management.md#repairthetrust) с Azure AD несколькими щелчками мыши.

В AD FS можно настроить поддержку [нескольких доменов](how-to-connect-install-multiple-domains.md). Например, у вас может быть несколько доменов верхнего уровня, которые необходимо использовать для федерации.

Если на сервере ADFS не настроено автоматическое обновление сертификатов из Azure AD или если используется решение, отличное от ADFS, то при необходимости [обновления сертификатов](how-to-connect-fed-o365-certs.md)вы получите соответствующее оповещение.

### <a name="next-steps-to-configure-federation-features"></a>Дальнейшие действия по настройке функций федерации
|Раздел |Ссылка|  
| --- | --- |
|Все статьи, посвященные AD FS | [Azure AD Connect и федерация](how-to-connect-fed-whatis.md)|
|Настройка служб AD FS с поддоменами | [Поддержка нескольких доменов для федерации с Azure AD](how-to-connect-install-multiple-domains.md)|
|Управление фермой AD FS | [Управление службами федерации Active Directory и их настройка с помощью Azure AD Connect](how-to-connect-fed-management.md)|
|Обновление сертификатов федерации вручную | [Продление сертификатов федерации для Microsoft 365 и Azure AD](how-to-connect-fed-o365-certs.md)|


## <a name="get-started-with-azure-ad-connect-health"></a>Приступая к работе с Azure AD Connect Health
Чтобы начать работу с Azure AD Connect Health, сделайте следующее:

1. [Получите Azure AD Premium](../fundamentals/active-directory-get-started-premium.md) или [запустите пробную версию](https://azure.microsoft.com/trial/get-started-active-directory/).
2. [Скачайте и установите агенты Azure AD Connect Health](#download-and-install-azure-ad-connect-health-agent) на серверах удостоверений.
3. Просмотрите панель мониторинга Azure AD Connect Health по адресу [https://aka.ms/aadconnecthealth](https://aka.ms/aadconnecthealth) .

> [!NOTE]
> Помните, чтобы на панели мониторинга Azure AD Connect Health отображались какие-либо данные, потребуется установить агенты Azure AD Connect Health на целевых серверах.
>
>

## <a name="download-and-install-azure-ad-connect-health-agent"></a>Скачивание и установка агента Azure AD Connect Health
* Обязательно [выполните требования](how-to-connect-health-agent-install.md#requirements) для Azure AD Connect Health.
* Приступая к работе с Azure AD Connect Health для AD FS
    * [Скачайте агент Azure AD Connect Health для AD FS.](https://go.microsoft.com/fwlink/?LinkID=518973)
    * [См. инструкции по установке](how-to-connect-health-agent-install.md#install-the-agent-for-ad-fs).
* Приступая к работе с Azure AD Connect Health для синхронизации
    * [Скачайте и установите последнюю версию Azure AD Connect.](https://go.microsoft.com/fwlink/?linkid=615771) Агент Azure AD Connect Health для синхронизации будет установлен вместе с Azure AD Connect (версии 1.0.9125.0 или более поздней).
* Приступая к работе с Azure AD Connect Health для AD DS
    * [Скачайте агент Azure AD Connect Health для AD DS](https://go.microsoft.com/fwlink/?LinkID=820540).
    * [См. инструкции по установке](how-to-connect-health-agent-install.md#install-the-agent-for-azure-ad-ds).


## <a name="azure-ad-connect-health-portal"></a>Портал Azure AD Connect Health
На портале Azure AD Connect Health можно просматривать оповещения, отслеживать производительность и аналитику использования. С помощью URL-адреса https://aka.ms/aadconnecthealth можно перейти в главную колонку Azure AD Connect Health. Ее можно считать окном. В главной колонке вы увидите **быстрое начало**, службы в Azure AD Connect Health и дополнительные параметры конфигурации. Просмотрите снимок экрана колонки ниже и ознакомьтесь с кратким описанием содержащихся в ней элементов. После развертывания агентов служба работоспособности автоматически определит службы, отслеживаемые в Azure AD Connect Health.

> [!NOTE]
> Сведения о лицензировании см. в статье [Часто задаваемые вопросы об Azure AD Connect Health](reference-connect-health-faq.md) или [на странице с ценами на Azure AD](https://aka.ms/aadpricing).
    
![Azure AD Connect Health](./media/whatis-hybrid-identity-health/portalsidebar.png)

* **Быстрый запуск.** При выборе этого элемента откроется колонка **быстрого запуска**. Вы можете скачать агент Azure AD Connect Health, выбрав **Получить средства**. Кроме того, вы также можете получить доступ к документации и оставить отзыв.
* **Azure Active Directory Connect (Sync).** В этом разделе отображаются серверы Azure AD Connect, которые в настоящее время отслеживает служба Azure AD Connect Health. В записи **Ошибки синхронизации** отображаются основные ошибки синхронизации в первой подключенной службе синхронизации по категориям. Когда вы выберете запись **Синхронизировать службы**, откроется колонка с информацией о серверах Azure AD Connect. Дополнительные сведения о возможностях см. в статье [Использование Azure AD Connect Health для синхронизации](how-to-connect-health-sync.md).
* **Службы федерации Active Directory.** Этот раздел позволяет просмотреть все службы AD FS, которые в настоящее время отслеживает служба Azure AD Connect Health. Если выбрать экземпляр, откроется колонка со сведениями об этом экземпляре службы. Сюда входит обзор, свойства, оповещения, мониторинг и аналитика по использованию. Дополнительные сведения о возможностях см. в статье [Использование Azure AD Connect Health для синхронизации](how-to-connect-health-adfs.md).
* **Доменные службы Active Directory.** В этом разделе представлены все леса AD DS, которые в настоящее время отслеживает служба Azure AD Connect Health. Если выбрать лес, откроется колонка со сведениями об этом лесе. Эти сведения включают обзор основной информации, панель мониторинга контроллеров домена, панель мониторинга состояния репликации, оповещения и мониторинг. Дополнительные сведения о возможностях см. в статье [Использование Azure AD Connect Health с AD DS](how-to-connect-health-adds.md).
* **Настройка.** В этом разделе можно включить или отключить следующие функции.

   - **Автоматическое обновление** агента Azure AD Connect Health до последней версии: агент Azure AD Connect Health автоматически обновляется каждый раз, когда доступны новые версии. Этот параметр по умолчанию включен.
   - **Доступ к данным** из каталога Azure AD только в целях устранения неполадок. Если этот параметр включен, корпорация Майкрософт может получить доступ к тем же данным, которые отображаются у пользователя. Эти сведения могут быть полезны для устранения неполадок и предоставления необходимой помощи. По умолчанию этот параметр отключен
* Раздел **Управление доступом на основе ролей (IAM)** предназначен для управления доступом к данным Connect Health с помощью ролей. 

## <a name="next-steps"></a>Next Steps

- [Оборудование и предварительные требования](how-to-connect-install-prerequisites.md) 
- [Стандартные параметры](how-to-connect-install-express.md)
- [Настраиваемые параметры](how-to-connect-install-custom.md)
- [Синхронизация хэша паролей](how-to-connect-password-hash-synchronization.md)|
- [Сквозная проверка подлинности](how-to-connect-pta.md)
- [Azure AD Connect и федерация](how-to-connect-fed-whatis.md)
- [Установка агентов Azure AD Connect Health](how-to-connect-health-agent-install.md) 
- [Службы синхронизации Azure AD Connect](how-to-connect-sync-whatis.md)