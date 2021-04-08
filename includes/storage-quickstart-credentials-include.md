---
title: включить файл
description: включить файл
services: storage
author: mhopkins-msft
ms.service: storage
ms.topic: include
ms.date: 11/23/2019
ms.author: mhopkins
ms.custom: include file
ms.openlocfilehash: 7dd22886d11c3a35a7a866ff7c9a4f56ea74cab7
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "75351220"
---
### <a name="copy-your-credentials-from-the-azure-portal"></a>Копирование учетных данных с портала Azure

Когда пример приложения выполняет запрос к службе хранилища Azure, он должен быть авторизован. Для авторизации запроса добавьте в приложение учетные данные учетной записи хранения в виде строки подключения. Чтобы просмотреть учетные данные учетной записи хранения, выполните следующие действия:

1. Войдите на [портал Azure](https://portal.azure.com).
2. Перейдите к учетной записи хранения.
3. В разделе **Параметры** учетной записи хранения выберите параметр **Ключи доступа**. На этой странице вы увидите ключи доступа к учетной записи и полную строку подключения для каждого ключа.
4. Найдите значение для параметра **Строка подключения** в разделе **Key1** и нажмите кнопку **Скопировать**, чтобы скопировать строку подключения. На следующем этапе вы добавите значение строки подключения в переменную среды.

    ![Снимок экрана, на котором показано, как скопировать строку подключения с портала Azure](./media/storage-copy-connection-string-portal/portal-connection-string.png)

### <a name="configure-your-storage-connection-string"></a>Настройка строки подключения хранилища

После копирования строки подключения запишите ее в переменной среды на локальном компьютере, где выполняется приложение. Чтобы задать переменную среды, откройте окно консоли и следуйте инструкциям для используемой операционной системы. Замените `<yourconnectionstring>` фактической строкой подключения.

#### <a name="windows"></a>Windows

```cmd
setx AZURE_STORAGE_CONNECTION_STRING "<yourconnectionstring>"
```

После добавления переменной среды в Windows вам необходимо запустить новый экземпляр командного окна.

#### <a name="linux"></a>Linux

```bash
export AZURE_STORAGE_CONNECTION_STRING="<yourconnectionstring>"
```

#### <a name="macos"></a>macOS

```bash
export AZURE_STORAGE_CONNECTION_STRING="<yourconnectionstring>"
```

#### <a name="restart-programs"></a>Перезапуск программ

После добавления переменной среды перезапустите все запущенные программы, которым может понадобиться считать переменную среды. Например, перезапустите среду разработки или редактор, прежде чем продолжить.
