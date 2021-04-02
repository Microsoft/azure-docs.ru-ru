---
title: Создание пользовательских действий и ресурсов
description: В этом учебнике описано, как создавать настраиваемые действия и ресурсы в Azure Resource Manager. Он также продемонстрирует, как настраиваемые рабочие процессы взаимодействуют с шаблонами Azure Resource Manager, Azure CLI, Политикой Azure и журналами действий Azure.
author: jjbfour
ms.topic: tutorial
ms.date: 06/19/2019
ms.author: jobreen
ms.openlocfilehash: 45f18727b53b802ba746da41b47fe955543ed9d0
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "75648551"
---
# <a name="create-custom-actions-and-resources-in-azure"></a>Создание пользовательских действий и ресурсов в Azure

Настраиваемым поставщиком называется контракт между Azure и конечной точкой. Настраиваемые поставщики позволяют настраивать рабочие процессы в Azure путем добавления новых API в Azure Resource Manager. Resource Manager с этими пользовательскими API может использовать новые возможности для развертывания и управления.

В этом учебнике описывается простой пример добавления новых действий и ресурсов в Azure и их интеграции.

## <a name="set-up-azure-functions-for-azure-custom-providers"></a>Настройка Функций Azure для настраиваемых поставщиков Azure

Первая часть этого учебника демонстрирует процесс настройки приложения-функции Azure для работы с настраиваемыми поставщиками.

- [Настройка Функций Azure для настраиваемых поставщиков Azure](./tutorial-custom-providers-function-setup.md)

Настраиваемые поставщики могут работать с любым общедоступным URL-адресом.

## <a name="author-a-restful-endpoint-for-custom-providers"></a>Создание конечной точки RESTful для настраиваемых поставщиков

Вторая часть учебника описывает, как создать конечную точку RESTful для пользовательских поставщиков.

- [Создание конечной точки RESTful для настраиваемых поставщиков](./tutorial-custom-providers-function-authoring.md)

## <a name="create-and-use-a-custom-provider"></a>Создание и использование настраиваемого поставщика

Третья часть учебника описывает создание настраиваемых поставщиков и использование их настраиваемых действий и ресурсов.

- [Создание и использование настраиваемого поставщика](./tutorial-custom-providers-create.md)

## <a name="next-steps"></a>Дальнейшие действия

Из этой статьи вы узнаете о настраиваемых поставщиках и их создании. Чтобы перейти к следующему руководству, см. статью [Настройка Функций Azure для настраиваемых поставщиков Azure](./tutorial-custom-providers-function-setup.md).

Дополнительные сведения приведены в следующих статьях:

- [Краткое руководство. Создание настраиваемого поставщика ресурсов и развертывание настраиваемых ресурсов](./create-custom-provider.md)
- [Руководство. Добавление настраиваемых действий в Azure REST API](./custom-providers-action-endpoint-how-to.md)
- [Руководство. Добавление настраиваемых ресурсов в Azure REST API](./custom-providers-resources-endpoint-how-to.md)
