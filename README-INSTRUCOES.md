# MR. GREEN — Guia de Configuração do Sistema de Trial + Paywall

## O que foi adicionado

### 1. `index.html` (modificado)
Um script de controle de acesso foi injetado. Ele:
- **Detecta o primeiro acesso** → salva a hora de início e libera 24h gratuitas
- **Enquanto o trial está ativo** → exibe uma barra fixa na parte inferior com countdown ao vivo
- **Quando as 24h expiram** → redireciona automaticamente para `paywall.html`
- **Usuário com pagamento confirmado** → acesso liberado sem interrupção

### 2. `paywall.html` (novo)
Página de planos de pagamento com:
- Banner mostrando se o trial expirou ou ainda está ativo
- Cards dos planos **Mensal** e **Vitalício**
- Botões de checkout que redirecionam para a PerfectPay
- Design consistente com o MR. GREEN

---

## ⚙️ Como configurar

### Passo 1 — Configure os links da PerfectPay no `paywall.html`

Abra `paywall.html` e localize o bloco `CONFIG` no script (perto do final do arquivo):

```javascript
const CONFIG = {
  perfectpay: {
    monthly:  'https://perfectpay.com.br/pay/SEU-LINK-MENSAL',    // ← substitua
    lifetime: 'https://perfectpay.com.br/pay/SEU-LINK-VITALICIO', // ← substitua
  },
  prices: {
    monthly:  '29,90',   // ← preço exibido (só visual)
    lifetime: '97,00',   // ← preço exibido (só visual)
  },
};
```

Substitua `SEU-LINK-MENSAL` e `SEU-LINK-VITALICIO` pelos checkouts gerados na PerfectPay.

### Passo 2 — Configure o webhook pós-pagamento (importante!)

O sistema atual usa `localStorage` para controlar o acesso, o que é suficiente para começar. Mas para segurança real, você precisa:

1. Na PerfectPay, configure um **webhook** que chame seu backend após o pagamento confirmado
2. Seu backend salva o acesso em um banco de dados (ex: Firebase, Supabase, PlanetScale)
3. No `index.html`, ao carregar, você consulta o backend para validar o acesso

#### Solução rápida com Supabase (gratuito)
Se quiser um banco de dados real sem servidor:
1. Crie conta em [supabase.com](https://supabase.com)
2. Crie uma tabela `users` com colunas: `email`, `plan`, `activated_at`, `expires_at`
3. Configure o webhook da PerfectPay para inserir/atualizar nessa tabela
4. No `index.html`, substitua a lógica de `localStorage` por uma consulta à API do Supabase

---

## 🔑 Como liberar acesso manualmente (para testes)

Abra o console do navegador (F12) no site e cole:

```javascript
// Simular pagamento mensal aprovado
localStorage.setItem('mrgreen_paid', JSON.stringify({
  plan: 'monthly',
  activated: Date.now(),
  email: 'teste@email.com'
}));
location.reload();

// Simular pagamento vitalício aprovado
localStorage.setItem('mrgreen_paid', JSON.stringify({
  plan: 'lifetime',
  activated: Date.now(),
  email: 'teste@email.com'
}));
location.reload();

// Resetar para simular primeiro acesso (trial zerado)
localStorage.removeItem('mrgreen_trial');
localStorage.removeItem('mrgreen_paid');
location.reload();

// Simular trial expirado (para ver o redirect para paywall)
localStorage.setItem('mrgreen_trial', JSON.stringify({
  start: Date.now() - (25 * 60 * 60 * 1000), // 25h atrás
  version: 1
}));
location.reload();
```

---

## 🚀 Deploy no Vercel

Como o projeto é estático (Next.js exportado), suba os arquivos normalmente:

```bash
# Se usar Vercel CLI
vercel --prod

# Ou arraste a pasta pelo painel em vercel.com
```

Certifique-se que `paywall.html` está na raiz do projeto junto com `index.html`.

---

## ⚠️ Limitações do sistema atual

| Recurso | Status atual | Solução futura |
|---------|-------------|----------------|
| Controle de acesso | localStorage (por dispositivo) | Backend + banco de dados |
| Verificação de pagamento | Manual / webhook | Webhook PerfectPay → Backend |
| Multi-dispositivo | ❌ Não sincroniza | Login com email + JWT |
| Bypass fácil | ⚠️ Via console | Backend valida cada request |

Para um produto em produção sério, o passo seguinte é adicionar autenticação real (ex: NextAuth, Supabase Auth ou Firebase Auth) vinculada ao email usado no pagamento.

---

*Arquivos modificados: `index.html`, `paywall.html` (novo), `README-INSTRUCOES.md` (novo)*
