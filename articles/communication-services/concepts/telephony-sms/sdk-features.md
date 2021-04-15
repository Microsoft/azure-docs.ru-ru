---
title: Общие сведения о пакете SDK для SMS для Служб коммуникации Azure
titleSuffix: An Azure Communication Services concept document
description: Содержит общие сведения о пакете SDK для SMS и его возможностях.
author: mikben
manager: jken
services: azure-communication-services
ms.author: prakulka
ms.date: 03/26/2021
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: c25dfea077510580daf2c1aab584ab9ff5caa7fe
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105930446"
---
# <a name="sms-sdk-overview"></a>Общие сведения о пакете SDK для SMS

[!INCLUDE [Public Preview Notice](../../includes/public-preview-include-phone-numbers.md)]

[!INCLUDE [Regional Availability Notice](../../includes/regional-availability-include.md)]

Пакеты SDK для текстовых сообщений (SMS) для Служб коммуникации Azure можно использовать для добавления в приложения возможности обмена текстовыми сообщениями.

## <a name="sms-sdk-capabilities"></a>Возможности пакета SDK для SMS

В следующем списке содержится набор функций, которые сейчас доступны в наших пакетах SDK.

| Группа функций | Функция                                                                            | JS  | Java | .NET | Python |
| ----------------- | ------------------------------------------------------------------------------------- | --- | ---- | ---- | ------ |
| Основные возможности | Отправка и получение текстовых сообщений                                                         | ✔️   | ✔️    | ✔️    | ✔️      |
|                   | Включение функции получения отчетов о доставке отправленных сообщений                                             | ✔️   | ✔️    | ✔️    | ✔️      |
|                   | Все наборы символов (поддержка языков и Юникода)                                         | ✔️   | ✔️    | ✔️    | ✔️      |
|                   | Поддержка длинных сообщений (до 2048 байтов)                                          | ✔️   | ✔️    | ✔️    | ✔️      |
|                   | Автоматическое объединение длинных сообщений                                                   | ✔️   | ✔️    | ✔️    | ✔️      |
|                   | Отправка сообщений нескольким получателям за раз                                        | ✔️   | ✔️    | ✔️    | ✔️      |
|                   | Поддержка идемпотентности                                                               | ✔️   | ✔️    | ✔️    | ✔️      |
|                   | Пользовательские теги для сообщений.                                                             | ✔️   | ✔️    | ✔️    | ✔️      |
| События            | Использование Сетки событий для настройки веб-перехватчиков с целью получения входящих сообщений и отчетов о доставке | ✔️   | ✔️    | ✔️    | ✔️      |
| номер телефона;      | Бесплатные номера                                                                     | ✔️   | ✔️    | ✔️    | ✔️      |
| Вызовы ТСОП      | Добавление возможностей вызова ТСОП по бесплатному номеру телефона с поддержкой SMS                    | ✔️   | ✔️    | ✔️    | ✔️      |

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Начало работы с функцией отправки текстовых сообщений](../../quickstarts/telephony-sms/send.md)

Вас могут заинтересовать следующие документы:

- Знакомство с общими [понятиями обмена SMS](../telephony-sms/concepts.md)
- Получение [номера телефона](../../quickstarts/telephony-sms/get-phone-number.md), поддерживающего обмен SMS
- [Типы телефонных номеров в Службах коммуникации Azure](../telephony-sms/plan-solution.md)
