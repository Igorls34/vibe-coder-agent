---
name: documentation-sync
description: >
  Após qualquer mudança significativa de código — nova funcionalidade,
  modificação de API, mudança de configuração ou refatoração arquitetural —
  execute esta skill para identificar quais arquivos de documentação ficaram
  desatualizados e atualizá-los. Evita que docs e código se desalinhem.
---

<!--
  DESCRIÇÃO
  ───────────────
  Esta skill detecta e atualiza quais arquivos de documentação ficaram
  desatualizados após mudanças importantes no código. README, documentação de
  API, .env.example, CHANGELOG e diagramas de arquitetura são revisados e as
  informações obsoletas são corrigidas. Fecha o problema do "abismo entre docs e
  código".

  QUANDO: Quando uma API é adicionada ou alterada, quando uma nova env variable
          é criada, após uma refatoração grande, antes ou depois do merge.
  SAÍDA:   Arquivos de doc atualizados + relatório de resumo das mudanças.
-->

# Skill de Sincronização de Documentação

## Quando é Disparado

Execute após:
- Um endpoint público de API ser adicionado, alterado ou removido
- Um comando ou flag de CLI ser modificado
- Uma nova variável de ambiente ou chave de config ser introduzida
- Um componente ou módulo principal ser refatorado
- Um pacote npm/pip/etc. novo ser adicionado ou removido das dependências
- Qualquer referência em `README.md` ficar imprecisa

## Processo Passo a Passo

### 1. Identifique o que Mudou

Revise o git diff do trabalho recém-concluído:

```bash
git diff main --name-only          # Arquivos alterados em relação à main
git diff HEAD~1 --name-only        # Arquivos alterados no último commit
```

Categorize as mudanças:
- **Mudanças de API** → afetam `docs/api/`, specs OpenAPI/Swagger, `README.md`
- **Mudanças de config** → afetam `.env.example`, `docs/configuration.md`
- **Mudanças de dependências** → afetam `README.md` (seção de setup), `docs/getting-started.md`
- **Mudanças de arquitetura** → afetam `docs/architecture.md`, diagramas
- **Novas funcionalidades** → afetam `docs/features/`, changelogs

### 2. Audite a Documentação Existente

Para cada categoria afetada, verifique se a documentação correspondente existe:

```
docs/
  api/             ← docs de endpoints REST/GraphQL
  architecture.md  ← design do sistema, diagrama de componentes
  configuration.md ← todas as env vars e opções de config
  getting-started.md ← guia de setup local
  features/        ← descrições por funcionalidade
CHANGELOG.md       ← histórico de versões
README.md          ← visão geral do projeto + quickstart
```

Para cada arquivo de doc que toca a área alterada, leia-o e sinalize as seções desatualizadas.

### 3. Atualize a Documentação — Ordem de Prioridade

Atualize nesta ordem (do maior impacto para o menor):

#### A. README.md
- As instruções de setup continuam precisas?
- A seção de stack de tecnologia reflete a stack atual?
- Ainda há funcionalidades removidas sendo mencionadas?

#### B. .env.example / Documentação de Configuração
Para cada nova env variable adicionada ao código, adicione a entrada correspondente:

```bash
# Descrição do que isto faz
NEW_VAR_NAME=example_value
```

Documente em `docs/configuration.md`:

```markdown
| Variável | Obrigatória | Padrão | Descrição |
|----------|-------------|--------|-----------|
| NEW_VAR_NAME | Sim | — | O que ela controla |
```

#### C. Documentação de API
Para cada mudança de endpoint, atualize o doc correspondente:

```markdown
### POST /api/resource

**Alterado em:** <data ou versão>
**Descrição:** ...
**Request Body:** ...
**Resposta:** ...
**Breaking Change:** Sim/Não
```

#### D. CHANGELOG.md
Adicione uma entrada em `## [Unreleased]`:

```markdown
### Added
- <descrição da nova funcionalidade>

### Changed
- <o que foi modificado e por quê>

### Fixed
- <bug corrigido>

### Removed
- <o que foi removido>
```

Siga o formato do [Keep a Changelog](https://keepachangelog.com).

#### E. Documentação de Arquitetura
Se foi adicionado um novo serviço, módulo ou componente significativo:
- Atualize a lista de componentes em `docs/architecture.md`
- Registre quaisquer novas dependências externas ou integrações
- Se houver um diagrama (Mermaid, Draw.io), atualize-o

### 4. Verifique os Links Internos

Após editar, faça uma verificação rápida de referências internas quebradas:

```bash
# Encontrar links markdown apontando para arquivos inexistentes
grep -r "\[.*\](.*\.md)" docs/ README.md | grep -v "http"
```

Verifique manualmente se os 5 links internos mais usados ainda resolvem.

### 5. Reporte as Mudanças

Gere um resumo:

```
## Sincronização de Documentação Concluída

**Arquivos Atualizados:**
- README.md — seção de setup atualizada
- docs/configuration.md — NEW_VAR_NAME adicionada
- CHANGELOG.md — entrada não lançada adicionada

**Arquivos que Podem Precisar de Revisão Manual:**
- docs/architecture.md — o diagrama pode estar desatualizado (atualização manual necessária)

**Sem necessidade de mudanças:**
- docs/api/ — nenhuma mudança de endpoint detectada
```

## Regras

- **Nunca exclua documentação** sem confirmar com o usuário. Arquive ou marque como obsoleta.
- **Não invente informações.** Se não souber o que uma variável de config faz, escreva `# TODO: documentar isto` e sinalize.
- **Mantenha tudo em `[Unreleased]` no CHANGELOG.md** até um bump/release explícito de versão.
- **Toda nova env variable precisa aparecer no `.env.example`** — sem exceções.
- **"Docs primeiro" tudo bem, "docs nunca" não.** Mesmo uma documentação mínima é melhor do que nenhuma.
