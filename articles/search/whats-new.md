---
title: Новые возможности в Когнитивном поиске Azure
description: Объявления о новых и улучшенных возможностях, включая переименование Поиска Azure в Когнитивный поиск Azure.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: overview
ms.date: 02/09/2021
ms.custom: references_regions
ms.openlocfilehash: 6de91d51f3700c25ba1e0f361c948dc8ab338c43
ms.sourcegitcommit: e559daa1f7115d703bfa1b87da1cf267bf6ae9e8
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/17/2021
ms.locfileid: "100590556"
---
# <a name="whats-new-in-azure-cognitive-search"></a>Новые возможности в Когнитивном поиске Azure

Узнайте о новых возможностях службы. Создайте закладку для этой страницы, чтобы получать последние сведения об обновлениях службы. Просмотрите список [предварительных версий функций](search-api-preview.md), чтобы узнать, какие функции предоставляются в общедоступной предварительной версии.

## <a name="february-2021"></a>Февраль 2021 года

|Функция&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  |  Описание | Доступность  |
|------------------------------|---------------|---------------|
| [Сброс документов (предварительная версия)](search-howto-run-reset-indexers.md) |  Повторно обрабатывает отдельные выбранные документы поиска в рабочих нагрузках индексатора. | [REST API службы "Поиск" версии 2020-06-30-Preview](/rest/api/searchservice/index-preview) |
| [Зоны доступности](search-performance-optimization.md#availability-zones)| Службы поиска с двумя или более репликами в определенных регионах, как описано в статье [Масштабирование для производительности в Когнитивном поиске Azure](search-performance-optimization.md#availability-zones), обеспечивают устойчивость, размещая реплики в двух или более разных физических расположениях.  | Регион и дата создания службы "Поиск" определяют уровень доступности. Дополнительные сведения см. в статье "Масштабирование для производительности в Когнитивном поиске Azure". |
| [Azure CLI](/cli/azure/search) </br>[Azure PowerShell](/powershell/module/az.search/) | Теперь новые редакции обеспечивают полный диапазон операций в REST API управления 2020-08-01, включая поддержку правил брандмауэра для IP-адресов и закрытых конечных точек. | Общедоступная версия. |

## <a name="january-2021"></a>Январь 2021 г.

|Функция&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  |  Описание | Доступность  |
|------------------------------|-------------|---------------|
| [Акселератор решений для Когнитивного поиска Azure и QnA Maker](https://github.com/Azure-Samples/search-qna-maker-accelerator) | Извлекает вопросы и ответы из документа и предлагает наиболее релевантные ответы. Демонстрационное приложение можно найти по адресу [https://aka.ms/qnaWithAzureSearchDemo](https://aka.ms/qnaWithAzureSearchDemo).  | Проекты с открытым исходным кодом (без соглашения об уровне обслуживания) |

## <a name="2020-archive"></a>Архив за 2020 г.

| Месяц | Функция | Описание |
|-------|---------|-------------|
| Ноябрь | [Шифрование на базе управляемых клиентом ключей (расширенное)](search-security-manage-encryption-keys.md) | Позволяет выполнять управляемое клиентами шифрование с широким спектром ресурсов, созданных и управляемых службой поиска. Общедоступная версия.|
| Сентябрь | [Расширение Visual Studio Code для Когнитивного поиска Azure](search-get-started-vs-code.md) | Добавляет рабочую область, возможности навигации, IntelliSense и шаблоны для создания индексов, индексаторов, источников данных и наборов навыков. | Общедоступная предварительная версия |
| Сентябрь | [Управляемое удостоверение службы (индексаторы)](search-howto-managed-identities-data-sources.md) | Общедоступная версия.  |
| Сентябрь | [Отправка исходящих запросов с использованием приватного канала](search-indexer-howto-access-private.md) | Общедоступная версия.  |
| Сентябрь | [REST API управления (2020-08-01)](/rest/api/searchmanagement/management-api-versions) | Общедоступная версия. |
| Сентябрь | [REST API управления (2020-08-01, предварительная версия)](/rest/api/searchmanagement/management-api-versions) | Добавляет общий ресурс приватного канала для Функций Azure и Azure SQL для баз данных MySQL. |
| Сентябрь | [Пакет SDK .NET версии 4.0 для управления](/dotnet/api/overview/azure/search/management) |  Обновление пакета SDK Azure для пакета SDK управления, целевая версия REST API 2020-08-01. Общедоступная версия.|
| Август | [Двойное шифрование](search-security-overview.md#encryption) | Общедоступно для всех служб поиска, созданных после 1 августа 2020 г. в таких регионах: "Западная часть США 2", "Восточная часть США", "Центрально-южная часть США", "US Gov (Вирджиния)", "US Gov (Аризона)". |
| Июль | [Клиентская библиотека Azure.Search.Documents](/dotnet/api/overview/azure/search.documents-readme) | Пакет Azure SDK для .NET (общедоступный). |
| Июль | [Клиентская библиотека Azure.Search.Documents](/python/api/overview/azure/search-documents-readme)  | Пакет Azure SDK для Python (общедоступный). |
| Июль | [Клиентская библиотека @azure/search-documents](/javascript/api/overview/azure/search-documents-readme)  | Пакет Azure SDK для JavaScript (общедоступный). |
| Июнь | [Хранилище знаний](knowledge-store-concept-intro.md) | Общедоступная версия. |
| Июнь | [REST API поиска 2020-06-30](/rest/api/searchservice/) | Общедоступная версия. |
| Июнь | [REST API службы "Поиск" версии 2020-06-30-Preview](/rest/api/searchservice/) | Включает возможность сброса набора навыков для выборочной повторной обработки навыков и добавочного обогащения. |
| Июнь | [Алгоритм релевантности Okapi BM25](index-ranking-similarity.md) | Общедоступная версия. |
| Июнь |  **executionEnvironment** (применяется к службам поиска, использующим Приватный канал Azure). | Общедоступная версия. |
| Июнь | [Навык Машинного обучения Azure (предварительная версия)](cognitive-search-aml-skill.md) | Когнитивный навык, расширяющий обогащение ИИ с помощью пользовательской модели Машинного обучения Azure (AML). |
| Май | [Сеансы отладки (предварительная версия)](cognitive-search-debug-session.md) | Отладчик набора навыков на портале.  |
| Май | [Правила IP-адресов для встроенной поддержки брандмауэра](service-configure-firewall.md) | Общедоступная версия.  |
| Май | [Приватный канал Azure для частной конечной точки поиска](service-create-private-endpoint.md) | Общедоступная версия.  |
| Май | [Управляемое удостоверение службы (индексаторы), предварительная версия](search-howto-managed-identities-data-sources.md) | Подключение к источникам данных Azure с помощью управляемого удостоверения.  |
| Май | [Параметр запроса sessionId](index-similarity-and-scoring.md), [параметр scoringStatistics=global](index-similarity-and-scoring.md#scoring-statistics)  | Глобальная статистика поиска, используемая [моделями машинного обучения (LearnToRank) для определения релевантности поиска](https://github.com/Azure-Samples/search-ranking-tutorial).  |
| Май | [Детализация результатов оценки релевантности featuresMode (предварительная версия)](index-similarity-and-scoring.md#featuresMode-param)  |   |
|Март  | [Нативная функция обратимого удаления большого двоичного объекта (предварительная версия)](search-howto-index-changed-deleted-blobs.md) | Удаляет документы поиска, если исходный большой двоичный объект обратимо удален в Хранилище BLOB-объектов. |
|Март  | [REST API управления (2020-03-13)](/rest/api/searchmanagement/management-api-versions) | Общедоступная версия. |
|Февраль | [Навык обнаружения персональных данных (предварительная версия)](cognitive-search-skill-pii-detection.md)  | Когнитивный навык, который извлекает и маскирует персональные данные. |
|Февраль | [Навык поиска настраиваемых сущностей (предварительная версия)](cognitive-search-skill-custom-entity-lookup.md) | Когнитивный навык, который находит слова и фразы из списка и помечает все документы с совпадающими сущностями.  |
|Январь | [Шифрование на базе управляемых клиентом ключей](search-security-manage-encryption-keys.md) | Общедоступная версия  |
|Январь | [Правила IP-адресов для встроенной поддержки брандмауэра (предварительная версия)](service-configure-firewall.md) | Новые свойства **IpRule** и **NetworkRuleSet** в [API CreateOrUpdate](/rest/api/searchmanagement/2019-10-01-preview/createorupdate-service).  |
|Январь | [Создание частной конечной точки (предварительная версия)](service-create-private-endpoint.md) | Настройка Приватного канала для безопасных подключений к службе поиска. Эта предварительная версия функции включает в себя зависимый [Приватный канал Azure](../private-link/private-link-overview.md) и [Виртуальную сеть Azure](../virtual-network/virtual-networks-overview.md). |

## <a name="2019-archive"></a>Архив за 2019 г.

| Месяц | Функция | Описание |
|-------|---------|-------------|
|Декабрь | [Создание демонстрационного приложения (предварительная версия)](search-create-app-portal.md) | Мастер, который создает скачиваемый HTML-файл с доступом запросов (только для чтения) к индексу, предназначенный для проверки и тестирования, а не для быстрого создания полноценного клиентского приложения.|
|Ноябрь | [Добавочное обогащение данных (предварительная версия)](cognitive-search-incremental-indexing-conceptual.md) | Кэширует обработку наборов навыков для повторного использования в будущем.  |
|Ноябрь | [Навык извлечения документов (предварительная версия)](cognitive-search-skill-document-extraction.md) | Когнитивный навык для извлечения содержимого файла из набора навыков.|
|Ноябрь | [Навык перевода текста](cognitive-search-skill-text-translation.md) | Когнитивный навык, используемый во время индексирования, который оценивает и переводит текст. Общедоступная версия.|
|Ноябрь | [Шаблоны Power BI](https://github.com/Azure-Samples/cognitive-search-templates/blob/master/README.md) | Шаблон для визуализации содержимого в хранилище знаний. |
|Ноябрь | [Azure Data Lake Storage 2-го поколения (предварительная версия)](search-howto-index-azure-data-lake-storage.md), [API Cosmos DB Gremlin (предварительная версия)](search-howto-index-cosmosdb.md) и [API Cosmos DB Cassandra (предварительная версия)](search-howto-index-cosmosdb.md) | Новые источники данных индексаторов в общедоступной предварительной версии. |
|Июль | [Поддержка облака Azure для государственных организаций](https://azure.microsoft.com/global-infrastructure/services/?regions=usgov-non-regional,us-dod-central,us-dod-east,usgov-arizona,usgov-texas,usgov-virginia&products=search) | Общедоступная версия.|

<a name="new-service-name"></a>

## <a name="new-service-name"></a>Новое имя службы

Поиск Azure будет переименован в **Когнитивный поиск Azure** в октябре 2019 г., чтобы правильно отражать расширенную (но не обязательную) возможность использовать когнитивные навыки и искусственный интеллект для базовых операций. Все версии API, пакеты NuGet, пространства имен и конечные точки остаются прежними. Изменение имени службы никак не затронет существующие и новые решения поиска.

## <a name="service-updates"></a>Обновления службы

[Объявления об обновлении службы](https://azure.microsoft.com/updates/?product=search&status=all) для Когнитивного поиска Azure доступны на веб-сайте Azure.