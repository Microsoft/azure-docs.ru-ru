---
title: Локальная отладка функций Azure в Сетке событий
description: Сведения о локальной отладке функций Azure, активируемых событием службы "Сетка событий"
author: craigshoemaker
ms.topic: conceptual
ms.date: 10/18/2018
ms.author: cshoe
ms.openlocfilehash: be05d237d2799404c3fd8b5733464e23eeb49aa3
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94833067"
---
# <a name="azure-function-event-grid-trigger-local-debugging"></a>Локальная отладка триггера Сетки событий для функций Azure

В этой статье показано, как отладить локальную функцию, которая обрабатывает событие в Сетке событий Azure, вызванное учетной записью хранения. 

## <a name="prerequisites"></a>Предварительные условия

- Создайте новое приложение-функцию или используйте существующее.
- Создайте новую учетную запись хранения или используйте существующую.
- Скачайте [ngrok](https://ngrok.com/), чтобы обеспечить вызов локальной функции в Azure.

## <a name="create-a-new-function"></a>Создание функции

Откройте приложение-функцию в Visual Studio, щелкните правой кнопкой мыши имя проекта в обозревателе решений, а затем последовательно выберите **Добавить > Новая функция Azure**.

В окне *Новая функция Azure* выберите **Триггер сетки событий** и нажмите кнопку **ОК**.

![Создание функции](./media/functions-debug-event-grid-trigger-local/functions-debug-event-grid-trigger-local-add-function.png)

После создания функции откройте файл кода и скопируйте URL-адрес в комментарии в верхней части файла. Это расположение используется при настройке триггера Сетки событий.

![Копирование расположения](./media/functions-debug-event-grid-trigger-local/functions-debug-event-grid-trigger-local-copy-location.png)

Затем задайте точку останова в строке, которая начинается с `log.LogInformation`.

![Задание точки останова](./media/functions-debug-event-grid-trigger-local/functions-debug-event-grid-trigger-local-set-breakpoint.png)


Далее **нажмите клавишу F5**, чтобы запустить сеанс отладки.

## <a name="allow-azure-to-call-your-local-function"></a>Вызов локальной функции с помощью Azure

Чтобы прервать процесс отладки функции на вашем компьютере, включите возможность взаимодействия Azure с вашей локальной функцией из облака.

Программа [ngrok](https://ngrok.com/) позволяет Azure вызывать функцию, выполняемую на вашем компьютере. Запустите *ngrok* с помощью приведенной ниже команды.

```bash
ngrok http -host-header=localhost 7071
```
Если программа настроена правильно, окно командной строки будет выглядеть примерно как на следующем снимке экрана.

![Снимок экрана, показывающий командную строку после запуска программы "ngrok".](./media/functions-debug-event-grid-trigger-local/functions-debug-event-grid-trigger-local-ngrok.png)

Скопируйте URL-адрес **HTTPS**, созданный при выполнении *ngrok*. Это значение используется при настройке конечной точки сетки событий.

## <a name="add-a-storage-event"></a>Добавление события хранилища

Откройте портал Azure и перейдите к учетной записи хранения, а затем выберите параметр **События**.

![Добавление события учетной записи хранения](./media/functions-debug-event-grid-trigger-local/functions-debug-event-grid-trigger-local-add-event.png)

В окне *События* нажмите кнопку **Подписка на события**. В окне *Подписка на события* щелкните раскрывающийся список *тип конечной точки* и выберите пункт **веб-перехватчик**.

![Выбор типа подписки](./media/functions-debug-event-grid-trigger-local/functions-debug-event-grid-trigger-local-event-subscription-type.png)

После настройки типа конечной точки щелкните **Выбрать конечную точку**, чтобы настроить значение конечной точки.

![Выбор типа конечной точки](./media/functions-debug-event-grid-trigger-local/functions-debug-event-grid-trigger-local-event-subscription-endpoint.png)

Значение *конечной точки подписчика* включает три разных значения. Префиксом служит URL-адрес HTTPS, созданный *ngrok*. В оставшейся части содержится URL-адрес из файла кода функции с именем функции в конце. Начиная с URL-адреса из файла кода функции, URL-адрес *ngrok* заменяет `http://localhost:7071`, а имя функции заменяет `{functionname}`.

В итоге URL-адрес должен выглядеть, как показано на приведенном ниже снимке экрана.

![Выбор конечной точки](./media/functions-debug-event-grid-trigger-local/functions-debug-event-grid-trigger-local-event-subscription-endpoint-selection.png)

Указав соответствующее значение, щелкните **Подтвердить выбор**.

> [!IMPORTANT]
> Каждый раз при запуске *ngrok* создается URL-адрес HTTPS и изменяется значение. Поэтому требуется создавать новую подписку на события каждый раз, когда функция взаимодействует с Azure посредством *ngrok*.

## <a name="upload-a-file"></a>Отправка файла

Теперь можно отправить файл в учетную запись хранения, чтобы активировать событие в Сетке событий. Это событие будет обработано локальной функцией. 

Откройте [Обозреватель службы хранилища](https://azure.microsoft.com/features/storage-explorer/) и подключитесь к учетной записи хранения. 

- Разверните раздел **Контейнеры больших двоичных объектов**. 
- Щелкните правой кнопкой мыши и выберите **Создать контейнер BLOB-объектов**.
- Присвойте контейнеру имя **test**.
- Выберите контейнер с именем *test*.
- Нажмите кнопку " **Отправить** ".
- Щелкните **отправить файлы** .
- Выберите файл и отправьте его в контейнер BLOB-объектов.

## <a name="debug-the-function"></a>Отладка функции

Как только служба Сетки событий определит, что файл отправлен в контейнер хранилища, будет достигнута точка останова в вашей локальной функции.

![Запуск ngrok](./media/functions-debug-event-grid-trigger-local/functions-debug-event-grid-trigger-local-breakpoint.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы очистить ресурсы, созданные при изучении этой статьи, удалите контейнер с именем **test** из своей учетной записи хранения.

## <a name="next-steps"></a>Дальнейшие действия

- [Автоматизация изменения размера передаваемых изображений с помощью сетки событий](../event-grid/resize-images-on-storage-blob-upload-event.md)
- [Триггер службы "Сетка событий" для Функций Azure](./functions-bindings-event-grid.md)
