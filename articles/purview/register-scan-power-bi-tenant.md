---
title: Регистрация и сканирование клиента Power BI (Предварительная версия)
description: Узнайте, как использовать портал Azure зрения для регистрации и сканирования клиента Power BI.
author: chanuengg
ms.author: csugunan
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 11/19/2020
ms.openlocfilehash: 2ecc5df9db51bb6c923b9e0f47163e492bd76cfa
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101695756"
---
# <a name="register-and-scan-a-power-bi-tenant-preview"></a>Регистрация и сканирование клиента Power BI (Предварительная версия)

В этой статье показано, как использовать портал Azure зрения для регистрации и сканирования клиента Power BI.

> [!Note]
> Если экземпляр зрения и клиент Power BI находятся в одном клиенте Azure, для настройки проверки клиента Power BI можно использовать только проверку подлинности управляемого удостоверения (MSI). 

## <a name="create-a-security-group-for-permissions"></a>Создание группы безопасности для разрешений

Чтобы настроить проверку подлинности, создайте группу безопасности и добавьте в нее управляемое удостоверение зрения.

1. В [портал Azure](https://portal.azure.com)найдите **Azure Active Directory**.
1. Создайте новую группу безопасности в Azure Active Directory, выполнив инструкции по [созданию базовой группы и добавив члены с помощью Azure Active Directory](../active-directory/fundamentals/active-directory-groups-create-azure-portal.md).

    > [!Tip]
    > Этот шаг можно пропустить, если у вас уже есть группа безопасности, которую вы хотите использовать.

1. Выберите **Безопасность** в качестве **типа группы**.

    :::image type="content" source="./media/setup-power-bi-scan-PowerShell/security-group.png" alt-text="Тип группы безопасности":::

1. Добавьте управляемое удостоверение зрения в эту группу безопасности. Выберите **элементы**, а затем щелкните **+ Добавить членов**.

    :::image type="content" source="./media/setup-power-bi-scan-PowerShell/add-group-member.png" alt-text="Добавьте управляемый экземпляр каталога в группу.":::

1. Найдите управляемое удостоверение зрения и выберите его.

    :::image type="content" source="./media/setup-power-bi-scan-PowerShell/add-catalog-to-group-by-search.png" alt-text="Добавьте каталог, выполнив поиск":::

    Вы должны увидеть уведомление об успешном выполнении, показывающее, что оно было добавлено.

    :::image type="content" source="./media/setup-power-bi-scan-PowerShell/success-add-catalog-msi.png" alt-text="Добавление MSI каталога успешно завершено":::

## <a name="associate-the-security-group-with-the-tenant"></a>Связывание группы безопасности с клиентом

1. Войдите на [портал администрирования Power BI](https://app.powerbi.com/admin-portal/tenantSettings).
1. Выберите страницу **Параметры клиента** .

    > [!Important]
    > Для просмотра страницы параметры клиента необходимо быть администратором Power BI.

1. Выберите **Параметры API администратора**  >  **Разрешить субъектам-службам использовать интерфейсы API администратора Power BI только для чтения (Предварительная версия)**.
1. Выберите **определенные группы безопасности**.

    :::image type="content" source="./media/setup-power-bi-scan-PowerShell/allow-service-principals-power-bi-admin.png" alt-text="Изображение, показывающее, как разрешить субъектам-службам получать разрешения API администратора Power BI только для чтения":::

    > [!Caution]
    > Если вы разрешаете созданной группе безопасности (с управляемым удостоверением зрения в качестве члена) использовать API-интерфейсы администрирования Power BI только для чтения, вы также можете получить доступ к метаданным (например, к панелям мониторинга и именам отчетов, владельцам, описаниям и т. д.) для всех артефактов Power BI в этом клиенте. После получения метаданных в Azure зрения, разрешения зрения, а не Power BI разрешения, определите, кто может просматривать эти метаданные.

    > [!Note]
    > Вы можете удалить группу безопасности из параметров разработчика, но ранее извлеченные метаданные не будут удалены из учетной записи зрения. При необходимости его можно удалить отдельно.

## <a name="register-your-power-bi-and-set-up-a-scan"></a>Регистрация Power BI и Настройка проверки

Теперь, когда вы предоставили разрешения управляемой идентификации зрения для подключения к API администратора вашего клиента Power BI, вы можете настроить проверку из Azure зрения Studio.

Сначала добавьте специальный флаг функции в URL-адрес зрения. 

1. Щелкните значок **центра управления** .

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/management-center.png" alt-text="Значок центра управления.":::

1. Затем выберите **+ создать** в **источниках данных**.

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/data-sources.png" alt-text="Изображение кнопки &quot;создать источник данных&quot;":::

    Выберите **Power BI** в качестве источника данных.

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/select-power-bi-data-source.png" alt-text="Изображение, показывающее список доступных источников данных":::

3. Присвойте экземпляру Power BI понятное имя.

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/power-bi-friendly-name.png" alt-text="Изображение, показывающее понятное имя источника данных Power BI":::

    Длина имени должна составлять от 3-63 символов и должна содержать только буквы, цифры, символы подчеркивания и дефисы.  Пробелы не допускаются.

    По умолчанию система обнаружит Power BI клиент, который существует в той же подписке Azure.

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/power-bi-datasource-registered.png" alt-text="Power BI зарегистрированный источник данных":::

    > [!Note]
    > Для Power BI регистрация и сканирование источника данных разрешены только для одного экземпляра.


4. Укажите имя для проверки. Затем выберите параметр, чтобы включить или исключить личные рабочие области. Обратите внимание, что единственным поддерживаемым методом проверки подлинности является **управляемое удостоверение**.

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/power-bi-scan-setup.png" alt-text="Изображение, показывающее настройку сканирования Power BI":::

    > [!Note]
    > * При переключении конфигурации проверки на включение или исключение личной рабочей области будет запущена полная проверка источника PowerBI.
    > * Имя проверки должно иметь длину от 3-63 символов и должно содержать только буквы, цифры, символы подчеркивания и дефисы. Пробелы не допускаются.

5. Настройка триггера сканирования. Варианты доступны **один раз**, **каждые 7 дней** и **каждые 30 дней**.

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/scan-trigger.png" alt-text="Изображение триггера сканирования":::

6. На **Просмотр новой проверки** выберите **сохранить и запустить** , чтобы запустить проверку.

    :::image type="content" source="media/setup-power-bi-scan-catalog-portal/save-run-power-bi-scan.png" alt-text="Сохранить и запустить Power BI изображение экрана":::

## <a name="next-steps"></a>Дальнейшие действия

- [Обзор каталога данных Azure Purview](how-to-browse-catalog.md)
- [Поиск по каталогу данных Azure Purview](how-to-search-catalog.md)
