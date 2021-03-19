---
title: Общие сведения о службе KUDU
description: Узнайте о механизме, который обеспечивает непрерывное развертывание в службе приложений и ее функциях.
ms.date: 03/17/2021
ms.topic: reference
ms.openlocfilehash: ad1456f1ca123127f3d84aa8195c99fc34872aee
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104609634"
---
# <a name="kudu-service-overview"></a>Общие сведения о службе KUDU

KUDU — это подсистема для ряда функций [службы приложений Azure](overview.md) , относящаяся к развертыванию на основе системы управления версиями, и другие методы развертывания, такие как синхронизация Dropbox и OneDrive. 

## <a name="access-kudu-for-your-app"></a>Доступ к KUDU для вашего приложения
Когда вы создаете приложение, служба приложений создает для него вспомогательное приложение, защищенное по протоколу HTTPS. Это приложение KUDU доступно по адресу:

- Приложение не находится на изолированном уровне: `https://<app-name>.scm.azurewebsites.net`
- Приложение на изолированном уровне (Среда службы приложений): `https://<app-name>.scm.<ase-name>.p.azurewebsites.net`

Дополнительные сведения см. [в разделе доступ к службе KUDU](https://github.com/projectkudu/kudu/wiki/Accessing-the-kudu-service).

## <a name="kudu-features"></a>Функции KUDU

KUDU предоставляет полезную информацию о приложении службы приложений, например:

- Параметры приложения
- Строки подключения
- Переменные среды
- Переменные сервера
- Заголовки HTTP

Он также предоставляет другие функции, такие как:

- Выполните команды в [консоли KUDU](https://github.com/projectkudu/kudu/wiki/Kudu-console).
- Загрузите диагностические дампы IIS или журналы DOCKER.
- Управление процессами и расширениями сайтов IIS.
- Добавление веб-перехватчиков развертывания для точек доступа Windows.
- Разрешите пользовательский интерфейс развертывания ZIP с помощью `/ZipDeploy` .
- Создает [пользовательские скрипты развертывания](https://github.com/projectkudu/kudu/wiki/Custom-Deployment-Script).
- Разрешает доступ с [REST API](https://github.com/projectkudu/kudu/wiki/REST-API).

## <a name="more-resources"></a>Дополнительные ресурсы

KUDU — это [проект с открытым исходным кодом](https://github.com/projectkudu/kudu), который содержит документацию по [KUDU wiki](https://github.com/projectkudu/kudu/wiki).

