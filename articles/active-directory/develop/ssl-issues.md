---
title: Устранение проблем с TLS/SSL (MSAL iOS/macOS) | Службы
titleSuffix: Microsoft identity platform
description: Узнайте, что делать с различными проблемами, используя сертификаты TLS/SSL с MSAL. Библиотека цели-C.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 08/28/2019
ms.author: marsma
ms.custom: aaddev
ms.openlocfilehash: 1507231c3ab395319d5ce95ec06dbb592c324aa6
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "80881083"
---
# <a name="how-to-troubleshoot-msal-for-ios-and-macos-tlsssl-issues"></a>Как устранять неполадки MSAL для iOS и macOS TLS/SSL

В этой статье содержатся сведения, помогающие устранить проблемы, которые могут возникнуть при использовании [библиотеки проверки подлинности Майкрософт (MSAL) для iOS и macOS](reference-v2-libraries.md)

## <a name="network-issues"></a>проблемы с сетью.

**Ошибка-1200**: "произошла ошибка SSL, не удается установить безопасное соединение с сервером".

Эта ошибка означает, что соединение не защищено. Это происходит, когда сертификат недействителен. Дополнительные сведения, в том числе о том, какой сервер не проходит проверку TLS, см `NSURLErrorFailingURLErrorKey` `userInfo` . в словаре объекта ошибки.

Эта ошибка связана с сетевой библиотекой Apple. Полный список кодов ошибок NSURL находится в Нсурлеррор. h в пакетах SDK для macOS и iOS. Дополнительные сведения об этой ошибке см. в разделе [URL-Загрузка кодов системных ошибок](https://developer.apple.com/documentation/foundation/1508628-url_loading_system_error_codes?language=objc).

## <a name="certificate-issues"></a>Проблемы с сертификатом

Если URL-адрес, предоставляющий недопустимый сертификат, подключается к серверу, который планируется использовать в процессе проверки подлинности, хороший Запуск диагностики проблемы заключается в проверке URL-адреса с помощью службы проверки SSL, такой как [Проверка сервера SSL](https://www.ssllabs.com/ssltest/analyze.html). Он тестирует сервер на основе широкого спектра сценариев и браузеров и проверяет наличие многих известных уязвимостей.

По умолчанию новая функция [безопасности транспорта приложений Apple (ATS)](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html#//apple_ref/doc/uid/TP40009251-SW35) применяет более строгие политики безопасности к приложениям, ИСПОЛЬЗУЮЩИМ сертификаты TLS/SSL. Некоторые операционные системы и веб-браузеры приступили к применению некоторых из этих политик по умолчанию. По соображениям безопасности рекомендуется не отключать ATS.

Сертификаты, использующие хэши SHA-1, имеют известные уязвимости. Большинство современных веб-браузеров не разрешают сертификаты с хэш SHA-1.

## <a name="captive-portals"></a>Порталы для пере(пере)

Портал с пределами предоставляет пользователю веб-страницу при первом доступе к Wi-Fi сети и еще не предоставил доступ к этой сети. Он перехватывает трафик Интернета до тех пор, пока пользователь не будет удовлетворять требованиям портала. Сетевые ошибки, так как пользователь не может подключиться к сетевым ресурсам, пока пользователь не подключится через портал.

## <a name="next-steps"></a>Дальнейшие действия

Сведения о [порталах](https://en.wikipedia.org/wiki/Captive_portal) с пределом и новой функции [безопасности транспорта приложений Apple (ATS)](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html#//apple_ref/doc/uid/TP40009251-SW35) .
