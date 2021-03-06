---
title: Просмотр результатов тестов системы и развертывания — Custom Translator
titleSuffix: Azure Cognitive Services
description: Если обучение прошло успешно, ознакомьтесь с тестами системы, чтобы проанализировать результаты обучения. Если вы удовлетворены результатами обучения, разместите запрос на развертывание обученной модели.
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 08/17/2020
ms.author: lajanuar
ms.topic: conceptual
ms.openlocfilehash: cae2c95e56312c58d396d1e578f4677ce2b14aa2
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98895940"
---
# <a name="view-system-test-results"></a>Просмотр результатов теста системы

Если обучение прошло успешно, ознакомьтесь с тестами системы, чтобы проанализировать результаты обучения. Если вы удовлетворены результатами обучения, разместите запрос на развертывание обученной модели.

## <a name="system-test-results-page"></a>Страница с результатами тестов системы

Выберите проект, а затем — вкладку моделей данного проекта, найдите нужную модель и, наконец, перейдите на вкладку тестирования.

На вкладке тестирования отображаются такие элементы:

1.  **Результаты теста системы:** Результат тестового процесса в обучения. В результате тестирования создается оценка BLEU.

    **Sentence Count** (Число предложений). Количество параллельных предложений, которое было использовано в тестовом наборе.

     **BLEU Score** (Оценка BLEU). Оценка BLEU, сформированная для модели после завершения обучения.

    **Состояние**. Указывает состояние теста: завершен или в процессе выполнения.

    ![Результаты тестов системы](media/how-to/how-to-system-test-results.png)

2.  Щелкнув результаты тестов системы, вы перейдете на страницу сведений о результатах тестирования. На этой странице показан машинный перевод предложений, которые были частью тестового набора данных.

3.  Таблица на странице сведений о результатах тестов содержит два столбца — по одному для каждого языка в паре. В столбце исходного языка отображается предложение, которое будет переведено. В столбце целевого языка в каждой строке содержатся два предложения.

    **REF:** Это эталонный перевод исходного предложения, который содержится в тестовом наборе данных.

    **MT:** Это автоматический перевод исходного предложения, который выполнен моделью, созданной после обучения.

    ![Сравнение результатов тестов системы](media/how-to/how-to-system-test-results-2.png)

## <a name="download-test"></a>Скачивание теста

Щелкните ссылку скачивания перевода, чтобы скачать ZIP-файл. ZIP-файл содержит машинные переводы исходных предложений в тестовом наборе данных.

![Скачивание теста](media/how-to/how-to-system-test-download.png)

Этот скачанный ZIP-архив содержит три файла.

1.  **custom.mt.txt:** Этот файл содержит переводы машин из предложений языка исходного кода на целевом языке, выполненном моделью, обученной данными пользователя.

2.  **ref.txt:** Этот файл содержит переданные пользователем переводы предложений языка исходного кода на целевом языке.

3.  **source.txt:** Этот файл содержит предложения на исходном языке.

    ![Скачанные результаты тестов системы](media/how-to/how-to-download-system-test.png)

## <a name="deploy-a-model"></a>Развертывание модели

Чтобы запросить развертывание:

1.  Выберите проект и перейдите на вкладку "Модели".

2. Для успешно обученной модели, которая не развернута, отображается кнопка "Развернуть".

    ![Снимок экрана, на котором выделена кнопка развертывания для развертывания модели.](media/how-to/how-to-deploy-model.png)

3.  Нажмите кнопку "Развернуть".
4.  Выберите **Развернуто** для регионов, в которых необходимо развернуть модель, и нажмите кнопку "Сохранить". Вы можете выбрать **Развернуто** для нескольких регионов.

    ![Снимок экрана, на котором показано, где можно развернуть или отменить развертывание модели.](media/how-to/how-to-deploy-model-regions.png)

5.  Состояние модели можно просмотреть в столбце "Состояние".

>[!Note]
>Пользовательский Переводчик поддерживает 10 развернутых моделей в рабочей области в любой момент времени.

## <a name="update-deployment-settings"></a>Обновление параметров развертывания

Чтобы обновить параметры развертывания, выполните следующие действия.

1.  Выберите проект и перейдите на вкладку **Модели**.

2. Для успешного развернутой модели отображается кнопка **Обновить**.

    ![Снимок экрана, на котором выделена кнопка обновления для обновления параметров развертывания.](media/how-to/how-to-update-undeploy-model.png)

3.  Нажмите кнопку **Обновить**.
4.  Выберите **Развернуто** или **Не развернуто** для регионов, в которых необходимо развернуть модель или отменить развертывание модели, и нажмите кнопку **Сохранить**.

    ![Развертывание модели](media/how-to/how-to-undeploy-model.png)

>[!Note]
>При выборе **Не развернуто** во всех регионах отменяется развертывание модели для всех регионов и модель переводится в неразвернутое состояние. После этого модель недоступна для использования.

## <a name="next-steps"></a>Дальнейшие действия

- Начните использовать развернутую пользовательскую модель перевода с помощью [API перевода текстов Майкрософт версии 3](../reference/v3-0-translate.md?tabs=curl).
- Узнайте, как [настроить параметры](how-to-manage-settings.md) для предоставления общего доступа к рабочей области и управлять ключом подписки.
- Узнайте, как [перенести рабочую область и проект](how-to-migrate.md) из [Microsoft Translator Hub](https://hub.microsofttranslator.com).