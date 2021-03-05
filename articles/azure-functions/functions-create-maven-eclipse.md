---
title: Создание приложения функции Azure с помощью Java и Eclipse
description: Практическое руководство по созданию простого бессерверного HTTP-приложения с использованием Java и Eclipse и его публикации в решении "Функции Azure".
author: jeffhollan
ms.topic: how-to
ms.date: 07/01/2018
ms.author: jehollan
ms.custom: mvc, devcenter, devx-track-java
ms.openlocfilehash: 7dd881d130b9df19335ac64be501553af99d58d8
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102179550"
---
# <a name="create-your-first-function-with-java-and-eclipse"></a>Создание первой функции с помощью Java и Eclipse 

В этой статье показано, как создать проект [бессерверной](https://azure.microsoft.com/solutions/serverless/) функции с помощью Eclipse IDE и Apache Maven, протестировать и отладить его и развернуть в решении "Функции Azure". 

<!-- TODO ![Access a Hello World function from the command line with cURL](media/functions-create-java-maven/hello-azure.png) -->

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="set-up-your-development-environment"></a>Настройка среды разработки

Для разработки приложения-функции с помощью Java и Eclipse должны быть установлены следующие компоненты:

-  [Пакет Java Developer Kit](https://www.azul.com/downloads/zulu/), версия 8.
-  [Apache Maven](https://maven.apache.org) 3.0 или более поздней версии.
-  [Eclipse](https://www.eclipse.org/downloads/packages/) с поддержкой Java и Maven.
-  [Azure CLI](/cli/azure)

> [!IMPORTANT] 
> Переменной среде JAVA_HOME необходимо присвоить расположение установки JDK, чтобы завершить выполнение заданий этого краткого руководства.

Рекомендуется установить [основные инструменты решения "Функции Azure" версии 2](functions-run-local.md#v2), которые предоставляют локальную среду для выполнения и отладки Функций Azure. 

## <a name="create-a-functions-project"></a>Создание проекта Функций Azure

1. В Eclipse откройте меню **файл** и выберите **создать- &gt; Maven проект**. 
1. Примите значения по умолчанию в диалоговом окне **New Maven Project** (Новый проект Maven) и щелкните **Next** (Далее).
1. Найдите и выберите [Azure-functions-архетипа](https://mvnrepository.com/artifact/com.microsoft.azure/azure-functions-archetype) и нажмите кнопку **Далее**.
1. Обязательно заполните значения для всех полей `resourceGroup` , включая, `appName` и `appRegion` (используйте другое AppName, отличное от **fabrikam-Function-20170920120101928**), и в конечном итоге **завершите работу**.
    ![Eclipse Maven Create2](media/functions-create-first-java-eclipse/functions-create-eclipse2.png)  

Maven создает файлы проекта в новой папке с именем _artifactId_. Созданный код проекта представляет собой простую функцию[активации HTTP](./functions-bindings-http-webhook.md), которая возвращает текст HTTP-запроса.

## <a name="run-functions-locally-in-the-ide"></a>Запуск функций в локальной среде IDE

> [!NOTE]
> Для локальных запуска и отладки функций, нужно установить [Azure Functions Core Tools версии 2](functions-run-local.md#v2).

1. Щелкните правой кнопкой мыши созданный проект, а затем выберите **Run As** (Выполнить как) и **Maven build** (Сборка Maven).
1. В диалоговом окне **Edit Configuration** (Изменение конфигурации) введите `package` в поля **Goals** (Целевые объекты) и **Name** (Имя), затем щелкните **Run** (Выполнить). В результате будет создан и упакован код функции.
1. После завершения сборки создайте другую конфигурацию запуска так, как показано выше, используя `azure-functions:run` для целевого объекта и имени. Щелкните **Run** (Выполнить), чтобы запустить функцию в интегрированной среде разработки.

Когда закончите тестировать функцию, завершите работу среды выполнения в окне консоли. Только один узел функции может быть активным и работать локально одновременно.

### <a name="debug-the-function-in-eclipse"></a>Отладка функции в Eclipse

В конфигурации **запуска от имени** , настроенной на предыдущем шаге, измените `azure-functions:run` на `azure-functions:run -DenableDebug` и запустите обновленную конфигурацию, чтобы запустить приложение функции в режиме отладки.

Выберите меню **запуска** и откройте **Debug Configurations** (Конфигурации отладки). Выберите пункт **Remote Java Application** (Удаленное приложение Java) и создайте новое. Присвойте конфигурации имя и задайте параметры. Порт должен соответствовать порту отладки, открытому узлом функции, который по умолчанию равен `5005`. После завершения установки щелкните `Debug`, чтобы запустить отладку.

![Функции отладки в Eclipse](media/functions-create-first-java-eclipse/debug-configuration-eclipse.PNG)

Установите точки останова и проверьте объекты в функции, используя интегрированную среду разработки. По завершении остановите отладчик и запущенный узел функции. Только один узел функции может быть активным и работать локально одновременно.

## <a name="deploy-the-function-to-azure"></a>Развертывание функции для Azure

В процессе развертывания для Функций Azure используются данные учетной записи из Azure CLI. [Войдите с помощью Azure CLI](/cli/azure/authenticate-azure-cli) прежде чем продолжить, используя командную строку компьютера.

```azurecli
az login
```

Разверните свой код в новом приложении-функции, используя целевой объект Maven `azure-functions:deploy` в новой конфигурации **Run As** (Выполнить как).

После завершения развертывания появится URL-адрес, который можно использовать для доступа к приложению-функции Azure:

```output
[INFO] Successfully deployed Function App with package.
[INFO] Deleting deployment package from Azure Storage...
[INFO] Successfully deleted deployment package fabrikam-function-20170920120101928.20170920143621915.zip
[INFO] Successfully deployed Function App at https://fabrikam-function-20170920120101928.azurewebsites.net
[INFO] ------------------------------------------------------------------------
```

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о разработке функции Java см. в статье [Azure Functions Java developer guide](functions-reference-java.md) (Руководство разработчика Java для Функций Azure).
- Добавьте в проект дополнительные функции с помощью различных триггеров целевого объекта Maven`azure-functions:add`.
