---
title: 'Синхронизация Azure AD Connect: общие сведения о синхронизации и ее настройка | Документация Майкрософт'
description: В этой статье описываются принципы работы и настройка служб синхронизации Azure AD Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: ee4bf802-045b-4da0-986e-90aba2de58d6
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 11/08/2017
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: cea26cb119f64679807bc6c5eaadb41b341e5d5a
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89662390"
---
# <a name="azure-ad-connect-sync-understand-and-customize-synchronization"></a>Службы синхронизации Azure AD Connect: общие сведений о синхронизации и ее настройка
Службы синхронизации Azure Active Directory Connect (службы синхронизации Azure AD Connect) являются основным компонентом Azure AD Connect. Они отвечают за все операции, относящиеся к синхронизации идентификационных данных между локальной средой и Azure AD. Служба синхронизации Azure AD Connect — это преемник таких решений, как DirSync, Azure AD Sync и Forefront Identity Manager с настроенным соединителем Azure Active Directory.

В этой статье описывается **служба синхронизации Azure AD Connect** (также называемая **модулем синхронизации**) и приводятся ссылки на другие статьи на эту тему. Ссылки на Azure AD Connect см. в статье [интеграция локальных удостоверений с Azure Active Directory](whatis-hybrid-identity.md).

Служба синхронизации состоит из двух компонентов — локального компонента **синхронизации Azure AD Connect** и компонента на стороне службы в Azure AD, который называется **службой синхронизации Azure AD Connect**.

## <a name="azure-ad-connect-sync-topics"></a>Статьи, имеющие отношение к службе синхронизации Azure AD Connect
| Раздел | Содержание и целевая аудитория |
| --- | --- |
| **Знакомство со службой синхронизации Azure AD Connect** | |
| [Understanding the architecture (Основные сведения об архитектуре)](concept-azure-ad-connect-sync-architecture.md) |Для пользователей без опыта работы с модулем синхронизации, которые хотят узнать о его архитектуре и используемых терминах. |
| [Технические концепции](how-to-connect-sync-technical-concepts.md) |Сокращенная версия статьи об архитектуре с краткими пояснениями по используемым терминам. |
| [Топологии для Azure AD Connect](plan-connect-topologies.md) |Описание разных топологий и сценариев, поддерживаемых модулем синхронизации. |
| **Настраиваемая конфигурация** | |
| [Повторный запуск мастера установки](how-to-connect-installation-wizard.md) |Объясняет, какие есть варианты действий при повторном запуске мастера установки Azure AD Connect. |
| [Общие сведения о декларативной подготовке](concept-azure-ad-connect-sync-declarative-provisioning.md) |Описание модели конфигурации, которая называется декларативной подготовкой. |
| [Знакомство с выражениями декларативной подготовки](concept-azure-ad-connect-sync-declarative-provisioning-expressions.md) |Описание синтаксиса для выражений языка, используемого в декларативной подготовке. |
| [Общие сведения о конфигурации по умолчанию](concept-azure-ad-connect-sync-default-configuration.md) |Описание стандартных правил и конфигурации по умолчанию. Также описывается, как правила используются при работе со стандартными сценариями. |
| [Общие сведения о пользователях и контактах](concept-azure-ad-connect-sync-user-and-contacts.md) |Это продолжение предыдущей статьи, где описано взаимодействие конфигураций для пользователей и контактов, в частности в среде с несколькими лесами. |
| [How to make a change to the default configuration (Изменение конфигурации по умолчанию)](how-to-connect-sync-change-the-configuration.md) |Описывает типичные изменения конфигурации для потоков атрибутов. |
| [Рекомендации по изменению конфигурации по умолчанию](how-to-connect-sync-best-practices-changing-default-configuration.md) |Ограничения поддержки и изменение стандартной конфигурации. |
| [Настройка фильтрации](how-to-connect-sync-configure-filtering.md) |Описывает возможности установки ограничений для синхронизации объектов с Azure AD. Содержит пошаговые рекомендации по настройке этих ограничений. |
| **Функции и сценарии** | |
| [Предотвращение случайного удаления](how-to-connect-sync-feature-prevent-accidental-deletes.md) |Описание функции *предотвращения случайного удаления* и рекомендации по ее настройке. |
| [Планировщик](how-to-connect-sync-feature-scheduler.md) |Описывает встроенный планировщик, который импортирует, синхронизирует и экспортирует данные. |
| [Реализация синхронизации хэшированных паролей в службе синхронизации Azure AD Connect](how-to-connect-password-hash-synchronization.md) |Описано, как работает синхронизация паролей, как реализовать и использовать эту функцию, а также как устранять связанные с ней неполадки. |
| [Обратная запись устройств](how-to-connect-device-writeback.md) |Описывается, как работает обратная запись устройства в Azure AD Connect. |
| [Расширения каталогов](how-to-connect-sync-feature-directory-extensions.md) |Описывается, как расширить схему Azure AD с помощью собственных пользовательских атрибутов. |
| [Microsoft 365 PreferredDataLocation](how-to-connect-sync-feature-preferreddatalocation.md) |Описывает, как разместить ресурсы Microsoft 365 пользователя в том же регионе, что и пользователь. |
| **Служба синхронизации MIM** | |
| [Функции службы синхронизации Azure AD Connect](how-to-connect-syncservice-features.md) |Описание компонента на стороне службы синхронизации и действий по изменению параметров синхронизации в Azure AD. |
| [Устойчивость повторяющихся атрибутов](how-to-connect-syncservice-duplicate-attribute-resiliency.md) |Описание включения и применения функции устойчивости значений повторяющихся атрибутов **userPrincipalName** и **proxyAddresses**. |
| **Операции и пользовательский интерфейс** | |
| [Synchronization Service Manager](how-to-connect-sync-service-manager-ui.md) |Описание пользовательского интерфейса Synchronization Service Manager, включая вкладки [Operations](how-to-connect-sync-service-manager-ui-operations.md) (Операции), [Connectors](how-to-connect-sync-service-manager-ui-connectors.md) (Соединители), [Metaverse Designer](how-to-connect-sync-service-manager-ui-mvdesigner.md) (Конструктор метавселенной) и [Metaverse Search](how-to-connect-sync-service-manager-ui-mvsearch.md) (Поиск в метавселенной). |
| [Службы синхронизации Azure AD Connect: рабочие задачи и рекомендации](./how-to-connect-sync-staging-server.md) |Описание рабочих задач, например аварийного восстановления. |
| **Инструкции...** | |
| [Сброс учетной записи Azure AD](how-to-connect-azureadaccount.md) |Как сбросить учетные данные учетной записи службы, используемые для подключения службы синхронизации Azure AD Connect к Azure AD. |
| **Дополнительные сведения и ссылки** | |
| [Порты](reference-connect-ports.md) |Список портов, которые необходимо открыть между модулем синхронизации, локальными каталогами и Azure AD. |
| [Службы синхронизации Azure AD Connect: атрибуты, синхронизируемые с Azure Active Directory](reference-connect-sync-attributes-synchronized.md) |Список всех атрибутов, которые синхронизируются между локальной средой AD и Azure AD. |
| [Справочник по функциям](reference-connect-sync-functions-reference.md) |Список всех функций, доступных в рамках декларативной подготовки. |

## <a name="additional-resources"></a>Дополнительные ресурсы
* [Интеграция локальных удостоверений с Azure Active Directory](whatis-hybrid-identity.md)