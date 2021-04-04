---
title: 'Azure AD Connect: приступая к работе с использованием стандартных параметров | Документация Майкрософт'
description: Узнайте, как скачать, установить и запустить мастер установки Azure AD Connect.
services: active-directory
author: billmath
manager: daveba
editor: curtand
ms.assetid: b6ce45fd-554d-4f4d-95d1-47996d561c9f
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 09/28/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0a655f355bb77d937f4daff2f8987769416ebd8c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "89279675"
---
# <a name="getting-started-with-azure-ad-connect-using-express-settings"></a>Приступая к работе с Azure AD Connect с использованием стандартных параметров
**Стандартные параметры** Azure AD Connect применяются, если вы используете для проверки подлинности топологию с одним лесом и [синхронизацию хэша паролей](how-to-connect-password-hash-synchronization.md). **Стандартные параметры** заданы по умолчанию и используются в большинстве часто развертываемых сценариев. Выполнив всего несколько простых действий, можно расширить локальный каталог в облако.

Перед установкой Azure AD Connect [скачайте Azure AD Connect](https://go.microsoft.com/fwlink/?LinkId=615771) и выполните предварительные шаги, перечисленные в перечне [требований к оборудованию и программному обеспечению](how-to-connect-install-prerequisites.md).

Если стандартные параметры не соответствуют топологии, см. сведения о других сценариях в [дополнительной документации](#related-documentation).

## <a name="express-installation-of-azure-ad-connect"></a>Экспресс-установка Azure AD Connect
Чтобы ознакомиться с этими действиями на практике, см. раздел с [видео](#videos).

1. Войдите с правами локального администратора на сервер, на котором требуется установить Azure AD Connect. Это должен быть сервер, который нужно сделать сервером синхронизации.
2. Перейдите к файлу **AzureADConnect.msi** и дважды щелкните его.
3. На экране приветствия установите флажок, подтверждающий ваше согласие с условиями лицензионного соглашения, и нажмите кнопку **Продолжить**.  
4. На экране «Стандартные параметры» щелкните **Использовать стандартные параметры**.  
   ![Приветствие мастера установки Azure AD Connect.](./media/how-to-connect-install-express/express.png)
5. На экране "Подключение к Azure AD" введите имя пользователя и пароль глобального администратора Azure AD. Щелкните **Далее**.  
   ![Подключение к Azure AD](./media/how-to-connect-install-express/connectaad.png)  
   Если вы получаете сообщение об ошибке и испытываете проблемы с подключением, см. сведения в статье [Устранение неполадок подключения в Azure AD Connect](tshoot-connect-connectivity.md).
6. На экране "Подключение к AD DS" введите имя пользователя и пароль учетной записи администратора предприятия. Вы можете указать имя домена в формате NetBios, либо ввести полное доменное имя, т. е. FABRIKAM\administrator или fabrikam.com\administrator. Щелкните **Далее**.  
   ![Подключение к AD DS](./media/how-to-connect-install-express/connectad.png)
7. Страница [**Настройка входа в Azure AD**](plan-connect-user-signin.md#azure-ad-sign-in-configuration) будет отображаться, только если вы не завершили [проверку доменов](../fundamentals/add-custom-domain.md) при выполнении [предварительных требований](how-to-connect-install-prerequisites.md).
   ![Непроверенные домены](./media/how-to-connect-install-express/unverifieddomain.png)  
   Если отобразилась эта страница, просмотрите все домены с пометкой **Не добавлено** и **Не проверено**. Убедитесь, что используемые домены прошли проверку в Azure AD. Проверив домены, щелкните значок обновления.
8. На экране "Все готово к настройке" нажмите кнопку **Установить**.
   * При необходимости на странице "Готово к настройке" можно снять флажок **Start the synchronization process as soon as configuration completes** (Запустить синхронизацию сразу после завершения настройки). Снимите этот флажок, если нужно настроить дополнительные параметры, например [фильтрацию](how-to-connect-sync-configure-filtering.md). Если отменить выбор этого параметра, мастер настроит синхронизацию, но оставит планировщик отключенным. Он не запустится, пока вы не включите его вручную, [повторно запустив мастер установки](how-to-connect-installation-wizard.md).
   * Если установить флажок **Start the synchronization process as soon as configuration completes** (Начать синхронизацию сразу по завершении настройки), немедленно запустится полная синхронизация всех пользователей, групп и контактов в Azure AD.
   * Если Exchange находится в локальной службе Active Directory, вы также можете включить [**гибридное развертывание Exchange**](/exchange/exchange-hybrid). Этот флажок нужно установить, если почтовые ящики Exchange будут одновременно использоваться в облаке и локальной среде.
     ![Все готово к настройке Azure AD Connect](./media/how-to-connect-install-express/readytoconfigure.png)
9. По завершении установки щелкните **Выход**.
10. После завершения установки выполните выход и снова войдите, прежде чем начинать использовать диспетчер службы синхронизации или редактор правил синхронизации.

## <a name="videos"></a>Видеоролики
Рекомендуем посмотреть видео по экспресс-установке:

> [!VIDEO https://channel9.msdn.com/Series/Azure-Active-Directory-Videos-Demos/Azure-Active-Directory-Connect-Express-Settings/player]
>
>

## <a name="next-steps"></a>Дальнейшие действия
После установки Azure AD Connect можно [проверить установку и назначить лицензии](how-to-connect-post-installation.md).

Дополнительные сведения о функциях, которые были включены в процессе установки, см. в следующих статьях: [Azure AD Connect: автоматическое обновление](how-to-connect-install-automatic-upgrade.md), [Синхронизация Azure AD Connect: предотвращение случайного удаления](how-to-connect-sync-feature-prevent-accidental-deletes.md) и [Использование Azure AD Connect Health для синхронизации](how-to-connect-health-sync.md).

Дополнительные сведения см. в статье [Синхронизация Azure AD Connect: планировщик](how-to-connect-sync-feature-scheduler.md).

Узнайте больше об [интеграции локальных удостоверений с Azure Active Directory](whatis-hybrid-identity.md).

## <a name="related-documentation"></a>Дополнительная документация

| Раздел | Ссылка |
| --- | --- |
| Обзор Azure AD Connect | [Интеграция локальных каталогов с Azure Active Directory](whatis-hybrid-identity.md)
| Установка с помощью настроенных параметров | [Выборочная установка Azure AD Connect](how-to-connect-install-custom.md) |
| Обновление из DirSync | [Azure AD Connect: обновление DirSync](how-to-dirsync-upgrade-get-started.md)|
| Учетные записи, используемые для установки | [Azure AD Connect: учетные записи и разрешения](reference-connect-accounts-permissions.md) |