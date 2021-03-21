---
title: Управление сборками U-SQL в конвейере CI/CD — Azure Data Lake
description: Ознакомьтесь с рекомендациями по управлению сборками U-SQL C# в конвейере CI/CD с помощью Azure DevOps.
author: liudan66
ms.author: liud
ms.service: data-lake-analytics
ms.topic: how-to
ms.date: 10/30/2018
ms.openlocfilehash: e88616f45c69d33234aa35333e0d82ad8cc59bb6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96015287"
---
# <a name="best-practices-for-managing-u-sql-assemblies-in-a-cicd-pipeline"></a>Рекомендации по управлению сборками U-SQL в конвейере CI/CD

В этой статье вы узнаете, как управлять исходным кодом сборки U-SQL с помощью нового проекта базы данных U-SQL. Здесь вы также узнаете, как настроить конвейер непрерывной интеграции и развертывания (CI/CD) для регистрации сборки с помощью Azure DevOps.

## <a name="use-the-u-sql-database-project-to-manage-assembly-source-code"></a>Использование проекта базы данных U-SQL для управления исходным кодом сборки

[Проект базы данных U-SQL](data-lake-analytics-data-lake-tools-develop-usql-database.md) — это тип проекта в Visual Studio, который помогает разработчикам быстро и легко разрабатывать, развертывать и администрировать свои базы данных U-SQL. Всеми объектами базы данных U-SQL (за исключением учетных данных) можно управлять с помощью проекта базы данных U-SQL.

Для управления исходным кодом сборки C# и сценариями U-SQL на языке DDL для регистрации сборки используйте следующие компоненты:

- проект базы данных U-SQL — для управления сценариями U-SQL регистрации сборки;
- библиотека классов (для приложения U-SQL) — для управления исходным кодом C# и зависимостями для операторов, функций и агрегаторов, определяемых пользователем (UDO/UDF/UDAG);
- проект базы данных U-SQL — для ссылки на проект библиотеки классов.

Проект базы данных U-SQL может ссылаться на проект библиотеки классов (для приложения U-SQL). Сборки, зарегистрированные в базе данных U-SQL, можно создать с помощью исходного кода C#, на который есть ссылка, из этого проекта библиотеки классов (для приложения U-SQL).

Чтобы создать проекты и добавить ссылки, выполните следующие действия.

1. Создайте проект библиотеки классов (для приложения U-SQL), выбрав **файл**  >  **создать**  >  **проект**. Проект находится в узле **Azure Data Lake > U-SQL**.

   ![Создание проекта библиотеки классов C# с помощью средств Azure Data Lake для Visual Studio](./media/data-lake-analytics-cicd-manage-assemblies/create-c-sharp-class-library-project.png)

1. Добавьте свой код C#, определяемый пользователем, в проект библиотеки классов (для приложения U-SQL).

1. Создайте проект U-SQL, выбрав **файл**  >  **создать**  >  **проект**. Проект находится в **Azure Data Lake**  >  узле **U-SQL** .

   ![Создание проекта базы данных U-SQL с помощью средств Azure Data Lake для Visual Studio](media/data-lake-analytics-cicd-manage-assemblies/create-u-sql-database-project.png)

1. Добавьте ссылку на проект библиотеки классов C# для проекта базы данных U-SQL.

   ![Средства Data Lake для Visual Studio — Добавление ссылки](./media/data-lake-analytics-cicd-manage-assemblies/data-lake-tools-add-project-reference.png)

   ![Средства Data Lake для Visual Studio. Добавление ссылки на проект базы данных U-SQL](./media/data-lake-analytics-cicd-manage-assemblies/data-lake-tools-add-project-reference-wizard.png)

1. Создайте сценарий сборки в проекте базы данных U-SQL. Для этого щелкните проект правой кнопкой мыши, а затем выберите пункт **Добавить новый элемент**.

   ![Добавление сценария сборки с помощью средств Azure Data Lake для Visual Studio](media/data-lake-analytics-cicd-manage-assemblies/add-assembly-script.png)

1. Откройте сценарий сборки в представлении конструктора сборки. В раскрывающемся меню **Создать сборку из ссылки** выберите сборку, на которую указывает ссылка.

   ![Создание сборки из ссылки с помощью Средств Data Lake для Visual Studio](./media/data-lake-analytics-cicd-manage-assemblies/data-lake-tools-create-assembly-from-reference.png)

1. При необходимости добавьте **Управляемые зависимости** и **Дополнительные файлы**. При добавлении дополнительных файлов средство будет использовать относительный путь, чтобы позже обеспечить возможность определения сборок на локальном компьютере и компьютере сборки.

**\@ _DeployTempDirectory** в окне редактора внизу представляет собой стандартную переменную, которая указывает инструменту на выходную папку построения. В выходной папке сборки каждая сборка имеет вложенную папку с именем сборки. Все библиотеки DLL, а также дополнительные файлы, находятся в этой вложенной папке.

## <a name="build-a-u-sql-database-project"></a>Создание проекта базы данных U-SQL

Выходные данные сборки для проекта базы данных U-SQL представляют собой пакет развертывания базы данных U-SQL. У него есть суффикс `.usqldbpack`. Пакет `.usqldbpack` представляет собой ZIP-файл, который содержит все инструкции DDL в едином сценарии U-SQL в папке DDL. Все созданные DLL-файлы и дополнительные файлы для сборок находятся в папке Temp.

## <a name="deploy-a-u-sql-database"></a>Развертывание базы данных U-SQL

Пакет `.usqldbpack` можно развернуть для локальной учетной записи или учетной записи Azure Data Lake Analytics. Используйте Visual Studio или пакет SDK для развертывания.

### <a name="deploy-a-u-sql-database-in-visual-studio"></a>Развертывание базы данных U-SQL в Visual Studio

Вы можете развернуть базу данных U-SQL, используя проект базы данных U-SQL или пакет `.usqldbpack` в среде Visual Studio.

#### <a name="deploy-by-using-a-u-sql-database-project"></a>Развертывание с помощью проекта базы данных U-SQL

1. Щелкните правой кнопкой мыши проект базы данных U-SQL и выберите **Развернуть**.

1. В **мастере развертывания баз данных U-SQL** выберите **учетную запись ADLA**, для которой требуется развернуть базу данных. Поддерживаются оба типа учетных записей — локальные и ADLA.

1. Поле **Источник базы данных** будет заполнено автоматически. Значение указывает на пакет .usqldbpack в выходной папке сборки проекта.

1. Введите имя в раздел **Имя базы данных**, чтобы создать базу данных. Если база данных с этим именем уже существует в целевой учетной записи Azure Data Lake Analytics, все объекты, указанные в проекте базы данных, создаются без ее повторного создания.

1. Щелкните **Отправить** для развертывания базы данных U-SQL. Загружаются все ресурсы (сборки и дополнительные файлы). Отправляется задание U-SQL, которое включает все инструкции DDL.

   ![Развертывание проекта базы данных U-SQL с помощью Средств Data Lake для Visual Studio](./media/data-lake-analytics-cicd-manage-assemblies/data-lake-tools-deploy-usql-database-project.png)

   ![Мастер развертывания проекта базы данных U-SQL с помощью Средств Data Lake для Visual Studio](./media/data-lake-analytics-cicd-manage-assemblies/data-lake-tools-deploy-usql-database-project-wizard.png)

### <a name="deploy-a-u-sql-database-in-azure-devops"></a>Развертывание базы данных U-SQL в Azure DevOps

`PackageDeploymentTool.exe` предоставляет программный интерфейс и интерфейс командной строки, которые позволяют развернуть базы данных U-SQL. Пакет SDK включен в [пакет U-SQL SDK для NuGet](https://www.nuget.org/packages/Microsoft.Azure.DataLake.USQL.SDK/), размещенный в файле `build/runtime/PackageDeploymentTool.exe`.

В Azure DevOps вы можете использовать задачу командной строки и этот пакет SDK, чтобы настроить конвейер автоматизации для обновления базы данных U-SQL. См. дополнительные сведения об [использовании пакета SDK и настройке конвейера CI/CD для развертывания базы данных U-SQL](data-lake-analytics-cicd-overview.md#deploy-u-sql-database-through-azure-pipelines).

## <a name="next-steps"></a>Дальнейшие действия

- [Настройка конвейера CI/CD для Azure Data Lake Analytics](data-lake-analytics-cicd-overview.md)
- [Тестирование кода Azure Data Lake Analytics](data-lake-analytics-cicd-test.md)
- [Запуск скриптов U-SQL на локальном компьютере](data-lake-analytics-data-lake-tools-local-run.md)
