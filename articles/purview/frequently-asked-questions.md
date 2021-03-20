---
title: Вопросы и ответы
description: В этой статье содержатся ответы на часто задаваемые вопросы об Azure зрения.
author: SunetraVirdi
ms.author: suvirdi
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: conceptual
ms.date: 10/20/2020
ms.openlocfilehash: 94b765cbcbdd81505b08052845207ee1d93a28d9
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101667808"
---
# <a name="frequently-asked-questions-faq-about-azure-purview"></a>Часто задаваемые вопросы об Azure зрения

## <a name="overview"></a>Обзор

Многим организациям не хватает целостного понимания их данных. Трудно понять, какие данные существуют, где находятся данные, а также как найти и получить доступ к нужным данным. В данных отсутствует контекст, такой как журнал преобразований, классификация и исчерпывающие метаданные, что затрудняет для бизнес-пользователей поиск нужных данных и их использование соответствующим образом. В результате для информирования бизнес-решений используется только небольшая часть собранных данных. Наконец, определение проблем безопасности данных и защита конфиденциальных данных не согласуются. Он требует постоянного времени и усилий, особенно при сохранении гибкости данных.

Azure зрения — это решение по управлению данными. Она помогает клиентам получать глубокие знания о всех данных, сохраняя при этом контроль над их использованием. С помощью Azure зрения организации обнаруживают и исходят данные. Они получают ценную информацию о разположении данных и централизованно управляют доступом к данным.

## <a name="purpose-of-this-faq"></a>Назначение этой часто задаваемых вопросов

В этой статье содержатся ответы на часто задаваемые вопросы, которые часто запрашиваются у клиентов и групп полей. Она предназначена для уточнения вопросов о Зренияах Azure и связанных с ней решениях, таких как каталог данных Azure (ADC) Gen 2 (не рекомендуется) и Azure Information Protection.

### <a name="what-are-the-source-types-available-for-metadata-scanning-and-classification"></a>Какие типы источников доступны для сканирования и классификации метаданных?

|Azure|Узел, который не относится к Azure|
|---------|---------|
|Хранилище BLOB-объектов Azure|Power BI|
|Azure Synapse Analytics (хранилище данных SQL)|SQL Server |
|Azure Cosmos DB|Teradata (доступно в конце 2020)|
|Управляемый экземпляр SQL Azure|Протокол SAP ECC (доступен по окончании 2020)|
|Обозреватель данных Azure|SAP S/4 HANA (доступно в конце 2020)|
|Хранилище Azure Data Lake Storage 1-го поколения|Hive хранилище метаданных (доступно в конце 2020)|
|Azure Data Lake Storage 2-го поколения|Amazon S3|
|Файлы Azure|--|
|База данных SQL Azure|--|

### <a name="what-data-systemsprocessors-can-we-connect-and-get-lineage"></a>Какие системы данных и процессоры могут подключаться и получать журналы преобразований?

|Система данных и процессор 
|---------
|Фабрика данных Azure: действие копирования, действие потока данных 
|Пользовательский журнал преобразований   
|Azure Data Share   
|Power BI    |
|SQL Server Integration Services  

### <a name="how-are-adc-gen-2-azure-information-protection-and-azure-purview-related"></a>Как работают службы ADC Gen 2, Azure Information Protection и Azure зрения?

Изначально Azure зрения был создан как ADC Gen 2, но он был расширен в области охвата. Теперь он использует расширенные возможности каталога ADC Gen 2 в сочетании с возможностями классификации данных, добавления меток и применения политик соответствия требованиям Azure Information Protection. На сегодняшний день он ближе всего к более широкому отраслевым определениям системы управления данными.

### <a name="what-happens-to-customers-using-adc-gen-1"></a>Что происходит с клиентами, использующими ADC Gen 1?

Azure зрения — это основное внимание всем нововведениям в области решения каталога для Майкрософт. Поддержка ADC Gen 1 будет по прежнему поддерживаться.

### <a name="can-customers-have-multiple-azure-purview-accounts-in-the-same-subscription"></a>Могут ли клиенты иметь несколько учетных записей Azure зрения в одной подписке?

Да, мы поддерживаем множество учетных записей Azure зрения для каждой подписки и для каждого клиента.

### <a name="can-i-run-adc-gen-1-and-azure-purview-in-parallel"></a>Могу ли я параллельно запускать службы ADC Gen 1 и Azure зрения?

Да. Оба являются независимыми службами.

### <a name="how-do-i-migrate-existing-adc-gen-1-data-assets-to-azure-purview"></a>Разделы справки перенести существующие ресурсы данных зрения для службы ADC в Azure

Используйте интерфейсы API зрения Azure для извлечения из ADC Gen 1 и приема в Azure зрения. Для глоссария поддерживаются средства, основанные на CSV.

### <a name="how-do-i-encrypt-sensitive-data-for-sql-tables-using-azure-purview"></a>Разделы справки Шифровать конфиденциальные данные для таблиц SQL с помощью Azure зрения?

Шифрование данных выполняется на уровне источника данных. Azure зрения хранит только метаданные. Данные не проявляются предварительной версии.

### <a name="will-all-the-capabilities-of-adc-gen-2-exist-in-azure-purview"></a>Будут ли все возможности службы ADC Gen 2 существовать в Azure зрения?

Да.

<!--## Is the data lineage feature available in Azure Purview?

Yes, but it's limited to the Azure Data Factory connector.

<!-- ## How can I scan SQL Server on-premises? 

Use the self-host integration runtime capability. !-->

<!--### What is the difference between classification in Azure SQL Database and classification in Azure Purview?

|Azure SQL DB classification  |Azure Purview classification  |
|---------|---------|
|Classification is based on SQL metadata from system catalogs. |Classification is based on Azure Purview's sampling technique by using the system-defined or custom-defined regex pattern.|
|Custom classification is supported.     |Custom classification is supported.         |
|Doesn't use Microsoft 365 system classifiers out of the box.    | Uses Microsoft 365 system classifiers out of the box.        |
-->

### <a name="whats-the-difference-between-a-glossary-and-classification"></a>В чем разница между Глоссарием и классификацией?

Глоссарий использует соглашение об именовании, за которым следуют пользователи, не являющиеся техническими и бизнес-пользователями данных, которые также называются потребителями данных. Эти типы людей являются бизнес-аналитиками или специалистами по обработке и анализу данных, которые используют Azure зрения для поиска определенных типов данных на основе использования бизнеса. Например, аналитикам цепочки поставок может потребоваться найти условия для *типов SKU* и *сведения о поставке*. Они выполняют поиск этих терминов в глоссарии, чтобы найти нужные данные.
Классификация — это тег, применяемый к ресурсу данных на уровне таблицы, столбца или файла, который определяет, какие данные существуют в ресурсе. Классификацию можно применять автоматически или вручную в зависимости от типа найденных данных. Как правило, Теги классификации используются для того, чтобы определить, содержит ли ресурс конфиденциальные данные и какой тип конфиденциальных данных может быть.

### <a name="does-azure-purview-scan-and-classify-emails-pdfs-etc-in-my-sharepoint-and-onedrive"></a>Выполняет ли Azure зрения проверку и классификацию сообщений электронной почты, PDF-файлов и т. д. в SharePoint и OneDrive?

Сканирование для локальных сайтов и библиотек SharePoint осуществляется с помощью средства проверки Azure Information Protection. Сканер доступен для использования с помощью подписки клиента Microsoft 365 со следующими номерами SKU: точка административной установки P1, EMS E3 и M365 E3. Если у вас есть один из этих номеров SKU, у вас должны быть правильные права для начала использования сканера Azure Information Protection.

<!--### What is the difference between classifications and sensitivity labels in Azure Purview?

Azure Purview's data governance solution is based on the Apache Atlas framework. As defined by Atlas, classification is a way to identify the contents of an asset (table or file) or an entity (table column or structured file). This classification becomes a metadata property that allows Azure Purview to understand the data within each asset and govern and protect them.

Sensitivity labels are a Microsoft 365 concept that resembles classification at the asset level. You create a label with a collection of classifications applied at the asset or entity level.

Atlas-centric customers will see no real distinction between classifications and labels. To these customers, everything is a classification and labels aren't needed.

Security-focused customers will see a distinction between classification and labeling, but only because in Microsoft 365 the classifications aren't exposed directly to the user; only labels are visible. So, similar to Atlas, Office 365 security customers don't need to deal with both entities.
-->

### <a name="what-is-the-compute-used-for-the-scan"></a>Что такое вычисление, используемое для проверки?
Существует инфраструктура сканирования, управляемая корпорацией Майкрософт. Для большинства ресурсов Azure/AWS, которые мы поддерживаем, развертывание инфраструктуры сканирования не требуется.

### <a name="is-there-a-way-to-provision-azure-purview-via-azure-resource-manager-arm-template--cli--powershell"></a>Есть ли способ подготавливать Azure зрения через Azure Resource Manager (ARM) Template/CLI/PowerShell?

Да, доступен шаблон ARM

<!--### Does Azure Purview support guest users in AAD?-->

### <a name="im-already-using-atlas-can-i-easily-move-to-azure-purview"></a>Я уже использую Atlas, могу ли я легко перейти на Azure зрения?

Azure зрения совместим с Atlas API. При миграции из Atlas рекомендуется сначала проверить источники данных с помощью Azure зрения. После того как ресурсы будут доступны в вашей учетной записи, можно использовать аналогичные API-интерфейсы Atlas для интеграции, такие как обновление ресурсов или Добавление пользовательского журнала преобразований. Azure зрения изменяет API поиска для использования поиска Azure, поэтому вы должны иметь возможность использовать поиск по расширенному поиску.

### <a name="can-i-create-multiple-catalogs-in-my-tenant"></a>Можно ли создать несколько каталогов в моем клиенте?

Да, можно создать несколько учетных записей Azure зрения для каждой подписки и для каждого клиента. Вы можете ознакомиться со статьей ограничения по [управлению и увеличению квот для ресурсов с помощью Azure зрения](how-to-manage-quotas.md).

Дополнительные рекомендации о том, когда следует или не должны иметь несколько учетных записей, описаны в рекомендациях по [развертыванию Azure зрения](deployment-best-practices.md).

### <a name="can-i-register-multiple-tenants-within-a-single-azure-purview-account"></a>Можно ли зарегистрировать несколько клиентов в одной учетной записи Azure зрения?

Нет. сейчас для проверки источника данных другого клиента необходимо создать отдельную учетную запись Azure зрения в этом клиенте.

### <a name="does-azure-purview-support-column-level-lineage"></a>Поддерживает ли Azure зрения журнал преобразований на уровне столбцов?

Да, Azure зрения поддерживает журнал преобразований на уровне столбцов.