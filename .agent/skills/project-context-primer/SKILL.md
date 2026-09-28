---
name: project-context-primer
description: >
  Execute esta skill no início de qualquer nova conversa ou sessão de agente,
  antes de escrever uma única linha de código. Ela carrega as decisões
  arquiteturais, as convenções, as armadilhas conhecidas e o status atual da
  tarefa do projeto, para que o agente opere com contexto completo — e não do
  zero.
---

<!--
  DESCRIÇÃO
  ───────────────
  Esta skill é usada para carregar rapidamente o contexto do projeto quando uma
  nova conversa ou sessão de agente começa. Decisões arquiteturais, regras,
  armadilhas conhecidas e o status atual da tarefa são carregados para que o
  agente trabalhe com contexto completo, em vez do modo "página em branco".
  Deve ser executada OBRIGATORIAMENTE antes de escrever código — é a prevenção
  mais barata contra retrabalho.

  QUANDO: Como primeiro passo de toda nova conversa. NUNCA deve ser pulada.
  SAÍDA:   Bloco de resumo "Contexto Carregado" — nome do projeto, stack, tarefa
           atual, informações carregadas.
-->

# Skill de Contexto do Projeto

## Quando Disparar

**Sempre executar primeiro** quando:
- Iniciar uma nova conversa sobre este projeto
- Um subagente for criado para trabalhar em um módulo
- Retomar o trabalho após um intervalo de mais de um dia
- Alguém disser "você pode me ajudar com este projeto?" sem nenhum contexto

## Processo Passo a Passo

### 1. Carregar o Índice de Conhecimento

Leia `.agent/knowledge/INDEX.md` primeiro.

O arquivo INDEX.md contém linhas neste formato:
```
| Data | Tipo | Tópico (linkado) | Resumo em uma linha |
```

- Examine todas as linhas em busca de entradas relevantes para a tarefa de hoje
- Para cada linha relevante, siga o link e leia o arquivo completo `.agent/knowledge/<slug>.md`
- Preste atenção especial às entradas dos tipos `decision` e `gotcha` — elas evitam os erros mais caros
- Extraia o slug do link: `[topic](./auth/token-refresh.md)` → o slug é `auth/token-refresh`

**Se o INDEX.md não existir**, emita exatamente este aviso e prossiga:
```
⚠️ Base de conhecimento não inicializada (.agent/knowledge/INDEX.md ausente).
Isso significa que nenhuma decisão arquitetural, convenção ou armadilha foi registrada ainda.
Ação: use a skill `knowledge-base-update` ao final desta sessão para começar a construí-la.
```

### 2. Carregar o README do Projeto

Leia `README.md` (ou `docs/README.md`, se existir).

Extraia e internalize:
- O que o projeto faz (uma frase)
- Tech stack (linguagem, framework, principais bibliotecas)
- Como executá-lo localmente
- Como rodar os testes

### 3. Carregar o Contexto da Tarefa Ativa

Verifique estes arquivos nesta ordem e leia os que existirem:

```
.agent/CURRENT_TASK.md       ← O que está sendo trabalhado agora
.agent/OPEN_QUESTIONS.md     ← Decisões ainda não tomadas
```

> **Observação:** decisões arquiteturais e decisões passadas são armazenadas em `.agent/knowledge/` (via a skill `knowledge-base-update`),
> e não em um DECISIONS_LOG separado. O INDEX.md (Etapa 1) é o seu histórico de decisões.

Se `.agent/CURRENT_TASK.md` existir, trate-o como a fonte da verdade para a sua missão atual.
Se não existir, pergunte ao usuário qual é a tarefa de hoje antes de prosseguir.

### 4. Mapear a Estrutura do Projeto

Execute uma listagem de diretórios rasa (2 primeiros níveis) para entender a organização:

```
src/
  components/
  services/
  utils/
docs/
tests/
.agent/
```

Anote quaisquer pastas fora do padrão e verifique `.agent/knowledge/` em busca de explicações.

### 5. Verificar as Convenções Ativas

Leia `.agent/rules/`, se existir. Estas são restrições rígidas — não são sugestões.

Arquivos de regras comuns a procurar:
- `code-style.md`
- `commit-messages.md`
- `test-coverage.md`
- `no-secrets.md`

### 6. Produzir um Resumo de Contexto

Antes de executar qualquer trabalho, emita um breve resumo neste formato:

```
## Contexto Carregado

**Projeto:** <nome> — <descrição em uma linha>
**Stack:** <linguagens, frameworks>
**Tarefa Atual:** <o que vamos fazer hoje, de CURRENT_TASK.md ou do usuário>
**Entradas de Conhecimento Carregadas:** <lista de slugs>
**Regras Ativas:** <lista ou "nenhuma encontrada">
**Perguntas Abertas:** <quaisquer bloqueios ou pontos não decididos>
**Pronto para prosseguir:** Sim / Não (explique se Não)
```

Só depois de produzir este resumo você deve iniciar a tarefa de fato.

## Regras

- **Nunca pule esta skill** em uma nova sessão. Uma varredura de 2 minutos economiza horas de retrabalho.
- **Não assuma.** Se o README contradisser uma entrada de conhecimento, sinalize isso — não escolha uma em silêncio.
- **Atualize CURRENT_TASK.md** ao final de cada sessão com o que foi feito e o que vem a seguir.
- **Se faltar contexto**, pergunte ao usuário o mínimo de perguntas necessárias para preencher as lacunas antes de começar.

## Mantendo o CURRENT_TASK.md

Ao final de toda sessão, atualize `.agent/CURRENT_TASK.md`:

```markdown
# Tarefa Atual

## O Que Estamos Construindo
<descrição>

## Status
<Em Andamento | Bloqueado | Requer Revisão | Pronto>

## Resumo da Última Sessão
<data> — <o que foi feito>

## Próximos Passos
1. <próxima ação>
2. <próxima ação>

## Bloqueios
- <qualquer coisa que esteja impedindo o avanço>
```
