---
title: Запуск Integration Runtime Self-Hosted в контейнере Windows
description: Узнайте, как запускать Integration Runtime Self-Hosted в контейнере Windows.
ms.author: abnarain
author: nabhishek
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 08/05/2020
ms.openlocfilehash: e34e1f589442be41e2a4eac3ac893f377675e4b5
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100379511"
---
# <a name="how-to-run-self-hosted-integration-runtime-in-windows-container"></a>Запуск Integration Runtime Self-Hosted в контейнере Windows

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-xxx-md.md)]

В этой статье объясняется, как запустить Self-Hosted Integration Runtime в контейнере Windows.
Фабрика данных Azure предоставляет официальную поддержку контейнеров Windows Self-Hosted Integration Runtime. Вы можете скачать исходный код сборки DOCKER и объединить процесс создания и запуска в собственном конвейере непрерывной доставки. 

## <a name="prerequisites"></a>Предварительные условия 
- [Требования к контейнеру Windows](/virtualization/windowscontainers/deploy-containers/system-requirements)
- DOCKER версии 2,3 и более поздней 
- Self-Hosted Integration Runtime версии 4.11.7512.1 и более поздней 
## <a name="get-started"></a>Начало работы 
1.  Установка DOCKER и включение контейнера Windows 
2.  Загрузка исходного кода со страницы https://github.com/Azure/Azure-Data-Factory-Integration-Runtime-in-Windows-Container
3.  Скачайте последнюю версию Шир в папку "Шир" 
4.  Откройте папку в оболочке: 
```console
cd "yourFolderPath"
```

5.  Создание образа Windows docker: 
```console
docker build . -t "yourDockerImageName" 
```
6.  Запустить контейнер docker: 
```console
docker run -d -e NODE_NAME="irNodeName" -e AUTH_KEY="IR_AUTHENTICATION_KEY" -e ENABLE_HA=true HA_PORT=8060 "yourDockerImageName"    
```
> [!NOTE]
> Для этой команды AUTH_KEY является обязательным. NODE_NAME, ENABLE_HA и HA_PORT являются необязательными. Если значение не задано, команда будет использовать значения по умолчанию. Значение по умолчанию ENABLE_HA равно false, а HA_PORT — 8060.

## <a name="container-health-check"></a>Проверка работоспособности контейнера 
Через 120 секунд средство проверки работоспособности будет выполняться периодически каждые 30 секунд. Он обеспечит состояние работоспособности IR для модуля контейнера. 

## <a name="limitations"></a>Ограничения
В настоящее время не поддерживаются следующие функции при запуске Self-Hosted Integration Runtime в контейнере Windows:
- Прокси-сервер HTTP 
- Зашифрованный обмен данными между узлами с помощью TLS/SSL-сертификата 
- Создание и Импорт резервной копии 
- Служба управляющей программы 
- Автоматическое обновление 

### <a name="next-steps"></a>Дальнейшие действия
- Ознакомьтесь с [основными понятиями среды выполнения интеграции в Фабрике данных Azure](./concepts-integration-runtime.md).
- Узнайте, как [создать локальную среду выполнения интеграции на портале Azure](./create-self-hosted-integration-runtime.md).