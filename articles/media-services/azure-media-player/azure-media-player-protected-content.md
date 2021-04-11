---
title: Защищенное содержимое Проигрывателя мультимедиа Azure
description: В настоящее время Проигрыватель мультимедиа Azure поддерживает 128-битное шифрование AES и общее зашифрованное содержимое.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: overview
ms.date: 04/05/2021
ms.custom: devx-track-js
ms.openlocfilehash: 8c04a3847ce1005b530042926efb8fbbe75f9cfe
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2021
ms.locfileid: "106448723"
---
# <a name="protected-content"></a>Защищенное содержимое #

В настоящее время Проигрыватель мультимедиа Azure поддерживает 128-битное шифрование AES и общее зашифрованное содержимое (через PlayReady и Widevine) или шифрование содержимого через FairPlay. Чтобы правильно воспроизводить защищенное содержимое, необходимо передать `protectionInfo` Проигрывателю мультимедиа Azure. Эта информация индивидуальна для каждого источника, и вы можете добавить ее прямо в теге `<source>` через `data-setup`.  Можно также добавить `protectionInfo` в виде параметра, если источник задается динамически.

`protectionInfo` принимает объект JSON и включает в себя следующее:

- для `type`: `AES`, `PlayReady`, `Widevine` или `FairPlay`.
- для `certificateUrl` предоставляется прямая ссылка на размещенный сертификат FairPlay;

- поле `authenticationToken` обозначает, нужно ли добавить незакодированный маркер проверки подлинности.

> [!IMPORTANT]
> Объект **certificateUrl** требуется только для FairPlay DRM. ***
>[!NOTE]
> Значение techOrder по умолчанию изменено с учетом новой технологии `html5FairPlayHLS`, в частности для воспроизведения содержимого FairPlay в тех браузерах, которые имеют встроенную поддержку этой возможности (Safari в OSX 8 и выше). Если у вас есть содержимое FairPlay для воспроизведения **и** в вашем приложении используется значение techOrder, отличное от варианта по умолчанию, обязательно добавьте новую технологию в объект techOrder. Мы рекомендуем расположить ее перед silverlightSS, чтобы содержимое не воспроизводилось через PlayReady.

## <a name="code-sample"></a>Пример кода ##

```html
Ex:

    <video id="vid1" class="azuremediaplayer amp-default-skin">
        <source
            src="//example/path/to/myVideo.ism/manifest"
            type="application/vnd.ms-sstr+xml"
            data-setup='{"protectionInfo": [{"type": "AES", "authenticationToken": "Bearer=urn%3amicrosoft%3aazure%3amediaservices%3acontentkeyidentifier=8130520b-c116-45a9-824e-4a0082f3cb3c&Audience=urn%3atest&ExpiresOn=1450207516&Issuer=http%3a%2f%2ftestacs.com%2f&HMACSHA256=eV7HDgZ9msp9H9bnEPGN91sBdU7XsZ9OyB6VgFhKBAU%3d"}]}'
        />
    </video>
or

```javascript
    var myPlayer = amp("vid1", /* Options */);
    myPlayer.src([{
        src: "//example/path/to/myVideo.ism/manifest",
        type: "application/vnd.ms-sstr+xml",
        protectionInfo: [{
            type: "PlayReady",
            authenticationToken: "Bearer=urn%3amicrosoft%3aazure%3amediaservices%3acontentkeyidentifier=d5646e95-63ee-4fbe-ba4e-295c8d9502e0&Audience=urn%3atest&ExpiresOn=1450222961&Issuer=http%3a%2f%2ftestacs.com%2f&HMACSHA256=4Jop3kNJdzVI8L5IZLgFtPdImyE%2fHTRil0x%2bEikSdPs%3d"
        }] }, ]
    );
```

или с несколькими DRM

```javascript
    var myPlayer = amp("vid1", /* Options */);
    myPlayer.src([{
        src: "//example/path/to/myVideo.ism/manifest",
        type: "application/vnd.ms-sstr+xml",
        protectionInfo: [{
                type: "PlayReady",
                authenticationToken: "Bearer=urn%3amicrosoft%3aazure%3amediaservices%3acontentkeyidentifier=d5646e95-63ee-4fbe-ba4e-295c8d9502e0&Audience=urn%3atest&ExpiresOn=1450222961&Issuer=http%3a%2f%2ftestacs.com%2f&HMACSHA256=4Jop3kNJdzVI8L5IZLgFtPdImyE%2fHTRil0x%2bEikSdPs%3d"
            },
            {
                type: "Widevine",
                authenticationToken: "Bearer=urn%3amicrosoft%3aazure%3amediaservices%3acontentkeyidentifier=d5646e95-63ee-4fbe-ba4e-295c8d9502e0&Audience=urn%3atest&ExpiresOn=1450222961&Issuer=http%3a%2f%2ftestacs.com%2f&HMACSHA256=4Jop3kNJdzVI8L5IZLgFtPdImyE%2fHTRil0x%2bEikSdPs%3d"
            },
            {
                   type: "FairPlay",
                  certificateUrl: "//example/path/to/myFairplay.der",
                   authenticationToken: "Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1cm46bWljcm9zb2Z0OmF6dXJlOm1lZGlhc2VydmljZXM6Y29udGVudGtleWlkZW50aWZpZXIiOiIyMTI0M2Q2OC00Yjc4LTRlNzUtYTU5MS1jZWMzMDI0NDNhYWMiLCJpc3MiOiJodHRwOi8vY29udG9zbyIsImF1ZCI6InVybjp0ZXN0IiwiZXhwIjoxNDc0NTkyNDYzLCJuYmYiOjE0NzQ1ODg1NjN9.mE7UxgNhkieMMqtM_IiYQj-FK1KKIzB6lAptw4Mi67A"
        }] } ]
    );
```

> [!NOTE]
> Не все браузеры и платформы способны воспроизводить защищенное содержимое. Дополнительные сведения о поддерживаемых возможностях см. в разделе [о технологиях воспроизведения](azure-media-player-playback-technology.md).
> [!IMPORTANT]
> Передаваемый в проигрыватель маркер используется для защищенного содержимого и только для пользователей, прошедших проверку подлинности. Предполагается, что приложение использует SSL или другой вариант мер безопасности. Также предполагается, что пользователю можно доверять и он не будет использовать маркер не по назначению. Если это не так, обсудите ситуацию с экспертами по безопасности.

## <a name="next-steps"></a>Дальнейшие действия ##

- [Краткое руководство. Проигрыватель мультимедиа Azure](azure-media-player-quickstart.md)