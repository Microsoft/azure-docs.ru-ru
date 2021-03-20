---
title: Обновления TLS концентраторов уведомлений
description: Узнайте о поддержке TLS в центрах уведомлений Azure.
services: notification-hubs
documentationcenter: .net
author: sethmanheim
manager: femila
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-multiple
ms.devlang: multiple
ms.topic: article
ms.date: 04/29/2020
ms.author: sethm
ms.reviewer: thsomasu
ms.lastreviewed: 01/28/2020
ms.openlocfilehash: 87a3627d7820f9f456ac08e2f20b70af961f817e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87084251"
---
# <a name="transport-layer-security-tls"></a>Протокол TLS

Для обеспечения более высокого уровня безопасности концентраторы уведомлений отключают поддержку TLS версий 1,0 и 1,1 **31 декабря 2020** (продлено с 30 апреля 2020 г.). Эти старые протоколы обеспечивают слабую криптографию и уязвимы для атак МОНСТРУ и пудель. Эти изменения не влияют на приложения, развернутые на устройствах под управлением Android версии 5 или более поздней версии или iOS с версией 5 или более поздней, так как эти операционные системы поддерживают протокол TLS 1,2, а клиент и сервер будут согласовывать наивысшую взаимоподдерживаемую версию протокола при подключении.

Рекомендуется просматривать все приложения, использующие центры уведомлений Azure, чтобы убедиться, что они используют наиболее подходящие библиотеки и стеки TLS, поддерживающие TLS 1,2.

## <a name="update-apps"></a>Обновление приложений

Вы можете убедиться, что приложения iOS используют TLS 1,2 с помощью функции безопасности сети Apple, именуемой безопасностью транспорта приложений (ATS). ATS нельзя использовать для пакетов SDK старше iOS 9,0 или macOS 10,11. Дополнительные сведения см. в [документации Apple](https://developer.apple.com/documentation/security/preventing_insecure_network_connections).

Для приложений Android, использующих экземпляры Сслсоккет, установите включенные протоколы для каждого экземпляра Сслсоккет, как указано в [сетенабледпротоколс](https://developer.android.com/reference/javax/net/ssl/SSLSocket#setEnabledProtocols(java.lang.String%5B%5D)).

Таблица на странице Поддержка [совместимости протокола TLS](https://support.globalsign.com/customer/portal/articles/2934392-tls-protocol-compatibility) помогает сопоставлять операционные системы с совместимыми версиями TLS.

Дополнительные сведения см. в обзоре [поддержки протоколов TLS в Windows](/archive/blogs/kaushal/support-for-ssltls-protocols-on-windows).

## <a name="next-steps"></a>Дальнейшие действия

- [Общие сведения о концентраторах уведомлений](notification-hubs-push-notification-overview.md)
