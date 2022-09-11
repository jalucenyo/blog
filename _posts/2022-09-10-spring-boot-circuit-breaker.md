---
layout: post
current: post
cover:  assets/images/spring-circuit-breaker.png
navigation: True
title: Circuit Breaker en Spring Boot
date: 2020-09-10 10:00:00
tags: [Spring Boot, Circuit Breaker, Backend, Kotlin]
class: post-template
subclass: 'post'
author: jose
---

Porque todo falla en todo momento, patrón Circuit Breaker.

En un ecosistema de microservicios es muy común que necesiten comunicarse entre servicios de forma sincrona, por lo tanto hay servicio que son dependiente de otros servicios. Un fallo en el servicio del cual depende se propaga a los otros servicios, degradando gran parte del sistema.

Podemos proteger los servicios de estas incidencias utilizando algunos de los patrones existentes, en este caso veremos un ejemplo con el patrón "Circuit Breaker"

### ¿ Como funciona el patrón "Circuit Breaker" ?

En un estado optimo donde no tenemos existen problemas los microservicios, la peticiones que realiza el microservicio A al B se realizan normalmente, recibiendo los datos que hemos soliciado.

![](/blog/assets/images/spring-circuit-breaker/circuit-breaker-diagram-01.png)

En el caso de que el microservicio B tenga un problema, las peticiones realizadas por el microservicio A devolverán un error, en este momento se ejecuta el método que definamos como recover o fallback. En este método implementaremos como debe actuar por ejemplo devolver datos que tengamos almacenado o guardar la acción para procesarla mas tarde.

![](/blog/assets/images/spring-circuit-breaker/circuit-breaker-diagram-02.png)

En este momento se pueden dar dos situaciones, puede ser un error puntual y en las siguientes peticiones el microservicio B atienda las peticiones con normalidad o puede que no se trate de un error puntual y el microservicio B este saturado o caído.

Si las llamada del microservicio A continua acumulando errores en las llamadas al microservicio B, se activara el circuit breaker. Como de un interruptor se tratara desconecta el microservicio A del B, circuito abierto, todas las llamadas serán atendidas por el método de recover/fallback

![](/blog/assets/images/spring-circuit-breaker/circuit-breaker-diagram-03.png)

De este modo conseguimos que la degradación o caída del microservicio B no afecte al microservicio A. El circuit breaker seguirá abierto hasta que el timeout definido cierre el circuito de nuevo, en este momento si el microservicio B se ha recuperado las llamadas serán atendidas por el microservicio B y NO por el método recover/fallback.

![](/blog/assets/images/spring-circuit-breaker/circuit-breaker-diagram-04.png)

Como ver este patrón nos puede ayudar a evitar la degradación de otros servicios por la caída de un servicio concreto, evitando la caída en cascada de otros servicios.

### Implementación con Spring Boot con retry.

Dentro de Spring Boot hay disponible el patrón circuit breaker, con dos implementaciones que podemos elegir [Resilience4J](https://github.com/resilience4j/resilience4j) o [Spring Retry](https://github.com/spring-projects/spring-retry).

En este ejemplo vamos a usar la implementación de Spring Retry, es algo mas simple y con menos opciones de configuración pero mas fácil de implementar.

Puedes descargar el código de ejemplo desde mi repositorio de GitHub.

Lo primero de todo es añadir las dependencias en el proyecto

```xml
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
</dependency>
```

Necesitaremos las dependencias de Spring AOP y Aspect, el Circuit Breaker hace uso de Aspects.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

Deberemos de anotar nuestra clase principal con la anotación **@EnableRetry**

```kotlin
@SpringBootApplication
@EnableRetry
class CircuitBreakerApplication

fun main(args: Array<String>) {
  runApplication<CircuitBreakerApplication>(*args)
}
```

Para ejemplo vamos a simular que nuestro microservicio tiene un endpoint para obtener los usuarios, el cual debe de obtenerlos de un servicio externo como [randomuser.me](https://randomuser.me/api).

El método findUsers se encarga de realizar la llamada con RestTemplate y obtener los usuarios, este método es el que debemos anotar con @CircuitBreaker.

Implementaremos otra función que anotaremos con @Recover, esta función se ejecutar cuando la llamada para obtener los usuarios falle y cuando el circuit breaker este abierto.


```kotlin
@Service
class UserClient {

  private val log = LoggerFactory.getLogger(this.javaClass)

  @CircuitBreaker(maxAttempts = 10, openTimeout = 30000, resetTimeout = 60000)
  fun findUsers(): List<User>{
    log.info("Call other service get users")
    return (RestTemplate())
      .getForObject("https://randomuser.me/api", Results::class.java)!!.results!!
  }

  @Recover
  fun findUsersRecover(e: Throwable): List<User>{
    log.error("Recover method of circuit breaker, exception: ", e.message)

    //TODO: Aqui implementariamos logica en caso de fallo.
    return listOf(User(name = Name(title = "Mr.", first = "Fallback", last = ":-)")))
  }

}
```

La anotación de CircuitBreaker tiene tres parámetros para ajustar el funcionamiento:

- maxAttemps: Número de intentos erróneos que deben ocurrir en el tiempo que definimos en openTimeout.
- openTimeout: Tiempo en ms en el que se abre el circuito, si se han producido los intentos definidos en maxAttemps. 
- resetTimeout: Tiempo en ms para que el circuito se cierre de nuevo y se vuelven hacer la peticiones.

En el ejemplo cuando se produzcan 10 errores en medio minuto (30.000 ms), el Circuit Breaker actúa, desconectando las llamadas a findUsers y llamara al método anotado con @Recover, durante un minuto (60.000 ms)

## Conclusion

En fin como hemos podido comprobar con el patron de Circuit Breaker y la implementación que ofrece Spring Boot, podemos utilizarlo en nuestros proyectos de microservicios y evitar que la degración de otros servicios puedan producir una degración en cascada de otros servicios, desconectando de forma automatica el servicio que esta ocasionando el problema.

El codigo de ejemplo lo teneis en mi GitHub [https://github.com/jalucenyo/spring-circuit-breaker-retry](https://github.com/jalucenyo/spring-circuit-breaker-retry)

Si te ha gustado este articulo compartelo, es una forma fácil de ayudarme a crear mas contenido como este. Si tienes dudas o quieres que escriba sobre algun tema de programación dejame un mensaje.

### Enalaces de interes

- [Documentación de Spring](https://spring.io/projects/spring-cloud-circuitbreaker)
