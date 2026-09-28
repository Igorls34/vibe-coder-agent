<!--
  DESCRIÇÃO
  ───────────────
  Regras que mantêm o código limpo e fácil de manter.
  O estilo de comentário, a limpeza de logs de debug e o tratamento
  de variáveis de ambiente e dados sensíveis estão definidos neste arquivo.
  Tudo é verificado antes de cada commit.
-->

---
name: code-hygiene
description: >
  Mantém o código limpo e sustentável. Define o estilo de comentário (somente no
  nível de método), a política de limpeza de logs de debug e o tratamento de
  variáveis de ambiente e dados sensíveis.
---

# Regras de Higiene de Código

## 1. Estilo de Comentário — Somente no Nível de Método

**Não se escreve comentário no meio do código.** Os comentários existem apenas no início de funções/métodos.

```typescript
// ✅ Correto — JSDoc acima do método, explicando o que ele faz
/**
 * Encontra o usuário pelo email e retorna null se não encontrar.
 */
async function findUserByEmail(email: string): Promise<User | null> {
  return db.users.findUnique({ where: { email } });
}

// ❌ Errado — comentário inline
async function findUserByEmail(email: string) {
  // converte o email para minúsculas
  const normalized = email.toLowerCase();
  // procura no banco de dados
  return db.users.findUnique({ where: { email: normalized } });
}
```

**Regra:** Se você sente a necessidade de explicar uma linha, o código não está claro o suficiente. Apague o comentário e corrija o código.

Exceções (as únicas aceitas):
- `// TODO: #123 — descrição` (o número do ticket é obrigatório)
- `// FIXME: #456 — motivo` (o número do ticket é obrigatório)

---

## 2. Limpeza de Logs de Debug

Depois de concluir a tarefa e antes do commit, remova:

| Linguagem | O que remover |
|-----|----------------|
| JavaScript/TypeScript | `console.log`, `console.warn`, `console.error`, `console.table`, `debugger` |
| Python | `print()` (os usados para debug, não para logging) |
| CSS/HTML | `outline: 1px solid red` temporário ou cores de teste |

**Comando de verificação:**
```bash
# Antes do commit — há logs de debug nos arquivos em staging?
git diff --staged | grep -E "console\.(log|warn|error|table)|debugger|print\("
```

O logging real (`logger.info`, `logger.error`) está fora do escopo desta regra — eles permanecem.

---

## 3. Variáveis de Ambiente & Dados Sensíveis

### 3a. Proibido Valor Visível
API key, token, senha, connection string — nenhum deles aparece no código:

```typescript
// ❌ Proibido
const apiKey = "sk-1234abcd...";
const dbUrl = "postgresql://user:pass@host/db";

// ✅ Correto
const apiKey = process.env.OPENAI_API_KEY;
const dbUrl = process.env.DATABASE_URL;
```

### 3b. Valores que Podem Vir da API
Se um valor puder ser obtido da API em runtime → busque da API em vez de escrever um valor fixo no código:

```typescript
// ❌ Errado — embute a config no código
const maxUploadSize = 10 * 1024 * 1024; // 10MB definido no código (hardcoded)

// ✅ Correto — se existir um endpoint de config, busque de lá
const { maxUploadSize } = await configService.getAppConfig();
```

### 3c. Nova Variável de Ambiente → .env.example é Obrigatório
Sempre que uma nova variável de ambiente for adicionada, o `.env.example` é atualizado:

```bash
# .env.example
OPENAI_API_KEY=your-key-here          # Chave da API da OpenAI
DATABASE_URL=postgresql://...          # URL de conexão com o banco de dados
```

A linha de descrição (começando com #) é obrigatória. O valor deve ser um exemplo/placeholder — nunca insira um valor real.
