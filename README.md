# Ralph for GitHub Copilot CLI

> **Adaptação do projeto [frankbria/ralph-claude-code](https://github.com/frankbria/ralph-claude-code) para GitHub Copilot CLI**  
> Loop autônomo de IA com detecção inteligente de saída, orquestração multi-agente e suporte nativo aos agentes do Copilot CLI.

---

## O que é o Ralph?

Ralph é um loop autônomo de desenvolvimento baseado em IA, implementado originalmente por [Frank Bria](https://github.com/frankbria) para o Claude Code. Esta adaptação traz o mesmo poder para o **GitHub Copilot CLI** (`copilot` command), com três melhorias fundamentais:

1. **3 bugs críticos corrigidos** para funcionar com o Copilot CLI
2. **Suporte a agentes** (`~/.copilot/agents/`) com troca automática por fase
3. **Orquestração multi-agente via `@fix_plan.md`** — zero configuração extra

---

## Instalação

```bash
git clone https://github.com/xavanty/ralph_copilot.git
cd ralph_copilot
chmod +x install.sh
./install.sh
```

**Pré-requisito:** GitHub Copilot CLI instalado e autenticado.
```bash
copilot --version   # versão ≥ 1.0.0
```

Isso instala os comandos globais: `ralph`, `ralph-setup`, `ralph-monitor`.

---

## Início Rápido

```bash
# 1. Criar novo projeto
ralph-setup meu-projeto
cd meu-projeto

# 2. Editar PROMPT.md com o objetivo do projeto
# 3. Editar @fix_plan.md com as tarefas (veja formato abaixo)
# 4. Executar
ralph -v
```

---

## Os dois arquivos principais

### `PROMPT.md` — Instruções para o agente

O `PROMPT.md` é lido a cada loop e fornece:
- **Contexto do projeto**: o que está sendo construído/gerado
- **Instruções por agente**: o que cada agente deve fazer quando for ativado
- **Ferramentas disponíveis**: `view`, `create`, `edit`, `bash`, `glob`, `grep`
- **Bloco RALPH_STATUS**: formato obrigatório ao final de cada resposta

```markdown
# Instruções Ralph — [Nome do Projeto]

## Contexto
[Descreva o projeto e o objetivo]

## Se você é o `pesquisador`:
[Instruções específicas para o agente pesquisador]

## Se você é o `desenvolvedor`:
[Instruções específicas para o agente desenvolvedor]

## 🎯 Status Reporting (OBRIGATÓRIO ao final de cada resposta)

\`\`\`
---RALPH_STATUS---
STATUS: IN_PROGRESS | COMPLETE | BLOCKED
TASKS_COMPLETED_THIS_LOOP: <número>
FILES_MODIFIED: <número>
TESTS_STATUS: NOT_RUN | PASSING | FAILING
WORK_TYPE: RESEARCH | GENERATION | PUBLICATION | DOCUMENTATION
EXIT_SIGNAL: false | true
COMPONENTS_PROCESSED: <concluídas>/<total>
RECOMMENDATION: <próxima ação em uma linha>
---END_RALPH_STATUS---
\`\`\`

**EXIT_SIGNAL: true** apenas quando TODAS as tarefas `[ ]` da sua seção no `@fix_plan.md` estiverem marcadas `[x]`.
```

### `@fix_plan.md` — Lista de tarefas com agente por seção

O `@fix_plan.md` é o arquivo central do Ralph. Ele define:
- **As tarefas** a serem executadas (`- [ ]` para pendentes, `- [x]` para concluídas)
- **O agente responsável** por cada seção (sintaxe `## [agente] Título`)

**Ralph lê este arquivo a cada loop** e ativa automaticamente o agente certo.

```markdown
# Fix Plan — [Nome do Projeto]

## [pesquisador] Fase 1: Pesquisa
- [ ] Pesquisar [tecnologia] (CIS, NIST, Well-Architected, OWASP)
- [ ] Salvar resultado em docs/pesquisas/[tecnologia].md

## [desenvolvedor] Fase 2: Geração do Documento
- [ ] Ler pesquisa de docs/pesquisas/[tecnologia].md
- [ ] Gerar output/[tecnologia]_doc.html
- [ ] Verificar que não há placeholders restantes

## [publicacao_confluence] Fase 3: Publicação
- [ ] Publicar documento no Confluence
- [ ] Registrar URL em output/paginas_publicadas.txt
```

**Como funciona a detecção de agente:**
1. Ralph lê `@fix_plan.md` linha a linha no início de cada loop
2. Encontra a primeira seção `## [agente]` com pelo menos um `- [ ]` incompleto
3. Ativa esse agente via `--agent <nome>` na chamada ao Copilot CLI
4. Quando a seção completa (todos `[x]`), Ralph detecta a transição automaticamente:
   - Reseta os sinais de saída
   - Inicia nova sessão Copilot
   - Carrega o próximo agente

---

## Fluxo Multi-Agente

```
@fix_plan.md                    Ralph                    Copilot CLI
─────────────────────────────────────────────────────────────────────
## [pesquisador] Fase 1         ──→  --agent pesquisador  ──→  pesquisa
  - [x] Task 1 ✓                                               & salva
  - [x] Task 2 ✓                ←── EXIT_SIGNAL: true ←──

                                 reseta signals, nova sessão
                                 detecta próxima seção incompleta

## [publicacao_confluence] F2   ──→  --agent publicacao_confluence
  - [ ] Task 3                                                  gera SBB
  - [ ] Task 4                  ←── EXIT_SIGNAL: true ←──  e publica

                                 projeto completo — exit 0
```

---

## Agentes compatíveis (`~/.copilot/agents/`)

| Agente | Especialidade |
|--------|--------------|
| `pesquisador` | Pesquisa técnica: CIS, NIST, OWASP, CVEs, documentação oficial |
| `desenvolvedor` | Python, Shell Script, geração de código e documentos HTML |
| `publicacao_confluence` | Modelos ABB, SBB, Blueprint, Modelagem de Ameaça — publica no Confluence |
| `security-engineer` | Arquitetura de segurança e modelagem de ameaças |
| `devops-engineer` | CI/CD, Docker, infraestrutura como código |
| `cloud-architect` | Arquitetura multi-cloud, AWS/Azure/GCP |
| `terraform-engineer` | IaC com Terraform |

Qualquer arquivo `.md` em `~/.copilot/agents/` pode ser usado como valor no `## [agente]`.

---

## Bloco RALPH_STATUS — Formato obrigatório

Todo `PROMPT.md` deve instruir o agente a terminar **cada resposta** com:

```
---RALPH_STATUS---
STATUS: IN_PROGRESS | COMPLETE | BLOCKED
TASKS_COMPLETED_THIS_LOOP: <número>
FILES_MODIFIED: <número>
TESTS_STATUS: PASSING | FAILING | NOT_RUN
WORK_TYPE: RESEARCH | GENERATION | DOCUMENTATION | PUBLICATION
EXIT_SIGNAL: false | true
COMPONENTS_PROCESSED: <concluídas>/<total>
RECOMMENDATION: <próxima ação em uma linha>
---END_RALPH_STATUS---
```

| Campo | Quando `true` / valor específico |
|-------|----------------------------------|
| `EXIT_SIGNAL: true` | TODAS as tarefas `[ ]` da seção atual no `@fix_plan.md` estão `[x]` |
| `STATUS: BLOCKED` | Dependência externa ausente (ex: credenciais, arquivo faltando) |
| `STATUS: COMPLETE` | Projeto inteiro concluído |

---

## Configuração por projeto (`.ralph.env`)

Crie `.ralph.env` no diretório do projeto para sobrescrever configurações:

```bash
# .ralph.env — opcional, para projetos sem ## [agente] no fix_plan
COPILOT_AGENT="desenvolvedor"                          # agente fixo (fallback)
COPILOT_ALLOWED_TOOLS="create,view,edit,bash,glob,grep"  # ferramentas disponíveis
```

> **Nota**: quando `@fix_plan.md` usa a sintaxe `## [agente]`, o `COPILOT_AGENT` do `.ralph.env` é ignorado e o agente é sempre lido do fix_plan.

---

## Exemplos

### `examples/abb-multiagente/`
Pipeline completo ABB com 3 agentes:
```
pesquisador → desenvolvedor → publicacao_confluence
```
- `.ralph.env` com configuração
- `prompts/01_pesquisa.md`, `prompts/02_geracao.md`, `prompts/03_publicacao.md`
- `@fix_plan.md` com seções `## [agente]`

---

## Diferenças em relação ao projeto original

| Aspecto | frankbria/ralph-claude-code | xavanty/ralph_copilot |
|---------|-----------------------------|-----------------------|
| Runtime de IA | Claude Code | GitHub Copilot CLI |
| Nomes de ferramentas | `write`, `read`, `shell` | `create`, `view`, `edit`, `bash` |
| Continuidade de sessão | `--continue` / `--resume` | `COPILOT_USE_CONTINUE=false` (evita conflito com sessão pai) |
| Detecção de conclusão | keywords + EXIT_SIGNAL | EXIT_SIGNAL explícito (keywords ignoradas quando EXIT_SIGNAL: false) |
| Agentes | Claude Code subagents | `~/.copilot/agents/` + `--agent` flag |
| Orquestração multi-agente | não nativo | via `## [agente]` no `@fix_plan.md` |

### Bugs corrigidos

**Bug #1 — `has_completion_signal` falso positivo** (`lib/response_analyzer.sh`)  
Keyword detection era executada mesmo quando `EXIT_SIGNAL: false` estava explícito no RALPH_STATUS,  
causando saída prematura do loop.  
*Fix: detecção por keywords só ocorre quando não há EXIT_SIGNAL explícito.*

**Bug #2 — Conflito de sessão HTTP 400** (`ralph_loop.sh`)  
Loop 2 tentava resumir a sessão pai do Copilot CLI (a sessão do usuário atual),  
que tinha `tool_use` pendentes → erro HTTP 400 "tool_use without tool_result".  
*Fix: `COPILOT_USE_CONTINUE=false` — cada loop começa sessão nova.*

**Bug #3 — Nomes de ferramentas incorretos** (`ralph_loop.sh`)  
`--available-tools` usava nomes do Claude Code (`write`, `read`, `shell`) que não existem no Copilot CLI,  
desativando efetivamente todas as ferramentas de I/O.  
*Fix: `COPILOT_ALLOWED_TOOLS="create,view,edit,bash,glob,grep"`*

---

## Estrutura do projeto

```
ralph_copilot/
├── ralph_loop.sh          # Loop principal — lê @fix_plan.md, detecta agente, chama copilot
├── install.sh             # Instala ralph, ralph-setup, ralph-monitor globalmente
├── setup.sh               # Usado por ralph-setup para criar novo projeto
├── lib/
│   ├── response_analyzer.sh  # Analisa saída JSONL do copilot, detecta RALPH_STATUS
│   ├── circuit_breaker.sh    # Previne loops infinitos (abre após N loops sem progresso)
│   └── date_utils.sh         # Utilitários de data/hora
├── templates/
│   ├── PROMPT.md          # Template de PROMPT.md para novos projetos
│   ├── fix_plan.md        # Template de @fix_plan.md para novos projetos
│   ├── AGENT.md           # Template de @AGENT.md
│   └── modelo_abb.html    # Template HTML para geração de ABBs de segurança
├── examples/
│   └── abb-multiagente/   # Exemplo completo: pesquisador → desenvolvedor → publicacao_confluence
└── README-COPILOT.md      # Documentação detalhada das adaptações
```

---

## Créditos

Baseado em **[ralph-claude-code](https://github.com/frankbria/ralph-claude-code)** por [@frankbria](https://github.com/frankbria).  
Adaptado para GitHub Copilot CLI por [@xavanty](https://github.com/xavanty).

---

## Licença

MIT — veja [LICENSE](LICENSE)
