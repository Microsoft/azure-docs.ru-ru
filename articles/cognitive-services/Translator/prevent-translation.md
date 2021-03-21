---
title: Запретить перевод содержимого — переводчик
titleSuffix: Azure Cognitive Services
description: Запрет перевода содержимого с помощью транслятора. Переводчик позволяет пометить содержимое так, чтобы оно не переводилось.
services: cognitive-services
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: conceptual
ms.date: 05/26/2020
ms.author: lajanuar
ms.openlocfilehash: e89ff3f4c1f54d9e137d3dd51e325b908c826b03
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98897482"
---
# <a name="how-to-prevent-translation-of-content-with-the-translator"></a>Как предотвратить перевод содержимого с помощью транслятора

Переводчик позволяет пометить содержимое так, чтобы оно не переводилось. Например, таким образом можно пометить код, название торговой марки, а также слово или фразу, которые при локализации теряют смысл.

## <a name="methods-for-preventing-translation"></a>Способы запрета перевода

1. Пометьте содержимое с помощью `notranslate`. Это можно сделать, только если входные Тексттипе заданы в формате HTML.

   Пример

   ```html
   <span class="notranslate">This will not be translated.</span>
   <span>This will be translated. </span>
   ```
   
   ```html
   <div class="notranslate">This will not be translated.</div>
   <div>This will be translated. </div>
   ```

2. Пометьте содержимое с помощью `translate="no"`. Работает только в том случае, если входной Тексттипе задан как HTML

   Пример

   ```html
   <span translate="no">This will not be translated.</span>
   <span>This will be translated. </span>
   ```
   
   ```html
   <div translate="no">This will not be translated.</div>
   <div>This will be translated. </div>
   ```
   
3. Используйте [динамический словарь](dynamic-dictionary.md), чтобы настроить требуемый перевод.

4. Не передавайте строку транслятору для перевода.

5. Пользовательский переводчик. Используйте [словарь в пользовательском трансляторе](custom-translator/what-is-dictionary.md) , чтобы предписывает перевод фразы с вероятностью 100%.


## <a name="next-steps"></a>Дальнейшие действия
> [!div class="nextstepaction"]
> [Преобразование текста с помощью операции преобразования](reference/v3-0-translate.md)
