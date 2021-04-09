---
title: Руководство по переходу с Google Карт на Azure Maps | Microsoft Azure Maps
description: В руководстве показано, как перейти с использования Google Карт на Microsoft Azure Maps. Это руководство содержит инструкции по переходу на API-интерфейсы и пакеты средств разработки Azure Maps.
author: rbrundritt
ms.author: richbrun
ms.date: 09/23/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.custom: ''
ms.openlocfilehash: 6241f6156b01c3c90f00578ae5416e4e77270930
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100386804"
---
# <a name="tutorial-migrate-from-google-maps-to-azure-maps"></a>Руководство по Переход с Google Карт на Azure Maps

В этой статье содержатся сведения о переносе веб-приложений, мобильных и серверных программ из Google Карт на платформу Microsoft Azure Maps. Руководство предоставляет примеры кода для сравнения, а также предложения и рекомендации по переходу на Azure Maps. Из этого руководства вы узнаете:

> [!div class="checklist"]
> * о сходстве и различиях между эквивалентными функциями Google Карт и Azure Maps;
> * какие различия в лицензировании нужно учесть;
> * как спланировать миграцию;
> * где найти технические ресурсы и поддержку.

## <a name="prerequisites"></a>Предварительные требования

1. Войдите на [портал Azure](https://portal.azure.com). Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/), прежде чем начинать работу.
2. [Создайте учетную запись службы Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account)
3. [Получите первичный ключ подписки](quick-demo-map-app.md#get-the-primary-key-for-your-account), который иногда называется первичным ключом или ключом подписки. Дополнительные сведения о проверке подлинности в Azure Maps см. в [этой статье](how-to-manage-authentication.md).

## <a name="azure-maps-platform-overview"></a>Обзор платформы Azure Maps

Azure Maps предоставляет разработчикам из всех отраслей мощные геопространственные возможности. Картографические данные регулярно обновляются, что позволяет обеспечить географический контекст для веб-приложений и мобильных приложений. Azure Maps имеет набор интерфейсов REST API, совместимых с Azure One API. Эти интерфейсы REST API предоставляют функции отрисовки карт, поиска, маршрутизации, передачи трафика, работы с учетом часовых поясов, геолокации, геозон, данных карт, прогнозирования погоды, мобильности и пространственных операций. Операции снабжены пакетами SDK для веб-приложений и Android, чтобы обеспечить простоту и гибкость разработки, а также возможность переноса кода на разные платформы.

## <a name="high-level-platform-comparison"></a>Обобщенное сравнение платформ

В таблице представлен обобщенный список функций Azure Maps, которые аналогичны функциям в Google Картах. В этом списке отображаются не все функции Azure Maps. Некоторые дополнительные функции Azure Maps включают специальные возможности, геозоны, вычисления изохроны, пространственные операции, прямой доступ к фрагменту карты, пакетные службы и сравнение показателей покрытия данных (т. е. покрытие изображений).

| Функция Google Карт         | Поддержка в Azure Maps                     |
|-----------------------------|:--------------------------------------:|
| веб-пакет SDK.                     | ✓                                      |
| Android SDK                 | ✓                                      |
| iOS SDK                     | Запланировано                                |
| API-интерфейсы RESTful           | ✓                                      |
| Направления (построение маршрутов)        | ✓                                      |
| Матрица расстояний             | ✓                                      |
| Elevation                   | ✓ (предварительная версия)                            |
| Геокодирование (прямое и обратное) | ✓                                      |
| Географическое положение                 | Недоступно                                    |
| Ближайшие дороги               | ✓                                      |
| Поиск мест               | ✓                                      |
| Сведения о местах              | Недоступно. Предоставляется адрес веб-сайта и номер телефона |
| Фотографии мест               | Недоступно                                    |
| Автозавершение мест          | ✓                                      |
| Привязка к дороге                | ✓                                      |
| Ограничения скорости                | ✓                                      |
| Статические карты                 | ✓                                      |
| Статическое представление улиц          | Недоступно                                    |
| Часовой пояс                   | ✓                                      |
| Внедренный API Карт           | Недоступно                                    |
| URL-адреса карт                    | Недоступно                                    |

Служба Google Карты использует базовую проверку подлинности на основе ключа. Azure Maps поддерживает не только базовую проверку подлинности на основе ключа, но и проверку подлинности Azure Active Directory. Проверка подлинности с помощью Azure Active Directory предоставляет больше функций обеспечения безопасности, чем базовая проверка подлинности на основе ключа.

## <a name="licensing-considerations"></a>Вопросы лицензирования

При переходе на Azure Maps из Google Карт учитывайте следующие моменты, связанные с лицензированием.

* За использование интерактивных карт Azure Maps взымает плату, основанную на количестве загруженных фрагментов карты. С другой стороны, служба Google Карты взимает плату за загрузку элемента управления картой. В пакетах средств разработки для интерактивного использования Azure Maps плитки карт автоматически кэшируются, что позволяет снизить затраты на разработку. Одна транзакция Azure Maps создается для каждых 15 загруженных плиток карт. В пакетах средств разработки для интерактивной работы с Azure Maps используются фрагменты размером 512 пикселей. Это дает возможность выполнять в среднем не более одной транзакции на каждое представление страницы.
* Зачастую гораздо экономичнее заменить статические изображения карты из веб-служб Google Карт на веб-пакет SDK Azure Maps. Веб-пакет SDK для Azure Maps использует фрагменты карт. Если пользователь не изменяет карту и не масштабирует ее, служба зачастую создает лишь часть транзакции на каждую загрузку карты. Веб-пакет SDK для Azure Maps позволяет отключить панорамирование и масштабирование при необходимости. Кроме того, веб-пакет SDK для Azure Maps дает гораздо больше возможностей для визуализации, чем веб-служба статических карт.
* Azure Maps позволяет хранить в Azure данные для этой платформы. Также данные можно кэшировать в любом расположении на срок до шести месяцев согласно [условиям использования](https://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&DocumentTypeId=46).

Ниже перечислены некоторые ресурсы с информацией об Azure Maps.

* [Цены на Azure Maps](https://azure.microsoft.com/pricing/details/azure-maps/)
* [Калькулятор цен Azure](https://azure.microsoft.com/pricing/calculator/?service=azure-maps).
* [Условия использования Azure Maps](https://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&DocumentTypeId=46) (в составе условий для служб Microsoft Online Services)
* [Choose the right pricing tier in Azure Maps](./choose-pricing-tier.md) (Выбор подходящей ценовой категории для Azure Maps)

## <a name="suggested-migration-plan"></a>Предлагаемый план миграции

Далее описан обобщенный план миграции.

1. Проведите инвентаризацию пакетов SDK Google Карт и служб, которые использует ваше приложение. Убедитесь, что Azure Maps предоставляет альтернативные службы и пакеты SDK.
2. Если у вас еще нету подписки Azure, создайте ее на странице [https://azure.com](https://azure.com).
3. Создайте учетную запись Azure Maps ([документация](./how-to-manage-account-keys.md)) и ключ проверки подлинности или службу Azure Active Directory ([документация](./how-to-manage-authentication.md)).
4. Переместите код приложения.
5. Протестируйте перенесенное приложение.
6. Разверните перенесенное приложение в рабочей среде.

## <a name="create-an-azure-maps-account"></a>создание учетной записи службы Azure Maps

Чтобы создать учетную запись Azure Maps и получить доступ к платформе Azure Maps, выполните следующие действия:

1. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/), прежде чем начинать работу.
2. Войдите на [портал Azure](https://portal.azure.com/).
3. Создайте [учетную запись службы Azure Maps](./how-to-manage-account-keys.md). 
4. [Получите ключ подписки Azure Maps](./how-to-manage-authentication.md#view-authentication-details) или настройте проверку подлинности Azure Active Directory для повышения уровня безопасности.

## <a name="azure-maps-technical-resources"></a>Технические ресурсы по Azure Maps

Ниже приведен список полезных технических ресурсов для Azure Maps.

- Общие сведения: [https://azure.com/maps](https://azure.com/maps)
- Документация: [https://aka.ms/AzureMapsDocs](./index.yml)
- Примеры кода для веб-пакетов SDK: [https://aka.ms/AzureMapsSamples](https://aka.ms/AzureMapsSamples)
- Форумы разработчиков: [https://aka.ms/AzureMapsForums](/answers/topics/azure-maps.html)
- Видео: [https://aka.ms/AzureMapsVideos](https://aka.ms/AzureMapsVideos)
- Блог: [https://aka.ms/AzureMapsBlog](https://aka.ms/AzureMapsBlog)
- Технический блог: [https://aka.ms/AzureMapsTechBlog](https://aka.ms/AzureMapsTechBlog)
- Отзывы об Azure Maps (UserVoice): [https://aka.ms/AzureMapsFeedback](https://aka.ms/AzureMapsFeedback)
- [Jupyter Notebook для Azure Maps](https://github.com/Azure-Samples/Azure-Maps-Jupyter-Notebook)

## <a name="migration-support"></a>Поддержка миграции

Разработчики могут получить поддержку по миграции на [форумах](/answers/topics/azure-maps.html) или через любой из многих каналов поддержки Azure: [https://azure.microsoft.com/support/options](https://azure.microsoft.com/support/options)

## <a name="clean-up-resources"></a>Очистка ресурсов

Нет ресурсов для очистки.

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как перенести приложение Google Карт, изучив следующие статьи:

> [!div class="nextstepaction"]
> [Перенос веб-приложения](migrate-from-google-maps-web-app.md)
