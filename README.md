# LlamaChat

Chat desktop local que utiliza modelos **Bonsai** (GGUF) via [llama.cpp](https://github.com/ggml-org/llama.cpp), executando 100% na CPU.

## Como funciona

1. Na inicialização, carrega o modelo leve **Bonsai-1.7B** (porta 8080) para interação
2. Em background, carrega o modelo **Bonsai-27B** (porta 8081), envia "oi" para obter uma resposta e salva no histórico (semente de contexto)
3. O modelo 27B é descartado, liberando a RAM
4. A partir daí, o chat funciona apenas com o 1.7B, já com contexto pré-aquecido

## Requisitos

- Windows 10/11
- .NET 10 SDK
- ~4 GB de RAM livre (1.7B usa ~40% da memória)
- Modelos GGUF na pasta `Bonsai/models/`:
  - `Bonsai-1.7B-Q1_0.gguf` (~0.23 GB)
  - `Bonsai-27B-Q1_0.gguf` (~3.5 GB) — opcional, usado apenas no pré-aquecimento

## Como rodar

```bash
cd Privado
dotnet run
```

## Configurações

| Parâmetro | Porta 8080 (1.7B) | Porta 8081 (27B) |
|---|---|---|
| Contexto | 32.768 tokens | 2.048 tokens |
| Max tokens resposta | 32.768 | — |
| Temperatura | 0.7 | 0.7 |
| Threads | Metade dos cores | Metade dos cores |
| Limite de RAM | 4 GB | 4 GB |

## Segurança de processos

- **Job Objects** (Win32 API) garantem que os servidores `llama-server.exe` são encerrados automaticamente ao fechar o app
- Limite de memória de 4 GB por processo via `ProcessMemoryLimit`

## Estrutura do repositório

```
BonsaiLauncher/
├── Privado/              ← código fonte C# .NET 10
│   ├── Config.cs         ← configurações e parâmetros do servidor
│   ├── MainForm.cs       ← UI e ciclo de vida
│   ├── LlamaServerManager.cs ← gerenciamento de processos
│   ├── LlamaCliService.cs    ← cliente de streaming SSE
│   ├── ChatClient.cs     ← cliente HTTP para a API
│   ├── HistoryService.cs ← persistência de histórico
│   └── Models/           ← modelos de requisição/resposta
├── Publico/              ← documentação e assets públicos
│   └── docs/
└── Bonsai/               ← binários e modelos (não versionados)
    ├── llama-b10068-bin-win-cpu-x64/
    └── models/
```

## Referências

- [llama.cpp](https://github.com/ggml-org/llama.cpp)
- [Modelo Bonsai no Hugging Face](https://huggingface.co/Bonsai)
- [GGUF — formato de modelo](https://github.com/ggml-org/llama.cpp/blob/master/gguf-py/README.md)
