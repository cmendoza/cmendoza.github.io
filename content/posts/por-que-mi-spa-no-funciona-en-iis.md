---
title         : "¿Por qué mi SPA no funciona en IIS?"
date          : 2021-08-05T22:20:17-05:00
description   : Hace unos años cuando empecé a trabajar con Angular y subí la aplicación al servidor me di cuenta que algo no andaba bien al hacer refresh en el navegador.
author        : Carlos Mendoza
enableWhoami  : false
image         : images/posts/iis-angular-reactjs.svg
tags:
- IIS
- Angular
- ReactJS
---

## Introducción

Hace unos años cuando empecé a desarrollar Single Page Applications (SPA), comencé usando AngularJS dentro de aplicaciones MVC de C#, por lo que siempre que hacía el despliegue al servidor de producción/pruebas ambas aplicaciones front-end y back-end convivían juntas dentro de la misma aplicación/folder de IIS, además de que la aplicación MVC era quien redirigía los requests al front-end haciendo uso de las rutas de fallback con `MapFallbackToController` en .Net Core, y hasta ahí todo era felicidad 🌈.

Cuando migré a frameworks más modernos (Angular, ReactJS) empecé a separar los proyectos en repositorios diferentes y por lo tanto mi idea fué alojarlos en diferentes folders dentro de IIS. Cuando entré por primera vez a la aplicación vi que todo se veía bien:

{{< img src="/images/posts/webconfig-app-index.png" title="Imagen 1" caption="Mi página de inicio funcionando al 100%" alt="image alt" width="400px" position="center" >}}

Pero al hacer _refresh_ de la página... ¡bum! obtenía un bello error 404 (not found):

{{< img src="/images/posts/webconfig-app-404.png" title="Imagen 2" caption="Error 404" alt="image alt" width="400px" position="center" >}}

## Solución

Después de investigar un poco sobre el tema vi que lo que se tenia que hacer era un reescritura de las urls usando el módulo UrlRewrite de IIS, esto se logra agregando unas líneas al archivo `web.config` (o creándolo si es que no existe, que es lo más seguro 😜), de la siguiente forma:

{{< codes xml >}}
  {{< code >}}
  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <configuration>
    <system.webServer>
      <rewrite>
        <rules>
          <rule name="SPA Routes" stopProcessing="true">
            <match url=".*" />
            <conditions logicalGrouping="MatchAll">
              <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
              <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
            </conditions>
            <action type="Rewrite" url="./index.html" />
          </rule>
        </rules>
      </rewrite>
    </system.webServer>
  </configuration>
  ```
  {{< /code >}}
{{< /codes >}}

Básicamente lo que hace es indicar a IIS que todas las rutas de esa aplicación (a nivel IIS) van a ser redirigidas a index.html, que es el punto de entrada de nuestra aplicación cliente (Angular/ReactJS) y el router de esta sea el que las procese.

## ¿Dónde lo pongo?

Para el caso particular de cada tipo de aplicación hay que poner ese `web.config` en alguna carpeta en específico.

### Angular

Para este tipo de proyectos suelo colocarlo dentro de la carpeta `src` como se puede ver a continuación:

{{< img src="/images/posts/webconfig-angular-project.png" title="Imagen 3" caption="Donde colocar el web.config en un proyecto Angular" alt="image alt" width="400px" position="center" >}}

Con la configuración por defecto del proyecto creado con Angular CLI todo lo que esté dentro del la carpeta `assets` será copiado al hacer _build_, peeero como yo lo pongo en la raíz de `src` tengo que agregar la ruta dentro del archivo `angular.json`:

{{< img src="/images/posts/webconfig-angular-json-file.png" title="web.config" caption="Registrándolo como asset" alt="image alt" width="600px" position="center" >}}

Para evitar este paso simplemente habría que ponerlo en la carpeta `assets`, pero al hacer el _build_ la salida de este mantendría el archivo en esa misma carpeta, por lo que sería necesario en la fase de despliegue 'sacarlo' de ahí y dejarlo en la raíz de la aplicación, unas por otras 🤷‍♂️.

### ReactJS

En los proyectos ReactJS opté ponerlo en la carpeta `public`.

{{< img src="/images/posts/webconfig-reactjs-project.png" title="Imagen 4" caption="Donde colocar el web.config en un proyecto ReactJS" alt="image alt" width="400px" position="center" >}}

Uso [create-react-app](https://create-react-app.dev/docs/getting-started) para crear mis proyectos, y una de las cosas que configura este script es copiar todo lo que se encuentra en el folder `public` a la salida del _build_.

Y básicamente a eso se debía que mi SPA no estuviera funcionando correctamente en IIS, algo parecido me sucedió hace tiempo con un API que desarrolle en PHP y corría sobre Apache, la solución era muy parecida... pero bueno, eso es otra historia.

¡Saludos! ☕ 😉
