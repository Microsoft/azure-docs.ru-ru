---
title: Получение строки подключения на портале Azure
description: Получение строки подключения на портале Azure
keywords: Подключения SQL, строка подключения
services: sql-database
author: dalechen
manager: craigg
ms.service: sql-database
ms.custom: develop apps
ms.topic: include
ms.date: 07/13/2018
ms.author: ninarn
ms.openlocfilehash: b06004143d4a1ec5a2808e21b1f78833e410c195
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95558920"
---
### <a name="obtain-the-connection-string-from-the-azure-portal"></a>Получение строки подключения на портале Azure
Для получения строки подключения, необходимой для взаимодействия клиентской программы с Базой данных SQL Azure, используйте [портал Azure](https://portal.azure.com/).

1. Выберите **все службы**  >  **базы данных SQL**.

2. Введите имя базы данных в текстовое поле фильтра рядом с верхней левой частью колонки **Базы данных SQL**.

3. Выберите строку со своей базой данных.

4. Для удобства после появления колонки для базы данных нажмите кнопку **Свернуть**, чтобы свернуть колонки, используемые для просмотра и фильтрации базы данных.

5. В колонке для базы данных выберите **Показать строки подключения к базе данных**.

6. Скопируйте нужную строку подключения. т.e. Если вы планируете использовать библиотеку подключений ADO.NET, скопируйте соответствующую строку из вкладки **ADO.NET**.

    ![Копирование строки подключения ADO для базы данных][20-CopyAdoConnectionString]

7. При необходимости исправьте информацию в строке подключения. т.e. Вставьте пароль в строку подключения или удалите "@&lt;servername&gt;" в имени пользователя, если имя пользователя или имя сервера слишком длинные.

8. Вставьте строку подключения в код клиентской программы в одном или другом формате.

Дополнительные сведения см. в описании [строк подключения и файлов конфигурации](/dotnet/framework/data/adonet/connection-strings-and-configuration-files).

<!-- Image references. -->



[20-CopyAdoConnectionString]: ./media/sql-database-include-connection-string-20-portalshots/connqry-connstr-b.png


<!--
These three includes/ files are a sequenced set, but you can pick and choose:

includes/sql-database-include-connection-string-20-portalshots.md
includes/sql-database-include-connection-string-30-compare.md
includes/sql-database-include-connection-string-40-config.md
-->