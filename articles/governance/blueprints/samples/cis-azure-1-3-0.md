---
title: Пример схемы CIS Microsoft Azure Foundations Benchmark версии 1.3.0
description: Общие сведения о примере схемы CIS Microsoft Azure Foundations Benchmark версии 1.3.0. Этот пример схемы помогает клиентам оценить определенные средства управления.
ms.date: 03/11/2021
ms.topic: sample
ms.openlocfilehash: 7030b4e9e9ddd3d5bbf1a5a7deaf1e1aa241b38e
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105566002"
---
# <a name="cis-microsoft-azure-foundations-benchmark-v130-blueprint-sample"></a>Пример схемы CIS Microsoft Azure Foundations Benchmark версии 1.3.0

В примере схемы CIS Microsoft Azure Foundations Benchmark версии 1.3.0, используемом со службой [Политика Azure](../../policy/overview.md), предоставляются средства для обеспечения соответствия требованиям, чтобы вы могли оценить рекомендации этого теста. Схема помогает клиентам развернуть основной набор политик для любой развернутой в Azure архитектуры, в которой необходимо реализовать рекомендации согласно CIS Microsoft Azure Foundations Benchmark версии 1.3.0.

## <a name="recommendation-mapping"></a>Сопоставление рекомендаций

[Сопоставление рекомендаций службы "Политика Azure"](../../policy/samples/cis-azure-1-3-0.md) позволяет получить подробные сведения об определениях политик, включенных в эту схему, и о сопоставлении этих определений с **рекомендациями** в CIS Microsoft Azure Foundations Benchmark версии 1.3.0. Назначаемые архитектуре ресурсы оцениваются службой "Политика Azure" на предмет соответствия назначенным определениям политик. Дополнительные сведения см. в статье [Что такое служба "Политика Azure"?](../../policy/overview.md)

## <a name="deploy"></a>Развертывание

Чтобы развернуть пример схемы CIS Microsoft Azure Foundations Benchmark версии 1.3.0 в Azure Blueprints, необходимо сделать следующее:

> [!div class="checklist"]
> - создание схемы на основе примера;
> - установка метки копии образца **Опубликовано**;
> - назначение копии схемы существующей подписке;

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free), прежде чем начинать работу.

### <a name="create-blueprint-from-sample"></a>Создание схемы на основе примера

Для начала внедрите пример схемы, создав новую схему в среде на основе этого примера.

1. Выберите **Все службы** в левой области. Найдите и выберите пункт **Схемы**.

1. На странице **Начало работы** с левой стороны в разделе _Создание схемы_ щелкните кнопку **Создать**.

1. Найдите пример схемы **CIS Microsoft Azure Foundations Benchmark v1.3.0** в разделе _Другие примеры_ и выберите **Использовать этот пример**.

1. Введите _основные данные_ образца схемы.

   - **Имя схемы**. Укажите имя для своей копии схемы CIS Microsoft Azure Foundations Benchmark.
   - **Расположение определения**. Используйте кнопку с многоточием и выберите группу управления, в которой нужно сохранить копию примера.

1. В верхней части страницы выберите вкладку _Артефакты_ или внизу страницы щелкните **Далее: Артефакты**.

1. Просмотрите список артефактов, добавленных в образец схемы. Многие артефакты имеют параметры, которые мы определим позднее. После завершения просмотра образца схемы выберите **Сохранить черновик**.

### <a name="publish-the-sample-copy"></a>Публикации копии образца

Теперь в вашей среде создана копия образца схемы. Она создана в режиме **Черновик**, и прежде чем назначить и развернуть эту копию, ее необходимо **опубликовать**. Вы можете изменить параметры своей копии этого примера схемы с учетом среды и требований, но такие изменения могут нарушить соответствие рекомендациям CIS Microsoft Azure Foundations Benchmark версии 1.3.0.

1. Выберите **Все службы** в левой области. Найдите и выберите пункт **Схемы**.

1. В меню слева выберите страницу **Определения схем**. Примените фильтры, чтобы найти копию примера схемы, и выберите его.

1. В верхней части страницы выберите **Опубликовать схему**. В правой части новой страницы укажите **версию** для копии примера схемы. Это свойство позволяет вносить изменения позднее. Укажите **сведения об изменениях**, например "Первая версия, опубликованная из примера схемы CIS Microsoft Azure Foundations Benchmark". Затем щелкните **Опубликовать** в нижней части страницы.

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
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Список утвержденных для использования расширений виртуальной машины|Разделенный точками с запятой список расширений виртуальных машин. Для просмотра полного списка используйте команду Get-AzVMExtensionImage из Azure PowerShell.|
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: управляемые экземпляры SQL должны использовать управляемые клиентом ключи для шифрования неактивных данных|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: параметры оценки уязвимостей для сервера SQL должны включать адрес электронной почты для получения отчетов о проверке|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: необходимо включить журналы диагностики в Azure Data Lake Storage|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: Шифрование дисков должно применяться к виртуальным машинам.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: в хранилище ключей должна быть включена защита от очистки|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: параметр "Сертификаты клиента (входящие сертификаты клиента)" для приложения API должен иметь значение "Вкл"|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: серверы SQL должны использовать управляемые клиентом ключи для шифрования неактивных данных|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: в вашем приложении-функции необходимо использовать управляемое удостоверение|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: необходимо включить Azure Defender для Key Vault|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: роли владельца настраиваемой подписки не должны существовать|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: для ключей должны быть заданы даты окончания срока действия|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: Прозрачное шифрование данных должно быть включено в базах данных SQL.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: для Управляемого экземпляра SQL должна быть включена оценка уязвимостей|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: в приложении API должна использоваться последняя версия PHP|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: Администратор Azure Active Directory должен быть подготовлен для серверов SQL Server.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: необходимо включить Azure Defender для Службы приложений|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: учетные записи хранения должны ограничивать доступ к сети, используя правила виртуальной сети|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: в вашем веб-приложении необходимо использовать управляемое удостоверение|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: доступ из Интернета по протоколу SSH должен быть заблокирован|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: неподключенные диски должны быть зашифрованы|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: необходимо включить Azure Defender для службы хранилища|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: учетные записи хранения должны ограничивать сетевой доступ|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: в Logic Apps должны быть включены журналы диагностики|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: в Центре Интернета вещей должны быть включены журналы диагностики|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: в вашем приложении-функции необходимо принудительно использовать только FTPS|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: для определенных операций безопасности должно существовать оповещение журнала действий (Microsoft.Security/securitySolutions/delete)|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: для определенных операций безопасности должно существовать оповещение журнала действий (Microsoft.Security/securitySolutions/write)|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: Должно выполняться безопасное перемещение в учетные записи хранения.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: в учетных записях пакетной службы должны быть включены журналы диагностики|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: в подписке должна быть включена автоматическая подготовка агента Log Analytics|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: В веб-приложении должна использоваться последняя версия Java|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: в вашем веб-приложении необходимо принудительно использовать FTPS|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: необходимо включить Azure Defender для серверов|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: подписки должны включать адрес электронной почты контактного лица по проблемам безопасности|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: открытый доступ к учетной записи хранения должен быть запрещен|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: необходимо включить Azure Defender для Kubernetes|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: для серверов баз данных PostgreSQL должно быть включено регулирование подключения|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: параметр "Сертификаты клиента (входящие сертификаты клиента)" для веб-приложения должен иметь значение "Вкл"|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: Внешние учетные записи с разрешениями на запись следует удалять из подписки.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: Внешние учетные записи с разрешениями на чтение следует удалять из подписки.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: необходимо включить Azure Defender для серверов SQL Server на компьютерах|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: для оповещений высокого уровня серьезности должны быть включены уведомления по электронной почте|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: учетная запись хранения должна использовать управляемый клиентом ключ шифрования|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: В веб-приложении должна использоваться последняя версия Python|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: В приложении-функции должна использоваться последняя версия Python|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: В веб-приложении должна использоваться последняя версия PHP|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: в приложении API должна использоваться последняя версия Python|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: в Масштабируемых наборах виртуальных машин должны быть включены журналы диагностики|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: необходимо включить Azure Defender для серверов Базы данных SQL Azure|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: в Центре событий должны быть включены журналы диагностики|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: На ваших компьютерах должны быть установлены обновления системы|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: в приложении API должна использоваться последняя версия Java|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: на серверах SQL должен быть настроен период хранения данных аудита более 90 дней|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: В веб-приложении должна использоваться последняя версия HTTP|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: В вашем приложении API необходимо использовать последнюю версию TLS|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: MFA должна быть включена для учетных записей с разрешениями на запись в вашей подписке.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: должна быть включена проверка подлинности для веб-приложения|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: для секретов должны быть заданы даты окончания срока действия|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: в приложении API должна использоваться последняя версия HTTP|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: в вашем приложении API необходимо принудительно использовать только FTPS|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: В приложении-функции должна использоваться последняя версия Java|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: Веб-приложение должно быть доступно только по HTTPS|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: Необходимо включить аудит на сервере SQL.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: MFA должна быть включена для учетных записей с разрешениями владельца в вашей подписке.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: Расширенная защита данных должна быть включена на серверах SQL Server.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: для Управляемого экземпляра SQL должна быть включена Расширенная защита данных|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: в службах Kubernetes следует использовать управление доступом на основе ролей (RBAC)|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: Мониторинг отсутствия Endpoint Protection в Центре безопасности Azure|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: в службах "Поиск" должны быть включены журналы диагностики|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: в службах приложений должны быть включены журналы диагностики|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: для выполнения определенных административных операций должно существовать оповещение журнала действий (Microsoft.Network/networkSecurityGroups/delete)|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: для выполнения определенных административных операций должно существовать оповещение журнала действий (Microsoft.Network/networkSecurityGroups/securityRules/delete)|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: для выполнения определенных административных операций должно существовать оповещение журнала действий (Microsoft.Network/networkSecurityGroups/securityRules/write)|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: для выполнения определенных административных операций должно существовать оповещение журнала действий (Microsoft.Network/networkSecurityGroups/write)|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: для выполнения определенных административных операций должно существовать оповещение журнала действий (Microsoft.Sql/servers/firewallRules/delete)|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: для выполнения определенных административных операций должно существовать оповещение журнала действий (Microsoft.Sql/servers/firewallRules/write)|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: должны быть установлены только утвержденные расширения виртуальных машин|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: необходимо включить Azure Defender для реестров контейнеров|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: в вашем приложении API необходимо использовать управляемое удостоверение|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: должна быть включена проверка подлинности для приложения API|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: для определенных операций политики должно существовать оповещение журнала действий (Microsoft.Authorization/policyAssignments/delete)|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: для определенных операций политики должно существовать оповещение журнала действий (Microsoft.Authorization/policyAssignments/write)|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: должна быть включена проверка подлинности для приложения-функции|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: необходимо включить журналы диагностики в Data Lake Analytics|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: учетные записи хранения должны разрешать доступ из доверенных служб Майкрософт|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: в Key Vault необходимо включить журналы диагностики|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: для серверов базы данных PostgreSQL должен быть включен параметр "Принудительно использовать SSL-соединение"|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: В приложении-функции должна использоваться последняя версия HTTP|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: MFA должна быть включена для учетных записей с разрешениями на чтение в вашей подписке.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: доступ из Интернета по протоколу RDP должен быть заблокирован|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: для серверов базы данных MySQL должен быть включен параметр "Принудительно использовать SSL-соединение"|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: параметр "Сертификаты клиента (входящие сертификаты клиента)" для приложения-функции должен иметь значение "Вкл"|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: на серверах баз данных PostgreSQL должны быть включены контрольные точки журнала|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: на серверах баз данных PostgreSQL должны быть включены журналы подключений|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: на серверах баз данных PostgreSQL должны быть включены журналы отключений|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: На серверах SQL Server должна быть включена оценка уязвимости.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: В вашем веб-приложении необходимо использовать последнюю версию TLS|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: Внешние учетные записи с разрешениями владельца следует удалять из подписки.|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: в Служебной шине должны быть включены журналы диагностики|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: в Azure Stream Analytics должны быть включены журналы диагностики|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: В вашем приложении-функции необходимо использовать последнюю версию TLS|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Эффект политики: учетная запись хранения, содержащая контейнер с журналами действий, должна быть зашифрована с помощью BYOK|Дополнительные сведения об эффектах см. в разделе [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Включить кластеры AKS при аудите, если включены журналы диагностики масштабируемого набора виртуальных машин||
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Последняя версия Java для служб приложений|Последняя поддерживаемая версия Java для служб приложений|
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Последняя версия Python для Linux для служб приложений|Последняя поддерживаемая версия Python для служб приложений|
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Список регионов, где должен быть включен Наблюдатель за сетями|Чтобы просмотреть полный список регионов, выполните команду PowerShell Get-AzLocation.|
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Последняя версия PHP для служб приложений|Последняя поддерживаемая версия PHP для служб приложений|
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Требуемый период хранения (в днях) для журналов ресурсов|Дополнительные сведения о журналах ресурсов см. в разделе [https://aka.ms/resourcelogs](../../../azure-monitor/essentials/resource-logs.md). |
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Имя группы ресурсов для Наблюдателя за сетями|Имя группы ресурсов, в которой находятся Наблюдатели за сетями|
|CIS Microsoft Azure Foundations Benchmark версии 1.3.0|Назначение политики|Обязательный параметр аудита для серверов SQL Server||

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные статьи о схемах и способах их использования:

- Ознакомьтесь со сведениями о [жизненном цикле схем](../concepts/lifecycle.md).
- Узнайте, как использовать [статические и динамические параметры](../concepts/parameters.md).
- Научитесь настраивать [последовательность схемы](../concepts/sequencing-order.md).
- Узнайте, как применять [блокировку ресурсов схемы](../concepts/resource-locking.md).
- Узнайте, как [обновлять существующие назначения](../how-to/update-existing-assignments.md).