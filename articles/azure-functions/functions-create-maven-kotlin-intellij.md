---
title: Создание функции Kotlin в Функциях Azure с помощью IntelliJ
description: Узнайте, как с помощью IntelliJ создать простую функцию Kotlin, активируемую по HTTP-запросу, чтобы в дальнейшем опубликовать ее для запуска в бессерверной среде в Azure.
author: dglover
ms.service: azure-functions
ms.topic: quickstart
ms.date: 03/25/2020
ms.author: dglover
ms.openlocfilehash: f02643ee28d76d4f90206a1aa2879b4672da2a38
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102179465"
---
# <a name="create-your-first-kotlin-function-in-azure-using-intellij"></a>Создание первой функции Kotlin в Azure с помощью IntelliJ

В этой статье показано, как создать функцию Java, активируемую по HTTP-запросу, в проекте IntelliJ IDEA, запустить и отладить проект в интегрированной среде разработки (IDE) и развернуть проект функции в приложении-функции в Azure.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="set-up-your-development-environment"></a>Настройка среды разработки

Чтобы создавать и публиковать функции Kotlin в Azure с помощью IntelliJ, установите следующее программное обеспечение:

- [Java Developer Kit (JDK) версии 8](/azure/developer/java/fundamentals/java-jdk-long-term-support)
- [Apache Maven 3.0](https://maven.apache.org) или более поздней версии
- [IntelliJ IDEA](https://www.jetbrains.com/idea/download), версия Community или Ultimate с Maven
- [Azure CLI](/cli/azure)
- [Azure Functions Core Tools](functions-run-local.md#v2) версии 2.x. Они предоставляют локальной среде разработки возможность записи, выполнения и отладки Функций Azure.

> [!IMPORTANT]
> Чтобы завершить выполнение шагов из этой статьи, переменной среде JAVA_HOME необходимо присвоить расположение установки JDK.

## <a name="create-a-function-project"></a>Создание проекта функции

1. В IntelliJ IDEA выберите **Создать новый проект**.  
1. В левой панели окна **Новый проект** выберите **Maven**.
1. Установите флажок **Create from archetype** (Создать на основе архетипа), а затем выберите **Add Archetype** (Добавить архетип) для [azure-functions-kotlin-archetype](https://mvnrepository.com/artifact/com.microsoft.azure/azure-functions-kotlin-archetype).
1. Заполните следующие поля в окне **Add Archetype** (Добавить архетип).
    - _GroupId_: com.microsoft.azure
    - _ArtifactId_: azure-functions-kotlin-archetype.
    - _Версия._ Используйте последнею версию, которую можно найти в [центральном репозитории](https://mvnrepository.com/artifact/com.microsoft.azure/azure-functions-kotlin-archetype)
    ![Создание проекта Maven на основе архетипа в IntelliJ IDEA](media/functions-create-first-kotlin-intellij/functions-create-intellij.png)  
1. Нажмите кнопку **ОК**, а затем кнопку **Далее**.
1. Введите подробные сведения и выберите **Готово**.

Maven создает файлы проекта в новой папке с именем, которое соответствует значению _ArtifactId_. Созданный код проекта представляет собой простую функцию[активации HTTP](./functions-bindings-http-webhook.md), которая возвращает текст HTTP-запроса.

## <a name="run-project-locally-in-the-ide"></a>Запуск проекта в локальной среде IDE

> [!NOTE]
> Для запуска и отладки проекта в локальной среде нужно установить [Azure Functions Core Tools версии 2](functions-run-local.md#v2).

1. Импортируйте изменения вручную или включите [автоматический импорт](https://www.jetbrains.com/help/idea/creating-and-optimizing-imports.html).
1. Откройте панель инструментов **Maven Projects** (Проекты Maven).
1. Разверните **Жизненный цикл**, а затем откройте **Пакет**. Решение будет создано и упаковано в созданном целевом каталоге.
1. Чтобы запустить локальную среду выполнения Azure, расширьте **Подключаемые модули** > **azure-functions** и откройте **azure-functions:run**.  
  ![Панель инструментов Maven для Функций Azure](media/functions-create-first-kotlin-intellij/functions-intellij-kotlin-maven-toolbar.png)  

1. После того как вы протестировали функцию, закройте диалоговое окно запуска. Только один узел функции может быть активным и работать локально одновременно.

## <a name="debug-the-project-in-intellij"></a>Отладка проекта в IntelliJ

1. Чтобы запустить узел функции в режиме отладки, добавьте при выполнении функции **-DenableDebug** в качестве аргумента. Конфигурацию можно изменить либо в [целях Maven](https://www.jetbrains.com/help/idea/maven-support.html#run_goal),либо запустив в окне терминала следующую команду.  

   ```
   mvn azure-functions:run -DenableDebug
   ```

   С помощью данной команды узел функции откроет порт отладки с номером 5005.

1. В меню **Run** (Запуск) выберите **Edit Configurations** (Изменить конфигурации).
1. Выберите **(+)** , чтобы добавить **Уд. доступ**.
1. Чтобы сохранить конфигурацию, заполните поля _Имя_ и _Параметры_, а затем нажмите кнопку **OK**.
1. После завершения установки для начала отладки выберите **Отладка < Имя конфигурации >** или нажмите комбинацию клавиш SHIFT+F9.

   ![Отладка проекта в IntelliJ](media/functions-create-first-kotlin-intellij/debug-configuration-intellij.PNG)

1. По завершении остановите отладчик и запущенный процесс. Только один узел функции может быть активным и работать локально одновременно.

## <a name="deploy-the-project-to-azure"></a>Развертывание проекта в Azure

1. Перед развертыванием проекта в приложении-функции в Azure необходимо [выполнить вход с помощью Azure CLI](/cli/azure/authenticate-azure-cli).

   ``` azurecli
   az login
   ```

1. Разверните свой код в новом приложении-функции, используя целевой объект Maven `azure-functions:deploy`. Также в окне "Проекты Maven" можно выбрать параметр **azure-functions:deploy**.

   ```
   mvn azure-functions:deploy
   ```

1. После успешного развертывания приложения-функции найдите в выходных данных Azure CLI URL-адрес функции, активируемой по HTTP-запросу.

   ``` output
   [INFO] Successfully deployed Function App with package.
   [INFO] Deleting deployment package from Azure Storage...
   [INFO] Successfully deleted deployment package fabrikam-function-20170920120101928.20170920143621915.zip
   [INFO] Successfully deployed Function App at https://fabrikam-function-20170920120101928.azurewebsites.net
   [INFO] ------------------------------------------------------------------------
   ```

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы развернули первое приложение-функцию Kotlin в Azure, ознакомьтесь с [руководством разработчика Java по Функциям Azure](functions-reference-java.md) для получения дополнительных сведений о разработке функций Java и Kotlin.
- Добавьте в проект дополнительные приложения-функции с помощью различных триггеров, используя целевой объект Maven `azure-functions:add`.
