---
title: включить файл
description: включить файл
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.date: 06/04/2020
ms.subservice: language-understanding
ms.topic: include
ms.custom: include file
ms.author: aahi
ms.reviewer: roy-har
ms.openlocfilehash: a021b7c5b26e1bca1e3f623ad762c2f3ee8d6020
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "97820623"
---
Создайте приложение Pizza.

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

Если отобразится диалоговое окно **How to create an effective LUIS app** (Как создать эффективное приложение LUIS), закройте его.

## <a name="train-and-publish-the-pizza-app"></a>Обучение и публикация приложения для заказа пиццы

Должна отобразиться страница **Intents** (Намерения) со списком намерений в приложении для заказа пиццы.

[!INCLUDE [How to train](howto-train.md)]

[!INCLUDE [How to publish](howto-publish.md)]

## <a name="add-an-authoring-resource-to-the-pizza-app"></a>Добавление ресурса разработки в приложение Pizza

1. Щелкните **MANAGE** (Управление).
1. Щелкните **Azure Resources** (Ресурсы Azure).
1. Щелкните **Authoring Resource** (Ресурс разработки).
1. Щелкните **Change authoring resource** (Изменить ресурс разработки).

Если у вас есть ресурс разработки, введите для него **имя клиента**, **имя подписки** и **имя ресурса LUIS**.

Если у вас нет ресурса разработки, сделайте следующее:

1. Выберите **Create new resource** (Создать ресурс).
1. Введите **имя арендатора**, **имя ресурса**, **имя подписки** и **имя группы ресурсов Azure**.

Теперь приложение для заказа пиццы можно использовать.

## <a name="record-the-access-values-for-your-pizza-app"></a>Запись значений доступа для приложения Pizza

Чтобы использовать новое приложение Pizza, вам потребуются его идентификатор, а также ключ и конечная точка разработки. Для получения прогнозов потребуется отдельная конечная точка прогнозирования и ключ прогнозирования.

Вот как найти эти значения:

1. На странице **Intents** (Намерения) щелкните **MANAGE** (Управление).
1. На странице **Application Settings** (Параметры приложения) запишите значение **App ID** (Идентификатор приложения).
1. Щелкните **Azure Resources** (Ресурсы Azure).
1. Щелкните **Authoring Resource** (Ресурс разработки).
1. На вкладках **Authoring Resource** (Ресурс разработки) и **Prediction Resources** (Ресурсы прогнозирования) запишите **первичный ключ**. Это значение является ключом разработки.
1. Скопируйте значение **Endpoint URL** (URL-адрес конечной точки). Это значение является конечной точкой разработки.
