---
title: Условия соответствия обработчика правил Azure CDN от Verizon Premium
description: Справочная документация по условиям соответствия для обработчика правил сети доставки содержимого Azure от Verizon Premium.
services: cdn
author: asudbring
ms.service: azure-cdn
ms.topic: article
ms.date: 05/26/2020
ms.author: allensu
ms.openlocfilehash: efd6e6a93cd4ca79e6c4b6de69f8514e2d71b252
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "84323320"
---
# <a name="azure-cdn-from-verizon-premium-rules-engine-match-conditions"></a>Условия соответствия обработчика правил Azure CDN от Verizon Premium

В этой статье подробно описаны доступные условия соответствия для [обработчика правил](cdn-verizon-premium-rules-engine.md) сети доставки содержимого (CDN) Azure от Verizon Premium.

Вторая часть правила — это условие соответствия. Условие соответствия определяет различные типы запросов, для которых будет выполнен ряд функций.

Например, с помощью условия соответствия вы сможете:

- Отфильтровать запросы содержимого в определенном расположении.
- Отфильтровать запросы, отправленные с определенного IP-адреса или из определенной страны или региона.
- Отфильтровать запросы по информации из заголовка.

## <a name="match-conditions"></a><a name="top"></a>Условия соответствия

* [Всегда](#always)
* [Устройство](#device)
* [Расположение](#location)
* [Исходный домен](#origin)
* [Запрос](#request)
* [URL-адрес](#url)

### <a name="always"></a><a name="always"></a>Всегда

[Условие "всегда соответствие"](https://docs.vdms.com/cdn/Content/HRE/M/Always.htm) предназначено для применения набора функций по умолчанию ко всем запросам.

### <a name="device"></a><a name="device"></a>Устройство

Эти условия соответствия предназначены для обнаружения запросов на основе агента пользователя клиента.

| Имя       | Назначение                                                           |
|------------|-------------------------------------------------------------------|
| Название торговой марки | Определяет запросы, если фирменное имя устройства совпадает с: <br> **-** Конкретное значение ([литерал фирменного названия](https://docs.vdms.com/cdn/Content/HRE/M/D-Brand-Name-Literal.htm)) <br> **-** Регулярное выражение ([имя бренда Regex](https://docs.vdms.com/cdn/Content/HRE/M/D-Brand-Name-Regex.htm)) <br> **-** Специальный шаблон (шаблон [фирменного названия](https://docs.vdms.com/cdn/Content/HRE/M/D-Brand-Name-Wildcard.htm)) |
| ОС устройства | Определяет запросы, соответствующие операционной системе устройства: <br> **-** Конкретное значение ([ЛИТЕРАЛ ОС устройства](https://docs.vdms.com/cdn/Content/HRE/M/D-Device-OS-Literal.htm)) <br> **-** Регулярное выражение (выражение [ОС устройства](https://docs.vdms.com/cdn/Content/HRE/M/D-Device-OS-Regex.htm)) <br> **-** Конкретный шаблон (шаблон [ОС устройства](https://docs.vdms.com/cdn/Content/HRE/M/D-Device-OS-Wildcard.htm)) |
| Версия ОС устройства | Определяет запросы, если версия ОС устройства совпадает с: <br> **-** Конкретное значение ([литерал версии ОС устройства](https://docs.vdms.com/cdn/Content/HRE/M/D-Device-OS-Version-Literal.htm)) <br> **-** Регулярное выражение ([ОС устройства версия Regex](https://docs.vdms.com/cdn/Content/HRE/M/D-Device-OS-Version-Regex.htm)) <br> **-** Конкретный шаблон (шаблон [версии ОС устройства](https://docs.vdms.com/cdn/Content/HRE/M/D-Device-OS-Version-Wildcard.htm)) |
| [Двойная ориентация?](https://docs.vdms.com/cdn/Content/HRE/M/D-Dual-Orientation.htm) | Определяет запросы с поддержкой двойной ориентации устройства. |
| Предпочитаемое DTD для HTML | Определяет запросы, указывающие, соответствует ли предпочтительное в HTML устройства DTD: <br> **-** Конкретное значение ([предпочтительный ЛИТЕРАЛ DTD в формате HTML](https://docs.vdms.com/cdn/Content/HRE/M/D-HTML-Preferred-DTD-Literal.htm)) <br> **-** Регулярное выражение ([предпочтительное регулярное выражение DTD в формате HTML](https://docs.vdms.com/cdn/Content/HRE/M/D-HTML-Preferred-DTD-Regex.htm)) <br> **-** Особый шаблон ([предпочтительный подстановочный знак DTD в формате HTML](https://docs.vdms.com/cdn/Content/HRE/M/D-HTML-Preferred-DTD-Wildcard.htm)) |
| [Встраивание изображений?](https://docs.vdms.com/cdn/Content/HRE/M/D-Image-Inlining.htm) | Определяет запросы, если устройство поддерживает образы в кодировке Base64. |
| [Является ли Android?](https://docs.vdms.com/cdn/Content/HRE/M/D-Is-Android.htm) | Определяет запросы, использующиеся ли на устройстве ОС Android. |
| [Приложение?](https://docs.vdms.com/cdn/Content/HRE/M/D-Is-App.htm) | Определяет запросы, поступающие ли в собственном приложении запрошенное содержимое. |
| [Полный рабочий стол?](https://docs.vdms.com/cdn/Content/HRE/M/D-Is-Full-Desktop.htm) | Определяет запросы, обеспечивающие работу устройства в режиме полного рабочего стола. |
| [Является ли iOS?](https://docs.vdms.com/cdn/Content/HRE/M/D-Is-iOS.htm) | Определяет запросы, использующие устройство iOS. |
| [Робот?](https://docs.vdms.com/cdn/Content/HRE/M/D-Is-Robot.htm) | Определяет запросы, определяющие, считается ли устройство автоматическим клиентом HTTP (например, обходчиком робота). |
| [Есть ли интеллектуальные телепередачи?](https://docs.vdms.com/cdn/Content/HRE/M/D-Is-Smart-TV.htm) | Определяет запросы с помощью, является ли устройство интеллектуальным ТВ. |
| [Смартфоны?](https://docs.vdms.com/cdn/Content/HRE/M/D-Is-Smartphone.htm) | Определяет запросы, если устройство является смартфоном.
| [Планшет?](https://docs.vdms.com/cdn/Content/HRE/M/D-Is-Tablet.htm) | Определяет запросы, если устройство является планшетом. |
| [Является ли сенсорный экран?](https://docs.vdms.com/cdn/Content/HRE/M/D-Is-Touchscreen.htm) | Определяет запросы, указывающие, является ли основное указывающее устройство устройства сенсорным экраном. |
| [Windows Phone?](https://docs.vdms.com/cdn/Content/HRE/M/D-Is-Windows-Phone.htm) | Определяет запросы, если устройство является сервером Windows Mobile 6.5/Windows Phone 7 или более поздней версии. |
| [Беспроводное устройство?](https://docs.vdms.com/cdn/Content/HRE/M/D-Is-Wireless-Device.htm) | Определяет запросы, которые определяют, является ли устройство беспроводным. 
| Маркетинговое имя | Определяет запросы, которые указывают, соответствует ли маркетинговое имя устройства: <br> **-** Конкретное значение ([литерал маркетингового имени](https://docs.vdms.com/cdn/Content/HRE/M/D-Marketing-Name-Literal.htm)) <br> **-** Регулярное выражение (выражение для [коммерческих имен](https://docs.vdms.com/cdn/Content/HRE/M/D-Marketing-Name-Regex.htm)) <br> **-** Специальный шаблон (шаблон [маркетингового имени](https://docs.vdms.com/cdn/Content/HRE/M/D-Marketing-Name-Wildcard.htm)) |
| Браузер мобильного устройства | Определяет запросы, совпадающие с тем, соответствует ли браузер устройства: <br> **-** Конкретное значение ([литерал мобильного браузера](https://docs.vdms.com/cdn/Content/HRE/M/D-Mobile-Browser-Literal.htm)) <br> **-** Регулярные выражения ([регулярное выражение браузера для мобильных устройств](https://docs.vdms.com/cdn/Content/HRE/M/D-Mobile-Browser-Regex.htm)) <br> **-** Специальный шаблон (шаблон [мобильного браузера](https://docs.vdms.com/cdn/Content/HRE/M/D-Mobile-Browser-Wildcard.htm)) |
| Версия браузера мобильного устройства | Определяет запросы, указывающие, соответствует ли версия браузера устройства: <br> **-** Конкретное значение ([литерал версии браузера мобильного устройства](https://docs.vdms.com/cdn/Content/HRE/M/D-Mobile-Browser-Version-Literal.htm)) <br> **-** Регулярное выражение (для [мобильного браузера версия Regex](https://docs.vdms.com/cdn/Content/HRE/M/D-Mobile-Browser-Version-Regex.htm)) <br> **-** Специальный шаблон (шаблон [версии браузера для мобильных устройств](https://docs.vdms.com/cdn/Content/HRE/M/D-Mobile-Browser-Version-Wildcard.htm)) |
| Имя модели | Определяет запросы, если имя модели устройства совпадает с: <br> **-** Конкретное значение ([литерал имени модели](https://docs.vdms.com/cdn/Content/HRE/M/D-Model-Name-Literal.htm)) <br> **-** Регулярное выражение ([имя модели Regex](https://docs.vdms.com/cdn/Content/HRE/M/D-Model-Name-Regex.htm)) <br> **-** Специальный шаблон (шаблон [имени модели](https://docs.vdms.com/cdn/Content/HRE/M/D-Model-Name-Wildcard.htm)) |
| [Последовательное скачивание?](https://docs.vdms.com/cdn/Content/HRE/M/D-Progressive-Download.htm) | Определяет запросы, если устройство поддерживает прогрессивную загрузку. |
| Дата выпуска | Определяет запросы, указывающие, совпадает ли Дата выпуска устройства со следующим: <br> **-** Конкретное значение ([литерал даты выпуска](https://docs.vdms.com/cdn/Content/HRE/M/D-Release-Date-Literal.htm)) <br> **-** Регулярное выражение ([Дата выпуска Regex](https://docs.vdms.com/cdn/Content/HRE/M/D-Release-Date-Regex.htm)) <br> **-** Специальный шаблон ([шаблон даты выпуска](https://docs.vdms.com/cdn/Content/HRE/M/D-Release-Date-Wildcard.htm)) |
| [Высота в разрешении](https://docs.vdms.com/cdn/Content/HRE/M/D-Resolution-Height.htm) | Определяет запросы по высоте устройства. |
| [Ширина в разрешении](https://docs.vdms.com/cdn/Content/HRE/M/D-Resolution-Width.htm) | Определяет запросы по ширине устройства. |

**[К началу](#top)**

### <a name="location"></a><a name="location"></a>Расположение

Эти условия соответствия предназначены для определения запросов на основе расположения запрашивающей стороны.

| Имя       | Назначение                                                           |
|------------|-------------------------------------------------------------------|
| [Номер AS](https://docs.vdms.com/cdn/Content/HRE/M/AS-Number.htm) | Определяет запросы, поступающие из конкретной сети. |
| Название города | Определяет запросы, полученные из города, имя которого соответствует: <br> **-** Конкретное значение ([литерал названия города](https://docs.vdms.com/cdn/Content/HRE/M/City-Name-Literal.htm)) <br> **-** Регулярное выражение ([название города, Regex](https://docs.vdms.com/cdn/Content/HRE/M/City-Name-Regex.htm)) |
| [Континент](https://docs.vdms.com/cdn/Content/HRE/M/Continent.htm) | Определяет запросы, которые берутся из указанных континентов. |
| [Страна](https://docs.vdms.com/cdn/Content/HRE/M/Country.htm) | Определяет запросы, поступающие из указанных стран. |
| [Код DMA](https://docs.vdms.com/cdn/Content/HRE/M/DMA-Code.htm) | Определяет запросы, которые берутся из указанных metros (назначенные области рынка). |
| [Широта](https://docs.vdms.com/cdn/Content/HRE/M/Latitude.htm) | Определяет запросы, которые берутся из указанных широт. |
| [Долгот](https://docs.vdms.com/cdn/Content/HRE/M/Longitude.htm) | Определяет запросы, которые берутся из указанных долгот. |
| [Код Metro](https://docs.vdms.com/cdn/Content/HRE/M/Metro-Code.htm) | Определяет запросы, которые берутся из указанных metros (назначенные области рынка). |
| [Почтовый индекс](https://docs.vdms.com/cdn/Content/HRE/M/Postal-Code.htm) | Определяет запросы, которые берутся из указанных почтовых индексов. |
| [Код региона](https://docs.vdms.com/cdn/Content/HRE/M/Region-Code.htm) | Определяет запросы, которые берутся из указанных регионов. |

> [!NOTE]
> **Следует ли использовать код Metro или код DMA?** <br>
Оба эти условия соответствия обеспечивают одну и ту же возможность. Однако мы рекомендуем использовать условие соответствия кода Metro для распознавания запросов по каналу DMA.

**[К началу](#top)**

### <a name="origin"></a><a name="origin"></a>Исходный домен

Эти условия соответствия предназначены для определения запросов, которые указывают на хранилище CDN или сервер-источник клиента.

| Имя       | Назначение                                                           |
|------------|-------------------------------------------------------------------|
| [Источник CDN](https://docs.vdms.com/cdn/Content/HRE/M/CDN-Origin.htm) | Определяет запросы к содержимому в хранилище CDN. |
| [Источник клиента](https://docs.vdms.com/cdn/Content/HRE/M/Customer-Origin.htm) | Определяет запросы к содержимому, хранящемуся на конкретном сервере-источнике клиента. |

**[К началу](#top)**

### <a name="request"></a><a name="request"></a>Запрос

Эти условия соответствия предназначены для определения запросов на основе их свойств.

| Имя              | Назначение                                                                |
|-------------------|------------------------------------------------------------------------|
| [IP-адрес клиента](https://docs.vdms.com/cdn/Content/HRE/M/Client-IP-Address.htm) | Определяет запросы, поступающие с конкретного IP-адреса. |
| Параметр файла cookie  | Определяет запрос, содержащий файл cookie, соответствующий: <br> **-** Конкретное значение ([литерал параметра cookie](https://docs.vdms.com/cdn/Content/HRE/M/Cookie-Parameter-Literal.htm)) <br> **-** Регулярное выражение ([выражение параметра cookie](https://docs.vdms.com/cdn/Content/HRE/M/Cookie-Parameter-Regex.htm) <br> **-** Конкретный шаблон (шаблон [параметра cookie](https://docs.vdms.com/cdn/Content/HRE/M/Cookie-Parameter-Wildcard.htm)) |
| [Пограничные записи CNAME](https://docs.vdms.com/cdn/Content/HRE/M/Edge-CNAME.htm) | Определяет запрос, который указывает на конкретную граничную запись CNAME. |
| Домен источников ссылок | Определяет запрос, на который указывает имя узла, совпадающее с: <br> **-** Конкретное значение ([литеральный литерал домена](https://docs.vdms.com/cdn/Content/HRE/M/Referring-Domain-Literal.htm)) <br> **-** Конкретный шаблон ([подстановочный знак домена](https://docs.vdms.com/cdn/Content/HRE/M/Referring-Domain-Wildcard.htm)) |
| Заголовок запроса | Определяет запрос, содержащий заголовок, соответствующий: <br> **-** Конкретное значение ([литерал заголовка запроса](https://docs.vdms.com/cdn/Content/HRE/M/Request-Header-Literal.htm)) <br> **-** Регулярное выражение (регулярные выражения [заголовка запроса](https://docs.vdms.com/cdn/Content/HRE/M/Request-Header-Regex.htm)) <br> **-** Конкретный шаблон (шаблон [заголовка запроса](https://docs.vdms.com/cdn/Content/HRE/M/Request-Header-Wildcard.htm)) |
| [Метод Request](https://docs.vdms.com/cdn/Content/HRE/M/Request-Method.htm) | Определяет запросы по их HTTP-методу. |
| [Схема запроса](https://docs.vdms.com/cdn/Content/HRE/M/Request-Scheme.htm) | Определяет запросы по их HTTP-протоколу. |

**[К началу](#top)**

### <a name="url"></a><a name="url"></a>URL-адрес

| Имя              | Назначение                                                                |
|-------------------|------------------------------------------------------------------------|
| путь_URL-адреса | Определяет запросы, указывающие, соответствует ли их относительный путь, включая имя файла, параметру: <br> **-** Конкретное значение ([литерал URL-пути](https://docs.vdms.com/cdn/Content/HRE/M/URL-Path-Literal.htm)) <br> **-** Регулярное выражение (выражение [URL-пути](https://docs.vdms.com/cdn/Content/HRE/M/URL-Path-Regex.htm)) <br> **-** Специальный шаблон (шаблон [URL-пути](https://docs.vdms.com/cdn/Content/HRE/M/URL-Path-Wildcard.htm)) |
| Каталог URL-пути | Определяет запросы, указывающие, соответствует ли относительный путь a: <br> **-** Конкретное значение ([литерал каталога URL-пути](https://docs.vdms.com/cdn/Content/HRE/M/URL-Path-Directory-Literal.htm)) <br> **-** Специальный шаблон ([подстановочный знак каталога URL-пути](https://docs.vdms.com/cdn/Content/HRE/M/URL-Path-Directory-Wildcard.htm)) |
| Расширение URL-пути | Определяет запросы, соответствующие расширениям файлов: <br> **-** Конкретное значение ([литерал расширения URL-пути](https://docs.vdms.com/cdn/Content/HRE/M/URL-Path-Extension-Literal.htm)) <br> **-** Специальный шаблон ([подстановочный знак расширения URL-пути](https://docs.vdms.com/cdn/Content/HRE/M/URL-Path-Extension-Wildcard.htm)) |
| Имя файла URL-пути | Определяет запросы, если их имена файлов совпадают со следующим: <br> **-** Конкретное значение ([литерал имени файла URL-пути](https://docs.vdms.com/cdn/Content/HRE/M/URL-Path-Filename-Literal.htm)) <br> **-** Конкретный шаблон (шаблон [имени файла URL-пути](https://docs.vdms.com/cdn/Content/HRE/M/URL-Path-Filename-Wildcard.htm)) |
| Запрос URL-адреса | Определяет запросы, соответствующие строке запроса: <br> **-** Конкретное значение ([литерал запроса URL-адреса](https://docs.vdms.com/cdn/Content/HRE/M/URL-Query-Literal.htm)) <br> **-** Регулярное выражение (выражение [запроса URL-адреса](https://docs.vdms.com/cdn/Content/HRE/M/URL-Query-Regex.htm)) <br> **-** Специальный шаблон (шаблон [запроса URL-адреса](https://docs.vdms.com/cdn/Content/HRE/M/URL-Query-Wildcard.htm)) |
| Параметр запроса URL-адреса | Определяет запросы, которые содержат параметр строки запроса, которому задано значение, совпадающее с: <br> **-** Конкретное значение ([литерал параметра запроса URL-адреса](https://docs.vdms.com/cdn/Content/HRE/M/URL-Query-Parameter-Literal.htm)) <br> **-** Конкретный шаблон (шаблон [параметра запроса URL-адреса](https://docs.vdms.com/cdn/Content/HRE/M/URL-Query-Parameter-Wildcard.htm)) |

**[К началу](#top)**

Новейшие условия соответствия см. в [документации для обработчика правил Verizon](https://docs.vdms.com/cdn/index.html#Quick_References/HRE_QR.htm#Conditio).

## <a name="next-steps"></a>Дальнейшие действия

- [Общие сведения о сети доставки содержимого Azure](cdn-overview.md)
- [Справочник по обработчику правил](cdn-verizon-premium-rules-engine-reference.md)
- [Условные выражения обработчика правил](cdn-verizon-premium-rules-engine-reference-conditional-expressions.md)
- [Возможности обработчика правил](cdn-verizon-premium-rules-engine-reference-features.md)
- [Переопределение стандартного поведения через HTTP с помощью обработчика правил](cdn-verizon-premium-rules-engine.md)
