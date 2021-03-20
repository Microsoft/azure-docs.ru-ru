---
title: Часто задаваемые вопросы о доменных службах Azure AD | Документация Майкрософт
description: Прочитайте и ознакомьтесь с некоторыми часто задаваемыми вопросами о настройке, администрировании и доступности для Azure Active Directory доменных служб.
services: active-directory-ds
author: justinha
manager: daveba
ms.assetid: 48731820-9e8c-4ec2-95e8-83dba1e58775
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 02/09/2021
ms.author: justinha
ms.openlocfilehash: 3d0f2b44f37cb318be2117b5dc5d8b42b418ff19
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100090995"
---
# <a name="frequently-asked-questions-faqs-about-azure-active-directory-ad-domain-services"></a>Часто задаваемые вопросы о доменных службах Azure Active Directory (AD)

На этой странице приведены ответы на часто задаваемые вопросы об Azure Active Directory доменных службах.

## <a name="configuration"></a>Конфигурация

* [Можно ли создать несколько управляемых доменов для одного каталога Azure AD?](#can-i-create-multiple-managed-domains-for-a-single-azure-ad-directory)
* [Можно ли включить доменные службы Azure AD в классической виртуальной сети?](#can-i-enable-azure-ad-domain-services-in-a-classic-virtual-network)
* [Можно ли включить доменные службы Azure AD в виртуальной сети Azure Resource Manager?](#can-i-enable-azure-ad-domain-services-in-an-azure-resource-manager-virtual-network)
* [Можно ли перенести существующий управляемый домен из классической виртуальной сети в диспетчер ресурсовную виртуальную сеть?](#can-i-migrate-my-existing-managed-domain-from-a-classic-virtual-network-to-a-resource-manager-virtual-network)
* [Можно ли включить доменные службы Azure AD в подписку Azure CSP (поставщик облачных решений)?](#can-i-enable-azure-ad-domain-services-in-an-azure-csp-cloud-solution-provider-subscription)
* [Можно ли включить доменные службы Azure AD в федеративный каталог Azure AD? Не синхронизировать хэши паролей с Azure AD. Можно ли включить доменные службы Azure AD для этого каталога?](#can-i-enable-azure-ad-domain-services-in-a-federated-azure-ad-directory-i-do-not-synchronize-password-hashes-to-azure-ad-can-i-enable-azure-ad-domain-services-for-this-directory)
* [Можно ли сделать доменные службы AD Azure доступными в нескольких виртуальных сетях в рамках подписки?](#can-i-make-azure-ad-domain-services-available-in-multiple-virtual-networks-within-my-subscription)
* [Можно ли включить доменные службы Azure AD с помощью PowerShell?](#can-i-enable-azure-ad-domain-services-using-powershell)
* [Можно ли включить доменные службы Azure AD с помощью шаблона диспетчер ресурсов?](#can-i-enable-azure-ad-domain-services-using-a-resource-manager-template)
* [Могу ли я добавить контроллеры домена в управляемый домен доменных служб Azure AD?](#can-i-add-domain-controllers-to-an-azure-ad-domain-services-managed-domain)
* [Можно ли приглашать гостевых пользователей в моем каталоге использовать доменные службы Azure AD?](#can-guest-users-be-invited-to-my-directory-use-azure-ad-domain-services)
* [Можно ли переместить существующий управляемый домен доменных служб Azure AD в другую подписку, группу ресурсов, регион или виртуальную сеть?](#can-i-move-an-existing-azure-ad-domain-services-managed-domain-to-a-different-subscription-resource-group-region-or-virtual-network)
* [Имеются ли в доменных службах Azure AD варианты высокой доступности?](#does-azure-ad-domain-services-include-high-availability-options)

### <a name="can-i-create-multiple-managed-domains-for-a-single-azure-ad-directory"></a>Можно ли создать несколько управляемых доменов для одного каталога Azure AD?
Нет. Для одного каталога Azure AD можно создать только один управляемый домен, поддерживаемый доменными службами Azure AD.

### <a name="can-i-enable-azure-ad-domain-services-in-a-classic-virtual-network"></a>Можно ли включить доменные службы Azure AD в классической виртуальной сети?
Классические виртуальные сети не поддерживаются для новых развертываний. Существующие управляемые домены, развернутые в классических виртуальных сетях, продолжают поддерживаться до тех пор, пока они не будут сняты с 1 марта 2023 г. Для существующих развертываний можно [перенести доменные службы Azure AD из классической модели виртуальной сети в Диспетчер ресурсов](migrate-from-classic-vnet.md).

Дополнительные сведения см. в [официальном](https://azure.microsoft.com/updates/we-are-retiring-azure-ad-domain-services-classic-vnet-support-on-march-1-2023/)объявлении об устаревании.

### <a name="can-i-enable-azure-ad-domain-services-in-an-azure-resource-manager-virtual-network"></a>Можно ли включить доменные службы Azure AD в виртуальной сети Azure Resource Manager?
Да. Доменные службы Azure AD можно включить в виртуальной сети Azure Resource Manager. Классические виртуальные сети Azure больше не доступны при создании управляемого домена.

### <a name="can-i-migrate-my-existing-managed-domain-from-a-classic-virtual-network-to-a-resource-manager-virtual-network"></a>Можно ли перенести существующий управляемый домен из классической виртуальной сети в диспетчер ресурсовную виртуальную сеть?
Да. Дополнительные сведения см. [в статье перенос доменных служб Azure AD из классической модели виртуальной сети в Диспетчер ресурсов](migrate-from-classic-vnet.md).

### <a name="can-i-enable-azure-ad-domain-services-in-an-azure-csp-cloud-solution-provider-subscription"></a>Можно ли включить доменные службы Azure AD в подписку Azure CSP (поставщик облачных решений)?
Да. Дополнительные сведения см. [в статье Включение доменных служб Azure AD в подписках Azure CSP](csp.md).

### <a name="can-i-enable-azure-ad-domain-services-in-a-federated-azure-ad-directory-i-do-not-synchronize-password-hashes-to-azure-ad-can-i-enable-azure-ad-domain-services-for-this-directory"></a>Можно ли включить доменные службы Azure AD в федеративном каталоге Azure AD? Хэши паролей не синхронизируются с Azure AD. Можно включить доменные службы Azure AD для этого каталога?
Нет. Для проверки подлинности пользователей с помощью NTLM или Kerberos доменным службам Azure AD требуется доступ к хэшам паролей учетных записей пользователей. В Федеративной папке хэши паролей не хранятся в каталоге Azure AD. Поэтому доменные службы Azure AD не работают с такими каталогами Azure AD.

Но если вы используете Azure AD Connect для синхронизации хэшей паролей, вы можете использовать доменные службы Azure AD, так как значения хэша пароля хранятся в Azure AD.

### <a name="can-i-make-azure-ad-domain-services-available-in-multiple-virtual-networks-within-my-subscription"></a>Можно ли сделать доменные службы AD Azure доступными в нескольких виртуальных сетях в рамках подписки?
Сама служба не поддерживает этот сценарий напрямую. В определенный момент времени управляемый домен доступен только в одной виртуальной сети. Однако можно настроить подключение между несколькими виртуальными сетями, чтобы предоставить доменные службы Azure AD другим виртуальным сетям. Дополнительные сведения см. в статье [подключение виртуальных сетей в Azure с помощью VPN-шлюзов](../vpn-gateway/vpn-gateway-howto-vnet-vnet-portal-classic.md) или [пиринга виртуальных сетей](../virtual-network/virtual-network-peering-overview.md).

### <a name="can-i-enable-azure-ad-domain-services-using-powershell"></a>Можно ли включить доменные службы Azure AD с помощью PowerShell?
Да. Дополнительные сведения см. [в статье Включение доменных служб Azure AD с помощью PowerShell](powershell-create-instance.md).

### <a name="can-i-enable-azure-ad-domain-services-using-a-resource-manager-template"></a>Можно ли включить доменные службы Azure AD с помощью шаблона Resource Manager?
Да, вы можете создать управляемый домен доменных служб Azure AD с помощью шаблона диспетчер ресурсов. Субъект-службу и группу Azure AD для администрирования необходимо создать с помощью портал Azure или Azure PowerShell перед развертыванием шаблона. Дополнительные сведения см. в статье [Создание управляемого домена AD DS Azure с помощью шаблона Azure Resource Manager](template-create-instance.md). При создании управляемого домена доменных служб Azure AD в портал Azure также имеется возможность экспортировать шаблон для использования с дополнительными развертываниями.

### <a name="can-i-add-domain-controllers-to-an-azure-ad-domain-services-managed-domain"></a>Могу ли я добавить контроллеры домена в управляемый домен доменных служб Azure AD?
Нет. Домен, предоставленный доменными службами Azure AD — это управляемый домен. Вам не нужно подготавливать, настраивать или иным образом управлять контроллерами домена для этого домена. Эти действия по управлению предоставляются корпорацией Майкрософт как услуга. Таким образом, невозможно добавить дополнительные контроллеры домена (для чтения и записи или только для чтения) для управляемого домена.

### <a name="can-guest-users-be-invited-to-my-directory-use-azure-ad-domain-services"></a>Можно ли приглашать гостевых пользователей в моем каталоге использовать доменные службы Azure AD?
Нет. Гостевые пользователи, приглашенные в каталог Azure AD с помощью процесса [Azure AD B2B](../active-directory/external-identities/what-is-b2b.md), синхронизируются в управляемом домене доменных служб Azure AD. Однако пароли для этих пользователей не хранятся в каталоге Azure AD. Таким образом, доменные службы Azure AD не позволяют синхронизировать хэши NTLM и Kerberos для этих пользователей с управляемым доменом. Такие пользователи не могут выполнять вход или присоединять компьютеры к управляемому домену.

### <a name="can-i-move-an-existing-azure-ad-domain-services-managed-domain-to-a-different-subscription-resource-group-region-or-virtual-network"></a>Можно ли переместить существующий управляемый домен доменных служб Azure AD в другую подписку, группу ресурсов, регион или виртуальную сеть?
Нет. После создания управляемого домена доменных служб Azure AD вы не сможете переместить управляемый домен в другую группу ресурсов, виртуальную сеть, подписку и т. д. Выбирайте наиболее подходящую подписку, группу ресурсов, регион и виртуальную сеть при развертывании управляемого домена.

### <a name="does-azure-ad-domain-services-include-high-availability-options"></a>Имеются ли в доменных службах Azure AD варианты высокой доступности?

Да. Каждый управляемый домен доменных служб Azure AD содержит два контроллера домена. Вы не управляете этими контроллерами домена и не подключаются к ним, они входят в управляемую службу. При развертывании доменных служб Azure AD в регионе, который поддерживает Зоны доступности, контроллеры домена распределяются между зонами. В регионах, которые не поддерживают Зоны доступности, контроллеры домена распределяются по наборам доступности. В этом распространении нет параметров конфигурации или управления ими. Дополнительные сведения см. [в статье параметры доступности для виртуальных машин в Azure](../virtual-machines/availability.md).

## <a name="administration-and-operations"></a>Администрирование и операции

* [Можно ли подключиться к контроллеру управляемого домена с помощью удаленного рабочего стола?](#can-i-connect-to-the-domain-controller-for-my-managed-domain-using-remote-desktop)
* [Я включил доменные службы Azure AD. Какую учетную запись пользователя следует использовать для приподключения к домену компьютеров к этому домену?](#ive-enabled-azure-ad-domain-services-what-user-account-do-i-use-to-domain-join-machines-to-this-domain)
* [Есть ли у меня привилегии администратора домена для управляемого домена, предоставляемого доменными службами AD Azure?](#do-i-have-domain-administrator-privileges-for-the-managed-domain-provided-by-azure-ad-domain-services)
* [Можно ли изменить членство в группах с помощью LDAP или других инструментов администрирования AD для управляемых доменов?](#can-i-modify-group-memberships-using-ldap-or-other-ad-administrative-tools-on-managed-domains)
* [Через какое время изменения в каталоге Azure AD будут отображаться в управляемом домене?](#how-long-does-it-take-for-changes-i-make-to-my-azure-ad-directory-to-be-visible-in-my-managed-domain)
* [Могу ли я расширить схему управляемого домена, предоставляемого доменными службами Azure AD?](#can-i-extend-the-schema-of-the-managed-domain-provided-by-azure-ad-domain-services)
* [Можно ли изменять или добавлять записи DNS в управляемом домене?](#can-i-modify-or-add-dns-records-in-my-managed-domain)
* [Что из себя представляет политика времени существования пароля в управляемом домене?](#what-is-the-password-lifetime-policy-on-a-managed-domain)
* [Предоставляют ли доменные службы Azure AD защиту учетной записи AD с помощью блокировки?](#does-azure-ad-domain-services-provide-ad-account-lockout-protection)
* [Можно ли настроить распределенная файловая система (DFS) и репликацию в доменных службах Azure AD?](#can-i-configure-distributed-file-system-and-replication-within-azure-ad-domain-services)
* [Как обновления Windows применяются в доменных службах Azure AD?](#how-are-windows-updates-applied-in-azure-ad-domain-services)

### <a name="can-i-connect-to-the-domain-controller-for-my-managed-domain-using-remote-desktop"></a>Можно ли подключиться к контроллеру управляемого домена с помощью удаленного рабочего стола?
Нет. У вас нет разрешений на подключение к контроллерам домена для управляемого домена с помощью удаленный рабочий стол. Члены группы *администраторов контроллера домена AAD* могут администрировать управляемый домен с помощью средств администрирования AD, таких как центр администрирования Active Directory (ADAC) или AD PowerShell. Эти средства устанавливаются с помощью функции *средства удаленного администрирования сервера* на сервере Windows, присоединенном к управляемому домену. Дополнительные сведения см. [в статье Создание виртуальной машины управления для настройки и администрирования управляемого домена доменных служб Azure AD](tutorial-create-management-vm.md).

### <a name="ive-enabled-azure-ad-domain-services-what-user-account-do-i-use-to-domain-join-machines-to-this-domain"></a>Я включил доменные службы Azure AD. Какой учетной записью пользователя нужно воспользоваться для подключения компьютеров к этому домену?
Любая учетная запись пользователя, входящая в управляемый домен, может быть присоединена к виртуальной машине. Членам группы " *Администраторы контроллера домена AAD* " предоставляется удаленный доступ к рабочему столу для компьютеров, которые были присоединены к управляемому домену.

### <a name="do-i-have-domain-administrator-privileges-for-the-managed-domain-provided-by-azure-ad-domain-services"></a>Есть ли у меня привилегии администратора домена для управляемого домена, предоставляемого доменными службами AD Azure?
Нет. Вы не предоставили права администратора в управляемом домене. Права администратора *домена* и *администратора предприятия* недоступны для использования в домене. Члены групп администраторов домена или администраторов предприятия в локальных Active Directory также не получают права администратора домена или предприятия в управляемом домене.

### <a name="can-i-modify-group-memberships-using-ldap-or-other-ad-administrative-tools-on-managed-domains"></a>Можно ли изменить членство в группах с помощью LDAP или других инструментов администрирования AD для управляемых доменов?
Пользователи и группы, синхронизированные из Azure Active Directory в доменные службы Azure AD, не могут быть изменены, так как их источник происхождения Azure Active Directory. Сюда входит перемещение пользователей или групп из управляемого подразделения пользователей AADDC в пользовательское подразделение. Все пользователи или группы, созданные в управляемом домене, могут быть изменены.  

### <a name="how-long-does-it-take-for-changes-i-make-to-my-azure-ad-directory-to-be-visible-in-my-managed-domain"></a>Через какое время изменения в каталоге Azure AD будут отображаться в управляемом домене?
Изменения, внесенные в каталог Azure AD с помощью пользовательского интерфейса Azure AD или PowerShell, автоматически синхронизируются с управляемым доменом. Процесс синхронизации происходит в фоновом режиме. Не определен период времени для этой синхронизации, чтобы выполнить все изменения объекта.

### <a name="can-i-extend-the-schema-of-the-managed-domain-provided-by-azure-ad-domain-services"></a>Могу ли я расширить схему управляемого домена, предоставляемого доменными службами Azure AD?
Нет. Схемой управляемого домена управляет корпорация Майкрософт. Расширения схемы не поддерживаются доменными службами Azure AD.

### <a name="can-i-modify-or-add-dns-records-in-my-managed-domain"></a>Можно ли изменять или добавлять записи DNS в управляемом домене?
Да. Членам группы *администраторов контроллера домена AAD* предоставляются права *администратора DNS* для изменения записей DNS в управляемом домене. Эти пользователи могут использовать консоль диспетчера DNS на компьютере под управлением Windows Server, присоединенном к управляемому домену, для управления DNS. Чтобы использовать консоль диспетчера DNS, установите *средства DNS-сервера*, которые являются частью *средства удаленного администрирования сервера* дополнительного компонента на сервере. Дополнительные сведения см. [в статье администрирование DNS в управляемом домене доменных служб Azure AD](manage-dns.md).

### <a name="what-is-the-password-lifetime-policy-on-a-managed-domain"></a>Что из себя представляет политика времени существования пароля в управляемом домене?
Время существования пароля по умолчанию в управляемом домене доменных служб Azure AD составляет 90 дней. Время существования этого пароля не синхронизировано со временем жизни пароля, настроенным в Azure AD. Таким образом может возникнуть ситуация, когда срок действия паролей пользователей заканчивается в управляемом домене, но они все еще действуют в Azure AD. В таких случаях пользователям требуется сменить пароль в Azure AD, после чего новый пароль синхронизируется с управляемым доменом. Если вы хотите изменить время существования пароля по умолчанию в управляемом домене, можно [создать и настроить пользовательские политики паролей.](password-policy.md)

Кроме того, политика паролей Azure AD для *DisablePasswordExpiration* синхронизируется с управляемым доменом. При применении *DisablePasswordExpiration* к пользователю в Azure AD значение *userAccountControl* для синхронизированного пользователя в управляемом домене *DONT_EXPIRE_PASSWORD* применено.

Когда пользователи сбрасывают свой пароль в Azure AD, применяется атрибут *форцечанжепассворднекстсигнин = true* . Управляемый домен синхронизирует этот атрибут из Azure AD. Если управляемый домен обнаруживает *форцечанжепассворднекстсигнин* для синхронизированного пользователя из Azure AD, атрибуту *pwdLastSet* в управляемом домене присваивается значение *0*, что делает недействительным текущий установленный пароль.

### <a name="does-azure-ad-domain-services-provide-ad-account-lockout-protection"></a>Предоставляют ли доменные службы Azure AD защиту учетной записи AD с помощью блокировки?
Да. При пяти недействительных попытках ввода пароля в течение 2 минут в управляемом домене происходит блокировка учетной записи пользователя на 30 минут. Через 30 минут учетная запись пользователя будет автоматически разблокирована. Недопустимые попытки ввода пароля в управляемом домене не блокируют учетную запись пользователя в Azure AD. Учетная запись пользователя заблокирована только в управляемом домене доменных служб Azure AD. Дополнительные сведения см. [в статье политики блокировки паролей и учетных записей в управляемых доменах](password-policy.md).

### <a name="can-i-configure-distributed-file-system-and-replication-within-azure-ad-domain-services"></a>Можно ли настроить распределенная файловая система и репликацию в доменных службах Azure AD?
Нет. Распределенная файловая система (DFS) и репликация недоступны при использовании доменных служб Azure AD.

### <a name="how-are-windows-updates-applied-in-azure-ad-domain-services"></a>Как обновления Windows применяются в доменных службах Azure AD?
Контроллеры домена в управляемом домене автоматически применяют необходимые обновления Windows. Здесь нет ничего для настройки или администрирования. Убедитесь, что не созданы правила группы безопасности сети, которые блокируют исходящий трафик для обновлений Windows. Для собственных виртуальных машин, присоединенных к управляемому домену, вы несете ответственность за настройку и применение необходимых обновлений ОС и приложений.

## <a name="billing-and-availability"></a>Выставление счетов и доступность

* [Являются ли доменные службы Azure AD платной службой?](#is-azure-ad-domain-services-a-paid-service)
* [Существует ли бесплатная пробная версия для службы?](#is-there-a-free-trial-for-the-service)
* [Можно ли приостановить управляемый домен доменных служб Azure AD?](#can-i-pause-an-azure-ad-domain-services-managed-domain)
* [Можно ли выполнить отработку отказа доменных служб Azure AD в другой регион для события аварийного восстановления?](#can-i-pause-an-azure-ad-domain-services-managed-domain)
* [Можно ли получить доменные службы Azure AD в составе Enterprise Mobility Suite (EMS)? Требуется Azure AD Premium для использования доменных служб Azure AD?](#can-i-fail-over-azure-ad-domain-services-to-another-region-for-a-dr-event)
* [В каких регионах Azure доступна служба?](#can-i-get-azure-ad-domain-services-as-part-of-enterprise-mobility-suite-ems-do-i-need-azure-ad-premium-to-use-azure-ad-domain-services)

### <a name="is-azure-ad-domain-services-a-paid-service"></a>Являются ли доменные службы Azure AD платной службой?
Да. Дополнительные сведения см. на [странице с расценками](https://azure.microsoft.com/pricing/details/active-directory-ds/).

### <a name="is-there-a-free-trial-for-the-service"></a>Существует ли бесплатная пробная версия для службы?
Доменные службы Azure AD входят в бесплатную пробную версию Azure. Вы можете зарегистрировать [бесплатную пробную версию Azure на один месяц](https://azure.microsoft.com/pricing/free-trial/).

### <a name="can-i-pause-an-azure-ad-domain-services-managed-domain"></a>Можно ли приостановить управляемый домен доменных служб Azure AD?
Нет. После включения управляемого домена доменных служб Azure AD служба будет доступна в выбранной виртуальной сети, пока не будет удален управляемый домен. Приостановить работу службы невозможно. Счета выставляются на почасовой основе, пока вы не удалите управляемый домен.

### <a name="can-i-fail-over-azure-ad-domain-services-to-another-region-for-a-dr-event"></a>Можно ли выполнить отработку отказа доменных служб Azure AD в другой регион для события аварийного восстановления?
Да, чтобы обеспечить географическую устойчивость для управляемого домена, можно создать дополнительный [набор реплик](tutorial-create-replica-set.md) для одноранговой виртуальной сети в любом регионе Azure, поддерживающем AD DS Azure. Наборы реплик совместно используют одно и то же пространство имен и конфигурацию с управляемым доменом.

### <a name="can-i-get-azure-ad-domain-services-as-part-of-enterprise-mobility-suite-ems-do-i-need-azure-ad-premium-to-use-azure-ad-domain-services"></a>Можно ли получить доменные службы Azure AD в составе Enterprise Mobility Suite (EMS)? Требуется ли Azure AD Premium для использования доменных служб Azure AD?
Нет. Доменные службы Azure AD — это служба Azure с оплатой по мере использования и не входящая в EMS. Доменные службы Azure AD можно использовать со всеми выпусками Azure AD (Free и Premium). Счет выставляется на почасовой основе в зависимости от использования.

### <a name="what-azure-regions-is-the-service-available-in"></a>В каких регионах Azure доступна служба?
Перейдите на страницу [служб Azure по регионам](https://azure.microsoft.com/regions/#services/), чтобы просмотреть список регионов Azure, в которых доступны доменные службы Azure AD.

## <a name="troubleshooting"></a>Устранение неполадок

Ознакомьтесь с нашим [руководством по устранению неполадок](troubleshoot.md) для решения распространенных проблем при настройке или администрировании доменных служб Azure AD.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о доменных службах Azure AD см. в статье [что такое Azure Active Directory доменных служб?](overview.md).

Чтобы приступить к работе, см. раздел [Создание и настройка Azure Active Directory управляемого домена доменных служб](tutorial-create-instance.md).
