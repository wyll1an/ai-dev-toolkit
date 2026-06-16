# Configuração de IA Multiplataforma — Briefing & Instalador

> Uma única fonte de verdade que alimenta **Devin, Windsurf, Cursor, Copilot e Claude Code**. O dev clona, escolhe a plataforma, e os slash commands, skills e docs do projeto funcionam — sem reconfigurar nada.
>
> Estrutura validada contra a documentação oficial de cada plataforma (jun/2026).

---

## 00 · O que precisa ser construído

Existe hoje um repositório de configuração de IA de um produto. As skills (padrões de criação de classes constantes e propriedades) e os playbooks (fluxos: criar feature, fazer chamada de API) estão hoje espalhados dentro de `.devin/` e `.windsurf/`.

O objetivo é **reestruturar** esse repositório para um modelo de **fonte de verdade única**, de onde se gera automaticamente a configuração de cada plataforma de IA. Assim, qualquer desenvolvedor clona o repositório, escolhe a ferramenta que usa (Devin, Windsurf, Cursor, Copilot ou Claude Code) e tudo funciona: slash commands, documentação do produto e os padrões/fluxos como referência.

> **Resultado esperado:** Um dev abre o projeto no VS Code ou no Windsurf, digita `/gerar-specs` e a IA já conhece os padrões do produto — porque o comando instrui a LLM a consultar os playbooks e skills do repositório.

---

## 01 · O conceito central

Escreve-se tudo **uma única vez** dentro da pasta `.ai/`. Um script (`sync-ai.mjs`) lê essa pasta e **gera** os arquivos no formato exato que cada plataforma espera. Ninguém edita as pastas `.claude/`, `.github/`, `.cursor/` ou `.windsurf/` à mão — elas são saída do script.

```
         .ai/  (você escreve aqui — fonte de verdade)
              │
              ▼
     node scripts/sync-ai.mjs
              │
   ┌──────────┼───────────┬───────────┬──────────┐
   ▼          ▼           ▼           ▼          ▼
.claude/  .github/    .cursor/    .windsurf/   (Devin lê
 skills/   prompts/     rules/     workflows/   o AGENTS.md)
```

Acima de tudo isso existe o `AGENTS.md` na raiz — um padrão aberto lido nativamente por praticamente todos os agentes (Claude Code, Copilot, Cursor, Windsurf, Devin, Gemini CLI, Aider e outros). Ele carrega o contexto universal sem precisar de conversão.

---

## 02 · Estrutura de pastas completa

Legenda:
- **[FONTE]** — fonte de verdade, você edita aqui
- **[GERADO]** — gerado pelo script, não editar
- **[FIXO]** — config fixa, editar uma vez

```
projeto/
├── AGENTS.md                  # [FONTE] contexto universal — todas as plataformas leem
├── README.md
├── .ai/                       # ◀ FONTE DE VERDADE — só edite aqui
│   ├── commands/              # [FONTE] SDD: viram slash commands (você dispara)
│   │   ├── criar-tarefa.md
│   │   ├── gerar-prd.md
│   │   ├── gerar-specs.md
│   │   ├── implementar.md
│   │   └── review.md
│   ├── playbooks/             # [FONTE] fluxos do produto: contexto/referência
│   │   ├── criar-feature.md
│   │   └── chamada-api.md
│   ├── skills/                # [FONTE] padrões de código: contexto/referência
│   │   ├── classe-constante.md
│   │   └── propriedade-padrao.md
│   └── context/               # [FONTE] arquitetura + convenções
│       ├── architecture.md
│       └── conventions.md
├── docs/produto/              # documentação do produto (comitada junto)
│   └── README.md
├── scripts/
│   └── sync-ai.mjs            # gera as pastas das plataformas
│
├── .claude/                   # [GERADO] Claude Code
│   ├── CLAUDE.md              #   (config [FIXO], editar 1x)
│   └── skills/<cmd>/SKILL.md  #   (gerado: /comando)
├── .github/                   # [GERADO] GitHub Copilot
│   ├── copilot-instructions.md #  (config [FIXO])
│   ├── instructions/*.md      #   (regras por glob)
│   └── prompts/*.prompt.md    #   (gerado: /comando, só IDE)
├── .cursor/                   # [GERADO] Cursor
│   └── rules/*.mdc            #   (000=fixo · 3xx=comandos · 5xx=refs)
├── .windsurf/                 # [GERADO] Windsurf / Devin Desktop
│   ├── rules/*.md             #   (contexto + refs)
│   └── workflows/*.md         #   (gerado: /comando)
└── .devin/                    # Devin Desktop (regras locais) + LEIA-ME
    ├── rules/projeto.md
    └── LEIA-ME.md             #   como usar no Devin Cloud (UI)
```

---

## 03 · Os três tipos de conteúdo

Esta é a distinção mais importante do modelo. Cada tipo tem comportamento e destino diferentes.

### ① Commands — você dispara

Ficam em `.ai/commands/`. São os comandos de SDD (Spec-Driven Development): gerar tarefa, PRD, specs, implementação, review. Viram **slash commands** (`/comando`) em todas as plataformas. A LLM só os executa quando o dev digita o comando.

### ② Playbooks — a IA consulta sozinha

Ficam em `.ai/playbooks/`. São os fluxos do produto: como criar as partes de uma feature, como fazer uma chamada de API. **Não** viram slash command. São referência que a LLM lê por conta própria quando um comando precisa.

### ③ Skills — a IA consulta sozinha

Ficam em `.ai/skills/`. São os padrões de código: como criar uma classe constante, como definir uma propriedade no padrão. Também são referência consultável, não slash command.

> **O fluxo na prática:** O dev digita `/implementar` → a LLM lê o comando → o próprio comando a instrui a verificar `.ai/playbooks/` (qual fluxo seguir) e `.ai/skills/` (quais padrões aplicar) → ela implementa no padrão do produto. Os comandos `gerar-specs` e `implementar` já trazem essa instrução embutida.

| Pasta em .ai/ | Vira | A IA usa quando… |
|---|---|---|
| `commands/` | slash command | você digita `/comando` |
| `playbooks/` | regra de referência | um comando precisa do fluxo |
| `skills/` | regra de referência | um comando precisa do padrão |

---

## 04 · Onde cada plataforma carrega o quê

Mapeamento validado contra a documentação oficial de cada ferramenta.

| Plataforma | Contexto (automático) | Slash commands (de .ai/commands/) |
|---|---|---|
| **Todas** | `AGENTS.md` | — |
| **Claude Code** | `.claude/CLAUDE.md` | `.claude/skills/<nome>/SKILL.md` → `/nome` |
| **Copilot** (VS Code · VS · JetBrains) | `.github/copilot-instructions.md` | `.github/prompts/<nome>.prompt.md` → `/nome` |
| **Cursor** | `.cursor/rules/000-*.mdc` (`alwaysApply:true`) | `.cursor/rules/3xx-*.mdc` (agent-requested*) |
| **Windsurf** | `.windsurf/rules/*.md` | `.windsurf/workflows/*.md` → `/nome` |
| **Devin Desktop** | `.devin/rules/*.md` | `.windsurf/workflows/*.md` (compartilhado) |
| **Devin Cloud** | `AGENTS.md` (auto) | Playbooks na UI (app.devin.ai) |

> ⚠️ **Atenção · Copilot:** Os prompt files (`/comando`) estão em preview e só funcionam em **VS Code, Visual Studio e JetBrains**. O **Copilot CLI** ainda não reconhece prompt files de `.github/prompts/` como slash commands. O contexto de `copilot-instructions.md`, porém, é carregado em toda conversa do Copilot Chat no repositório.

> ⚠️ **Atenção · Cursor:** O Cursor não tem `/command` nativo como as outras. Os comandos viram regras `.mdc` que o agente puxa por descrição (agent-requested). Funcionam, mas o gatilho é por intenção, não por digitar `/`.

> 🔴 **Importante · Devin Cloud:** No Devin Cloud, playbooks e knowledge **vivem na plataforma web, não em arquivos do repositório**. Para usá-los lá, copie o conteúdo de `.ai/commands/` e `.ai/playbooks/` para a UI em `app.devin.ai → Playbooks / Knowledge`. O Devin Cloud lê o `AGENTS.md` da raiz automaticamente. A pasta `.devin/rules/` no repo serve apenas ao **Devin Desktop** (Cascade local).

---

## 05 · Instalador — script de setup

Em vez de um executável (que dispara antivírus e não roda em qualquer sistema), o caminho limpo e multiplataforma é gerar um **script de setup**. Ele cria só as pastas das plataformas escolhidas e dispara o sync a partir do `.ai/`.

### macOS / Linux — `setup-ai.sh`

```bash
#!/usr/bin/env bash
# setup-ai.sh — configura IA multiplataforma na raiz do projeto
set -e

echo "› Verificando fonte de verdade (.ai/)..."
if [ ! -d ".ai" ]; then
  echo "✗ Pasta .ai/ não encontrada. Copie o repo de config para cá primeiro." >&2
  exit 1
fi

echo "› Garantindo pastas das plataformas selecionadas..."
# Descomente as pastas das plataformas que o dev usa:
# Claude Code:
# mkdir -p ".claude/skills"
# GitHub Copilot:
# mkdir -p ".github/prompts" ".github/instructions"
# Cursor:
# mkdir -p ".cursor/rules"
# Windsurf:
# mkdir -p ".windsurf/rules" ".windsurf/workflows"
# Devin Desktop:
# mkdir -p ".devin/rules" ".windsurf/rules" ".windsurf/workflows"

echo "› Gerando configuração a partir de .ai/ ..."
if command -v node >/dev/null 2>&1; then
  node scripts/sync-ai.mjs
else
  echo "⚠ Node não encontrado. Os arquivos já comitados continuam válidos;"
  echo "  instale Node para regenerar a partir de .ai/."
fi

# Se usar Devin Cloud:
# echo "▸ Devin Cloud: copie .ai/commands/ e .ai/playbooks/ para app.devin.ai"
# echo "  (Playbooks/Knowledge). Veja .devin/LEIA-ME.md"

echo "✓ Pronto. Abra o projeto na sua ferramenta e use os slash commands."
```

Rodar:
```bash
chmod +x setup-ai.sh && ./setup-ai.sh
```

### Windows — `setup-ai.ps1`

```powershell
# setup-ai.ps1 — configura IA multiplataforma na raiz do projeto
$ErrorActionPreference = "Stop"

Write-Host "> Verificando fonte de verdade (.ai/)..."
if (-not (Test-Path ".ai")) {
  Write-Error "Pasta .ai/ nao encontrada. Copie o repo de config para ca primeiro."
  exit 1
}

Write-Host "> Garantindo pastas das plataformas selecionadas..."
# Descomente as pastas das plataformas que o dev usa:
# Claude Code:
# New-Item -ItemType Directory -Force -Path ".claude/skills" | Out-Null
# GitHub Copilot:
# New-Item -ItemType Directory -Force -Path ".github/prompts",".github/instructions" | Out-Null
# Cursor:
# New-Item -ItemType Directory -Force -Path ".cursor/rules" | Out-Null
# Windsurf:
# New-Item -ItemType Directory -Force -Path ".windsurf/rules",".windsurf/workflows" | Out-Null
# Devin Desktop:
# New-Item -ItemType Directory -Force -Path ".devin/rules",".windsurf/rules",".windsurf/workflows" | Out-Null

Write-Host "> Gerando configuracao a partir de .ai/ ..."
if (Get-Command node -ErrorAction SilentlyContinue) {
  node scripts/sync-ai.mjs
} else {
  Write-Host "! Node nao encontrado. Os arquivos ja comitados continuam validos;"
  Write-Host "  instale Node para regenerar a partir de .ai/."
}

Write-Host "Pronto. Abra o projeto na sua ferramenta e use os slash commands."
```

Rodar:
```powershell
.\setup-ai.ps1
```

> **Por que não um .exe:** Um executável binário trava em antivírus corporativo, exige assinatura e só roda num SO. Um script de shell é texto puro, auditável, versionável no git e roda em qualquer máquina com o interpretador nativo (bash no Mac/Linux, PowerShell no Windows). É o que ferramentas sérias de scaffolding usam.

---

## 06 · O script de sync

É um arquivo Node puro, sem dependências externas, sem internet. Lê `.ai/` e escreve os arquivos de cada plataforma. Roda em qualquer máquina com Node instalado.

### Regras de conversão

- `.ai/commands/` → slash commands em `.claude/skills/`, `.github/prompts/`, `.windsurf/workflows/` e `.cursor/rules/3xx`
- `.ai/playbooks/` + `.ai/skills/` → referência consultável em `.cursor/rules/5xx`, `.windsurf/rules/ref-*` e `.claude/skills/ref-*`

### Frontmatter — command (vira slash command)

```markdown
---
name: gerar-specs
description: Gera specs técnicas
slash-command: /gerar-specs
platforms: [claude, windsurf, cursor, copilot]
---
Conteúdo do comando aqui...
```

### Frontmatter — playbook / skill (contexto, NÃO vira slash command)

```markdown
---
name: criar-feature
description: Como criar as partes de uma feature
type: playbook
---
Conteúdo de referência aqui...
```

### Código completo do `sync-ai.mjs`

```javascript
#!/usr/bin/env node
/**
 * sync-ai.mjs — Sincroniza .ai/ (fonte de verdade) para todas as plataformas.
 * Uso: node scripts/sync-ai.mjs
 *
 * REGRAS:
 *  - .ai/commands/  → viram SLASH COMMANDS (você dispara: /comando)
 *                     destinos: .windsurf/workflows, .github/prompts,
 *                               .claude/skills, .cursor/rules (agent-requested)
 *  - .ai/playbooks/ → CONTEXTO/REFERÊNCIA (o agente consulta sozinho)
 *  - .ai/skills/    → CONTEXTO/REFERÊNCIA (padrões de código)
 *                     playbooks+skills viram regras consultáveis em cada plataforma
 *  - .ai/context/   → arquitetura + convenções (referência)
 *
 * Sem dependências externas. Roda com Node puro.
 */

import fs from 'node:fs';
import path from 'node:path';

const ROOT = path.resolve(import.meta.dirname, '..');
const AI = path.join(ROOT, '.ai');

function read(p) { return fs.readFileSync(p, 'utf8'); }
function write(p, content) {
  fs.mkdirSync(path.dirname(p), { recursive: true });
  fs.writeFileSync(p, content);
  console.log('  ✓', path.relative(ROOT, p));
}
function listMd(dir) {
  const full = path.join(AI, dir);
  if (!fs.existsSync(full)) return [];
  return fs.readdirSync(full).filter(f => f.endsWith('.md'))
    .map(f => ({ name: f.replace(/\.md$/, ''), path: path.join(full, f) }));
}
function parse(filePath) {
  const raw = read(filePath);
  const m = raw.match(/^---\n([\s\S]*?)\n---\n?([\s\S]*)$/);
  if (!m) return { meta: {}, body: raw.trim() };
  const meta = {};
  for (const line of m[1].split('\n')) {
    const kv = line.match(/^(\w[\w-]*):\s*(.*)$/);
    if (!kv) continue;
    let [, k, v] = kv; v = v.trim();
    if (v.startsWith('[') && v.endsWith(']'))
      meta[k] = v.slice(1, -1).split(',').map(s => s.trim()).filter(Boolean);
    else meta[k] = v.replace(/^["']|["']$/g, '');
  }
  return { meta, body: m[2].trim() };
}
function has(meta, p) { return !meta.platforms || meta.platforms.includes(p); }

const commands  = listMd('commands').map(x => ({ ...x, ...parse(x.path) }));
const playbooks = listMd('playbooks').map(x => ({ ...x, ...parse(x.path) }));
const skills    = listMd('skills').map(x => ({ ...x, ...parse(x.path) }));

console.log(`\nFonte: ${commands.length} commands, ${playbooks.length} playbooks, ${skills.length} skills\n`);

// ============ COMMANDS → SLASH COMMANDS ============

// Claude Code: .claude/skills/<nome>/SKILL.md  (invocável como /nome)
console.log('Claude Code (slash commands):');
for (const c of commands.filter(c => has(c.meta, 'claude')))
  write(path.join(ROOT, '.claude/skills', c.name, 'SKILL.md'),
    `---\nname: ${c.name}\ndescription: ${c.meta.description || ''}\n---\n\n${c.body}\n`);

// Copilot: .github/prompts/<nome>.prompt.md  (só VS Code/VS/JetBrains)
console.log('GitHub Copilot (slash commands - só IDE):');
for (const c of commands.filter(c => has(c.meta, 'copilot')))
  write(path.join(ROOT, '.github/prompts', `${c.name}.prompt.md`),
    `---\nmode: ask\ndescription: ${c.meta.description || ''}\n---\n\n${c.body}\n`);

// Windsurf: .windsurf/workflows/<nome>.md  (invocável como /nome)
console.log('Windsurf (slash commands):');
for (const c of commands.filter(c => has(c.meta, 'windsurf')))
  write(path.join(ROOT, '.windsurf/workflows', `${c.name}.md`),
    `---\ndescription: ${c.meta.description || ''}\n---\n\n${c.body}\n`);

// Cursor: .cursor/rules/<nnn>-<nome>.mdc agent-requested (Cursor não tem /command nativo)
console.log('Cursor (regras agent-requested):');
let ci = 300;
for (const c of commands.filter(c => has(c.meta, 'cursor'))) {
  write(path.join(ROOT, '.cursor/rules', `${ci}-${c.name}.mdc`),
    `---\ndescription: "Comando: ${c.meta.description || ''}"\nalwaysApply: false\n---\n\n${c.body}\n`);
  ci += 10;
}

// ============ PLAYBOOKS + SKILLS → CONTEXTO/REFERÊNCIA ============
const refs = [...playbooks, ...skills];

// Cursor: regras agent-requested (LLM puxa quando o comando precisar)
console.log('Cursor (playbooks/skills como referência):');
let ri = 500;
for (const r of refs) {
  write(path.join(ROOT, '.cursor/rules', `${ri}-${r.name}.mdc`),
    `---\ndescription: "${r.meta.type || 'ref'}: ${r.meta.description || ''}"\nalwaysApply: false\n---\n\n${r.body}\n`);
  ri += 10;
}

// Windsurf/Devin: regras (markdown puro)
console.log('Windsurf/Devin (playbooks/skills como referência):');
for (const r of refs)
  write(path.join(ROOT, '.windsurf/rules', `ref-${r.name}.md`),
    `# ${r.name}\n\n${r.body}\n`);

// Claude: skills agent-invocable (Claude decide usar) — mesmo formato, sem slash
console.log('Claude (playbooks/skills como referência):');
for (const r of refs)
  write(path.join(ROOT, '.claude/skills', `ref-${r.name}`, 'SKILL.md'),
    `---\nname: ref-${r.name}\ndescription: ${r.meta.description || ''}\n---\n\n${r.body}\n`);

console.log('\n✅ Sync completo.\n');
console.log('Lembrete: no Devin Cloud, copie .ai/playbooks/ e .ai/commands/');
console.log('para a UI (app.devin.ai). Veja .devin/LEIA-ME.md\n');
```

---

## 07 · O que um dev novo faz

Exemplo: um dev que usa Devin chega ao time e quer plugar a configuração no projeto dele.

1. **Clona o repositório de config** — o repo com `.ai/`, `AGENTS.md`, scripts e docs.
2. **Copia as pastas para a raiz do projeto dele** — ou usa o script de setup, escolhendo "Devin Desktop".
3. **Roda o setup** — o script garante as pastas e roda `node scripts/sync-ai.mjs`, gerando `.devin/` e `.windsurf/`.
4. **Abre o projeto no Windsurf / Devin Desktop** — os slash commands (`/criar-tarefa`, `/gerar-specs`…) já aparecem.
5. **Se usar Devin Cloud:** cola o conteúdo de `.ai/commands/` e `.ai/playbooks/` na UI (app.devin.ai) — único passo manual; está explicado em `.devin/LEIA-ME.md`.

Se o dev usasse Copilot, o passo 2 escolheria "GitHub Copilot", geraria `.github/`, e ele abriria no VS Code. Mesma lógica para Cursor e Claude Code.

---

## 08 · Instruções para a IA aplicar isto

Esta seção é o resumo acionável para a IA reproduzir a estrutura a partir do repositório atual.

1. **Inventarie o que já existe** — liste tudo dentro das pastas `.devin/` e `.windsurf/` atuais: skills, playbooks e comandos SDD.
2. **Crie a pasta fonte `.ai/`** com as subpastas `commands/`, `playbooks/`, `skills/` e `context/`.
3. **Classifique e mova cada arquivo existente:** comandos SDD (gerar tarefa/PRD/specs/implementação/review) → `commands/`; fluxos do produto (criar feature, chamada de API) → `playbooks/`; padrões de código (classe constante, propriedade) → `skills/`.
4. **Adicione o frontmatter correto a cada arquivo** — commands recebem `slash-command` + `platforms`; playbooks/skills recebem `type`. Veja os modelos na seção 06.
5. **Crie o `AGENTS.md`, o `sync-ai.mjs` e os arquivos de config fixa** — CLAUDE.md, copilot-instructions.md, regra 000 do Cursor, regra base do Windsurf, .devin/.
6. **Rode `node scripts/sync-ai.mjs`** e confira que as pastas das plataformas foram geradas corretamente.
7. **Comite tudo**, inclusive as pastas geradas, para que quem clonar não precise rodar nada.

> **Princípio guia:** Nunca duplique conteúdo entre plataformas. Tudo nasce em `.ai/`. Se algo precisa mudar, muda lá e roda o sync. As pastas `.claude/`, `.github/`, `.cursor/` e `.windsurf/workflows/` são saída descartável e regenerável.

---

## Anexo · Templates dos arquivos fonte

Modelos prontos para preencher em `.ai/`. Substitua os placeholders `[...]` pelo conteúdo real.

### `.ai/commands/gerar-specs.md`

```markdown
---
name: gerar-specs
description: Gera especificações técnicas a partir do PRD
slash-command: /gerar-specs
platforms: [claude, windsurf, cursor, copilot]
---

# Gerar Specs

A partir do PRD aprovado, gere specs técnicas:
1. Modelo de dados / contratos
2. Endpoints / interfaces afetadas
3. Fluxo de implementação passo a passo
4. Casos de teste

IMPORTANTE: verifique os playbooks em `.ai/playbooks/` (ex: criar feature,
chamada de API) e as skills em `.ai/skills/` (padrões de classe, propriedade)
e aplique os padrões pertinentes.
```

### `.ai/commands/implementar.md`

```markdown
---
name: implementar
description: Implementa a feature seguindo as specs
slash-command: /implementar
platforms: [claude, windsurf, cursor, copilot]
---

# Implementar

Implemente a feature seguindo as specs geradas.

ANTES de escrever código, verifique:
- `.ai/playbooks/` → o fluxo correto (ex: como criar as partes de uma feature,
  como fazer uma chamada de API)
- `.ai/skills/` → os padrões de código (classe constante, propriedade no padrão)

Aplique os padrões que se encaixam. Escreva testes junto (AAA).
Rode build + testes + lint antes de finalizar.
```

### `.ai/playbooks/criar-feature.md`

```markdown
---
name: criar-feature
description: Como criar as partes de uma feature
type: playbook
---

# Playbook: Criar Feature

Para criar as partes de uma feature:
1. [parte 1 - ex: criar o handler]
2. [parte 2 - ex: registrar no manager]
3. [parte 3]

Use os padrões das skills: classe-constante, propriedade-padrao.
```

### `.ai/skills/classe-constante.md`

```markdown
---
name: classe-constante
description: Padrão para criar classes constantes
type: skill
---

# Padrão: Classe Constante

Quando criar uma classe constante:
1. [seu padrão de nomenclatura]
2. [estrutura esperada]
3. [exemplo de código]
```

### `AGENTS.md` (raiz — contexto universal)

```markdown
# AGENTS.md

> Arquivo universal lido por: Claude Code, Cursor, Windsurf, GitHub Copilot,
> Devin, Gemini CLI, OpenAI Codex, Aider e outros.

## Projeto
Nome: [nome]
Objetivo: [o que faz]

## Stack
- Linguagem: [...]
- Framework: [...]
- Gerenciador de pacotes: [...]

## Comandos essenciais
\`\`\`bash
# Build / Testes / Lint
[...]
\`\`\`

## Convenções
- Commits: tipo(escopo): descrição (Conventional Commits)
- [outras convenções]

## Comandos disponíveis (você dispara)
- `/criar-tarefa` · `/gerar-prd` · `/gerar-specs` · `/implementar` · `/review`

## Referências (o agente consulta sozinho)
- Playbooks em `.ai/playbooks/` · Skills em `.ai/skills/`
```
