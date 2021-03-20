---
title: Уровни служб и компоненты реестра
description: Сведения о функциях и ограничениях (квотах) на уровнях служб "базовый", "Стандартный" и "Премиум" для реестра контейнеров Azure.
ms.topic: article
ms.date: 05/18/2020
ms.openlocfilehash: b55fc16dcd6dcb544ed4597ce4bdc6ba17b52646
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98015674"
---
# <a name="azure-container-registry-service-tiers"></a>Уровни службы Реестра контейнеров Azure

Служба "Реестр контейнеров Azure" доступна на нескольких уровнях служб, известных также как номера SKU. Эти уровни предоставляют прогнозируемые цены и несколько вариантов, относящихся к согласованию с емкостью и шаблонами потребления частного реестра Docker в Azure.

| Уровень | Описание |
| --- | ----------- |
| **Основной** | Недорогой начальный уровень для разработчиков, изучающих реестр контейнеров Azure. Реестры уровня "Базовый" располагают теми же программными возможностями, что и реестры уровней "Стандартный" и "Премиум" (как, например, [интеграция аутентификации Azure Active Directory](container-registry-authentication.md#individual-login-with-azure-ad), [удаление образов][container-registry-delete] и [веб-перехватчики][container-registry-webhook]). Тем не менее поставляемые хранилище и пропускная способность образов подходят для сценариев с меньшим коэффициентом использования. |
| **Standard Edition** | Реестры уровня "Стандартный" предоставляют те же возможности, что и реестры уровня "Базовый", но больший размер хранилища и более высокую пропускную способность образов. Реестры уровня "Стандартный" подходят для большинства рабочих сценариев. |
| **Премиальный** | В реестрах уровня "Премиум" обеспечивается наивысший объем хранилища и количество параллельных операций, что делает возможными сценарии с большим объемом данных. Помимо повышения пропускной способности образа, на уровне "Премиум" доступны дополнительные функции, как [георепликация][container-registry-geo-replication] для управления одним реестром в нескольких регионах, [доверие к содержимому](container-registry-content-trust.md) для подписывания тега образа, [связь с частными конечными точками](container-registry-private-link.md) для ограничения доступа к реестру. |

Уровни "Базовый", "Стандартный" и "Премиум" предоставляют одни и те же программные возможности. Кроме того, все преимущества [хранилища образов][container-registry-storage] полностью управляются службой Azure. Выбрав более высокий уровень служб, можно получить больший уровень производительности и масштабируемости. При наличии нескольких уровней служб можно начать работу с "Базового" уровня служб, а затем перейти к уровню "Стандартный" и "Премиум" по мере увеличения потребления реестра.

## <a name="service-tier-features-and-limits"></a>Функции и ограничения уровня служб

В следующей таблице описаны возможности и ограничения для уровней служб "базовый", "Стандартный" и "Премиум".

[!INCLUDE [container-instances-limits](../../includes/container-registry-limits.md)]

## <a name="changing-tiers"></a>Изменение уровней

Уровень служб реестра можно изменить с помощью Azure CLI или на портале Azure. Можно свободно перемещаться между уровнями, если уровень, на который вы переключаете, имеет требуемую максимальную емкость хранилища. 

В реестре нет времени простоя или влияния на операции с реестром при переходе между уровнями служб.

### <a name="azure-cli"></a>Azure CLI

Для перемещения между уровнями в Azure CLI используйте команду [az acr update][az-acr-update]. Например, чтобы перейти на уровень обслуживания "Премиум":

```azurecli
az acr update --name myregistry --sku Premium
```

### <a name="azure-portal"></a>Портал Azure

В колонке **Обзор** реестра контейнеров на портале Azure выберите **Обновить**, а затем выберите новый **номер SKU** из раскрывающегося списка номеров SKU.

![Обновление номера SKU реестра контейнеров на портале Azure][update-registry-sku]

## <a name="pricing"></a>Цены

Сведения о ценах на каждый уровень служб Реестра контейнеров Azure см. в статье [Цены на Реестр контейнеров][container-registry-pricing].

Дополнительные сведения о стоимости передачи данных см. на [странице цен на пропускную способность](https://azure.microsoft.com/pricing/details/bandwidth/). 

## <a name="next-steps"></a>Дальнейшие действия

**Схема развития реестра контейнеров Azure**

См. [схему развития ACR][acr-roadmap] в GitHub, чтобы найти сведения о новых возможностях в службе.

**UserVoice реестра контейнеров Azure**

Отправляйте свои предложения и голосуйте за предложения функций на [форуме UserVoice для ACR][container-registry-uservoice].

<!-- IMAGES -->
[update-registry-sku]: ./media/container-registry-skus/update-registry-sku.png

<!-- LINKS - External -->
[acr-roadmap]: https://aka.ms/acr/roadmap
[container-registry-pricing]: https://azure.microsoft.com/pricing/details/container-registry/
[container-registry-uservoice]: https://feedback.azure.com/forums/903958-azure-container-registry

<!-- LINKS - Internal -->
[az-acr-update]: /cli/azure/acr#az-acr-update
[container-registry-geo-replication]: container-registry-geo-replication.md
[container-registry-storage]: container-registry-storage.md
[container-registry-delete]: container-registry-delete.md
[container-registry-webhook]: container-registry-webhook.md