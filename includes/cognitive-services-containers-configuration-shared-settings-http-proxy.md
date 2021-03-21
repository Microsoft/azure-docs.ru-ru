---
author: IEvangelist
ms.author: dapine
ms.date: 06/25/2019
ms.service: cognitive-services
ms.topic: include
ms.openlocfilehash: ce4cc68826b39b5707549afc799d2d214e8876c6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96001207"
---
Чтобы настроить прокси-сервер HTTP для исходящих запросов, используйте следующие два аргумента.

| Имя | Тип данных | Описание |
|--|--|--|
|HTTPS_PROXY|строка|Используемый прокси-сервер, например `https://proxy:8888`.<br>`<proxy-url>`|
|HTTPS_PROXY_CREDS|строка|Любые учетные данные, необходимые для выполнения аутентификации на прокси-сервере, например username:password.|
|`<proxy-user>`|строка|Пользователь прокси-сервера.|
|`<proxy-password>`|строка|Пароль, связанный с параметром `<proxy-user>` прокси-сервера.|
||||


```bash
docker run --rm -it -p 5000:5000 \
--memory 2g --cpus 1 \
--mount type=bind,src=/home/azureuser/output,target=/output \
<registry-location>/<image-name> \
Eula=accept \
Billing=<endpoint> \
ApiKey=<api-key> \
HTTPS_PROXY=<proxy-url> \
HTTPS_PROXY_CREDS=<proxy-user>:<proxy-password> \
```
