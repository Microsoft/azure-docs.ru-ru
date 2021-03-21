---
title: включить файл
description: включить файл
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 02/08/2021
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: f3eb2d9469ab3a3d2c1d09e4adc3ee2cb1f86e6e
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99979079"
---
## <a name="extract-the-zip-file"></a>Извлечение ZIP-файла

Извлеките ZIP-файл. Файл содержит следующие папки.

* AzureVPN
* Универсальный шаблон
* Опенвпн (если вы включили параметр Опенвпн с **сертификатом Azure** или параметрами **проверки подлинности RADIUS** на шлюзе). Выберите соответствующую статью, соответствующую вашей конфигурации, чтобы создать клиент.

  * [VPN-шлюз — создание клиента](../articles/vpn-gateway/openvpn-azure-ad-tenant.md).
  * [Виртуальная глобальная сеть — создание клиента](../articles/virtual-wan/openvpn-azure-ad-tenant.md).

## <a name="retrieve-information"></a>Получение сведений

В папке **AzureVPN** перейдите к файлу **_azurevpnconfig.xml_** и откройте его в блокноте. Запишите текст между следующими тегами.

```
<audience>          </audience>
<issuer>            </issuer>
<tennant>           </tennant>
<fqdn>              </fqdn>
<serversecret>      </serversecret>
```

## <a name="profile-details"></a>Сведения о профиле

При добавлении подключения используйте сведения, собранные на предыдущем шаге, для страницы сведений о профиле. Поля соответствуют следующим сведениям.

* **Аудитория:** Определяет ресурс получателя, для которого предназначен маркер.
* **Издатель:** Определяет службу маркеров безопасности (STS), которая выдает маркер, а также клиент Azure AD.
* **Клиент:** Содержит неизменяемый уникальный идентификатор клиента каталога, который выдал маркер.
* **Полное доменное имя:** Полное доменное имя (FQDN) VPN-шлюза Azure.
* **Серверсекрет:** Общий ключ VPN-шлюза.

## <a name="folder-contents"></a>Содержимое папки

* **Универсальная папка** содержит общедоступный сертификат сервера и файл VpnSettings.xml. Файл VpnSettings.xml содержит сведения, необходимые для настройки универсального клиента.

* Скачанный ZIP-файл также может содержать папки **WindowsAmd64** и **WindowsX86**. Эти папки содержат установщик для SSTP и IKEv2 для клиентов Windows. Для их установки требуются права администратора на клиенте.
