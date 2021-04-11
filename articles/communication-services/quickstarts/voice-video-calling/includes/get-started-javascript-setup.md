---
author: mikben
ms.service: azure-communication-services
ms.topic: include
ms.date: 03/10/2021
ms.author: mikben
ms.openlocfilehash: 4a493d5f0d34cd4621d55c0371036c03e267c466
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105108306"
---
## <a name="setting-up"></a>Настройка

### <a name="create-a-new-nodejs-application"></a>создание приложения Node.js;

Откройте терминал или командное окно, создайте каталог для своего приложения и перейдите к нему.

```console
mkdir calling-quickstart && cd calling-quickstart
```

Воспользуйтесь командой `npm init -y`, чтобы создать файл **package.json** с параметрами по умолчанию.

```console
npm init -y
```

### <a name="install-the-package"></a>Установка пакета

Используйте команду `npm install`, чтобы установить пакет SDK Служб коммуникации Azure для реализации вызовов на JavaScript.

```console
npm install @azure/communication-common --save
npm install @azure/communication-calling --save
```

Для работы с этим кратким руководством рекомендуется использовать следующие версии этого пакета:

```console
"webpack": "^4.42.0",
"webpack-cli": "^3.3.11",
"webpack-dev-server": "^3.10.3"
```

Параметр `--save` указывает библиотеку как зависимость в файле пакета **package.json**.

### <a name="set-up-the-app-framework"></a>Настройка платформы приложения

В этом кратком руководстве для объединения ресурсов приложения используется webpack. Выполните следующую команду, чтобы установить пакеты npm webpack, webpack-cli и webpack-dev-server, а также указать их в качестве зависимостей разработки в файле **package.json**:

```console
npm install webpack@4.42.0 webpack-cli@3.3.11 webpack-dev-server@3.10.3 --save-dev
```

Создайте файл **index.html** в корневом каталоге проекта. Мы будем использовать этот файл для настройки базового макета, с помощью которого пользователь сможет выполнить вызов.
