---
title: Квоты в предварительной версии Статических веб-приложений
description: Узнайте о квотах, связанных с предварительной версией Статических веб-приложений
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: overview
ms.date: 05/08/2020
ms.author: cshoe
ms.openlocfilehash: e3538e90a6dea69c703f56871fde86a18557a022
ms.sourcegitcommit: 99fc6ced979d780f773d73ec01bf651d18e89b93
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106095175"
---
# <a name="quotas-in-azure-static-web-apps-preview"></a>Квоты в предварительной версии Статических веб-приложений

Для предварительной версии Статических веб-приложений существуют следующие квоты.

> [!IMPORTANT]
> Предварительная версия Статических веб-приложений не предназначена для использования в рабочей среде.

| Компонент                     | План "Бесплатный"        |
|-----------------------------|------------------|
| Предоставляемая пропускная способность          | 100 ГБ в месяц |
| Избыточная пропускная способность           | Рекомендации недоступны      |
| Приложений на подписку Azure | 10               |
| Размер приложения                    | 250 МБ           |
| Подготовительные среды | 3                |
| Личные домены              | 1                |
| Авторизация (с пользовательскими ролями и правилами маршрутизации) | Максимум 25 конечных пользователей, которые могут относиться к настраиваемым ролям |
| Функции Azure             | Доступно        |
| Соглашение об уровне обслуживания                         | None             |

## <a name="github-storage"></a>Хранилище GitHub

GitHub Actions и GitHub Packages используют хранилище GitHub с собственным набором квот. Когда вы создаете сайт статических веб-приложений, GitHub хранит артефакты для сайта даже после развертывания.

Для получения дополнительных сведений см. следующие ресурсы:

- [Управление дисковым пространством GitHub Actions](https://github.community/t5/GitHub-Actions/Managing-Actions-storage-space/td-p/38944)
- [Сведения о выставлении счетов за GitHub Actions](https://help.github.com/github/setting-up-and-managing-billing-and-payments-on-github/about-billing-for-github-actions#about-billing-for-github-actions)
- [Управление предельной суммой расходов для GitHub Actions](https://help.github.com/github/setting-up-and-managing-billing-and-payments-on-github/managing-your-spending-limit-for-github-actions)

## <a name="next-steps"></a>Дальнейшие действия

- [Обзор](overview.md)
