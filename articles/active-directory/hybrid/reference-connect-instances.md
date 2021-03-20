---
title: 'Azure AD Connect: экземпляры службы синхронизации | Документация Майкрософт'
description: На этой странице приводятся специальные рекомендации для экземпляров Azure AD.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: f340ea11-8ff5-4ae6-b09d-e939c76355a3
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 05/27/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0c4d8b0a33763a967550453d8a205258f7583084
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90015268"
---
# <a name="azure-ad-connect-special-considerations-for-instances"></a>Azure AD Connect: специальные рекомендации для экземпляров
Azure AD Connect наиболее часто используется с экземпляром Azure AD и Microsoft 365 в мире мира. Но существуют и другие экземпляры, и они имеют другие требования к URL-адресам и прочие особенности.

## <a name="microsoft-cloud-germany"></a>Microsoft Cloud Germany
[Microsoft Cloud Germany](https://www.microsoft.de/cloud-deutschland) — это независимое облако, обслуживаемое управителем данными из Германии.

| URL-адреса для открытия на прокси-сервере |
| --- |
| \*.microsoftonline.de |
| \*.windows.net |
| +Списки отзыва сертификатов |

При входе в клиент Azure AD необходимо использовать учетную запись в домене onmicrosoft.de.

Функции, которые пока отсутствуют в Microsoft Cloud Germany:

* **Обратная запись паролей** доступна для предварительного просмотра с Azure AD Connect версии 1.1.570.0 и после.
* Другие службы Azure AD Premium недоступны.

## <a name="microsoft-azure-government"></a>Azure для государственных организаций
[Облако Microsoft Azure для государственных организаций](https://azure.microsoft.com/features/gov/) — это облако для правительства США.

Это облако поддерживается в более ранних выпусках DirSync. Начиная со сборки 1.1.180 Azure AD Connect поддерживается следующее поколение облака. Это поколение использует базирующиеся только в США конечные точки и имеет другой список URL-адресов для открытия на прокси-сервере.

| URL-адреса для открытия на прокси-сервере |
| --- |
| \*.microsoftonline.com |
| \*.microsoftonline.us |
| \*. windows.net (требуется для автоматического обнаружения клиентов Azure для государственных организаций) |
| \*.gov.us.microsoftonline.com |
| +Списки отзыва сертификатов |

> [!NOTE]
> Начиная с версии Azure AD Connect 1.1.647.0 Установка значения Азуреинстанце в реестре больше не требуется, если на прокси-серверах открыт параметр *. windows.net. Однако для клиентов, которые не разрешают подключение к Интернету со своих Azure AD Connect серверов, можно использовать следующую конфигурацию вручную.

### <a name="manual-configuration"></a>Настроить вручную

Чтобы убедиться, Azure AD Connect использует конечные точки синхронизации Azure для государственных организаций, выполните следующие действия по настройке вручную.

1. Начните установку Azure AD Connect
2. Когда отобразится первая страница, на которой предлагается принять условия лицензионного соглашения, не продолжайте, но и не прерывайте работу мастера установки.
3. Запустите редактор реестра (regedit) и измените раздел реестра `HKLM\SOFTWARE\Microsoft\Azure AD Connect\AzureInstance`, присвоив ему значение `4`.
4. Вернитесь в мастер установки Azure AD Connect, примите условия лицензионного соглашения и перейдите к следующему шагу. Во время установки убедитесь, что используется путь установки **настраиваемой конфигурации** (а не Экспресс-установка), а затем продолжайте установку обычным образом.

## <a name="next-steps"></a>Дальнейшие действия
Узнайте больше об [интеграции локальных удостоверений с Azure Active Directory](whatis-hybrid-identity.md).
