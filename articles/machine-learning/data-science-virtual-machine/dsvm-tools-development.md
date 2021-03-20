---
title: Инструменты разработки
titleSuffix: Azure Data Science Virtual Machine
description: Сведения о средствах и интегрированных средах разработки, доступных на виртуальной машине для обработки и анализа данных.
keywords: средства анализа и обработки данных, виртуальная машина для анализа и обработки данных, средства для анализа и обработки данных, анализ и обработка данных Linux
services: machine-learning
ms.service: data-science-vm
author: lobrien
ms.author: laobri
ms.topic: conceptual
ms.date: 12/12/2019
ms.openlocfilehash: cecc195b8b97ffd9b25cf12898726352ddd698a9
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100519445"
---
# <a name="development-tools-on-the-azure-data-science-virtual-machine"></a>Средства разработки на виртуальной машине Azure для обработки и анализа данных

Виртуальная машина для обработки и анализа данных (DSVM) объединяет несколько популярных средств в высокопроизводительную интегрированную среду разработки (IDE). Ниже приведены некоторые средства, предоставляемые на виртуальной машине для обработки и анализа данных.

## <a name="visual-studio-community-edition"></a>выпуск Visual Studio Community;

| Категория | Значение |
| ------------- | ------------- |
| Что это такое?   | Среда IDE общего назначения      |
| Поддерживаемые версии DSVM      | Windows: Visual Studio 2017, Windows 2019: Visual Studio 2019      |
| Распространенные способы применения      | Разработка программного обеспечения    |
| Как она настраивается и устанавливается на DSVM?      | Рабочая нагрузка обработки и анализа данных (средства Python и R), рабочая нагрузка Azure (Hadoop, Data Lake), средства Node.js и SQL Server, [Машинное обучение Azure для Visual Studio Code](https://github.com/Microsoft/vs-tools-for-ai)    |
| Использование и запуск      | Ярлык рабочего стола ( `C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\Common7\IDE\devenv.exe` ). Графически откройте Visual Studio с помощью значка рабочего стола или меню " **Пуск** ". Откройте окно поиска программ (кнопки Windows + S) и введите **Visual Studio**. Здесь можно создавать проекты на разных языках, например C#, Python, R и Node.js.   |
| Связанные средства на DSVM      |     Visual Studio Code, RStudio, Juno  |

> [!NOTE]
> Может появиться сообщение о том, что период ознакомления завершился. Введите учетные данные для учетной записи Майкрософт. Вы можете создать новую бесплатную учетную запись, чтобы получить доступ к Visual Studio Community.

## <a name="visual-studio-code"></a>Visual Studio Code 

| Категория | Значение |
| ------------- | ------------- |
| Что это такое?   | Среда IDE общего назначения      |
| Поддерживаемые версии DSVM      | Windows, Linux     |
| Распространенные способы применения      | Редактор кода и интеграция Git   |
| Использование и запуск      | Ярлык рабочего стола ( `C:\Program Files (x86)\Microsoft VS Code\Code.exe` ) в Windows, ярлык рабочего стола или терминал ( `code` ) в Linux    |
| Связанные средства на DSVM      |     Visual Studio, RStudio, Juno  |

## <a name="rstudio-desktop"></a>RStudio Desktop;

| Категория | Значение |
| ------------- | ------------- |
| Что это такое?   | Интегрированная среда разработки клиента для языка R   |
| Поддерживаемые версии DSVM      | Windows, Linux      |
| Распространенные способы применения      |  Разработка на языке R     |
| Использование и запуск      | Ярлык рабочего стола ( `C:\Program Files\RStudio\bin\rstudio.exe` ) в Windows, ярлык на рабочем столе ( `/usr/bin/rstudio` ) в Linux      |
| Связанные средства на DSVM      |   Visual Studio, Visual Studio Code, Juno      |

## <a name="rstudio-server"></a>RStudio Server

| Категория | Значение |
| ------------- | ------------- |
| Что это такое?   | Интегрированная среда разработки клиента для языка R   |
| Что это такое?   | Интегрированная веб-среда разработки для R    |
| Поддерживаемые версии DSVM      | Linux      |
| Распространенные способы применения      |  Разработка на языке R     |
| Использование и запуск      | Включите службу с помощью _systemctl Enable Rstudio-Server_, а затем запустите службу с помощью _systemctl Start Rstudio-Server_. Затем войдите на сервер RStudio по адресу http: \/ /йоур-ВМ-ИП: 8787.       |
| Связанные средства на DSVM      |   Visual Studio, Visual Studio Code, RStudio Desktop      |

## <a name="juno"></a>Juno 

| Категория | Значение |
| ------------- | ------------- |
| Что это такое?   | Интегрированная среда разработки клиента для языка Julia   |
| Поддерживаемые версии DSVM      | Windows, Linux      |
| Распространенные способы применения      |  Разработка на языке Julia     |
| Использование и запуск      | Ярлык рабочего стола ( `C:\JuliaPro-0.5.1.1\Juno.bat` ) в Windows, ярлык на рабочем столе ( `/opt/JuliaPro-VERSION/Juno` ) в Linux      |
| Связанные средства на DSVM      |   Visual Studio, Visual Studio Code, RStudio      |

## <a name="pycharm"></a>PyCharm

| Категория | Значение |
| ------------- | ------------- |
| Что это такое?   | Интегрированная среда разработки клиента для языка Python    |
| Поддерживаемые версии DSVM      | Windows 2019, Linux      |
| Распространенные способы применения      |  Разработка на Python     |
| Использование и запуск      | Ярлык рабочего стола ( `C:\Program Files\tk` ) в Windows. Ярлык на рабочем столе ( `/usr/bin/pycharm` ) в Linux      |
| Связанные средства на DSVM      |   Visual Studio, Visual Studio Code, RStudio      |
