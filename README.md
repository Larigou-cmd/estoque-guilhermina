# Controle de Estoque

Site simples de controle de estoque (itens, quantidade, estoque mínimo,
importação de relatórios do Olostech e estatística de CMM),
usando o Firebase (Firestore) como banco de dados em tempo real.

Não precisa de nenhuma ferramenta de build — são só arquivos estáticos
(`index.html`, `firebase-config.js`) que podem ser hospedados em qualquer
lugar.

## 1. Criar o projeto no Firebase

1. Acesse https://console.firebase.google.com/ e faça login com sua conta Google.
2. Clique em **"Adicionar projeto"**, dê um nome (ex: `controle-estoque`) e siga o assistente (pode desativar o Google Analytics, não é necessário).
3. Dentro do projeto, no menu lateral, vá em **Firestore Database** → **Criar banco de dados**.
   - Escolha uma localização (ex: `southamerica-east1` para Brasil).
   - Comece em **modo de teste** (test mode) — ajustamos as regras de segurança no passo 3 abaixo.
4. Ainda no painel do projeto, clique no ícone **"</>"** ("Adicionar app" → Web) para registrar um app da Web.
   - Dê um nome ao app e clique em **"Registrar app"**.
   - O Firebase vai mostrar um objeto `firebaseConfig` parecido com este:
     ```js
     const firebaseConfig = {
       apiKey: "AIza...",
       authDomain: "controle-estoque-xxxx.firebaseapp.com",
       projectId: "controle-estoque-xxxx",
       storageBucket: "controle-estoque-xxxx.appspot.com",
       messagingSenderId: "123456789",
       appId: "1:123456789:web:abcdef"
     };
     ```
   - Copie esses valores e cole no arquivo [`firebase-config.js`](firebase-config.js) deste projeto, substituindo os textos `SUA_..._AQUI`.

## 2. Configurar as regras de segurança do Firestore

Por padrão, o "modo de teste" libera leitura/escrita para qualquer pessoa por
30 dias e depois bloqueia tudo. Como este site não tem login, defina uma regra
fixa. No Firebase Console, vá em **Firestore Database → Regras** e cole:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /estoque/{itemId} {
      allow read, write: if true;
    }
    match /movimentos/{movId} {
      allow read, write: if true;
    }
  }
}
```

> **Atenção:** essas regras deixam o banco aberto para qualquer pessoa que
> souber a URL do site — ok para uso interno/compartilhado por link, mas não
> recomendado se o site for público. Se precisar restringir o acesso, me
> avise que adicionamos login (Firebase Authentication).

Clique em **Publicar** depois de colar as regras.

## 3. Testar localmente

Basta abrir o `index.html` num servidor local (não funciona bem abrindo o
arquivo direto com duplo-clique, por causa de restrições do navegador). A
forma mais simples, com Python já instalado:

```bash
python -m http.server 8000
```

Depois acesse `http://localhost:8000` no navegador.

## 4. Publicar o site (hospedagem)

Qualquer serviço de hospedagem de site estático funciona. Duas opções fáceis
e gratuitas:

### Opção A — Firebase Hosting (mesma conta do banco)

1. Instale o Firebase CLI: `npm install -g firebase-tools`
2. Rode `firebase login` e depois, dentro desta pasta, `firebase init hosting` (escolha o projeto que você criou, pasta pública `.`, e "single-page app" = não).
3. Rode `firebase deploy`.

### Opção B — Vercel ou Netlify

1. Crie uma conta em https://vercel.com ou https://netlify.com.
2. Arraste esta pasta (`controle-estoque`) para o painel de deploy (ambos aceitam "drag and drop" de pasta), ou conecte a um repositório do GitHub.
3. Pronto — o site fica no ar com uma URL pública.

## 5. Instalar como aplicativo no celular

Depois que o site estiver publicado (passo 4 — precisa ser um endereço `https://`):

- **Android (Chrome):** abra o site → menu **⋮** → **Instalar app** (ou "Adicionar à tela inicial").
- **iPhone (Safari):** abra o site → botão **Compartilhar** (quadrado com seta) → **Adicionar à Tela de Início**.

O ícone aparece junto com os outros apps e abre em tela cheia. Os dados continuam
em tempo real pelo Firebase; sem internet, a tela abre mas não atualiza o estoque.

## Estrutura dos arquivos

- `index.html` — todo o site (interface + lógica).
- `firebase-config.js` — chaves do seu projeto Firebase (edite com os seus dados).
- `manifest.json`, `sw.js`, `icons/` — permitem instalar o site como aplicativo no celular.
- `README.md` — este guia.
