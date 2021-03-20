---
title: Планирование проекта командного процесса обработки и анализа данных
description: Шаблоны Microsoft Project и Microsoft Excel для планирования проектов обработки и анализа данных и управления ими.
author: marktab
manager: marktab
editor: marktab
services: machine-learning
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: ac6055029b8fc7bbba11a8e3b789df3b6b1622e2
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93322749"
---
# <a name="team-data-science-process-project-planning"></a>Планирование проекта командного процесса обработки и анализа данных

Процесс обработки и анализа данных группы (TDSP) предоставляет жизненный цикл, позволяя структурировать разработку проектов по обработке и анализу данных. В этой статье приведены ссылки на шаблоны Microsoft Project и Excel, которые помогут вам при планировании этапов этого проекта и управлении ими.

Этот жизненный цикл представляет основные этапы, которые обычно выполняются проектами, часто итеративно:

- Коммерческий аспект.
- Получение и анализ данных
- Моделирование
- Развертывание
- Приемка клиентом

Описание каждого этапа см. в статье [Жизненный цикл процесса обработки и анализа данных группы](./lifecycle.md).

 
## <a name="microsoft-project-template"></a>Шаблон Microsoft Project

Шаблон Microsoft Project для командного процесса обработки и анализа данных доступен по этой ссылке: [шаблон Microsoft Project](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Team-Data-Science-Process/Project-Planning-and-Governance/Advanced%20Analytics%20Microsoft%20Project%20Plan.mpp). 

Когда вы откроете план, щелкните ссылку TDSP в крайнем левом углу. Измените имя и описание, а затем добавьте необходимые ресурсы группы. Оцените требуемые даты, основываясь на своем опыте.

![1](./media/team-data-science-process-project-templates/ms-project-templates.png)

В каждой задаче есть примечание. Откройте эти задачи, чтобы увидеть, какие ресурсы уже созданы автоматически.

![2](./media/team-data-science-process-project-templates/ms-project-template-task.png)


## <a name="excel-template"></a>Шаблон Excel

Если у вас нет доступа к Microsoft Project, вы также можете скачать лист Excel с такими же данными по этой ссылке: [шаблон Excel](https://github.com/Azure/Azure-MachineLearning-DataScience/blob/master/Team-Data-Science-Process/Project-Planning-and-Governance/Advanced%20Analytics%20Microsoft%20Project%20Plan.xlsx). Вы можете открыть его в любом средстве, с которым привыкли работать.

Вы используете эти шаблоны на свой риск. Применяются [стандартные заявления об отказе от ответственности](https://www.gnu.org/licenses/gpl-3.0.en.html).

## <a name="repository-template"></a>Шаблон из репозитория

Используйте этот [репозиторий шаблонов проектов](https://github.com/Azure/Azure-TDSP-ProjectTemplate), чтобы обеспечить эффективное выполнение проекта и совместную работу над ним. Этот репозиторий предоставляет стандартизированную структуру каталогов и шаблоны документов, которые можно использовать для собственного проекта TDSP.

## <a name="next-steps"></a>Дальнейшие действия

[Гибкая разработка проектов обработки и анализа данных](agile-development.md) В этом документе описывается проект обработки и анализа данных в систематической, контролируемой версиями и совместной работе с помощью процесса анализа и работы с данными группы.

Также доступны пошаговые руководства, которые демонстрируют все этапы процесса для **конкретных сценариев**. Эти этапы с иллюстрациями и краткими описаниями перечислены в статье [Пошаговые руководства по процессу обработки и анализа данных группы](walkthroughs.md). В них показано, как объединить облачные и локальные средства и службы в единый рабочий процесс или конвейер, чтобы создать интеллектуальное приложение.