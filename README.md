# Blazor WASM App on GitHub Codespaces Template
There are the stepsfor SPA development using ASP.net BlazorWASM on GitHub Codespaces.
## Usage package
[jsakamoto/PublishSPAforGitHubPages.Build](https://github.com/jsakamoto/PublishSPAforGitHubPages.Build)
>This is a NuGet package that provides post published processing to deploy the .NET Core SPA project (such as Blazor WebAssembly) as a GitHub pages site.

## Create new GitHub Repogitory
select **Visual Studio** template with **Add .gitignore**
## Open WebEditor
push [ **.** ] key
## Add devcontainer setting
file name: `.devcontainer/devcontainer.json``
```json
{
    "image": "mcr.microsoft.com/devcontainers/dotnet:0-7.0",
    "name": "dev-container",
    "customizations": {
      "vscode": {
        "extensions": [
          "ms-dotnettools.csdevkit"
        ]
      }
    },
    "forwardPorts": [5000, 5001],
    "portsAttributes": {
      "5001": {
        "protocol": "https"
      }
    },
    "postCreateCommand": "dotnet restore"
}
```
## Add GitHub Actions setting
file name: `.github/workflows/publish-gh-pages.yml`
```yml
name: Publish Blazor App to GitHub Pages

on: [ push, workflow_dispatch ]

env:
  project_name: BlazorApp
  
permissions:
  contents: write
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: true

jobs:
  build_test_artifact:
    name: Build App

    runs-on: ubuntu-latest

    steps:
    - name: Checkout
      uses: actions/checkout@v3

    - name: Setup .NET SDK
      uses: actions/setup-dotnet@v2
      with:
        dotnet-version: '7.x'

    - name: Restore NuGet packages
      shell: bash
      run: |
        dotnet restore .

    - name: Build solution
      shell: bash
      run: |
        dotnet build .

    # - name: Test solution
    #   shell: bash
    #   run: |
    #     dotnet test . -c Release

    - name: Publish artifact
      shell: bash
      run: |
        dotnet publish ${{ env.project_name }} -c Release -o published -p:GHPages=true

    - name: Upload artifact for web
      uses: actions/upload-pages-artifact@v1
      with:
        path: published/wwwroot

  deploy:
    name: Deploy App to GitHub Pages
    needs:
    - build_test_artifact

    runs-on: ubuntu-latest

    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}

    steps:
    - name: Deploy to GitHub Pages
      id: deployment
      uses: actions/deploy-pages@v1
```
## Add debug setting
file name: `.vscode/launch.json`
```
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "blazorwasm",
            "name": "Launch and Debug Blazor WebAssembly Application",
            "request": "launch",
            "cwd": "${workspaceFolder}/BlazorApp"
        }
    ]
  }
```
## Commit & push
## Start GitHub Codespaces
1. push Ctrl + Shift + p keys, open command palet
1. type `continue working in new codespace`
## Create new blazor project & solution & some packages
copy & paste this command into the Terminal and execute it.
```
dotnet new blazorwasm -f net7.0 -o BlazorApp && dotnet new sln && dotnet sln add BlazorApp && dotnet add BlazorApp package PublishSPAforGitHubPages.Build
```
## Debug Run
push F5 key
## Deploy
only commit & push, run GitHub Actions

Check the URL on the GitHub Actions Page. 