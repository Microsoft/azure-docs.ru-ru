---
title: Проблемы совместимости с приложениями сторонних производителей и Azure синапсе Analytics
description: Описание известных проблем, которые сторонние приложения могут найти с помощью Azure синапсе
services: synapse-analytics
author: periclesrocha
ms.service: synapse-analytics
ms.topic: troubleshooting
ms.subservice: ''
ms.date: 11/18/2020
ms.author: procha
ms.reviewer: jrasnick
ms.openlocfilehash: a1031656eaa5125d07ae078773379270b26625e7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98121385"
---
# <a name="compatibility-issues-with-third-party-applications-and-azure-synapse-analytics"></a>Проблемы совместимости с приложениями сторонних производителей и Azure синапсе Analytics

Приложения, созданные для SQL Server, будут беспрепятственно работать с выделенными пулами SQL Azure синапсе. Однако в некоторых случаях функции и элементы языка, которые обычно используются в SQL Server, могут быть недоступны в Azure синапсе, иначе они могут вести себя по-разному.

В этой статье перечислены известные проблемы, которые могут возникнуть при использовании сторонних приложений с Azure синапсе Analytics. 

## <a name="tableau-error-an-attempt-to-complete-a-transaction-has-failed-no-corresponding-transaction-found"></a>Ошибка Tableau: "не удалось выполнить транзакцию. Соответствующие транзакции не найдены "

Начиная с выделенного пула SQL Azure синапсе версии 10.0.11038.0 некоторые запросы Tableau, выполняющие вызовы хранимых процедур, могут завершиться со следующим сообщением об ошибке: "**[Microsoft] [ODBC Driver 17 for SQL Server] [SQL Server] 111214; Сбой при выполнении транзакции. Соответствующие транзакции не найдены».**

### <a name="cause"></a>Причина

Это проблема в выделенном пуле SQL Azure синапсе, вызванная введением новых системных хранимых процедур, которые вызываются автоматически драйверами ODBC и JDBC. Одна из этих системных хранимых процедур может привести к прерыванию открытых транзакций в случае сбоя их выполнения. Эта проблема может возникнуть в зависимости от логики клиентского приложения.

### <a name="solution"></a>Решение
Клиенты, которые видят эту конкретную задачу при использовании Tableau, подключенных к выделенным пулам SQL синапсе, должны установить для FMTONLY значение Да в подключении SQL. Для Tableau Desktop и Tableau Server следует использовать файл настройки источника данных Tableau (TDC), чтобы гарантировать, что Tableau передает этот параметр драйверу.  

> [!NOTE] 
> Корпорация Майкрософт не предоставляет поддержку сторонним средствам. Хотя мы протестировали, что это решение работает с Tableau Desktop 2020.3.2, следует использовать это решение в своей емкости.
>

* [Сведения о том, как выполнять глобальные настройки с помощью TDC-файла в Tableau Desktop, см. в документации по Tableau Desktop.](https://help.tableau.com/current/pro/desktop/en-us/odbc_customize.htm)
* [Сведения о том, как выполнять глобальные настройки с помощью TDC-файла на сервере Tableau, см. в разделе Использование. TDC файл с сервером Tableau.](https://kb.tableau.com/articles/howto/using-a-tdc-file-with-tableau-server)

В приведенном ниже примере показан файл TDC Tableau, передающий параметр FMTONLY = YES строке подключения SQL:

```json
<connection-customization class='azure_sql_dw' enabled='true' version='18.1'>
    <vendor name='azure_sql_dw' />
    <driver name='azure_sql_dw' />
    <customizations>        
        <customization name='odbc-connect-string-extras' value='UseFMTONLY=yes' />
    </customizations>
</connection-customization>
```
Для получения дополнительных сведений об использовании файлов TDC обратитесь в службу поддержки Tableau. 

## <a name="see-also"></a>См. также раздел

* [Элементы языка T-SQL для выделенного пула SQL в Azure синапсе Analytics.](./sql-data-warehouse-reference-tsql-language-elements.md?bc=%2fazure%2fsynapse-analytics%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fsynapse-analytics%2ftoc.json)
* [Инструкции T-SQL, поддерживаемые для выделенного пула SQL в Azure синапсе Analytics.](./sql-data-warehouse-reference-tsql-statements.md)