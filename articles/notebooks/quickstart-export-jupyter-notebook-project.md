---
title: Использование Jupyter Notebook с предложениями Майкрософт
description: Сведения о том, как использовать Jupyter Notebook с предложениями Майкрософт.
ms.topic: quickstart
ms.date: 02/01/2021
ms.openlocfilehash: 5679c28d9cc8a4f1893ffad72002b66ad59861e6
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99507383"
---
# <a name="use-a-jupyter-notebook-with-microsoft-offerings"></a>Использование Jupyter Notebook с предложениями Майкрософт

[!INCLUDE [notebooks-status](../../includes/notebooks-status.md)]

Из этого краткого руководства вы узнаете, как импортировать Jupyter Notebook для использования в разных предложениях Майкрософт. 

Если у вас есть готовые записные книжки Jupyter Notebook или вы хотите создать новый проект, их можно использовать во многих предложениях Майкрософт. Ниже приведены некоторые варианты, описанные в следующих разделах. 
- [Visual Studio Code](#use-notebooks-in-visual-studio-code)
- [GitHub Codespaces](#use-notebooks-in-github-codespaces)
- [Машинное обучение Azure](#use-notebooks-with-azure-machine-learning)
- [Службы лабораторий Azure](#use-azure-lab-services)
- [GitHub](#use-github)

## <a name="create-an-environment-for-notebooks"></a>Создание среды для записных книжек

Если вы хотите создать среду, соответствующую устаревшей версии Записных книжек Azure (предварительная версия) можно использовать файл скрипта, доступный в GitHub.

1. Перейдите в [репозиторий GitHub](https://github.com/microsoft/AzureNotebooks) Записных книжек Azure или [непосредственно откройте папку среды](https://aka.ms/aznbrequirementstxt).
1. В командной строке перейдите в каталог, который вы хотите использовать для проектов.
1. Скачайте содержимое папки среды и следуйте инструкциям в файле README, чтобы установить зависимости пакета Записных книжек Azure.


## <a name="use-notebooks-in-visual-studio-code"></a>Использование Записных книжек в Visual Studio Code

[VS Code](https://code.visualstudio.com/) — это бесплатный редактор кода, который можно использовать локально или можно подключать к удаленным вычислительным ресурсам. В сочетании с расширением Python он предлагает полную среду для разработки на Python, включая полнофункциональный нативный интерфейс для работы с Jupyter Notebook. 

![Поддержка Jupyter Notebook в VS Code](media/vs-code-jupyter-notebook.png)

Если у вас есть готовые файлы проекта или вы хотите создать новую записную книжку, можно использовать VS Code. См. руководства по [работе с Jupyter Notebook в Visual Studio Code](https://code.visualstudio.com/docs/python/jupyter-support) и [обработке и анализу данных в Visual Studio Code](https://code.visualstudio.com/docs/python/data-science-tutorial).

Вы также можете использовать [скрипт среды Записных книжек Azure](#create-an-environment-for-notebooks) с Visual Studio Code для создания среды, соответствующей Azure Notebooks предварительной версии.

## <a name="use-notebooks-in-github-codespaces"></a>Использование Записных книжек в GitHub Codespaces

Служба GitHub Codespaces предоставляет облачные среды, в которых можно редактировать записные книжки с помощью Visual Studio Code или в веб-браузере. Она предлагает те же возможности для работы с Jupyter, что и VS Code, но без необходимости устанавливать какие-либо компоненты на устройстве. Если вы не хотите настраивать локальную среду и предпочитаете облачное решение, создать пространство кода будет отличным решением. Чтобы начать работу:
1. (Необязательно) Соберите все файлы проекта, которые вы решили применить для GitHub Codespaces.
1. [Создайте репозиторий GitHub](https://help.github.com/github/getting-started-with-github/create-a-repo) для хранения записных книжек.   
1. [Добавьте файлы](https://help.github.com/github/managing-files-in-a-repository/adding-a-file-to-a-repository) в репозиторий.
1. [Запросите доступ к предварительной версии GitHub Codespaces](https://github.com/features/codespaces).

## <a name="use-notebooks-with-azure-machine-learning"></a>Использование Записных книжек с Машинным обучением Azure

Машинное обучение Azure — это комплексная платформа машинного обучения, с помощью которой пользователи могут быстро создавать и развертывать модели в Azure. Машинное обучение Azure позволяет запускать записные книжки Jupyter Notebook на виртуальной машине или в общей вычислительной среде кластера. Если вам требуется облачное решение для рабочей нагрузки машинного обучения с возможностями отслеживания экспериментов, управления наборами данных и пр., мы рекомендуем использовать Машинное обучение Azure. Чтобы начать работу с Машинным обучением Azure:

1. (Необязательно) Соберите все файлы проекта, которые вы решили применить для Машинного обучения Azure.
1. [Создайте рабочую область](../machine-learning/how-to-manage-workspace.md) на портале Azure.

   ![создать рабочую область;](../machine-learning/media/how-to-manage-workspace/create-workspace.gif)
 
1. Откройте [Студию машинного обучения Azure (предварительная версия)](https://ml.azure.com/).
1. В области навигации слева выберите **Записные книжки**.
1. Нажмите кнопку **Отправить файлы** и отправьте файлы проекта.

Дополнительные сведения о Машинном обучении Azure и запуске записных книжек Jupyter Notebook см. в [этой документации](../machine-learning/how-to-run-jupyter-notebooks.md) или модуле [Введение в машинное обучение Azure](/learn/modules/intro-to-azure-machine-learning-service/) на сайте Microsoft Learn.


## <a name="use-azure-lab-services"></a>Использование Служб лабораторий Azure

[Службы лабораторий Azure](https://azure.microsoft.com/services/lab-services/) позволяют преподавателям легко настраивать и предоставлять доступ по требованию к предварительно настроенным виртуальным машинам для всей аудитории. Если вам нужно работать с записными книжками Jupyter Notebook и облачными средами вычислений в специализированной среде класса, Службы лаборатории являются отличным вариантом.

![Изображение](../lab-services/media/tutorial-setup-classroom-lab/new-lab-button.png)

Если у вас есть готовые файлы проекта или вы хотите создать новую записную книжку, можно использовать Службы лабораторий Azure. См. статью [Настройка лаборатории для обучения обработке и анализу данных с использованием Python и Jupyter Notebook](../lab-services/class-type-jupyter-notebook.md).

## <a name="use-github"></a>Использование GitHub

GitHub предоставляет бесплатное решение для хранения записных книжек (и других файлов) с возможностью управления версиями, позволяя совместно использовать записные книжки с другими пользователями. Если вам нужно совместно использовать проекты и работать с другими пользователями, GitHub — это отличный вариант, который можно сочетать с [GitHub Codespaces](#use-notebooks-in-github-codespaces) для разработки. Чтобы начать работу с GitHub:

1. (Необязательно) Соберите все файлы проекта, которые вы решили применить для GitHub.
1. [Создайте репозиторий GitHub](https://help.github.com/github/getting-started-with-github/create-a-repo) для хранения записных книжек. 
1. [Добавьте файлы](https://help.github.com/github/managing-files-in-a-repository/adding-a-file-to-a-repository) в репозиторий.

## <a name="next-steps"></a>Дальнейшие действия

- [Узнайте об использовании Python в Visual Studio Code](https://code.visualstudio.com/docs/python/python-tutorial)
- [Узнайте о Машинном обучении Azure и записных книжках Jupyter Notebook](../machine-learning/how-to-run-jupyter-notebooks.md)
- [Сведения о GitHub Codespaces](https://github.com/features/codespaces)
- [Узнайте о Службах лабораторий Azure](https://azure.microsoft.com/services/lab-services/)
- [Узнайте о GitHub](https://help.github.com/github/getting-started-with-github/)
