---
title: Подключение Фабрики данных к Azure Purview
description: Узнайте, как подключить фабрику данных к Azure зрения
ms.author: lle
author: lrtoyou1223
ms.service: data-factory
ms.topic: conceptual
ms.custom:
- seo-lt-2019
- references_regions
ms.date: 12/3/2020
ms.openlocfilehash: 44f093f96d0f4653a6fcca94aaa97264c93e3c7d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101727946"
---
# <a name="connect-data-factory-to-azure-purview-preview"></a>Подключение фабрики данных к Azure зрения (Предварительная версия)
[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

В этой статье объясняется, как подключить фабрику данных к Azure зрения и как сообщить о преобразовании данных из действий фабрики данных Azure копирование данных, поток данных и выполнение пакета служб SSIS.


## <a name="connect-data-factory-to-azure-purview"></a>Подключение фабрики данных к Azure зрения
Azure зрения — это новая облачная служба, которая используется пользователями данных для централизованного управления данными, охватывающими облачные и локальные среды. Вы можете подключить фабрику данных к Azure зрения, и подключение позволяет использовать зрения Azure для сбора данных журнала обращений и преобразований для копирования, потока данных и выполнения пакета служб SSIS. Существует два способа подключения фабрики данных к Azure зрения.
### <a name="register-azure-purview-account-to-data-factory"></a>Регистрация учетной записи Azure зрения в фабрике данных
1. На портале ADF перейдите к разделу **Управление**  ->  **Azure зрения**. Выберите команду **Connect to a Purview account** (Подключиться к учетной записи Purview). 

:::image type="content" source="./media/data-factory-purview/register-purview-account.png" alt-text="Снимок экрана регистрации учетной записи зрения.":::
2. Вы можете выбрать вариант **From Azure subscription** (Из подписки Azure) или **Ввести вручную**. **В подписке Azure** можно выбрать учетную запись, к которой у вас есть доступ. 
3. После подключения вы сможете увидеть имя учетной записи зрения в **учетной записи зрения** вкладки. 
4. Для поиска данных можно использовать панель поиска в верхнем центре портала фабрики данных Azure. 

Если вы видите предупреждение на портале фабрики данных Azure после регистрации учетной записи Azure зрения в фабрике данных, выполните действия ниже, чтобы устранить проблему.

:::image type="content" source="./media/data-factory-purview/register-purview-account-warning.png" alt-text="Снимок экрана с предупреждением о регистрации учетной записи зрения.":::

1. Перейдите в портал Azure и найдите свою фабрику данных. Выберите раздел "Теги" и проверьте наличие тега с именем **каталогури**. Если нет, отключите и повторно подключите учетную запись Azure зрения на портале ADF.

:::image type="content" source="./media/data-factory-purview/register-purview-account-tag.png" alt-text="Снимок экрана для тегов регистрации учетной записи зрения.":::

2. Проверьте, предоставлено ли разрешение на регистрацию учетной записи Azure зрения в фабрике данных. См. статью [Подключение фабрики данных Azure и Azure зрения](../purview/how-to-link-azure-data-factory.md#create-new-data-factory-connection) .

### <a name="register-data-factory-in-azure-purview"></a>Регистрация фабрики данных в Azure зрения
Сведения о регистрации фабрики данных в Azure зрения см. в статье [Подключение фабрики данных Azure и Azure зрения](../purview/how-to-link-azure-data-factory.md). 

## <a name="report-lineage-data-to-azure-purview"></a>Отчет о данных журнала преобразований в Azure зрения
Когда клиенты выполняют операции копирования, потока данных или выполнения пакетов служб SSIS в фабрике данных Azure, клиенты могут получить отношение зависимости и иметь общий обзор всего процесса рабочего процесса между источниками данных и назначением.
Сведения о том, как получить журнал обращений и преобразований из фабрики данных Azure, см. в разделе [Журнал преобразований](../purview/how-to-link-azure-data-factory.md#supported-azure-data-factory-activities)

## <a name="next-steps"></a>Дальнейшие действия
[Пользовательская структура журнала преобразований каталога](../purview/catalog-lineage-user-guide.md)

[Руководство. Отправка данных журнала преобразований в фабрике данных в Azure зрения](turorial-push-lineage-to-purview.md)