---
title: Создание среды, ориентированной на приложения, в Azure
description: В этой статье показано, как создать среду, ориентированную на приложения, с Cloud Shell колонии и Microsoft Azure.
ms.topic: how-to
ms.date: 11/26/2020
ms.openlocfilehash: 88244f268d5ed038e9bb7082d3d5cc1179e5ec4e
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94918262"
---
# <a name="create-an-application-centric-environment"></a>Создание среды, ориентированной на приложения

[Куали CloudShell колонии](https://azuremarketplace.microsoft.com/marketplace/apps/quali_systems.cloudshell_colony?tab=Overview) — это платформа SaaS для реализации автоматизации инфраструктуры в масштабе для сложных сред, ориентированных на приложения, в отношении облачных технологий, включая Azure и Kubernetes. CloudShell колонии дополняет Azure DevTest Labs, чтобы помочь группам разработчиков развертывать сложные приложения во всем потоке значений, а не в рабочей среде.

В этой статье показано, как создать среду, ориентированную на приложения, с помощью CloudShell колонии и Microsoft Azure.

## <a name="set-up-the-environment-with-cloudshell-colony-and-microsoft-azure"></a>Настройка среды с помощью CloudShell колонии и Microsoft Azure

1. Подпишитесь на бесплатную пробную версию [колонии](https://azuremarketplace.microsoft.com/marketplace/apps/quali_systems.cloudshell_colony?tab=Overview).

    :::image type="content" source="./media/create-application-centric-environment/free-trial.png" alt-text="Зарегистрируйтесь для получения бесплатной пробной версии":::    
1. Свяжите учетную запись Azure ([Просмотрите шаги здесь](https://colonysupport.quali.com/hc/articles/360008222234)).

    :::image type="content" source="./media/create-application-centric-environment/welcome.png" alt-text="Добро пожаловать в колонии":::     
1. Пригласите пользователей в свой модуль.
1. Создайте свою первую схему с помощью файла YAML ([Просмотрите шаги здесь](https://colonysupport.quali.com/hc/articles/360001680807-Steps-to-Developing-a-Blueprint)).
    1. Свяжите репозиторий схем на GitHub или BitBucket с колонии.
    1. Используйте примерный проект колонии в качестве основы и измените соответствующим образом.

        :::image type="content" source="./media/create-application-centric-environment/performance-stress-tests.png" alt-text="Нагрузочные тесты":::    
    1. Опубликуйте свой проект для использования другими пользователями.
1. Запустите среду приложения в песочнице с помощью колонии.

    :::image type="content" source="./media/create-application-centric-environment/blueprints.png" alt-text="Запуск среды приложения в песочнице с помощью колонии":::    

> [!NOTE]
> Вы также можете интегрировать свой проект как часть рабочего процесса CI/CD в Azure DevOps ([Просмотр шагов](https://colonysupport.quali.com/hc/articles/360008464234)).

:::image type="content" source="./media/create-application-centric-environment/devops-pipeline.png" alt-text="Подключение к конвейеру Azure DevOps":::    

## <a name="next-steps"></a>Дальнейшие действия

[Запросить демонстрацию колонии](https://info.quali.com/cloudshell-colony-demo-request)
