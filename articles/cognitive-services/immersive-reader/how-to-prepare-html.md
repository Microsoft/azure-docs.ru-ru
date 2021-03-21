---
title: Подготовка содержимого HTML для иммерсивное средство чтения
titleSuffix: Azure Cognitive Services
description: Узнайте, как запустить иммерсивное средство чтения с помощью HTML, JavaScript, Python, Android или iOS. Иммерсивное средство чтения использует проверенные методы для улучшения навыков чтения для средств обучения, новых читателей и учащихся с учетом отличий.
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: immersive-reader
ms.topic: include
ms.date: 03/04/2021
ms.author: erhopf
ms.openlocfilehash: 40aab9510eb90c405f92be49a3b4c0a3f756c872
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102620165"
---
# <a name="how-to-prepare-html-content-for-immersive-reader"></a>Подготовка содержимого HTML для иммерсивное средство чтения

В этой статье показано, как структурировать HTML и получить содержимое, чтобы его можно было использовать в иммерсивное средство чтения.

## <a name="prepare-the-html-content"></a>Подготовка содержимого HTML

Поместите содержимое, которое требуется визуализировать, в иммерсивное средство чтения внутри элемента контейнера. Убедитесь, что элемент контейнера имеет уникальное значение `id` . Иммерсивное средство чтения предоставляет поддержку основных элементов HTML. Дополнительные сведения см. в [справочнике](reference.md#html-support) .

```html
<div id='immersive-reader-content'>
    <b>Bold</b>
    <i>Italic</i>
    <u>Underline</u>
    <strike>Strikethrough</strike>
    <code>Code</code>
    <sup>Superscript</sup>
    <sub>Subscript</sub>
    <ul><li>Unordered lists</li></ul>
    <ol><li>Ordered lists</li></ol>
</div>
```

## <a name="get-the-html-content-in-javascript"></a>Получение HTML-содержимого в JavaScript

Используйте `id` элемент контейнера для получения HTML-содержимого в коде JavaScript.

```javascript
const htmlContent = document.getElementById('immersive-reader-content').innerHTML;
```

## <a name="launch-the-immersive-reader-with-your-html-content"></a>Запуск иммерсивное средство чтения с помощью HTML-содержимого

При вызове `ImmersiveReader.launchAsync` Задайте для свойства фрагмента значение `mimeType` `text/html` , чтобы разрешить HTML-код отрисовки.

```javascript
const data = {
    chunks: [{
        content: htmlContent,
        mimeType: 'text/html'
    }]
};

ImmersiveReader.launchAsync(YOUR_TOKEN, YOUR_SUBDOMAIN, data, YOUR_OPTIONS);
```

## <a name="next-steps"></a>Дальнейшие действия

* Ознакомьтесь со справочной документацией по [пакету SDK для иммерсивного средства чтения](reference.md).