---
name: knowledge-base-update
description: >
  Use esta skill sempre que aprender algo importante durante uma conversa: uma
  decisão arquitetural chave, uma convenção específica do projeto, a causa raiz
  de um bug, uma peculiaridade de uma API de terceiros, ou qualquer fato que
  ajude um agente futuro a não refazer a mesma pesquisa. Escreve entradas
  estruturadas na base de conhecimento do projeto para que o contexto persista
  entre conversas e agentes.
---

<!--
  DESCRIÇÃO
  ───────────────
  Esta skill é usada para que as informações importantes aprendidas durante uma
  conversa não se percam. Decisões de arquitetura, regras do projeto, causas raiz
  de bugs, armadilhas de serviços de terceiros e qualquer coisa que você não
  quer que precise ser pesquisada novamente são escritas de forma estruturada no
  diretório .agent/knowledge/. Assim, os agentes futuros não precisam pesquisar a
  mesma pergunta do zero.

  QUANDO: Quando uma decisão importante é tomada, quando um bug surpreendente
          é resolvido, quando o comportamento de uma API é documentado, quando
          uma nova regra de projeto é criada.
  SAÍDA:   .agent/knowledge/<slug>.md + uma linha no INDEX.md
-->

# Skill de Atualização da Base de Conhecimento

## Quando é Disparado

Ative esta skill quando ocorrer qualquer um dos casos abaixo:

- Uma decisão arquitetural ou de stack de tecnologia é tomada (e o *porquê* importa)
- Um bug não óbvio é corrigido (causa raiz + correção documentadas)
- Uma peculiaridade, limite ou armadilha de um serviço/API de terceiros é descoberta
- Uma convenção de projeto é estabelecida (nomenclatura, estrutura de pastas, padrões)
- Uma pergunta de pesquisa é respondida após uma investigação significativa
- Acontece um momento do tipo "tentamos X e falhou por causa de Y"

## Processo Passo a Passo

### 1. Identifique o Tipo da Entrada

Escolha a categoria que melhor descreve o que foi aprendido:

| Tipo | Quando Usar |
|------|-------------|
| `decision` | Arquitetura, escolhas de tecnologia, trade-offs |
| `convention` | Regras de nomenclatura, padrões de código, estruturas de pastas |
| `bug` | Causa raiz + correção de um problema não óbvio |
| `gotcha` | Peculiaridade de terceiros, problema de ambiente, armadilha de edge case |
| `research` | Resposta a uma pergunta que exigiu investigação significativa |

### 2. Defina o Caminho do Arquivo

Escreva em: `.agent/knowledge/<slug>.md`

- `slug` = resumo do tópico em kebab-case (ex.: `auth-token-refresh-strategy`)
- Se já existir uma entrada para este tópico, **atualize-a** em vez de criar um arquivo novo
- Agrupe tópicos relacionados em subpastas: `.agent/knowledge/auth/`, `.agent/knowledge/infra/`, etc.

### 3. Escreva a Entrada

Use exatamente este modelo:

```markdown
---
type: <decision|convention|bug|gotcha|research>
topic: <título curto legível por humanos>
date: <YYYY-MM-DD>
tags: [<tag1>, <tag2>]
---

## Resumo
<Uma ou duas frases — o que foi aprendido, a conclusão primeiro>

## Contexto
<Por que isso apareceu. Que problema estávamos resolvendo?>

## Decisão / Descoberta
<O conteúdo real. Seja específico. Inclua trechos de código, se ajudar.>

## Justificativa
<Por que esta escolha em vez das alternativas. Quais alternativas foram rejeitadas e por quê.>

## Consequências
(O que isso afeta daqui para frente? No que é preciso ficar atento?>

## Referências
- <Caminho de arquivo, link de PR, trecho de conversa ou URL externa>
```

### 4. Atualize o Índice

Após escrever a entrada, acrescente um resumo de uma linha em `.agent/knowledge/INDEX.md`:

```
| <date> | <type> | [<topic>](./<path>.md) | <resumo de uma linha> |
```

Se o `INDEX.md` ainda não existir, crie-o com este cabeçalho primeiro:

```markdown
# Base de Conhecimento do Projeto

| Data | Tipo | Tópico | Resumo |
|------|-----|--------|--------|
```

### 5. Confirme ao Usuário

Após escrever, relate:
- O que foi salvo e onde
- O resumo de uma linha que foi adicionado ao índice

## Regras

- **Nunca resuma de forma vaga.** "Decidimos usar X" não serve de nada sem o *porquê*.
- **Inclua o anti-pattern.** Se algo foi testado e rejeitado, documente — costuma ser o conhecimento mais valioso.
- **Mantenha as entradas atômicas.** Um tópico por arquivo. Não agrupe decisões sem relação.
- **A data é obrigatória.** Agentes futuros precisam saber quão recente é esse conhecimento.
- **Na dúvida, escreva.** O custo de uma entrada redundante é baixo. O custo de uma ausente é refazer o trabalho.
