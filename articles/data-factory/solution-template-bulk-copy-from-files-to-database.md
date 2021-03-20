---
title: Массовое копирование из файлов в базу данных
description: Узнайте, как использовать шаблон решения для массового копирования данных из Azure Data Lake Storage 2-го поколения в Azure Synapse Analytics или базу данных SQL Azure.
author: linda33wj
ms.author: jingwang
ms.service: data-factory
ms.topic: conceptual
ms.date: 12/09/2020
ms.openlocfilehash: 5f68c7b33a26bc8c01f3f413943351ff8d61b380
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100376127"
---
# <a name="bulk-copy-from-files-to-database"></a>Массовое копирование из файлов в базу данных

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

В этой статье описан шаблон решения, который можно использовать для массового копирования данных из Azure Data Lake Storage 2-го поколения в Azure Synapse Analytics или базу данных SQL Azure.

## <a name="about-this-solution-template"></a>Информация о шаблоне решения

Этот шаблон извлекает файлы из источника Azure Data Lake Storage 2-го поколения. Затем он выполняет итерацию по каждому файлу в источнике и копирует файл в целевое хранилище данных. 

Сейчас этот шаблон поддерживает копирование данных только в формате **DelimitedText**. Файлы в других форматах данных также можно получить из исходного хранилища данных, но их нельзя скопировать в целевое хранилище данных.  

Шаблон состоит из трех действий.
- Действие **Get Metadata** извлекает файлы из Azure Data Lake Storage 2-го поколения и передает их следующему действию *ForEach*.
- Действие **ForEach** получает файлы из действия *Get Metadata* и выполняет итерацию каждого файла для действия *Copy*.
- Действие **Copy** размещается в действии *ForEach* и копирует каждый файл из исходного хранилища данных в целевое хранилище данных.

Шаблон определяет два параметра:
- *SourceContainer* — это путь к корневому контейнеру, из которого данные копируются в Azure Data Lake Storage 2-го поколения. 
- *SourceDirectory* — это путь к каталогу в корневом контейнере, из которого данные копируются в Azure Data Lake Storage 2-го поколения.

## <a name="how-to-use-this-solution-template"></a>Использование шаблона решения

1. Перейдите к шаблону **Массовое копирование из файлов в базу данных**. Выберите **Создать**, чтобы создать подключение к исходному хранилищу 2-го поколения. Имейте в виду, что GetMetadataDataset и SourceDataset — это ссылки на одно и то же подключение к исходному хранилищу файлов.

    ![Создание подключения к исходному хранилищу данных](media/solution-template-bulk-copy-from-files-to-database/source-connection.png)

2. Выберите **Создать**, чтобы создать подключение к хранилищу данных — приемнику, в который вы копируете данные.

    ![Создание подключения к хранилищу данных — приемнику](media/solution-template-bulk-copy-from-files-to-database/destination-connection.png)
    
3. Щелкните **Использовать этот шаблон**.

    ![Использовать этот шаблон](media/solution-template-bulk-copy-from-files-to-database/use-template.png)
    
4. Вы увидите созданный конвейер, как показано в следующем примере:

    ![Проверка конвейера](media/solution-template-bulk-copy-from-files-to-database/new-pipeline.png)

    > [!NOTE]
    > Если вы выбрали **Azure синапсе Analytics** в качестве назначения данных на **шаге 2** , упомянутого выше, необходимо ввести подключение к хранилищу BLOB-объектов Azure для промежуточного хранения в соответствии с требованиями Azure синапсе Analytics polybase. Как показано на следующем снимке экрана, шаблон автоматически создаст *Путь к хранилищу* для хранилища BLOB-объектов. Проверьте, создан ли контейнер, после выполнения конвейера.
        
    ![Настройка Polybase](media/solution-template-bulk-copy-from-files-to-database/staging-account.png)

5. Выберите **Отладка**, введите **Параметры**, а затем нажмите **Готово**.

    ![Щелкните **Отладка**](media/solution-template-bulk-copy-from-files-to-database/debug-run.png)

6. После успешного выполнения конвейера отобразятся результаты, аналогичные приведенным в следующем примере:

    ![Просмотр результатов](media/solution-template-bulk-copy-from-files-to-database/run-succeeded.png)

       
## <a name="next-steps"></a>Дальнейшие действия

- [Общие сведения о службе фабрики данных Azure, службе интеграции данных в облаке](introduction.md)