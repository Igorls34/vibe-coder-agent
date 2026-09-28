# Configuração de Agent do OpenCode

Este arquivo é lido automaticamente pelo OpenCode em todas as sessões.

---

## Primeira Etapa Obrigatória

Em cada nova sessão, leia e aplique **`.agent/skills/project-context-primer/SKILL.md`**. Isso não pode ser pulado.

---

## Regras Críticas

### Segurança
- `DROP`, `DELETE`, `TRUNCATE`, `rm -rf`, `git push --force` (branch principal): **Peça aprovação antes de executar**
- Se os testes atuais estão passando e a tarefa não exige mudar esse código diretamente — **não mexa**
- Antes de migration/production deploy: **defina a etapa de rollback**

### Limpeza de Código
- Comentários apenas no início de funções/métodos
- `console.log`, `console.warn`, `debugger`, `print()`: **remova** antes do commit
- Valores sensíveis (API key, token): **use .env**, nunca escreva no código
- Ao adicionar uma nova variável de ambiente: **atualize o .env.example** (descrição obrigatória)

### Qualidade de Código
- Se o mesmo código se repete em 3+ lugares → **extraia**
- Antes de criar uma nova abstração: "Está sendo usado em 3+ lugares? Vai ser compreensível daqui a 6 meses?" → Se a resposta for não, **deixe como está**
- Refactor: commit separado do commit da feature, commit independente

---

## Fluxo de Trabalho

```
NOVA SESSÃO
  └─ project-context-primer (obrigatório)

IDEIA      → brainstorming → SCOPE-<slug>.md
PLANO      → writing-plans → implementation_plan.md → architecture-review
DEV        → test-driven-execution
COMMIT     → github skill (não EXECUTE comando git diretamente)
           → code-review → conventional commit → PR
```

---

## Estrutura de Skills e Agentes

Diretório **.agent/**:
- `skills/` — definições de processo (cada uma com um SKILL.md)
- `agents/` — definições de personas de especialista
- `rules/` — regras obrigatórias (code-hygiene, code-quality, safety)
- `knowledge/` — conhecimento persistente entre conversas

**Skills principais:**

| Skill | Quando Usar |
|-------|----------|
| `project-context-primer` | No início de cada sessão — obrigatório |
| `github` | Em todas as operações git |
| `code-review` | Antes do commit |
| `brainstorming` | Ao esclarecer ideia/requisito |
| `writing-plans` | Depois que o escopo for aprovado |
| `knowledge-base-update` | A cada coisa aprendida |

**Personas de agentes:** Aria (UI/UX), Felix (Frontend), Bora (Backend), Deva (DevOps)

---

## Comportamento Crítico

- **Na dúvida, pergunte** — não faça premissas
- **Não amplie o escopo silenciosamente** — se encontrar um problema fora da tarefa, comunique
- **"Funciona" ≠ "Está pronto"** — testes passando + código limpo + docs atualizados
- **Se afetarem 5+ arquivos** — mostre o plano primeiro e peça aprovação
