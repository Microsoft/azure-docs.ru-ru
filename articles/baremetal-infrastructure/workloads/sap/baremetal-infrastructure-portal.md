---
title: Единицы экземпляра BareMetal в Azure
description: Узнайте, как обнаруживать единицы экземпляра BareMetal и взаимодействовать с ними с помощью портал Azure.
ms.topic: how-to
ms.date: 02/17/2021
ms.openlocfilehash: 076e84473a7d067712625dd12a2d5cae42bfa91a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100548171"
---
# <a name="manage-baremetal-instances-through-the-azure-portal"></a>Управление экземплярами BareMetal на портале Azure
 
В этой статье показано, как [портал Azure](https://portal.azure.com/) отображает [экземпляры BareMetal](baremetal-overview-architecture.md). В этой статье также показаны действия, которые можно выполнить в портал Azure с развернутыми единицами экземпляра BareMetal. 
 
## <a name="register-the-resource-provider"></a>Регистрация поставщика ресурсов
Поставщик ресурсов Azure для экземпляров BareMetal обеспечивает видимость экземпляров в портал Azure, в настоящее время в общедоступной предварительной версии. По умолчанию в подписке Azure, используемой для развертываний экземпляров BareMetal, регистрируется поставщик ресурсов *бареметалинфраструктуре* . Если вы не видите развернутых единиц экземпляра BareMetal, необходимо зарегистрировать поставщик ресурсов в подписке. 

Поставщик ресурсов экземпляра BareMetal можно зарегистрировать с помощью портал Azure или Azure CLI.

### <a name="portal"></a>[Портал](#tab/azure-portal)
 
Вам потребуется получить список подписок в портал Azure а затем дважды щелкнуть подписку, используемую для развертывания единиц экземпляра BareMetal.
 
1. Войдите на [портал Azure](https://portal.azure.com).

1. В меню портала Azure выберите **Все службы**.

1. В поле **Все службы** введите **подписка**, а затем выберите **Подписки**.

1. Выберите подписку из списка подписок для просмотра.

1. Выберите **поставщики ресурсов** и введите **бареметалинфраструктуре** в поле поиска. Поставщик ресурсов должен быть **зарегистрирован**, как показано на рисунке.
 
>[!NOTE]
>Если поставщик ресурсов не зарегистрирован, выберите **Зарегистрировать**.
 
:::image type="content" source="media/baremetal-infrastructure-portal/register-resource-provider-azure-portal.png" alt-text="Снимок экрана, на котором показана зарегистрированная единица экземпляра BareMetal":::

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы начать работу с Azure CLI:

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../../../includes/azure-cli-prepare-your-environment-no-header.md)]

Войдите в подписку Azure, которая используется для развертывания экземпляра BareMetal, с помощью Azure CLI. Зарегистрируйте поставщик ресурсов `BareMetalInfrastructure` с помощью команды [az provider register](/cli/azure/provider#az_provider_register):

```azurecli
az provider register --namespace Microsoft.BareMetalInfrastructure
```

Для просмотра всех доступных поставщиков можно использовать команду [az provider list](/cli/azure/provider#az_provider_list).

---

Дополнительные сведения о поставщиках ресурсов см. в статье [Поставщики и типы ресурсов Azure](../../../azure-resource-manager/management/resource-providers-and-types.md).

## <a name="baremetal-instance-units-in-the-azure-portal"></a>Единицы экземпляра BareMetal в портал Azure
 
При отправке запроса на развертывание экземпляра BareMetal вы указываете подписку Azure, которую вы подключаете к экземплярам BareMetal. Используйте ту же подписку, которая используется для развертывания уровня приложения, который работает с единицами экземпляра BareMetal.
 
Во время развертывания экземпляров BareMetal Новая [Группа ресурсов Azure](../../../azure-resource-manager/management/manage-resources-portal.md) создается в подписке Azure, которая использовалась в запросе на развертывание. В этой новой группе ресурсов перечислены все единицы экземпляра BareMetal, развернутые в конкретной подписке.

### <a name="portal"></a>[Портал](#tab/azure-portal)

1. В подписке BareMetal в портал Azure выберите **группы ресурсов**.
 
   :::image type="content" source="media/baremetal-infrastructure-portal/view-baremetal-instance-units-azure-portal.png" alt-text="Снимок экрана, на котором показан список групп ресурсов":::

1. В списке выберите новую группу ресурсов.
 
   :::image type="content" source="media/baremetal-infrastructure-portal/filter-resource-groups.png" alt-text="Снимок экрана, на котором показана единица экземпляра BareMetal в отфильтрованном списке групп ресурсов" lightbox="media/baremetal-infrastructure-portal/filter-resource-groups.png":::
   
   >[!TIP]
   >Вы можете выполнить фильтрацию по подписке, которая использовалась для развертывания экземпляра BareMetal. После фильтрации по подходящей подписке может существовать длинный список групп ресурсов. Найдите его с помощью действия после исправления **-ткскскс** , где XXX — три цифры, например **-T250**.

1. Выберите новую группу ресурсов, чтобы просмотреть сведения о ней. На изображении показана одна единица экземпляра BareMetal.
   
   >[!NOTE]
   >Если вы развернули несколько клиентов экземпляра BareMetal в одной подписке Azure, вы увидите несколько групп ресурсов Azure.

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы просмотреть все экземпляры BareMetal, выполните команду [AZ бареметалинстанце List](/cli/azure/ext/baremetal-infrastructure/baremetalinstance#ext_baremetal_infrastructure_az_baremetalinstance_list) для группы ресурсов:

```azurecli
az baremetalinstance list --resource-group DSM05A-T550 –output table
```

> [!TIP]
> Параметр `--output` является глобальным, то есть доступным для всех команд. Значение **table** позволяет получить выходные данные в удобном формате. Дополнительные сведения см. в статье [Форматы выходных данных для команд Azure CLI](/cli/azure/format-output-azure-cli).

---

## <a name="view-the-attributes-of-a-single-instance"></a>Просмотр атрибутов одного экземпляра

Вы можете просмотреть сведения об одном единице.

### <a name="portal"></a>[Портал](#tab/azure-portal)

В списке экземпляра BareMetal выберите единственный экземпляр, который необходимо просмотреть.
 
:::image type="content" source="media/baremetal-infrastructure-portal/view-attributes-single-baremetal-instance.png" alt-text="Снимок экрана, на котором показаны атрибуты единицы экземпляра BareMetal одного экземпляра" lightbox="media/baremetal-infrastructure-portal/view-attributes-single-baremetal-instance.png":::
 
Атрибуты в изображении не выглядят значительно иначе, чем атрибуты виртуальной машины Azure (ВМ). Слева вы увидите группу ресурсов, регион Azure и имя и идентификатор подписки. Если вы назначили теги, вы также увидите их здесь. По умолчанию единицы экземпляра BareMetal не имеют назначенных тегов.
 
Справа вы увидите имя единицы, операционную систему (ОС), IP-адрес и номер SKU, которые показывают количество потоков ЦП и памяти. Вы также увидите состояние питания и версию оборудования (редакция метки экземпляра BareMetal). Состояние электропитания указывает, включена или выключена единица оборудования. Однако сведения об операционной системе не указывают, работает ли он.
 
Возможные редакции оборудования:

* Редакция 3 (редакция 3)

* Редакция 4 (редакция 4)
 
* Редакция 4,2 (версия 4,2)
 
>[!NOTE]
>Версия 4,2 — это последняя переработанная инфраструктура BareMetal, использующая существующую архитектуру версии 4. Версия 4 обеспечивает ближе близко к узлам виртуальной машины Azure. В ней значительно улучшена задержка в сети между виртуальными машинами Azure и единицами экземпляров BareMetal, развернутыми в отметках и строках версии 4. Вы можете получать доступ к экземплярам BareMetal и управлять ими с помощью портал Azure. Дополнительные сведения см. [в статье BareMetal Infrastructure in Azure](baremetal-overview-architecture.md).
 
Кроме того, с правой стороны вы найдете имя [группы размещения для службы расположения Azure](../../../virtual-machines/co-location.md) , которое создается автоматически для каждой развернутой единицы экземпляра BareMetal. Сослаться на группу размещения с учетом расположения при развертывании виртуальных машин Azure, на которых размещается уровень приложения. При использовании группы размещения с учетом расположения, связанной с единицей экземпляра BareMetal, вы гарантируете, что виртуальные машины Azure будут развернуты до единицы экземпляра BareMetal.
 
>[!TIP]
>Чтобы найти слой приложения в том же центре обработки данных Azure, что и версия 4. x, см. раздел [группы размещения службы "близость" для оптимальной сетевой задержки](../../../virtual-machines/workloads/sap/sap-proximity-placement-scenarios.md).

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы просмотреть сведения об экземпляре BareMetal, выполните команду [AZ бареметалинстанце Показать](/cli/azure/ext/baremetal-infrastructure/baremetalinstance#ext_baremetal_infrastructure_az_baremetalinstance_show) :

```azurecli
az baremetalinstance show --resource-group DSM05A-T550 --instance-name orcllabdsm01
```

Если имя экземпляра не определено, выполните `az baremetalinstance list` команду, описанную выше.

---
 
## <a name="check-activities-of-a-single-instance"></a>Проверка активности одного экземпляра
 
Вы можете проверить действия одного элемента. Одно из основных действий, записываемых, — это перезапуски единицы измерения. В список данных входят состояние действия, метка времени запуска действия, идентификатор подписки и пользователь Azure, который инициировал действие.
 
:::image type="content" source="media/baremetal-infrastructure-portal/check-activities-single-baremetal-instance.png" alt-text="Снимок экрана, на котором показаны операции с единицей измерения экземпляра BareMetal" lightbox="media/baremetal-infrastructure-portal/check-activities-single-baremetal-instance.png":::
 
Изменения в метаданных единицы измерения в Azure также записываются в журнал действий. Помимо инициированной перезагрузки, можно увидеть действие **Write бареметаллнстанцес**. Это действие не вносит изменений в саму единицу экземпляра BareMetal, но документирует изменения метаданных единицы в Azure.
 
Другое записываемое действие — это добавление или удаление [тега](../../../azure-resource-manager/management/tag-resources.md) в экземпляре.
 
## <a name="add-and-delete-an-azure-tag-to-an-instance"></a>Добавление и удаление тега Azure в экземпляре

### <a name="portal"></a>[Портал](#tab/azure-portal)
 
Вы можете добавить теги Azure в единицу экземпляра BareMetal или удалить их. Способ назначения тегов не отличается от назначения тегов для виртуальных машин. Как и в случае с виртуальными машинами, Теги существуют в метаданных Azure, а для экземпляров BareMetal они имеют те же ограничения, что и теги для виртуальных машин.
 
Удаление тегов работает так же, как с виртуальными машинами. Применение и удаление тега перечислены в журнале действий единицы экземпляра BareMetal.

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Назначение тегов для экземпляров BareMetal выполняется так же, как и для виртуальных машин. Теги существуют в метаданных Azure, а для экземпляров BareMetal они имеют те же ограничения, что и теги для виртуальных машин.

Чтобы добавить теги в единицу экземпляра BareMetal, выполните команду [AZ бареметалинстанце Update](/cli/azure/ext/baremetal-infrastructure/baremetalinstance#ext_baremetal_infrastructure_az_baremetalinstance_update) :

```azurecli
az baremetalinstance update --resource-group DSM05a-T550 --instance-name orcllabdsm01 --set tags.Dept=Finance tags.Status=Normal
```

Для удаления тега используйте ту же команду:

```azurecli
az baremetalinstance update --resource-group DSM05a-T550 --instance-name orcllabdsm01 --remove tags.Dept
```

---

## <a name="check-properties-of-an-instance"></a>Проверка свойств экземпляра
 
При получении экземпляров можно перейти к разделу свойства, чтобы просмотреть данные, собранные об экземплярах. Собранные данные включают в себя подключение Azure, серверную часть хранилища, идентификатор цепи ExpressRoute, уникальный идентификатор ресурса и идентификатор подписки. Эти сведения будут использоваться в запросах на поддержку или при настройке конфигурации моментальных снимков хранилища.
 
Еще одним важным элементом информации, который вы увидите, является IP-адрес NFS для хранилища. Он изолирует хранилище для **клиента** в стеке экземпляров BareMetal. Этот IP-адрес будет использоваться при изменении [файла конфигурации для резервных копий моментальных снимков хранилища](../../../virtual-machines/workloads/sap/hana-backup-restore.md#set-up-storage-snapshots).
 
:::image type="content" source="media/baremetal-infrastructure-portal/baremetal-instance-properties.png" alt-text="Снимок экрана, на котором показаны параметры свойства экземпляра BareMetal" lightbox="media/baremetal-infrastructure-portal/baremetal-instance-properties.png":::
 
## <a name="restart-a-unit-through-the-azure-portal"></a>Перезапустите модуль с помощью портал Azure

Существуют различные ситуации, когда ОС не завершает перезагрузку, что требует перезапуска блока BareMetal экземпляра.

### <a name="portal"></a>[Портал](#tab/azure-portal)

Вы можете выполнить перезагрузку блока питания непосредственно из портал Azure:
 
Выберите **перезапустить** , а затем **Да** , чтобы подтвердить перезагрузку единицы измерения.
 
:::image type="content" source="media/baremetal-infrastructure-portal/baremetal-instance-restart.png" alt-text="Снимок экрана, показывающий, как перезапустить единицу экземпляра BareMetal":::
 
При перезапуске единицы экземпляра BareMetal возникает задержка. В течение этой задержки состояние электропитания перемещается с **начала** до **начала**, что означает, что ОС полностью запущена. В результате после перезагрузки вы не сможете войти в модуль, как только начнется переход в состояние " **запущено**".

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы перезапустить единицу экземпляра BareMetal, используйте команду [AZ бареметалинстанце Restart](/cli/azure/ext/baremetal-infrastructure/baremetalinstance#ext_baremetal_infrastructure_az_baremetalinstance_restart) :

```azurecli
az baremetalinstance restart --resource-group DSM05a-T550 --instance-name orcllabdsm01
```

---

>[!IMPORTANT]
>В зависимости от объема памяти в единице экземпляра BareMetal перезапуск и перезагрузка оборудования и операционной системы могут занять до одного часа.
 
## <a name="open-a-support-request-for-baremetal-instances"></a>Открытие запроса в службу поддержки для экземпляров BareMetal
 
Запросы на поддержку можно отправлять специально для единицы экземпляра BareMetal.
1. В портал Azure в разделе **Справка и поддержка** создайте **[новый запрос на поддержку](https://rc.portal.azure.com/#create/Microsoft.Support)** и предоставьте следующие сведения для билета:
 
   - **Тип проблемы:** Выберите тип проблемы
 
   - **Подписка:** Выберите подписку
 
   - **Служба:** Инфраструктура BareMetal
 
   - **Ресурс:** Укажите имя экземпляра
 
   - **Сводка:** Предоставление сводной информации о запросе
 
   - **Тип проблемы:** Выберите тип проблемы
 
   - **Подтип проблемы:** Выберите подтип для проблемы

1. Перейдите на вкладку **решения** , чтобы найти решение проблемы. Если вы не можете найти решение, перейдите к следующему шагу.

1. Перейдите на вкладку **сведения** и выберите, связана ли эта ошибка с виртуальными машинами или единицами экземпляра BareMetal. Эти сведения помогают направить запрос в службу поддержки соответствующим специалистам.

1. Укажите время начала проблемы и выберите регион экземпляра.

1. Укажите дополнительные сведения о запросе и отправьте файл при необходимости.

1. Выберите **Проверка + создать** , чтобы отправить запрос.
 
Для подтверждения вашего запроса специалистом службы поддержки требуется до пяти рабочих дней.

## <a name="next-steps"></a>Дальнейшие действия

Если вы хотите узнать больше о рабочих нагрузках, см. раздел [типы рабочих нагрузок BareMetal](../../../virtual-machines/workloads/sap/get-started.md).