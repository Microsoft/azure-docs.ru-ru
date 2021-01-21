---
title: Синтаксис действия SQL правила подписки служебной шины Azure | Документация Майкрософт
description: В этой статье содержится ссылка на синтаксис действия правила SQL. Действия записываются в синтаксисе на основе языка SQL, который выполняется для сообщения.
ms.topic: article
ms.date: 11/24/2020
ms.openlocfilehash: 606281d42d5598d7f73312990d3a19775a202c08
ms.sourcegitcommit: 484f510bbb093e9cfca694b56622b5860ca317f7
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/21/2021
ms.locfileid: "98632817"
---
# <a name="subscription-rule-sql-action-syntax"></a>Синтаксис действия правила подписки SQL

*Действие SQL* используется для работы с метаданными сообщений после того, как сообщение было выбрано фильтром правила подписки. Это текстовое выражение, которое используется в качестве основы для подмножества стандарта SQL-92. Выражения действий используются с `sqlExpression` элементом свойства Action служебной шины `Rule` в [шаблоне Azure Resource Manager](service-bus-resource-manager-namespace-topic-with-rule.md)или в Azure CLI `az servicebus topic subscription rule create` [`--action-sql-expression`](/cli/azure/servicebus/topic/subscription/rule#az_servicebus_topic_subscription_rule_create) аргументе команды и нескольких функциях пакета SDK, которые позволяют управлять правилами подписки.
  
  
```  
<statements> ::=
    <statement> [, ...n]  
  
```  
  
```  
<statement> ::=
    <action> [;]
    Remarks
    -------
    Semicolon is optional.  
  
```  
  
```  
<action> ::=
    SET <property> = <expression>
    REMOVE <property>  
```

```
<expression> ::=
    <constant>
    | <function>
    | <property>
    | <expression> { + | - | * | / | % } <expression>
    | { + | - } <expression>
    | ( <expression> )
``` 

```
<property> := 
    [<scope> .] <property_name>
``` 
  
## <a name="arguments"></a>Аргументы  
  
-   `<scope>` — необязательная строка, указывающая область `<property_name>`. Допустимые значения: `sys` или `user`. Значение `sys` указывает область системы, где `<property_name>` — имя общедоступного свойства [класса BrokeredMessage](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage). Значение `user` указывает область пользователя, где `<property_name>` — ключ словаря [класса BrokeredMessage](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage). Область `user` используется по умолчанию, если аргумент `<scope>` не указан.  
  
### <a name="remarks"></a>Комментарии  

Попытка доступа к несуществующему системному свойству вызывает ошибку, а попытка доступа к несуществующему свойству пользователя — нет. Вместо этого несуществующее свойство пользователя вычисляется внутри системы как неизвестное значение. Неизвестное значение обрабатывается особым образом во время вычисления оператора.  
  
## <a name="property_name"></a>property_name  
  
```  
<property_name> ::=  
     <identifier>  
     | <delimited_identifier>  
  
<identifier> ::=  
     <regular_identifier> | <quoted_identifier> | <delimited_identifier>  
  
```  
  
### <a name="arguments"></a>Аргументы  
 `<regular_identifier>` — строка, представленная следующим регулярным выражением:  
  
```  
[[:IsLetter:]][_[:IsLetter:][:IsDigit:]]*  
```  
  
 Это любая строка, которая начинается с буквы, после которой идет один или несколько символов подчеркивания, букв или цифр.  
  
 `[:IsLetter:]` — любой символ Юникода, который относится к категории букв Юникода. `System.Char.IsLetter(c)` возвращает значение `true`, если `c` является буквой Юникода.  
  
 `[:IsDigit:]` — любой символ Юникода, который относится к категории десятичных цифр. `System.Char.IsDigit(c)` возвращает значение `true`, если `c` является цифрой Юникода.  
  
 `<regular_identifier>` не может быть зарезервированным ключевым словом.  
  
 `<delimited_identifier>` — любая строка, заключенная в квадратные скобки ([]). Закрывающая квадратная скобка представляется в виде двух закрывающих квадратных скобок. Ниже приведены примеры `<delimited_identifier>`.  
  
```  
[Property With Space]  
[HR-EmployeeID]  
  
```  
  
 `<quoted_identifier>` — любая строка, заключенная в двойные кавычки. Двойные кавычки в идентификаторе представляются в виде двух пар двойных кавычек. Мы не рекомендуем использовать заключенные в кавычки идентификаторы, так как их можно легко спутать со строковой константой. По возможности используйте идентификаторы с разделителем. Ниже приведен пример `<quoted_identifier>`.  
  
```  
"Contoso & Northwind"  
```  
  
## <a name="pattern"></a>pattern  
  
```  
<pattern> ::=  
      <expression>  
```  
  
### <a name="remarks"></a>Комментарии
  
 Свойство `<pattern>` должно быть выражением, которое будет вычисляться как строка. Оно используется в качестве шаблона для оператора LIKE.      Оно может содержать следующие подстановочные знаки:  
  
-   `%` — любая строка без символов или с несколькими символами.  
  
-   `_` — любой один символ.  
  
## <a name="escape_char"></a>escape_char  
  
```  
<escape_char> ::=  
      <expression>  
```  
  
### <a name="remarks"></a>Комментарии
  
 Свойство `<escape_char>` должно быть выражением, которое будет вычисляться в качестве строки с 1 символом. Оно используется в качестве escape-символа для оператора LIKE.  
  
 Например, `property LIKE 'ABC\%' ESCAPE '\'` соответствует `ABC%`, а не строке, начинающейся с `ABC`.  
  
## <a name="constant"></a>constant  
  
```  
<constant> ::=  
      <integer_constant> | <decimal_constant> | <approximate_number_constant> | <boolean_constant> | NULL  
```  
  
### <a name="arguments"></a>Аргументы  
  
-   `<integer_constant>` — строка чисел без кавычек и десятичного разделителя. Значения хранятся в виде `System.Int64` внутри системы и попадают в тот же диапазон.  
  
     Ниже приведены примеры длинных констант.  
  
    ```  
    1894  
    2  
    ```  
  
-   `<decimal_constant>` — строка чисел без кавычек с десятичным разделителем. Значения хранятся в виде `System.Double` внутри системы и попадают в тот же диапазон и точность.  
  
     В следующих версиях это число может храниться в другом типе данных для обеспечения поддержки более точной числовой семантики. Поэтому не следует полагаться на тот факт, что базовый тип данных для `<decimal_constant>` — `System.Double`.  
  
     Ниже приведены примеры десятичных констант.  
  
    ```  
    1894.1204  
    2.0  
    ```  
  
-   `<approximate_number_constant>` — число, записанное в экспоненциальном представлении чисел. Значения хранятся в виде `System.Double` внутри системы и попадают в тот же диапазон и точность. Ниже приведены примеры констант с приближенными числами.  
  
    ```  
    101.5E5  
    0.5E-2  
    ```  
  
## <a name="boolean_constant"></a>boolean_constant  
  
```  
<boolean_constant> :=  
      TRUE | FALSE  
```  
  
### <a name="remarks"></a>Комментарии
  
Логические константы представлены в виде ключевого слова `TRUE` или `FALSE`. Значения хранятся в виде `System.Boolean`.  
  
## <a name="string_constant"></a>string_constant  
  
```  
<string_constant>  
```  
  
### <a name="remarks"></a>Комментарии
  
Строковые константы заключаются в одинарные кавычки и включают любые допустимые символы Юникода. Одинарная кавычка, внедренная в строковую константу, представляется в виде двух одинарных кавычек.  
  
## <a name="function"></a>function  
  
```  
<function> :=  
      newid() |  
      property(name) | p(name)  
```  
  
### <a name="remarks"></a>Комментарии  

Функция `newid()` возвращает значение **System.Guid**, созданное методом `System.Guid.NewGuid()`.  
  
Функция `property(name)` возвращает значение свойства, на которое указывает `name`. В качестве значения `name` может использоваться любое допустимое выражение, возвращающее строковое значение.  
  
## <a name="considerations"></a>Рекомендации

- Оператор SET используется для создания свойства или обновления значения имеющегося свойства.
- Оператор REMOVE используется для удаления свойства.
- По возможности оператор SET выполняет неявное преобразование, если тип выражения и тип имеющегося свойства отличаются.
- Если ссылка указывает на несуществующие свойства системы, действие завершается сбоем.
- Если ссылка указывает на несуществующие свойства пользователя, действие не завершается сбоем.
- Несуществующее свойство пользователя вычисляется внутри системы как неизвестное значение с использованием той же семантики, что и [SQLFilter](/dotnet/api/microsoft.servicebus.messaging.sqlfilter) при вычислении операторов.

## <a name="next-steps"></a>Дальнейшие действия

- [Класс SQLRuleAction (платформа .NET Framework)](/dotnet/api/microsoft.servicebus.messaging.sqlruleaction)
- [Класс SQLRuleAction (.NET Standard)](/dotnet/api/microsoft.azure.servicebus.sqlruleaction)
- [Класс SqlRuleAction (Java)](/java/api/com.microsoft.azure.servicebus.rules.sqlruleaction)
- [SqlRuleAction (JavaScript)](/javascript/api/@azure/service-bus/sqlruleaction)
- [AZ servicebus раздел правило подписки](/cli/azure/servicebus/topic/subscription/rule)
- [New-Азсервицебусруле](/powershell/module/az.servicebus/new-azservicebusrule)