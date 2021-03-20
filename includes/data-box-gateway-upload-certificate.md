---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 10/15/2020
ms.author: alkohli
ms.openlocfilehash: 27a4d775b8d88e02be69655e5adfc6155f867ef6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96582745"
---
Правильный SSL-сертификат обеспечивает отправку зашифрованных данных на нужный сервер. Кроме шифрования, сертификат обеспечивает также проверку подлинности. Вы можете передать свой собственный доверенный SSL-сертификат через интерфейс PowerShell устройства.

1. [Подключитесь к интерфейсу PowerShell](#connect-to-the-powershell-interface).
2. Используйте `Set-HcsCertificate` командлет для отправки сертификата. При появлении запроса укажите следующие параметры:

   - `CertificateFilePath` — Путь к общей папке, содержащей файл сертификата в формате *PFX* .
   - `CertificatePassword` — Пароль, используемый для защиты сертификата.
   - `Credentials` -UserName для доступа к общей папке, содержащей сертификат. При появлении запроса укажите пароль для сетевой папки.

     В следующем примере показано использование этого командлета:

     ```
     Set-HcsCertificate -Scope LocalWebUI -CertificateFilePath "\\myfileshare\certificates\mycert.pfx" -CertificatePassword "mypassword" -Credential "Username"
     ```
