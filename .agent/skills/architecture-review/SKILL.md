---
name: architecture-review
description: >
  Antes de se comprometer com um plano de implementação, rode esta skill para
  submeter a arquitetura proposta a um teste de estresse. Identifica
  over-engineering, dependências circulares, modos de falha que faltam,
  lacunas de segurança e gargalos de escalabilidade — antes de escrever
  qualquer linha de código. Atua como um "segundo olhar" sobre o plano.
---

<!--
  DESCRIÇÃO
  ───────────────
  Esta skill submete um plano de implementação a um teste de estresse como
  "segundo olhar" antes de o código ser escrito. Detecta cedo o
  over-engineering, as dependências circulares, os cenários de erro que
  faltam, as falhas de segurança e os problemas de escalabilidade.
  Construir sobre uma base errada tem um custo muito alto — esta skill leva
  esse custo a zero.

  QUANDO: DEPOIS de o plano estar pronto e ANTES de começar a escrever código.
          É obrigatória.
  SAÍDA:   Lista de 🔴 Bloqueador / 🟡 Aviso / 🔵 Nota + a decisão
          Aprovado/Revisões Necessárias.
-->

# Skill de Revisão de Arquitetura

## Quando Disparar

- Depois que `writing-plans` produzir um plano de implementação, *antes* de `executing-plans`
- Quando uma refatoração relevante é proposta
- Ao adicionar um novo serviço externo ou integração
- Quando o time está debatendo entre duas abordagens
- Quando algo "parece errado" em um design, mas você não consegue explicar por quê

## Processo Passo a Passo

### 1. Carregar o Plano

Leia o plano de implementação (normalmente `implementation_plan.md` ou o artefato de `writing-plans`).

Extraia:
- Componentes que estão sendo construídos
- Fluxo de dados entre os componentes
- Dependências externas
- Premissas que o plano faz

### 2. Percorrer os Checklists

Percorra cada seção de checklist. Para cada resposta "Não" ou "Não claro", sinalize como um achado.

---

#### A. Verificação de Complexidade
- [ ] Esta é a solução mais simples que poderia funcionar?
- [ ] Existem soluções prontas (bibliotecas, serviços) que cobrem 80%+ disso?
- [ ] Cada componente tem uma responsabilidade única e clara?
- [ ] Um desenvolvedor júnior entenderia isso em 30 minutos?

#### B. Acoplamento e Coesão
- [ ] Cada módulo pode ser testado isoladamente?
- [ ] Existem dependências circulares? (A precisa de B, B precisa de A)
- [ ] A propriedade dos dados está clara? (apenas um lugar escreve cada parte do estado)
- [ ] A superfície da API pública é mínima? (não exponha o que não precisa ser exposto)

#### C. Dados e Estado
- [ ] A fonte da verdade de cada dado está claramente definida?
- [ ] O que acontece se o datastore cair?
- [ ] Existe algum estado global mutável? Se sim, isso se justifica?
- [ ] Há migrations de banco planejadas para qualquer mudança de schema?

#### D. Modos de Falha
- [ ] O que acontece quando cada serviço externo está indisponível?
- [ ] Existem mecanismos de retry com backoff para as chamadas de rede?
- [ ] Há timeouts em todas as operações de I/O?
- [ ] Existe um circuit breaker caso uma dependência a jusante se degrade?
- [ ] Como uma falha parcial aparece para o usuário?

#### E. Segurança
- [ ] Toda entrada do usuário é validada e sanitizada?
- [ ] Os segredos são gerenciados via env vars / secrets manager (não hardcoded)?
- [ ] Autenticação e autorização estão claramente separadas?
- [ ] Há algum vetor de SQL injection / XSS / CSRF?
- [ ] O princípio do menor privilégio se aplica às contas de serviço?

#### F. Escalabilidade
- [ ] Qual é a carga esperada em 6 meses? O design dá conta dela?
- [ ] Há algum problema óbvio de query N+1?
- [ ] Está previsto algum cache? A estratégia de invalidação está definida?
- [ ] Jobs em background são usados para tudo que possa bloquear o ciclo da requisição?

#### G. Observabilidade
- [ ] Está previsto logging estruturado?
- [ ] Há métricas/health endpoints?
- [ ] Os erros aparecerão em uma ferramenta de monitoramento (e não só em logs)?
- [ ] Há uma forma de rastrear uma requisição de ponta a ponta?

#### H. Operabilidade
- [ ] Isso pode ser implantado sem downtime?
- [ ] Existe um plano de rollback caso o deploy saia errado?
- [ ] As diferenças entre ambientes (dev/staging/prod) estão documentadas?

---

### 3. Classificar os Achados

Para cada "Não" ou "Não claro" encontrado acima, classifique:

| Classe | Significado | Ação |
|-------|---------|--------|
| 🔴 Bloqueador | Precisa ser corrigido antes da implementação | Revise o plano |
| 🟡 Aviso | Deve ser tratado; é possível seguir com cuidado | Adicione uma tarefa ao backlog |
| 🔵 Nota | Seria bom ter; risco baixo | Documente e siga em frente |

### 4. Produzir o Relatório de Revisão

Sempre escreva em um arquivo separado — **não suja o arquivo do plano**:
```
.agent/reviews/architecture-review-<plan-slug>.md
```

Se alguma decisão arquitetural importante saiu desta revisão, use também a skill `knowledge-base-update` para adicionar o item à base de conhecimento com o tipo `decision` — o `project-context-primer` vai lê-lo de lá.

```markdown
# Architecture Review — <nome do plano>
**Data:** <YYYY-MM-DD>
**Revisor:** Agent

## Veredito
<Aprovado | Aprovado com condições | Revisões necessárias>

## 🔴 Bloqueadores
### 1. <título do achado>
**Item do checklist:** Modos de Falha — timeout em I/O
**Problema:** A chamada ao serviço de email não tem timeout. Se o SendGrid travar, a thread inteira da requisição fica bloqueada.
**Recomendação:** Envolva em um timeout de 5 segundos; use uma fila de jobs em background para emails não críticos.

## 🟡 Avisos
### 1. <título do achado>
...

## 🔵 Notas
### 1. <título do achado>
...

## Resumo
<Saúde geral da arquitetura em 2-3 frases.>
```

### 5. Gate: Prosseguir ou Revisar?

- Se houver algum 🔴 **Bloqueador**: pare. Compartilhe o relatório com o usuário e revise o plano antes de escrever qualquer código.
- Se houver apenas 🟡/**🔵**: prossiga, mas garanta que os avisos sejam rastreados como tarefas.

## Regras

- **Seja específico, não genérico.** "Você deveria adicionar tratamento de erro" não serve para nada. "A chamada `fetchUser()` na linha 42 do plano não tem tratamento para 404" é uma revisão de verdade.
- **Não bloqueie por preferências de estilo.** Bloqueadores são riscos objetivos, não questão de gosto.
- **Nunca aprove um plano com um modo de falha indefinido em um caminho crítico.**
- **Se você não conseguir entender uma parte do plano, isso por si só já é um achado** — clareza é um requisito.
