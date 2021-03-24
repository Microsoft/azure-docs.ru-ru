---
title: Начало работы с функциями Azure
description: Выполните первые шаги для работы с функциями Azure.
author: craigshoemaker
ms.topic: overview
ms.date: 11/19/2020
ms.author: cshoe
zone_pivot_groups: programming-languages-set-functions-lang-workers
ms.openlocfilehash: 77d370b895c777278d3136c7d2c511e7f9e23b36
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102179261"
---
# <a name="getting-started-with-azure-functions"></a>Начало работы с функциями Azure

## <a name="introduction"></a>Введение

[Функции Azure](./functions-overview.md) позволяют реализовать логику системы в готовых блоках кода. Эти блоки кода называются "функциями".

Воспользуйтесь следующими ресурсами, чтобы начать работу.

::: zone pivot="programming-language-csharp"

| Действие | Ресурсы |
| --- | --- |
| **Создание первой функции** | С помощью одного из следующих средств<br><br><li>[Visual Studio](./functions-create-your-first-function-visual-studio.md)<li>[Visual Studio Code](./create-first-function-vs-code-csharp.md)<li>[Командная строка](./create-first-function-cli-csharp.md) |
| **можно увидеть функцию, выполняющую** | <li>[Браузер Azure Samples](/samples/browse/?expanded=azure&languages=csharp&products=azure-functions)<li>[Azure Community Library](https://www.serverlesslibrary.net/?technology=Functions%202.x&language=C%23) |
| **Знакомство с интерактивным учебником**| <li>[Выбор оптимальной бессерверной технологии Azure для бизнес-сценария](/learn/modules/serverless-fundamentals/)<li>[Хорошо спроектированная платформа. Эффективность](/learn/modules/azure-well-architected-performance-efficiency/)<li>[Выполнение функции Azure с помощью триггеров](/learn/modules/execute-azure-function-with-triggers/) <br><br>[Полный список интерактивных учебников](/learn/browse/?expanded=azure&products=azure-functions) см. в Microsoft Learn.|
| **Анализ рекомендаций** |<li>[Повышение производительности и надежности](./functions-best-practices.md)<li>[Управление коллекциями](./manage-connections.md)<li>[Обработка ошибок и повторные попытки функций](./functions-bindings-error-pages.md?tabs=csharp)<li>[Security](./security-concepts.md)|
| **Узнать больше** | <li>Узнайте, как функции [автоматически увеличивают или уменьшают](./functions-scale.md) количество экземпляров в соответствии с потребностями.<li>Изучите различные доступные [методы развертывания](./functions-deployment-technologies.md)<li>Используйте встроенные [средства мониторинг](./functions-monitoring.md) для анализа функций<li>Ознакомьтесь со [Справочником по C#](./functions-dotnet-class-library.md)|

::: zone-end

::: zone pivot="programming-language-java"
| Действие | Ресурсы |
| --- | --- |
| **Создание первой функции** | С помощью одного из следующих средств<br><br><li>[Visual Studio Code](./create-first-function-vs-code-java.md)<li>[Функция Java/Maven в терминале / командной строке](./create-first-function-cli-java.md)<li>[Gradle](./functions-create-first-java-gradle.md)<li>[Eclipse](./functions-create-maven-eclipse.md)<li>[IntelliJ IDEA](./functions-create-maven-intellij.md) |
| **можно увидеть функцию, выполняющую** | <li>[Браузер Azure Samples](/samples/browse/?expanded=azure&languages=java&products=azure-functions)<li>[Azure Community Library](https://www.serverlesslibrary.net/?technology=Functions%202.x&language=Java) |
| **Знакомство с интерактивным учебником**| <li>[Выбор оптимальной бессерверной технологии Azure для бизнес-сценария](/learn/modules/serverless-fundamentals/)<li>[Хорошо спроектированная платформа. Эффективность](/learn/modules/azure-well-architected-performance-efficiency/)<li>[Разработка приложения с помощью подключаемого модуля Maven для Функций Azure](/learn/modules/develop-azure-functions-app-with-maven-plugin/) <br><br>[Полный список интерактивных учебников](/learn/browse/?expanded=azure&products=azure-functions) см. в Microsoft Learn.|
| **Анализ рекомендаций** |<li>[Повышение производительности и надежности](./functions-best-practices.md)<li>[Управление коллекциями](./manage-connections.md)<li>[Обработка ошибок и повторные попытки функций](./functions-bindings-error-pages.md?tabs=java)<li>[Security](./security-concepts.md)|
| **Узнать больше** | <li>Узнайте, как функции [автоматически увеличивают или уменьшают](./functions-scale.md) количество экземпляров в соответствии с потребностями.<li>Изучите различные доступные [методы развертывания](./functions-deployment-technologies.md)<li>Используйте встроенные [средства мониторинг](./functions-monitoring.md) для анализа функций<li>Ознакомьтесь со [Справочником по Java](./functions-reference-java.md)|
::: zone-end

::: zone pivot="programming-language-javascript"
| Действие | Ресурсы |
| --- | --- |
| **Создание первой функции** | С помощью одного из следующих средств<br><br><li>[Visual Studio Code](./create-first-function-vs-code-node.md)<li>[Функция Node.js в терминале / командной строке](./create-first-function-cli-node.md) |
| **можно увидеть функцию, выполняющую** | <li>[Браузер Azure Samples](/samples/browse/?expanded=azure&languages=javascript%2ctypescript&products=azure-functions)<li>[Azure Community Library](https://www.serverlesslibrary.net/?technology=Functions%202.x&language=JavaScript%2CTypeScript) |
| **Знакомство с интерактивным учебником** | <li>[Выбор оптимальной бессерверной технологии Azure для бизнес-сценария](/learn/modules/serverless-fundamentals/)<li>[Хорошо спроектированная платформа. Эффективность](/learn/modules/azure-well-architected-performance-efficiency/)<li>[Создание бессерверных API с помощью Функций Azure](/learn/modules/build-api-azure-functions/)<li>[Создание бессерверной логики с помощью Функций Azure](/learn/modules/create-serverless-logic-with-azure-functions/)<li>[Рефакторинг API-интерфейсов Node.js и Express в бессерверные API с помощью Функций Azure](/learn/modules/shift-nodejs-express-apis-serverless/) <br><br>[Полный список интерактивных учебников](/learn/browse/?expanded=azure&products=azure-functions) см. в Microsoft Learn.|
| **Анализ рекомендаций** |<li>[Повышение производительности и надежности](./functions-best-practices.md)<li>[Управление коллекциями](./manage-connections.md)<li>[Обработка ошибок и повторные попытки функций](./functions-bindings-error-pages.md?tabs=javascript)<li>[Security](./security-concepts.md)|
| **Узнать больше** | <li>Узнайте, как функции [автоматически увеличивают или уменьшают](./functions-scale.md) количество экземпляров в соответствии с потребностями.<li>Изучите различные доступные [методы развертывания](./functions-deployment-technologies.md)<li>Используйте встроенные [средства мониторинг](./functions-monitoring.md) для анализа функций<li>Ознакомьтесь со справочником по [JavaScript](./functions-reference-node.md) или [TypeScript](./functions-reference-node.md#typescript)|
::: zone-end

::: zone pivot="programming-language-powershell"
| Действие | Ресурсы |
| --- | --- |
| **Создание первой функции** | <li>Использование [Visual Studio Code](./create-first-function-vs-code-powershell.md) |
| **можно увидеть функцию, выполняющую** | <li>[Браузер Azure Samples](/samples/browse/?expanded=azure&languages=powershell&products=azure-functions)<li>[Azure Community Library](https://www.serverlesslibrary.net/?technology=Functions%202.x&language=PowerShell) |
| **Знакомство с интерактивным учебником** | <li>[Выбор оптимальной бессерверной технологии Azure для бизнес-сценария](/learn/modules/serverless-fundamentals/)<li>[Хорошо спроектированная платформа. Эффективность](/learn/modules/azure-well-architected-performance-efficiency/)<li>[Создание бессерверных API с помощью Функций Azure](/learn/modules/build-api-azure-functions/)<li>[Создание бессерверной логики с помощью Функций Azure](/learn/modules/create-serverless-logic-with-azure-functions/)<li>[Выполнение функции Azure с помощью триггеров](/learn/modules/execute-azure-function-with-triggers/) <br><br>[Полный список интерактивных учебников](/learn/browse/?expanded=azure&products=azure-functions) см. в Microsoft Learn.|
| **Анализ рекомендаций** |<li>[Повышение производительности и надежности](./functions-best-practices.md)<li>[Управление коллекциями](./manage-connections.md)<li>[Обработка ошибок и повторные попытки функций](./functions-bindings-error-pages.md?tabs=powershell)<li>[Security](./security-concepts.md)|
| **Узнать больше** | <li>Узнайте, как функции [автоматически увеличивают или уменьшают](./functions-scale.md) количество экземпляров в соответствии с потребностями.<li>Изучите различные доступные [методы развертывания](./functions-deployment-technologies.md)<li>Используйте встроенные [средства мониторинг](./functions-monitoring.md) для анализа функций<li>Ознакомьтесь со [справочником по PowerShell](./functions-reference-powershell.md))|
::: zone-end

::: zone pivot="programming-language-python"
| Действие | Ресурсы |
| --- | --- |
| **Создание первой функции** | С помощью одного из следующих средств<br><br><li>[Visual Studio Code](./create-first-function-vs-code-csharp.md?pivots=programming-language-python)<li>[Терминал / командная строка](./create-first-function-cli-csharp.md?pivots=programming-language-python) |
| **можно увидеть функцию, выполняющую** | <li>[Браузер Azure Samples](/samples/browse/?expanded=azure&languages=python&products=azure-functions)<li>[Azure Community Library](https://www.serverlesslibrary.net/?technology=Functions%202.x&language=Python) |
| **Знакомство с интерактивным учебником** | <li>[Выбор оптимальной бессерверной технологии Azure для бизнес-сценария](/learn/modules/serverless-fundamentals/)<li>[Хорошо спроектированная платформа. Эффективность](/learn/modules/azure-well-architected-performance-efficiency/)<li>[Создание бессерверных API с помощью Функций Azure](/learn/modules/build-api-azure-functions/)<li>[Создание бессерверной логики с помощью Функций Azure](/learn/modules/create-serverless-logic-with-azure-functions/) <br><br>[Полный список интерактивных учебников](/learn/browse/?expanded=azure&products=azure-functions) см. в Microsoft Learn.|
| **Анализ рекомендаций** |<li>[Повышение производительности и надежности](./functions-best-practices.md)<li>[Управление коллекциями](./manage-connections.md)<li>[Обработка ошибок и повторные попытки функций](./functions-bindings-error-pages.md?tabs=python)<li>[Security](./security-concepts.md)<li>[Повышение производительности и пропускной способности](./python-scale-performance-reference.md)|
| **Узнать больше** | <li>Узнайте, как функции [автоматически увеличивают или уменьшают](./functions-scale.md) количество экземпляров в соответствии с потребностями.<li>Изучите различные доступные [методы развертывания](./functions-deployment-technologies.md)<li>Используйте встроенные [средства мониторинг](./functions-monitoring.md) для анализа функций<li>Ознакомьтесь со [Справочником по Python](./functions-reference-python.md)|
::: zone-end

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Анатомия приложения Функций Azure](./functions-reference.md)
