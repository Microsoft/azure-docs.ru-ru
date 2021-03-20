---
title: Разработка и отладка заданий Azure Stream Analytics локально
description: Узнайте, как разрабатывать и тестировать задания Azure Stream Analytics на локальном компьютере, прежде чем запускать их в портал Azure.
ms.author: sujie
author: su-jie
ms.topic: conceptual
ms.date: 03/31/2020
ms.service: stream-analytics
ms.openlocfilehash: 18df480dab90d9ab127bb96971fc19cdc5a361ce
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98016479"
---
# <a name="develop-and-debug-azure-stream-analytics-jobs-locally"></a>Разработка и отладка заданий Azure Stream Analytics локально

Хотя вы можете создавать и тестировать задания Azure Stream Analytics в портал Azure, многие разработчики предпочитают локальную среду разработки. Stream Analytics упрощает использование избранного редактора кода и средств разработки для создания и тестирования заданий с динамическими потоками событий из концентратора событий Azure, центра Интернета вещей и хранилища BLOB-объектов с помощью полностью функциональной локальной среды выполнения на одном узле. Вы также можете отправлять задания в Azure непосредственно из локальной среды разработки.

## <a name="local-development-environments"></a>Локальные среды разработки

Способ разработки Stream Analytics заданий на локальном компьютере зависит от настроек инструментария и доступности функций. Дополнительные сведения о функциях, поддерживаемых в каждой среде разработки, см. в разделе [Сравнение функций Azure Stream Analytics](feature-comparison.md) .

В нижеприведенной таблице указаны среды, которые поддерживают локальную разработку:

|Среда                              |Описание    |
|-----------------------------------------|------------|
|[Visual Studio Code](visual-studio-code-explore-jobs.md)| [Расширение Azure Stream Analytics Tools](https://marketplace.visualstudio.com/items?itemName=ms-bigdatatools.vscode-asa) для Visual Studio Code позволяет создавать, управлять и тестировать задания Stream Analytics как локально, так и в облаке с богатыми функциями IntelliSense и собственными средствами управления версиями. Поддерживает разработку в Linux, MacOS и Windows. Дополнительные сведения см. [в разделе Создание задания Azure Stream Analytics в Visual Studio Code](quick-create-visual-studio-code.md). Расширение также поддерживает [Visual Studio кодеспацес](https://visualstudio.microsoft.com/services/visual-studio-codespaces/) , которая является средой разработки, размещенной в облаке.|
|[Visual Studio 2019](stream-analytics-tools-for-visual-studio-install.md) |Средства Stream Analytics являются частью рабочих нагрузок разработки и хранения данных Azure в Visual Studio. Visual Studio можно использовать для написания пользовательских определяемых пользователем функций и десериализаторов C#. Дополнительные сведения см. в статье [создание Azure Stream Analytics задания с помощью Visual Studio](stream-analytics-quick-create-vs.md).|
|[Командная строка или терминал](stream-analytics-tools-for-visual-studio-cicd.md)|Пакет NuGet Azure Stream Analytics CI/CD предоставляет средства для сборки проекта Visual Studio, локальное тестирование на произвольном компьютере. Пакет Azure Stream Analytics CI/CD NPM предоставляет средства для сборок проектов Visual Studio Code (которые формируют шаблон Azure Resource Manager) на произвольном компьютере.|

## <a name="next-steps"></a>Дальнейшие действия

* [Локальное тестирование запросов Stream Analytics с использованием примера данных и Visual Studio Code](visual-studio-code-local-run.md)
* [Тестирование Stream Analytics запросов локально для входа в динамический поток с помощью Visual Studio Code](visual-studio-code-local-run-live-input.md)
* [Локальное тестирование запросов Stream Analytics с помощью Visual Studio](stream-analytics-vs-tools-local-run.md)
* [Тестирование реальных данных в локальной среде с помощью инструментов Azure Stream Analytics для Visual Studio (предварительная версия)](stream-analytics-live-data-local-testing.md)
