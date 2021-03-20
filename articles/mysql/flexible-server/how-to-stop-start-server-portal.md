---
title: Окончание или запуск — портал Azure — гибкий сервер базы данных Azure для MySQL
description: В этой статье описывается, как с помощью портал Azure останавливаются и запускаются операции в базе данных Azure для MySQL.
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: how-to
ms.date: 09/29/2020
ms.openlocfilehash: 15b08ea67afe0d307470b5a4fb0f7d26e0f4ea82
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93241928"
---
# <a name="stopstart-an-azure-database-for-mysql---flexible-server-preview"></a>Завершение работы и запуск базы данных Azure для MySQL — гибкого сервера (Предварительная версия)

> [!IMPORTANT]
> Сейчас предоставляется общедоступная предварительная версия Гибкого сервера Базы данных Azure для MySQL.

В этой статье приводятся пошаговые инструкции по выполнению и запуску гибкого сервера.

## <a name="prerequisites"></a>Предварительные условия

Вот что вам нужно, чтобы выполнить инструкции, приведенные в этом руководстве:

-   Необходимо иметь гибкий сервер базы данных Azure для MySQL.

## <a name="stop-a-running-server"></a>Завершение работы сервера

1.  В [портал Azure](https://portal.azure.com/)выберите гибкий сервер, который требуется отключить.

2.  На странице **Обзор** нажмите кнопку " **Закрыть** " на панели инструментов.
    
    :::image type="content" source="media/how-to-stop-start-server-portal/stop-server.png" alt-text="Останавливает гибкий сервер."::: 

3.  Нажмите кнопку **Да** , чтобы подтвердить остановку работы сервера.

    :::image type="content" source="media/how-to-stop-start-server-portal/confirm-stop.png" alt-text="Подтвердите остановку гибкого сервера."::: 

> [!NOTE]
> После остановки сервера другие операции управления недоступны для гибкого сервера.

## <a name="start-a-stopped-server"></a>Запуск остановленного сервера

1.  В [портал Azure](https://portal.azure.com/)выберите гибкий сервер, который вы хотите запустить.

2.  На странице **Обзор** нажмите кнопку **Запуск** на панели инструментов.

    :::image type="content" source="media/how-to-stop-start-server-portal/start-server.png" alt-text="Запуск гибкого сервера.":::  

> [!NOTE]
> После запуска сервера все операции управления теперь доступны для гибкого сервера.

## <a name="next-steps"></a>Дальнейшие действия
- Дополнительные сведения о работе [в сети в базе данных Azure для гибкого сервера MySQL](./concepts-networking.md)
- [Создание виртуальной сети для Гибкого сервера Базы данных Azure для MySQL и управление ею с помощью портала Azure](./how-to-manage-virtual-network-portal.md).

