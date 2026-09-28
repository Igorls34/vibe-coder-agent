<!--
  DESCRIÇÃO
  ───────────────
  Regras que evitam danos ao sistema.
  A obrigatoriedade de aprovação para comandos irreversíveis e
  o princípio de não mexer em sistemas funcionando estão definidos neste arquivo.
  Essas regras são aplicadas em todas as tarefas, sem exceção.
-->

---
name: safety
description: >
  Impede danos irreversíveis em sistemas e dados.
  Comandos destrutivos exigem aprovação explícita do usuário.
  Sistemas em funcionamento não devem ser alterados, a menos que a tarefa
  exija isso diretamente.
---

# Regras de Segurança

## 1. Aprovação de Comandos Destrutivos

Comandos que contenham qualquer um dos patterns abaixo são **exibidos ao usuário antes de serem executados** e aguardam aprovação:

```
DROP TABLE      DROP DATABASE     DROP INDEX
DELETE FROM     TRUNCATE
rm -rf          rmdir /s
git push --force (na branch principal)
kubectl delete
docker system prune
```

**Formato de exibição:**
```
⚠️  Comando destrutivo detectado — aprovação necessária:

  rm -rf ./dist

Esta operação não pode ser desfeita. Devo continuar? (sim/não)
```

O comando não é executado até que o usuário responda "sim". Não existe exceção para "mudança pequena" ou "temporário".

---

## 2. Não Mexer em Sistemas em Funcionamento

Se um sistema está funcionando no momento (testes passando, ativo em production, em uso pelo usuário):

- Se a tarefa não exige **mudar esse sistema diretamente** → **não mexa**
- Se, ao adicionar uma nova funcionalidade, você precisar quebrar o código que está funcionando → avise o usuário antes e proponha uma alternativa

**Regra prática:**
```
Ao adicionar algo novo, se os testes existentes quebrarem → pare e avise.
Em vez de mudar em silêncio, diga: "Para corrigir isso, preciso fazer o seguinte".
```

Ao adicionar uma nova funcionalidade, siga esta ordem:
1. Execute os testes existentes — estão verdes? → comece
2. Execute os testes novamente após a alteração
3. Se algum teste quebrar → reporte isso primeiro e só então corrija

---

## 3. Obrigatoriedade do Plano de Rollback

Antes das operações abaixo, a etapa de rollback é definida:

| Operação | Rollback |
|-------|---------|
| Database migration | Down migration escrita e testada |
| Production deploy | O tag da versão anterior existe, é possível reverter o deploy |
| Alteração de config | Valor antigo guardado como comentário ou no `.env.example` |
| Refactor grande | Feito em uma feature branch, sem mexer na main |

Nenhuma operação destrutiva começa antes de o rollback estar definido.
