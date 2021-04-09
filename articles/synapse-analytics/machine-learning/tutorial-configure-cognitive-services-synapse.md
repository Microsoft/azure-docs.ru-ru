---
title: Руководство. Необходимые компоненты для Cognitive Services в Azure Synapse Analytics
description: Сведения о том, как настроить необходимые компоненты для использования Cognitive Services в Azure Synapse.
services: synapse-analytics
ms.service: synapse-analytics
ms.subservice: machine-learning
ms.topic: tutorial
ms.reviewer: jrasnick, garye
ms.date: 11/20/2020
author: nelgson
ms.author: negust
ms.openlocfilehash: 3ab861caca0ef6f58c2c1bc722412774deb725ce
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98936682"
---
# <a name="tutorial-prerequisites-for-using-cognitive-services-in-azure-synapse-analytics"></a>Руководство. Необходимые компоненты для использования Cognitive Services в Azure Synapse Analytics

В этом учебнике показано, как настроить необходимые компоненты для безопасного использования Cognitive Services в Azure Synapse Analytics.

Темы, рассматриваемые в этом руководстве:
> [!div class="checklist"]
> - создание ресурса Cognitive Services, например для Анализа текста или Детектора аномалий;
> - сохранение ключа аутентификации в ресурсах Cognitive Services в виде секретов в хранилище Key Vault и настройка доступа для рабочей области Azure Synapse Analytics;
> - создание связанной службы Azure Key Vault в рабочей области Azure Synapse Analytics.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись, прежде чем начинать работу](https://azure.microsoft.com/free/).

## <a name="prerequisites"></a>Предварительные требования

- [Рабочая область Azure Synapse Analytics](../get-started-create-workspace.md) с учетной записью хранения Azure Data Lake Storage 2-го поколения, настроенной в качестве хранилища по умолчанию. При работе с файловой системой Azure Data Lake Storage 2-го поколения вам нужно иметь права *участника данных Хранилища BLOB-объектов*.

## <a name="sign-in-to-the-azure-portal"></a>Вход на портал Azure

Войдите на [портал Azure](https://portal.azure.com/).

## <a name="create-a-cognitive-services-resource"></a>Создание ресурса Cognitive Services

[Azure Cognitive Services](../../cognitive-services/index.yml) содержит множество типов служб. Анализ текста и Детектор аномалий — это два примера, используемые при работе с руководствами по Azure Synapse.

Вы можете создать ресурс [Анализа текста](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics) на портале Azure:

![Снимок экрана: Анализ текста на портале с кнопкой "Создать".](media/tutorial-configure-cognitive-services/tutorial-configure-cognitive-services-00b.png)

Вы можете создать ресурс [Детектора аномалий](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics) на портале Azure:

![Снимок экрана: Детектор аномалий на портале с кнопкой "Создать".](media/tutorial-configure-cognitive-services/tutorial-configure-cognitive-services-00a.png)

## <a name="create-a-key-vault-and-configure-secrets-and-access"></a>Создание хранилища ключей, а также настройка секретов и доступа

1. Создайте [хранилище ключей](https://ms.portal.azure.com/#create/Microsoft.KeyVault) на портале Azure.
2. Выберите **Key Vault** > **Политики доступа** и предоставьте [MSI рабочей области Azure Synapse](../security/synapse-workspace-managed-identity.md) разрешения на чтение секретов из Azure Key Vault.

   > [!NOTE]
   > Убедитесь, что изменения политики сохранены. Этот важный шаг легко пропустить.

   ![Снимок экрана: варианты добавления политики доступа.](media/tutorial-configure-cognitive-services/tutorial-configure-cognitive-services-00c.png)

3. Перейдите к ресурсу Cognitive Services. Например, выберите **Детектор аномалий** > **Ключи и конечная точка**. Затем скопируйте один из двух ключей в буфер обмена.

4. Выберите **Key Vault** > **Секрет**, чтобы создать секрет. Укажите имя секрета, а затем вставьте ключ из предыдущего шага в поле **Значение**. Наконец, выберите **Создать**.

   ![Снимок экрана: варианты создания секрета.](media/tutorial-configure-cognitive-services/tutorial-configure-cognitive-services-00d.png)

   > [!IMPORTANT]
   > Обязательно запомните или запишите имя секрета. Оно будет использоваться позже при подключении к Cognitive Services из Azure Synapse Studio.

## <a name="create-an-azure-key-vault-linked-service-in-azure-synapse"></a>Создание связанной службы Azure Key Vault в Azure Synapse

1. Откройте рабочую область в Azure Synapse Studio. 
2. Выберите **Управление** > **Связанные службы**. Создайте связанную службу **Azure Key Vault**, указывающую на созданный экземпляр Key Vault. 
3. Проверьте подключение, нажав кнопку **Проверить подключение**. Если подключение успешно, выберите **Создать** и **Опубликовать все**, чтобы сохранить изменения.

![Снимок экрана: Azure Key Vault как новая связанная служба.](media/tutorial-configure-cognitive-services/tutorial-configure-cognitive-services-00e.png)

Теперь вы можете продолжить работу с одним из учебников по использованию Azure Cognitive Services в Azure Synapse Studio.

## <a name="next-steps"></a>Дальнейшие действия

- [Учебник. Анализ тональности с помощью Azure Cognitive Services](tutorial-cognitive-services-sentiment.md)
- [Учебник. Обнаружение аномалий с помощью Cognitive Services](tutorial-cognitive-services-sentiment.md)
- [Учебник. Оценка моделей машинного обучения в выделенных пулах Azure Synapse SQL](tutorial-sql-pool-model-scoring-wizard.md).
- [Возможности машинного обучения в Azure Synapse Analytics](what-is-machine-learning.md)