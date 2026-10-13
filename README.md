# RA Cosméticos — Catálogo online com pedido via WhatsApp

Catálogo de produtos em HTML/CSS/JS puro (sem build) que usa **Firebase Firestore** como banco de dados, **Firebase Authentication** para o painel admin e **Firebase Hosting** para publicar. O cliente monta o carrinho e o pedido é enviado pelo WhatsApp.

## Arquivos

| Arquivo | Função |
|---|---|
| `index.html` + `style.css` + `script.js` | Loja pública (lê produtos, categorias, cupons e configurações do Firestore) |
| `admin.html` | Painel administrativo (login, produtos, categorias, cupons, vendas, configurações) |
| `seed-firebase.html` | Carga inicial: envia 6 produtos, as categorias (Maquiagens, Skin care, Acessórios) e o cupom `DESCONTO10` ao Firestore |
| `regras.md` | Regras de segurança do Firestore para colar no console |
| `firebase.json` | Configuração do Firebase Hosting (já ignora seed, regras e README no deploy) |
| `assets/` | Imagens dos produtos e logos |

## Passo a passo

### 1. Criar o projeto no Firebase
1. Acesse <https://console.firebase.google.com> → **Adicionar projeto** (nome sugerido: `ra-cosmetico`).
2. Em **Visão geral → `</>` (Web)**, registre um app web. Copie o objeto `firebaseConfig`.
3. Em **Build → Firestore Database → Criar banco de dados** (modo produção, região `southamerica-east1`).
4. Em **Build → Authentication → Sign-in method**, ative **E-mail/senha**.
5. Em **Authentication → Users → Adicionar usuário**, crie o e-mail e a senha do admin.

### 2. Colar o `firebaseConfig` (obrigatório — sem isso a loja não carrega)
O mesmo bloco precisa ser preenchido em **3 arquivos**:

- `script.js` (linhas 1–13, loja)
- `admin.html` (bloco `firebaseConfig`, por volta da linha 542)
- `seed-firebase.html` (bloco `firebaseConfig`, por volta da linha 45)

```js
const firebaseConfig = {
  apiKey: "...",
  authDomain: "SEU-PROJETO.firebaseapp.com",
  projectId: "SEU-PROJETO",
  storageBucket: "SEU-PROJETO.appspot.com",
  messagingSenderId: "...",
  appId: "..."
};
```

> A `apiKey` do Firebase web não é segredo; a segurança vem das regras do Firestore (passo 3).

### 3. Publicar as regras do Firestore
Console → **Firestore Database → Regras** → cole o bloco de [regras.md](regras.md) → **Publicar**.
Leitura pública para produtos/categorias/cupons/configurações; escrita e vendas só para usuário logado.

### 4. Rodar localmente
Use um servidor local (abrir o `index.html` com duplo clique via `file://` pode falhar com o Firebase Auth):

```bash
npx serve .
# ou: python -m http.server 5500
```

Abra `http://localhost:5500/seed-firebase.html`.

### 5. Carga inicial dos dados (uma única vez)
1. Em `seed-firebase.html`, faça login com o usuário admin criado no passo 1.
2. Clique em **Enviar Produtos ao Firebase**. Os documentos usam ID fixo, então rodar de novo não duplica.
3. Abra `index.html` e confira que produtos e categorias aparecem.
4. Abra `admin.html`, entre com o admin e ajuste nome da loja, **WhatsApp**, cores, banner, horários de retirada, produtos e cupons.

### 6. Implantar no Firebase Hosting
```bash
npm install -g firebase-tools
firebase login
firebase init hosting     # escolha o projeto existente; public = "."; NÃO sobrescrever index.html; SPA = não
firebase deploy --only hosting
```

O `firebase.json` já está no projeto. Ao final o CLI mostra a URL `https://SEU-PROJETO.web.app` — a loja fica na raiz e o painel em `/admin.html`.

Depois do deploy, remova o acesso ao seed se quiser: ele já não é publicado (está no `ignore`).

### 7. Domínio próprio (opcional)
Console → **Hosting → Adicionar domínio personalizado** e siga os registros DNS. Em **Authentication → Settings → Domínios autorizados** adicione o domínio, senão o login do admin falha.

## Solução de problemas

| Sintoma | Causa provável |
|---|---|
| Loja vazia / "Nenhum produto encontrado" | `firebaseConfig` vazio, seed não executado ou regras não publicadas (veja o Console do navegador, F12) |
| `Missing or insufficient permissions` | Regras do passo 3 não publicadas, ou usuário não está logado no admin/seed |
| Login do admin não funciona | E-mail/senha não ativado no Authentication, ou domínio não autorizado |
| Imagem quebrada após o deploy | O Hosting diferencia maiúsculas/minúsculas; o nome no Firestore deve ser idêntico ao arquivo em `assets/` |
| Pedido abre WhatsApp do número errado | Ajuste em **admin → Configurações → WhatsApp** (padrão atual: `558182362638`, formato DDI+DDD+número) |

## Ajustes já aplicados neste projeto
- Botão "Todos" não aparece mais duplicado (a categoria `all` do Firestore é ignorada na barra, que já cria o seu próprio botão).
- Seed grava o cupom com ID = código, evitando duplicar ao rodar de novo.
- `firebase.json` adicionado para o deploy no Hosting.
