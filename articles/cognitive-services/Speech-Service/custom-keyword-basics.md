---
title: Создание ключевого слова краткое руководство — служба речи
titleSuffix: Azure Cognitive Services
description: Устройство всегда прослушивает ключевое слово (или фразу). Когда пользователь говорит о ключевом слове, устройство отправляет все последующие аудио в облако, пока пользователь не прекратит говорить. Настройка ключевого слова является эффективным способом различения устройства и усиления фирменной символики.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 11/03/2020
ms.author: trbye
ms.custom: devx-track-csharp
zone_pivot_groups: keyword-quickstart
ms.openlocfilehash: 49ac70b6881085f48c8bc3a12e31e4a1aa220c6a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95021955"
---
# <a name="get-started-with-custom-keyword"></a>Начало работы с Пользовательскими ключевыми словами

В этом кратком руководстве вы узнаете основы работы с пользовательскими ключевыми словами с помощью Speech Studio и речевого пакета SDK. Ключевое слово — это слово или короткая фраза, которая позволяет активировать ваш продукт. Вы создаете модели ключевых слов в Speech Studio, затем экспортируете файл модели, который используется в приложениях с пакетом SDK для распознавания речи.

## <a name="prerequisites"></a>Предварительные условия

Для выполнения действий, описанных в этой статье, требуется подписка на речь и речевой пакет SDK. Если у вас еще нет подписки, [попробуйте службу распознавания речи бесплатно](overview.md#try-the-speech-service-for-free). Чтобы получить пакет SDK, ознакомьтесь с [руководством по установке](quickstarts/setup-platform.md) платформы.

## <a name="create-a-keyword-in-speech-studio"></a>Создание ключевого слова в Speech Studio

Прежде чем использовать пользовательское ключевое слово, необходимо создать ключевое слово, используя страницу [настраиваемого ключевого слова](https://aka.ms/sdsdk-wakewordportal) в [Speech Studio](https://aka.ms/sdsdk-speechportal). После ввода ключевого слова он создает `.table` файл, который можно использовать с пакетом SDK для распознавания речи.

> [!IMPORTANT]
> Пользовательские модели ключевых слов и итоговые `.table` файлы можно создавать **только** в Speech Studio.
> Нельзя создавать пользовательские ключевые слова из пакета SDK или с помощью вызовов RESTFUL.

1. Перейдите в [Speech Studio](https://aka.ms/sdsdk-speechportal) и **Войдите в систему** или, если у вас еще нет подписки на речь, выберите [**создать подписку**](https://go.microsoft.com/fwlink/?linkid=2086754).

1. На странице [настраиваемое ключевое слово](https://aka.ms/sdsdk-wakewordportal) создайте **Новый проект**. 

1. Введите **имя**, необязательное **Описание** и выберите язык. Требуется один проект для каждого языка, а поддержка в настоящее время ограничена `en-US` языком.

    ![Описание проекта ключевого слова](media/custom-keyword/custom-kws-portal-new-project.png)

1. Выберите проект из списка. 

    ![Выбор проекта ключевого слова](media/custom-keyword/custom-kws-portal-project-list.png)

1. Чтобы создать новую модель ключевых слов, нажмите кнопку **обучение модели**.

1. Введите **имя** модели, необязательное **Описание** и **ключевое слово** , а затем нажмите кнопку **Далее**. Ознакомьтесь с [рекомендациями](./custom-keyword-overview.md#choose-an-effective-keyword) по выбору эффективного ключевого слова.

    ![Введите ключевое слово](media/custom-keyword/custom-kws-portal-new-model.png)

1. На портале создаются произношение кандидата для ключевого слова. Прослушивать каждый кандидат, нажимая кнопки воспроизведения и удаляя проверки рядом со всеми неправильными произношениями. Если проверяются только хорошие произношение, нажмите кнопку **обучение** , чтобы начать создание модели ключевых слов. 

    ![Снимок экрана, на котором показано, где вы выбираете правильный пронаунЦиатионс.](media/custom-keyword/custom-kws-portal-choose-prons.png)

1. Создание модели может занять до тридцати минут. После завершения работы с моделью список ключевых слов будет изменен с " **Обработка** выполнена" на " **выполнено** ". Затем можно скачать файл.

    ![Проверка ключевого слова](media/custom-keyword/custom-kws-portal-download-model.png)

1. Скачанный файл является `.zip` архивом. Извлеките архив, и вы увидите файл с `.table` расширением. Это файл, используемый с пакетом SDK, в следующем разделе, поэтому обязательно запишите его путь. имя файла отражает имя ключевого слова, например, ключевое слово **Activate Device** имеет имя файла `Activate_device.table` .

## <a name="use-a-keyword-model-with-the-sdk"></a>Использование модели ключевых слов с пакетом SDK

::: zone pivot="programming-language-csharp"
[!INCLUDE [C# Basics include](includes/how-to/keyword-recognition/keyword-basics-csharp.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [Python Basics include](includes/how-to/keyword-recognition/keyword-basics-python.md)]
::: zone-end

::: zone pivot="programming-languages-objectivec-swift"
[!INCLUDE [ObjectiveC/Swift Basics include](includes/how-to/keyword-recognition/keyword-basics-objc.md)]
::: zone-end

## <a name="next-steps"></a>Дальнейшие действия

Протестируйте пользовательское ключевое слово в [кратком руководстве по пакету SDK для речевых устройств](./speech-devices-sdk-quickstart.md?pivots=platform-android).