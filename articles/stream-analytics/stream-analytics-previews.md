---
title: Предварительная версия функций Azure Stream Analytics
description: В этой статье перечислены Azure Stream Analytics функции, которые сейчас доступны в предварительной версии.
author: enkrumah
ms.author: ebnkruma
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 03/16/2021
ms.openlocfilehash: 55745c022038fa85f5b114f2bc347ed7292665eb
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104589656"
---
# <a name="azure-stream-analytics-preview-features"></a>Предварительная версия функций Azure Stream Analytics

В этой статье перечислены все функции Azure Stream Analytics, которые сейчас доступны в предварительной версии. Предварительную версию функций не рекомендуется использовать в рабочей среде.

## <a name="authenticate-to-sql-database-output-with-managed-identities-preview"></a>Проверка подлинности в выходных данных SQL с помощью управляемых удостоверений (Предварительная версия)

Azure Stream Analytics поддерживает [проверку подлинности управляемого удостоверения](../active-directory/managed-identities-azure-resources/overview.md) для приемников выходных данных Базы данных SQL Azure. Управляемые удостоверения устраняют ограничения методов проверки подлинности на основе пользователей, например необходимость повторной проверки подлинности из-за изменения пароля. 

## <a name="real-time-high-performance-scoring-with-custom-ml-models-managed-by-azure-machine-learning"></a>Высокопроизводительная оценка в режиме реального времени с помощью пользовательских моделей ML, управляемых Машинным обучением Azure.

Azure Stream Analytics поддерживает высокопроизводительную оценку в режиме реального времени, используя предварительно подготовленные модели Машинного обучения, управляемые Машинным обучением Azure и размещенные в Службе Azure Kubernetes Service (AKS) или Экземплярах контейнеров Azure (ACI), используя рабочий процесс, не требующий написания кода. [Регистрация](https://aka.ms/asapreview1) для предварительной версии

## <a name="c-custom-de-serializers"></a>Пользовательские десериализаторы на C#
Разработчики могут использовать возможности Azure Stream Analytics для обработки данных в Protobuf, XML или любом другом пользовательском формате. Вы можете реализовать [пользовательские десериализаторы](custom-deserializer-examples.md) на C#, которые затем можно использовать для десериализации событий, получаемых Azure Stream Analytics.

## <a name="extensibility-with-c-custom-code"></a>Расширяемость с помощью пользовательского кода на C#

Разработчики, создающие модули Stream Analytics в облаке или IoT Edge, могут записывать или повторно использовать пользовательские функции C# и вызывать их непосредственно в запросе через [пользовательские функции](stream-analytics-edge-csharp-udf-methods.md).

## <a name="debug-query-steps-in-visual-studio"></a>Отладка шагов запроса в Visual Studio

Вы можете легко просмотреть промежуточный набор строк на диаграмме данных при локальном тестировании в инструментах Azure Stream Analytics для Visual Studio. 


## <a name="live-data-testing-in-visual-studio"></a>Динамическое тестирование данных в Visual Studio

Инструменты Visual Studio для Azure Stream Analytics усовершенствуют локальный компонент тестирования, который позволяет тестировать запросы к событиям прямой трансляции из облачных источников, таких как Концентратор событий или центр Интернета вещей. Дополнительные сведения см. в статье [Тестирование реальных данных в локальной среде с помощью инструментов Azure Stream Analytics для Visual Studio (предварительная версия)](stream-analytics-live-data-local-testing.md).

## <a name="visual-studio-code-for-azure-stream-analytics"></a>Visual Studio Code для Azure Stream Analytics

Задания Azure Stream Analytics можно создать в Visual Studio Code. Дополнительные сведения см. в [руководстве по началу работы в VS Code](./quick-create-visual-studio-code.md).

## <a name="local-testing-with-live-data-in-visual-studio-code"></a>Локальное тестирование с фактическими данными в Visual Studio Code

Перед отправкой задания в Azure вы можете протестировать ваши запросы на предмет наличия данных на локальном компьютере в режиме реального времени. Каждая тестовая итерация занимает в среднем менее двух-трех секунд, что делает процесс разработки очень эффективным.

