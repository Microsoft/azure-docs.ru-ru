---
title: Устранение неполадок защищенного протокола LDAP в доменных службах Azure AD | Документация Майкрософт
description: Узнайте, как устранять неполадки защищенного протокола LDAP (LDAPs) для управляемого домена доменных служб Azure Active Directory
services: active-directory-ds
author: justinha
manager: daveba
ms.assetid: 445c60da-e115-447b-841d-96739975bdf6
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: troubleshooting
ms.date: 07/09/2020
ms.author: justinha
ms.openlocfilehash: e21ec7d8fdf7a5c7a8ba7a9f22aa5709581cbae6
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96618405"
---
# <a name="troubleshoot-secure-ldap-connectivity-issues-to-an-azure-active-directory-domain-services-managed-domain"></a>Устранение проблем с защищенным подключением LDAP к управляемому домену доменных служб Azure Active Directory

Приложения и службы, использующие протокол LDAP для взаимодействия с доменными службами Azure Active Directory (Azure AD DS), можно [настроить для использования защищенного протокола LDAP](tutorial-configure-ldaps.md). Чтобы безопасный протокол LDAP работал правильно, должен быть открыт соответствующий сертификат и необходимые сетевые порты.

Эта статья поможет вам устранить проблемы с защищенным доступом LDAP в Azure AD DS.

## <a name="common-connection-issues"></a>Распространенные неполадки с подключением

При возникновении проблем с подключением к управляемому домену Azure AD DS с помощью защищенного протокола LDAP ознакомьтесь со следующими шагами по устранению неполадок. После каждого шага устранения неполадок попробуйте подключиться к управляемому домену еще раз:

* Цепочка издателей сертификата защищенного протокола LDAP должна быть доверенной в клиенте. Корневой центр сертификации (ЦС) можно добавить в хранилище доверенных корневых сертификатов на клиенте, чтобы установить доверие.
    * Обязательно [экспортируйте и примените сертификат на клиентских компьютерах][client-cert].
* Убедитесь, что сертификат защищенного протокола LDAP для управляемого домена имеет DNS-имя в *субъекте* или в атрибуте " *альтернативные имена субъектов* ".
    * Ознакомьтесь с [требованиями к защищенному сертификату LDAP][certs-prereqs] и при необходимости создайте сертификат для замены.
* Убедитесь, что клиент LDAP, например *ldp.exe* , подключается к конечной точке ЗАЩИЩЕННОГО протокола LDAP, используя DNS-имя, а не IP-адрес.
    * Сертификат, применяемый к управляемому домену, не включает IP-адреса службы, а только DNS-имена.
* Проверьте имя DNS, к которому подключается клиент LDAP. Оно должно разрешаться в общедоступный IP-адрес для защищенного протокола LDAP в управляемом домене.
    * Если DNS-имя разрешается во внутренний IP-адрес, обновите запись DNS для разрешения на внешний IP-адрес.
* Для внешних подключений группа безопасности сети должна включать правило, разрешающее трафик через TCP-порт 636 из Интернета.
    * Если вы можете подключиться к управляемому домену с помощью защищенного протокола LDAP из ресурсов, непосредственно подключенных к виртуальной сети, но не с внешними подключениями, убедитесь, что [создано правило группы безопасности сети, разрешающее безопасный трафик LDAP][ldaps-nsg].

## <a name="next-steps"></a>Дальнейшие действия

Если проблемы сохраняются, [откройте запрос на поддержку в Azure][azure-support] для получения дополнительной помощи в устранении неполадок.

<!-- INTERNAL LINKS -->
[azure-support]: ../active-directory/fundamentals/active-directory-troubleshooting-support-howto.md
[configure-ldaps]: tutorial-configure-ldaps.md
[certs-prereqs]: tutorial-configure-ldaps.md#create-a-certificate-for-secure-ldap
[client-cert]: tutorial-configure-ldaps.md#export-a-certificate-for-client-computers
[ldaps-nsg]: tutorial-configure-ldaps.md#lock-down-secure-ldap-access-over-the-internet
