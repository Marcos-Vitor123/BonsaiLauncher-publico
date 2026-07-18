# LlamaChat

Aplicação desktop Windows que roda dois modelos de linguagem lado a lado para gerar e revisar respostas em tempo real.

![Tela do LlamaChat](./screenshot.png)
<!-- Substitua por um print real do programa em funcionamento -->

## Funcionalidades

- Dois servidores llama-server gerenciados automaticamente
- Modelo principal gera respostas criativas
- Modelo revisor refina e corrige a resposta gerada
- Interface simples e direta

## Pré-requisitos

- .NET 8 SDK
- GPU compatível (recomendado) ou CPU
- Arquivos de modelo no formato GGUF

## Como usar

```bash
cd Privado
dotnet run
```

O programa inicia os servidores, carrega os modelos e libera o chat quando ambos estiverem prontos.

## Estrutura do repositório

```
BonsaiLauncher/
├── Privado/      ← código fonte do projeto
└── Público/      ← documentação e assets públicos
```
