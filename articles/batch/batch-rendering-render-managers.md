---
title: Поддержка диспетчера рендеринга
description: Использование интеграции диспетчера рендеринга в пакетной службе Azure. Здесь содержатся сведения о встроенной поддержке или поддержке через надстройки для многих популярных диспетчеров рендеринга.
author: mscurrell
ms.author: markscu
ms.date: 08/02/2018
ms.topic: how-to
ms.openlocfilehash: c44cbf86d8bf2fe83a6dc91dee1c4f58eec156c0
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "83726457"
---
# <a name="using-azure-batch-with-render-farm-managers"></a>Использование пакетной службы Azure с диспетчерами фермы рендеринга

Если вы используете существующую локальную ферму рендеринга, скорее всего для управления ее емкостью и заданиями рендеринга настроен диспетчер рендеринга.

Azure предоставляет встроенную поддержку или поддержку через надстройки для многих популярных диспетчеров рендеринга. Это позволяет добавлять и удалять виртуальные машины Azure, включая содержащие лицензии приложений с оплатой по мере использования и (или) низкоприоритетные виртуальные машины.

Поддерживаются следующие диспетчеры рендеринга:

* [PipelineFX Qube!](https://www.pipelinefx.com/);
* [Royal Render](https://www.royalrender.de/);
* [Thinkbox Deadline](https://deadline.thinkboxsoftware.com/).

## <a name="azure-render-hub"></a>Azure Render Hub

Azure Render Hub упрощает создание ферм рендеринга Azure и управление ими.  В Render Hub имеется собственная поддержка PipelineFx Qube и Deadline 10.  Дополнительные сведения и подробные инструкции см. в [репозитории GitHub](https://github.com/Azure/azure-render-hub).

## <a name="using-azure-with-pipelinefx-qube"></a>Использование Azure с PipelineFX Qube

Azure Render Hub поддерживает популярные диспетчеры рендеринга, включая Deadline.  Инструкции по развертыванию и использованию Render Hub см. в [репозитории GitHub](https://github.com/Azure/azure-render-hub).

Соответствующие скрипты и инструкции по использованию виртуальных машин пула пакетной службы Azure в качестве рабочих ролей Qube также доступны в [репозитории GitHub](https://github.com/Azure/azure-qube).

## <a name="using-azure-with-royal-render"></a>Использование Azure с Royal Render

Для Royal Render в Azure и пакетной службе Azure реализована встроенная поддержка интеграции, что позволяет расширить ферму рендеринга за счет виртуальных машин на платформе Azure. Краткое описание этого варианта вы найдете в [файлах справки](https://www.royalrender.de/help8/index.html?Cloudrendering.html).

Пример клиента Royal Render с интеграцией в Azure можно изучить в статье [об истории клиента Jellyfish Pictures](https://customers.microsoft.com/story/jellyfishpictures).

## <a name="using-azure-with-thinkbox-deadline"></a>Использование Azure с Thinkbox Deadline

Azure Render Hub поддерживает популярные диспетчеры рендеринга, включая Deadline.  Инструкции по развертыванию и использованию Render Hub см. в [репозитории GitHub](https://github.com/Azure/azure-render-hub).

## <a name="next-steps"></a>Дальнейшие действия

Попробуйте настроить интеграцию пакетной службы Azure с используемым диспетчером рендеринга, применяя соответствующий подключаемый модуль и инструкции на сайте GitHub, если потребуется.
