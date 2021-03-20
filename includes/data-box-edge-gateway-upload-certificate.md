---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 06/26/2019
ms.author: alkohli
ms.openlocfilehash: 09d9b5bbf3f9ca7a4eef37891d03c9c865e7f74b
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "67448633"
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

