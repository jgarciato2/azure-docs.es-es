---
title: 'Ejemplo de script de la CLI de Azure: creación y envío de un trabajo'
description: El script de la CLI de Azure de este tema muestra cómo enviar un trabajo a una transformación de codificación sencilla mediante una dirección URL HTTPS.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: ''
ms.service: media-services
ms.devlang: azurecli
ms.topic: how-to
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: devx-track-azurecli
ms.openlocfilehash: 9291d53822f0fbb80f759908933db58f2224c3d7
ms.sourcegitcommit: 97c48e630ec22edc12a0f8e4e592d1676323d7b0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/18/2021
ms.locfileid: "101091901"
---
# <a name="cli-example-create-and-submit-a-job"></a>Ejemplo de CLI: creación y envío de un trabajo

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

En Media Services v3, cuando se envían trabajos para procesar los vídeos, es necesario indicar a Media Services dónde encontrar el vídeo de entrada. Una de las opciones es especificar una dirección URL HTTPS como entrada de trabajo (como se muestra en este ejemplo). 

## <a name="prerequisites"></a>Prerequisites 

[Cree una cuenta de Media Services](./create-account-howto.md).

## <a name="example-script"></a>Script de ejemplo

Al ejecutar `az ams job start`, puede establecer una etiqueta en la salida del trabajo. La etiqueta se puede usar posteriormente para identificar para qué sirve este recurso de salida. 

- Si asigna un valor a la etiqueta, establezca "--output-assets" en "assetname=label".
- En caso contrario, establezca "--output-assets" en "assetname=".
  Observe que agrega "=" a `output-assets`. 

```azurecli
az ams job start \
  --name testJob001 \
  --transform-name testEncodingTransform \
  --base-uri 'https://nimbuscdn-nimbuspm.streaming.mediaservices.windows.net/2b533311-b215-4409-80af-529c3e853622/' \
  --files 'Ignite-short.mp4' \
  --output-assets testOutputAssetName= \
  -a amsaccount \
  -g amsResourceGroup 
```

Recibe una respuesta similar a esta:

```
{
  "correlationData": {},
  "created": "2019-02-15T05:08:26.266104+00:00",
  "description": null,
  "id": "/subscriptions/<id>/resourceGroups/amsResourceGroup/providers/Microsoft.Media/mediaservices/amsaccount/transforms/testEncodingTransform/jobs/testJob001",
  "input": {
    "baseUri": "https://nimbuscdn-nimbuspm.streaming.mediaservices.windows.net/2b533311-b215-4409-80af-529c3e853622/",
    "files": [
      "Ignite-short.mp4"
    ],
    "label": null,
    "odatatype": "#Microsoft.Media.JobInputHttp"
  },
  "lastModified": "2019-02-15T05:08:26.266104+00:00",
  "name": "testJob001",
  "outputs": [
    {
      "assetName": "testOutputAssetName",
      "error": null,
      "label": "",
      "odatatype": "#Microsoft.Media.JobOutputAsset",
      "progress": 0,
      "state": "Queued"
    }
  ],
  "priority": "Normal",
  "resourceGroup": "amsResourceGroup",
  "state": "Queued",
  "type": "Microsoft.Media/mediaservices/transforms/jobs"
}
```

## <a name="next-steps"></a>Pasos siguientes

[az ams job (CLI)](/cli/azure/ams/job?view=azure-cli-latest)
