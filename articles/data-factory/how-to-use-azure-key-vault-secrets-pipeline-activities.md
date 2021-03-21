---
title: Использование секретов Azure Key Vault в действиях конвейера
description: Узнайте, как извлечь сохраненные учетные данные из хранилища ключей Azure и использовать их во время выполнения конвейера фабрики данных.
author: ChrisLound
ms.author: chlound
ms.service: data-factory
ms.topic: conceptual
ms.date: 10/31/2019
ms.openlocfilehash: d7e1133b8f3880c4c1616ef5ca955ed014348935
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100383965"
---
# <a name="use-azure-key-vault-secrets-in-pipeline-activities"></a>Использование секретов Azure Key Vault в действиях конвейера

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Можно сохранить учетные данные или секретные значения в Azure Key Vault и использовать их во время выполнения конвейера для передачи в действия.

## <a name="prerequisites"></a>Предварительные условия

Эта функция основана на управляемом удостоверении фабрики данных.  Узнайте, как это работает из [управляемого удостоверения для фабрики данных](./data-factory-service-identity.md) и убедитесь, что фабрика данных имеет одну связь.

## <a name="steps"></a>Шаги

1. Откройте свойства фабрики данных и скопируйте значение идентификатора приложения управляемого удостоверения.

    ![Значение управляемого идентификатора](media/how-to-use-azure-key-vault-secrets-pipeline-activities/managedidentity.png)

2. Откройте политики доступа к хранилищу ключей и добавьте разрешения управляемого удостоверения для получения и перечисления секретов.

    ![Снимок экрана, на котором показана страница "политики доступа" с выделенным действием "добавить политику доступа".](media/how-to-use-azure-key-vault-secrets-pipeline-activities/akvaccesspolicies.png)

    ![Политики доступа Key Vault](media/how-to-use-azure-key-vault-secrets-pipeline-activities/akvaccesspolicies-2.png)

    Нажмите кнопку **Добавить**, а затем — **сохранить**.

3. Перейдите к Key Vault секрету и скопируйте идентификатор секрета.

    ![Идентификатор секрета](media/how-to-use-azure-key-vault-secrets-pipeline-activities/secretidentifier.png)

    Запишите URI секрета, который вы хотите получить во время выполнения конвейера фабрики данных.

4. В конвейере фабрики данных добавьте новое веб-действие и настройте его следующим образом.  

    |Свойство.  |Значение  |
    |---------|---------|
    |Безопасный вывод     |Верно         |
    |URL-адрес     |[Значение URI секрета]? API-Version = 7.0         |
    |Метод     |GET         |
    |Аутентификация     |MSI         |
    |Ресурс        |https://vault.azure.net       |

    ![Веб-действие](media/how-to-use-azure-key-vault-secrets-pipeline-activities/webactivity.png)

    > [!IMPORTANT]
    > Необходимо добавить **? API-Version = 7.0** в конец URI секрета.  

    > [!CAUTION]
    > Задайте для параметра безопасный вывод значение true, чтобы предотвратить запись секретного значения в обычный текст.  Для всех дальнейших действий, которые используют это значение, параметр "Защищенный ввод" должен иметь значение true.

5. Чтобы использовать значение в другом действии, используйте следующее выражение кода **@activity ("web1"). Output. Value**.

    ![Выражение кода](media/how-to-use-azure-key-vault-secrets-pipeline-activities/usewebactivity.png)

## <a name="next-steps"></a>Дальнейшие действия

Сведения об использовании Azure Key Vault для хранения учетных данных для хранилищ данных и вычислений см. [в разделе Хранение учетных данных в Azure Key Vault](./store-credentials-in-key-vault.md)