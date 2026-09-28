<!--
  DESCRIÇÃO
  ───────────────
  Esta skill refina uma ideia ou um problema com o método socrático. O agente
  não responde a você — ele faz perguntas. Por meio dessas perguntas a ideia
  fica clara, as premissas são questionadas, o escopo é definido e a
  necessidade real vem à tona. Ela transforma "o que vamos fazer?" em "o que
  exatamente precisamos fazer e por quê?".

  QUANDO: Quando surge uma nova funcionalidade ou ideia de projeto, quando os
          requisitos estão indefinidos ou quando existem entendimentos
          diferentes dentro do time.
  SAÍDA:   Scope doc — problema esclarecido, escopo, restrições, critérios de
           sucesso.
-->

---
name: brainstorming
description: >
  Skill de refino de design socrático. Em vez de pular direto para a solução,
  esta skill faz perguntas direcionadas para esclarecer o problema, desafiar
  premissas, definir o escopo e revelar a necessidade real. Transforma "o que
  devemos construir?" em uma declaração de problema clara e acordada, com
  critérios de sucesso.
---

# Skill de Brainstorming — Refino de Design Socrático

## Filosofia

Boas soluções vêm de boas perguntas. Esta skill não produz uma resposta — ela oferece uma estrutura para fazer as perguntas certas. 80% de uma ideia está incompleta no momento em que é expressa. Essa lacuna precisa ser encontrada enquanto se pensa, não enquanto se escreve código.

> "Se eu tivesse uma hora para resolver um problema, gastaria 55 minutos pensando no problema e 5 minutos pensando em soluções." — Einstein

---

## Processo

### Fase 1: Ouvir o Problema

O usuário ou o time expressa a ideia. **Não proponha soluções imediatamente.** Antes, faça estas perguntas:

#### 🎯 Clareza do Problema
- Por que estamos construindo esta funcionalidade/sistema? Qual dor ela resolve?
- O que acontece sem isso? Como as pessoas atendem a essa necessidade hoje?
- Para quem estamos construindo? Como é a rotina diária dessas pessoas hoje?

#### 📏 Limites de Escopo
- O que com certeza está **dentro** do escopo deste projeto?
- O que com certeza está **fora** do escopo?
- O que dizemos "adicionamos depois"? (defina isso agora; a discussão fica para depois)

#### ⚡ Restrições
- Existe restrição de tempo? Há um deadline fixo?
- Existe restrição técnica? (stack atual, orçamento, capacidade do time)
- Existe restrição regulatória ou legal?

#### ✅ Critérios de Sucesso
- Como saberemos que este projeto foi bem-sucedido?
- Qual métrica ou observação vai nos fazer dizer "terminou"?
- O que vai nos dizer, 6 meses depois, se esta decisão estava certa ou errada?

---

### Fase 2: Forçar as Premissas

Toda ideia tem premissas não explicitadas por baixo. Traga-as para a superfície:

```
"Os usuários vão querer usar esta funcionalidade"
→ Como sabemos disso? Já testamos?

"Esta mudança no backend vai ser simples"
→ Quem disse isso? Alguém que conhece o código atual fez essa avaliação?

"Primeiro X, depois Y"
→ Por que nesta ordem? X funciona sem Y?

"Usuários mobile não são importantes"
→ Já olhamos as analytics atuais? Isso é uma decisão ou uma premissa?
```

Classifique cada premissa assim:
- **Verificada:** há evidência, prossiga com confiança
- **Razoável:** faz sentido, mas não foi testada — anote o risco
- **Perigosa:** sem evidência e com alto impacto — valide antes

---

### Fase 3: Avaliar as Alternativas

Produza pelo menos 3 abordagens e compare-as:

| Abordagem | Prós | Contras | Risco | Esforço |
|----------|---------|---------|------|------|
| Opção A | ... | ... | Baixo | 2 dias |
| Opção B | ... | ... | Médio | 1 semana |
| Opção C | ... | ... | Alto | 3 semanas |

**Sempre inclua a opção "não fazer nada" na tabela.** Qual é o custo dessa opção?

---

### Fase 4: Encontrar o Slice de Mínimo Valor

Em vez de construir o projeto inteiro de uma vez:
- Qual é a parte de 20% que entrega 80% do valor?
- Qual é o primeiro slice que entrega valor ao usuário mais rápido?
- O que com certeza não deve existir na primeira iteração?

```
MVP = [funcionalidade A, funcionalidade B]
Iteração 2 = [funcionalidade C, funcionalidade D]
Futuro = [funcionalidade E] — se necessário
```

---

### Fase 5: Escrever o Scope Doc

Depois que todas as perguntas forem respondidas, escreva no arquivo `.agent/SCOPE-<slug>.md`:

```markdown
# Escopo: <nome do projeto>
**Data:** <YYYY-MM-DD>
**Responsáveis pela Decisão:** <nomes>

## Problema
<Um parágrafo — qual dor estamos resolvendo, para quem>

## Dentro do Escopo
- <item>

## Fora do Escopo
- <item>

## Critérios de Sucesso
1. <critério mensurável>
2. <critério mensurável>

## Premissas Não Verificadas
- <premissa> → <como será verificada>

## Abordagem Escolhida
<A Opção X foi escolhida porque...>

## Primeiro Slice (MVP)
<O que será feito, o que não será feito>

## Próximo Passo
→ Use a skill `writing-plans` para criar o plano de implementação
```

---

## Regras

- **Pular para a solução é proibido.** O `writing-plans` não começa antes de o scope doc estar completo.
- **É proibido dizer "na minha visão, vamos fazer assim".** Esta skill faz perguntas, não dá respostas.
- **Registre todas as premissas.** Premissa não verificada = risco oculto.
- **O scope doc deve ser curto.** 1 página basta. Se estiver mais longo, o escopo está grande demais.
- **"Adicionamos depois" não entra na primeira iteração.** Se entrou, não há argumento para cortar.
