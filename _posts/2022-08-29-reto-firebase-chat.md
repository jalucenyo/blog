---
layout: post
current: post
cover:  assets/images/mouredev_rviewer_firebasechat.webp
navigation: True
title: Reto Firebase Chat
date: 2020-08-29 10:00:00
tags: [Android, Retos, Jetpack Compose]
class: post-template
subclass: 'post'
author: jose
---

Resolviendo el reto de crear una app de chat con Firebase

Este mes de Agosto me animado hacer uno de los retos que propone Mouredev, si queréis aprender o mejorar vuestras habilidades en programación os recomiendo que hachéis un vistazo a sú pagina web [Restos de Programación](https://retosdeprogramacion.com).

### ¿ De que va el reto ?

El resto consigue en hacer una aplicación que permita tener una única sala de chat donde varios usuarios puedan enviar y recibir mensajes, para llevarlo acabo se requiere utilizar Firebase como backend de la aplicación. 
Ademas los usuarios deben de poder autenticarse con la cuenta de Gmail.

Como apartados extras la aplicación los usuarios deben poder enviar y recibir imágenes y recibir notificaciones.

Desde mi punto de vista creo que es un reto muy interesante, nos permite explorar los principales servicios de Firebase, asi como la comunicación en tiempo real. El reto no impone ninguna tecnología para resolver el reto, asi que he decidido resolverlo en Android con Kotlin y Jetpack Compose, simplemente porque son las tecnología que me prepuesto aprender durante este año 2022 y que mejor manera que practicando. 

### ¿ Que es Firebase ?

Firebase es una plataforma de Google con multitud de servicios para facilitar la creación de aplicaciones tanto móviles como web. Podemos encontrar servicios para:

- Autenticación y registro de usuarios.
- Base de datos en tiempo real
- Notificaciones
- Hosting
- Almacenaje de ficheros
- Servicios de AI (Inteligencia Artificial)
- Analítica
- Y muchos mas que podemos encontrar en la pagina web de [Firebase](https://firebase.google.com)


De todos los servicios que ofrece para el reto he utilizado los siguientes:

[**Firestore Database**](https://firebase.google.com/products/firestore)
Base de datos NoSQL que almacena y sincroniza los datos en tiempo real, la utilizare para almacenar los mensajes que los usuarios escriben en la sala de chat.

[**Authentication**](https://firebase.google.com/products/auth)
Sistema de autenticación con integración con Google, FaceBook, Twitter, etc.. Este servicio lo utilizara para la autenticación y registro de los usuarios. 

[**Storage**](https://firebase.google.com/products/storage)
Permite almacenar ficheros de todo tipo como imagenes, documentos, etc... Nos viene ideal para almacenar las fotos que envían los usuarios al chat.

[**Messaging**](https://firebase.google.com/products/cloud-messaging)
Este servicio permite enviar notificaciones a los dispositivos aunque la aplicación no este abierta. 

[**Functions**](https://firebase.google.com/products/functions)
Permite ejecutar código de backend sin administrar servidores, en este caso lo utilizo para enviar notificaciones cuando se guardan registros en la base de datos


### Primeros pasos para crear la aplicación

Voy a explicar los pasos que considere mas relevantes que he llevado en el desarrollo de la aplicación, todo el código lo podéis encontrar en en repositorio de [GitHub](https://github.com/jalucenyo/FirebaseChat).

El primer paso que realice fue tener una estructura donde organizar cada funcionalidad(features), podemos encontrar las siguientes carpetas: 

- **feature_auth**: Todo lo relativo a autenticación de los usuarios
- **feature_chat**: Toda la funcionalidad relativa a el chat entre usuarios
- **feature_notifications**: Funcionalidad de notificaciones 
- **shared**: Componentes compartidos
- **ui-theme**: El tema de la aplicación
- **firebase_splash**: Una simple pantalla de inicio

Creo que con esta organización es fácil que cualquier otro desarrollador/a puede situarse rápidamente en cualquier parte de la aplicación.

### Arquitectura de la aplicación

El siguiente paso mas importante es decidir la arquitectura que va a seguir la aplicación, en este caso la [guía de arquitectura de apps](https://developer.android.com/jetpack/guide?hl=es-419#recommended-app-arch), nos recomienda unos patrones de diseño muy probados y maduros, que podemos utilizar.

En mi caso he decido seguir la siguiente arquitectura, con una capa para los datos (Data Layer), un ViewModel por cada pantalla con la lógica de negocio y una ultima capa con la UI, construida con "composables".
De modo que entre el ViewModel y la UI, tenemos un sentido unidireccional de información, es decir el para enviar datos a la pantalla se realiza mediante cambios en el estado y la UI envía eventos que que atenderá el ViewModel, realizando las operaciones y cambios de estado necesarios.

![](/blog/assets/images/mad-arch-ui-udf.png)


Otro componente de la arquitectura que decido utilizar es la inyección de dependencias con Hilt, este sera el encargado de construir y
facilitar las dependencias necesarias a cada componente de la aplicación.

### Configurar Hilt/Dagger y la Navegación

No voy entrar en demasiado detalle sobre Hilt, podemos encontrar la siguiente [guía de desarrollo de Hilt](https://developer.android.com/training/dependency-injection/hilt-android?hl=es-419), nos indica los pasos a seguir para incorporar la inyección de dependencias en
nuestros proyectos.

En resumen deberemos de: 

- Añadir las dependencias tanto en build.gradle del raíz del proyecto como en el de la aplicación
- Habilitar o revisar que Java8 esta habilitado en el proyecto
- Crear una clase de que extienda de Application y anotarla con @HiltAndroidApp
- Modificar el Manifest.xml e indicar "android:name" apunte a la clase de aplicación que hemos creado.
- Anotar la clase de la Activity principal con @AndroidEntryPoint

Con estos pasos iniciales ya tenemos una configuración de Hilt que nos permita trabajar, en cada una de las carpetas feature_*, he
creado un capeta "module" que contiene la clase de Hilt encargada de construir las dependencias.

### La Navegación

Utilizo el componente de [Jetpack Compose navigation](https://developer.android.com/jetpack/compose/navigation?hl=es-419), que permite
navegar entre los elementos que admiten composición y aprovechar la infraestructura y las funciones del componente Navigation.

Añadimos las dependencias, para poder utilizarlo en la aplicación.

``` 
    implementation "androidx.navigation:navigation-compose:2.5.1"
    implementation "androidx.navigation:navigation-ui-ktx:2.5.1"
```

En la clase FirebaseChatNavigation implementamos un NavHost que contiene los composables de las cuatro rutas que tiene la aplicación: 

- SPLASH_SCREEN : Pantalla de inicio de la aplicación en este caso es simplemente decorativa, se puede utilizada para preparar datos de
la aplicación antes de presentar la UI del usuario.
- SIGN_IN_SCREEN: Pantalla donde los usuarios pueden autenticarse.
- SING_UP_SCREEN: Aunque el reto no lo pide, he decidido implementar el registro de usuarios, mediante email.
- CHAT_ROOM_SCREEN: Pantalla donde se muestran los mensajes de los usuarios

A destacar de la implementación he decido inyectar el ViewModel al composable de cada pantalla en la clase de navegación, de este modo el composable de la pantalla no tiene dependencias internas y podemos utilizar las preview sin tener que crear una instancia del ViewModel con todos los problemas que esto conlleva.

No estoy del todo convencido de esta implementación, pero a la hora de diseñar las pantallas ayuda mucho en el flujo de trabajo tener las preview de una forma sencilla, ademas de este modo incluso permite hacer preview con diferentes estados y ver como quedaría el diseño.

### Feature Autenticación

Para poder utilizar el servicio de autenticación de firebase lo primero que he tenido que hacer es habilitar este servicio en Firebase, para eso iremos a la [consola de Firebase](https://console.firebase.google.com) 

Crearemos un proyecto si no tenemos un ya creado, iremos al menu "Compilación" a la sección "Authentication", donde podemos habilar el servicio de autenticación de Firebase, nos muestra una pantalla con las opciones de los diferentes proveedores que podemos utilizar, en este caso he utilizado el de "Correo electrónico/contraseña" y el de Google.

![](/blog/assets/images/firebase_auth_03_screenshot.png.png)

Firebase nos facilita un SDK que facilita gran parte del trabajo, para poder utilizarlo añadimos las siguientes dependecias al fichero gradle de la aplicación

```
    implementation platform('com.google.firebase:firebase-bom:30.3.0')
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-play-services:1.6.4'
    implementation 'com.google.firebase:firebase-auth-ktx'
    implementation 'com.google.android.gms:play-services-auth:20.2.0'
```

Veamos para que sirve cada una de las dependencias que acabamos de añadir:

- firebase-bom: Permite administrar las versiones de las bibliotecas de Firebase.
- kotlinx-coroutines-play-services: Integra las clases de tipo Task de Google Play Service dentro de las coroutinas de kotlin
- firebase-auth-ktx: Dependencias para usar las librerías de autenticación de Firebase
- play-services-auth: Dependecias para poder utilizar los servicios de Google de registro de usuarios.

En la carpeta feature_auth, encontramos la implementación de la funcionalidad principalmente tenemos las siguientes clases:

**AccountService**: Interface que hace de proxy con los métodos que abstraen la funcionalidad para autenticar y registrar los usuarios, en la implementación de est interface, encontramos los llamadas y lógica concreta para autenticar con Firebase. De este modo los cambios que pueda sufrir el SDK de Firebase no afectaran a las otras capas de la aplicación.

**AuthModule**: Contiene las construcción de las dependencias necesarias para el inyector de dependencias Hilt.

**SignInScreen**: Contiene el diseño de la pantalla de Autenticación, con diferentes componentes JetpackCompose, en este caso he decidido que la pantalla reciba en el constructor tanto el estado como los diferentes métodos que enlazan con el ViewModel, de modo que el el componente queda sin estado, mejorando el mantenimiento y reutilización.

**SignInViewModel**: Contiene toda la lógica de negocio para la autenticación y el estado de la UI. 

**SignUpScreen / SignUpViewModel**: Seguimos la misma arquitectura para el resto de pantallas de la aplicación separada en UI y ViewModel.

No entrare en mas detalle sobre estas funcionalidad, ya que con la utilización del SDK de Firebase para la autenticación, queda una implementación bastante sencilla de seguir y entender.

En este punto podemos mejorar esta funcionalidad añadiendo la recuperación de contraseña y la verificación de la cuenta de Firebase, al darse de alta via email.


### Feature Chat

Para realizar el chat entre usuarios utilizo el servicio de "Firestore Database" para almacenar los mensajes de cada uno de los usuarios, uso también el servicio "Storage" para almacenar las imágenes que se envían al chat. De modo que lo primero es activar estos dos servicios en Firebase. 

Al activar estos servicios tenemos que especificar las reglas para poder leer y escribir, para este caso de uso con verificar que el usuario este autenticado es suficiente, ya que todos los usuarios pueden leer y escribir en el chat siempre que estén autenticados.

Para Storage especificamos la siguiente regla: 

```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /{allPaths=**} {
      allow read, write: if request.auth != null
    }
  }
}
```

Y las reglas para Firestore Database:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if request.auth != null
    }
  }
}
```

Necesitaremos añadir las siguientes dependencias para poder utilizar las librerias:

```
    implementation 'com.google.firebase:firebase-firestore-ktx'
    implementation 'com.google.firebase:firebase-storage-ktx'
```

Con Firebase configurado podemos empezar con la implementación para mantener una arquitectura limpia en la aplicación, para este caso he considerado utilizar el patrón "Repository", siguiendo la [guía de Android](https://developer.android.com/jetpack/guide?hl=es-419), para obtener y guardar los mensajes e imágenes, aislando asi las implementación de Firebase.

![](/blog/assets/images/mad-arch-overview-data.png)

El las implementaciones de las interfaces **MessagesRepository** y **StorageRepository**, contienen la lógica para enviar y recibir mensajes, asi como subir las imágenes.

La recepcion de mensajes en tiempo real, queda en el sdk de Firestore, el cual admite que le indiquemos un listener que escucha los mensajes que 



### Feature Notificaciones


### Recopilación de links utilizados.

Links utilizados en el desarrollo de la app, que creo que pueden ser útiles.

- [Firebase](https://firebase.google.com)
- [Repositorio GitHub de la aplicación](https://github.com/jalucenyo/FirebaseChat)
- [Guía de arquitectura de apps Android](https://developer.android.com/jetpack/guide?hl=es-419#recommended-app-arch)
- [Guía de desarrollo de Hilt](https://developer.android.com/training/dependency-injection/hilt-android?hl=es-419)
