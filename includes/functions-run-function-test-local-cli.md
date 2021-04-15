---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 10/18/2020
ms.author: glenga
ms.openlocfilehash: d400533039ea74a878cb8e543c22de02ee77e4f5
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/03/2021
ms.locfileid: "106283109"
---
## <a name="run-the-function-locally"></a>Локальное выполнение функции

1. Выполните функцию, запустив локальное хост-приложение среды выполнения Функций Azure из папки *LocalFunctionProj*:

    ```
    func start
    ```

    Ближе к концу выходных данных появятся следующие строки: 
    
    <pre>
    ...
    
    Now listening on: http://0.0.0.0:7071
    Application started. Press Ctrl+C to shut down.
    
    Http Functions:
    
            HttpExample: [GET,POST] http://localhost:7071/api/HttpExample
    ...
    
    </pre>
    
    >[!NOTE]  
    > Если результат HttpExample не похож на пример выше, скорее всего, вы запустили основное приложение из папки, отличной от корневой папки проекта. В этом случае остановите хост-приложение клавишами **CTRL**+**C**, перейдите в корневую папку проекта и снова выполните указанную выше команду.

1. Скопируйте URL-адрес функции `HttpExample` из этих выходных данных в браузер и добавьте строку запроса `?name=<YOUR_NAME>`, сформировав полный URL-адрес, например `http://localhost:7071/api/HttpExample?name=Functions`. Браузер отобразит ответное сообщение, содержащее значение строки запроса. Терминал, в котором вы запустили проект, также выводит данные журнала при выполнении запросов.

1. Когда все будет готово, нажмите клавиши **CTRL**+**C** и выберите `y`, чтобы отключить хост-приложение функции.
