---
title: 'Restablecimiento del estado de canje de un usuario invitado: Azure AD'
description: Obtenga información sobre cómo restablecer el estado de canje de invitación para un usuario invitado de Azure Active Directory B2B en Azure AD External Identities.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: how-to
ms.date: 02/03/2021
ms.author: mimart
author: msmimart
manager: celestedg
ms.collection: M365-identity-device-management
ms.openlocfilehash: d843755847d074e00aec9ed2830cb873b6bb3382
ms.sourcegitcommit: d4734bc680ea221ea80fdea67859d6d32241aefc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/14/2021
ms.locfileid: "100365419"
---
# <a name="reset-redemption-status-for-a-guest-user"></a>Restablecimiento del estado de canje para un usuario invitado

Una vez que un usuario invitado ha canjeado su invitación para la colaboración B2B, puede haber ocasiones en las que tenga que actualizar la información de inicio de sesión, por ejemplo, cuando:

- El usuario quiera iniciar sesión con otro correo electrónico y proveedor de identidades.
- Se haya eliminado y vuelto a crear la cuenta del usuario en su inquilino principal.
- El usuario esté trabajando en otra empresa, pero todavía necesite el mismo acceso a los recursos.
- Las responsabilidades del usuario se hayan pasado a otro usuario.

Para administrar estos escenarios, anteriormente tenía que eliminar manualmente la cuenta del usuario invitado de su directorio y volver a invitar al usuario. Ahora puede usar PowerShell o la API de invitación de Microsoft Graph para restablecer el estado de canje del usuario y volver a invitarlo, a la vez que conserva el identificador de objeto del usuario, las pertenencias a grupos y las asignaciones de aplicaciones. Cuando el usuario canjea la invitación nueva, el UPN del usuario no cambia, aunque su nombre de inicio de sesión se cambia al nuevo correo electrónico. Posteriormente, el usuario puede iniciar sesión con el nuevo correo electrónico o con un correo electrónico que se haya agregado a la propiedad `otherMails` del objeto de usuario.

## <a name="use-powershell-to-reset-redemption-status"></a>Uso de PowerShell para restablecer el estado de canje

Instale el módulo AzureADPreview de PowerShell más reciente y cree una nueva invitación con `InvitedUserEMailAddress` establecido en la nueva dirección de correo electrónico, y `ResetRedemption` establecido en `true`.

```powershell  
Uninstall-Module AzureADPreview 
Install-Module AzureADPreview 
Connect-AzureAD 
$ADGraphUser = Get-AzureADUser -objectID "UPN of User to Reset"  
$msGraphUser = New-Object Microsoft.Open.MSGraph.Model.User -ArgumentList $ADGraphUser.ObjectId 
New-AzureADMSInvitation -InvitedUserEmailAddress <<external email>> -SendInvitationMessage $True -InviteRedirectUrl "http://myapps.microsoft.com" -InvitedUser $msGraphUser -ResetRedemption $True 
```

## <a name="use-microsoft-graph-api-to-reset-redemption-status"></a>Uso de Microsoft Graph API para restablecer el estado de canje

Con la [API de invitación de Microsoft Graph](/graph/api/resources/invitation?view=graph-rest-1.0), establezca la propiedad `resetRedemption` en `true` y especifique la nueva dirección de correo electrónico en la propiedad `invitedUserEmailAddress`.

```json
POST https://graph.microsoft.com/beta/invitations  
Authorization: Bearer eyJ0eX...  
ContentType: application/json  
{  
   "invitedUserEmailAddress": "<<external email>>",  
   "sendInvitationMessage": true,  
   "invitedUserMessageInfo": {  
      "messageLanguage": "en-US",  
      "ccRecipients": [  
         {  
            "emailAddress": {  
               "name": null,  
               "address": "<<optional additional notification email>>"  
            }  
         } 
      ],  
      "customizedMessageBody": "<<custom message>>"  
},  
"inviteRedirectUrl": "https://myapps.microsoft.com?tenantId=",  
"invitedUser": {  
   "id": "<<ID for the user you want to reset>>"  
}, 
"resetRedemption": true 
}
```

## <a name="next-steps"></a>Pasos siguientes

- [Incorporación de usuarios de colaboración B2B de Azure Active Directory con PowerShell](customize-invitation-api.md#powershell)
- [Propiedades de un usuario invitado de Azure AD B2B](user-properties.md)
