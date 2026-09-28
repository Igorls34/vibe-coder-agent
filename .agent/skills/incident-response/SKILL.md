---
name: incident-response
description: >
  Quando algo quebra em produção: faça a triagem da severidade, reúna evidências,
  identifique a causa raiz, faça o deploy de uma correção ou mitigação e escreva
  um post-mortem. Oferece um processo calmo e estruturado para momentos de
  alta pressão.
---

<!--
  DESCRIÇÃO
  ───────────────
  Esta skill oferece uma gestão de crise calma e estruturada para quando algo
  quebra em produção. Classifique a severidade → reúna evidências → encontre a
  causa raiz → corrija → escreva o post-mortem. Comandos executados no panico
  ampliam o problema; esta skill aplica o princípio "pense primeiro, aja depois".

  QUANDO: Quando a produção cai, em um pico de erros, na suspeita de violação de
          segurança, em regressões inesperadas após um deploy.
  SAÍDA:   .agent/incidents/INC-<data>-<slug>.md — timeline + causa raiz + itens de ação.
-->

# Skill de Resposta a Incidentes

## Quando é Disparado

- A produção está fora do ar ou severamente degradada
- A taxa de erros dá um pico significativamente acima do baseline
- Um cliente relata um bug crítico em produção
- Há suspeita de violação de segurança ou vazamento de dados
- Um deploy causou uma regressão inesperada

## Processo Passo a Passo

### 1. Declare e Classifique o Incidente

**Primeiro: respire. Depois classifique.**

| Severidade | Definição | Tempo de Resposta |
|------------|-----------|-------------------|
| P1 — Crítica | Indisponibilidade total, perda de dados, violação de segurança | Imediato |
| P2 — Alta | Funcionalidade principal quebrada, muitos usuários afetados | < 30 min |
| P3 — Média | Degradação parcial, existe uma solução alternativa (workaround) | < 2 horas |
| P4 — Baixa | Bug menor, questão cosmética | Próximo sprint |

Abra `.agent/incidents/INC-<YYYY-MM-DD>-<slug>.md` e escreva:

```markdown
# Incidente: <título curto>
**Data:** <YYYY-MM-DD HH:MM UTC>
**Severidade:** P<N>
**Status:** Investigando
**Incident Commander:** <nome>
```

### 2. Triagem — Reúna Evidências Rápido

Não adivinhe. Colete os dados primeiro.

Checklist:
- [ ] Qual é o sintoma visível para o usuário?
- [ ] Quando começou? (verifique o monitoramento, não apenas "alguém relatou")
- [ ] O que mudou recentemente? (último deploy, mudança de config, atualização de dependência)
- [ ] O que o log de erro diz? (obtenha as mensagens de erro exatas + stack traces)
- [ ] Está afetando todos os usuários ou apenas um subconjunto?
- [ ] Existe um alerta automático, ou isso foi relatado por um usuário?

Comandos para executar imediatamente:

```bash
# Deploys recentes
git log --oneline -20

# Verificar processo em execução / saúde do container
# (ajuste para sua stack)
docker ps
systemctl status <service>

# Acompanhar logs ao vivo
docker logs -f <container> --tail 100
journalctl -u <service> -n 100 -f
```

Adicione as descobertas ao arquivo do incidente em `## Evidências`.

### 3. Contenha a Explosão

Antes de corrigir, limite os danos:

- **Se a causa foi um deploy ruim:** reverta imediatamente
  ```bash
  git revert HEAD
  # ou reimplante a versão anterior conhecida como boa
  ```
- **Se uma funcionalidade específica está falhando:** desligue-a via feature flag
- **Se os dados estão sendo corrompidos:** tire o serviço afetado do ar temporariamente
- **Se um segredo foi exposto:** rotacione-o imediatamente e depois investigue

Documente no arquivo do incidente todas as ações tomadas, com timestamps.

### 4. Análise de Causa Raiz

Uma vez contido, encontre a causa *real* — não apenas o sintoma.

Use a técnica dos **5 Porquês**:
```
Por que os usuários viram um erro 500?
→ Porque a consulta ao banco de dados deu timeout.
Por que a consulta deu timeout?
→ Porque faltava um novo índice após a migration.
Por que o índice estava faltando?
→ Porque o script de migration não incluía CREATE INDEX.
Por que o script de migration não incluía isso?
→ Porque não temos uma convenção de verificar query plans antes do deploy.
→ Causa raiz: processo de revisão de query plan pré-deploy ausente.
```

A causa raiz quase nunca é a falha técnica imediata — é a lacuna de processo que a permitiu.

### 5. Corrija e Faça o Deploy

Escreva a correção:
- A menor alteração possível que resolve o problema
- Adicione um teste que teria pegado isso
- Verifique a correção em staging antes da produção

Faça o deploy:
- Monitore as métricas durante o deploy
- Deixe alguém olhando os logs por 15 minutos após o deploy
- Confirme que o sintoma foi resolvido com um teste real de usuário

### 6. Escreva o Post-Mortem

Em até 24h após a resolução, escreva um post-mortem em `.agent/incidents/INC-<date>-<slug>.md`:

```markdown
# Post-Mortem: <título>

## Resumo
<2-3 frases: o que aconteceu, por quanto tempo, quantos usuários foram afetados>

## Timeline
| Hora (UTC) | Evento |
|------------|--------|
| HH:MM | O problema ocorreu pela primeira vez (inferido a partir dos logs) |
| HH:MM | Primeiro alerta / relato de usuário |
| HH:MM | Incidente declarado |
| HH:MM | Mitigação aplicada |
| HH:MM | Causa raiz identificada |
| HH:MM | Correção implantada |
| HH:MM | Incidente resolvido |

## Causa Raiz
<A causa raiz real — não apenas o sintoma>

## O Que Deu Certo
- <algo que ajudou — bom monitoramento, rollback rápido, etc.>

## O Que Deu Errado
- <algo que piorou as coisas — detecção lenta, runbook confuso, etc.>

## Action Items

| Ação | Responsável | Prazo |
|------|-------------|-------|
| Adicionar índice ausente à convenção de migration | <nome> | <data> |
| Adicionar teste de performance de consultas | <nome> | <data> |
| Criar alerta para latência p99 de queries no banco | <nome> | <data> |
```

### 7. Atualize a Base de Conhecimento

Após o post-mortem, use a skill `knowledge-base-update` para criar uma entrada do tipo `gotcha` sobre a causa raiz, para que agentes e desenvolvedores futuros saibam a respeito.

## Regras

- **Contenha primeiro, diagnostique depois.** Parar o sangramento é mais importante do que entender o ferimento.
- **Sem culpa nos post-mortems.** Foque nas lacunas de processo, não nas pessoas.
- **Todo incidente gera pelo menos um action item.** Se nada mudar, vai acontecer de novo.
- **Nunca declare um incidente resolvido antes que um usuário real confirme a correção.** Métricas podem mentir; usuários não.
- **Todos os incidentes P1/P2 exigem post-mortem.** Sem exceções.
