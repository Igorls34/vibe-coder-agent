<!--
  DESCRIÇÃO
  ───────────────
  Esta skill produz um plano de implementação concreto e aplicável a partir de
  um scope ou de uma ideia já aprovados. Ela esclarece qual arquivo vai mudar,
  em que ordem, quem faz, quanto tempo leva e quais são os riscos. A saída do
  plano entra direto nas skills `executing-plans` ou `dispatching-parallel-agents`.

  QUANDO: Depois que o brainstorming termina e o scope é aprovado, antes de
          começar a escrever código.
  SAÍDA:   implementation_plan.md — com checkpoints, ordenado por dependências
           e com as etapas de risco sinalizadas.
-->

---
name: writing-plans
description: >
  Transforma um scope ou uma ideia aprovados em um plano de implementação
  concreto e executável. Define o que muda, em que ordem, com quais
  dependências, sinaliza os riscos e marca os checkpoints. A saída alimenta
  diretamente executing-plans ou dispatching-parallel-agents.
---

# Skill de Elaboração de Planos

## Pré-requisito

Antes de iniciar esta skill, o seguinte precisa estar pronto:
- [ ] `brainstorming` concluído e com scope doc existente → `.agent/SCOPE-<slug>.md`
- [ ] Critérios de sucesso esclarecidos
- [ ] Tech stack decidida

**Carregue o scope doc:** procure por arquivos no padrão `.agent/SCOPE-*.md` e leia o de data mais recente.
Se nenhum existir, rode primeiro a skill `brainstorming`.

---

## Processo

### Etapa 1: Mapear as Áreas Afetadas

Vasculhe a base de código atual e determine o que vai mudar:

```
src/
├── components/     ← Novo componente: UserCard
├── services/       ← Vai mudar: userService.ts
├── api/            ← Novo endpoint: POST /api/users
├── db/migrations/  ← Nova migration: add_users_table
└── tests/          ← Novos testes: userService.test.ts
```

Para cada arquivo afetado:
- **É novo, está mudando ou está sendo removido?**
- **O que mais depende deste arquivo?** (dá para alterá-lo sem quebrar nada?)

---

### Etapa 2: Dividir as Tarefas em Partes Atômicas

Cada tarefa:
- faz **uma única coisa**
- pode ser um **commit independente**
- é **testável** (como você sabe que terminou?)
- tem **menos de 2 horas** (se levar mais, divida)

```markdown
## Lista de Tarefas

### T1: Database migration — tabela users
- Arquivo: db/migrations/001_create_users.sql
- Conteúdo: id, email, password_hash, created_at, deleted_at
- Dependência: Nenhuma (primeira etapa)
- Teste: Migration up/down funciona

### T2: Model e repository de User
- Arquivo: src/db/repositories/userRepository.ts
- Conteúdo: findById, findByEmail, create, softDelete
- Dependência: T1 (a migration precisa existir)
- Teste: testes unitários do repository passam

### T3: UserService — lógica de negócio
- Arquivo: src/services/userService.ts
- Conteúdo: createUser (hash password), getUser, deleteUser
- Dependência: T2
- Teste: testes unitários do service passam

### T4: Endpoint de API — POST /api/users
- Arquivo: src/api/routes/users.ts
- Conteúdo: validation, auth middleware, chamada ao service
- Dependência: T3
- Teste: teste de integração passa

### T5: Frontend — componente UserCard
- Arquivo: src/components/UserCard/
- Conteúdo: componente + teste + storybook
- Dependência: T4 (API mock ou real)
- Teste: testes do componente passam
```

---

### Etapa 3: Desenhar o Grafo de Dependências

```
T1 (migration)
  └── T2 (repository)
        └── T3 (service)
              └── T4 (API endpoint)
                    └── T5 (frontend)
```

Marque como ramos separados as **tarefas que podem rodar em paralelo**:
```
T1
├── T2 → T3 → T4 (ramo de backend)
└── T5-mock (ramo de frontend — pode começar com API mockada)
```

---

### Etapa 4: Marcar Riscos e Checkpoints

Ao lado de cada tarefa:
- `[RISK]` — se falhar, bloqueia as demais
- `[CHECKPOINT]` — é possível pausar neste ponto e validar o resultado
- `[BREAKING]` — quebra o código existente, exige atenção
- `[PARALLEL]` — pode ser feita ao mesmo tempo que outra tarefa

```markdown
### T3: UserService [RISK] [CHECKPOINT]
Esta tarefa forma a base de todo o backend.
É preciso parar aqui e fazer uma review, e só depois passar para a camada de API.
```

---

### Etapa 5: Escrever o Plano

Escreva no arquivo `implementation_plan.md`:

```markdown
# Implementation Plan: <nome do projeto>
**Data:** <YYYY-MM-DD>  
**Duração Estimada:** <X dias/horas>
**Origem:** [Scope Doc](.agent/SCOPE-<slug>.md)

## Resumo
<2-3 frases: o que será feito, como será feito>

## Decisões de Tech Stack
| Área | Escolha | Justificativa |
|------|-------|---------|
| ORM | Prisma | Type-safe, bom gerenciamento de migration |
| Auth | JWT + refresh token | Stateless, escalável |

## Tarefas

### 🔵 Fase 1: Fundação (não roda em paralelo)
- [ ] **T1:** Database migration `[CHECKPOINT]`
- [ ] **T2:** Camada de repository

### 🟡 Fase 2: Núcleo (depois que T1 e T2 terminarem)
- [ ] **T3:** Camada de service `[RISK][CHECKPOINT]`
- [ ] **T4:** Endpoint de API `[BREAKING]`

### 🟢 Fase 3: UI (quando T4 ou o mock estiver pronto)
- [ ] **T5:** Componente de frontend `[PARALLEL]`

## Lista de Riscos
| Risco | Probabilidade | Impacto | Mitigação |
|------|----------|------|-------|
| Migration não pode ser revertida | Baixa | Alto | Escrever e testar a down migration |

## Pontos em Aberto
- [ ] Fluxo de verificação de email: é necessário? (decisão pendente)

## Próximo Passo
1. Rode a skill `architecture-review` — o código não começa antes da aprovação do plano
2. Depois da aprovação, `executing-plans` ou `dispatching-parallel-agents`
```

---

## Regras

- **Toda tarefa termina com um critério de teste.** "Concluída" = testes passando.
- **Nenhuma tarefa contém "e também".** Se tiver, divida.
- **"Depois a gente resolve" não entra no plano.** Ou vira uma tarefa ou sai do escopo.
- **Se o plano mudar, ele é atualizado.** O plano antigo perde a validade — não apague, dê a data.
- **`architecture-review` não pode ser pulado.** É obrigatório, principalmente se houver tarefas marcadas com `[RISK]`.
