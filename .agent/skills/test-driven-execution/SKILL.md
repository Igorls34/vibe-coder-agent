---
name: test-driven-execution
description: >
  Antes de escrever qualquer código de implementação, defina os critérios de
  aceitação e os casos de teste que o código precisa satisfazer. Os agentes
  então escrevem código para passar nesses testes — e não para corresponder a
  uma descrição vaga. Elimina resultados do tipo "na minha máquina funciona" e
  "acho que era isso que você queria".
---

<!--
  DESCRIÇÃO
  ───────────────
  Esta skill define os "critérios de sucesso" e os cenários de teste antes de
  qualquer código de implementação ser escrito. O agente escreve o código para
  passar em testes concretos — e não para atender a uma definição vaga. Ela
  aplica o ciclo Vermelho → Verde → Refatoração. Em vez de "acho que funciona",
  ela diz "os testes passam".

  QUANDO: Antes de cada nova funcionalidade ou correção de bug. Nenhum código é
          escrito sem uma spec de teste.
  SAÍDA:   Testes passando, relatório de coverage, resumo de "Tarefa Concluída".
-->

# Skill de Execução Orientada a Testes

## Quando Disparar

- No início de qualquer funcionalidade ou tarefa vinda de um plano
- Quando uma correção de bug é pedida (primeiro o teste, depois a correção)
- Quando um subagente está prestes a implementar um módulo
- Quando "terminou" é ambíguo e precisa de uma definição clara

## Filosofia

> "Uma funcionalidade não está pronta quando o código é escrito. Ela está pronta quando os testes passam."

Testes escritos *antes* do código servem a três propósitos:
1. Eles forçam a clareza sobre o que significa "correto" de fato
2. Eles impedem que o agente se desvie para o over-engineering
3. Eles fornecem verificação instantânea de que a tarefa está completa

## Processo Passo a Passo

### 1. Extrair os Critérios de Aceitação

Leia a descrição da tarefa e extraia todo "deve" e todo "deveria":

**Tarefa de exemplo:** *"Adicionar um endpoint de registro de usuário"*

Critérios extraídos:
- Deve aceitar `email`, `password`, `name` no body da requisição
- Deve retornar 201 em caso de sucesso, com o usuário criado (sem a senha)
- Deve retornar 400 se o email já estiver registrado
- Deve retornar 422 se algum campo estiver faltando ou for inválido
- Deve fazer o hash da senha antes de armazenar

### 2. Escrever a Especificação de Testes

Antes de escrever qualquer implementação, escreva os testes (ou os outlines):

#### Para testes unitários:

```javascript
describe('registerUser()', () => {
  it('creates a user with hashed password', async () => { ... })
  it('throws DuplicateEmailError if email already exists', async () => { ... })
  it('throws ValidationError if email format is invalid', async () => { ... })
  it('never stores plaintext password', async () => { ... })
})
```

#### Para testes de integração/API:

```javascript
describe('POST /api/auth/register', () => {
  it('returns 201 and user object (without password) on success', async () => { ... })
  it('returns 400 when email already registered', async () => { ... })
  it('returns 422 when required fields missing', async () => { ... })
})
```

#### Para testes de UI/E2E:

```
Scenario: Successful registration
  Given I am on the /register page
  When I fill in valid email, password, and name
  And I click "Create Account"
  Then I should be redirected to /dashboard
  And I should see "Welcome, <name>"
```

### 3. Confirmar que os Testes Falham (Vermelho)

Rode os testes *antes* de escrever qualquer implementação:

```bash
npm test
# ou
pytest
# etc.
```

**Esperado:** todos os novos testes falham. Isso confirma que os testes estão detectando corretamente a ausência da funcionalidade. Se um teste passar antes da implementação, o teste está errado — corrija-o.

### 4. Escrever a Implementação Mínima (Verde)

Agora implemente a funcionalidade. O objetivo é simples: **fazer os testes passarem**.

Regras durante a implementação:
- Escreva apenas o que for necessário para os testes passarem — nada além
- Se você se pegar escrevendo código sem teste correspondente, pare e escreva o teste primeiro
- Não otimize ainda — corretude primeiro

```bash
# Continue rodando os testes conforme avança
npm test --watch
```

### 5. Refatoração (Refatorar)

Depois que todos os testes passarem:
- Limpe o código (nomenclatura, estrutura, duplicação)
- Rode os testes novamente após cada passo de refatoração para confirmar que nada quebrou
- Adicione testes de edge case para tudo o que for descoberto durante a implementação

### 6. Definir o Checklist de Conclusão

Uma tarefa está completa quando:

- [ ] Todos os critérios de aceitação da Etapa 1 têm um teste correspondente
- [ ] Todos os testes passam (`npm test` termina com código 0)
- [ ] Nenhum teste foi pulado (sem `.skip`, `xtest`, `# noqa`)
- [ ] A cobertura de código do novo módulo é ≥ 80% (verifique com `--coverage`)
- [ ] Nenhum teste existente foi quebrado por esta mudança

### 7. Reportar a Conclusão

```
## Tarefa Concluída: <nome da funcionalidade>

**Testes escritos:** <N>
**Testes passando:** <N>/<N>
**Cobertura:** <X>%
**Critérios de aceitação cobertos:**
  ✅ Cria o usuário com a senha em hash
  ✅ Retorna 400 em caso de email duplicado
  ✅ Retorna 422 em caso de falha de validação
  ✅ Nunca armazena a senha em texto puro

**Não cobertos (e por quê):**
  ⚠️ Teste de entrega de email — pulado, exige a configuração de um SMTP mock
```

## Convenção de Nomenclatura de Testes

Os nomes dos testes devem ser lidos como requisitos:

✅ `it('returns 400 when email already exists')`
✅ `it('never exposes password in API response')`
❌ `it('test email duplicate')`
❌ `it('works correctly')`

Se você não consegue nomear o teste claramente, o requisito não está claro o suficiente — esclareça antes de escrever.

## Regras

- **Nenhuma implementação antes da spec de teste.** A spec pode ser pseudocódigo / outlines, mas ela precisa existir.
- **Um teste que falha não é uma falha** — é um teste escrito corretamente na fase vermelha.
- **Nunca exclua um teste para fazer a suíte passar.** Corrija o código ou corrija o teste — com justificativa.
- **Testes de integração não são opcionais** para APIs públicas.
- **Coverage % é um proxy, não uma meta.** 100% de cobertura com asserções sem sentido é pior do que 70% com asserções de verdade.
