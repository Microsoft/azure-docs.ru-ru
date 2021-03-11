---
title: Поддержка браузера FIDO2 проверки подлинности с паролем | Azure Active Directory
description: Браузеры и сочетания операционных систем поддерживают FIDO2 проверку подлинности с паролем для приложений, использующих Azure Active Directory
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 02/02/2021
ms.author: nichola
author: knicholasa
manager: martinco
ms.reviewer: ''
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3f90edd5729ff5229be09bc3798082c33bdeead2
ms.sourcegitcommit: b572ce40f979ebfb75e1039b95cea7fce1a83452
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "102632107"
---
# <a name="browser-support-of-fido2-passwordless-authentication"></a>Поддержка браузера FIDO2 проверки подлинности с паролем

Azure Active Directory позволяет использовать [ключи безопасности FIDO2](./concept-authentication-passwordless.md#fido2-security-keys) в качестве устройства, не использующего пароль. Доступность FIDO2 проверки подлинности для учетных записей Майкрософт была [объявлена в 2018](https://techcommunity.microsoft.com/t5/identity-standards-blog/all-about-fido2-ctap2-and-webauthn/ba-p/288910). Как обсуждалось в объявлении, для поддержки безопасной проверки подлинности с учетными записями Майкрософт и Azure Active Directory необходимо реализовать некоторые дополнительные функции и расширения спецификации FIDO2 CTAP. На следующей схеме показано, какие сочетания браузеров и операционных систем поддерживают проверку подлинности без пароля с помощью ключей проверки подлинности FIDO2 с Azure Active Directory.

## <a name="supported-browsers"></a>Поддерживаемые браузеры

В этой таблице представлена поддержка проверки подлинности Azure Active Directory (Azure AD) и учетных записей Майкрософт (MSA). Учетные записи Майкрософт создаются потребителями для таких служб, как Xbox, Skype или Outlook.com. Поддерживаются следующие типы устройств: **USB**, связь ближнего поля (**NFC**) и Bluetooth Low Energy (**BLE**).

| ОС | Chrome | Chrome  | Chrome | Edge | Edge | Edge | Firefox | Firefox | Firefox |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| | USB | NFC | BLE | USB | NFC | BLE | USB | NFC | BLE |
| **Windows**  | ![Chrome поддерживает USB в Windows для учетных записей AAD.][y] | ![Chrome поддерживает использование NFC в Windows для учетных записей AAD.][y] | ![Chrome поддерживает BLE в Windows для учетных записей AAD.][y] | ![Ребро поддерживает USB в Windows для учетных записей AAD.][y] | ![Ребро поддерживает использование NFC в Windows для учетных записей AAD.][y] | ![Ребро поддерживает BLE в Windows для учетных записей AAD.][y] | ![Firefox поддерживает USB в Windows для учетных записей AAD.][y] | ![Firefox поддерживает использование NFC в Windows для учетных записей AAD.][y] | ![Firefox поддерживает BLE в Windows для учетных записей AAD.][y] |
| **macOS**  | ![Chrome поддерживает USB на macOS для учетных записей AAD.][y] | ![Chrome не поддерживает NFC в macOS для учетных записей AAD.][n] | ![Chrome не поддерживает BLE для учетных записей AAD в macOS.][n] | ![Ребро поддерживает USB на macOS для учетных записей AAD.][y] | ![Ребро не поддерживает NFC в macOS для учетных записей AAD.][n] | ![Ребро не поддерживает BLE на macOS для учетных записей AAD.][n] | ![Firefox не поддерживает USB на macOS для учетных записей AAD.][n] | ![Firefox не поддерживает NFC в macOS для учетных записей AAD.][n] | ![Firefox не поддерживает BLE для учетных записей AAD в macOS.][n] |
| **Linux**  | ![Chrome поддерживает USB на платформе Linux для учетных записей AAD.][y] | ![Chrome не поддерживает NFC в Linux для учетных записей AAD.][n] | ![Chrome не поддерживает BLE в Linux для учетных записей AAD.][n] | ![Ребро не поддерживает USB в Linux для учетных записей AAD.][n] | ![Ребро не поддерживает использование NFC в Linux для учетных записей AAD.][n] | ![Ребро не поддерживает BLE в Linux для учетных записей AAD.][n] | ![Firefox не поддерживает поддержку USB в Linux для учетных записей AAD.][n] | ![Firefox не поддерживает использование NFC в Linux для учетных записей AAD.][n] | ![Firefox не поддерживает BLE в Linux для учетных записей AAD.][n] |



## <a name="unsupported-browsers"></a>Неподдерживаемые браузеры

Следующие сочетания операционных систем и браузеров не поддерживаются, однако исследование и тестирование осуществляется в будущем. Если вы хотите увидеть другие поддерживаемые операционные системы и браузеры, оставьте отзыв, используя средство обратной связи с продуктом в нижней части страницы.

| Операционная система | Браузер |
| ---- | ---- |
| iOS | Safari, дивный |
| macOS | Safari |
| Android | Chrome |
| чромеос | Chrome |

## <a name="minimum-browser-version"></a>Минимальная версия браузера

Ниже приведены минимальные требования к версии браузера. 

| Браузер | Минимальная версия |
| ---- | ---- |
| Chrome | 76 |
| Edge | Windows 10, версия 1903<sup>1</sup> |
| Firefox | 66 |

<sup>1</sup> Все версии новой Chromium поддержки Microsoft ребра на основе Fido2. Поддержка в версии Microsoft ребра Legacy была добавлена в 1903.

## <a name="next-steps"></a>Дальнейшие действия
[Включить вход в систему без пароля (Предварительная версия)](./howto-authentication-passwordless-security-key.md)

<!--Image references-->
[y]: ./media/fido2-compatibility/yes.png
[n]: ./media/fido2-compatibility/no.png
