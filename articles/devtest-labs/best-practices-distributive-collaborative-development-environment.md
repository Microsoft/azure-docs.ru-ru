---
title: Распределенная совместная разработка Azure DevTest Labsных ресурсов
description: Содержит рекомендации по настройке распределенной и совместной среды разработки для разработки ресурсов DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: caf4bd13f2ec9c45db392a027db269b492cbd802
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102550082"
---
# <a name="best-practices-for-distributed-and-collaborative-development-of-azure-devtest-labs-resources"></a>Рекомендации по распределенным и совместным разработкам Azure DevTest Labsных ресурсов
Распределенная совместная разработка позволяет различным группам или людям разрабатывать и обслуживать базу кода. Для успешного выполнения процесс разработки зависит от возможности создания, совместного использования и интеграции информации. Этот принцип разработки ключа можно использовать в Azure DevTest Labs. В лаборатории существует несколько типов ресурсов, которые обычно распределяются между различными лабораториями предприятия. Различные типы ресурсов сосредоточены на двух областях:

- Ресурсы, которые внутренне хранятся в лаборатории (на основе лаборатории);
- Ресурсы, которые хранятся во [внешних репозиториях, подключенных к лаборатории](devtest-lab-add-artifact-repo.md) (на основе репозитория кода). 

В этом документе описаны некоторые рекомендации, обеспечивающие совместную работу и распространение в нескольких группах и обеспечивающие настройку и качество на всех уровнях.

## <a name="lab-based-resources"></a>Ресурсы на основе лаборатории

### <a name="custom-virtual-machine-images"></a>Пользовательские образы виртуальных машин
Вы можете использовать общий источник пользовательских образов, развернутых в лабораториях на ночь. Подробные сведения см. в разделе [фабрика изображений](image-factory-create.md).    

### <a name="formulas"></a>Формулы
[Формулы](devtest-lab-manage-formulas.md) являются специфичными для лаборатории и не имеют механизма распространения. Лабораторные элементы выполняют всю разработку формул. 

## <a name="code-repository-based-resources"></a>Ресурсы на основе репозитория кода
Существуют две различные функции, основанные на репозиториях кода, артефактах и средах. В этой статье рассматриваются функции и способы наиболее эффективного настройки репозиториев и рабочих процессов, позволяющие настраивать доступные артефакты и среды на уровне Организации или группы.  Этот рабочий процесс основан на стандартной [стратегии ветвления управления исходным кодом](/azure/devops/repos/tfvc/branching-strategies-with-tfvc). 

### <a name="key-concepts"></a>Основные понятия
Сведения об источнике артефактов включают метаданные, скрипты. Сведения об источнике для сред включают в себя метаданные и шаблоны диспетчер ресурсов с любыми вспомогательными файлами, такими как сценарии PowerShell, сценарии DSC, ZIP-файлы и т. д.  

### <a name="repository-structure"></a>Структура репозитория  
Наиболее распространенной конфигурацией для управления исходным кодом (SCC) является настройка многоуровневой структуры для хранения файлов кода (шаблонов диспетчер ресурсов, метаданных и скриптов), используемых в лабораториях. В частности, создайте различные репозитории для хранения ресурсов, управляемых с помощью различных уровней бизнеса:   

- Ресурсы всей компании.
- Подразделение и ресурсы на уровне подразделения
- Ресурсы для конкретной группы.

Каждый из этих уровней связан с другим репозиторием, в котором основная ветвь должна иметь качество производства. [Ветви](/azure/devops/repos/git/git-branching-guidance) в каждом репозитории будут использоваться для разработки этих конкретных ресурсов (артефактов или шаблонов). Эта структура хорошо соответствует DevTest Labs, так как вы можете легко подключать несколько репозиториев и несколько ветвей одновременно к лабораториям Организации. Имя репозитория включается в пользовательский интерфейс, чтобы избежать путаницы при наличии идентичных имен, описаний и издателя.
     
На следующей схеме показаны два репозитория: репозиторий компании, обслуживаемый ИТ-подразделением, и репозиторий подразделений, поддерживаемый подразделением R&D.

![Пример дистрибутиве и среды совместной разработки](./media/best-practices-distributive-collaborative-dev-env/distributive-collaborative-dev-env.png)
   
Эта многоуровневая структура позволяет выполнять разработку, сохраняя более высокий уровень качества в основной ветви, при этом при наличии нескольких репозиториев, подключенных к лаборатории, обеспечивается большая гибкость.

## <a name="next-steps"></a>Дальнейшие действия    
См. следующие статьи:

- Добавление репозитория в лабораторию с помощью [портал Azure](devtest-lab-add-artifact-repo.md) или с помощью [шаблона управления ресурсами Azure](add-artifact-repository.md)
- [Артефакты DevTest Labs](devtest-lab-artifact-author.md)
- [Среды DevTest Labs](devtest-lab-create-environment-from-arm.md).
