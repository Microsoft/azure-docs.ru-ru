---
title: Обновление учетных данных субъекта-службы
description: Обновление учетных данных для субъекта-службы
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: dnethi
ms.author: mikeray
ms.reviewer: mikeray
ms.date: 12/09/2020
ms.topic: how-to
ms.openlocfilehash: 1c7ccec6228a6dcfb457bda8f6ecd384775d4b27
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104669548"
---
# <a name="update-service-principal-credentials"></a>Обновление учетных данных субъекта-службы

При изменении учетных данных субъекта-службы необходимо обновить секреты в контроллере данных.

Например, если вы развернули контроллер данных с помощью определенного набора значений для идентификатора клиента субъекта-службы, идентификатора клиента и секрета клиента, а затем измените одно или несколько из этих значений, необходимо обновить секреты в контроллере данных.  Ниже приведены инструкции по обновлению идентификатора клиента, идентификатора клиента или секрета клиента. 

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="background"></a>Историческая справка

Субъект-служба был создан при [создании субъекта-службы](upload-metrics-and-logs-to-azure-monitor.md#create-service-principal). 

## <a name="steps"></a>Шаги

1. Доступ к секрету субъекта-службы в редакторе по умолчанию.

   ```console
   kubectl edit secret/upload-service-principal-secret -n <name of namespace>
   ```

   Например, чтобы изменить секрет субъекта-службы на контроллере данных в `arc` пространстве имен, выполните следующую команду:

   ```console
   kubectl edit secret/upload-service-principal-secret -n arc
   ```

   `kubecl edit`Команда открывает файл Credentials. yml в редакторе по умолчанию. 


1. Измените секрет субъекта-службы. 

   В редакторе по умолчанию замените значения в разделе данных обновленными учетными данными.

   Например:

   ```console
   # Please edit the object below. Lines beginning with a '#' will be ignored,
   # and an empty file will abort the edit. If an error occurs while saving this file will be
   # reopened with the relevant failures.
   #
   apiVersion: v1
   data:
     authority: aHR0cHM6Ly9sb2dpbi5taWNyb3NvZnRvbmxpbmUuY29t
     clientId: NDNiNDcwYrFTGWYzOC00ODhkLTk0ZDYtNTc0MTdkN2YxM2Uw
     clientSecret: VFA2RH125XU2MF9+VVhXenZTZVdLdECXFlNKZi00Lm9NSw==
     tenantId: NzJmOTg4YmYtODZmMRFVBGTJLSATkxYWItMmQ3Y2QwMTFkYjQ3
   kind: Secret
   metadata:
     creationTimestamp: "2020-12-02T05:02:04Z"
     name: upload-service-principal-secret
     namespace: arc
     resourceVersion: "7235659"
     selfLink: /api/v1/namespaces/arc/secrets/upload-service-principal-secret
     uid: 7fb693ff-6caa-4a31-b83e-9bf22be4c112
   type: Opaque
   ```

   Измените значения для `clientID` , `clientSecret` и (или) `tenantID` соответственно. 

> [!NOTE]
>Значения должны быть в кодировке Base64. Не изменяйте другие свойства.

Если для указано неверное значение `clientId` или, то `clientSecret` `tenantID` в `control-xxxx` журналах контейнера Pod/Controller появится сообщение об ошибке, как показано ниже.

```output
YYYY-MM-DD HH:MM:SS.mmmm | ERROR | [AzureUpload] Upload task exception: A configuration issue is preventing authentication - check the error message from the server for details.You can modify the configuration in the application registration portal. See https://aka.ms/msal-net-invalid-client for details.  Original exception: AADSTS7000215: Invalid client secret is provided.
```



## <a name="next-steps"></a>Дальнейшие действия

[Получение имени пользователя и пароля для подключения к контроллеру данных Arc](retrieve-the-username-password-for-data-controller.md)

[Создание субъекта-службы](upload-metrics-and-logs-to-azure-monitor.md#create-service-principal)
