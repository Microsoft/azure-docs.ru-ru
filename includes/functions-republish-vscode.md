---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 05/20/2020
ms.author: glenga
ms.openlocfilehash: 574756aa9d9bac4aa42febf6a4fca4c62014db3f
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96010466"
---
1. В Visual Studio Code нажмите клавишу F1, чтобы открыть палитру команд. В палитре команд найдите и щелкните **Azure Functions: Deploy to Function App** (Функции Azure: развертывание приложения-функции).

1. Если вы еще не вошли в систему, вам будет предложено **войти в Azure**. После входа из браузера вернитесь в Visual Studio Code. Если у вас несколько подписок, **выберите подписку** со своим приложением-функцией.

1. Выберите существующее приложение-функцию в Azure. Если отобразится предупреждение о перезаписи всех файлов в приложении-функции, выберите **Deploy** (Развернуть), чтобы подтвердить ознакомление с предупреждением и продолжить.

Проект будет пересобран, переупакован и отправлен в Azure. Существующий проект заменяется новым пакетом, а приложение-функция перезапускается.