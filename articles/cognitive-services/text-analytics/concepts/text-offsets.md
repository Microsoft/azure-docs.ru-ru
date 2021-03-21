---
title: Смещение текста в API анализа текста
titleSuffix: Azure Cognitive Services
description: Сведения о смещениях, вызванных кодировками многоязычного и эмодзи.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: article
ms.date: 03/09/2020
ms.author: aahi
ms.reviewer: jdesousa
ms.openlocfilehash: f5b63503792b13e089568004ba67e5be8a3d0c7f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98932363"
---
# <a name="text-offsets-in-the-text-analytics-api-output"></a>Смещения текста в выходных данных API анализа текста

Поддержка многоязычности и эмодзи привела к кодированиям в Юникоде, которые используют более одной [кодовой точки](https://wikipedia.org/wiki/Code_point) для представления одного отображаемого символа, называемого графеме. Например, эмодзи, такие как 🌷 и, 👍 могут использовать несколько символов для создания фигуры с дополнительными символами для визуальных атрибутов, таких как тон кожи. Аналогичным образом, слово хинди `अनुच्छेद` кодируется как пять букв и три Объединенных знака.

Из-за различной длины возможных кодировок многоязычности и эмодзи API анализа текста может возвращать смещение в ответе.

## <a name="offsets-in-the-api-response"></a>Смещение в ответе API. 

Всякий раз, когда смещение возвращает ответ API, например [Распознавание сущностей](../how-tos/text-analytics-how-to-entity-linking.md) или [Анализ тональности](../how-tos/text-analytics-how-to-sentiment-analysis.md), помните следующее:

* Элементы в ответе могут быть специфическими для вызванной конечной точки. 
* Полезные данные HTTP POST/GET кодируются в [кодировке UTF-8](https://www.w3schools.com/charsets/ref_html_utf8.asp), которая может быть или не совпадать с кодировкой символов по умолчанию в клиентском компиляторе или операционной системе.
* Смещения относятся к счетчикам графеме, основанным на стандарте [Unicode 8.0.0](https://unicode.org/versions/Unicode8.0.0) , а не по числу символов.

## <a name="extracting-substrings-from-text-with-offsets"></a>Извлечение подстрок из текста с использованием смещений

Смещение может вызвать проблемы при использовании символьных методов подстроки, например метода .NET [substring ()](/dotnet/api/system.string.substring) . Одна из проблем заключается в том, что смещение может привести к тому, что метод подстроки завершится в середине многосимвольной кодировки графеме, а не в конце.

В .NET рассмотрите возможность использования класса [StringInfo](/dotnet/api/system.globalization.stringinfo) , который позволяет работать со строкой в виде набора текстовых элементов, а не отдельных символьных объектов. Вы также можете найти библиотеки графеме Splitter в предпочитаемой программной среде. 

API анализа текста также возвращает эти текстовые элементы для удобства.

## <a name="offsets-in-api-version-31-preview"></a>Смещения в API версии 3,1-Preview

Начиная с API версии 3,1-Preview. 1 все конечные точки API анализа текста, которые возвращают смещение, будут поддерживать этот `stringIndexType` параметр. Этот параметр корректирует `offset` `length` атрибуты и в выходных данных API в соответствии с требуемой схемой итерации строк. В настоящее время поддерживаются три типа:

1. `textElement_v8`(по умолчанию): выполняет перебор по графемес, как определено стандартом [Unicode 8.0.0](https://unicode.org/versions/Unicode8.0.0)
2. `unicodeCodePoint`: выполняет перебор [кодовых позиций Юникода](http://www.unicode.org/versions/Unicode13.0.0/ch02.pdf#G25564), схемы по умолчанию для Python 3
3. `utf16CodeUnit`: выполняет итерацию по [блокам кода UTF-16](https://unicode.org/faq/utf_bom.html#UTF16), схеме по умолчанию для JavaScript, Java и .NET

Если `stringIndexType` запрошенное соответствие выбранному программному обеспечению, извлечение подстроки может выполняться с помощью стандартных методов SUBSTRING или slice. 

## <a name="see-also"></a>См. также раздел

* [Text Analytics overview](../overview.md) (Общие сведения об анализе текста)
* [Пример. Как определить тональность с помощью Анализа текста](../how-tos/text-analytics-how-to-sentiment-analysis.md)
* [Распознавание сущностей](../how-tos/text-analytics-how-to-entity-linking.md)
* [Пример. Как определить язык с помощью Анализа текста](../how-tos/text-analytics-how-to-keyword-extraction.md)
* [Распознавание языка](../how-tos/text-analytics-how-to-language-detection.md)
