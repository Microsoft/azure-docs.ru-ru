---
author: laurenhughes
ms.service: batch
ms.topic: include
ms.date: 11/09/2018
ms.author: lahugh
ms.openlocfilehash: bd51eabcff412de4928c682683b2a69b3e82e601
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "67185637"
---
### <a name="retrieve-output-files"></a>Извлечение выходных файлов

Портал Azure можно использовать для скачивания выходных MP3-файлов, созданных задачами ffmpeg. 

1. Выберите **Все службы** > **Учетные записи службы хранилища** и щелкните имя учетной записи службы хранилища.
2. Щелкните **Большие двоичные объекты** > *Вывод*.
3. Щелкните правой кнопкой мыши один из выходных MP3-файлов и нажмите кнопку **Загрузить**. Следуйте инструкциям в браузере, чтобы открыть или сохранить этот файл.

![Скачивание выходного файла](./media/batch-common-tutorial-download/download.png)

Хотя это не показано в этом примере, файлы можно также скачать программным способом из вычислительных узлов или контейнера хранилищ.
