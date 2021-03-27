---
author: IEvangelist
ms.author: dapine
ms.date: 06/25/2019
ms.service: cognitive-services
ms.topic: include
ms.openlocfilehash: 7f51b7fe95445cbd3a8aff530f89ce55b5abfb1e
ms.sourcegitcommit: a9ce1da049c019c86063acf442bb13f5a0dde213
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/27/2021
ms.locfileid: "105630121"
---
Чтобы настроить прокси-сервер HTTP для исходящих запросов, используйте следующие два аргумента.

| Имя | Тип данных | Описание |
|--|--|--|
|HTTP_PROXY|строка|Используемый прокси-сервер, например `http://proxy:8888`.<br>`<proxy-url>`|
|HTTP_PROXY_CREDS|строка|Все учетные данные, необходимые для проверки подлинности на прокси-сервере, например `username:password` . Это значение **должно быть в нижнем регистре**. |
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
HTTP_PROXY=<proxy-url> \
HTTP_PROXY_CREDS=<proxy-user>:<proxy-password> \
```
