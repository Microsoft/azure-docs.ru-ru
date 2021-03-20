---
title: Служба регистрации Azure Active Directory применения TLS 1,2
description: Удалите поддержку TLS 1,0 и 1,1 для службы регистрации устройств Azure AD.
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: reference
ms.date: 07/10/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: spunukol
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2bb8c6c64e0a68f5176c4eb0c0177c5220394695
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89268763"
---
# <a name="enforce-tls-12-for-the-azure-ad-registration-service"></a>Принудительное применение TLS 1,2 для службы регистрации Azure AD

Служба регистрации устройств Azure Active Directory (Azure AD) используется для подключения устройств к облаку с удостоверением устройства. Сейчас служба регистрации устройств Azure AD поддерживает использование протокола TLS 1,2 для обмена данными с Azure. Чтобы обеспечить безопасность и лучшее в своем классе шифрование, корпорация Майкрософт рекомендует отключить TLS 1,0 и 1,1. В этом документе содержатся сведения о том, как убедиться, что компьютеры, используемые для завершения регистрации и взаимодействия с службой регистрации устройств Azure AD, используют TLS 1,2.

Протокол TLS версии 1,2 — это протокол шифрования, предназначенный для обеспечения безопасной связи. Протокол TLS в основном предназначен для обеспечения конфиденциальности и целостности данных. В протоколе TLS пропало много итераций с версией 1,2, определенной в [RFC 5246 (внешняя ссылка)](https://tools.ietf.org/html/rfc5246).

В текущем анализе подключений показано небольшое использование TLS 1,1 и 1,0, но мы предоставляем эти сведения, чтобы при необходимости можно было обновить все затронутые клиенты или серверы, прежде чем поддерживать TLS 1,1 и 1,0. Если вы используете локальную инфраструктуру для гибридных сценариев или службы федерации Active Directory (AD FS) (AD FS), убедитесь, что инфраструктура поддерживает как входящие, так и исходящие подключения, использующие TLS 1,2.

## <a name="update-windows-servers"></a>Обновление серверов Windows

Для серверов Windows, использующих службу регистрации устройств Azure AD или выступающих в качестве прокси-серверов, выполните следующие действия, чтобы обеспечить включение TLS 1,2.

> [!IMPORTANT]
> После обновления реестра необходимо перезапустить Windows Server, чтобы изменения вступили в силу.

### <a name="enable-tls-12"></a>Включите протокол TLS 1.2.

Убедитесь, что следующие строки реестра настроены так, как показано ниже.

- HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2 \ клиент
  - "DisabledByDefault" = DWORD: 00000000
  - "Enabled" = DWORD: 00000001
- HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2 \ сервер
  - "DisabledByDefault" = DWORD: 00000000
  - "Enabled" = DWORD: 00000001
- HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\. NETFramework\v4.0.30319
  - "SchUseStrongCrypto"=dword:00000001

## <a name="update-non-windows-proxies"></a>Обновление прокси-серверов, отличных от Windows

Все компьютеры, работающие в качестве прокси-серверов между устройствами и службой регистрации устройств Azure AD, должны обеспечить включение TLS 1,2. Следуйте указаниям поставщика, чтобы обеспечить поддержку.

## <a name="update-ad-fs-servers"></a>Обновление серверов AD FS

Все AD FS серверы, используемые для связи со службой регистрации устройств Azure AD, должны обеспечить включение TLS 1,2. Сведения о том, как включить или проверить конфигурацию, см. в разделе [Управление протоколами SSL/TLS и комплектами шифров для AD FS](/windows-server/identity/ad-fs/operations/manage-ssl-protocols-in-ad-fs) .

## <a name="client-updates"></a>Клиентские обновления

Так как все комбинации "клиент-сервер" и "браузер-сервер" должны использовать TLS 1,2 для подключения к службе регистрации устройств Azure AD, может потребоваться обновить эти устройства.

Для следующих клиентов известно, что поддержка TLS 1,2 не поддерживается. Обновите клиенты, чтобы обеспечить непрерывный доступ.

- Android версии 4,3 и более ранних версий
- Firefox версии 5,0 и более ранних
- Internet Explorer версии 8-10 в Windows 7 и более ранних версиях
- Internet Explorer 10 на Windows Phone 8,0
- Safari версии 6.0.4 в OS X 10.8.4 и более ранних версий

## <a name="next-steps"></a>Дальнейшие действия

[Обзор TLS/SSL (поставщик общих служб Schannel)](/windows-server/security/tls/tls-ssl-schannel-ssp-overview)