---
title: Настройка политики авторизации ключей содержимого с помощью портала Azure | Документация Майкрософт
description: В этой статье показано, как настроить политику авторизации для ключа содержимого.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: ee82a3fa-c34b-48f2-a108-8ba321f1691e
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.openlocfilehash: 68bd4bd2472e80a663294745368fb505767a230a
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103008320"
---
# <a name="configure-a-content-key-authorization-policy"></a>Настройка политики авторизации ключей содержимого

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

[!INCLUDE [media-services-selector-content-key-auth-policy](../../../includes/media-services-selector-content-key-auth-policy.md)]

## <a name="overview"></a>Обзор
 Службы мультимедиа Azure позволяют защищать потоковое содержимое MPEG-DASH, Smooth Streaming и HTTP Live Streaming (HLS) с помощью стандарта AES (использующего 128-разрядные ключи шифрования) или технологии [ управления цифровыми правами (DRM) PlayReady](https://www.microsoft.com/playready/overview/). Кроме того, Службы мультимедиа позволяют передавать потоки MPEG DASH с шифрованием Widevine DRM. PlayReady и Widevine шифруются согласно спецификации общего шифрования (ISO/IEC 23001-7 CENC).

Службы мультимедиа также включают в себя службы доставки ключей и лицензий, с помощью которых клиенты могут получать ключи AES либо лицензии PlayReady или Widevine для воспроизведения зашифрованного содержимого.

В этой статье показано, как настроить политики авторизации ключей содержимого с помощью портала Azure. Ключ может использоваться позже для динамического шифрования содержимого. Сейчас поддерживается шифрование таких форматов потоковой передачи: HLS, MPEG-DASH и Smooth Streaming. Невозможно шифровать последовательно скачиваемые данные.

Когда проигрыватель запрашивает поток, для которого настроено динамическое шифрование, Службы мультимедиа, используя настроенный ключ, выполняют динамическое шифрование содержимого с помощью AES или DRM. Чтобы расшифровать поток, проигрыватель запросит ключ у службы доставки ключей. Чтобы определить, есть ли у пользователя право на получение ключа, служба оценивает политики авторизации, заданные для ключа.

Если вы планируете использовать несколько ключей содержимого или хотите задать URL-адрес службы доставки ключей или лицензий, отличный от адреса службы доставки ключей Служб мультимедиа, используйте пакет SDK для .NET или интерфейсы REST API для Служб мультимедиа. Дополнительные сведения см. в разделе:

* [Настройка политики авторизации ключей содержимого с помощью пакета SDK Служб мультимедиа для .NET](media-services-dotnet-configure-content-key-auth-policy.md)
* [Настройка политики авторизации ключей содержимого с помощью REST API для Служб мультимедиа](media-services-rest-configure-content-key-auth-policy.md)

### <a name="some-considerations-apply"></a>Важные особенности
* При создании учетной записи служб мультимедиа в нее добавляется конечная точка потоковой передачи по умолчанию в состоянии "Остановлена". Чтобы начать потоковую передачу содержимого и воспользоваться динамической упаковкой и динамическим шифрованием, конечная точка потоковой передачи должна находиться в состоянии "Выполняется". 
* Ресурс должен содержать набор MP4-файлов с адаптивной скоростью или файлов Smooth Streaming с адаптивной скоростью. Дополнительные сведения см. в статье о [кодировании ресурсов](media-services-encode-asset.md).
* Служба доставки ключей кэширует политику ContentKeyAuthorizationPolicy и связанные с ней объекты (параметры и ограничения политики) за 15 минут. Вы можете создать ContentKeyAuthorizationPolicy и указать ограничение авторизации по токену, протестировать его, а затем обновить политику, задав открытое ограничение. Этот процесс занимает примерно 15 минут, после чего политика переключается на открытую версию.
* Конечная точка потоковой передачи Служб мультимедиа задает значение CORS-заголовка Access-Control-Allow-Origin в предварительном ответе как подстановочный знак "\*". Это значение подходит для большинства проигрывателей, включая Проигрыватель мультимедиа Azure, Roku, JWPlayer и др. Но некоторые проигрыватели, в которых используется dash.js, при таком сценарии не работают. Если режим учетных данных имеет значение include, запрос XMLHttpRequest в dash.js не допускает использования подстановочного знака "\*" в качестве значения Access-Control-Allow-Origin. Это ограничение в dash.js можно обойти. Если вы размещаете клиент из одного домена, в службах мультимедиа Azure этот домен можно указать в заголовке предварительного ответа. Для получения помощи отправьте запрос в службу поддержки через портал Azure.

## <a name="configure-the-key-authorization-policy"></a>Настройка политики авторизации ключей
Чтобы настроить политику авторизации ключей, выберите страницу **ЗАЩИТА СОДЕРЖИМОГО** .

Службы мультимедиа поддерживают несколько способов аутентификации пользователей, которые запрашивают ключи. В политике авторизации ключей содержимого можно задать ограничения авторизации: открытое, по токену или по IP-адресу. (IP-адрес можно настроить, используя пакет SDK для .NET или REST.)

### <a name="open-restriction"></a>Ограничение открытого типа
Ограничение открытого типа означает, что система доставляет ключ всем, кто подает на него запрос. Это ограничение подходит для тестирования.

![OpenPolicy][open_policy]

### <a name="token-restriction"></a>Ограничение по маркеру
Чтобы выбрать политику ограничения по токену, нажмите кнопку **TOKEN** (Токен).

При ограничении этого типа к политике должен прилагаться токен, выданный службой токенов безопасности. Службы мультимедиа поддерживают маркеры в форматах простого веб-маркера ([SWT](/previous-versions/azure/azure-services/gg185950(v=azure.100)#BKMK_2)) и JSON Web Token (JWT). Дополнительные сведения см. в статье [JWT token Authentication in Azure Media Services and Dynamic Encryption](http://www.gtrifonov.com/2015/01/03/jwt-token-authentication-in-azure-media-services-and-dynamic-encryption/) (Проверка подлинности токена JWT в Службах мультимедиа Azure и динамическое шифрование).

Службы мультимедиа не предоставляют службу маркеров безопасности. Можно создать настраиваемую службу токенов безопасности для выдачи токенов. Чтобы создать маркер, подписанный указанным ключом, и получить утверждения, указанные в конфигурации ограничения по маркерам, должна быть настроена служба маркеров безопасности. Если маркер является допустимым и утверждения маркера соответствуют утверждениям, настроенным для ключа содержимого, служба доставки ключей для служб мультимедиа возвращает клиенту ключ шифрования.

При настройке политики ограничения по маркеру необходимо задать такие параметры, как основной ключ проверки, издатель и аудитория. Основной ключ проверки содержит ключ, которым был подписан маркер. Издателем является служба STS, которая выдала маркер. Аудитория (иногда называется областью) описывает назначение маркера или ресурс, доступ к которому обеспечивает маркер. Служба доставки ключей служб мультимедиа проверяет, соответствуют ли эти значения в маркере значениям в шаблоне.

### <a name="playready"></a>PlayReady
При защите содержимого с помощью PlayReady в политике авторизации среди прочего необходимо указать XML-строку, определяющую шаблон лицензии PlayReady. По умолчанию задана следующая политика:

```xml
<PlayReadyLicenseResponseTemplate xmlns:i="https://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/PlayReadyTemplate/v1">
  <LicenseTemplates>
    <PlayReadyLicenseTemplate><AllowTestDevices>true</AllowTestDevices>
      <ContentKey i:type="ContentEncryptionKeyFromHeader" />
      <LicenseType>Nonpersistent</LicenseType>
      <PlayRight>
        <AllowPassingVideoContentToUnknownOutput>Allowed</AllowPassingVideoContentToUnknownOutput>
      </PlayRight>
    </PlayReadyLicenseTemplate>
  </LicenseTemplates>
</PlayReadyLicenseResponseTemplate>
```

Вы можете нажать кнопку **импорта XML-файла политики** и выбрать другой XML-файл, который соответствует XML-схеме, определенной в статье [Обзор шаблонов лицензий PlayReady Служб мультимедиа](media-services-playready-license-template-overview.md).

## <a name="additional-notes"></a>Дополнительные замечания

* Widevine — это служба, которая предоставляется компанией Google Inc. и подпадает под условия предоставления услуг и политику конфиденциальности Google Inc.

## <a name="next-steps"></a>Дальнейшие действия
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

[open_policy]: ./media/media-services-portal-configure-content-key-auth-policy/media-services-protect-content-with-open-restriction.png
[token_policy]: ./media/media-services-key-authorization-policy/media-services-protect-content-with-token-restriction.png
