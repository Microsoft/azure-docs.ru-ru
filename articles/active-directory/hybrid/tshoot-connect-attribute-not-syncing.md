---
title: Устранение неполадок атрибута, связанных с отсутствием синхронизации в Azure AD Connect | Документация Майкрософт
description: В этой статье приводятся пошаговые инструкции по устранению неполадок, связанных с синхронизацией атрибутов, с помощью задач устранения неполадок.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: curtand
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 01/31/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: a6df1347eab57a6971fe2e39c0a55869c8f23939
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91317493"
---
# <a name="troubleshoot-an-attribute-not-synchronizing-in-azure-ad-connect"></a>Устранение неполадок атрибута, связанных с отсутствием синхронизации в Azure AD Connect

## <a name="recommended-steps"></a>**Рекомендуемые действия**

Прежде чем исследовать проблемы синхронизации атрибутов, давайте разберемся с процессом синхронизации **Azure AD Connect**.

  ![Процесс синхронизации Azure AD Connect](media/tshoot-connect-attribute-not-syncing/tshoot-connect-attribute-not-syncing/syncingprocess.png)

### <a name="terminology"></a>**Терминология**

* **CS:** Пространство соединителя, таблица в базе данных.
* **MV:** Метавселенной, таблица в базе данных.
* **Реклама:** Active Directory
* **AAD:** Azure Active Directory

### <a name="synchronization-steps"></a>**Этапы синхронизации**

* Импорт из Active Directory: Active Directory объекты переносятся в AD CS.

* Импорт из AAD: Azure Active Directory объекты передаются в AAD CS.

* Синхронизация: **правила входящей синхронизации** и **Исходящие правила синхронизации** выполняются в порядке приоритета от меньшего к большему. Чтобы просмотреть правила синхронизации, можно перейти в **Редактор правил синхронизации** из приложений рабочего стола. **Правила входящей синхронизации** перемещают данные из CS в MV. **Правила исходящей синхронизации** перемещают данные из MV в CS.

* Экспорт в AD: после выполнения синхронизации объекты экспортируются из AD CS в **Active Directory**.

* Экспорт в AAD: после выполнения синхронизации объекты экспортируются из AAD CS в **Azure Active Directory**.

### <a name="step-by-step-investigation"></a>**Пошаговое исследование**

* Мы начнем поиск из **метавселенной** и рассмотрим сопоставление атрибутов между источником и целевым объектом.

* Запустите **Synchronization Service Manager** из приложений на рабочем столе, как показано ниже:

  ![Запуск средства Synchronization Service Manager](media/tshoot-connect-attribute-not-syncing/tshoot-connect-attribute-not-syncing/startmenu.png)

* В **Synchronization Service Manager** щелкните **Metaverse Search** (Поиск в метавселенной), **Scope by Object Type** (Определить область по типу объекта), выберите объект с помощью атрибута, а затем нажмите кнопку **Поиск**.

  ![Параметр поиска в метавселенной](media/tshoot-connect-attribute-not-syncing/tshoot-connect-attribute-not-syncing/mvsearch.png)

* Дважды щелкните объект, обнаруженный в результате поиска **в метавселенной**, чтобы просмотреть его атрибуты. Вы можете щелкнуть вкладку **Соединители**, чтобы просмотреть соответствующий объект во всех **пространствах соединителей**.

  ![Соединители объекта метавселенной](media/tshoot-connect-attribute-not-syncing/tshoot-connect-attribute-not-syncing/mvattributes.png)

* Дважды щелкните **соединитель Active Directory**, чтобы просмотреть атрибуты **пространства соединителя**. Нажмите кнопку **Предварительный просмотр**. В открывшемся диалоговом окне нажмите кнопку **Generate Preview** (Создать предварительный просмотр).

  ![Снимок экрана, на котором показан экран свойств объекта пространства соединителя с выделенной кнопкой "предварительный просмотр".](media/tshoot-connect-attribute-not-syncing/tshoot-connect-attribute-not-syncing/csattributes.png)

* Теперь щелкните **Import Attribute Flow** (Поток атрибутов импорта). Этот параметр позволяет просмотреть поток атрибутов из **пространства соединителя Active Directory** в **метавселенную**. В столбце **Sync Rule** (Правило синхронизации) показано, какое **правило синхронизации** влияет на этот атрибут. В столбце **Источник данных** показаны атрибуты из **пространства соединителя**. Столбец **Metaverse Attribute** (Атрибут метавселенной) содержит атрибуты, находящиеся в **метавселенной**. В нем вы можете найти атрибут, который не синхронизируется. Если вам не удается найти атрибут в этом столбце, это значит, что он не сопоставлен. Чтобы сопоставить этот атрибут, вам необходимо создать настраиваемое **правило синхронизации**.

  ![Атрибуты пространства соединителя](media/tshoot-connect-attribute-not-syncing/tshoot-connect-attribute-not-syncing/cstomvattributeflow.png)

* Щелкните **Export Attribute Flow** (Поток атрибутов экспорта) в области слева, чтобы просмотреть обратный поток атрибутов из **метавселенной** в **пространство соединителя Active Directory**, использующий **правила исходящей синхронизации**.

  ![Снимок экрана, показывающий поток атрибутов из метавселенной обратно в Active Directory пространство соединителя с использованием правил исходящей синхронизации.](media/tshoot-connect-attribute-not-syncing/tshoot-connect-attribute-not-syncing/mvtocsattributeflow.png)

* Аналогичным образом вы можете просмотреть объект **пространства соединителя Azure Active Directory** и создать **предварительный просмотр**, чтобы просмотреть поток атрибутов из **метавселенной** в **пространство соединителя** и обратно. Таким образом вы можете выяснить, почему атрибут не синхронизируется.

## <a name="recommended-documents"></a>**Рекомендуемые документы**
* [Синхронизация Azure AD Connect: технические понятия](./how-to-connect-sync-technical-concepts.md)
* [Синхронизация Azure AD Connect: общие сведения об архитектуре](./concept-azure-ad-connect-sync-architecture.md)
* [Служба синхронизации Azure AD Connect: общие сведения о декларативной подготовке](./concept-azure-ad-connect-sync-declarative-provisioning.md)
* [Служба синхронизации Azure AD Connect: общие сведения о выражениях декларативной подготовки](./concept-azure-ad-connect-sync-declarative-provisioning-expressions.md)
* [Службы синхронизации Azure AD Connect: общие сведения о конфигурации по умолчанию](./concept-azure-ad-connect-sync-default-configuration.md)
* [Синхронизация Azure AD Connect: общие сведения о пользователях, группах и контактах](./concept-azure-ad-connect-sync-user-and-contacts.md)
* [Синхронизация Azure AD Connect: теневые атрибуты](./how-to-connect-syncservice-shadow-attributes.md)

## <a name="next-steps"></a>Дальнейшие действия

- [Синхронизация Azure AD Connect](how-to-connect-sync-whatis.md).
- [Что такое Гибридная идентификация?](whatis-hybrid-identity.md)