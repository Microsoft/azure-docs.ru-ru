---
title: Изменение версии SQL Server на месте
description: Узнайте, как изменить версию SQL Server виртуальной машины в Azure.
services: virtual-machines-windows
documentationcenter: na
author: ramakoni1
manager: ramakoni1
tags: azure-resource-manager
ms.service: virtual-machines-sql
ms.subservice: management
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 06/08/2020
ms.author: RamaKoni
ms.reviewer: sqlblt, daleche
ms.custom: seo-lt-2019
ms.openlocfilehash: 5dbd5725c666afe04d57d4432f0a4798fcb34c03
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97358849"
---
# <a name="in-place-change-of-sql-server-version-on-azure-vm"></a>Изменение версии SQL Server на месте на виртуальной машине Azure

[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

В этой статье описывается, как изменить версию Microsoft SQL Server на виртуальной машине Windows в Microsoft Azure.

## <a name="prerequisites"></a>Предварительные условия

Чтобы выполнить обновление на месте SQL Server, применяются следующие условия.

- Требуется установочный носитель требуемой версии SQL Server. Клиенты, у которых есть программа [Software Assurance](https://www.microsoft.com/licensing/licensing-programs/software-assurance-default), установочный носитель могут получить в [Центре корпоративного лицензирования](https://www.microsoft.com/Licensing/servicecenter/default.aspx). Клиенты, у которых нет программы Software Assurance, могут использовать установочный носитель из Azure Marketplace SQL Server образ виртуальной машины с более поздней версией SQL Server (обычно находится в К:\склсерверфулл).
- Обновления выпуска должны соответствовать [путям обновления поддержки](/sql/database-engine/install-windows/supported-version-and-edition-upgrades-version-15).

## <a name="planning-for-version-change"></a>Планирование изменения версии

Перед изменением версии рекомендуется ознакомиться со следующими элементами:

1. Проверьте новые возможности в версии, до которой вы планируете обновить:

   - Новые возможности [SQL 2019](/sql/sql-server/what-s-new-in-sql-server-ver15)
   - Новые возможности [SQL 2017](/sql/sql-server/what-s-new-in-sql-server-2017)
   - Новые возможности [SQL 2016](/sql/sql-server/what-s-new-in-sql-server-2016)


1. Рекомендуется проверить [сертификат совместимости](/sql/database-engine/install-windows/compatibility-certification) для версии, которую вы собираетесь изменить, чтобы можно было использовать режимы совместимости базы данных для снижения влияния обновления.
1. Чтобы обеспечить успешный результат, можно обратиться к следующим статьям:

   - [Видео: модернизации SQL Server | PAM Лахауд & Педро Лопес | 20 лет PASS](https://www.youtube.com/watch?v=5RPkuQHcxxs&feature=youtu.be)
   - [Database Experimentation Assistant для тестирования адресной книги](/sql/dea/database-experimentation-assistant-overview)
   - [Обновление баз данных с помощью помощника по настройке запросов](/sql/relational-databases/performance/upgrade-dbcompat-using-qta)
   - [Изменение уровня совместимости базы данных и использование хранилища запросов](/sql/database-engine/install-windows/change-the-database-compatibility-mode-and-use-the-query-store)

## <a name="upgrade-sql-version"></a>Обновление версии SQL

> [!WARNING]
> Обновление версии SQL Server приведет к перезапуску службы для SQL Server в дополнение к любым связанным службам, таким как службы Analysis Services и R.

Чтобы обновить версию SQL Server, получите установочный носитель SQL Server для более поздней версии, которая [поддерживает путь обновления](/sql/database-engine/install-windows/supported-version-and-edition-upgrades-version-15) SQL Server, и выполните следующие действия.

1. Перед началом процесса создайте резервную копию баз данных, включая систему (кроме tempdb) и пользовательские базы данных. Вы также можете создать для приложения резервную копию на уровне виртуальной машины с помощью служб Azure Backup Services.
1. Запустите Setup.exe с установочного носителя SQL Server.
1. Мастер установки запускает центр установки SQL Server. Чтобы обновить существующий экземпляр SQL Server, выберите **установить** на панели навигации, а затем выберите **обновление с более ранней версии SQL Server**.

   :::image type="content" source="./media/change-sql-server-version/upgrade.png" alt-text="Выбор для обновления версии SQL Server":::

1. На странице **ключ продукта** выберите параметр, чтобы указать, выполняется ли обновление до бесплатного выпуска SQL Server или имеется ключ PID для рабочей версии продукта. Дополнительные сведения см. в статьях [выпуски и поддерживаемые функции SQL Server 2019 (15. x)](/sql/sql-server/editions-and-components-of-sql-server-version-15) и [поддерживаемые обновления версий и выпусков (SQL Server 2016)](/sql/database-engine/install-windows/supported-version-and-edition-upgrades).
1. Нажмите кнопку **Далее** , чтобы перейти на страницу **все готово к обновлению** , а затем выберите **Обновить**. В течение нескольких минут окно программы установки может перестать отвечать на запросы, пока изменения вступают в действие. На **полной** странице будет подтверждено, что обновление завершено. Пошаговые инструкции по обновлению см. [в полной процедуре](/sql/database-engine/install-windows/upgrade-sql-server-using-the-installation-wizard-setup#procedure).

   :::image type="content" source="./media/change-sql-server-version/complete-page.png" alt-text="Завершить страницу":::

Если вы изменили SQL Server Edition в дополнение к изменению версии, также обновите выпуск и ознакомьтесь с разделом **Проверка версии и выпуска в портале** , чтобы изменить экземпляр ВИРТУАЛЬНОЙ машины SQL.

   :::image type="content" source="./media/change-sql-server-version/change-portal.png" alt-text="Изменение метаданных версии":::

## <a name="downgrade-the-version-of-sql-server"></a>Понизить версию SQL Server

Чтобы понизить версию SQL Server, необходимо полностью удалить SQL Server и повторно установить ее с использованием нужной версии. Это похоже на новую установку SQL Server так как вы не сможете восстановить предыдущую базу данных из более поздней версии до недавно установленной более ранней версии. Базы данных придется создавать заново с нуля. Если вы также изменили выпуск SQL Server во время обновления, измените свойство **Edition** виртуальной машины SQL Server в портал Azure на новое значение выпуска. При этом обновляются метаданные и выставление счетов, связанные с этой виртуальной машиной.

> [!WARNING]
> Переход на более раннюю версию SQL Server не поддерживается.

Версию SQL Server можно понизить, выполнив следующие действия.

1. Убедитесь, что вы не используете какие-либо функции, [доступные только в более поздней версии](https://social.technet.microsoft.com/wiki/contents/articles/24222.find-enterprise-only-features-in-your-database.aspx).
1. Создайте резервную копию всех баз данных, включая систему (кроме базы данных tempdb) и пользовательские базы.
1. Экспортируйте все необходимые объекты уровня сервера (такие как триггеры сервера, роли, имена входа, связанные серверы, задания, учетные данные и сертификаты).
1. Если у вас нет скриптов для повторного создания пользовательских баз данных в более ранней версии, необходимо создать скрипты для всех объектов и экспортировать все данные с помощью служб BCP.exe, SSIS или DACPAC.

   Убедитесь, что выбраны правильные параметры при создании скрипта таких элементов, как Целевая версия, зависимые объекты и дополнительные параметры.

   :::image type="content" source="./media/change-sql-server-version/scripting-options.png" alt-text="Параметры создания скриптов":::

1. Полностью удалите SQL Server и все связанные службы.
1. Перезапустите виртуальную машину.
1. Установите SQL Server, используя носитель для требуемой версии программы.
1. Установите актуальные пакеты обновления и накопительные обновления.
1. Импортируйте все необходимые объекты уровня сервера (которые были экспортированы на шаге 3).
1. Повторно создайте все необходимые пользовательские базы данных с нуля (с помощью созданных скриптов или файлов из шага 4).

## <a name="verify-the-version-and-edition-in-the-portal"></a>Проверка версии и выпуска на портале

После изменения версии SQL Server снова Зарегистрируйте SQL Serverную виртуальную машину с [расширением агента IaaS SQL](sql-agent-extension-manually-register-single-vm.md) , чтобы можно было использовать портал Azure для просмотра версии SQL Server. В указанном номере версии теперь будет отражена только что обновленная версия и выпуск SQL Server установки.

:::image type="content" source="./media/change-sql-server-version/verify-portal.png" alt-text="Проверить версию":::

> [!NOTE]
> Если вы уже зарегистрировались в расширении агента IaaS SQL, [Отмените регистрацию в RP](sql-agent-extension-manually-register-single-vm.md#unregister-from-extension) и снова [зарегистрируйте ресурс виртуальной машины SQL](sql-agent-extension-manually-register-single-vm.md#register-with-extension) , чтобы он обнаружил правильную версию и выпуск SQL Server, установленных на виртуальной машине. При этом обновляются метаданные и сведения о выставлении счетов, связанные с этой виртуальной машиной.

## <a name="remarks"></a>Remarks

- Рекомендуется инициировать резервное копирование, обновление статистики, перестроение индексов и проверку согласованности после завершения обновления. Кроме того, можно проверить уровни совместимости отдельных баз данных, чтобы убедиться, что они соответствуют нужному уровню.
- После обновления SQL Server на виртуальной машине убедитесь, что свойство **edition** SQL Server в портал Azure соответствует номеру установленного выпуска для выставления счетов.
- Возможность [изменения выпуска](change-sql-server-edition.md#change-edition-in-portal) является компонентом расширения агента IaaS SQL. Развертывание образа Azure Marketplace с помощью портал Azure автоматически регистрирует SQL Server виртуальную машину с расширением. Тем не менее клиенты, которые самостоятельно устанавливают SQL Server, должны вручную [зарегистрировать свои виртуальные машины SQL Server](sql-agent-extension-manually-register-single-vm.md).
- Если удалить ресурс виртуальной машины SQL Server, будет восстановлен параметр жестко запрограммированного выпуска образа.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в следующих статьях:

- [Обзор SQL Server на виртуальных машинах Windows](sql-server-on-azure-vm-iaas-what-is-overview.md).
- [Вопросы и ответы об SQL Server на виртуальных машинах Windows](frequently-asked-questions-faq.md)
- [Руководство по ценам для виртуальных машин SQL Server в Azure](pricing-guidance.md)
- [Заметки о выпуске SQL Server на виртуальных машинах Windows](doc-changes-updates-release-notes.md)