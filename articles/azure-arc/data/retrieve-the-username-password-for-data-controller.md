---
title: Получение имени пользователя и пароля для подключения к контроллеру данных Arc
description: Получение имени пользователя и пароля для подключения к контроллеру данных Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 8cabb48e3620f1a17d1bb9b87e1646ce2793143b
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91761708"
---
# <a name="retrieve-the-user-name-and-password-to-connect-to-the-arc-data-controller"></a>Получение имени пользователя и пароля для подключения к контроллеру данных Arc

Может возникнуть ситуация, когда необходимо получить имя пользователя и пароль для контроллера данных. Это команды, необходимые при запуске. 

```console
azdata login
```

Если вы являетесь администратором Kubernetes для кластера. Таким образом, у вас есть права на выполнение команд для получения из секрета Kubernetes, в котором хранятся сведения, хранящиеся в службе "Дуга Azure".

> [!NOTE]
>  Если вы использовали другое имя для пространства имен, в котором был создан контроллер данных, измените `-n arc` параметр в приведенных ниже командах, чтобы использовать имя пространства имен, в которое был создан контроллер данных.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="linux"></a>Linux

Выполните следующую команду, чтобы получить имя пользователя:

```console
kubectl -n arc get secret controller-login-secret -o=jsonpath="{.data['username']}" | base64 -d
```

Чтобы получить пароль, выполните следующую команду:

```console
kubectl -n arc get secret controller-login-secret -o=jsonpath="{.data['password']}" | base64 -d
```

## <a name="powershell"></a>PowerShell

Выполните следующую команду, чтобы получить имя пользователя:

```console
[Text.Encoding]::Utf8.GetString([Convert]::FromBase64String((kubectl -n arc get secret controller-login-secret -o=jsonpath="{.data['username']}")))
```

Чтобы получить пароль, выполните следующую команду:

```console
[Text.Encoding]::Utf8.GetString([Convert]::FromBase64String((kubectl -n arc get secret controller-login-secret -o=jsonpath="{.data['password']}")))
```

## <a name="next-steps"></a>Дальнейшие действия

Попробуйте другие [сценарии](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/active-directory-domain-services/scenarios.md)
