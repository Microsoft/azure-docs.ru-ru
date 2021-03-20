---
title: Вход с защитой паролем в локальные ресурсы — Azure Active Directory
description: Узнайте, как включить вход с использованием ключа безопасности без пароля для локальных ресурсов с помощью Azure Active Directory
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 02/22/2021
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: librown, aakapo
ms.collection: M365-identity-device-management
ms.openlocfilehash: db1b559bb4f6a1f8866116c287df5b814500210b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101647478"
---
# <a name="enable-passwordless-security-key-sign-in-to-on-premises-resources-with-azure-active-directory"></a>Включение входа с помощью ключа безопасности без пароля для локальных ресурсов с Azure Active Directory 

В этом документе рассматривается включение проверки подлинности без пароля для локальных ресурсов в средах с присоединенными к **Azure AD** и гибридными устройствами Windows 10, **присоединенными к Azure** AD. Эта функция обеспечивает простой единый вход (SSO) в локальные ресурсы с помощью совместимых с Майкрософт ключей безопасности.

## <a name="sso-to-on-premises-resources-using-fido2-keys"></a>Единый вход в локальные ресурсы с помощью ключей FIDO2

Azure Active Directory (AD) может выдавать билеты Kerberos на выдачу билетов (TGT) для одного или нескольких доменов Active Directory. Эта функция позволяет пользователям входить в Windows с помощью современных учетных данных, таких как ключи безопасности FIDO2 и доступ к традиционным ресурсам на основе Active Directory. Управление билетами службы Kerberos и авторизацией по-прежнему осуществляется локальными контроллерами домена Active Directory.

Объект сервера Kerberos Azure AD создается в локальной Active Directory, а затем безопасно публикуется в Azure Active Directory. Объект не связан ни с одним физическим сервером. Это просто ресурс, который может использоваться Azure Active Directory для создания TGT Kerberos для домен Active Directory.

![Получение билета предоставления билета (TGT) из Azure AD и AD DS](./media/howto-authentication-passwordless-on-premises/fido2-ticket-granting-ticket-exchange-process.png)

1. Пользователь входит в систему на устройстве Windows 10 с помощью ключа безопасности FIDO2 и выполняет проверку подлинности в Azure AD.
1. Azure AD проверяет каталог на наличие ключа сервера Kerberos, соответствующего локальному домену AD пользователя.
   1. Azure AD создает TGT Kerberos для локального домена AD пользователя. TGT включает только идентификатор безопасности пользователя. Данные авторизации не включаются в TGT.
1. Билет TGT возвращается клиенту вместе с основным маркером обновления Azure AD (PRT).
1. Клиентский компьютер обращается к локальному контроллеру домена AD и задействует частичный TGT для полностью сформированного TGT.
1. Теперь на клиентском компьютере имеется Azure AD PRT и полное Active Directory TGT, а также доступ к облачным и локальным ресурсам.

## <a name="requirements"></a>Требования

Перед выполнением действий, описанных в этой статье, организации должны выполнить действия по [включению ключа безопасности без пароля для входа на устройства с Windows 10](howto-authentication-passwordless-security-key.md) .

Организации должны также отвечать следующим требованиям к программному обеспечению.

- Устройства должны работать под управлением Windows 10 версии 2004 или более поздней.
- Необходимо использовать версию 1.4.32.0 или более позднюю [Azure AD Connect](../hybrid/how-to-connect-install-roadmap.md#install-azure-ad-connect).
  - Дополнительные сведения о доступных параметрах гибридной проверки подлинности в Azure AD см. в статье [Выбор правильного метода проверки подлинности для Azure Active Directory гибридного решения для идентификации](../hybrid/choose-ad-authn.md) и [Выбор типа установки для Azure AD Connect](../hybrid/how-to-connect-install-select-installation.md).
- На контроллерах домена Windows Server должны быть установлены следующие исправления:
    - Для Windows Server 2016 — https://support.microsoft.com/help/4534307/windows-10-update-kb4534307
    - Для Windows Server 2019 — https://support.microsoft.com/help/4534321/windows-10-update-kb4534321

### <a name="supported-scenarios"></a>Поддерживаемые сценарии

Сценарий поддерживает единый вход (SSO) в обоих следующих сценариях.

- Для облачных ресурсов, таких как Microsoft 365 и других приложений с поддержкой SAML.
- Для локальных ресурсов и Windows-Integrated проверки подлинности на веб-сайтах. Ресурсы могут включать веб-сайты и сайты SharePoint, для которых требуется проверка подлинности IIS, а также ресурсы, использующие проверку подлинности NTLM.

### <a name="unsupported-scenarios"></a>Неподдерживаемые сценарии

Следующие сценарии не поддерживаются:

- Развертывание служб домен Active Directory Windows Server (AD DS) (только для локальных устройств).
- Сценарии RDP, VDI и Citrix, использующие ключ безопасности.
- S/MIME с помощью ключа безопасности.
- "Запуск от имени" с помощью ключа безопасности.
- Войдите на сервер с помощью ключа безопасности.

## <a name="create-kerberos-server-object"></a>Создание объекта сервера Kerberos

Администраторы используют средства PowerShell с сервера Azure AD Connect для создания объекта сервера Kerberos Azure AD в локальном каталоге. Выполните следующие действия в каждом домене и лесу в Организации, которые содержат пользователей Azure AD.

1. Выполните обновление до последней версии Azure AD Connect. В инструкциях предполагается, что вы уже настроили Azure AD Connect для поддержки гибридной среды.
1. На Azure AD Connect сервере откройте командную строку PowerShell с повышенными привилегиями и перейдите к `C:\Program Files\Microsoft Azure Active Directory Connect\AzureADKerberos\`
1. Выполните следующие команды PowerShell, чтобы создать объект сервера Kerberos Azure AD в локальном домене Active Directory и Azure Active Directoryном клиенте.

> [!NOTE]
> Замените `contoso.corp.com` в следующем примере локальным доменным именем Active Directory.

```powerShell
Import-Module ".\AzureAdKerberos.psd1"

# Specify the on-premises Active Directory domain. A new Azure AD
# Kerberos Server object will be created in this Active Directory domain.
$domain = "contoso.corp.com"

# Enter an Azure Active Directory global administrator username and password.
$cloudCred = Get-Credential

# Enter a domain administrator username and password.
$domainCred = Get-Credential

# Create the new Azure AD Kerberos Server object in Active Directory
# and then publish it to Azure Active Directory.
Set-AzureADKerberosServer -Domain $domain -CloudCredential $cloudCred -DomainCredential $domainCred
```

### <a name="viewing-and-verifying-the-azure-ad-kerberos-server"></a>Просмотр и проверка сервера Kerberos Azure AD

Вы можете просмотреть и проверить только что созданный сервер Kerberos AD Azure с помощью следующей команды:

```powerShell
Get-AzureADKerberosServer -Domain $domain -CloudCredential $cloudCred -DomainCredential $domainCred
```

Эта команда выводит свойства сервера Azure AD Kerberos. Можно проверить свойства, чтобы убедиться, что все находится в правильном порядке.

| Свойство. | Описание |
| --- | --- |
| ID | Уникальный идентификатор объекта AD DS контроллера домена. Этот идентификатор иногда называют "слотом" или "ИДЕНТИФИКАТОРом ветви". |
| DomainDnsName | Доменное имя DNS домен Active Directory. |
| ComputerAccount | Объект учетной записи компьютера для объекта сервера Kerberos Azure AD (DC). |
| UserAccount | Отключенный объект учетной записи пользователя, содержащий ключ шифрования TGT сервера Kerberos Azure AD. DN этой учетной записи `CN=krbtgt_AzureAD,CN=Users,<Domain-DN>` |
| кэйверсион | Версия ключа шифрования билета TGT сервера Azure AD Kerberos. Версия назначается при создании ключа. Затем версия увеличивается при каждом повороте ключа. Приращения основано на метаданных репликации и, скорее всего, больше, чем один. Например, начальным *кэйверсион* может быть *192272*. При первом повороте ключа версия может перейти на *212621*. Важно убедиться, что *кэйверсион* для локального объекта и *клаудкэйверсион* для облачного объекта одинаковы. |
| кэйупдатедон | Дата и время, когда ключ шифрования билета TGT сервера Azure AD был обновлен или создан. |
| кэйупдатедфром | Контроллер домена, на котором последний раз был обновлен ключ шифрования билета TGT сервера Azure AD. |
| CloudId | ИДЕНТИФИКАТОР из объекта Azure AD. Должен соответствовать указанному выше ИДЕНТИФИКАТОРу. |
| клауддомаинднснаме | *Домаинднснаме* из объекта Azure AD. Должен соответствовать *домаинднснаме* выше. |
| клаудкэйверсион | *Кэйверсион* из объекта Azure AD. Должен соответствовать *кэйверсион* выше. |
| клаудкэйупдатедон | *Кэйупдатедон* из объекта Azure AD. Должен соответствовать *кэйупдатедон* выше. |

### <a name="rotating-the-azure-ad-kerberos-server-key"></a>Вращение ключа сервера Kerberos Azure AD

Ключи KRBTGT для шифрования сервера Azure AD см. на регулярной основе. Рекомендуется следовать тому же расписанию, которое используется для смены всех остальных ключей домен Active Directory контроллеров KRBTGT.

> [!WARNING]
> Существуют и другие средства, которые могут поворачивать ключи krbtgt, однако для смены ключей KRBTGT сервера Kerberos Azure AD необходимо использовать средства, упомянутые в этом документе. Это гарантирует, что ключи будут обновлены как в локальной службе AD, так и в Azure AD.

```powerShell
Set-AzureADKerberosServer -Domain $domain -CloudCredential $cloudCred -DomainCredential $domainCred -RotateServerKey
```

### <a name="removing-the-azure-ad-kerberos-server"></a>Удаление сервера Kerberos Azure AD

Если вы хотите отменить сценарий и удалить сервер Azure AD Kerberos из локальной Active Directory и Azure Active Directory, выполните следующую команду:

```powerShell
Remove-AzureADKerberosServer -Domain $domain -CloudCredential $cloudCred -DomainCredential $domainCred
```

### <a name="multi-forest-and-multi-domain-scenarios"></a>Сценарии с несколькими лесами и несколькими доменами

Объект сервера Kerberos Azure AD представлен в Azure AD как объект *керберосдомаин* . Каждый локальный домен Active Directory представляется в Azure AD как один объект *керберосдомаин* .

Например, в Организации имеется Active Directory лес с двумя доменами `contoso.com` и `fabrikam.com` . Если вы решили разрешить Azure AD выдавать TGT Kerberos для всего леса, в Azure AD есть два объекта *керберосдомаин* . Один объект *керберосдомаин* для `contoso.com` , и один для `fabrikam.com` . При наличии нескольких Active Directory лесов существует один объект *керберосдомаин* для каждого домена в каждом лесу.

Необходимо выполнить шаги для [создания объекта сервера Kerberos](#create-kerberos-server-object) в каждом домене и лесу организации, которые содержат пользователей Azure AD.

## <a name="known-behavior"></a>Известное поведение

Если срок действия пароля истек, вход с помощью FIDO будет заблокирован. Ожидание заключается в том, что пользователь должен сбросить свой пароль, прежде чем сможет войти в систему с помощью FIDO.

## <a name="troubleshooting-and-feedback"></a>Устранение неполадок и обратная связь

Если вы хотите поделиться отзывами или столкнуться с проблемами с этой функцией, предоставьте общий доступ через приложение центра отзывов Windows, выполнив следующие действия.

1. Запустите **центр отзывов** и убедитесь, что вы вошли в него.
1. Отправьте отзыв по следующей категории:
   - Категория: безопасность и конфиденциальность
   - Подкатегория: FIDO
1. Для записи журналов используйте параметр, чтобы **повторно создать мою проблему**.

## <a name="frequently-asked-questions"></a>Часто задаваемые вопросы

### <a name="does-this-work-in-my-on-premises-environment"></a>Работает ли это в локальной среде?

Эта функция не работает в чистой локальной среде домен Active Directory служб (AD DS).

### <a name="my-organization-requires-two-factor-authentication-to-access-resources-what-can-i-do-to-support-this-requirement"></a>Для доступа к ресурсам Моя организация должна использовать двухфакторную проверку подлинности. Что можно сделать для поддержки этого требования?

Ключи безопасности бывают разными конструктивными факторами. Обратитесь к изготовителю устройства, чтобы обсудить, как можно включить устройства с помощью ПИН-кода или биометрической метрики в качестве второго фактора.

### <a name="can-admins-set-up-security-keys"></a>Могут ли администраторы настраивать ключи безопасности?

Мы работаем над этой возможностью в общедоступной версии этой функции.

### <a name="where-can-i-go-to-find-compliant-security-keys"></a>Где можно найти соответствующие ключи безопасности?

[Ключи безопасности FIDO2](concept-authentication-passwordless.md#fido2-security-keys)

### <a name="what-do-i-do-if-i-lose-my-security-key"></a>Что делать, если я потеряли ключ безопасности?

Чтобы удалить ключи из портал Azure, перейдите на страницу **сведений о безопасности** и удалите ключ безопасности.

### <a name="im-not-able-to-use-fido-immediately-after-i-create-a-hybrid-azure-ad-joined-machine"></a>Я не могу использовать FIDO сразу после создания гибридного компьютера, присоединенного к Azure AD

Если вы выполняете чистую установку гибридного компьютера, присоединенного к Azure AD, после присоединения и перезапуска домена необходимо войти в систему с помощью пароля и дождаться синхронизации политики, прежде чем использовать FIDO для входа.

- Проверьте текущее состояние, введя `dsregcmd /status` в командное окно и убедитесь, что для обоих *AzureAdJoined* и *домаинжоинед* отображается *значение Да*.
- Эта задержка является известным ограничением для устройств, присоединенных к домену, и не зависит от FIDO.

### <a name="im-unable-to-get-sso-to-my-ntlm-network-resource-after-signing-in-with-fido-and-get-a-credential-prompt"></a>Не удается получить единый вход для сетевого ресурса NTLM после входа с помощью FIDO и получения запроса учетных данных

Убедитесь, что все контроллеры домена исправлены на время ответа на обслуживание запроса ресурсов. Чтобы проверить, доступен ли контроллер домена, на котором работает эта функция, просмотрите выходные данные `nltest /dsgetdc:contoso /keylist /kdc` .

## <a name="next-steps"></a>Дальнейшие действия

[Дополнительные сведения о пароле](concept-authentication-passwordless.md)
