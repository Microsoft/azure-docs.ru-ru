---
title: Просмотр журнала веб-заданий
description: Просмотр журнала журналов для заданий, завершившихся сбоем, и выполненных успешно.
author: ggailey777
ms.topic: conceptual
ms.date: 10/16/2018
ms.author: glenga
ROBOTS: NOINDEX,NOFOLLOW
ms.openlocfilehash: 4b4e421458438aecc7c08e397f1fc919a8cc2225
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101746691"
---
# <a name="view-webjob-history-in-the-azure-portal"></a>Просмотр журнала веб-заданий в портал Azure

Просмотр журнала журналов для заданий, завершившихся сбоем, и выполненных успешно.

1. Выберите веб-задание, журнал которого необходимо просмотреть, а затем нажмите кнопку **Журналы**.

    ![Кнопка "Журналы"](./media/web-sites-create-web-jobs/wjbladelogslink.png)

1. На странице **WebJob Details** (Подробности веб-задания) выберите время, чтобы просмотреть подробные сведения об одном выполнении.

    ![Подробности веб-задания](./media/web-sites-create-web-jobs/webjobdetails.png)

1. На странице **WebJob Details** (Подробности веб-задания) выберите **Переключить выходные данные**, чтобы увидеть текст содержимого журнала.

    ![Сведения о выполнении веб-задания](./media/web-sites-create-web-jobs/webjobrundetails.png)

    Чтобы просмотреть текст вывода в отдельном окне браузера, щелкните ссылку **скачать**. Чтобы скачать текст, щелкните ссылку **скачать** правой кнопкой мыши и выберите команду для сохранения содержимого файла в контекстном меню браузера.

1. Выберите ссылку навигации **Веб-задания** в верхней части страницы, чтобы перейти к списку веб-заданий.

    ![Навигация веб-заданий](./media/web-sites-create-web-jobs/breadcrumb.png)

    ![Список веб-заданий на панели мониторинга журнала](./media/web-sites-create-web-jobs/webjobslist.png)

## <a name="next-steps"></a>Дальнейшие действия

* Использование [пакета SDK](https://github.com/Azure/azure-webjobs-sdk/wiki) для веб-заданий для упрощения многих задач программирования

* Изучение [разработки и развертывания веб-заданий с помощью Visual Studio](webjobs-dotnet-deploy-vs.md)