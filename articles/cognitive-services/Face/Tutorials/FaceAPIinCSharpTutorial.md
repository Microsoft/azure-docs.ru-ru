---
title: Руководство по Определение лиц и отображение данных о них на изображении с использованием пакета SDK для .NET
titleSuffix: Azure Cognitive Services
description: С помощью этого руководства вы создадите простое приложение Windows, которое использует службу "Распознавание лиц" для определения и выделения лиц на изображении.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: tutorial
ms.date: 11/23/2020
ms.author: pafarley
ms.custom: devx-track-csharp
ms.openlocfilehash: 7a9caf1c1785055cbc81ef56958fe8ce2aca229c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102428309"
---
# <a name="tutorial-create-a-windows-presentation-framework-wpf-app-to-display-face-data-in-an-image"></a>Руководство по созданию приложения Windows Presentation Framework (WPF) для отображения данных о лицах на изображении

Из этого руководства вы узнаете, как с помощью службы "Распознавание лиц" Azure, поставляемого через клиентский пакет SDK для .NET, распознавать лица на изображении, а затем представлять эти данные в пользовательском интерфейсе. Вы создадите приложение WPF, которое распознает лица, рисует рамку вокруг них и отображает описание лица в строке состояния. 

В этом учебнике описаны следующие процедуры.

> [!div class="checklist"]
> - создание приложения WPF;
> - установка клиентской библиотеки службы Распознавания лиц;
> - использование клиентской библиотеки для обнаружения лиц на изображении;
> - выделение рамкой каждого обнаруженного лица;
> - отображение описания выделенного лица в строке состояния.

![Снимок экрана, на котором обнаруженные лица выделены прямоугольниками](../Images/getting-started-cs-detected.png)

Полный пример кода доступен в репозитории [Cognitive Face CSharp sample](https://github.com/Azure-Samples/Cognitive-Face-CSharp-sample) на сайте GitHub.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/), прежде чем начинать работу. 


## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/).
* Получив подписку Azure, перейдите к <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesFace"  title="Создание ресурса Распознавания лиц"  target="_blank">созданию ресурса Распознавания лиц</a> на портале Azure, чтобы получить ключ и конечную точку. После развертывания щелкните **Перейти к ресурсам**.
    * Для подключения приложения к API Распознавания лиц потребуется ключ и конечная точка из созданного ресурса. Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
    * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.
* [Создайте переменные среды](../../cognitive-services-apis-create-account.md#configure-an-environment-variable-for-authentication) для строки ключа и конечной точки службы с именами `FACE_SUBSCRIPTION_KEY` и `FACE_ENDPOINT` соответственно.
- Любой выпуск [Visual Studio](https://www.visualstudio.com/downloads/).

## <a name="create-the-visual-studio-project"></a>Создание проекта Visual Studio

Чтобы создать проект нового приложения WPF, сделайте следующее.

1. В Visual Studio откройте диалоговое окно "Новый проект". Разверните **Установленные**, а затем — **Visual C#** и выберите **Приложение WPF (.NET Framework)** .
1. Присвойте приложению имя **FaceTutorial** и нажмите кнопку **ОК**.
1. Получите необходимые пакеты NuGet. Щелкните проект правой кнопкой мыши в обозревателе решений и выберите **Управление пакетами NuGet**, а затем найдите и установите следующий пакет:
    - [Microsoft.Azure.CognitiveServices.Vision.Face 2.6.0-preview.1](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.Face/2.6.0-preview.1)

## <a name="add-the-initial-code"></a>Добавление исходного кода

В этом разделе вы добавите базовую платформу приложения без функций обработки лиц.

### <a name="create-the-ui"></a>Создание пользовательского интерфейса

Откройте файл *MainWindow.xaml* и замените его содержимое приведенным ниже кодом, который создаст окно интерфейса. Методы `FacePhoto_MouseMove` и `BrowseButton_Click` являются обработчиками событий, которые будут определены позже.

[!code-xaml[](~/Cognitive-Face-CSharp-sample/FaceTutorialCS/FaceTutorialCS/MainWindow.xaml?name=snippet_xaml)]

### <a name="create-the-main-class"></a>Создание основного класса

Откройте файл *MainWindow.xaml.cs* и добавьте пространства имен клиентской библиотеки, а также другие необходимые пространства имен. 

[!code-csharp[](~/Cognitive-Face-CSharp-sample/FaceTutorialCS/FaceTutorialCS/MainWindow.xaml.cs?name=snippet_using)]

Вставьте следующий код в класс **MainWindow**. Этот код создает экземпляр **FaceClient** с использованием ключа подписки и конечной точки.

[!code-csharp[](~/Cognitive-Face-CSharp-sample/FaceTutorialCS/FaceTutorialCS/MainWindow.xaml.cs?name=snippet_mainwindow_fields)]

Затем добавьте конструктор **MainWindow**. Он проверяет строку URL-адреса конечной точки и связывает ее с клиентским объектом.

[!code-csharp[](~/Cognitive-Face-CSharp-sample/FaceTutorialCS/FaceTutorialCS/MainWindow.xaml.cs?name=snippet_mainwindow_constructor)]

Наконец, добавьте методы **BrowseButton_Click** и **FacePhoto_MouseMove** в класс. Они соответствуют обработчикам событий, объявленным в файле *MainWindow.xaml*. Метод **BrowseButton_Click** создает объект **OpenFileDialog**, что позволяет пользователю выбрать изображение в формате JPG. В главном окне будет выведено событие. Вы вставите оставшийся код для **BrowseButton_Click** и **FacePhoto_MouseMove** на последующих шагах. Обратите также внимание на ссылку `faceList`&mdash;список объектов **DetectedFace**. Здесь ваше приложение будет хранить и вызывать фактические данные о лицах.

[!code-csharp[](~/Cognitive-Face-CSharp-sample/FaceTutorialCS/FaceTutorialCS/MainWindow.xaml.cs?name=snippet_browsebuttonclick_start)]

<!-- [!code-csharp[](~/Cognitive-Face-CSharp-sample/FaceTutorialCS/FaceTutorialCS/MainWindow.xaml.cs?name=snippet_browsebuttonclick_end)] -->

[!code-csharp[](~/Cognitive-Face-CSharp-sample/FaceTutorialCS/FaceTutorialCS/MainWindow.xaml.cs?name=snippet_mousemove_start)]

<!-- [!code-csharp[](~/Cognitive-Face-CSharp-sample/FaceTutorialCS/FaceTutorialCS/MainWindow.xaml.cs?name=snippet_mousemove_end)] -->

### <a name="try-the-app"></a>Использование приложения

В меню выберите команду **Начать**, чтобы протестировать приложение. В открывшемся окне приложения в левом нижнем углу нажмите кнопку **Обзор**. Появится диалоговое окно **открытия файла**. Выберите изображение из файловой системы и убедитесь, что оно отображается в окне. Закройте приложение и перейдите к следующему шагу.

![Снимок экрана с фотографией лиц без изменений](../Images/getting-started-cs-ui.png)

## <a name="upload-image-and-detect-faces"></a>Передача изображений и определение лиц

Ваше приложение будет распознавать лица, вызывая метод **FaceClient.Face.DetectWithStreamAsync**, который инкапсулирует REST API [обнаружения](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236) для передачи локального изображения.

Вставьте следующий метод в класс **MainWindow** ниже метода **FacePhoto_MouseMove**. Этот метод определяет список атрибутов лица для извлечения и считывает представленный файл изображения в параметр **Stream**. Затем он передает оба этих объекта в вызов метода **DetectWithStreamAsync**.

[!code-csharp[](~/Cognitive-Face-CSharp-sample/FaceTutorialCS/FaceTutorialCS/MainWindow.xaml.cs?name=snippet_uploaddetect)]

## <a name="draw-rectangles-around-faces"></a>Размещение прямоугольников вокруг лиц

Теперь нужно вставить код, чтобы добавить прямоугольник вокруг каждого распознанного лица на изображении. В классе **MainWindow** вставьте следующий код в конце метода **BrowseButton_Click** после строки `FacePhoto.Source = bitmapSource`. Этот код позволяет передать список распознанных лиц из вызова в метод **UploadAndDetectFaces**. Затем вокруг каждого лица рисуется прямоугольник, и измененное изображение отображается в главном окне.

[!code-csharp[](~/Cognitive-Face-CSharp-sample/FaceTutorialCS/FaceTutorialCS/MainWindow.xaml.cs?name=snippet_browsebuttonclick_mid)]

## <a name="describe-the-faces"></a>Описания лиц

Добавьте следующий метод в класс **MainWindow** ниже метода **UploadAndDetectFaces**. Этот метод преобразует полученные атрибуты лица в строку описания лица.

[!code-csharp[](~/Cognitive-Face-CSharp-sample/FaceTutorialCS/FaceTutorialCS/MainWindow.xaml.cs?name=snippet_facedesc)]

## <a name="display-the-face-description"></a>Отображение описания лица

Добавьте следующий код в метод **FacePhoto_MouseMove**. Этот обработчик событий отображает строку описания лица в `faceDescriptionStatusBar`, когда курсор наводится на прямоугольник распознанного лица.

[!code-csharp[](~/Cognitive-Face-CSharp-sample/FaceTutorialCS/FaceTutorialCS/MainWindow.xaml.cs?name=snippet_mousemove_mid)]

## <a name="run-the-app"></a>Запустите приложение

Запустите приложение и с помощью обзора найдите изображение, содержащее лицо. Подождите несколько секунд, чтобы служба распознавания лиц смогла ответить. На каждом лице на изображении отобразится красный прямоугольник. При наведении указателя мыши на прямоугольник вокруг лица в строке состояния появится описание лица.

![Снимок экрана, на котором обнаруженные лица выделены прямоугольниками](../Images/getting-started-cs-detected.png)


## <a name="next-steps"></a>Дальнейшие действия

С помощью этого руководства вы изучили базовый процесс использования пакета SDK NET для службы распознавания лиц и создали приложение, в котором лица на изображениях отображаются и выделяются рамкой. Ознакомьтесь с дополнительными сведениями об определении лиц.

> [!div class="nextstepaction"]
> [Как обнаруживать лица на изображении](../Face-API-How-to-Topics/HowtoDetectFacesinImage.md)