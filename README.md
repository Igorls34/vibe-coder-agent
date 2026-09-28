<div align="center">

# ⚡ Vibe Coder Kit

**Agent Framework para Desenvolvimento Profissional de Software com IA**

*Transforme seu assistente de código com IA em uma equipe de desenvolvimento disciplinada e focada em processo.*

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![OpenCode Compatible](https://img.shields.io/badge/OpenCode-Compatible-purple.svg)](https://opencode.ai)
[![PRs Welcome](https://img.shields.io/badge/PRs-Welcome-brightgreen.svg)](#como-contribuir)

</div>

---

## 🎯 O Que é o Vibe Coder Kit?

O Vibe Coder Kit é um **agent framework orientado a configuração** que transforma assistentes de código com IA (como o [OpenCode](https://opencode.ai)) em uma equipe de desenvolvimento estruturada e focada em processo. Em vez de escrever prompts improvisados, você tem:

- 🤖 **4 Personas de Agentes Especialistas** — Designer, Desenvolvedor Frontend, Desenvolvedor Backend, DevOps
- 🛠️ **12 Skills Comprovadas** — Fluxos de trabalho estruturados, da geração de ideias à gestão de crises
- 📏 **3 Conjuntos de Regras** — Padrões de higiene de código, qualidade e segurança
- 🧠 **Base de Conhecimento Persistente** — Conhecimento compartilhado entre sessões, o que evita erros repetidos

> **"Funciona" ≠ "Está pronto".** Testes passando + código limpo + documentação atualizada = está pronto.

---

## ✨ Recursos em Destaque

### 🎭 Personas de Agentes

| Agente | Nome | Área de Especialização |
|-------|------|---------------|
| 🎨 **Designer** | Aria | UI/UX, design de movimento, sistemas de design, acessibilidade |
| 💻 **Frontend** | Felix | React/Vue, arquitetura CSS, performance, Core Web Vitals |
| ⚙️ **Backend** | Bora | Design de API, bancos de dados, auth, segurança, microsserviços |
| 🚀 **DevOps** | Deva | Docker, CI/CD, infraestrutura em cloud, monitoramento, gestão de crises |

<!-- Cada agente foi projetado como um especialista sênior em sua própria área. Consulte o diretório .agent/agents/ para mais detalhes. -->

### 🛠️ Skills (Fluxos de Trabalho Estruturados)

| Skill | Quando Usar | O Que Faz |
|-------|---------------------|----------|
| `project-context-primer` | No início de cada sessão *(obrigatório)* | Carrega o contexto do projeto, as decisões e as armadilhas |
| `prompt-enhancer` | Solicitações ambíguas ou incompletas | Clareia o escopo e preenche as dimensões que faltam |
| `brainstorming` | Novas funcionalidades ou ideias de projeto | Refina com método socrático → gera um documento de escopo |
| `writing-plans` | Após a aprovação do escopo | Cria um plano de implementação com checkpoints |
| `architecture-review` | Antes de começar a escrever código | Submete o plano a um teste de estresse e sinaliza os riscos |
| `test-driven-execution` | Na implementação de cada funcionalidade | Aplica a abordagem teste primeiro, código depois |
| `code-review` | Antes de cada commit | Checklist pré-commit + resposta ao feedback |
| `github` | Em todas as operações git | Conventional commits, gestão de branches, fluxo de trabalho de PR |
| `knowledge-base-update` | Sempre que algo importante for aprendido | Armazenamento de conhecimento persistente entre sessões |
| `documentation-sync` | Após mudanças de código importantes | Garante a conformidade entre documentação e código |
| `dependency-audit` | Antes do release / rotina mensal | Varredura de vulnerabilidades e pacotes desatualizados |
| `incident-response` | Em problemas de produção | Triagem → contenção → correção → post-mortem |

<!-- As skills podem ser usadas de forma independente, mas para obter o máximo de produtividade, execute-as na ordem recomendada. -->

### 📏 Regras (Padrões Obrigatórios)

| Regra | Finalidade |
|-------|-------|
| `code-hygiene` | Estilo de comentários, limpeza de logs de depuração, tratamento de dados sensíveis |
| `code-quality` | DRY sem overengineering, limite de abstração |
| `safety` | Aprovação de comandos destrutivos, planos de rollback |

<!-- As regras não são recomendações, são obrigatórias. O agente aplica essas regras em todas as tarefas. -->

---

## 🚀 Instalação e Uso

Você pode usar o Vibe Coder Kit de duas formas: **integrando-o ao seu projeto existente** ou **iniciando um novo projeto**.

### Método 1: Integrar ao Projeto Existente (Recomendado)

Se você já tem um projeto, basta copiar os arquivos de configuração de agente para o seu projeto:

```bash
# 1. Baixe o repositório (em uma pasta temporária)
git clone https://github.com/omergocmen/vibe-coder-kit.git /tmp/vibe-coder-kit

# 2. Copie o diretório .agent para o seu projeto
cp -r /tmp/vibe-coder-kit/.agent /caminho/do/seu/projeto/

# 3. Copie o arquivo de configuração do OpenCode
cp /tmp/vibe-coder-kit/opencode.json /caminho/do/seu/projeto/

# 4. Copie o arquivo AGENTS.md (o OpenCode o lê automaticamente)
cp /tmp/vibe-coder-kit/AGENTS.md /caminho/do/seu/projeto/

# 5. Limpe os arquivos temporários
rm -rf /tmp/vibe-coder-kit

# 6. Vá para o diretório do projeto
cd /caminho/do/seu/projeto
```

**Usuários de Windows (PowerShell):**

```powershell
# 1. Baixe o repositório
git clone https://github.com/omergocmen/vibe-coder-kit.git "$env:TEMP\vibe-coder-kit"

# 2. Copie o diretório .agent para o seu projeto
Copy-Item -Recurse "$env:TEMP\vibe-coder-kit\.agent" "C:\caminho\do\seu\projeto\.agent"

# 3. Copie os arquivos de configuração
Copy-Item "$env:TEMP\vibe-coder-kit\opencode.json" "C:\caminho\do\seu\projeto\"
Copy-Item "$env:TEMP\vibe-coder-kit\AGENTS.md" "C:\caminho\do\seu\projeto\"

# 4. Limpe os arquivos temporários
Remove-Item -Recurse "$env:TEMP\vibe-coder-kit"
```

### Método 2: Iniciar como Novo Projeto

Se você está começando um novo projeto do zero, clone o repositório diretamente e construa sobre ele:

```bash
# Clone o repositório
git clone https://github.com/omergocmen/vibe-coder-kit.git meu-projeto
cd meu-projeto

# Agora .agent/, opencode.json e AGENTS.md já estão prontos
# Adicione seu próprio código sobre essa estrutura de diretórios
```

### O Que Fazer Depois de Baixar?

1. **[Instale o OpenCode](https://opencode.ai)** — Se você ainda não tiver instalado:
   ```bash
   # Instalação global via npm
   npm install -g opencode

   # ou execute diretamente
   npx opencode
   ```

2. **Inicie o OpenCode no diretório do projeto:**
   ```bash
   cd /caminho/do/seu/projeto
   opencode
   ```

3. **Na primeira sessão**, o OpenCode lerá automaticamente o arquivo `AGENTS.md` e executará a skill `project-context-primer`. O agente fará perguntas sobre o projeto e carregará o contexto.

4. **Inicie a base de conhecimento** — Na primeira sessão, o arquivo `.agent/knowledge/INDEX.md` é criado. A partir daí, todas as decisões, regras e armadilhas passam a ser registradas automaticamente.

### Customização da Estrutura

Você pode fazer customizações específicas para o seu projeto:

- **Edite as personas de agente:** altere os arquivos do diretório `.agent/agents/` conforme a necessidade do seu projeto
- **Adicione uma nova skill:** crie `.agent/skills/nova-skill/SKILL.md`
- **Torne as regras mais rigorosas:** edite os arquivos do diretório `.agent/rules/`
- **Remova os agentes desnecessários:** por exemplo, se você só faz desenvolvimento backend, pode excluir os arquivos `designer.md` e `frontend-dev.md`

<!-- Cada projeto tem necessidades diferentes. Não tenha medo de customizar o framework de acordo com o seu projeto. -->

---

## 📁 Estrutura do Projeto

```
vibe-coder-kit/
├── AGENTS.md                          # Instruções de início de sessão (lidas automaticamente pelo OpenCode)
├── opencode.json                      # Arquivo de configuração do OpenCode
├── .agent/
│   ├── agents/                        # Personas especialistas
│   │   ├── designer.md                # Aria — UI/UX e design de movimento
│   │   ├── frontend-dev.md            # Felix — Engenharia de frontend
│   │   ├── backend-dev.md             # Bora — Backend e sistemas
│   │   └── devops.md                  # Deva — Infraestrutura e operações
│   ├── skills/                        # Fluxos de trabalho estruturados
│   │   ├── project-context-primer/    # Início de sessão (obrigatório)
│   │   ├── prompt-enhancer/           # Refinamento da solicitação
│   │   ├── brainstorming/             # Refinamento de design socrático
│   │   ├── writing-plans/             # Planejamento da implementação
│   │   ├── architecture-review/       # Teste de estresse do plano
│   │   ├── test-driven-execution/     # Fluxo de trabalho TDD
│   │   ├── code-review/               # Revisão antes do commit
│   │   ├── github/                    # Operações git e commits
│   │   ├── knowledge-base-update/     # Memória persistente
│   │   ├── documentation-sync/        # Sincronização da documentação
│   │   ├── dependency-audit/          # Varredura de segurança
│   │   └── incident-response/         # Gestão de crises
│   ├── rules/                         # Padrões obrigatórios
│   │   ├── code-hygiene.md            # Regras de código limpo
│   │   ├── code-quality.md            # Regras de DRY e abstração
│   │   └── safety.md                  # Proteções para operações destrutivas
│   └── knowledge/                     # Memória entre sessões (preenchida automaticamente)
├── .kilo/                             # Área de trabalho de planejamento
│   └── plans/
├── LICENSE
└── README.md
```

---

## 🔄 Fluxo de Trabalho

```
🆕 NOVA SESSÃO
  └─ project-context-primer (obrigatório)

💡 IDEIA     → brainstorming → SCOPE-<slug>.md
📋 PLANO     → writing-plans → implementation_plan.md → architecture-review
🛠️ DEV       → test-driven-execution
✅ COMMIT    → code-review → github skill (conventional commits)
```

<!-- Este fluxo de trabalho é uma sugestão. Você pode pular ou customizar etapas conforme a necessidade do seu projeto. -->

### Exemplo de Fluxo

1. **Iniciar a sessão** → `project-context-primer` carrega o contexto
2. **Definir a funcionalidade** → `brainstorming` gera um documento de escopo
3. **Planejar a implementação** → `writing-plans` cria um plano com checkpoints
4. **Revisar a arquitetura** → `architecture-review` submete o plano a um teste de estresse
5. **Desenvolver** → a implementação é feita com `test-driven-execution`
6. **Revisar o código** → `code-review` executa o checklist pré-commit
7. **Commit e push** → a skill `github` gerencia conventional commits e PRs

---

## 🧠 Persistência de Conhecimento

Um dos recursos mais poderosos. Toda decisão, armadilha, regra e causa raiz de bug importante é armazenada no diretório `.agent/knowledge/` e carregada automaticamente no início da sessão:

| Tipo | O Que é Armazenado |
|-----|-------------|
| `decision` | Escolhas de arquitetura, preferências de tecnologia e suas justificativas |
| `convention` | Regras de nomenclatura, padrões de código e estruturas de diretórios |
| `bug` | Causa raiz de problemas ambíguos e sua solução |
| `gotcha` | Armadilhas em serviços de terceiros, problemas de ambiente e casos extremos (edge cases) |
| `research` | Respostas a perguntas que exigem pesquisa importante |

<!-- Você não precisa redescobrir os mesmos problemas a cada sessão. O conhecimento se acumula e é transmitido. -->

---

## 🤝 Compatibilidade

Desenvolvido para o **[OpenCode](https://opencode.ai)**, mas os conceitos são portáveis:

| Ferramenta | Como Usar |
|------|-----------------|
| **OpenCode** | `opencode.json` + `AGENTS.md` para suporte nativo |
| **Cursor** | Use as personas de agente como custom instructions |
| **Claude Code** | Referencie os arquivos de skill dentro do `CLAUDE.md` |
| **GitHub Copilot** | Adapte as regras às custom instructions |
| **Windsurf** | Carregue as personas como configuração de agente |

---

## 📋 Conventional Commits

Este kit segue o padrão [Conventional Commits](https://www.conventionalcommits.org/):

```
feat(auth): add refresh token rotation
fix(userService): handle null email in validation
refactor(Button): extract loading state to hook
docs(readme): update installation instructions
chore(deps): upgrade react to 19.2.0
```

<!-- As mensagens de commit são escritas em inglês e devem estar no modo imperativo. Veja o arquivo .agent/skills/github/SKILL.md para mais detalhes. -->

---

## 🌐 Idioma Preferido

Todas as instruções de agent, regras e a documentação das skills são escritas em **Português (Brasil)**, incluindo o frontmatter. Essa é uma escolha deliberada para equipes de desenvolvimento que falam português. Contribuições de tradução são bem-vindas.

---

## 🤝 Como Contribuir

Contribuições são muito bem-vindas! Siga a estrutura existente:

1. Faça um fork do repositório
2. Crie um branch de funcionalidade: `feat/nova-funcionalidade`
3. Faça alterações compatíveis com a estrutura existente de skills/agentes/regras
4. Envie um pull request

### Adicionar uma Nova Skill

Crie o arquivo `.agent/skills/nome-da-skill/SKILL.md`:

```markdown
---
name: nome-da-skill
description: >
  Quando esta skill deve ser usada e o que ela produz.
---

# Nome da Skill

## Quando é Acionada
## Processo Passo a Passo
## Regras
```

### Adicionar uma Nova Persona de Agente

Crie o arquivo `.agent/agents/nome-do-agente.md`:

```markdown
# Agente: Nome do Agente

## Identidade
## Áreas de Especialização
## Metodologia de Trabalho
## Padrões de Código
## Nunca Faça
## Skills Utilizadas
```

---

## 📄 Licença

Este projeto está sob a Licença MIT. Veja o arquivo [LICENSE](LICENSE) para mais detalhes.

---

## 🌟 Suporte

Se este kit melhorou o seu fluxo de trabalho de desenvolvimento:

- ⭐ Dê uma star neste repositório
- 🐛 Reporte problemas (aba Issues)
- 💡 Sugira novas skills ou novas personas de agente
- 🔀 Envie um pull request

---

<div align="center">

**⚡ desenvolvido com [omer gocmen](https://github.com/omergocmen)**

*Não apenas software que funciona — software que perdura.*

</div>
