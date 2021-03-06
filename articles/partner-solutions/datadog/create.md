---
title: Создание Datadog — решения партнеров Azure
description: В этой статье описано, как с помощью портала Azure создать экземпляр Datadog.
ms.service: partner-services
ms.topic: quickstart
ms.date: 02/19/2021
author: tfitzmac
ms.author: tomfitz
ms.custom: references_regions
ms.openlocfilehash: 7af8b82c5da6c60527b45b6e8e292b9f067016ec
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101747812"
---
# <a name="quickstart-get-started-with-datadog"></a>Краткое руководство. Начало работы с Datadog

В этом кратком руководстве показано, как создать экземпляр Datadog. Для этого можно создать новую организацию Datadog или использовать ссылку на существующую.

## <a name="pre-requisites"></a>Предварительные требования

Чтобы настроить интеграцию с Azure Datadog, вам нужны права **владельца** в подписке Azure. Перед началом настройки убедитесь, что у вас есть необходимый доступ.

Чтобы настроить для ресурса Datadog функцию единого входа, используя SAML (язык разметки заявлений системы безопасности), необходимо настроить корпоративное приложение. Чтобы добавить корпоративное приложение, нужно иметь одну из следующих ролей: глобальный администратор, администратор облачных приложений, администратор приложения или владелец субъекта-службы.

Выполните следующую процедуру, чтобы настроить корпоративное приложение:

1. Перейдите на [портал Azure](https://portal.azure.com). Выберите **Azure Active Directory**.
1. В области слева выберите **Корпоративные приложения**.
1. Щелкните **Новое приложение**.
1. В поле **Добавить из коллекции** введите *Datadog* для поиска. Выберите результат поиска и щелкните **Добавить**.

   :::image type="content" source="media/create/datadog-azure-ad-app-gallery.png" alt-text="Приложение Datadog в корпоративной галерее AAD." border="true":::

1. Когда создание приложения завершится, перейдите в раздел свойств на боковой панели. Для параметра **Требуется ли назн. польз.?** задайте значение **Нет**, а затем щелкните **Сохранить**.

   :::image type="content" source="media/create/user-assignment-required.png" alt-text="Настройка свойств для приложения Datadog" border="true":::

1. Выберите **Единый вход** на боковой панели. Затем выберите **SAML**.

   :::image type="content" source="media/create/saml-sso.png" alt-text="Проверка подлинности SAML." border="true":::

1. Выберите **Да** в ответ на запрос **Сохранение параметров единого входа**.

   :::image type="content" source="media/create/save-sso.png" alt-text="Сохранение параметров единого входа для приложения Datadog." border="true":::

1. Настройка единого входа завершена.

## <a name="find-offer"></a>Поиск предложения

На портале Azure найдите приложение Datadog.

1. Перейдите на [портал Azure](https://portal.azure.com/) и войдите в систему.

1. Если вы уже посещали **Marketplace** в текущем сеансе, выберите значок из списка доступных вариантов. Если нет, выполните поиск по строке _Marketplace_.

    :::image type="content" source="media/create/marketplace.png" alt-text="Значок Marketplace.":::

1. В Marketplace выполните поиск по строке **Datadog**.

1. На экране обзора плана щелкните **Настройка и подписка**.

   :::image type="content" source="media/create/datadog-app.png" alt-text="Приложение Datadog в Azure Marketplace.":::

## <a name="create-a-datadog-resource-in-azure"></a>Создание ресурса Datadog в Azure

На портале отобразится форма для создания ресурса Datadog.

:::image type="content" source="media/create/datadog-create-resource.png" alt-text="Создание ресурса Datadog." border="true":::

Укажите здесь следующие значения.

|Свойство | Описание
|:-----------|:-------- |
| Подписка | Выберите подписку Azure, в которой вы хотите создать ресурс Datadog. Необходимо иметь доступ владельца. |
| Группа ресурсов | Укажите, следует ли создать новую группу ресурсов или использовать имеющуюся. [Группа ресурсов](../../azure-resource-manager/management/overview.md#resource-groups) — это контейнер, содержащий связанные ресурсы для решения Azure. |
| Имя ресурса | Укажите имя для ресурса Datadog. Это имя будет использоваться как имя новой организации Datadog при ее создании. |
| Расположение | Выберите Западная часть США 2. В настоящее время поддерживается только регион "Западная часть США 2". |
| Организация Datadog | Чтобы создать новую организацию Datadog, щелкните **Новая**. Чтобы создать ссылку на существующую организацию Datadog, щелкните **Существующая**. |
| Ценовой план | При создании новой организации выберите в списке доступный план Datadog. |
| Срок выставления счетов | Ежемесячно. |

Если вы создаете ссылку на существующую организацию Datadog, переходите к следующему разделу. В противном случае выберите **Далее: метрики и журналы** и пропустите следующий раздел.

## <a name="link-to-existing-datadog-organization"></a>Создание связи с существующей организацией Datadog

Вы можете связать новый ресурс Datadog в Azure с существующей организацией Datadog.

В области выбора организации Datadog щелкните **Существующая** и щелкните **Link to Datadog org** (Связать с организацией Datadog).

:::image type="content" source="media/create/link-to-existing.png" alt-text="Создание связи с существующей организацией Datadog." border="true":::

Эта ссылка открывает окно проверки подлинности в Datadog. Войдите в Datadog.

По умолчанию Azure связывает с ресурсом Datadog текущую организацию Datadog. Если вы хотите создать связь с другой организацией, выберите нужную организацию в окне проверки подлинности, как показано ниже.

:::image type="content" source="media/create/select-datadog-organization.png" alt-text="Выбор организации Datadog для создания связи." border="true":::

## <a name="configure-metrics-and-logs"></a>Настройка метрик и журналов

Используйте теги ресурсов Azure настройте метрики и журналы, которые будут отправляться в Datadog. Вы можете включать или исключать метрики и журналы конкретных ресурсов.

Для отправки **метрик** используются следующие правила тегов:

- По умолчанию собираются метрики для всех ресурсов, за исключением виртуальных машин, масштабируемых наборов виртуальных машин и планов службы приложений.
- Виртуальные машины, масштабируемые наборы виртуальных машин и планы службы приложений с тегами *Include* отправляют метрики в Datadog.
- Виртуальные машины, масштабируемые наборы виртуальных машин и планы службы приложений с тегами *Exclude* не отправляют метрики в Datadog.
- Если возникает конфликт между правилами включения и исключения, приоритет имеет исключение.

Для отправки **журналов** используются следующие правила тегов:

- По умолчанию журналы собираются для всех ресурсов.
- Ресурсы Azure с тегами *Include* отправляют журналы в Datadog.
- Ресурсы Azure с тегами *Exclude* не отправляют журналы в Datadog.
- Если возникает конфликт между правилами включения и исключения, приоритет имеет исключение.

На следующем снимке экрана показан пример правила тегов: в Datadog отправляются только метрики виртуальных машин, масштабируемых наборов виртуальных машин и планов службы приложений с тегом *Datadog=True*.

:::image type="content" source="media/create/config-metrics-logs.png" alt-text="Настройка журналов и метрик." border="true":::

Azure может передавать в Datadog журналы двух типов.

1. **Журналы уровня подписки** предоставляют сведения о текущей работе ресурсов на уровне [плоскости управления](../../azure-resource-manager/management/control-plane-and-data-plane.md). Также включаются сведения о событиях работоспособности служб. Журнал действий позволяет определить, кто, что и когда выполнял для любых операций записи (PUT, POST, DELETE). Для каждой подписки Azure существует один журнал действий.

1. **Журналы ресурсов Azure** предоставляют сведения о действиях, выполняемых с ресурсами Azure на уровне [плоскости данных](../../azure-resource-manager/management/control-plane-and-data-plane.md). Например, получение секрета из Key Vault является операцией плоскости данных. Запрос к базе данных также считается операцией плоскости данных. Содержимое журналов ресурсов зависит от типа ресурса и конкретной службы Azure.

Чтобы отправлять в Datadog журналы уровня подписки, установите флажок **Send subscription activity logs** (Отправлять журналы действий для подписки). Если этот флажок не установлен, ни один из журналов уровня подписки не будет отправляться в Datadog.

Чтобы отправлять в Datadog журналы ресурсов Azure, установите флажок **Send Azure resource logs for all defined resources** (Отправлять журналы ресурсов Azure для всех определенных ресурсов). Типы журналов ресурсов Azure перечислены на странице с [категориями журнала ресурсов Azure Monitor](../../azure-monitor/essentials/resource-logs-categories.md).  Чтобы отфильтровать набор ресурсов Azure, отправляющих журналы в Datadog, используйте теги ресурсов Azure.  

За журналы, отправляемые в Datadog, в Azure взимается плата. Дополнительные сведения см. в разделе с [ценами на журналы платформы](https://azure.microsoft.com/pricing/details/monitor/), отправляемые партнерам Azure Marketplace.

Завершив настройку метрик и журналов, щелкните **Далее: единый вход**.

## <a name="configure-single-sign-on"></a>Настройка единого входа

Если ваша организация использует в качестве поставщика удостоверений Azure Active Directory, вы можете настроить единый вход с портала Azure в Datadog. Если в организации используется другой поставщик удостоверений или вас пока не интересует единый вход, этот раздел можно пропустить.

Если вы связываете ресурс Datadog с существующей организацией Datadog, на этом этапе вы не сможете настроить единый вход. Вместо этого единый вход следует настраивать после создания ресурса Datadog. Дополнительные сведения см. в разделе [Повторная настройка единого входа](manage.md#reconfigure-single-sign-on).

:::image type="content" source="media/create/linking-sso.png" alt-text="Единый вход при настройке связи с существующей организацией Datadog." border="true":::

Чтобы настроить единый вход через Azure Active Directory, установите флажок **Включить единый вход через Azure Active Directory**.

Портал Azure извлекает соответствующее приложение Datadog из каталога Azure Active Directory. Это то же самое корпоративное приложение, которое вы указали на предыдущем шаге.

Выберите имя приложения Datadog.

:::image type="content" source="media/create/sso.png" alt-text="Включение единого входа в Datadog." border="true":::

По завершении выберите **Next: Теги**.

## <a name="add-custom-tags"></a>Добавьте пользовательские теги.

Для нового ресурса Datadog можно указать пользовательские теги. Укажите теги, применяемые к ресурсу Datadog, в формате пар "имя — значение".

:::image type="content" source="media/create/tags.png" alt-text="Добавление пользовательских тегов к ресурсу Datadog." border="true":::

Завершив добавление тегов, щелкните **Далее: проверка и создание**.

## <a name="review--create-datadog-resource"></a>Проверка и создание ресурса Datadog

Проверьте выбранные параметры и условия использования. После завершения проверки щелкните **Создать**.

:::image type="content" source="media/create/review-create.png" alt-text="Проверка и создание ресурса Datadog." border="true":::

Теперь Azure развернет ресурс Datadog.

Когда этот процесс завершится, щелкните **Переход к ресурсу** и убедитесь, что ресурс Datadog создан.

:::image type="content" source="media/create/go-to-resource.png" alt-text="Развертывание ресурса Datadog." border="true":::

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Управление ресурсом Datadog](manage.md)
