---
title: Документация по Swagger — служба речи
titleSuffix: Azure Cognitive Services
description: Документацию по Swagger можно использовать для автоматического создания пакетов SDK для различных языков программирования. Все операции в нашей службе поддерживаются Swagger.
services: cognitive-services
author: alexeyo26
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: reference
ms.date: 02/16/2021
ms.author: alexeyo
ms.openlocfilehash: d4369b66bacbe8cff4fc9712ffcd0cb5a370c439
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100636274"
---
# <a name="swagger-documentation"></a>Документация по Swagger

Служба распознавания речи предоставляет спецификацию Swagger для взаимодействия с несколькими интерфейсами API-интерфейса, которые используются для импорта данных, создания моделей, проверки точности тестовой модели, создания пользовательских конечных точек, добавления в очередь записей пакетной обработки и управления подписками. Большинство операций, доступных [в области пользовательское распознавание речи Speech Studio,](https://aka.ms/customspeech) можно выполнить программно с помощью этих API.

> [!NOTE]
> Служба речи имеет несколько интерфейсов API для преобразования [речи в текст](rest-speech-to-text.md) и преобразования [текста в речь](rest-text-to-speech.md).  
>
> Однако в спецификации Swagger задокументированы только преобразования [речи в текст REST API v 3.0](rest-speech-to-text.md#speech-to-text-rest-api-v30) и v 2.0. Сведения обо всех других API-интерфейсах службы речи, см. в документах, упомянутых в предыдущем абзаце.

## <a name="generating-code-from-the-swagger-specification"></a>Создание кода из спецификации Swagger

[Спецификация Swagger](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0) содержит возможности, которые позволяют быстро протестировать различные пути. Тем не менее иногда желательно сгенерировать код для всех путей, создав единую библиотеку вызовов, на основе которой потом можно будет создавать решения. Давайте рассмотрим процесс создания библиотеки Python.

Необходимо задать для Swagger регион своего ресурса речи. Вы можете подтвердить регион в **обзорной** части параметров ресурсов речи в портал Azure. Полный список поддерживаемых регионов доступен [здесь](regions.md#speech-to-text).

1. В браузере перейдите к спецификации Swagger для вашего [региона](regions.md#speech-to-text):  
       `https://<your-region>.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0`
1. На этой странице щелкните **Определение API** и щелкните **Swagger**. Скопируйте URL-адрес страницы, которая отображается.
1. В новом браузере перейдите по адресу [https://editor.swagger.io](https://editor.swagger.io)
1. Щелкните **файл**, щелкните **Импортировать URL-адрес**, вставьте URL-адрес и нажмите кнопку **ОК**.
1. Щелкните **создать клиент** и выберите **Python**. Клиентская библиотека загружается на компьютер в `.zip` файле.
1. Извлеките все содержимое из скачиваемого файла. Вы можете использовать `tar -xf` для извлечения всех элементов.
1. Установите извлеченный модуль в среду Python:  
      `pip install path/to/package/python-client`
1. Установленный пакет называется `swagger_client` . Убедитесь, что установка работала:  
       `python -c "import swagger_client"`

Вы можете использовать библиотеку Python, созданную с [примерами службы распознавания речи на GitHub](https://aka.ms/csspeech/samples).

## <a name="reference-documents"></a>Справочные документы

* [Swagger: преобразование речи в текст REST API v 3.0](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0)
* [REST API преобразования речи в текст;](rest-speech-to-text.md)
* [REST API преобразования текста в речь.](rest-text-to-speech.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Примеры речевых служб на GitHub](https://aka.ms/csspeech/samples).
* [Try the Speech service for free](overview.md#try-the-speech-service-for-free) (Бесплатное использование службы "Речь")
