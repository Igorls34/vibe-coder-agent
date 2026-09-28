---
name: dependency-audit
description: >
  Verifica periodicamente as dependências do projeto em busca de vulnerabilidades
  de segurança, pacotes desatualizados e dependências não utilizadas. Produz um
  relatório de ações priorizado. Execute antes de cada release importante e pelo
  menos uma vez por mês.
---

<!--
  DESCRIÇÃO
  ───────────────
  Esta skill varre as dependências do projeto (npm, pip, etc.) em busca de
  vulnerabilidades de segurança, versões antigas e pacotes não utilizados. Lista
  os problemas encontrados por ordem de prioridade e aplica automaticamente
  atualizações seguras. Deve ser executada antes de cada release importante e
  uma vez por mês.

  QUANDO: Antes de cada release de produção. Rotina mensal. Quando um novo CVE
          for anunciado.
  SAÍDA:   .agent/audits/dependency-audit-YYYY-MM-DD.md — relatório de ações priorizado.
-->

# Skill de Auditoria de Dependências

## Quando é Disparado

- Antes de qualquer release de produção
- Na cadência mensal (incluir no planejamento do sprint)
- Depois que um CVE for anunciado publicamente e puder afetar sua stack
- Quando alguém do time diz "devemos atualizar nossos pacotes"
- Antes de integrar um novo contribuidor (baseline conhecido como seguro)

## Processo Passo a Passo

### 1. Detecte o Ecossistema de Pacotes

Identifique quais gerenciadores de pacotes estão em uso:

```
package.json        → npm / yarn / pnpm  (Node.js)
requirements.txt    → pip                (Python)
pyproject.toml      → pip / poetry       (Python)
Gemfile             → bundler            (Ruby)
go.mod              → go modules         (Go)
Cargo.toml          → cargo              (Rust)
```

Execute as etapas de auditoria para cada ecossistema encontrado.

---

### 2. Varredura de Vulnerabilidades de Segurança

#### Node.js
```bash
npm audit
# ou
yarn audit
# ou
pnpm audit
```

#### Python
```bash
pip install pip-audit
pip-audit
```

#### Geral (todos os ecossistemas)
```bash
# Se estiver usando GitHub: verifique a aba Security → alertas do Dependabot
# Se estiver usando Snyk:
snyk test
```

**Faça a triagem de cada achado:**

| Severidade | Ação |
|------------|------|
| Crítica | Corrija imediatamente, antes de qualquer outra coisa |
| Alta | Corrija antes do próximo release |
| Média | Agende para o sprint atual |
| Baixa | Acompanhe e corrija quando mexer no código relacionado |

---

### 3. Varredura de Pacotes Desatualizados

#### Node.js
```bash
npm outdated
```

#### Python
```bash
pip list --outdated
```

Para cada pacote desatualizado, verifique:
1. **Quantas versões majors está atrás?** (1 minor = baixa urgência, 2+ major = alta urgência)
2. **É uma dependência direta ou transitiva?**
3. **O changelog menciona breaking changes?**

---

### 4. Detecção de Dependências Não Utilizadas

#### Node.js
```bash
npx depcheck
```

#### Python
```bash
pip install deptry
deptry .
```

Para cada dependência não utilizada sinalizada:
- Verifique se ela realmente não é usada (algumas são peer deps ou são usadas em runtime via config)
- Se confirmar que não é usada, remova do `package.json` / `requirements.txt`
- Reexecute a aplicação e os testes para confirmar que nada quebrou

---

### 5. Verificação de Conformidade de Licenças

Sinalize quaisquer dependências cujas licenças possam ser incompatíveis com seu projeto:

```bash
# Node.js
npx license-checker --summary

# Python  
pip install pip-licenses
pip-licenses
```

**Níveis de risco de licença:**

| Licença | Risco |
|---------|-------|
| MIT, Apache 2.0, BSD | Seguro para a maioria dos usos |
| LGPL | Geralmente ok, verifique os requisitos de linking |
| GPL | Risco se você distribui software proprietário |
| AGPL | Risco alto — exige open source se usado via rede |
| Desconhecida | Precisa ser investigada antes do uso |

---

### 6. Produza o Relatório de Auditoria

Escreva os achados em `.agent/audits/dependency-audit-<YYYY-MM-DD>.md`:

```markdown
# Auditoria de Dependências — <YYYY-MM-DD>

## Resumo
- **Vulnerabilidades críticas:** <N>
- **Vulnerabilidades altas:** <N>
- **Pacotes desatualizados:** <N>
- **Pacotes não utilizados:** <N>
- **Problemas de licença:** <N>

## Críticas / Altas — Ação Obrigatória

### <package-name> @ <version>
- **CVE:** CVE-XXXX-XXXXX
- **Severidade:** Crítica
- **Descrição:** <o que é a vulnerabilidade>
- **Correção:** Atualizar para <version>
- **Breaking changes:** Sim/Não — <detalhes>

## Pacotes Desatualizados

| Pacote | Atual | Mais Recente | Urgência | Observações |
|--------|-------|--------------|----------|-------------|
| express | 4.17.1 | 5.0.0 | Alta | Versão major — revisar o changelog |
| lodash | 4.17.20 | 4.17.21 | Baixa | Apenas patch |

## Dependências Não Utilizadas (seguras para remover)
- `left-pad` — não é importado em lugar nenhum
- `moment` — substituído por `date-fns` mas não removido

## Problemas de Licença
- `<package>` usa GPL-3.0 — revisar com o time antes do próximo release

## Recomendações
1. <ação priorizada>
2. <ação priorizada>
```

---

### 7. Execute as Correções Seguras

Para pacotes com atualizações de **patch ou minor** e **sem breaking changes**:

```bash
# Node.js — apenas atualizações seguras
npm update

# Python
pip install --upgrade <package>
```

Após atualizar:
1. Execute a suíte de testes completa
2. Faça um smoke test da aplicação em execução
3. Faça o commit com a mensagem: `chore: dependency security updates <YYYY-MM-DD>`

Para atualizações de **versão major**, crie uma branch e uma tarefa separadas.

## Regras

- **Nunca atualize versões major em massa automaticamente** — faça uma de cada vez.
- **Sempre rode os testes após qualquer atualização** — até uma versão patch pode quebrar algo.
- **Guarde o relatório de auditoria** mesmo depois das correções — ele é um registro de conformidade.
- **Vulnerabilidades críticas bloqueiam releases** — sem exceções, a não ser que haja um aval explícito.
- **Pacotes não utilizados devem ser removidos**, não apenas ignorados — eles são superfície de ataque.
