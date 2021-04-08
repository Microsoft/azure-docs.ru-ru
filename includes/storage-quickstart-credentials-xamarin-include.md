---
title: включить файл
description: включить файл
services: storage
author: codemillmatt
ms.service: storage
ms.topic: include
ms.date: 11/23/2019
ms.author: masoucou
ms.custom: include file
ms.openlocfilehash: 02586d38903c60ba8982753ca0bd3e15192d5deb
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "83006380"
---
### <a name="copy-your-credentials-from-the-azure-portal"></a>Копирование учетных данных с портала Azure

Когда пример приложения выполняет запрос к службе хранилища Azure, он должен быть авторизован. Для авторизации запроса добавьте в приложение учетные данные учетной записи хранения в виде строки подключения. Чтобы просмотреть учетные данные учетной записи хранения, выполните следующие действия:

1. Войдите на [портал Azure](https://portal.azure.com).
2. Перейдите к учетной записи хранения.
3. В разделе **Параметры** учетной записи хранения выберите параметр **Ключи доступа**. На этой странице вы увидите ключи доступа к учетной записи и полную строку подключения для каждого ключа.
4. Найдите значение для параметра **Строка подключения** в разделе **Key1** и нажмите кнопку **Скопировать**, чтобы скопировать строку подключения. На следующем этапе вы добавите значение строки подключения в переменную среды.

    ![Снимок экрана, на котором показано, как скопировать строку подключения с портала Azure](./media/storage-copy-connection-string-portal/portal-connection-string.png)

### <a name="configure-your-storage-connection-string"></a>Настройка строки подключения хранилища

Когда скопируете строку подключения, укажите ее для переменной уровня класса в файле *MainPage.xaml.cs*. Откройте файл *MainPaage.xaml.cs* и найдите переменную `storageConnectionString`. Замените `<yourconnectionstring>` фактической строкой подключения.

Вот этот код:

```csharp
string storageConnectionString = "<yourconnectionstring>";
```