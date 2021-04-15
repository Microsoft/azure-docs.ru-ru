---
title: Включить файл
description: включить файл
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.date: 12/29/2020
ms.subservice: language-understanding
ms.topic: include
ms.custom: include file
ms.author: aahi
ms.reviewer: roy-har
ms.openlocfilehash: 7aa2fba6ef551a745ccaf5b00f36021b9d8680ce
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "97820530"
---
1. Щелкните [pizza-app-for-luis-v6.json](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/luis/apps/pizza-app-for-luis-v6.json), чтобы открыть на сайте GitHub страницу с файлом `pizza-app-for-luis.json`.
1. Щелкните правой кнопкой мыши кнопку **Raw** (Необработанный) или коснитесь ее с задержкой и выберите **Save link as** (Сохранить ссылку как), чтобы сохранить `pizza-app-for-luis.json` на компьютере.
1. Войдите на [портал LUIS](https://www.luis.ai).
1. Щелкните [My Apps](https://www.luis.ai/applications) (Мои приложения).
1. На странице **My Apps** (Мои приложения) щелкните **+ New app for conversation** (+ Новое приложение для общения).
1. Щелкните **Import as JSON** (Импортировать как JSON).
1. В диалоговом окне **Import new app** (Импорт нового приложения) нажмите кнопку **Choose File** (Выбрать файл).
1. Выберите скачанный файл `pizza-app-for-luis.json` и щелкните **Open** (Открыть).
1. В диалоговом окне **Import new app** (Импорт нового приложения) в поле **Name** (Имя) введите имя приложения для заказа пиццы, а затем нажмите кнопку **Done** (Готово).

Приложение будет импортировано.

Если отобразится диалоговое окно **Как создать эффективное приложение LUIS**, закройте его.

## <a name="train-and-publish-the-pizza-app"></a>Обучение и публикация приложения для заказа пиццы

Должна отобразиться страница **Intents** (Намерения) со списком намерений в приложении для заказа пиццы.

[!INCLUDE [How to train](howto-train.md)]

[!INCLUDE [How to publish](howto-publish.md)]

Теперь приложение для заказа пиццы можно использовать.

## <a name="record-the-app-id-prediction-key-and-prediction-endpoint-of-your-pizza-app"></a>Запись идентификатора приложения для заказа пиццы, а также ключа прогнозирования и конечной точки прогнозирования

Чтобы использовать новое приложение для заказа пиццы, вам потребуются его идентификатор, ключ прогнозирования и конечная точка прогнозирования.

Вот как найти эти значения:

1. На странице **Intents** (Намерения) щелкните **MANAGE** (Управление).
1. На странице **Application Settings** (Параметры приложения) запишите значение **App ID** (Идентификатор приложения).
1. Щелкните **Azure Resources** (Ресурсы Azure).
1. На странице **Azure Resources** (Ресурсы Azure) запишите значение **Primary Key** (Первичный ключ). Это значение является ключом прогнозирования.
1. Скопируйте значение **Endpoint URL** (URL-адрес конечной точки). Это значение является конечной точкой прогнозирования.
