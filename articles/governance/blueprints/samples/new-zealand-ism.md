---
title: Пример схемы для Руководства по мерам информационной безопасности для Новой Зеландии
description: Обзор примера схемы для Руководства по мерам информационной безопасности для Новой Зеландии. Этот пример схемы помогает клиентам оценить определенные средства управления.
ms.date: 03/22/2021
ms.topic: sample
ms.openlocfilehash: a52470ea45e6358007dc49122599e87091fbf8f7
ms.sourcegitcommit: ba3a4d58a17021a922f763095ddc3cf768b11336
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104802983"
---
# <a name="new-zealand-ism-restricted-blueprint-sample"></a>Пример схемы для Руководства по мерам информационной безопасности для Новой Зеландии

Примера схемы для Руководства по мерам информационной безопасности для Новой Зеландии предоставляет защитные средства управления с использованием [Политики Azure](../../policy/overview.md), которые помогут вам оценить конкретные элементы управления, указанные в [Руководстве по мерам информационной безопасности для Новой Зеландии](https://www.nzism.gcsb.govt.nz/). С помощью этой схемы клиенты могут развертывать основной набор политик для любой развернутой в Azure архитектуры, в которой необходимо реализовать элементы управления для Руководства по мерам информационной безопасности для Новой Зеландии.

## <a name="control-mapping"></a>Сопоставление элементов управления

[Сопоставление средств управления службы "Политика Azure"](../../policy/samples/new-zealand-ism.md) позволяет получить подробные сведения об определениях политик, включенных в эту схему, и о сопоставлении этих определений с **элементами управления** в Руководстве по мерам информационной безопасности для Новой Зеландии. Назначаемые архитектуре ресурсы оцениваются службой "Политика Azure" на предмет соответствия назначенным определениям политик. Дополнительные сведения см. в статье [Что такое служба "Политика Azure"?](../../policy/overview.md)

## <a name="deploy"></a>Развертывание

Чтобы развернуть пример схемы Azure Blueprints Руководства по мерам информационной безопасности для Новой Зеландии, необходимо выполнить следующие действия:

> [!div class="checklist"]
> - создание схемы на основе примера;
> - установка метки копии образца **Опубликовано**;
> - назначение копии схемы существующей подписке;

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free), прежде чем начинать работу.

### <a name="create-blueprint-from-sample"></a>Создание схемы на основе примера

Для начала внедрите пример схемы, создав новую схему в среде на основе этого примера.

1. Выберите **Все службы** в левой области. Найдите и выберите пункт **Схемы**.

1. На странице **Начало работы** с левой стороны в разделе _Создание схемы_ щелкните кнопку **Создать**.

1. Найдите пример схемы **Руководства по мерам информационной безопасности для Новой Зеландии** в разделе _Другие примеры_ и выберите **Использовать этот пример**.

1. Введите _основные данные_ образца схемы.

   - **Имя схемы**. Укажите имя для копии примера схемы Руководства по мерам информационной безопасности для Новой Зеландии.
   - **Расположение определения**. Используйте кнопку с многоточием и выберите группу управления, в которой нужно сохранить копию примера.

1. В верхней части страницы выберите вкладку _Артефакты_ или внизу страницы щелкните **Далее: Артефакты**.

1. Просмотрите список артефактов, добавленных в образец схемы. Многие артефакты имеют параметры, которые мы определим позднее. После завершения просмотра образца схемы выберите **Сохранить черновик**.

### <a name="publish-the-sample-copy"></a>Публикации копии образца

Теперь в вашей среде создана копия образца схемы. Она создана в режиме **Черновик**, и прежде чем назначить и развернуть эту копию, ее необходимо **опубликовать**. Копию примера схемы можно настроить в соответствии со своим окружением и потребностями, но такое изменение может нарушить согласование с элементами управления Руководства по мерам информационной безопасности для Новой Зеландии.

1. Выберите **Все службы** в левой области. Найдите и выберите пункт **Схемы**.

1. В меню слева выберите страницу **Определения схем**. Примените фильтры, чтобы найти копию примера схемы, и выберите его.

1. В верхней части страницы выберите **Опубликовать схему**. В правой части новой страницы укажите **версию** для копии примера схемы. Это свойство позволяет вносить изменения позднее. Укажите **сведения об изменениях**, например "Первая версия, опубликованная из примера схемы Руководства по мерам информационной безопасности для Новой Зеландии". Затем щелкните **Опубликовать** в нижней части страницы.

### <a name="assign-the-sample-copy"></a>Назначение копии образца

После успешной **публикации** копию образца схемы можно назначить подписке в группе управления, в которой ее сохранили. На этом шаге указывают параметры, которые позволяют сделать каждое развертывание образца схемы уникальным.

1. Выберите **Все службы** в левой области. Найдите и выберите пункт **Схемы**.

1. В меню слева выберите страницу **Определения схем**. Примените фильтры, чтобы найти копию примера схемы, и выберите его.

1. В верхней части страницы определения схемы выберите **Назначить схему**.

1. Укажите значения параметра для назначения схемы.

   - Основы

     - **Подписки**. Выберите одну или несколько подписок, которые находятся в группе управления, в которой вы сохранили копию образца схемы. Если вы выберите более одной подписки, для каждой из них будет создано назначение с использованием введенных параметров.
     - **Имя назначения.** Имя автоматически заполняется на основе имени схемы.
       Измените его при желании или сохраните вариант по умолчанию.
     - **Расположение.** Выберите регион, в котором будет создано управляемое удостоверение. Azure Blueprint использует это управляемое удостоверение для развертывания всех артефактов в назначенную схему. Дополнительные сведения см. в статье [Управляемые удостоверения для ресурсов Azure](../../../active-directory/managed-identities-azure-resources/overview.md).
     - **Версия определения схемы**. Выберите **опубликованную** версию копии примера схемы.

   - Блокировка назначения

     Выберите режим блокировки для схемы с учетом своей среды. Дополнительные сведения см. в разделе [Блокировка ресурсов схем](../concepts/resource-locking.md).

   - Управляемое удостоверение

     Сохраните значение по умолчанию _Назначено системой_ для управляемого удостоверения.

   - Параметры артефакта

     Параметры, определенные в этом разделе, применяются к артефакту, под которым они определены. Поскольку эти параметры определяются во время назначения схемы, они являются [динамическими](../concepts/parameters.md#dynamic-parameters). Полный список параметров артефактов с описаниями можно найти в [таблице параметров артефактов](#artifact-parameters-table).

1. После ввода всех параметров в нижней части страницы выберите **Назначить**. Это действие создает назначение схемы и начинает развертывание артефактов. Развертывание занимает около часа. Чтобы проверить состояние развертывания, откройте назначение схемы.

> [!WARNING]
> Служба Azure Blueprints и встроенные примеры схем предоставляются **бесплатно**. Ресурсы Azure оплачиваются согласно [тарифам на продукты](https://azure.microsoft.com/pricing/). С помощью [калькулятора цен](https://azure.microsoft.com/pricing/calculator/) вы можете оценить расходы на выполнение ресурсов, развертываемых этим примером схемы.

### <a name="artifact-parameters-table"></a>Таблица параметров артефактов

Следующая таблица содержит полный список параметров артефактов схемы.

|Имя артефакта|Тип артефакта|Имя параметра|Описание|
|-|-|-|-|
|ISM Restricted (Новая Зеландия)|Назначение политики|Список пользователей, которых нужно включить в группу администраторов виртуальных машин Windows|Список пользователей с разделителем точка с запятой, которых нужно добавить в локальную группу администраторов. Administrator; myUser1; myUser2|
|ISM Restricted (Новая Зеландия)|Назначение политики|Список пользователей, которых необходимо исключить из группы администраторов виртуальных машин Windows|Список пользователей с разделителем точка с запятой, которых нужно исключить из локальной группы администраторов. Administrator; myUser1; myUser2|
|ISM Restricted (Новая Зеландия)|Назначение политики|Список пользователей, которые одни должны войти в группу администраторов виртуальных машин Windows|Разделенный точками с запятой список всех ожидаемых членов локальной группы "Администраторы"; например, администратор; мой_пользователь1; мой_пользователь2|
|ISM Restricted (Новая Зеландия)|Назначение политики|Идентификатор рабочей области Log Analytics для отчетов агента виртуальных машин|Идентификатор (GUID) рабочей области Log Analytics, в которую агенты виртуальных машин должны передавать данные|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: для Azure Front Door Service должен быть включен Брандмауэр веб-приложений (WAF)|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: параметры оценки уязвимостей для сервера SQL должны включать адрес электронной почты для получения отчетов о проверке|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: на виртуальных машинах с выходом в Интернет должны применяться рекомендации по адаптивной защите сети|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: Подписке должно быть назначено несколько владельцев.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: Шифрование дисков должно применяться к виртуальным машинам.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: Удаленная отладка должна быть отключена для приложений-функций.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: Брандмауэр веб-приложений (WAF) должен использовать указанный режим для Шлюза приложений|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Требование режима WAF для Шлюза приложений|В службе "Шлюз приложений" необходимо включить режим предотвращения или обнаружения|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: Прозрачное шифрование данных должно быть включено в базах данных SQL.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: для Управляемого экземпляра SQL должна быть включена оценка уязвимостей|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Необязательно: список пользовательских образов виртуальных машин с поддерживаемой ОС Windows, которые добавляются в область дополнительно к образам коллекции для политики: развертывание — Настройка агента зависимостей, который будет включен в виртуальных машинах Windows|Дополнительные сведения о гостевой конфигурации см. на странице [https://aka.ms/gcpol](https://aka.ms/gcpol)|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: Администратор Azure Active Directory должен быть подготовлен для серверов SQL Server.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: должны быть включены только защищенные подключения к Кэшу Azure для Redis|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: Решение защиты конечных точек должно быть установлено в масштабируемых наборах виртуальных машин.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Включить серверы, подключенные к Arc, при оценке политики: аудит компьютеров с Windows без любого из указанных участников в группе администраторов|Выбирая значение true, вы соглашаетесь на ежемесячное взимание платы за каждый подключенный к Arc компьютер|
|ISM Restricted (Новая Зеландия)|Назначение политики|Необязательно: список образов пользовательских виртуальных машин с поддерживаемой ОС Windows, которые добавляются в область дополнительно к образам в коллекции для политики: [предварительная версия]: агент Log Analytics должен быть включен для перечисленных образов виртуальных машин|Дополнительные сведения о гостевой конфигурации см. на странице [https://aka.ms/gcpol](https://aka.ms/gcpol)|
|ISM Restricted (Новая Зеландия)|Назначение политики|Необязательно: список образов пользовательских виртуальных машин с поддерживаемой ОС Linux, которые добавляются в область дополнительно к образам в коллекции для политики: [предварительная версия]: агент Log Analytics должен быть включен для перечисленных образов виртуальных машин|Дополнительные сведения о гостевой конфигурации см. на странице [https://aka.ms/gcpol](https://aka.ms/gcpol)|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: учетные записи хранения должны ограничивать сетевой доступ|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Необязательно: список пользовательских образов виртуальных машин с поддерживаемой ОС Windows, которые добавляются в область дополнительно к образам коллекции для политики: развертывание — настройка агента зависимостей, который будет включен в масштабируемых наборах виртуальных машин Windows|Дополнительные сведения о гостевой конфигурации см. на странице [https://aka.ms/gcpol](https://aka.ms/gcpol)|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: Уязвимости в конфигурации безопасности в масштабируемом наборе виртуальных машин должны быть устранены.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Включить серверы, подключенные к Arc, при оценке политики: аудит компьютеров с ОС Windows с лишними учетными записями в группе администраторов|Выбирая значение true, вы соглашаетесь на ежемесячное взимание платы за каждый подключенный к Arc компьютер|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: Должно выполняться безопасное перемещение в учетные записи хранения.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: Брандмауэр веб-приложений (WAF) должен использовать указанный режим для Azure Front Door Service|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Требование режима WAF для Azure Front Door Service|В службе Azure Front Door необходимо включить режим предотвращения или обнаружения|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: на компьютерах должны быть включены адаптивные элементы управления приложениями для определения безопасных приложений|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: Подписке можно назначить не более 3 владельцев.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект для политики: [предварительная версия]: открытый доступ к учетной записи хранения должен быть запрещен|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: необходимо включить решение для оценки уязвимостей на виртуальных машинах|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: для Шлюза приложений должен быть включен Брандмауэр веб-приложений (WAF)|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: Функция CORS не должна разрешать всем ресурсам доступ к вашим веб-приложениям.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Включить серверы, подключенные к Arc, при оценке политики: аудит веб-серверов Windows без безопасных протоколов связи|Выбирая значение true, вы соглашаетесь на ежемесячное взимание платы за каждый подключенный к Arc компьютер|
|ISM Restricted (Новая Зеландия)|Назначение политики|Минимальная версия TLS для веб-серверов Windows|Веб-серверы Windows с более ранними версиями TLS будут оцениваться как не соответствующие требованиям|
|ISM Restricted (Новая Зеландия)|Назначение политики|Необязательно: список образов пользовательских виртуальных машин с поддерживаемой ОС Linux, которые добавляются в область дополнительно к образам в коллекции для политики: агент Log Analytics должен быть включен в масштабируемых наборах виртуальных машин для перечисленных образов виртуальных машин|Дополнительные сведения о гостевой конфигурации см. на странице [https://aka.ms/gcpol](https://aka.ms/gcpol)|
|ISM Restricted (Новая Зеландия)|Назначение политики|Необязательно: список образов пользовательских виртуальных машин с поддерживаемой ОС Windows, которые добавляются в область дополнительно к образам в коллекции для политики: агент Log Analytics должен быть включен в масштабируемых наборах виртуальных машин для перечисленных образов виртуальных машин|Дополнительные сведения о гостевой конфигурации см. на странице [https://aka.ms/gcpol](https://aka.ms/gcpol)|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: Внешние учетные записи с разрешениями на запись следует удалять из подписки.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Включить серверы, подключенные к Arc, при оценке политики: аудит компьютеров с Windows с заданными участниками в группе администраторов|Выбирая значение true, вы соглашаетесь на ежемесячное взимание платы за каждый подключенный к Arc компьютер|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: Устаревшие учетные записи следует удалять из подписки.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: Приложение-функция должно быть доступно только по HTTPS|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: подписки Azure должны иметь профиль журнала для журнала действий|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Список типов ресурсов, для которых должны быть включены журналы диагностики||
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: На ваших компьютерах должны быть установлены обновления системы|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: В вашем приложении API необходимо использовать последнюю версию TLS|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: MFA должна быть включена для учетных записей с разрешениями на запись в вашей подписке.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: На серверах Windows должно быть развернуто расширение Microsoft IaaSAntimalware.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: Веб-приложение должно быть доступно только по HTTPS|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: Защита от атак DDoS Azure уровня "Стандартный"должна быть включена|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: MFA должна быть включена для учетных записей с разрешениями владельца в вашей подписке.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: Расширенная защита данных должна быть включена на серверах SQL Server.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: для Управляемого экземпляра SQL должна быть включена Расширенная защита данных|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: Мониторинг отсутствия Endpoint Protection в Центре безопасности Azure|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: журнал действий должен храниться по меньшей мере один год|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: порты управления виртуальных машин должны быть защищены с помощью JIT-управления доступом к сети|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: Кластеры Service Fabric должны использовать для проверки подлинности клиента только Azure Active Directory.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: Приложение API должно быть доступно только по HTTPS.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: аудит компьютеров с Windows без включения Exploit Guard в Microsoft Defender|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Включить серверы, подключенные к Arc, при оценке политики: аудит компьютеров с Windows без включения Exploit Guard в Microsoft Defender|Выбирая значение true, вы соглашаетесь на ежемесячное взимание платы за каждый подключенный к Arc компьютер|
|ISM Restricted (Новая Зеландия)|Назначение политики|Состояние соответствия для компьютеров с Windows, на которых Exploit Guard в Защитнике Windows недоступен|Exploit Guard в Защитнике Windows доступен, только начиная с Windows 10 или Windows Server с обновлением 1709. Если задать для этого параметра значение "Не соответствует", отобразятся компьютеры с более старыми версиями, на которых Exploit Guard в Защитнике Windows будет недоступен (например, Windows Server 2012 R2). Если задать для этого параметра значение "Соответствует", компьютеры будут отображаться как соответствующие.|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: Обновления системы должны быть установлены в масштабируемых наборах виртуальных машин.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: Удаленная отладка должна быть отключена для веб-приложений.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: Уязвимости в настройках безопасности на ваших компьютерах должны быть устранены|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: MFA должна быть включена для учетных записей с разрешениями на чтение в вашей подписке.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: Необходимо устранить уязвимости в конфигурациях безопасности контейнера.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: Удаленная отладка должна быть отключена для приложений API.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: аудит компьютеров с Linux с разрешенным удаленным подключением для учетных записей без паролей|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Включить серверы, подключенные к Arc, при оценке политики: аудит компьютеров с Linux с разрешенным удаленным подключением для учетных записей без паролей|Выбирая значение true, вы соглашаетесь на ежемесячное взимание платы за каждый подключенный к Arc компьютер|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: Устаревшие учетные записи с разрешениями владельца следует удалять из подписки.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: На серверах SQL Server должна быть включена оценка уязвимости.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: В вашем веб-приложении необходимо использовать последнюю версию TLS|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект для политики: компьютеры под управлением ОС Windows должны соответствовать требованиям для категории "Параметры безопасности — Политики учетных записей"|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Вести журнал паролей для локальных учетных записей виртуальной машины Windows|Указывает на ограничения касательно повторного использования пароля, то есть число обязательных процессов создания нового пароля для учетной записи пользователя, прежде чем пользователь сможет использовать уже созданный пароль|
|ISM Restricted (Новая Зеландия)|Назначение политики|Включить серверы, подключенные к Arc, при оценке политики: компьютеры под управлением ОС Windows должны соответствовать требованиям для категории "Параметры безопасности — Политики учетных записей"|Выбирая значение true, вы соглашаетесь на ежемесячное взимание платы за каждый подключенный к Arc компьютер|
|ISM Restricted (Новая Зеландия)|Назначение политики|Максимальный срок действия пароля для локальных учетных записей виртуальной машины Windows|Задает максимальное число дней, по истечении которых требуется изменить пароль учетной записи пользователя. Формат значения — два целых числа через запятую, показывающие крайние числа, входящие в диапазон|
|ISM Restricted (Новая Зеландия)|Назначение политики|Минимальный срок действия пароля для локальных учетных записей в виртуальной машине Windows|Указывает минимальное число дней, по истечении которых пользователь сможет изменить пароль учетной записи|
|ISM Restricted (Новая Зеландия)|Назначение политики|Минимальная длина пароля для локальных учетных записей виртуальной машины Windows|Указывает минимальное число символов, которые может содержать пароль учетной записи пользователя|
|ISM Restricted (Новая Зеландия)|Назначение политики|Пароль должен соответствовать требованиям сложности для локальных учетных записей виртуальной машины Windows|Указывает, требуется ли сложный пароль для учетной записи пользователя. Сложный пароль не должен включать имя учетной записи (полностью или частично), и в нем должно быть не меньше 6 символов с обязательным наличием строчных и прописных букв, цифр и неалфавитных знаков|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: виртуальные машины с выходом в Интернет должны быть защищены с помощью групп безопасности сети|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: аудит компьютеров с Linux с учетными записями без паролей|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Включить серверы, подключенные к Arc, при оценке политики: аудит компьютеров Linux с учетными записями без паролей|Выбирая значение true, вы соглашаетесь на ежемесячное взимание платы за каждый подключенный к Arc компьютер|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: Внешние учетные записи с разрешениями владельца следует удалять из подписки.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: В вашем приложении-функции необходимо использовать последнюю версию TLS|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект для политики: [предварительная версия]: весь интернет-трафик должен направляться через развернутый Брандмауэр Azure|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|
|ISM Restricted (Новая Зеландия)|Назначение политики|Эффект политики: Уязвимости в базах данных SQL должны быть устранены.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](https://aka.ms/policyeffects).|

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные статьи о схемах и способах их использования:

- Ознакомьтесь со сведениями о [жизненном цикле схем](../concepts/lifecycle.md).
- Узнайте, как использовать [статические и динамические параметры](../concepts/parameters.md).
- Научитесь настраивать [последовательность схемы](../concepts/sequencing-order.md).
- Узнайте, как применять [блокировку ресурсов схемы](../concepts/resource-locking.md).
- Узнайте, как [обновлять существующие назначения](../how-to/update-existing-assignments.md).