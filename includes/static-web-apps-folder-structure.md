---
author: craigshoemaker
ms.service: static-web-apps
ms.topic: include
ms.date: 03/25/2021
ms.author: cshoe
ms.openlocfilehash: 806a30214e0307b8fa101c0de51ed2f16622d433
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105543569"
---
| Свойство | Описание | Пример | Обязательно |
|---|---|---|---|
| `app_location` | Расположение кода приложения. | Введите `/`, если исходный код приложения находится в корне репозитория, или `/app`, если в каталоге с именем `app`. | Да |
| `api_location` | Расположение кода Функций Azure. | Введите `/api`, если код приложения находится в папке с именем `api`. Если в папке не обнаружено ни одного приложения Функций Azure, в процессе сборки сбой не произойдет, и в рабочем процессе предполагается, что API не нужен. | нет |
| `output_location` | Расположение выходного каталога сборки относительно `app_location`. | Если исходный код приложения находится в `/app`, а скрипт сборки выводит файлы в папку `/app/build`, установите `build` в качестве значения `output_location`. | Нет |