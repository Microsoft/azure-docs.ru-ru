---
author: IEvangelist
ms.author: dapine
ms.date: 08/22/2019
ms.service: cognitive-services
ms.topic: include
ms.openlocfilehash: 8b3d856d255968b4a6736db908ce3999cbd56193
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95996468"
---
## <a name="validate-that-a-container-is-running"></a>Проверка состояния контейнера 

Проверить это можно несколькими способами. Перейдите к *внешнему IP-* адресу и предоставленному порту рассматриваемого контейнера и откройте свой любимый веб-браузер. Используйте различные URL-адреса запросов ниже, чтобы проверить, выполняется контейнер. Примеры URL-адресов запросов приведены ниже `http://localhost:5000` , но конкретный контейнер может отличаться. Помните, что вам нужно полагаться на *внешний IP-* адрес контейнера и доступный порт.

| URL-адрес запроса | Назначение |
|--|--|
| `http://localhost:5000/` | Контейнер предоставляет домашнюю страницу. |
| `http://localhost:5000/ready` | Запрашивается с помощью GET, это обеспечивает проверку готовности контейнера к принятию запроса к модели.  Этот запрос может использоваться для [проб активности и готовности](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/) Kubernetes. |
| `http://localhost:5000/status` | Кроме того, запрос с помощью GET проверяет, является ли ключ API, используемый для запуска контейнера, допустимым, не вызывая запрос конечной точки. Этот запрос может использоваться для [проб активности и готовности](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/) Kubernetes. |
| `http://localhost:5000/swagger` | Контейнер предоставляет полный набор документации по конечным точкам и функции **Попробовать**. Эта функция позволяет ввести параметры в веб-форму HTML и создать запрос без необходимости писать код. После возвращения результатов запроса предоставляется пример команды CURL с примером требуемого формата HTTP-заголовков и текста. |

![Домашняя страница контейнера](./media/cognitive-services-containers-api-documentation/container-webpage.png)
