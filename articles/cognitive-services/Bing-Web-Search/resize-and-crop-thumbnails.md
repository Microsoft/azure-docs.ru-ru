---
title: Изменение размера и обрезка эскизов изображений — API Bing для поиска в Интернете
titleSuffix: Azure Cognitive Services
description: Некоторые ответы из API-интерфейсы поиска Bing включают URL-адреса для эскизов изображений, обслуживаемых Bing, которые можно изменять, масштабировать и обрезать, а также могут содержать параметры запроса.
services: cognitive-services
author: aahill
manager: nitinme
ms.assetid: 05A08B01-89FF-4781-AFE7-08DA92F25047
ms.service: cognitive-services
ms.subservice: bing-web-search
ms.topic: conceptual
ms.date: 07/08/2019
ms.author: aahi
ms.openlocfilehash: a85c5b2333418367742678a529b69c95164eda53
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96350489"
---
# <a name="resize-and-crop-thumbnail-images"></a>Изменение размера и обрезка эскизов

> [!WARNING]
> API Поиска Bing будут перенесены из Cognitive Services в службы Поиска Bing. С **30 октября 2020 г.** подготовку всех новых экземпляров Поиска Bing необходимо будет выполнять в соответствии с процедурой, описанной [здесь](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> API-интерфейсы Поиска Bing, подготовленные с помощью Cognitive Services, будут поддерживаться в течение следующих трех лет или до завершения срока действия вашего Соглашения Enterprise (в зависимости от того, какой период окончится раньше).
> Инструкции по миграции см. в статье о [службах Поиска Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Некоторые ответы из API-интерфейсы поиска Bing включают URL-адреса для эскизов изображений, обслуживаемых Bing, которые можно изменять, масштабировать и обрезать, а также могут содержать параметры запроса. Пример:

`https://<host>/th?id=AMMS_92772df988...&w=110&h=73&rs=1&qlt=80&cdv=1&pid=16.1`

При отображении подмножества этих эскизов укажите параметр для просмотра остальных изображений.

> [!NOTE]
> Убедитесь, что обрезка и изменение размера эскизов будут предоставлять сценарий поиска, учитывающий сторонние права, в соответствии с [требованиями Поиск Bing использования API и отображаемых требований](use-display-requirements.md).

## <a name="resize-a-thumbnail"></a>Изменение размера эскиза 

Чтобы изменить размер эскиза, Bing рекомендует указать только один `w` параметр запроса (Width) или `h` (Height) в URL-адресе эскиза. Задание только высоты или ширины позволяет Bing сохранять исходный аспект изображения. Укажите ширину и высоту в пикселях. 

Например, если исходный эскиз является 480x620:

`https://<host>/th?id=JN.5l3yzwy%2f%2fHj59U6XhssIQ&pid=Api&w=480&h=620`

И вы хотите уменьшить его размер, присвойте `w` параметру новое значение (например `336` ,) и удалите `h`  параметр:

`https://<host>/th?id=JN.5l3yzwy%2f%2fHj59U6XhssIQ&pid=Api&w=336`

Если указать только высоту или ширину эскиза, будет поддерживаться исходное пропорции изображения. Если указать оба параметра и пропорции не поддерживаются, Bing добавит в границы изображения пробелы.

Например, если изменить размер изображения 480x359 на 200x200 без обрезки, то полная ширина будет содержать изображение, но высота будет содержать 25 пикселов белого отступа в верхней и нижней части изображения. Если изображение было 359x480, то левая и правая границы будут содержать пробелы. При обрезке изображения белые поля не добавляются.  

Ниже показано изображение эскиза в исходном размере (480 x 300).  
  
![Исходное изображение пейзажа](./media/resize-crop/bing-resize-crop-landscape.png)  
  
Ниже показано изображение, размер которого изменен до 200 x 200. Пропорции сохраняются, а верхняя и нижняя границы дополняются белым цветом (для отображения заполнения используется черная граница).  
  
![Изображение пейзажа с измененным размером](./media/resize-crop/bing-resize-crop-landscape-resized.png)  

Если указать размеры, превышающие исходную ширину и высоту изображения, то Bing добавит в левую и верхние границы квадратные дополнения.  

## <a name="request-different-thumbnail-sizes"></a>Запрос различных размеров эскиза

Чтобы запросить другой размер изображения эскиза, удалите все параметры запроса из URL-адреса эскиза, кроме `id` `pid` параметров и. Затем добавьте `&w` параметр запроса (Width) или `&h` (Height) с нужным размером изображения в пикселях, но не оба. Bing сохранит исходные пропорции изображения. 

Чтобы увеличить ширину изображения с указанным выше URL-адресом до 165 пикселей, используйте следующий URL-адрес:

`https://<host>/th?id=AMMS_92772df988...&w=165&pid=16.1`

Если вы запрашиваете изображение, размер которого превышает исходный, Bing добавляет в изображение пробелы по мере необходимости. Например, если исходный размер изображения — 474x316 и задано значение `&w` 500, Bing вернет изображение 500x333. На этом изображении будет 8,5 пикселей белого пробела по верхнему и нижнему краям, а также 13 пикселов заполнения на левом и правом краях.

Чтобы служба Bing не допустить добавления пробелов, если запрошенный размер больше исходного размера изображения, установите `&p` для параметра запроса значение 0. Например, если включить `&p=0` параметр в приведенный выше URL-адрес, Bing вернет образ 474x316 вместо образа 500x333:

`https://<host>/th?id=AMMS_92772df988...&w=500&p=0&pid=16.1`

Если заданы `&w` Параметры и `&h` запроса, Bing будет поддерживать пропорции изображения и при необходимости добавит пробелы. Например, если исходный размер изображения — 474x316 и для параметров ширины и высоты задано значение 200x200 ( `&w=200&h=200` ), то Bing возвращает изображение, содержащее 33 пикселей белого пробела сверху и снизу. Если включить `&p` параметр запроса, Bing возвращает изображение 200x134.

## <a name="crop-a-thumbnail"></a>Кадрирование эскиза 

Чтобы обрезать изображение, включите `c` параметр запроса (Кадрирование). Можно использовать следующие значения:
  
- `4`&mdash;Соотношение скрытых  
- `7`&mdash;Интеллектуальное соотношение  

### <a name="smart-ratio-cropping"></a>Обрезка интеллектуальной соотношения

Если вы запрашиваете обрезку интеллектуальной соотношения (установив `c` для параметра значение `7` ), Bing обрезает изображение из центра своего региона, а также сохраняет пропорции изображения. Интересующая область — это область изображения, где по определению Bing содержится самая важная часть. Ниже показан пример интересующей области.  
  
![Интересующая область](./media/resize-crop/bing-resize-crop-regionofinterest.png)

Если вы изменяете размер изображения и запрашиваете обрезку интеллектуальной соотношения, Bing сокращает изображение до запрошенного размера, сохраняя пропорции. Затем Bing обрезает изображение на основе размеров с измененным размером. Например, если ширина измененной ширины меньше или равна высоте, то Bing обработает изображение слева и справа от центра интересующей области. В противном случае Bing оббудет обрезать его в верхней и нижней части центра интересующей области.  
  
 
Ниже показано изображение, уменьшенное до размера 200 x 200 с использованием обрезки Smart Ratio. Так как Bing измеряет изображение из левого верхнего угла, Нижняя часть изображения обрезается. 
  
![Изображение пейзажа, обрезанное до размера 200 x 200](./media/resize-crop/bing-resize-crop-landscape200x200c7.png) 
  
Ниже показано изображение, уменьшенное до размера 200 x 100 с использованием обрезки Smart Ratio. Так как Bing измеряет изображение из левого верхнего угла, Нижняя часть изображения обрезается. 
   
![Изображение пейзажа, обрезанное до размера 200 x 100](./media/resize-crop/bing-resize-crop-landscape200x100c7.png)
  
Ниже показано изображение, уменьшенное до размера 100 x 200 с использованием обрезки Smart Ratio. Так как Bing измеряет изображение из центра, левая и правая части изображения обрезаются.
  
![Изображение пейзажа, обрезанное до размера 100 x 200](./media/resize-crop/bing-resize-crop-landscape100x200c7.png) 

Если Bing не может определить интересующую область изображения, служба будет использовать обрезку незакрытого соотношения.  

### <a name="blind-ratio-cropping"></a>Обрезка соотношения слепых

Если вы запрашиваете обрезку скрытого соотношения (путем установки `c` для параметра значения `4` ), то Bing использует следующие правила для кадрирования изображения.  
  
- Если `(Original Image Width / Original Image Height) < (Requested Image Width / Requested Image Height)` значение равно, изображение измеряется от верхнего левого угла и обрезается внизу.  
- Если `(Original Image Width / Original Image Height) > (Requested Image Width / Requested Image Height)` значение равно, изображение измеряется от центра и обрезается влево и вправо.  

Ниже показано изображение в книжной ориентации размером 225 x 300.  
  
![Исходное изображение подсолнуха](./media/resize-crop/bing-resize-crop-sunflower.png)
  
Ниже показано изображение, уменьшенное до размера 200 x 200 с использованием обрезки Blind Ratio. Изображение измеряется из верхнего левого угла, что приводит к обрезанию нижней части изображения.  
  
![Изображение подсолнуха, обрезанное до размера 200 x 200](./media/resize-crop/bing-resize-crop-sunflower200x200c4.png)
  
Ниже показано изображение, уменьшенное до размера 200 x 100 с использованием обрезки Blind Ratio. Изображение измеряется из верхнего левого угла, что приводит к обрезанию нижней части изображения.  
  
![Изображение подсолнуха, обрезанное до размера 200 x 100](./media/resize-crop/bing-resize-crop-sunflower200x100c4.png)
  
Ниже показано изображение, уменьшенное до размера 100 x 200 с использованием обрезки Blind Ratio. Изображение измеряется от центра и обрезается с левой и правой сторон.  
  
![Изображение подсолнуха, обрезанное до размера 100 x 200](./media/resize-crop/bing-resize-crop-sunflower100x200c4.png)

## <a name="next-steps"></a>Дальнейшие действия

* [Что такое API-интерфейсы Поиска Bing?](bing-api-comparison.md)
* [Требования к использованию и отображению API-интерфейсов Поиска Bing](use-display-requirements.md)