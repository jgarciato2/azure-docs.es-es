---
title: 'Inicio rápido: Obtención de conclusiones de imágenes mediante la API REST y Node.js: Bing Visual Search'
titleSuffix: Azure Cognitive Services
description: Aprenda a cargar una imagen con Bing Visual Search API y Node.js y, a continuación, obtener conclusiones sobre esta.
services: cognitive-services
author: swhite-msft
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-visual-search
ms.topic: quickstart
ms.date: 05/22/2020
ms.author: scottwhi
ms.custom: devx-track-js
ms.openlocfilehash: 94a642886b626eb84da3a2d02684b5dd170dcbb1
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/02/2020
ms.locfileid: "96499074"
---
# <a name="quickstart-get-image-insights-using-the-bing-visual-search-rest-api-and-nodejs"></a>Inicio rápido: Obtención de conclusiones de imágenes mediante Bing Visual Search API y Node.js

> [!WARNING]
> Las Bing Search API se mueven de Cognitive Services a Bing Search Services. A partir del **30 de octubre de 2020** las nuevas instancias de Bing Search deben aprovisionarse siguiendo el proceso documentado [aquí](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> El aprovisionamiento de las Bing Search API mediante Cognitive Services será posible durante los próximos tres años o hasta que finalice el Contrato Enterprise, lo que antes suceda.
> Puede encontrar instrucciones sobre la migración en [Bing Search Services](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Use este inicio rápido para realizar la primera llamada a la API Bing Visual Search. Esta sencilla aplicación de JavaScript carga una imagen en la API y muestra la información que se devuelve sobre ella. Aunque esta aplicación está escrita en JavaScript, la API es un servicio web RESTful compatible con la mayoría de los lenguajes de programación.

## <a name="prerequisites"></a>Prerrequisitos

* [Node.js](https://nodejs.org/en/download/)
* El módulo de solicitud para JavaScript. Puede usar el comando `npm install request` para instalar el módulo.
* El módulo de datos de formulario. Puede usar el comando `npm install form-data` para instalar el módulo. 

[!INCLUDE [cognitive-services-bing-visual-search-signup-requirements](../../../../includes/cognitive-services-bing-visual-search-signup-requirements.md)]

## <a name="initialize-the-application"></a>Inicialización de la aplicación

1. Cree un archivo de JavaScript en el editor o el IDE que prefiera y establezca los siguientes requisitos:

    ```javascript
    var request = require('request');
    var FormData = require('form-data');
    var fs = require('fs');
    ```

2. Cree variables para el punto de conexión de API, la clave de suscripción y la ruta de acceso a la imagen. Para el valor `baseUri` puede usar el punto de conexión global en el código siguiente, o el punto de conexión del [subdominio personalizado](../../../cognitive-services/cognitive-services-custom-subdomains.md) que se muestra en Azure Portal para el recurso.

    ```javascript
    var baseUri = 'https://api.cognitive.microsoft.com/bing/v7.0/images/visualsearch';
    var subscriptionKey = 'your-api-key';
    var imagePath = "path-to-your-image";
    ```

3. Cree una función denominada `requestCallback()` para imprimir la respuesta de la API.

    ```javascript
    function requestCallback(err, res, body) {
        console.log(JSON.stringify(JSON.parse(body), null, '  '))
    }
    ```

## <a name="construct-and-send-the-search-request"></a>Construcción y envío de la solicitud de búsqueda

1. Al cargar una imagen local, los datos del formulario deben incluir el encabezado `Content-Disposition`. Establezca su parámetro `name` en "image", y el parámetro `filename` en el nombre de archivo de la imagen. El contenido del formulario incluye los datos binarios de la imagen. El tamaño de imagen máximo que puede cargar es de 1 MB.

   ```
   --boundary_1234-abcd
   Content-Disposition: form-data; name="image"; filename="myimagefile.jpg"

   ÿØÿà JFIF ÖÆ68g-¤CWŸþ29ÌÄøÖ‘º«™æ±èuZiÀ)"óÓß°Î= ØJ9á+*G¦...

   --boundary_1234-abcd--
   ```

2. Cree un objeto `FormData` con `FormData()`, y anexe a él la ruta de acceso a la imagen mediante `fs.createReadStream()`.
    
    ```javascript
    var form = new FormData();
    form.append("image", fs.createReadStream(imagePath));
    ```

3. Use la biblioteca de solicitudes para cargar la imagen y llame a `requestCallback()` para imprimir la respuesta. Agregue la clave de suscripción al encabezado de solicitud.

    ```javascript
    form.getLength(function(err, length){
      if (err) {
        return requestCallback(err);
      }
      var r = request.post(baseUri, requestCallback);
      r._form = form; 
      r.setHeader('Ocp-Apim-Subscription-Key', subscriptionKey);
    });
    ```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de una aplicación web de página única de Visual Search](../tutorial-bing-visual-search-single-page-app.md)