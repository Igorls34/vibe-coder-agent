---
name: github
description: >
  Gerencia todas as operações git (commit, push, gestão de branches, criação de
  PRs) de forma padronizada, segura e consistente. Executa automaticamente a
  skill code-review antes de qualquer commit. Impõe o padrão Conventional Commits.
  Cobre o ciclo completo: branch → revisão → commit → push → PR.
---

<!--
  DESCRIÇÃO
  ───────────────
  Esta skill gerencia todas as operações git (commit, push, gestão de branches,
  criação de PRs) de forma padronizada, segura e consistente. Executa
  automaticamente a skill `code-review` antes do commit. Impõe o padrão
  Conventional Commits. Em vez de "está funcionando, enviei", diz
  "revisado, dentro do padrão, documentado, enviado".

  QUANDO: Sempre que for executar uma operação git — commit, push, criação de
          branch, abertura de PR, tag. Nenhum comando git é executado
          diretamente, pulando esta skill.
  SAÍDA:   Histórico de commits limpo, descrição do PR, CHANGELOG atualizado.
-->


# Skill de GitHub

## Regra Principal

**Nenhum commit é enviado sem executar a skill `code-review`.**  
Essa regra não pode ser violada. Não existe exceção para "mudança pequena" ou "só uma linha".

---

## Padrão de Commit — Conventional Commits

### Formato

```
<type>(<scope>): <subject>

[body — opcional]

[footer — opcional]
```

### Lista de Tipos

| Tipo | Quando Usar |
|------|-------------|
| `feat` | Nova funcionalidade |
| `fix` | Correção de bug |
| `refactor` | Reorganização de código sem mudança de comportamento |
| `style` | Formatação, espaços (sem mudança funcional) |
| `test` | Adição ou ajuste de testes |
| `docs` | Mudança de documentação |
| `chore` | Atualização de build, dependências, config |
| `perf` | Melhoria de performance |
| `ci` | Mudança no pipeline de CI/CD |
| `revert` | Reverter um commit anterior |

### Escopo (Opcional, mas Recomendado)

Módulo, componente ou camada afetado:
```
feat(auth): add refresh token rotation
fix(userService): handle null email in validation
refactor(Button): extract loading state to hook
chore(deps): upgrade react to 19.2.0
```

### Regras de Subject

- **Use o modo imperativo**: "add" e não "added", "fix" e não "fixed"
- **Comece em minúscula**
- **Não termine com ponto**
- **Limite de 72 caracteres**
- **Diga o que** mudou; explique **por que** no body

### Body (Por Que Mudou?)

```
feat(auth): add refresh token rotation

Se um refresh token estático for roubado, ele dá acesso indeterminado.
Com rotation, um novo token é gerado a cada uso e o anterior é invalidado.
Implementado conforme a recomendação de segurança da RFC 6749.
```

### Breaking Change

```
feat(api)!: change user ID format from integer to UUID

BREAKING CHANGE: todos os endpoints /api/users/:id agora esperam uma string UUID.
Clientes que usam ID inteiro precisam ser atualizados.

Migration guide: docs/migrations/integer-to-uuid.md
```

---

## Fluxo de Trabalho Completo

### 1. Criar a Branch

```bash
# Formato: <type>/<descrição-curta>
git checkout -b feat/user-registration
git checkout -b fix/token-refresh-null-check
git checkout -b refactor/button-loading-state
```

Regras de nomenclatura de branch:
- Minúsculas, kebab-case
- Começa com o tipo (`feat/`, `fix/`, `refactor/`, `hotfix/`)
- Nunca faça commit direto em `main` ou `master`

---

### 2. Executar o Code Review

Antes de fazer o commit, **abra a skill `code-review` e aplique o MODO A**.

```bash
# Primeiro verifique o que mudou
git status
git diff --staged
```

Não se avança para o passo 3 sem concluir o checklist de code review.

---

### 3. Stage e Commit

```bash
# Adicione ao stage os arquivos relevantes — não tudo, em grupos coerentes
git add src/services/userService.ts src/services/userService.test.ts

# Faça o commit — conforme o padrão
git commit -m "feat(userService): add createUser with password hashing

Adicionada a camada de service para o cadastro de novo usuário.
A senha é hasheada com bcrypt (saltRounds: 12).
Em caso de e-mail duplicado, é lançado o DuplicateEmailError.

Testes: userService.test.ts — 8 testes, todos passando"
```

**Commits atômicos:**
- Um commit, uma mudança lógica
- Migration + model + service = 3 commits separados
- Commits "misc fixes" ou "wip" são proibidos

---

### 4. Verificação Após Cada Commit

```bash
git log --oneline -5    # Veja os últimos commits
git show HEAD           # Confira o diff do último commit
```

Se a mensagem de commit estiver errada, corrija imediatamente (antes do push):
```bash
git commit --amend -m "fix(auth): handle null refresh token in rotation"
```

---

### 5. Push

```bash
# Primeiro push — defina o upstream
git push -u origin feat/user-registration

# Pushes seguintes
git push
```

**Antes do push:**
```bash
# Os testes são executados uma última vez
npm test

# Está atrás da main? Faça rebase
git fetch origin
git rebase origin/main
```

---

### 6. Criar o Pull Request

Com o GitHub CLI (template de PR inline — se `.github/PR_TEMPLATE.md` não existir, use o abaixo):

```bash
gh pr create \
  --title "feat(auth): user registration with email verification" \
  --base main \
  --label "feature" \
  --body "## O que mudou?
<2-3 frases: o que foi feito, por quê>

## Como foi testado?
- [ ] Testes unitários: \`npm test\` — X/X passando
- [ ] Teste de integração: validado manualmente
- [ ] Edge cases foram testados

## Checklist
- [ ] Code review (MODO A) concluído
- [ ] Testes passando
- [ ] CHANGELOG atualizado, se houver BREAKING CHANGE
- [ ] `.env.example` atualizado, se houver nova env variable
- [ ] Skill \`documentation-sync\` executada (se houve mudança de API)

## Relacionados
- Closes #NÚMERO_DA_ISSUE"
```

> **Nota:** Se a equipe criar `.github/PR_TEMPLATE.md`, substitua a parte do body por `--body "$(cat .github/PR_TEMPLATE.md)"`.

---

### 7. Atualizar o CHANGELOG

Para cada PR, adicione à seção `[Unreleased]` do `CHANGELOG.md`:

```markdown
## [Unreleased]

### Added
- User registration endpoint (POST /api/auth/register) with email verification

### Fixed
- Null refresh token causing 500 error on token rotation

### Changed
- User ID format migrated from integer to UUID (breaking)
```

---

## Situações Especiais

### Hotfix (Emergência em Produção)
```bash
# Abra uma branch de hotfix a partir da main
git checkout -b hotfix/null-pointer-login main

# Corrija → teste → commit
git commit -m "fix(auth): prevent null pointer when user session expires"

# Faça merge na main e na branch de development ativa
git checkout main && git merge hotfix/null-pointer-login
git checkout develop && git merge hotfix/null-pointer-login
```

### Reverter um Commit (Antes do Push)
```bash
git reset HEAD~1 --soft    # Reverte o commit, as alterações continuam staged
```

### Tag e Release
```bash
git tag -a v1.2.0 -m "Release v1.2.0 — user registration feature"
git push origin v1.2.0
```

---

## Proibições

- `git push --force` (force push) — apenas em branch pessoal, nunca na `main`
- `git commit -m "fix"` — mensagem de commit sem significado
- `git add .` — adicionar tudo sem conferir o staging
- `git commit --no-verify` — pular os pre-commit hooks
- Push direto na `main` — sem pull request

---

## Referência Rápida

```bash
# Fluxo de trabalho completo
git checkout -b feat/my-feature           # 1. Branch
# → code-review skill (MODO A)            # 2. Revisão
git add <arquivos relevantes>             # 3. Stage
git commit -m "feat(scope): description"  # 4. Commit
git push -u origin feat/my-feature        # 5. Push
gh pr create --title "..." --body "..."   # 6. PR
```
