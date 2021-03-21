---
title: включить файл
description: включить файл для целевых страниц сценария конфиденциального клиента (управляющая программа, веб-приложение, веб-API)
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.devlang: na
ms.topic: include
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 08/14/2020
ms.author: jmprieur
ms.openlocfilehash: 42102f38959911388cefcc141d949e59f24a2c31
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95996018"
---
## <a name="add-a-client-secret-or-certificate"></a>Добавление секрета клиента или сертификата

Как и в случае с любым конфиденциальным клиентским приложением, необходимо добавить секрет или сертификат, который будет использоваться в качестве *учетных данных* этого приложения, чтобы он мог проходить проверку подлинности без участия пользователя.

Учетные данные можно добавить в регистрацию клиентского приложения с помощью [портал Azure](#add-client-credentials-by-using-the-azure-portal) или с помощью программы командной строки, например [PowerShell](#add-client-credentials-by-using-powershell).

### <a name="add-client-credentials-by-using-the-azure-portal"></a>Добавление учетных данных клиента с помощью портал Azure

Чтобы добавить учетные данные в приложение конфиденциального клиентского приложения, выполните действия, описанные в разделе Краткое руководство по [регистрации приложения на платформе Microsoft Identity](../articles/active-directory/develop/quickstart-register-app.md) , для типа учетных данных, которые вы хотите добавить.

* [Добавление секрета клиента](../articles/active-directory/develop/quickstart-register-app.md#add-a-client-secret)
* [Добавление сертификата](../articles/active-directory/develop/quickstart-register-app.md#add-a-certificate)

### <a name="add-client-credentials-by-using-powershell"></a>Добавление учетных данных клиента с помощью PowerShell

Кроме того, можно добавить учетные данные при регистрации приложения на платформе Microsoft Identity с помощью PowerShell.

Пример кода [Active-Directory-команда dotnetcore-DAEMON-v2](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2) в GitHub показывает, как добавить секрет приложения или сертификат при регистрации приложения.

- Дополнительные сведения о добавлении **секрета клиента** с помощью PowerShell см. в разделе [аппкреатионскриптс/Configure.ps1](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2/blob/5199032b352a912e7cc0fce143f81664ba1a8c26/AppCreationScripts/Configure.ps1#L190).
- Дополнительные сведения о добавлении **сертификата** с помощью PowerShell см. в разделе [аппкреатионскриптс-висцерт/Configure.ps1](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2/blob/5199032b352a912e7cc0fce143f81664ba1a8c26/AppCreationScripts-withCert/Configure.ps1#L162-L178).
