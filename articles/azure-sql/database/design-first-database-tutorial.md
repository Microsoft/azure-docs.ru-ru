---
title: Руководство по Разработка первой реляционной базы данных с использованием SSMS
description: Узнайте, как создать свою первую реляционную базу данных в службе "База данных SQL Azure" с помощью SQL Server Management Studio.
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.topic: tutorial
author: stevestein
ms.author: sstein
ms.reviewer: v-masebo
ms.date: 07/29/2019
ms.custom: sqldbrb=1
ms.openlocfilehash: ae7baeac6cee2a692928642e3e38ce0adad17d1c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92674888"
---
# <a name="tutorial-design-a-relational-database-in-azure-sql-database-using-ssms"></a>Руководство по созданию реляционной базы данных в службе "База данных SQL Azure" с помощью SSMS
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]


База данных SQL Azure — это реляционная база данных как услуга (DBaaS) в Microsoft Cloud (Azure). В рамках этого руководства вы узнаете, как с помощью портала Azure и [SQL Server Management Studio](/sql/ssms/sql-server-management-studio-ssms) выполнять такие действия:

> [!div class="checklist"]
>
> - создать базу данных с помощью портала Azure*;
> - настроить правила брандмауэра на уровне сервера с помощью портала Azure;
> - Подключение к базе данных с помощью SQL Server Management Studio.
> - создать таблицы с помощью SSMS;
> - выполнить массовую загрузку данных с помощью BCP;
> - запросить данные с помощью SSMS.

\* Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

> [!TIP]
> Следующий модуль Microsoft Learn поможет вам бесплатно научиться [разрабатывать и настраивать приложение ASP.NET, которое запрашивает базу данных SQL Azure](/learn/modules/develop-app-that-queries-azure-sql/), а также создавать простую базу данных.
> [!NOTE]
> В рамках этого учебника используется База данных SQL Azure. Но вы также можете использовать базу данных в составе эластичного пула или в Управляемом экземпляре SQL. Чтобы подключиться к Управляемому экземпляру SQL, ознакомьтесь с соответствующими руководствами: [Краткое руководство. Настройка виртуальной машины Azure для подключения к Управляемому экземпляру SQL Azure](../managed-instance/connect-vm-instance-configure.md) и [Краткое руководство. Настройка подключения типа "точка — сеть" к Управляемому экземпляру SQL Azure из локальной сети](../managed-instance/point-to-site-p2s-configure.md).

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством вам потребуются:

- [SQL Server Management Studio](/sql/ssms/sql-server-management-studio-ssms) (последняя версия);
- [BCP и SQLCMD](https://www.microsoft.com/download/details.aspx?id=36433) (последняя версия).

## <a name="sign-in-to-the-azure-portal"></a>Вход на портал Azure

Войдите на [портал Azure](https://portal.azure.com/).

## <a name="create-a-blank-database-in-azure-sql-database"></a>Создание пустой базы данных в службе "База данных SQL Azure"

База данных в службе "База данных SQL Azure" создается с определенным набором вычислительных ресурсов и ресурсов хранения. База данных создается в [группе ресурсов Azure](../../active-directory-b2c/overview.md) и управляется с помощью [логического сервера SQL Server](logical-servers.md).

Чтобы создать пустую базу данных, выполните приведенные ниже действия.

1. На **домашней странице** или в меню портала Azure выберите **Создать ресурс**.
2. На странице **создания** в разделе Azure Marketplace выберите **Базы данных**, а затем в разделе **Избранные** выберите **База данных SQL**.

   ![Создание пустой базы данных](./media/design-first-database-tutorial/create-empty-database.png)

3. Заполните форму **База данных SQL**, указав следующую информацию, как показано на предыдущем рисунке.

    | Параметр       | Рекомендуемое значение | Описание |
    | ------------ | ------------------ | ------------------------------------------------- |
    | **Имя базы данных** | *yourDatabase* | Допустимые имена баз данных см. в статье [Идентификаторы баз данных](/sql/relational-databases/databases/database-identifiers). |
    | **Подписка** | *yourSubscription*  | Дополнительные сведения о подписках см. [здесь](https://account.windowsazure.com/Subscriptions). |
    | **Группа ресурсов** | *yourResourceGroup* | Допустимые имена групп ресурсов см. в статье о [правилах и ограничениях именования](/azure/architecture/best-practices/resource-naming). |
    | **Выбрать источник** | Пустая база данных | Указывает, что должна быть создана пустая база данных. |

4. Щелкните **Сервер**, чтобы использовать имеющийся сервер или создать и настроить новый. Выберите существующий сервер или нажмите кнопку **Создать сервер** и заполните форму **Новый сервер**, указав следующие сведения.

    | Параметр       | Рекомендуемое значение | Описание |
    | ------------ | ------------------ | ------------------------------------------------- |
    | **Имя сервера** | Любое глобально уникальное имя | Допустимые имена серверов см. в статье о [правилах и ограничениях именования](/azure/architecture/best-practices/resource-naming). |
    | **Имя для входа администратора сервера** | Любое допустимое имя | Сведения о допустимых именах для входа см. в статье [Идентификаторы базы данных](/sql/relational-databases/databases/database-identifiers). |
    | **Пароль** | Любой допустимый пароль | Длина пароля должна составлять минимум восемь символов. В пароле должны использоваться символы трех категорий из перечисленных: прописные буквы, строчные буквы, цифры и специальные символы. |
    | **Расположение** | Любое допустимое расположение | Дополнительные сведения о регионах Azure см. [здесь](https://azure.microsoft.com/regions/). |

    ![Создание базы данных — сервер](./media/design-first-database-tutorial/create-database-server.png)

5. Нажмите кнопку **Выбрать**.
6. Щелкните **Ценовая категория**, чтобы указать уровень служб, число DTU или виртуальных ядер и объем хранилища. Вы можете изучить доступные ресурсы для каждого уровня служб (число DTU или виртуальных ядер и объем хранилища).

    Выбрав уровень служб, число DTU или виртуальных ядер и объем хранилища, нажмите кнопку **Применить**.

7. Заполните поле **Параметры сортировки** для пустой базы данных. В этом руководстве используйте значение по умолчанию. Дополнительные сведения о параметрах сортировки см. в [этой статье](/sql/t-sql/statements/collations).

8. Заполнив форму **База данных SQL**, нажмите кнопку **Создать**, чтобы подготовить базу данных. Этот шаг может занять несколько минут.

9. На панели инструментов щелкните **Уведомления**, чтобы отслеживать процесс развертывания.

   ![Снимок экрана: меню "Уведомления" с сообщением о выполнении развертывания](./media/design-first-database-tutorial/notification.png)

## <a name="create-a-server-level-ip-firewall-rule"></a>Создание правила брандмауэра для IP-адресов на уровне сервера

Служба "База данных SQL Azure" создает брандмауэр IP-адресов на уровне сервера. Он не позволяет внешним приложениям и средствам подключаться к серверу и к любой базе данных на сервере, если не создано правило брандмауэра, позволяющее пропускать их IP-адреса через брандмауэр. Чтобы разрешить внешние подключения к базе данных, необходимо сначала добавить правило брандмауэра IP-адресов, указав в нем свой IP-адрес (или диапазон IP-адресов). Выполните следующие действия, чтобы создать [правило брандмауэра IP-адресов на уровне сервера](firewall-configure.md).

> [!IMPORTANT]
> База данных SQL обменивается данными через порт 1433. Если вы пытаетесь подключиться к этой службе из корпоративной сети, исходящий трафик через порт 1433 может быть запрещен сетевым брандмауэром. В таком случае вы не сможете подключиться к базе данных, пока ваш администратор не откроет порт 1433.

1. После завершения развертывания выберите **Базы данных SQL** в меню портала Azure или выполните поиск по запросу *Базы данных SQL* на любой странице и выберите этот пункт.  

1. Выберите *yourDatabase* на странице **Базы данных SQL**. После этого откроется страница обзора базы данных, где будет указано полное **имя сервера** (например, `contosodatabaseserver01.database.windows.net`) и будут предоставлены параметры для дальнейшей настройки.

   ![имя сервера](./media/design-first-database-tutorial/server-name.png)

1. Скопируйте полное имя сервера. Оно понадобится вам для подключения к серверу и связанным базам данных из SQL Server Management Studio.

1. Щелкните **Настройка брандмауэра для сервера** на панели инструментов. Откроется страница **Параметры брандмауэра** сервера.

   ![Правило брандмауэра для IP-адресов на уровне сервера](./media/design-first-database-tutorial/server-firewall-rule.png)

1. На панели инструментов щелкните **Добавить IP-адрес клиента**, чтобы добавить текущий IP-адрес в новое правило брандмауэра IP-адресов. С использованием правила брандмауэра IP-адресов можно открыть порт 1433 для одного IP-адреса или диапазона IP-адресов.

1. Выберите команду **Сохранить**. Для текущего IP-адреса будет создано правило брандмауэра для IP-адресов на уровне сервера, с помощью которого можно открыть порт 1433 сервера.

1. Нажмите кнопку **ОК**, а затем закройте страницу **Параметры брандмауэра**.

Теперь IP-адрес может проходить через брандмауэр IP-адресов. Теперь можно подключиться к базе данных с помощью SQL Server Management Studio или другого средства по своему усмотрению. Обязательно используйте созданную ранее учетную запись администратора сервера.

> [!IMPORTANT]
> По умолчанию доступ через брандмауэр IP-адресов Базы данных SQL включен для всех служб Azure. На этой странице щелкните **Откл.** , чтобы отключить доступ для всех служб Azure.

## <a name="connect-to-the-database"></a>Подключение к базе данных

Используйте [SQL Server Management Studio](/sql/ssms/sql-server-management-studio-ssms) для подключения к вашей базе данных.

1. Откройте среду SQL Server Management Studio.
2. В диалоговом окне **Соединение с сервером** введите следующие данные:

   | Параметр       | Рекомендуемое значение | Описание |
   | ------------ | ------------------ | ------------------------------------------------- |
   | **Тип сервера** | Ядро СУБД | Это значение обязательно. |
   | **Имя сервера** | Полное имя сервера | Например, *yourserver.database.windows.net*. |
   | **Аутентификация** | Проверка подлинности SQL Server | В рамках работы с этим руководством мы настроили только один тип проверки подлинности — проверку подлинности SQL. |
   | **Имя входа** | Учетная запись администратора сервера | Это учетная запись, указанная при создании сервера. |
   | **Пароль** | Пароль для учетной записи администратора сервера | Пароль, указанный при создании сервера. |

   ![подключение к серверу](./media/design-first-database-tutorial/connect.png)

3. Щелкните **Параметры** в диалоговом окне **Подключение к серверу**. В разделе **Подключение к базе данных** введите *yourDatabase*, чтобы подключиться к этой базе данных.

    ![Подключение к базе данных на сервере](./media/design-first-database-tutorial/options-connect-to-db.png)  

4. Нажмите кнопку **Соединить**. В SSMS откроется окно **Обозреватель объектов**.

5. В **обозревателе объектов** разверните **Базы данных**, а затем выберите *yourDatabase*, чтобы просмотреть объекты в образце базы данных.

   ![объекты базы данных](./media/design-first-database-tutorial/connected.png)  

## <a name="create-tables-in-your-database"></a>Создание таблиц в базе данных

Создайте схему базы данных с четырьмя таблицами, моделирующими систему управления студентами для университетов, с помощью [Transact-SQL](/sql/t-sql/language-reference):

- Модель Person
- Курс
- Студент
- Материалы

На приведенной ниже схеме показано, как эти таблицы связаны друг с другом. Некоторые из этих таблиц ссылаются на столбцы в других таблицах. Например, таблица *Student* ссылается на столбец *PersonId* таблицы *Person*. Изучите схему, чтобы понять, как таблицы в этом руководстве связаны друг с другом. Подробные сведения о создании эффективных таблиц баз данных см. в [этой статье](/previous-versions/tn-archive/cc505842(v=technet.10)). Дополнительные сведения о выборе типов данных см. в [этой статье](/sql/t-sql/data-types/data-types-transact-sql).

> [!NOTE]
> Для создания и проектирования таблиц можно также использовать [конструктор таблиц в SQL Server Management Studio](/sql/ssms/visual-db-tools/design-database-diagrams-visual-database-tools).

![Связи между таблицами](./media/design-first-database-tutorial/tutorial-database-tables.png)

1. В **обозревателе объектов** щелкните правой кнопкой мыши *yourDatabase* и выберите команду **Создать запрос**. Откроется пустое окно запроса, подключенное к базе данных.

2. Чтобы создать в базе данных четыре таблицы, в окне запроса выполните следующий запрос:

   ```sql
   -- Create Person table
   CREATE TABLE Person
   (
       PersonId INT IDENTITY PRIMARY KEY,
       FirstName NVARCHAR(128) NOT NULL,
       MiddelInitial NVARCHAR(10),
       LastName NVARCHAR(128) NOT NULL,
       DateOfBirth DATE NOT NULL
   )

   -- Create Student table
   CREATE TABLE Student
   (
       StudentId INT IDENTITY PRIMARY KEY,
       PersonId INT REFERENCES Person (PersonId),
       Email NVARCHAR(256)
   )

   -- Create Course table
   CREATE TABLE Course
   (
       CourseId INT IDENTITY PRIMARY KEY,
       Name NVARCHAR(50) NOT NULL,
       Teacher NVARCHAR(256) NOT NULL
   )

   -- Create Credit table
   CREATE TABLE Credit
   (
       StudentId INT REFERENCES Student (StudentId),
       CourseId INT REFERENCES Course (CourseId),
       Grade DECIMAL(5,2) CHECK (Grade <= 100.00),
       Attempt TINYINT,
       CONSTRAINT [UQ_studentgrades] UNIQUE CLUSTERED
       (
           StudentId, CourseId, Grade, Attempt
       )
   )
   ```

   ![Создание таблиц](./media/design-first-database-tutorial/create-tables.png)

3. В разделе *yourDatabase* в **обозревателе объектов** разверните узел **Tables**, чтобы просмотреть созданные таблицы.

   ![Создание таблиц в SSMS](./media/design-first-database-tutorial/ssms-tables-created.png)

## <a name="load-data-into-the-tables"></a>Загрузка данных в таблицу

1. В папке загрузок создайте папку с именем *sampleData* для хранения примеров данных базы данных.

2. Щелкните правой кнопкой мыши приведенные ниже ссылки и сохраните их в папку *sampleData*.

   - [SampleCourseData](https://sqldbtutorial.blob.core.windows.net/tutorials/SampleCourseData)
   - [SamplePersonData](https://sqldbtutorial.blob.core.windows.net/tutorials/SamplePersonData)
   - [SampleStudentData](https://sqldbtutorial.blob.core.windows.net/tutorials/SampleStudentData)
   - [SampleCreditData](https://sqldbtutorial.blob.core.windows.net/tutorials/SampleCreditData)

3. Откройте окно командной строки и перейдите в папку *sampleData*.

4. Выполните приведенные ниже команды, которые вставляют пример данных в таблицы. Укажите значения для *сервера*, *базы данных*, *пользователя* и *пароля*, соответствующие вашей среде.

   ```cmd
   bcp Course in SampleCourseData -S <server>.database.windows.net -d <database> -U <user> -P <password> -q -c -t ","
   bcp Person in SamplePersonData -S <server>.database.windows.net -d <database> -U <user> -P <password> -q -c -t ","
   bcp Student in SampleStudentData -S <server>.database.windows.net -d <database> -U <user> -P <password> -q -c -t ","
   bcp Credit in SampleCreditData -S <server>.database.windows.net -d <database> -U <user> -P <password> -q -c -t ","
   ```

Итак, вы загрузили пример данных в созданные ранее таблицы.

## <a name="query-data"></a>Данные запросов

Чтобы извлечь сведения из таблиц базы данных, выполните приведенные ниже запросы. Дополнительные сведения о создании запросов SQL см. в [этой статье](/previous-versions/sql/sql-server-2005/express-administrator/bb264565(v=sql.90)). Первый запрос объединяет четыре таблицы для поиска студентов, которые посещают занятия у преподавателя Dominick Pope и оценки которых выше 75 %. Второй запрос объединяет четыре таблицы и находит курсы, на которые когда-либо записывался Noe Coleman.

1. В окне запроса SQL Server Management Studio выполните следующий запрос:

   ```sql
   -- Find the students taught by Dominick Pope who have a grade higher than 75%
   SELECT  person.FirstName, person.LastName, course.Name, credit.Grade
   FROM  Person AS person
       INNER JOIN Student AS student ON person.PersonId = student.PersonId
       INNER JOIN Credit AS credit ON student.StudentId = credit.StudentId
       INNER JOIN Course AS course ON credit.CourseId = course.courseId
   WHERE course.Teacher = 'Dominick Pope'
       AND Grade > 75
   ```

2. В окне запроса выполните такой запрос:

   ```sql
   -- Find all the courses in which Noe Coleman has ever enrolled
   SELECT  course.Name, course.Teacher, credit.Grade
   FROM  Course AS course
       INNER JOIN Credit AS credit ON credit.CourseId = course.CourseId
       INNER JOIN Student AS student ON student.StudentId = credit.StudentId
       INNER JOIN Person AS person ON person.PersonId = student.PersonId
   WHERE person.FirstName = 'Noe'
       AND person.LastName = 'Coleman'
   ```

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали множество основных задач базы данных. Вы ознакомились с выполнением следующих задач:

> [!div class="checklist"]
>
> - создать базу данных с помощью портала Azure*;
> - настроить правила брандмауэра на уровне сервера с помощью портала Azure;
> - Подключение к базе данных с помощью SQL Server Management Studio.
> - создать таблицы с помощью SSMS;
> - выполнить массовую загрузку данных с помощью BCP;
> - запросить данные с помощью SSMS.

Дополнительные сведения о проектировании базы данных с помощью Visual Studio и C# см. в следующем руководстве.

> [!div class="nextstepaction"]
> [Проектирование реляционной базы данных в службе "База данных SQL Azure" с помощью C# и ADO.NET](design-first-database-csharp-tutorial.md)