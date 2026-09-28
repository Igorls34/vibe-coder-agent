---
name: code-review
description: >
  Skill de revisão de código que cobre tanto solicitar uma revisão (checklist
  antes do commit) quanto receber e responder a feedbacks de revisão. Verifica
  qualidade de código, segurança, cobertura de testes, alinhamento arquitetural
  e documentação antes de qualquer commit.
---

<!--
  DESCRIÇÃO
  ───────────────
  Esta skill garante que o código escrito seja revisado de forma sistemática
  antes de ser commitado ou enviado (push). Cobre tanto o processo de
  "preparar uma solicitação de revisão" (requesting) quanto o de
  "responder ao feedback de revisão" (receiving).
  São verificados qualidade de código, segurança, cobertura de testes,
  alinhamento com a arquitetura e documentação.

  QUANDO: Antes do commit (auto-revisão), antes de abrir um PR, em mudanças grandes.
  SAÍDA:   Relatório de checklist de revisão, lista de feedbacks respondidos, código corrigido.
-->


# Skill de Revisão de Código

## Dois Modos

Esta skill é acionada em duas situações diferentes:

| Modo | Quando | O que Faz |
|-----|--------|-----------|
| **A — Pre-Commit Review** | Antes do commit, revisando o próprio trabalho | Rode o checklist e decida se está pronto para o commit |
| **B — Feedback Response** | Alguém fez uma revisão e chegou feedback | Classifique cada comentário, responda e corrija |

---

## MODO A: Pre-Commit Review (Auto-revisão)

### 1. Obtenha e analise o diff

```bash
git diff HEAD          # alterações não staged
git diff --staged      # alterações staged
git diff main...HEAD   # todas as alterações da branch
```

Revise cada arquivo alterado. Corrija agora tudo que você não quer receber
comentário sobre no PR.

---

### 2. Checklist — siga em ordem

#### 🔴 Crítico (se houver algum, não faça commit)

- [ ] Há segredo hardcoded, API key ou senha?
  ```bash
  git grep -i "password\|secret\|api_key\|token" -- '*.ts' '*.js' '*.py' '*.env'
  ```
- [ ] Há vetor de SQL injection? (raw query + input do usuário)
- [ ] Há risco de bypass de autenticação ou autorização?
- [ ] Os testes estão passando?
  ```bash
  npm test
  ```
- [ ] O build está quebrado?
  ```bash
  npm run build
  ```
- [ ] Há breaking change que quebra outros módulos? (avise)

#### 🟡 Importante (se houver, corrija ou pule conscientemente com motivo)

- [ ] A nova função/endpoint pública tem JSDoc ou tipagem?
- [ ] O tratamento de erros foi feito? (try-catch, error boundary, tratamento de 404/500)
- [ ] Sobrou algum debug log no console? (`console.log`, `print`, `debugger`)
- [ ] Se foi criada uma nova env variable, ela foi adicionada ao `.env.example`?
- [ ] Sobrou algum TODO ou FIXME? (se proposital, adicione o número da ticket)
- [ ] A cobertura de código caiu?
  ```bash
  npm test -- --coverage
  ```
- [ ] Se foi adicionada uma nova dependência, a verificação de segurança foi feita?
  ```bash
  npm audit
  ```

#### 🔵 Qualidade (para código excelente)

- [ ] Os nomes das funções dizem o que elas fazem?
- [ ] Uma função está fazendo mais de uma coisa? (deveria ser dividida)
- [ ] Houve cópia de código? (copy-paste → extrair para uma função)
- [ ] Há magic number ou magic string? (extrair para uma constante)
- [ ] O uso de tipos é estrito o suficiente? (existe algum `any`?)

---

### 3. Crie o Card de Revisão

Para anexar à mensagem de commit ou à descrição do PR:

```markdown
## Resumo da Auto-revisão

**Mudança:** <o que eu fiz, por quê>
**Status dos Testes:** ✅ Todos os testes passando / ⚠️ X testes ignorados (justificativa: ...)
**Checklist:**
  - 🔴 Crítico: Limpo
  - 🟡 Importante: [observação, se houver]
  - 🔵 Qualidade: [observação, se houver]

**Pontos em que gostaria de atenção especial:**
- <local que você quer que o revisor olhe>

**Atalhos deliberadamente ignorados (deliberate shortcuts):**
- <com a justificativa, se houver>
```

---

## MODO B: Feedback Response (Revisão de Outra Pessoa)

### 1. Classifique Cada Comentário

Para cada comentário de revisão recebido:

```
[CRÍTICO]  → Bloqueia o merge, não é mergeável — corrija imediatamente
[IMPORT.]  → Sugestão forte, aplique se a justificativa for boa
[SUGESTÃO] → Elogio, pode exigir discussão aberta
[PERGUNTA] → Pedido de informação — explique ou corrija no código
[NIT]      → Detalhe de estilo/nomenclatura, depende da sua preferência
```

### 2. Responda Cada Comentário

**Nenhum comentário fica sem resposta.**

```markdown
Comentário: "Esta função não tem timeout, o que acontece se o serviço não responder?"
→ Classe: [CRÍTICO]
→ Resposta: "Você tem razão. Adicionei timeout de 5 segundos + retry. Você pode ver no commit T3."
→ Ação: Corrigido ✅

Comentário: "Talvez aqui você possa usar Set em vez de Map"
→ Classe: [SUGESTÃO]
→ Resposta: "Set realmente faz sentido aqui, não precisamos de key. Mudei."
→ Ação: Corrigido ✅

Comentário: "Essa nomenclatura não me parece semântica"
→ Classe: [NIT]
→ Resposta: "Entendi a lógica, mas `normalizeUserInput` é mais descritivo que `processUserData` — mudei."
→ Ação: Corrigido ✅
```

### 3. Agrupe as Correções e Faça os Commits

Envie cada grupo de correção como um commit separado (conforme o padrão de
commit da skill `github`):

```bash
# Cada correção é um commit
git commit -m "fix: add timeout and retry to userService.fetchProfile"
git commit -m "refactor: use Set instead of Map in deduplication logic"
git commit -m "refactor: rename processUserData to normalizeUserInput"
```

### 4. Solicite a Re-revisão

Quando todas as correções estiverem concluídas, atualize a revisão:

```markdown
## Resumo das Correções

Total de comentários: 8
- Corrigidos: 7
- Rejeitados (com justificativa): 1 → "Comentário NIT: a nomenclatura atual está alinhada à convenção do projeto"

Mudanças relevantes:
- Mecanismo de timeout + retry adicionado (crítico do ponto de vista de segurança)
- 2 refatorações (por clareza)

Pronto para re-revisão.
```

---

## Regras

- **Com item vermelho não há commit.** Sem exceções.
- **"Depois eu corrijo" é proibido.** Ou corrija agora, ou abra uma ticket e adicione o número dela no código.
- **Todo comentário de revisão é respondido.** "OK" ou silêncio não são aceitos — ou "corrigi", ou "não concordo, porque...".
- **A revisão existe para proteger o código, não para se enganar.** Não passe o checklist às pressas.
- **O feedback recebido não é pessoal.** O comentário é sobre o código, não sobre você.
