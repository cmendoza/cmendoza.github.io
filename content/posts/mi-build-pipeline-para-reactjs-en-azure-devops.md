---
title         : "Mi build pipeline para ReactJS en Azure DevOps"
date          : 2021-07-23T11:06:08-05:00
description   : Últimamente he estado trabajando en un par de proyectos con ReactJS, pero solo en el más reciente me di a la tarea de crear un script para hacer el deployment con Azure DevOps.
author        : Carlos Mendoza
enableWhoami  : false
image         : images/posts/devops-pipelines.svg
tags:
- Azure DevOps
- ReactJS
---

Desde hace algunos años trabajo también con el front-end, primero usando jQuery y jQueryUI, AngularJS y Angular en tiempos más recientes (hace como 3 años).
Últimamente he estado trabajando en un par de proyectos con ReactJS, pero solo en el más reciente me di a la tarea de crear un script para hacer el build con Azure DevOps.

Es muy parecido al script que uso para [Angular](/posts/mi-build-pipeline-para-angular-en-azuredevops), aquí el script 👇

{{< codes yaml >}}
  {{< code >}}

  ```yaml
  name: $(Date:yyyyMMdd)$(Rev:.r)-$(Build.SourceBranchName)

  trigger:
    - master

  pool:
    vmImage: ubuntu-latest

  variables:
    apiUrl: 'my-api-url'

  steps:
    - task: NodeTool@0
      displayName: 'Install Node.js'
      inputs:
        versionSpec: '14.x'

    - task: PowerShell@2
      displayName: 'Create .env file'
      inputs:
        targetType: 'inline'
        script: |
          $envfile = "./.env"

          'REACT_APP_API_URL=$(apiUrl)' | Set-Content $envfile

          exit 0
        failOnStderr: true
        ignoreLASTEXITCODE: true
        pwsh: true
        workingDirectory: '$(Build.SourcesDirectory)'

    - task: PowerShell@2
      displayName: 'Run npm install and npm run build'
      inputs:
        targetType: 'inline'
        script: |
          npm install
          npm run build

    - task: PublishBuildArtifacts@1
      displayName: 'Publish artifact'
      inputs:
        PathtoPublish: '$(Build.SourcesDirectory)/build'
        ArtifactName: 'drop'
        publishLocation: 'Container'
  ```

  {{< /code >}}
{{< /codes >}}

Como escribí más arriba es muy parecido al de Angular, así que solo comento lo más relevante:

- **Línea 10.** Variables:
    - `apiUrl` Url a donde está alojada el API del back-end.
- **Línea 18.** Se crea un archivo .env que usa la aplicación, por ahora hay un único setting y un único ambiente.
- **Línea 41.** Se publica el resultado del build, en el caso de ReactJS este cae en la carpeta 'build'.

Este es el estado inicial de este script (aun así es funcional 😁), lo más seguro es que siga evolucionando ya que aun no se toman en cuenta diferentes settings y ambientes, todavia tengo camino por recorrer con ReactJS.

¡Saludos! ☕ 😉
