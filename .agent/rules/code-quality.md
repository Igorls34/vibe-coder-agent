<!--
  DESCRIÇÃO
  ───────────────
  Regras que mantêm o código reutilizável e compreensível.
  O princípio DRY é aplicado, mas sem over-engineering.
  Define o equilíbrio entre "bom o suficiente" e "abstração excessiva".
-->

---
name: code-quality
description: >
  Exige DRY sem over-engineering. Código duplicado é refatorado quando
  aparece 3+ vezes ou causaria bugs se não fosse atualizado de forma
  consistente. Abstrações devem justificar sua complexidade — sem
  generalização prematura.
---

# Regras de Qualidade de Código

## 1. Princípio DRY — Sem Exagerar

### Quando Refatorar?
Se o mesmo bloco de código se repete em **3 ou mais** lugares → extraia.

```typescript
// ❌ A mesma validação repetida em 3 lugares
if (!email || !email.includes('@')) throw new Error('Invalid email');  // no cadastro de usuário
if (!email || !email.includes('@')) throw new Error('Invalid email');  // no login
if (!email || !email.includes('@')) throw new Error('Invalid email');  // na redefinição de senha

// ✅ Escreva uma vez, use em todos os lugares
function validateEmail(email: string): void {
  if (!email || !email.includes('@')) throw new ValidationError('Invalid email');
}
```

### Quando NÃO Refatorar?
Se dois trechos de código semelhantes funcionam em contextos diferentes — não unifique só porque "parecem iguais". Uma abstração errada é pior do que código repetido.

```
❌ Armadilha da abstração excessiva:
"Estas duas funções são 80% iguais, vamos unificar."
→ 3 meses depois uma mudou e a outra não — agora há 10 parâmetros + 5 blocos if.

✅ Abordagem correta:
Mesmo que o código pareça semelhante, se os motivos de mudança forem diferentes → mantenha separado.
```

---

## 2. Limite de Abstração

Antes de criar uma nova abstração (helper function, class, util module):

- [ ] Estou usando este código em 3+ lugares?
- [ ] Ao extrair este código, cada ponto de uso fica mais legível ou mais complexo?
- [ ] Outra pessoa consegue entender esta abstração daqui a 6 meses?

Se a resposta for "sim" para as 3 perguntas → extraia.  
Se uma for "não" → deixe no lugar.

---

## 3. Indicadores de Overdesign

Se você notar qualquer um destes sinais → pare e avise o usuário:

| Sinal | O que significa |
|----------|----------------|
| Factory de factory | Excesso de indireção |
| Função com mais de 5 parâmetros | Tarefa grande demais, divida |
| Interface implementada em apenas 1 lugar | Abstração prematura |
| 20+ funções pequenas em um diretório util | Crescimento de um cesto desorganizado |
| Tudo é configurável | Não fica claro o que aquilo faz |

---

## 4. Momento da Refatoração

Situações em que a refatoração pode ser feita:

- **Detectado durante a tarefa** → termine a tarefa e depois refatore em um commit separado
- **O refactor é pré-requisito de uma feature** → faça primeiro o commit do refactor e depois adicione a feature (2 commits separados no mesmo PR)
- **Refactor grande** (afeta 5+ arquivos) → branch separada, PR separada

```bash
# ✅ Ordem correta dos commits
git commit -m "refactor(validation): extract email validation to shared util"
git commit -m "feat(auth): add password reset flow using shared email validator"

# ❌ Errado — os dois no mesmo commit
git commit -m "feat(auth): add password reset + clean up email validation"
```

---

## 5. Princípio do "Bom o Suficiente"

Toda função deve passar por esta pergunta:

> **"Alguém que veja este código pela primeira vez daqui a 6 meses consegue entender o que ele faz em 2 minutos?"**

- Sim → bom o suficiente, continue
- Não → reescreva (não adicione comentários — deixe o código claro)
