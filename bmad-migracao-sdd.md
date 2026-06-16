# Migração SDD → BMAD
> Guia completo para mapear seu pipeline pessoal para o formato BMAD,  
> mantendo compatibilidade com Devin, Copilot e Claude Code.

---

## 1. Contexto — O que você tem hoje

Seu pipeline atual está salvo em `.ai/commands` e é copiado para:
- `.windsurf/workflows` → quando usa **Devin**
- `.github/prompts` → quando usa **Copilot**

### Fluxo atual

```
[Setup do Projeto — roda UMA VEZ por projeto]
sdd-agents-project → agents-project.md

[Por tarefa — roda para CADA demanda]
tarefa.txt (manual, copiado do Jira/iuClick)
    ↓
sdd-tarefa      (etapa 1)  → tarefa.md
    ↓
sdd-prd         (etapa 2a) → prd.md
sdd-agents      (etapa 2b) → agents.md
    ↓
sdd-spec        (etapa 3)  → spec.md
    ↓
sdd-impl        (etapa 4)  → código implementado
    ↓
sdd-review      (etapa 5)  → revisão do código
```

---

## 2. Mapeamento SDD → BMAD

| Seu comando | Etapa | Agente BMAD equivalente | Output |
|---|---|---|---|
| `sdd-agents-project` | 0 (projeto) | `architect` (setup) | `agents-project.md` |
| `sdd-tarefa` | 1 | `analyst` (Mary) | `tarefa.md` |
| `sdd-prd` | 2a | `pm` | `prd.md` |
| `sdd-agents` | 2b | `architect` (task-scoped) | `agents.md` |
| `sdd-spec` | 3 | `architect` + `dev` | `spec.md` |
| `sdd-impl` | 4 | `dev` | código |
| `sdd-review` | 5 | `qa` | revisão |

**Diferencial do seu fluxo que deve ser preservado:**  
O `sdd-agents-project` como etapa zero separada por projeto é um padrão mais inteligente que o BMAD padrão. O `agents-project.md` gerado deve ser referenciado como contexto em **todas** as etapas seguintes.

---

## 3. Estrutura de pastas final

```
projeto/
├── .ai/
│   └── commands/               ← seus comandos originais (fonte da verdade)
│       ├── sdd-agents-project.md
│       ├── sdd-tarefa.md
│       ├── sdd-prd.md
│       ├── sdd-agents.md
│       ├── sdd-spec.md
│       ├── sdd-impl.md
│       └── sdd-review.md
│
├── .windsurf/
│   └── workflows/              ← cópia para Devin
│       ├── sdd-agents-project.md
│       ├── sdd-tarefa.md
│       ├── sdd-prd.md
│       ├── sdd-agents.md
│       ├── sdd-spec.md
│       ├── sdd-impl.md
│       └── sdd-review.md
│
├── .github/
│   └── prompts/                ← cópia para Copilot
│       ├── sdd-agents-project.md
│       ├── sdd-tarefa.md
│       ├── sdd-prd.md
│       ├── sdd-agents.md
│       ├── sdd-spec.md
│       ├── sdd-impl.md
│       └── sdd-review.md
│
├── .bmad/
│   └── bmad-workflow.yaml      ← workflow BMAD formal (novo)
│
└── docs/
    ├── agents-project.md       ← gerado na etapa 0, persistente
    └── tasks/
        └── [nome-da-tarefa]/
            ├── tarefa.txt      ← entrada manual (do Jira/iuClick)
            ├── tarefa.md
            ├── prd.md
            ├── agents.md
            └── spec.md
```

---

## 4. Arquivo BMAD Workflow YAML

Criar em `.bmad/bmad-workflow.yaml`:

```yaml
name: sdd-pipeline
description: Pipeline SDD pessoal mapeado para BMAD
version: 1.0.0

# Etapa zero — roda uma vez por projeto
setup:
  - id: agents-project
    agent: architect
    action: generate-project-agents
    command: sdd-agents-project
    output: docs/agents-project.md
    description: >
      Gera o arquivo de contexto do projeto com arquitetura,
      stack, padrões e decisões técnicas. Independente de tarefas.

# Pipeline por tarefa
workflow:
  - id: tarefa
    step: 1
    agent: analyst
    action: structure-task
    command: sdd-tarefa
    input: docs/tasks/{task}/tarefa.txt
    context:
      - docs/agents-project.md
    output: docs/tasks/{task}/tarefa.md
    description: Transforma tarefa.txt em tarefa.md estruturado

  - id: prd
    step: 2a
    agent: pm
    action: generate-prd
    command: sdd-prd
    depends_on: tarefa
    context:
      - docs/agents-project.md
      - docs/tasks/{task}/tarefa.md
    output: docs/tasks/{task}/prd.md
    description: Gera especificações, requisitos e levanta ambiguidades

  - id: agents
    step: 2b
    agent: architect
    action: generate-task-agents
    command: sdd-agents
    depends_on: tarefa
    context:
      - docs/agents-project.md
      - docs/tasks/{task}/tarefa.md
    output: docs/tasks/{task}/agents.md
    description: Gera agents específico da tarefa

  - id: spec
    step: 3
    agent: architect+dev
    action: generate-spec
    command: sdd-spec
    depends_on: [prd, agents]
    context:
      - docs/agents-project.md
      - docs/tasks/{task}/tarefa.md
      - docs/tasks/{task}/prd.md
      - docs/tasks/{task}/agents.md
    output: docs/tasks/{task}/spec.md
    description: >
      Gera especificação completa com arquivos a adicionar,
      modificar, testes e critérios de aceite

  - id: impl
    step: 4
    agent: dev
    action: implement
    command: sdd-impl
    depends_on: spec
    context:
      - docs/agents-project.md
      - docs/tasks/{task}/spec.md
    description: Implementa o código conforme spec.md

  - id: review
    step: 5
    agent: qa
    action: review
    command: sdd-review
    depends_on: impl
    context:
      - docs/agents-project.md
      - docs/tasks/{task}/spec.md
    description: Revisa todo o código gerado contra a spec
```

---

## 5. O que pedir para a IA do trabalho

### Passo 1 — Cole esse contexto primeiro

```
Preciso converter meu pipeline SDD pessoal para o formato BMAD,
mantendo compatibilidade com Devin (.windsurf/workflows),
Copilot (.github/prompts) e Claude Code.

Meus comandos atuais em .ai/commands são:

- sdd-agents-project (etapa 0, por projeto): gera agents com
  arquitetura e contexto do projeto, independente de tarefas.
  Output: agents-project.md

- sdd-tarefa (etapa 1): transforma tarefa.txt em tarefa.md estruturado.
  Lê agents-project.md como contexto.

- sdd-prd (etapa 2a): gera especificações, requisitos e levanta
  ambiguidades. Lê agents-project.md e tarefa.md.

- sdd-agents (etapa 2b): gera agents específico da tarefa.
  Lê agents-project.md e tarefa.md.

- sdd-spec (etapa 3): gera spec completo com arquivos a adicionar,
  modificar e testes. Lê agents-project.md, prd.md e agents.md.

- sdd-impl (etapa 4): implementa o código. Lê agents-project.md
  e spec.md.

- sdd-review (etapa 5): revisa todo o código gerado contra a spec.
  Lê agents-project.md e spec.md.

Regras importantes:
1. agents-project.md é sempre contexto em todas as etapas 1 a 5
2. tarefa.txt é o único arquivo de entrada manual (vem do Jira/iuClick)
3. Cada comando deve funcionar nos 3 ambientes simultaneamente
4. Preservar a etapa 0 como setup separado por projeto
```

### Passo 2 — Peça os comandos convertidos

```
Agora me entrega cada comando sdd-* já no formato BMAD.
Para cada um preciso:
- O arquivo .md compatível com .ai/commands, .windsurf/workflows
  e .github/prompts ao mesmo tempo
- Com os contextos corretos referenciados
- Com instruções claras para o agente
```

### Passo 3 — Peça o workflow YAML

```
Gera o arquivo .bmad/bmad-workflow.yaml com meu pipeline completo,
separando a etapa 0 (setup de projeto) das etapas por tarefa.
```

### Passo 4 — Peça o script de sincronização

```
Cria um script sync-commands.sh que copia automaticamente os arquivos
de .ai/commands para .windsurf/workflows e .github/prompts,
para eu não precisar copiar manualmente toda vez que atualizar um comando.
```

### Passo 5 — Peça o README de uso para o time

```
Gera um README.md explicando como usar esse workflow do zero,
incluindo como uma pessoa de negócio pode alimentar o tarefa.txt
sem precisar saber programar, e como um dev novo no time
consegue rodar o mesmo pipeline.
```

---

## 6. O que validar em cada entrega da IA

Antes de commitar qualquer arquivo gerado, cheque:

- [ ] `agents-project.md` é referenciado em **todos** os comandos da etapa 1 em diante
- [ ] `tarefa.txt` continua sendo o **único** arquivo de entrada manual
- [ ] Cada comando funciona nos 3 ambientes sem alteração
- [ ] O YAML tem a etapa 0 (`sdd-agents-project`) **separada** do workflow por tarefa
- [ ] O `spec.md` lista explicitamente arquivos a adicionar, modificar e testes
- [ ] O `sdd-review` referencia o `spec.md` como critério de revisão

---

## 7. Script de sincronização (exemplo base)

Criar em `sync-commands.sh` na raiz do projeto:

```bash
#!/bin/bash
# Sincroniza .ai/commands para os ambientes Devin e Copilot

SOURCE=".ai/commands"
DEVIN=".windsurf/workflows"
COPILOT=".github/prompts"

echo "Sincronizando comandos..."

mkdir -p "$DEVIN" "$COPILOT"

cp "$SOURCE"/*.md "$DEVIN"/
cp "$SOURCE"/*.md "$COPILOT"/

echo "✓ Devin: $DEVIN"
echo "✓ Copilot: $COPILOT"
echo "Sincronização concluída."
```

```bash
chmod +x sync-commands.sh
./sync-commands.sh
```

---

## 8. Como fica o dia a dia após a migração

### Setup de projeto novo (uma vez)
```
1. Rodar sdd-agents-project → gera docs/agents-project.md
2. Commitar agents-project.md no repo
```

### Por tarefa
```
1. Copiar descrição do Jira/iuClick → colar em docs/tasks/[nome]/tarefa.txt
2. /sdd-tarefa     → tarefa.md
3. /sdd-prd        → prd.md       (revisar ambiguidades levantadas)
4. /sdd-agents     → agents.md
5. /sdd-spec       → spec.md      (revisar antes de implementar)
6. /sdd-impl       → código
7. /sdd-review     → revisão final
```

### Quando atualizar um comando
```
1. Editar em .ai/commands/
2. ./sync-commands.sh
3. Commitar tudo
```

---

## 9. Como o time de negócio pode entrar no fluxo

Em vez de você copiar do Jira, a pessoa de negócio pode:

1. Instalar o agente Analyst do BMAD (Mary):
   ```bash
   npx bmad-method install /analyst
   ```
2. Descrever a demanda em linguagem natural para a Mary
3. A Mary gera o `tarefa.txt` já estruturado
4. Você pega esse arquivo e começa no `/sdd-tarefa` normalmente

O ponto de entrada continua sendo `tarefa.txt` — só muda quem escreve.

---

## 10. Próximos passos após commitar os arquivos gerados

Quando tiver os arquivos prontos, trazer de volta para revisão:

1. Conferir se os prompts dos comandos convertidos estão completos
2. Ajustar referências de contexto se necessário
3. Testar o pipeline completo em uma tarefa real pequena
4. Documentar qualquer ajuste fino feito

---

*Gerado em: junho 2026*  
*Base: conversa de mapeamento SDD → BMAD*
