---
title        : "Mi build pipeline para Angular en Azure DevOps"
date         :   2021-07-20T18:52:36-05:00
description  : Siempre que empiezo un proyecto en Angular y llega la hora de crear el build pipeline en Azure DevOps, siempre termino entrando a ver el pipeline de algún proyecto previo y copiándolo.
author       : Carlos Mendoza
enableWhoami : false
image        : images/posts/devops-pipelines.svg
tags:
- Azure DevOps
- Angular
---

Siempre que empiezo un proyecto en Angular y llega la hora de crear el build pipeline en Azure DevOps, siempre termino entrando a ver el pipeline de algún proyecto previo y copiándolo.

Ha sufrido varias modificaciones a lo largo del tiempo (y proyectos) pero básicamente consta de 4 tareas:

1. Asignar variables que serán usadas en el script
2. Instalar NodeJS
3. Instalar Angular CLI, ejecutar npm install y hacer ng build al proyecto
4. Publicar el resultado del build como un artefacto

Aquí abajo el script 👇

{{< codes yaml >}}
  {{< code >}}

  ```yaml
  name: $(Date:yyyyMMdd)$(Rev:.r)-$(Build.SourceBranchName)

  trigger:
    - master
    - develop

  pool:
    vmImage: 'ubuntu-latest'

  variables:
    projectName: 'my-project-in-angular'
    buildConfiguration: 'production'

  steps:
    - task: PowerShell@2
      displayName: 'Setting variables'
      inputs:
        targetType: 'inline'
        script: |
          $branch = "$(Build.SourceBranchName)"
          if($branch -eq "develop")
          {
            Write-Host "##vso[task.setvariable variable=buildConfiguration]development"
          }
        pwsh: true

    - task: NodeTool@0
      displayName: 'Install Node.js'
      inputs:
        versionSpec: '14.x'

    - task: PowerShell@2
      displayName: 'Install Angular CLI, run npm install and ng build'
      inputs:
        targetType: 'inline'
        script: |
          npm install -g @angular/cli
          npm install
          ng build --configuration $(buildConfiguration)
        pwsh: true

    - task: PublishBuildArtifacts@1
      displayName: 'Publish artifact'
      inputs:
        PathtoPublish: '$(Build.SourcesDirectory)/dist/$(projectName)'
        ArtifactName: 'drop'
        publishLocation: 'Container'
  ```

  {{< /code >}}
{{< /codes >}}

Notas acerca de este script:

- **Línea 1.** El nombre del build `$(Date:yyyyMMdd)$(Rev:.r)-$(Build.SourceBranchName)` es para saber la fecha, revisión y nombre del branch con el que se ejecutó ese build, el nombre generado quedaría de la siguiente forma: `20210720.1-develop`.
- **Línea 3.** El build se disparará cada vez que haya un cambio en los branches `master` y `develop`.
- **Línea 10.** Las variables:
    - `projectName` es el nombre del proyecto Angular.
    - `buildConfiguration` es la configuración usada por la aplicación Angular, por default se asigna 'production'.
- **Línea 15.** Se asigna la configuración basado en el branch que disparó en build, normalmente manejo dos ambientes 'production' y 'development'.
- **Línea 42.** Se publica el resultado del build.

Y bueno, hasta este momento este es el estado actual de este script, puede que siga cambiando o que ahí se quede ya que ultimamente trabajo un poco más con ReactJS.

¡Saludos! ☕ 😉

Carlos Mendoza