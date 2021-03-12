---
title: Регистрация приложения-ресурса в Azure AD с помощью API Azure для FHIR
description: Зарегистрируйте приложение ресурсов (или API) в Azure Active Directory, чтобы клиентские приложения могли запрашивать доступ к ресурсу при проверке подлинности.
services: healthcare-apis
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: conceptual
ms.date: 02/07/2019
ms.author: matjazl
ms.openlocfilehash: 8d70a7b44893ba9c9a0cc2d1d01c65e8e1584e0f
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103020092"
---
# <a name="register-a-resource-application-in-azure-active-directory"></a>Регистрация приложения-ресурса в Azure Active Directory

В этой статье вы узнаете, как зарегистрировать приложение ресурсов (или API) в Azure Active Directory. Приложение ресурсов — это Azure Active Directory представление самого сервера API FHIR, а клиентские приложения могут запрашивать доступ к ресурсу при проверке подлинности. Приложение ресурса также называется *аудиторией* в OAuth терминах.

## <a name="azure-api-for-fhir"></a>Azure API для FHIR

При использовании API Azure для FHIR приложение ресурсов автоматически создается при развертывании службы. Пока вы используете API Azure для FHIR в том же клиенте Azure Active Directory, что и при развертывании приложения, вы можете пропустить это руководство и развернуть API Azure для FHIR, чтобы приступить к работе.

Если вы используете другой клиент Azure Active Directory (не связанный с вашей подпиской), вы можете импортировать Azure API для приложения ресурсов FHIR в клиент с помощью PowerShell:

```azurepowershell-interactive
New-AzADServicePrincipal -ApplicationId 4f6778d8-5aef-43dc-a1ff-b073724b9495
```

также можно использовать Azure CLI:

```azurecli-interactive
az ad sp create --id 4f6778d8-5aef-43dc-a1ff-b073724b9495
```

## <a name="fhir-server-for-azure"></a>Сервер FHIR для Azure

Если вы используете сервер FHIR с открытым исходным кодом для Azure, выполните действия, описанные в [репозитории GitHub](https://github.com/microsoft/fhir-server/blob/master/docs/Register-Resource-Application.md) , чтобы зарегистрировать приложение-ресурс. 

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы узнали, как зарегистрировать приложение-ресурс в Azure Active Directory. Затем зарегистрируйте конфиденциальное клиентское приложение.
 
>[!div class="nextstepaction"]
>[Регистрация конфиденциального клиентского приложения](register-confidential-azure-ad-client-app.md)