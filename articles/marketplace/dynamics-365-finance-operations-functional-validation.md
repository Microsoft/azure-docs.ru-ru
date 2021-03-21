---
title: Функциональная проверка предложения AppSource Dynamics 365 Finance и операций в Azure Marketplace.
description: Как функционально проверить предложение Dynamics 365 Finance и Operations в Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: emuench
ms.author: navits
ms.date: 07/17/2020
ms.openlocfilehash: 9be90cdac742a581c6346f923f44e769c8a70f76
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102613635"
---
# <a name="appsource-dynamics-365-finance-and-operations-functional-validation"></a>AppSource Dynamics 365 Финансы и функциональная проверка операций

Чтобы завершить первую публикацию в [центре партнеров](https://partner.microsoft.com/dashboard/home), предложения Dynamics 365 Finance и Operations нуждаются в двух функциональных проверках:

- Загрузите демонстрационное видео о среде Dynamics 365, в которой отображаются основные функциональные возможности.
- Показывать снимки экрана, демонстрирующие среду [обслуживания](https://lcs.dynamics.com/) решений (LCS).

> [!NOTE]
> В последующих публикациях повторных сертификатов не требуется демонстрация. Дополнительные сведения см. в [документе о политике AppSource](/legal/marketplace/certification-policies#1440-dynamics-365-finance-ops).

## <a name="how-to-validate"></a>Проверка

Существует два варианта функциональной проверки.

- Продемонстрируйте 30-минутный Звонок на конференцию США по тихоокеанскому времени (PST), чтобы продемонстрировать и записать среду и решение [LCS](https://lcs.dynamics.com/) .
- В центре партнеров перейдите на страницу [Обзор коммерческого рынка](https://partner.microsoft.com/dashboard/commercial-marketplace/overview)  >   и загрузите демонстрационный URL-адрес видео и карты LCS на вкладке дополнительное содержимое предложения.

Группа сертификации Майкрософт просматривает видео и файлы, а затем либо утверждает решение, либо отправляет вам сообщения электронной почты о дальнейших действиях.

### <a name="option-1-30-minute-conference-call"></a>Параметр 1:30 — поминутный вызов конференции

Чтобы запланировать окончательный вызов проверки, свяжитесь [appsourceCRM@microsoft.com](mailto:appsourceCRM@microsoft.com) с названием вашего предложения и некоторыми потенциальными слотами времени в диапазоне от 8 до 5 по тихоокеанскому времени.

### <a name="option-2-upload-a-demo-video-and-lcs-screenshots"></a>Вариант 2. Отправка демонстрационных видеороликов и снимков LCS

1. Запишите видео и отправьте адрес на ваш сайт размещения по своему усмотрению. Соблюдайте следующие правила.

    - Для просмотра командой сертификации Майкрософт.
    - Менее 20 минут.
    - Включает в себя до трех основных функциональных возможностей решения в среде Dynamics 365.

    > [!NOTE]
    > Можно использовать имеющийся маркетинговый видеоролик, если он соответствует рекомендациям.

2. Примите следующие снимки экрана среды [LCS](https://lcs.dynamics.com/) , соответствующие предложению или решению, которые вы хотите опубликовать. Они должны быть достаточно четкими, чтобы группа сертификации читала текст. Сохраните снимки экрана как JPG файлы. Вы можете предоставить [appSourceCRM@microsoft.com](mailto:appSourceCRM@microsoft.com) разрешение для среды LCS, чтобы мы могли проверить программу установки, а не предоставить снимки экрана.

    1. Перейдите в   >  библиотеку проекта "средство **моделирования бизнес-процессов** LCS"  >  . Создание снимков экрана всех этапов процесса. Включите **схемы** и **проверенные** столбцы, как показано ниже:

       :::image type="content" source="media/dynamics-365-finance-operations/project-library.png" alt-text="Отображает окно библиотеки проекта.":::

      2. Выберите   >    >  **пакет решения** для управления решением LCS. Используйте снимки экрана, которые включают в себя обзор пакета и содержимое, показанное в следующих примерах:

    | Поле | Образ — |
    | --- | --- |
    | Основные сведения о пакете | [![Снимок экрана, на котором показано окно "Обзор пакета".](media/dynamics-365-finance-operations/package-overview-45.png)](media/dynamics-365-finance-operations/package-overview.png#lightbox) |
    | <ul><li>Утверждающие решения</li></ul> | [![Экран обзора пакета](media/dynamics-365-finance-operations/solution-approvers-45.png)](media/dynamics-365-finance-operations/solution-approvers.png#lightbox) |
    | Содержимое пакета<ul><li>Модель</li><li>Развертываемый пакет программного обеспечения</li></ul> | [![Первый экран содержимого пакета](media/dynamics-365-finance-operations/package-contents-1-45.png)](media/dynamics-365-finance-operations/package-contents-1.png#lightbox) |
    | <ul><li>Конфигурация GER</li><li>Резервная копия базы данных</li></ul><br>Артефакты не требуются в разделе **конфигурации GER** . | [![Два экрана содержимого пакета](media/dynamics-365-finance-operations/package-contents-2-45.png)](media/dynamics-365-finance-operations/package-contents-2.png#lightbox) |
    | <ul><li>Модель отчета Power BI</li><li>Артефакт BPM</li></ul><br>Артефакты не требуются в разделе **Power BI** . | [![Фильтр содержимого пакета, три](media/dynamics-365-finance-operations/package-contents-3-45.png)](media/dynamics-365-finance-operations/package-contents-3.png#lightbox) |
    | <ul><li>Обработка пакета данных</li><li>Лицензионное соглашение и политика конфиденциальности решений</li></ul><br>Разделы " **Конфигурация GER** " и " **Power BI модели отчета** " являются необязательными для включения в Финансы и предложения операций. | [![Страница "содержимое пакета" 4](media/dynamics-365-finance-operations/package-contents-4-45.png)](media/dynamics-365-finance-operations/package-contents-4.png#lightbox) |

    Чтобы узнать больше о каждом разделе портала LCS, ознакомьтесь с [руководством пользователя LCS](/dynamics365/fin-ops-core/dev-itpro/lifecycle-services/lcs-user-guide).

3. Отправьте в центр партнеров.

    1. Создайте текстовый документ, содержащий демонстрационный видеоадрес и снимки экрана, или сохраните снимки экрана в виде отдельных JPG-файлов.
    2. Добавьте текст и все файлы JPG в ZIP-файл на [коммерческом рынке](https://partner.microsoft.com/dashboard/commercial-marketplace/overview) в центре партнеров на вкладке " **дополнительное содержимое** " финансового решения и предложения по операциям.

    [![Отображает окно библиотеки проекта](media/dynamics-365-finance-operations/supplemental-content.png)](media/dynamics-365-finance-operations/supplemental-content.png#lightbox)

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о создании предложения см. в статье [Создание Dynamics 365 для операций с предложениями](./partner-center-portal/create-new-operations-offer.md).