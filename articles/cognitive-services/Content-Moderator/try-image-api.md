---
title: Модерация изображений с помощью консоли API — Content Moderator
titleSuffix: Azure Cognitive Services
description: Используйте API модерации изображений в Azure Content Moderator для запуска рабочих процессов модерации "поиск и проверка" для графического содержимого.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 01/10/2019
ms.author: pafarley
ms.openlocfilehash: 542fadd5e5ab91be7b7113064bf8c998dae08d12
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92912945"
---
# <a name="moderate-images-from-the-api-console"></a>Модерация изображений с помощью консоли API

Используйте [API модерации изображений](https://westus.dev.cognitive.microsoft.com/docs/services/57cf753a3f9b070c105bd2c1/operations/57cf753a3f9b070868a1f66c) в Azure Content Moderator для запуска рабочих процессов модерации "поиск и проверка" для графического содержимого. Задание по созадаче просматривает содержимое на ненормативную лексику и сравнивает его с настраиваемыми и общими адресов.

## <a name="use-the-api-console"></a>Использование консоли API
Прежде чем можно будет опробовать API в веб-консоли, необходимо получить ключ подписки. Он указан на вкладке **Settings** (Параметры) в поле **Ocp-Apim-Subscription-Key**. Дополнительные сведения см. в этом [обзоре](overview.md).

1. Перейдите к [справочнику по API модерации изображений](https://westus.dev.cognitive.microsoft.com/docs/services/57cf753a3f9b070c105bd2c1/operations/57cf753a3f9b070868a1f66c).

   Откроется страница модерации **Image - Evaluate** (Изображение — оценка).

2. Для параметра **Open API testing console** (Открытая тестовая консоль API) выберите регион, лучше всего подходящий для вашего расположения. 

   ![Выбор региона на странице "Image - Evaluate" (Изображение — оценка)](images/test-drive-region.png)
  
   Откроется консоль API **Image - Evaluate** (Изображение — оценка).

3. В поле **Ocp-Apim-Subscription-Key** введите свой ключ подписки.

   ![Ввод ключа подписки в консоли "Image - Evaluate" (Изображение — оценка)](images/try-image-api-1.PNG)

4. В поле **Request body** (Текст запроса) укажите пример изображения по умолчанию или другое изображение для проверки. Вы можете отправить само изображение в двоичном формате или указать общедоступный URL-адрес этого изображения. 

   В этом примере используйте путь, указанный в поле **Request body** (Текст запроса), а затем щелкните **Send** (Отправить). 

   ![Поле "Request body" (Текст запроса) в консоли "Image - Evaluate" (Изображение — оценка)](images/try-image-api-2.PNG)

   Это изображение по указанному URL-адресу.

   ![Пример изображения в консоли "Image - Evaluate" (Изображение — оценка)](images/sample-image.jpg) 

5. Нажмите кнопку **Отправить**.

6. API возвращает оценку вероятности для каждой классификации. Он также сообщает, соответствует ли изображение условиям (**true** или **false**). 

   ![Оценка вероятности и определение выполнения условия в консоли "Image - Evaluate" (Изображение — оценка)](images/try-image-api-3.PNG)

## <a name="face-detection"></a>Определение лиц

API модерации изображений можно использовать для определения лиц на изображении. Это может быть удобно для соблюдения конфиденциальности, например, чтобы предотвратить публикацию чьего-либо лица на платформе. 

1. В [справочнике по API модерации изображений](https://westus.dev.cognitive.microsoft.com/docs/services/57cf753a3f9b070c105bd2c1/operations/57cf753a3f9b070868a1f66c) в меню слева, в разделе **Image** (Изображение), выберите **Find Faces** (Найти лица). 

   Откроется страница **Image - Find Faces** (Изображение — поиск лиц).

2. Для параметра **Open API testing console** (Открытая тестовая консоль API) выберите регион, лучше всего подходящий для вашего расположения. 

   ![Выбор региона на странице "Image - Find Faces" (Изображение — поиск лиц)](images/test-drive-region.png)

   Откроется консоль API **Image - Find Faces** (Изображение — поиск лиц).

3. Укажите изображение для проверки. Вы можете отправить само изображение в двоичном формате или указать общедоступный URL-адрес этого изображения. В этом примере используется ссылка на изображение из сюжета CNN.

   ![Пример изображения в консоли "Image - Find Faces" (Изображение — поиск лиц)](images/try-image-api-face-image.jpg)

   ![Пример запроса в консоли "Image - Find Faces" (Изображение — поиск лиц)](images/try-image-api-face-request.png)

4. Нажмите кнопку **Отправить**. В этом примере API находит два лица и возвращает их координаты в изображении.

   ![Поле "Response content" (Содержимое ответа) в консоли "Image - Find Faces" (Изображение — поиск лиц)](images/try-image-api-face-response.png)

## <a name="text-detection-via-ocr-capability"></a>Обнаружение текста с помощью функции OCR

С помощью функции OCR в Content Moderator находить текст в изображениях.

1. В [справочнике по API модерации изображений](https://westus.dev.cognitive.microsoft.com/docs/services/57cf753a3f9b070c105bd2c1/operations/57cf753a3f9b070868a1f66c) в меню слева, в разделе **Image** (Изображение), выберите **OCR**. 

   Откроется страница **Image - OCR** (Изображение — OCR).

2. Для параметра **Open API testing console** (Открытая тестовая консоль API) выберите регион, лучше всего подходящий для вашего расположения. 

   ![Выбор региона на странице "Image - OCR" (Изображение — OCR)](images/test-drive-region.png)

   Откроется консоль API **Image - OCR** (Изображение — OCR).

3. В поле **Ocp-Apim-Subscription-Key** введите свой ключ подписки.

4. В поле **Request body** (Текст запроса) укажите пример изображения по умолчанию. Это же изображение было использовано в предыдущем разделе.

5. Нажмите кнопку **Отправить**. Извлеченный текст отображается в формате JSON.

   ![Пример поля "Response content" (Содержимое ответа) в консоли "Image - OCR" (Изображение — OCR)](images/try-image-api-ocr.PNG)

## <a name="next-steps"></a>Дальнейшие действия

Используйте REST API в коде или следуйте указаниям в [кратком руководстве по пакету SDK для .NET](./client-libraries.md?pivots=programming-language-csharp%253fpivots%253dprogramming-language-csharp) , чтобы добавить в приложение отслеживание изображений.