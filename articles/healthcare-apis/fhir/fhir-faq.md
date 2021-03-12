---
title: Часто задаваемые вопросы о FHIR Services в Azure — API Azure для FHIR
description: Получите ответы на часто задаваемые вопросы о API Azure для FHIR, например о местоположении хранения данных для интерфейсов API FHIR и поддержки версий.
services: healthcare-apis
author: caitlinv39
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: reference
ms.date: 1/21/2021
ms.author: cavoeg
ms.openlocfilehash: 22d9df80bce6e79f0058567f8df76935e431e598
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103020566"
---
# <a name="frequently-asked-questions-about-the-azure-api-for-fhir"></a>Часто задаваемые вопросы об API Azure для FHIR

## <a name="azure-api-for-fhir-the-basics"></a>API Azure для FHIR: основные сведения

### <a name="what-is-fhir"></a>Что такое FHIR?
Ресурсы по быстрому взаимодействию в сфере здравоохранения (FHIR — «пожар») — это стандарт взаимодействия, предназначенный для обмена данными здравоохранения между различными системами исправности. Этот стандарт был разработан HL7 Организацией и принят организациями здравоохранения по всему миру. Самой последней версией FHIR является R4 (выпуск 4). API Azure для FHIR поддерживает R4, а также предыдущую версию STU3 (Standard для пробного использования 3). Дополнительные сведения о FHIR см. в [HL7.org](http://hl7.org/fhir/summary.html).

### <a name="is-the-data-behind-the-fhir-apis-stored-in-azure"></a>Данные за API FHIR хранятся в Azure?

Да, данные хранятся в управляемых базах данных в Azure. API Azure для FHIR не предоставляет прямой доступ к базовому хранилищу данных.

### <a name="what-identity-provider-do-you-support"></a>Какой поставщик удостоверений поддерживается?

В настоящее время в качестве поставщика удостоверений поддерживается Microsoft Azure Active Directory.

### <a name="what-is-the-recovery-point-objective-rpo-for-the-azure-api-for-fhir"></a>Что такое Целевая точка восстановления (RPO) для API Azure для FHIR?
API Azure для FHIR поддерживается Cosmos DB как наш поставщик сохраняемости. По этой причине значение RPO для службы равно [Cosmos DB (один регион)](../../cosmos-db/consistency-levels.md) и < 240 минут.

### <a name="what-fhir-version-do-you-support"></a>Какая версия FHIR поддерживается?

Мы поддерживаем версии 4.0.0 и 3.0.1 как в API Azure для FHIR (PaaS), так и на сервере FHIR для Azure (с открытым кодом).

Дополнительные сведения см. в разделе [Поддерживаемые функции](fhir-features-supported.md). Узнайте о том, что изменилось между версиями FHIR (например, STU3 и R4) в [журнале версий для HL7 FHIR](https://hl7.org/fhir/R4/history.html).

Соединитель Azure IoT для FHIR (Предварительная версия) сейчас поддерживает только FHIR версии R4. он доступен только в экземплярах R4 API Azure для FHIR.

### <a name="whats-the-difference-between-microsoft-fhir-server-for-azure-and-the-azure-api-for-fhir"></a>В чем разница между "Microsoft FHIR Server для Azure" и "API Azure для FHIR"?

API Azure для FHIR — это размещенная и управляемая версия сервера Microsoft FHIR с открытым исходным кодом для Azure. В управляемой службе Корпорация Майкрософт предоставляет все услуги по обслуживанию и обновлению. 

При запуске сервера FHIR для Azure у вас есть прямой доступ к базовым службам, но они отвечают за обслуживание и обновление сервера и всю необходимую работу по обеспечению соответствия, если вы храните данные фи.

Для точки зрения разработки все функции, которые не применяются только к управляемой службе, сначала развертываются на сервере Microsoft FHIR с открытым исходным кодом для Azure. После проверки в открытом источнике оно будет выпущено для решения PaaS Azure API for FHIR. Время между выпуском в с открытым исходным кодом и PaaS зависит от сложности функции и других приоритетов плана. Это тот же процесс для всех наших служб, например соединитель Azure IoT для FHIR (Предварительная версия).

### <a name="where-can-i-see-what-is-releasing-into-the-azure-api-for-fhir"></a>Где можно увидеть, что выпускается в API Azure для FHIR?

Некоторые сведения о выпуске в API Azure для FHIR см. в [выпуске](https://github.com/microsoft/fhir-server/releases) сервера FHIR с открытым исходным кодом. Начиная с ноября 2020, мы добавили элементы с тегами Azure-API-for-FHIR, если элемент с открытым исходным кодом выполнит выход в управляемую службу. Как правило, эти функции доступны через две недели, когда они находятся на странице выпуска с открытым кодом. Мы также включили инструкции по тестированию сборки [здесь] ( https://github.com/microsoft/fhir-server/blob/master/docs/Testing-Releases.md) Если вы хотите выполнить тестирование в собственной среде. Мы расспользум, как лучше поделиться дополнительными обновлениями управляемой службы.

### <a name="in-which-regions-is-azure-api-for-fhir-available"></a>В каких регионах доступен API Azure для FHIR?

Сейчас у нас есть общая доступность для общедоступных и государственных организаций в [нескольких географических регионах](https://azure.microsoft.com/global-infrastructure/services/?products=azure-api-for-fhir&regions=non-regional,us-east,us-east-2,us-central,us-north-central,us-south-central,us-west-central,us-west,us-west-2,canada-east,canada-central,usgov-non-regional,us-dod-central,us-dod-east,usgov-arizona,usgov-texas,usgov-virginia). Дополнительные сведения о облачных службах для государственных организаций см. в статье [службы Azure по](../../azure-government/compliance/azure-services-in-fedramp-auditscope.md)адресу FedRAMP.

### <a name="where-can-i-see-what-is-releasing-into-the-azure-api-for-fhir"></a>Где можно увидеть, что выпускается в API Azure для FHIR?

Некоторые сведения о выпуске в API Azure для FHIR см. в [выпуске](https://github.com/microsoft/fhir-server/releases) сервера FHIR с открытым исходным кодом. Мы работаем над отметкой элементов с помощью Azure-API-for-FHIR, если они будут выпущены для управляемой службы и обычно доступны через две недели после их появления на странице выпуска с открытым кодом. Мы также включили инструкции по тестированию сборки [здесь](https://github.com/microsoft/fhir-server/blob/master/docs/Testing-Releases.md) , если вы хотите выполнить тестирование в собственной среде. Мы расспользум, как лучше поделиться дополнительными обновлениями управляемой службы.

### <a name="what-is-smart-on-fhir"></a>Что такое SMART on FHIR?

Интеллектуальные (подставляемые медицинские приложения и многократно используемые технологии) на FHIR — это набор открытых спецификаций для интеграции партнерских приложений с серверами FHIR и другими системными ИТ-системами, такими как электронные записи о работоспособности и обмен информацией о работоспособности. Создав ИНТЕЛЛЕКТУАЛЬНое приложение на FHIR, вы сможете обеспечить доступ к приложению и использовать его в различных системах.
Проверка подлинности и API Azure для FHIR. Чтобы получить дополнительные сведения о смарт-разработе, перейдите в [интеллектуальную Подправку](https://smarthealthit.org/).

### <a name="where-can-i-find-what-version-of-fhir-is-running-on-my-database"></a>Где можно найти версию FHIR, работающую в базе данных. 

Точную версию FHIR, предоставляемую в инструкции Capability, можно найти в свойстве Фхирверсион.

## <a name="fhir-implementations-and-specifications"></a>Реализации и спецификации FHIR

### <a name="can-i-create-a-custom-fhir-resource"></a>Можно ли создать настраиваемый ресурс FHIR?

Мы не разрешены пользовательские ресурсы FHIR. Если вам нужен настраиваемый ресурс FHIR, вы можете создать настраиваемый ресурс на основе [базового ресурса](http://www.hl7.org/fhir/basic.html) с расширениями. 

### <a name="are-extensions-supported-on-azure-api-for-fhir"></a>Поддерживаются ли [расширения](https://www.hl7.org/fhir/extensibility.html) в API Azure для FHIR?

Мы позволяем загрузить на сервер любые допустимые данные JSON FHIR. Если требуется сохранить определение структуры, определяющее расширение, можно сохранить его как ресурс определения структуры. В настоящее время поиск по расширениям невозможен.

### <a name="what-is-the-limit-on-_count"></a>Каково предельное значение _count?

Текущее ограничение на _count — 100. Если задать _count более 100, в пакете появится предупреждение о том, что будут показаны только 100 записей.

### <a name="are-there-any-limitations-on-the-group-export-functionality"></a>Существуют ли ограничения на функции экспорта групп?

Для экспорта группы мы экспортируем только входящие ссылки из группы, а не все характеристики [ресурса группы](https://www.hl7.org/fhir/group.html).

### <a name="can-i-post-a-bundle-to-the-azure-api-for-fhir"></a>Можно ли отправить пакет в API Azure для FHIR?

В настоящее время поддерживаются пакеты [пакетной обработки](https://www.hl7.org/fhir/valueset-bundle-type.html) , но не поддерживаются учетные пакеты транзакций в API Azure для FHIR. Можно использовать сервер FHIR с открытым исходным кодом, поддерживаемый SQL, для размещения пакетов транзакций.

### <a name="how-can-i-get-all-resources-for-a-single-patient-in-the-azure-api-for-fhir"></a>Как можно получить все ресурсы для одного пациента в API Azure для FHIR?

Мы поддерживаем [Поиск секций](https://www.hl7.org/fhir/compartmentdefinition.html) в API Azure для FHIR. Это позволяет получить все ресурсы, связанные с конкретным пациентами. Обратите внимание, что в правой части секции содержатся все ресурсы, связанные с пациентом, но не с пациентами, поэтому необходимо также выполнить поиск, чтобы получить пациент, если в результатах потребуется ресурс пациента.

Ниже приведены некоторые примеры.

* ПОЛУЧЕНИЕ пациента/<id>/*
* ПОЛУЧЕНИЕ пациента/ <id> /обсерватион
* Получите пациент/ <id> /обсерватион? код = 8302-2

### <a name="what-is-the-default-sort-when-searching-for-resources-in-azure-api-for-fhir"></a>Что такое Сортировка по умолчанию при поиске ресурсов в API Azure для FHIR?

Мы поддерживаем сортировку по дате последнего обновления: _sort = _lastUpdated. Дополнительные сведения о других поддерживаемых параметрах поиска см. на [странице поддерживаемых возможностей](fhir-features-supported.md#search).

### <a name="does-the-azure-api-for-fhir-support-everything"></a>Поддерживает ли API Azure для FHIR $everything? 

Нет. В настоящее время не поддерживаются $everything. Однако его можно достичь двумя вызовами API. Например, чтобы получить сведения о пациентах $, вы можете сначала извлечь запись пациента с помощью/патиент/[ID], а затем второй вызов для получения всех данных пациента с помощью/патиент/[ID]/*.

Дополнительные сведения можно просмотреть в этой [публикации сообщества](https://chat.fhir.org/#narrow/stream/179166-implementers/topic/.24everything.20with.20_type). 

### <a name="how-does-export-work"></a>Как работает $export?

$export является частью спецификации FHIR: https://hl7.org/fhir/uv/bulkdata/export/index.html . Если служба FHIR настроена с управляемым удостоверением и учетной записью хранения, и если управляемое удостоверение имеет доступ к этой учетной записи хранения, можно просто вызвать $export в API FHIR, и все ресурсы FHIR будут экспортированы в учетную запись хранения. Дополнительные сведения см. [в статье $Export](export-data.md).

### <a name="is-de-identified-export-available-at-patient-and-group-level-as-well"></a>Доступен ли неизвестный экспорт, доступный на уровне пациента и группы?
Анонимный экспорт в настоящее время поддерживается только для полного экспорта системы (/$export), а не для экспорта пациента (/патиент/$export). Мы работаем над тем, чтобы сделать его доступным на уровне пациента.

## <a name="using-azure-api-for-fhir"></a>Использование API Azure для FHIR

### <a name="how-do-i-enable-log-analytics-for-azure-api-for-fhir"></a>Разделы справки включить службу log Analytics для Azure API для FHIR?

Мы включили ведение журнала диагностики и разрешаете Просмотр образцов запросов для этих журналов. Дополнительные сведения о включении журналов аудита и образцов запросов см. в [этом разделе](enable-diagnostic-logging.md). Если вы хотите включить дополнительные сведения в журналы, извлеките их [с помощью настраиваемых заголовков HTTP](use-custom-headers.md).

### <a name="where-can-i-see-some-examples-of-using-the-azure-api-for-fhir-within-a-workflow"></a>Где можно увидеть некоторые примеры использования API Azure для FHIR в рабочем процессе?

У нас есть коллекция эталонных архитектур, доступных на [странице GitHub архитектуры работоспособности](https://github.com/microsoft/health-architectures).

### <a name="where-can-i-see-an-example-of-connecting-a-web-application-to-azure-api-for-fhir"></a>Где можно увидеть пример подключения веб-приложения к API Azure для FHIR?

У нас есть [страница GitHub с архитектурой работоспособности](https://aka.ms/health-architectures) , которая содержит примеры приложений и сценариев. Здесь показано, как подключить веб-приложение к Azure API для FHIR.  

## <a name="azure-api-for-fhir-features-and-services"></a>API Azure для функций и служб FHIR 

### <a name="is-there-a-way-to-encrypt-my-data-using-my-personal-key-not-a-default-key"></a>Существует ли способ шифрования данных с помощью личного ключа, а не ключа по умолчанию?

Да, API Azure для FHIR позволяет настраивать управляемые клиентом ключи, используя поддержку из Cosmos DB. Дополнительные сведения о шифровании данных с помощью личного ключа см. в [этом разделе](customer-managed-key.md).

## <a name="azure-api-for-fhir-preview-features"></a>API Azure для FHIR: Предварительная версия функций

### <a name="can-i-configure-scaling-capacity-for-azure-iot-connector-for-fhir-preview"></a>Можно ли настроить масштабирование для соединителя Azure IoT для FHIR (Предварительная версия)?

Так как соединитель Azure IoT для FHIR предоставляется бесплатно во время общедоступной предварительной версии, его масштабирование является фиксированным и ограниченным. Соединитель Azure IoT для конфигурации FHIR, доступный в общедоступной предварительной версии, должен обеспечивать пропускную способность около 200 сообщений в секунду. Некоторые формы конфигурации емкости ресурсов будут доступны в общедоступной версии.

### <a name="why-cant-i-install-azure-iot-connector-for-fhir-preview-when-private-link-is-enabled-on-azure-api-for-fhir"></a>Почему не удается установить соединитель Azure IoT для FHIR (Предварительная версия), если частная ссылка включена в API Azure для FHIR?

В настоящее время соединитель Azure IoT для FHIR не поддерживает возможность использования частных ссылок. Таким образом, если в API Azure для FHIR включена частная ссылка, вы не сможете установить соединитель Azure IoT для FHIR и наоборот. Это ограничение должно исчезнуть, когда соединитель Azure IoT для FHIR доступен для общедоступной версии.
