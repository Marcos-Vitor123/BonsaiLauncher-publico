# LlamaChat — Detalhes Técnicos

## Arquitetura

Aplicação Windows Forms (.NET 10, CPU-only) que gerencia duas instâncias do `llama-server`:

- **Porta 8080** — modelo principal (1.7B): permanece ativo durante toda a sessão
- **Porta 8081** — modelo de aquecimento (27B): carregado apenas na inicialização, descartado após obter resposta

## Fluxo de inicialização

```
┌─────────────────────────────────────────┐
│ 1. Inicia 1.7B na porta 8080            │
│    - 4 threads, contexto 32K            │
│    - Aguarda /health → OK               │
├─────────────────────────────────────────┤
│ 2. Inicia 27B na porta 8081             │
│    - 4 threads, contexto 2K             │
│    - Aguarda /health → OK               │
├─────────────────────────────────────────┤
│ 3. Envia "oi" para 27B (streaming SSE)  │
│    - Salva resposta no histórico        │
├─────────────────────────────────────────┤
│ 4. Mata processo 27B (libera ~3.5 GB)   │
├─────────────────────────────────────────┤
│ 5. Habilita chat com 1.7B               │
│    - Mensagem do sistema em português    │
│    - StripThinking em respostas          │
└─────────────────────────────────────────┘
```

## Parâmetros do servidor

Todos os parâmetros são definidos em `Config.GetServerArgs()`:

| Flag | Descrição |
|---|---|
| `-m` | Caminho do modelo GGUF |
| `--host / --port` | Endereço e porta |
| `-ngl 0` | CPU-only (sem GPU) |
| `-c` | Tamanho do contexto |
| `-t` | Threads |
| `--temp 0.7` | Temperatura |
| `--top-p 0.85` | Top-p sampling |
| `--top-k 20` | Top-k sampling |
| `--jinja` | Template jinja habilitado |
| `--mlock` | Trava modelo na memória |
| `--no-mmap` | Desabilita mmap |
| `-np 1` | Slots de prediction: 1 |

## Segurança de processos

```csharp
// LlamaServerManager.cs
[DllImport("kernel32.dll")]
static extern IntPtr CreateJobObject(IntPtr lpJobAttributes, string? name);

// Limite de 4 GB por processo
JOBOBJECT_MEMORY_LIMIT_INFORMATION limitInfo;
limitInfo.ProcessMemoryLimit = 4294967295u; // 4 GB
SetInformationJobObject(hJob, JobObjectInfoClass, ref limitInfo, ...);

// Todos os filhos herdam o Job Object
AssignProcessToJobObject(hJob, processInfo.hProcess);
```

- Se o app crashar ou for fechado, os servidores são automaticamente encerrados pelo OS
- Limite de memória previne consumo excessivo de RAM

## Componentes

| Arquivo | Responsabilidade |
|---|---|
| `Config.cs` | Configurações centralizadas, auto-detecção de modelos, args do servidor |
| `MainForm.cs` | UI (WinForms), ciclo de vida, 5 fases de inicialização |
| `MainForm.Designer.cs` | Layout do formulário (sem ComboBox) |
| `LlamaServerManager.cs` | Inicia/processo, Job Objects, wait for ready |
| `LlamaCliService.cs` | Streaming SSE com parsing de reasoning/content |
| `ChatClient.cs` | HTTP POST para `/v1/chat/completions` |
| `HistoryService.cs` | Persistência de histórico de conversas |
| `Models/ChatRequest.cs` | Modelos de requisição e resposta da API |

## Especificações de hardware de referência

- **CPU**: 8 logical processors
- **RAM**: 15.4 GB
- **1.7B**: ~40% RAM, ~2s por resposta de 200 tokens
- **27B**: ~3.5 GB RAM (temporário)
