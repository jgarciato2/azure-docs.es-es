---
title: 'Inicio rápido: adición de llamadas a una aplicación de iOS mediante Azure Communication Services'
description: En este inicio rápido, aprenderá a usar la biblioteca cliente de llamadas de Azure Communication Services para iOS.
author: matthewrobertson
ms.author: marobert
ms.date: 07/24/2020
ms.topic: quickstart
ms.service: azure-communication-services
ms.openlocfilehash: 36ec27f3a0e69126a91b52bed26dc645ec89e46e
ms.sourcegitcommit: b4647f06c0953435af3cb24baaf6d15a5a761a9c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/02/2021
ms.locfileid: "101656648"
---
En este inicio rápido, aprenderá a iniciar una llamada mediante la biblioteca cliente de llamadas de Azure Communication Services para iOS.

> [!NOTE]
> En este documento se usa la versión 1.0.0-beta.8 de la biblioteca de cliente para llamadas.

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, debe cumplir los siguientes requisitos previos:

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F). 
- Mac con [Xcode](https://go.microsoft.com/fwLink/p/?LinkID=266532), junto con un certificado de desarrollador válido instalado en el Llavero.
- Un recurso de Communication Services implementado. [Cree un recurso de Communication Services](../../create-communication-resource.md).
- Un [token de acceso de usuario](../../access-tokens.md) para su instancia de Azure Communication Services.

## <a name="setting-up"></a>Instalación

### <a name="creating-the-xcode-project"></a>Creación del proyecto de Xcode

En Xcode, cree un nuevo proyecto de iOS y seleccione la plantilla **Aplicación de una vista**. En este tutorial se usa el [marco SwiftUI](https://developer.apple.com/xcode/swiftui/), por lo que debe establecer **Lenguaje** en **Swift** e **Interfaz de usuario** en **SwiftUI**. Durante este inicio rápido, no va a crear pruebas. No dude en desactivar **Incluir pruebas**.

:::image type="content" source="../media/ios/xcode-new-ios-project.png" alt-text="Captura de pantalla que muestra la ventana Nuevo proyecto en Xcode.":::

### <a name="install-the-package-and-dependencies-with-cocoapods"></a>Instalación del paquete y las dependencias con CocoaPods

1. Cree un podfile para la aplicación, de la siguiente manera:

   ```
   platform :ios, '13.0'
   use_frameworks!

   target 'AzureCommunicationCallingSample' do
     pod 'AzureCommunicationCalling', '~> 1.0.0-beta.8'
     pod 'AzureCommunication', '~> 1.0.0-beta.8'
     pod 'AzureCore', '~> 1.0.0-beta.8'
   end
   ```

2. Ejecute `pod install`.
3. Abra `.xcworkspace` con Xcode.

### <a name="request-access-to-the-microphone"></a>Solicitud de acceso al micrófono

Para acceder al micrófono del dispositivo, debe actualizar la lista de propiedades de información de la aplicación con el elemento `NSMicrophoneUsageDescription`. Puede establecer el valor asociado al elemento `string` que se incluirá en el cuadro de diálogo que el sistema usa para solicitar acceso al usuario.

Haga clic con el botón derecho en la entrada `Info.plist` del árbol del proyecto y seleccione **Abrir como** > **Código fuente**. Agregue las líneas siguientes a la sección `<dict>` de nivel superior y guarde el archivo.

```xml
<key>NSMicrophoneUsageDescription</key>
<string>Need microphone access for VOIP calling.</string>
```

### <a name="set-up-the-app-framework"></a>Instalación del marco de la aplicación

Abra el archivo **ContentView.swift** del proyecto y agregue una declaración `import` en la parte superior del archivo para importar el elemento `AzureCommunicationCalling library`. Además, importe `AVFoundation`; lo necesitaremos para la solicitud de permiso de audio en el código.

```swift
import AzureCommunicationCalling
import AVFoundation
```

Reemplace la implementación de la estructura `ContentView` por algunos controles de interfaz de usuario simples que permitan a los usuarios iniciar y finalizar una llamada. En este inicio rápido, asociaremos la lógica de negocios a estos controles.

```swift
struct ContentView: View {
    @State var callee: String = ""
    @State var callClient: CallClient?
    @State var callAgent: CallAgent?
    @State var call: Call?

    var body: some View {
        NavigationView {
            Form {
                Section {
                    TextField("Who would you like to call?", text: $callee)
                    Button(action: startCall) {
                        Text("Start Call")
                    }.disabled(callAgent == nil)
                    Button(action: endCall) {
                        Text("End Call")
                    }.disabled(call == nil)
                }
            }
            .navigationBarTitle("Calling Quickstart")
        }.onAppear {
            // Initialize call agent
        }
    }

    func startCall() {
        // Ask permissions
        AVAudioSession.sharedInstance().requestRecordPermission { (granted) in
            if granted {
                // Add start call logic
            }
        }
    }

    func endCall() {
        // Add end call logic
    }
}
```

## <a name="object-model"></a>Modelo de objetos

Las clases e interfaces siguientes controlan algunas de las características principales de la biblioteca cliente para llamadas de Azure Communication Services:

| Nombre                                  | Descripción                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| CallClient | CallClient es el punto de entrada principal a la biblioteca cliente para llamadas.|
| CallAgent | CallAgent se usa para iniciar y administrar llamadas. |
| CommunicationTokenCredential | CommunicationTokenCredential se usa como la credencial de token para crear una instancia de CallAgent.| 
| CommunicationUserIdentifier | CommunicationUserIdentifier se usa para representar la identidad del usuario, que puede ser una de las opciones siguientes: CommunicationUserIdentifier/PhoneNumberIdentifier/CallingApplication. |

## <a name="authenticate-the-client"></a>Autenticar el cliente

Inicialice una instancia de `CallAgent` con un token de acceso de usuario que nos permita realizar y recibir llamadas. Agregue el código siguiente a la devolución de llamada `onAppear` en **ContentView.swift**:

```swift
var userCredential: CommunicationTokenCredential?
do {
    userCredential = try CommunicationTokenCredential(token: "<USER ACCESS TOKEN>")
} catch {
    print("ERROR: It was not possible to create user credential.")
    return
}

self.callClient = CallClient()

// Creates the call agent
self.callClient?.createCallAgent(userCredential: userCredential) { (agent, error) in
    if error != nil {
        print("ERROR: It was not possible to create a call agent.")
        return
    }

    if let agent = agent {
        self.callAgent = agent
        print("Call agent successfully created.")
    }
}
```

Debe reemplazar `<USER ACCESS TOKEN>` por un token de acceso de usuario válido para el recurso. Consulte la documentación de [token de acceso de usuario](../../access-tokens.md) si aún no tiene ningún token disponible.

## <a name="start-a-call"></a>Inicio de una llamada

El método `startCall` se establece como la acción que se llevará a cabo cuando se toque el botón *Iniciar llamada*. Actualice la implementación para iniciar una llamada con `ASACallAgent`:

```swift
func startCall()
{
    // Ask permissions
    AVAudioSession.sharedInstance().requestRecordPermission { (granted) in
        if granted {
            // start call logic
            let callees:[CommunicationIdentifier] = [CommunicationUserIdentifier(identifier: self.callee)]
            self.call = self.callAgent?.call(participants: callees, options: StartCallOptions())
        }
    }
}
```

También puede usar las propiedades de `StartCallOptions` para establecer las opciones iniciales de la llamada (es decir, permite iniciar la llamada con el micrófono silenciado).

## <a name="end-a-call"></a>Finalización de una llamada

Implemente el método `endCall` para finalizar la llamada actual cuando se toque el botón *Finalizar llamada*.

```swift
func endCall()
{    
    self.call!.hangup(HangupOptions()) { (error) in
        if (error != nil) {
            print("ERROR: It was not possible to hangup the call.")
        }
    }
}
```

## <a name="run-the-code"></a>Ejecución del código

Para compilar y ejecutar la aplicación en el simulador de iOS, seleccione **Producto** > **Ejecutar** o use el método abreviado de teclado (&#8984;-R).

:::image type="content" source="../media/ios/quick-start-make-call.png" alt-text="Aspecto final de la aplicación de inicio rápido":::

Para hacer una llamada de VoIP saliente, proporcione un identificador de usuario en el campo de texto y toque el botón **Iniciar llamada**. La llamada a `8:echo123` le conecta a un bot de eco, lo que resulta ideal como introducción y para verificar que los dispositivos de audio funcionan. 

> [!NOTE]
> La primera vez que realice una llamada, el sistema le solicitará acceso al micrófono. En una aplicación de producción, debe usar la API `AVAudioSession` para [comprobar el estado del permiso](https://developer.apple.com/documentation/uikit/protecting_the_user_s_privacy/requesting_access_to_protected_resources) y actualizar correctamente el comportamiento de la aplicación cuando no se conceda el permiso.

## <a name="sample-code"></a>Código de ejemplo

Puede descargar la aplicación de ejemplo de [GitHub](https://github.com/Azure-Samples/communication-services-ios-quickstarts/tree/main/Add%20Voice%20Calling).
