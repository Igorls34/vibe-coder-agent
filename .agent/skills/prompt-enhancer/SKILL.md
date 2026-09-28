<!--
  DESCRIÇÃO
  ───────────────
  Esta skill recebe um pedido do usuário vago ou incompleto e o transforma em
  um prompt mais claro, mais específico e mais aplicável. Evita que o agente
  interprete errado e faça o trabalho de forma equivocada. Um pedido vago como
  "conserte isso" é convertido no formato "em qual condição o que é esperado,
  qual é o critério de sucesso".

  QUANDO: Quando o pedido é vago, o escopo não está definido ou há mais de uma
          interpretação possível. Executada antes das demais skills — melhora a
          qualidade da entrada.
  SAÍDA:   Texto do prompt aprimorado — segue direto para brainstorming ou
           writing-plans.
-->

---
name: prompt-enhancer
description: >
  Recebe um pedido do usuário vago ou incompleto e o transforma em um prompt
  preciso e acionável. Identifica as dimensões ausentes (o quê, por quê, quem,
  critérios de sucesso, restrições) e faz perguntas direcionadas ou preenche as
  lacunas com premissas explícitas. Executada antes das demais skills para
  melhorar a qualidade da entrada.
---

# Skill de Aprimoramento de Prompts

## Filosofia

Responder certo para a pergunta errada é pior do que responder errado para a pergunta certa.

A qualidade de um pedido determina a qualidade da saída. Esta skill impede que o agente trabalhe no modo "provavelmente era isso que ele quis". Ela traz a ambiguidade para a superfície e esclarece os pontos de decisão.

---

## Quando é Disparado

Esta skill entra em ação se houver qualquer um destes sinais:

- O pedido tem uma única frase e nenhum detalhe: *"Corrija o login"*, *"Aumente a performance"*, *"Quero que fique melhor"*
- O pedido pode ser interpretado de várias formas: *"Melhore o fluxo do usuário"*
- Não há público-alvo nem contexto: *"Adicione um dashboard"*
- O critério de sucesso é vago: *"Acelere"*, *"Limpe"*, *"Otimize"*
- Não há limite de escopo: *"Reformate o sistema de auth"*

---

## Processo

### Etapa 1: Identificar o Tipo de Pedido

Primeiro, determine que tipo de pedido é:

| Tipo | Exemplo | Dimensões Ausentes |
|-----|-------|----------------|
| **Correção de bug** | "O login não funciona" | Em qual condição? Qual mensagem de erro? Sempre? |
| **Nova funcionalidade** | "Adicionar página de perfil" | Quem vai usar? Qual conteúdo? Referência de design? |
| **Refatoração** | "Limpe o código" | Qual arquivo/módulo? O que significa "limpo"? |
| **Performance** | "Acelere" | Qual página/endpoint? Tempo alvo? Tempo atual? |
| **Design** | "Deixe mais bonito" | Existe alguma referência? Qual elemento? Em qual contexto? |
| **Infraestrutura** | "Automatizar o deploy" | Qual ambiente? Qual pipeline atual? Qual é o gatilho? |

---

### Etapa 2: Encontrar as Dimensões Ausentes

Para cada pedido, verifique estas 6 dimensões. Marque as que estiverem ausentes:

```
[ ] WHAT   — O que exatamente vai mudar/ser adicionado?
[ ] WHY    — Por que é necessário? Qual problema resolve?
[ ] WHO    — Quem vai usar? Qual usuário/sistema?
[ ] WHEN   — Existe deadline? Em qual ordem?
[ ] HOW    — Existe alguma restrição? (stack, plataforma, orçamento, API existente)
[ ] DONE   — Qual é o critério de sucesso? Quando será dito "terminou"?
```

---

### Etapa 3: Escolher a Estratégia de Aprofundamento

**Estratégia A — Perguntar** (se houver 1-2 ambiguidades críticas):

Faça apenas as perguntas mais importantes. Para cada pergunta, indique por que ela está sendo feita:

```
Tenho 2 perguntas — a resposta muda a abordagem que vou escolher:

1. Ao dizer "o login não funciona" — em qual condição?
   a) Sempre?
   b) Em algum navegador específico?
   c) Para um usuário específico?
   (Isso define o escopo do bug)

2. Existe alguma mensagem de erro ou log atual?
   (Se você compartilhar, a análise de causa raiz fica muito mais rápida)
```

**Estratégia B — Assumir e Mostrar** (se houver 3+ ambiguidades, com prioridade em avançar rápido):

Faça premissas razoáveis, liste-as claramente e produza o prompt aprimorado:

```
Pedido: "Adicionar página de perfil"

Premissas (estou aguardando sua aprovação):
- Usuário: usuários registrados que já fizeram login
- Conteúdo: nome, email, avatar, bio
- Design: alinhado ao design system existente
- Auth: rota protegida com JWT token
- Escopo: frontend + endpoint de backend

Há algo diferente nessas premissas? Se não, vamos começar com este escopo.
```

---

### Etapa 4: Produzir o Prompt Aprimorado

Em ambas as estratégias, o formato da saída final é o seguinte:

```markdown
## Prompt Aprimorado

**Pedido Original:** <pedido original do usuário>

**Tarefa Esclarecida:**
<explicação específica, com interpretação única>

**Dentro do Escopo:**
- <item>
- <item>

**Fora do Escopo:**
- <item> (justificativa para deixar de fora)

**Critérios de Sucesso:**
1. <critério mensurável>
2. <critério mensurável>

**Restrições Técnicas:**
- <stack, plataforma, dependência>

**Perguntas que Restam Abertas:**
- <se houver — serão respondidas na fase seguinte>

**Próxima Skill Sugerida:**
→ <brainstorming | writing-plans | test-driven-execution>
```

---

### Etapa 5: Obter Aprovação do Usuário

Mostre o prompt aprimorado ao usuário:

```
Entendi assim — está correto?

[Conteúdo do Prompt Aprimorado]

Se você aprovar, posso começar. Há algo que queira mudar?
```

Não avance para a próxima skill sem a aprovação.

---

## Exemplos

### Exemplo 1: Correção de Bug

**Usuário:** *"A redefinição de senha não funciona"*

**Diagnóstico:**
- WHAT: redefinição de senha — mas em qual etapa?
- HOW: sem mensagem de erro
- WHO: todos os usuários ou apenas um específico?

**Estratégia A — Perguntar:**
```
Em qual etapa do fluxo de "redefinição de senha" está o problema?
  a) O email não está sendo enviado?
  b) O link gera erro ao ser clicado?
  c) A nova senha não está sendo salva?
```

---

### Exemplo 2: Performance

**Usuário:** *"A página está muito lenta"*

**Diagnóstico:** 5 das 6 dimensões estão ausentes.

**Estratégia B — Assumir e Mostrar:**
```
Vou seguir com estas premissas:
- Qual página: /dashboard (a mais usada)
- Situação atual: 4-5 segundos de TTI (estimativa)
- Alvo: menos de 2 segundos
- Plataforma: web (não mobile)
- Restrição: o stack atual não muda

Se estiver diferente, me avise; caso contrário, começo capturando o perfil do Lighthouse.
```

---

## Regras

- **Sempre obtenha aprovação.** O prompt aprimorado não é aplicado sem a aprovação do usuário.
- **Faça no máximo 3 perguntas.** Mais do que isso cansa o usuário — escolha as 3 mais críticas.
- **Não esconda as premissas.** Cada premissa é listada explicitamente.
- **Escreva também o que está fora do escopo.** "Não estou fazendo isso porque" = a maior clareza possível.
- **Sugira a próxima skill.** Informe qual skill vai rodar depois do prompt aprimorado.
