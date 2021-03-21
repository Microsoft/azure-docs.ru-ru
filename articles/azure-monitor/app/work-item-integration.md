---
title: Интеграция рабочих элементов (Предварительная версия) — Application Insights
description: Узнайте, как создавать рабочие элементы в GitHub или Azure DevOps с помощью Application Insights данных, внедренных в них.
ms.topic: conceptual
ms.date: 02/9/2021
ms.openlocfilehash: ba0a67bad3ba47191414d6b406ab6cb4e6b7da78
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101731924"
---
# <a name="work-item-integration-preview"></a>Интеграция рабочих элементов (предварительная версия)

Функция интеграции рабочих элементов позволяет легко создавать рабочие элементы в GitHub или Azure DevOps, которые имеют соответствующие Application Insights внедренные в них данные.

> [!IMPORTANT]
> Интеграция рабочих элементов в настоящее время доступна в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены.
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="create-and-configure-a-work-item-template"></a>Создание и настройка шаблона рабочего элемента

1. Чтобы создать шаблон рабочего элемента, перейдите к ресурсу Application Insights и слева в разделе *Настройка* выберите **рабочие элементы** , а затем в верхней части выберите **создать новый шаблон** .

    :::image type="content" source="./media/work-item-integration/create-work-item-template.png" alt-text=" Снимок экрана вкладки &quot;рабочие элементы&quot; с выбранным параметром &quot;создать новый шаблон&quot;." lightbox="./media/work-item-integration/create-work-item-template.png":::

    Кроме того, можно создать шаблон рабочего элемента на вкладке сведения о сквозной транзакции, если ни одного шаблона в настоящее время не существует. Выберите событие и справа выберите **создать рабочий элемент**, а затем **Начните с шаблона книги**.

    :::image type="content" source="./media/work-item-integration/create-template-from-transaction-details.png" alt-text=" Снимок экрана вкладки &quot;сведения о сквозной транзакции&quot; с созданием рабочего элемента начинается с выбранного шаблона книги." lightbox="./media/work-item-integration/create-template-from-transaction-details.png":::

2. После выбора **создать новый шаблон** можно выбрать системы отслеживания, присвоить имя книге, связать выбранную систему отслеживания и выбрать регион для хранения шаблона (по умолчанию это регион, в котором находится ресурс Application Insights). Параметры URL-адреса — это URL-адрес по умолчанию для вашего репозитория, например `https://github.com/myusername/reponame` или `https://mydevops.visualstudio.com/myproject` .

    :::image type="content" source="./media/work-item-integration/create-workbook.png" alt-text=" Снимок экрана создания нового шаблона книги рабочих элементов.":::

## <a name="create-a-work-item"></a>Создать рабочий элемент

 Вы можете получить доступ к новому шаблону из всех подробных сведений о транзакции, к которым можно получить доступ с помощью вкладок производительность, сбои, доступность или другие вкладки.

1. Чтобы создать рабочий элемент, последовательно выберите события, а затем — **создать рабочий элемент** и выберите шаблон рабочего элемента.

    :::image type="content" source="./media/work-item-integration/create-work-item.png" alt-text=" Снимок экрана вкладки &quot;сведения о завершении транзакций&quot; с выбранным параметром &quot;создать рабочий элемент&quot;." lightbox="./media/work-item-integration/create-work-item.png":::

1. Откроется новая вкладка браузера в вашей системе отслеживания выбора. В Azure DevOps можно создать ошибку или задачу, а в GitHub можно создать новую ошибку в репозитории. Новый рабочий элемент автоматически создается с контекстными сведениями, предоставляемыми Application Insights.

    :::image type="content" source="./media/work-item-integration/github-work-item.png" alt-text=" Снимок экрана с автоматически созданной проблемой GitHub" lightbox="./media/work-item-integration/github-work-item.png":::

    :::image type="content" source="./media/work-item-integration/azure-devops-work-item.png" alt-text=" Снимок экрана: автоматическое создание ошибки в Azure DevOps." lightbox="./media/work-item-integration/azure-devops-work-item.png":::

## <a name="edit-a-template"></a>Изменение шаблона

Чтобы изменить шаблон, перейдите на вкладку **рабочие элементы** в разделе *Настройка* и щелкните значок карандаша рядом с книгой, которую требуется обновить.

:::image type="content" source="./media/work-item-integration/edit-template.png" alt-text=" Снимок экрана вкладки рабочего элемента с выбранным значком &quot;изменить карандаш&quot;.":::

Щелкните Изменить ![ значок изменить ](./media/work-item-integration/edit-icon.png) вверху, чтобы начать редактирование шаблона. Шаблоны рабочих элементов основаны на [Azure Monitor книгах](../visualize/workbooks-overview.md). Сведения о рабочем элементе создаются с помощью языка запросов ключевых слов. Вы можете изменить запросы, чтобы добавить дополнительный контекст, необходимый для вашей команды. Завершив редактирование, сохраните книгу, щелкнув значок сохранить значок ![ Сохранить ](./media/work-item-integration/save-icon.png) на верхней панели инструментов.

:::image type="content" source="./media/work-item-integration/edit-workbook.png" alt-text=" Снимок экрана книги шаблона рабочего элемента в режиме редактирования." lightbox="./media/work-item-integration/edit-workbook.png":::

Можно создать несколько конфигураций рабочих элементов и иметь пользовательскую книгу для удовлетворения каждого сценария. Книги также могут быть развернуты с Azure Resource Manager обеспечения стандартных реализаций в разных средах.