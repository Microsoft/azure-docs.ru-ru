---
title: Общие именованные сущности
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 01/15/2021
ms.author: aahi
ms.openlocfilehash: c1ff099dd6dffe06e9707ff23fffd57ae753ab64
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99500383"
---
Функция NER для Анализ текста возвращает следующие общие (не идентифицирующие) категории сущностей. Например, при отправке запросов в `/entities/recognition/general` конечную точку.


| Категория | Описание                          |
|------------|-------------|--------------------------------------|-------------------------------------------------------------|--------------------------------------|
| [Person](#category-person)     | Имена людей.  |
| [персонтипе](#category-persontype) | Типы заданий или роли, удерживаемые человеком. |
| [Расположение](#category-location)    | Естественные и человеческие ориентиры, структуры, географические функции и геофункциональные объекты |
| [Организация](#category-organization)  | Компании, «неправительственные группы», «музыкальные зоны», «спорт», «государственные органы» и «общественные организации».  |
| [Событие](#category-event)  | Исторические, социальные и естественным образом происходящие события. |
| [Продукт](#category-product) | Физические объекты различных категорий. |
| [Навык](#category-skill) | Возможность, навык или опыт.  |
| [Адрес](#category-address) | Полные адреса электронной почты.  |
| [Номер телефона](#category-phonenumber) | Номера телефонов. |
| [Электронная почта](#category-email) | Адреса электронной почты. |
| [URL-адрес](#category-url) | URL-адреса веб-сайтов. |
| [СМ](#category-ip) | Сетевые IP-адреса. |
| [DateTime](#category-datetime) | Даты и время суток. |
| [Количество](#category-quantity) | Числовые размеры и единицы измерения. |


### <a name="category-person"></a>Категория: Person

Эта категория содержит следующую сущность:

:::row:::
    :::column span="":::
        **Сущность**

        Человек

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Имена людей.
      
    :::column-end:::
    :::column span="2":::
      **Поддерживаемые языки документов**

      `ar`, `cs`, `da`, `nl`, `en`, `fi`, `fr`, `de`, `he`, <br> `hu`, `it`, `ja`, `ko`, `no`, `pl`, `pt-br`, `pt`-`pt`, `ru`, `es`, `sv`, `tr`   
      
   :::column-end:::
:::row-end:::

### <a name="category-persontype"></a>Категория: Персонтипе

Эта категория содержит следующую сущность:


:::row:::
    :::column span="":::
        **Сущность**

        персонтипе

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Типы заданий или роли, удерживаемые лицом
      
    :::column-end:::
    :::column span="2":::
      **Поддерживаемые языки документов**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`  
      
   :::column-end:::
:::row-end:::

### <a name="category-location"></a>Категория: расположение

Эта категория содержит следующую сущность:

:::row:::
    :::column span="":::
        **Сущность**

        Расположение

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Естественные и человеческие ориентиры, структуры, географические функции и нестоящие сущности.
      
    :::column-end:::
    :::column span="2":::
      **Поддерживаемые языки документов**

      `ar`, `cs`, `da`, `nl`, `en`, `fi`, `fr`, `de`, `he`, `hu`, `it`, `ja`, `ko`, `no`, `pl`, `pt-br`, `pt-pt`, `ru`, `es`, `sv`, `tr`   
      
   :::column-end:::
:::row-end:::

#### <a name="subcategories"></a>Подкатегории

Сущность в этой категории может иметь следующие подкатегории.

:::row:::
    :::column span="":::
        **Подкатегория сущности**

        Геоадминистративная сущность (ГПЕ)

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Города, страны и регионы, Штаты.
      
    :::column-end:::
    :::column span="2":::
      **Поддерживаемые языки документов**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`   
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Структурное

    :::column-end:::
    :::column span="2":::

        Структуры Манмаде. 
      
    :::column-end:::
    :::column span="2":::

      `en`   
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Географических

    :::column-end:::
    :::column span="2":::

        Географические и естественные функции, такие как рек, океанские и собственные.
      
    :::column-end:::
    :::column span="2":::

      `en`   
      
   :::column-end:::
:::row-end:::

### <a name="category-organization"></a>Категория: Организация

Эта категория содержит следующую сущность:

:::row:::
    :::column span="":::
        **Сущность**

        Организация

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Компании, «неправительственные группы», «музыкальные зоны», «спорт», «государственные органы» и «общественные организации». Национальные и религионсы не включаются в этот тип сущности.
      
    :::column-end:::
    :::column span="2":::
      **Поддерживаемые языки документов**

      `ar`, `cs`, `da`, `nl`, `en`, `fi`, `fr`, `de`, `he`, `hu`, `it`, `ja`, `ko`, `no`, `pl`, `pt-br`, `pt-pt`, `ru`, `es`, `sv`, `tr`   
      
   :::column-end:::
:::row-end:::

#### <a name="subcategories"></a>Подкатегории

Сущность в этой категории может иметь следующие подкатегории.

:::row:::
    :::column span="":::
        **Подкатегория сущности**

        Медицина

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Медицинские компании и группы.
      
    :::column-end:::
    :::column span="2":::
      **Поддерживаемые языки документов**

      `en`   
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Обмен на фондовой бирже

    :::column-end:::
    :::column span="2":::

        Группы обмена фондовой биржи. 
      
    :::column-end:::
    :::column span="2":::

      `en`   
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Спорт

    :::column-end:::
    :::column span="2":::

        Организации, связанные с спортивными делами.
      
    :::column-end:::
    :::column span="2":::

      `en`   
      
   :::column-end:::
:::row-end:::

### <a name="category-event"></a>Категория: событие

Эта категория содержит следующую сущность:

:::row:::
    :::column span="":::
        **Сущность**

        Событие

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Исторические, социальные и естественным образом происходящие события.
      
    :::column-end:::
    :::column span="2":::
      **Поддерживаемые языки документов**

      `en`, `es` , `fr` , `de` , `it` , `zh-hans` , `ja` , `ko` `pt-pt` и `pt-br`  
      
   :::column-end:::
:::row-end:::

#### <a name="subcategories"></a>Подкатегории

Сущность в этой категории может иметь следующие подкатегории.

:::row:::
    :::column span="":::
        **Подкатегория сущности**

        Различия

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Культурные мероприятия и праздники.
      
    :::column-end:::
    :::column span="2":::
      **Поддерживаемые языки документов**

      `en`   
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Natural

    :::column-end:::
    :::column span="2":::

        Естественное возникновение событий.
      
    :::column-end:::
    :::column span="2":::

      `en`   
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Спорт

    :::column-end:::
    :::column span="2":::

        Спортивные мероприятия.
      
    :::column-end:::
    :::column span="2":::

      `en`   
      
   :::column-end:::
:::row-end:::

### <a name="category-product"></a>Категория: продукт

Эта категория содержит следующую сущность:

:::row:::
    :::column span="":::
        **Сущность**

        Продукт

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Физические объекты различных категорий.
      
    :::column-end:::
    :::column span="2":::
      **Поддерживаемые языки документов**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`  
      
   :::column-end:::
:::row-end:::


#### <a name="subcategories"></a>Подкатегории

Сущность в этой категории может иметь следующие подкатегории.

:::row:::
    :::column span="":::
        **Подкатегория сущности**

        Вычисление продуктов
    :::column-end:::
    :::column span="2":::
        **Сведения**

        Вычисление продуктов.
      
    :::column-end:::
    :::column span="2":::
      **Поддерживаемые языки документов**

      `en`   
      
   :::column-end:::
:::row-end:::

### <a name="category-skill"></a>Категория: навык

Эта категория содержит следующую сущность:

:::row:::
    :::column span="":::
        **Сущность**

        Навык

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Возможность, навык или опыт.
      
    :::column-end:::
    :::column span="2":::
      **Поддерживаемые языки документов**

      `en`  
      
   :::column-end:::
:::row-end:::

### <a name="category-address"></a>Категория: адрес

Эта категория содержит следующую сущность:

:::row:::
    :::column span="":::
        **Сущность**

        Адрес

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Полный почтовый адрес.
      
    :::column-end:::
    :::column span="2":::
      **Поддерживаемые языки документов**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`
      
   :::column-end:::
:::row-end:::

### <a name="category-phonenumber"></a>Категория: PhoneNumber

Эта категория содержит следующую сущность:

:::row:::
    :::column span="":::
        **Сущность**

        PhoneNumber

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Номера телефонов (только для телефонных номеров США и ЕС).
      
    :::column-end:::
    :::column span="2":::
      **Поддерживаемые языки документов**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt` `pt-br`
      
   :::column-end:::
:::row-end:::

### <a name="category-email"></a>Категория: Электронная почта

Эта категория содержит следующую сущность:

:::row:::
    :::column span="":::
        **Сущность**

        Email

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Адреса электронной почты.
      
    :::column-end:::
    :::column span="2":::
      **Поддерживаемые языки документов**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`
      
   :::column-end:::
:::row-end:::

### <a name="category-url"></a>Категория: URL-адрес

Эта категория содержит следующую сущность:

:::row:::
    :::column span="":::
        **Сущность**

        URL-адрес

    :::column-end:::
    :::column span="2":::
        **Сведения**

        URL-адреса веб-сайтов. 
      
    :::column-end:::
    :::column span="2":::
      **Поддерживаемые языки документов**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`
      
   :::column-end:::
:::row-end:::

### <a name="category-ip"></a>Категория: IP-адрес

Эта категория содержит следующую сущность:

:::row:::
    :::column span="":::
        **Сущность**

        IP-адрес

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Сетевые IP-адреса. 
      
    :::column-end:::
    :::column span="2":::
      **Поддерживаемые языки документов**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`
      
   :::column-end:::
:::row-end:::

### <a name="category-datetime"></a>Категория: DateTime

Эта категория содержит следующие сущности:

:::row:::
    :::column span="":::
        **Сущность**

        Дата/время

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Даты и время суток. 
      
    :::column-end:::
    :::column span="2":::
      **Поддерживаемые языки документов**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`
      
   :::column-end:::
:::row-end:::

Сущности в этой категории могут иметь следующие подкатегории

#### <a name="subcategories"></a>Подкатегории

Сущность в этой категории может иметь следующие подкатегории.

:::row:::
    :::column span="":::
        **Подкатегория сущности**

        Дата

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Календарные даты.
      
    :::column-end:::
    :::column span="2":::
      **Поддерживаемые языки документов**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `pt-pt`, `pt-br`   
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        время;

    :::column-end:::
    :::column span="2":::

        Время суток.
      
    :::column-end:::
    :::column span="2":::

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `pt-pt`, `pt-br`   
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Диапазон дат

    :::column-end:::
    :::column span="2":::

        Диапазоны дат.
      
    :::column-end:::
    :::column span="2":::

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `pt-pt`, `pt-br`  
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Диапазон времени

    :::column-end:::
    :::column span="2":::

        Диапазоны времени.
      
    :::column-end:::
    :::column span="2":::

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `pt-pt`, `pt-br`  
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Duration

    :::column-end:::
    :::column span="2":::

        Длительности.
      
    :::column-end:::
    :::column span="2":::

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `pt-pt`, `pt-br`  
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Присвойте параметру

    :::column-end:::
    :::column span="2":::

        Набор, повторяющиеся значения времени.
      
    :::column-end:::
    :::column span="2":::

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `pt-pt`, `pt-br`  
      
   :::column-end:::
:::row-end:::

### <a name="category-quantity"></a>Категория: количество

Эта категория содержит следующие сущности:

:::row:::
    :::column span="":::
        **Сущность**

        Количество

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Числа и числовые величины.
      
    :::column-end:::
    :::column span="2":::
      **Поддерживаемые языки документов**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`
      
   :::column-end:::
:::row-end:::

#### <a name="subcategories"></a>Подкатегории

Сущность в этой категории может иметь следующие подкатегории.

:::row:::
    :::column span="":::
        **Подкатегория сущности**

        Число

    :::column-end:::
    :::column span="2":::
        **Сведения**

        числа.
      
    :::column-end:::
    :::column span="2":::
      **Поддерживаемые языки документов**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `pt-pt`, `pt-br`   
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::
        Процент

    :::column-end:::
    :::column span="2":::

        Процентов
      
    :::column-end:::
    :::column span="2":::

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `pt-pt`, `pt-br`   
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::
        Порядковые номера

    :::column-end:::
    :::column span="2":::

        Порядковые номера.
      
    :::column-end:::
    :::column span="2":::

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `pt-pt`, `pt-br`   
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::
        возраст;

    :::column-end:::
    :::column span="2":::

        Устаревают.
      
    :::column-end:::
    :::column span="2":::

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `pt-pt`, `pt-br`   
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::
        Валюта

    :::column-end:::
    :::column span="2":::

        Друг
      
    :::column-end:::
    :::column span="2":::

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `pt-pt`, `pt-br`   
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::
        Измерения

    :::column-end:::
    :::column span="2":::

        Измерения и измерения.
      
    :::column-end:::
    :::column span="2":::

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `pt-pt`, `pt-br`   
      
   :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::
        температура;

    :::column-end:::
    :::column span="2":::

        Измеряем.
      
    :::column-end:::
    :::column span="2":::

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `pt-pt`, `pt-br`   
      
   :::column-end:::
:::row-end:::
