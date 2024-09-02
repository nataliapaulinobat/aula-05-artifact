# Configuração do Workflow no GitHub Actions

Este repositório contém um exemplo de configuração do GitHub Actions com múltiplos jobs e um scheduler. O workflow é configurado para compilar um artefato Go, fazer upload dos artefatos, e executar jobs adicionais com base nos artefatos gerados.

## Estrutura do Workflow

O workflow `go-example.yml` é configurado para:

1. **Compilar o artefato Go para Linux e Windows**.
2. **Fazer upload dos artefatos compilados**.
3. **Baixar e executar o artefato Linux em um job separado**.
4. **Baixar o artefato Windows em outro job separado**.
5. **Executar periodicamente a cada 15 minutos de segunda a sábado**.

## Passos para Configuração

### 1. Criar o Repositório

1. **Criar um repositório no GitHub** chamado `aula-05-artifact`.

### 2. Adicionar o Arquivo `go-example.yml`

1. **Navegar para o diretório** `.github/workflows` no repositório.
2. **Criar um arquivo chamado** `go-example.yml` com o seguinte conteúdo:

```yaml
name: Go Example Workflow

on:
  push:
    branches:
      - main
  schedule:
    - cron: '*/15 * * * 1-6' # Execute a cada 15 minutos, de segunda a sábado

env:
  FILE_NAME: hello-server

jobs:
  BuildGo:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Build Go artifact for Linux
        run: go build ${{ env.FILE_NAME }}.go

      - name: Build Go artifact for Windows
        run: GOOS=windows GOARCH=amd64 go build ${{ env.FILE_NAME }}.go

  UploadArtifacts:
    needs: BuildGo
    runs-on: ubuntu-latest
    steps:
      - name: Upload artifact for Linux
        uses: actions/upload-artifact@v3
        with:
          name: linux
          path: ${{ env.FILE_NAME }}

      - name: Upload artifact for Windows
        uses: actions/upload-artifact@v3
        with:
          name: windows
          path: ${{ env.FILE_NAME }}.exe

  download-and-run-linux:
    needs: BuildGo
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Download Linux artifact
        uses: actions/download-artifact@v3
        with:
          name: linux

      - name: Run Linux script
        run: |
          chmod +x ./run.sh
          source ./run.sh

  download-only-windows:
    needs: BuildGo
    runs-on: windows-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Download Windows artifact
        uses: actions/download-artifact@v3
        with:
          name: windows
