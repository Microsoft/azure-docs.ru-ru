---
title: Как проверить базу данных Azure Cosmos (API SQL)
description: В этом разделе описывается, как проверить базу данных Azure Cosmos (API SQL).
author: djpmsft
ms.author: daperlov
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 10/9/2020
ms.openlocfilehash: 1aaeed1973ebd15af312b722ab61938aa4271947
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97696245"
---
# <a name="register-and-scan-azure-cosmos-database-sql-api"></a>Регистрация и проверка базы данных Azure Cosmos (API SQL)

В этой статье описано, как зарегистрировать учетную запись базы данных Azure Cosmos (SQL API) в Azure зрения и настроить проверку.

## <a name="supported-capabilities"></a>Поддерживаемые возможности

База данных Azure Cosmos (API SQL) поддерживает полный и добавочный просмотр для записи метаданных и схемы. Проверки также классифицируют данные автоматически на основе системных и настраиваемых правил классификации.

## <a name="prerequisites"></a>Предварительные условия

- Перед регистрацией источников данных создайте учетную запись Azure зрения. Дополнительные сведения о создании учетной записи зрения см. в разделе [Краткое руководство. Создание учетной записи Azure зрения](create-catalog-portal.md).
- Необходимо быть администратором источника данных Azure зрения.

## <a name="setting-up-authentication-for-a-scan"></a>Настройка проверки подлинности для сканирования

Существует только один способ настройки проверки подлинности для базы данных Azure Cosmos (API SQL):

- Ключ учетной записи
 
### <a name="account-key"></a>Ключ учетной записи

Если выбран метод проверки подлинности " **ключ учетной записи**", необходимо получить ключ доступа и сохранить его в хранилище ключей:

1. Перейдите к учетной записи Cosmos DB в портал Azure 
1. Выбор **параметров**  >  **ключи** 
1. Скопируйте *ключ* и сохраните его в любое место для дальнейших действий
1. Перейдите в хранилище ключей.
1. Выберите **Параметры > Секреты**.
1. Выберите **+ создать/импортировать** и введите **имя** и **значение** в качестве *ключа* из вашей учетной записи хранения.
1. Чтобы завершить процесс, щелкните **Создать**.
1. Если хранилище ключей еще не подключено к службе Purview, вам придется [создать новое подключение к хранилищу ключей](manage-credentials.md#create-azure-key-vaults-connections-in-your-azure-purview-account).
1. Наконец, [Создайте новые учетные данные](manage-credentials.md#create-a-new-credential) с помощью ключа, чтобы настроить проверку.

## <a name="register-an-azure-cosmos-database-sql-api-account"></a>Регистрация учетной записи базы данных Azure Cosmos (SQL API)

Чтобы зарегистрировать новую учетную запись базы данных Azure Cosmos (SQL API) в каталоге данных, выполните следующие действия.

1. Перейдите к учетной записи Purview.
1. В области навигации слева выберите **Источники**.
1. Нажмите кнопку **Зарегистрировать**.
1. В меню **Регистрация источников** выберите **Azure Cosmos DB (API SQL)** .
1. Выберите **Продолжить**

:::image type="content" source="media/register-scan-azure-cosmos-database/register-new-data-source.png" alt-text="Регистрация источника данных" border="true":::

На экране **Регистрация источников (Azure Cosmos DB (API SQL))** выполните следующие действия.

1. Введите **Имя**, под которым этот источник данных будет включен в каталог.
1. Выберите способ указания нужной учетной записи хранения:
   1. Выберите **из подписки Azure**, выберите подходящую подписку из раскрывающегося списка **Подписка Azure** и соответствующую учетную запись cosmosDB из раскрывающегося списка **имя учетной записи Cosmos DB** .
   1. Можно также нажать клавишу **Ввод вручную** и ввести конечную точку службы (URL-адрес).
1. Щелкните **Готово**, чтобы завершить регистрацию источника данных.

:::image type="content" source="media/register-scan-azure-cosmos-database/register-sources.png" alt-text="Параметры регистрации источников" border="true":::


[!INCLUDE [create and manage scans](includes/manage-scans.md)]

## <a name="next-steps"></a>Дальнейшие действия

- [Обзор каталога данных Azure Purview](how-to-browse-catalog.md)
- [Поиск по каталогу данных Azure Purview](how-to-search-catalog.md)
