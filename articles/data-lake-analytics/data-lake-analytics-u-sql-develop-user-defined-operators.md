---
title: Разработка определяемых пользователем операторов U-SQL — Azure Data Lake Analytics
description: Узнайте, как разрабатывать определяемые пользователем операторы (с возможностью повторного использования) в заданиях Azure Data Lake Analytics.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 12/05/2016
ms.openlocfilehash: 11efdb727bacadb674fb49374ef1c70fcc788ecc
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92219995"
---
# <a name="develop-u-sql-user-defined-operators-udos"></a>Разработка определяемых пользователем операторов U-SQLU (UDO)
В этой статье описано, как разрабатывать определяемые пользователем операторы для обработки данных в задании U-SQL.

## <a name="define-and-use-a-user-defined-operator-in-u-sql"></a>Определение и использование определяемых пользователем операторов в U-SQL

### <a name="to-create-and-submit-a-u-sql-job"></a>Создание и отправка задания U-SQL

1. В Visual Studio выберите **Файл > Создать > Проект > Проект U-SQL**.
2. Нажмите кнопку **OK**. Visual Studio создаст решение с помощью файла Script.usql.
3. В **обозревателе решений** разверните узел Script.usql и дважды щелкните файл **Script.usql.cs**.
4. Вставьте в файл приведенный ниже код.

   ```usql
   using Microsoft.Analytics.Interfaces;
   using System.Collections.Generic;
   namespace USQL_UDO
   {
       public class CountryName : IProcessor
       {
           private static IDictionary<string, string> CountryTranslation = new Dictionary<string, string>
           {
               {
                   "Deutschland", "Germany"
               },
               {
                   "Suisse", "Switzerland"
               },
               {
                   "UK", "United Kingdom"
               },
               {
                   "USA", "United States of America"
               },
               {
                   "中国", "PR China"
               }
           };
           public override IRow Process(IRow input, IUpdatableRow output)
           {
               string UserID = input.Get<string>("UserID");
               string Name = input.Get<string>("Name");
               string Address = input.Get<string>("Address");
               string City = input.Get<string>("City");
               string State = input.Get<string>("State");
               string PostalCode = input.Get<string>("PostalCode");
               string Country = input.Get<string>("Country");
               string Phone = input.Get<string>("Phone");
               if (CountryTranslation.Keys.Contains(Country))
               {
                   Country = CountryTranslation[Country];
               }
               output.Set<string>(0, UserID);
               output.Set<string>(1, Name);
               output.Set<string>(2, Address);
               output.Set<string>(3, City);
               output.Set<string>(4, State);
               output.Set<string>(5, PostalCode);
               output.Set<string>(6, Country);
               output.Set<string>(7, Phone);
               return output.AsReadOnly();
           }
       }
   }
   ```

5. Откройте файл **Script.usql** и вставьте следующий сценарий U-SQL:

   ```usql
   @drivers =
       EXTRACT UserID      string,
               Name        string,
               Address     string,
               City        string,
               State       string,
               PostalCode  string,
               Country     string,
               Phone       string
       FROM "/Samples/Data/AmbulanceData/Drivers.txt"
       USING Extractors.Tsv(Encoding.Unicode);

   @drivers_CountryName =
       PROCESS @drivers
       PRODUCE UserID string,
               Name string,
               Address string,
               City string,
               State string,
               PostalCode string,
               Country string,
               Phone string
       USING new USQL_UDO.CountryName();

   OUTPUT @drivers_CountryName
       TO "/Samples/Outputs/Drivers.csv"
       USING Outputters.Csv(Encoding.Unicode);
   ```

6. Укажите учетную запись аналитики озера данных, базу данных и схему.
7. В **обозревателе решений** щелкните правой кнопкой мыши файл **Script.usql** и выберите команду **Создать сценарий**.
8. В **обозревателе решений** щелкните правой кнопкой мыши файл **Script.usql** и выберите команду **Отправить сценарий**.
9. Если вы еще не подключились к своей подписке Azure, вам будет предложено ввести учетные данные Azure.
10. Щелкните **Отправить**. Итоги отправки и ссылка на задание появятся в окне результатов, когда операция отправки будет завершена.
11. Чтобы увидеть последнее состояние задания, нажмите кнопку **Обновить**.

### <a name="to-see-the-output"></a>Просмотр выходных данных

1. В **обозревателе сервера** разверните узлы **Azure**, **Data Lake Analytics**, а также учетную запись Data Lake Analytics. Затем разверните узел **Учетные записи хранения**, щелкните хранилище по умолчанию правой кнопкой мыши и выберите **Обозреватель**.

2. Разверните узлы «Примеры» и «Выходные данные», а затем дважды щелкните **Drivers.csv**.

## <a name="next-steps"></a>Дальнейшие действия

* [Extending U-SQL Expressions with User-Code](/u-sql/concepts/extending-u-sql-expressions-with-user-code) (Расширение выражений U-SQL с помощью пользовательского кода)
* [Использование инструментов озера данных для Visual Studio для разработки приложений U-SQL](data-lake-analytics-data-lake-tools-get-started.md)
