---
author: dlepow
ms.service: virtual-machines-linux
ms.topic: include
ms.date: 07/10/2019
ms.author: danlep
ms.openlocfilehash: 2b28cc742bb981e0db535d39cb0c7fbf027470ea
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "83673770"
---
### <a name="identity-tier"></a>Уровень идентификации 

Партнерство корпорации Майкрософт и Oracle позволяет настроить единую идентификацию в Azure, OCI и приложении Oracle. Для настройки единого входа между Azure AD и Oracle набору приложений JD Edwards EnterpriseOne или PeopleSoft необходим экземпляр сервера Oracle HTTP Server (OHS).

OHS действует как обратный прокси-сервер для уровня приложения. Это означает, что все запросы к конечным приложениям проходят через него. Oracle Access Manager WebGate — это подключаемый модуль веб-сервера OHS, который перехватывает все запросы к конечному приложению. Если доступ к ресурсу защищен (требуется сеанс с аутентификацией), то WebGate инициирует поток аутентификации OIDC с помощью службы Identity Cloud Service через браузер пользователя. Дополнительные сведения о потоках, поддерживаемых OpenID Connect WebGate, см. в [документации по Oracle Access Manager](https://docs.oracle.com/cd/E52734_01/oam/AIAAG/GUID-1E927D1B-FB83-425B-8768-85DB441821A4.htm#AIAAG7327).

При такой конфигурации пользователь, уже вошедший в Azure AD, может перейти к приложению JD Edwards EnterpriseOne или PeopleSoft без повторного входа благодаря службе Oracle Identity Cloud Service. Клиенты, которые развертывают это решение, получают преимущества единого входа, в том числе единый набор учетных данных, улучшенный интерфейс входа в систему, повышенный уровень безопасности и сокращение затрат на службу технической поддержки.

Чтобы узнать больше о настройке единого входа для JD Edwards EnterpriseOne или PeopleSoft с помощью Azure AD, ознакомьтесь с соответствующим [техническим документом Oracle](https://cloud.oracle.com/iaas/whitepapers/deploy_peoplesoft_jdedwards_across_oci_azure.pdf).