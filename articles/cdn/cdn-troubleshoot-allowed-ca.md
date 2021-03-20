---
title: Разрешенный ЦС для включения пользовательского HTTPS
titleSuffix: Azure Content Delivery Network
description: Если вы используете собственный сертификат для включения протокола HTTPS в пользовательском домене, для его создания необходимо использовать разрешенный центр сертификации (ЦС).
services: cdn
author: asudbring
ms.service: azure-cdn
ms.topic: troubleshooting
ms.date: 02/04/2021
ms.author: allensu
ms.custom: mvc
ms.openlocfilehash: f98e28c89fa70831108cfbbbaca6e2f316d1b039
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99573404"
---
# <a name="allowed-certificate-authorities-for-enabling-custom-https"></a>Разрешенные центры сертификации для включения пользовательского HTTPS

Требования к сертификатам необходимы при [включении функции HTTPS с помощью собственного сертификата](cdn-custom-ssl.md?tabs=option-2-enable-https-with-your-own-certificate#tlsssl-certificates) для Azure CDN (сети доставки содержимого). 

* **Azure CDN стандарт из профиля Майкрософт** требует сертификат от одного из утвержденных центров сертификации (CA) в следующем списке. Если используется сертификат из неутвержденного ЦС или самозаверяющий сертификат, запрос отклоняется. 

* **Azure CDN Standard от Verizon** и **Azure CDN Premium из профилей Verizon** принимают любой действительный сертификат от любого допустимого ЦС. Профили Verizon не поддерживают самозаверяющие сертификаты.

> [!NOTE]
> Возможность использования собственного сертификата для включения функции HTTPS пользовательского домена *недоступна* для **Azure CDN Standard из профилей Akamai** . 
>

[!INCLUDE [cdn-front-door-allowed-ca](../../includes/cdn-front-door-allowed-ca.md)]

