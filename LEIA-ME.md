# Build automático do APK e do Software — Funchal

Dois projetos, cada um com seu robô do GitHub Actions. A partir do momento em
que estiverem no repositório `funchal-app` (o mesmo onde vivem `index.html`,
`sistema.html` e `versao.json`), toda vez que você (ou eu) editar essas duas
telas e publicar, o `.apk` e o `.exe` saem sozinhos, sem ninguém rodar nada
na mão.

## 1. Onde colocar

Dentro do repositório `funchal-app`, ao lado do que já existe:

```
funchal-app/
├── index.html
├── sistema.html
├── versao.json
├── versao-sistema.json
├── .github/workflows/publicar.yml      ← já existe
├── funchal-mobile/                     ← pasta nova (deste pacote)
│   ├── android/
│   ├── www/index.html
│   ├── package.json
│   └── capacitor.config.ts
├── funchal-desktop/                    ← pasta nova (deste pacote)
│   ├── main.js
│   ├── preload.js
│   ├── build/icon.ico
│   ├── www/sistema.html
│   └── package.json
└── .github/workflows/
    ├── build-apk.yml                   ← deste pacote
    └── build-windows.yml               ← deste pacote
```

Os dois `www/*.html` dentro das pastas novas são só a cópia que os robôs
mesmos atualizam a cada build — não precisa mexer neles.

## 2. Assinatura do APK (opcional, mas recomendada)

Sem isso, o `.apk` sai assinado com a chave de debug — instala e funciona
normalmente, só não é o ideal pra distribuir. Pra assinar de verdade:

```bash
keytool -genkeypair -v -keystore funchal.keystore -alias funchal \
  -keyalg RSA -keysize 2048 -validity 10000
```

Depois, em **Settings → Secrets and variables → Actions** do repositório,
crie:

| Nome do Secret | Valor |
|---|---|
| `ANDROID_KEYSTORE_BASE64` | `base64 -w0 funchal.keystore` (o texto que isso imprime) |
| `ANDROID_KEYSTORE_PASSWORD` | a senha que você digitou no `keytool` |
| `ANDROID_KEY_ALIAS` | `funchal` |
| `ANDROID_KEY_PASSWORD` | a senha da chave (geralmente igual à do keystore) |

Guarde o `funchal.keystore` em lugar seguro — se perder, todo APK novo passa
a ser visto pelo Android como "de outro aplicativo" e não substitui o
instalado; todo mundo precisaria desinstalar e instalar de novo.

## 3. O que acontece a cada publicação

- Editar `index.html` → dispara `build-apk.yml` → gera `Funchal-Mobile.apk`
  na raiz do repositório e atualiza `apkVersionCode`/`apkVersionName`/`apkUrl`
  dentro de `versao.json`.
- Editar `sistema.html` → dispara `build-windows.yml` → gera
  `Funchal-Setup.exe` na raiz e atualiza `exeVersion`/`exeUrl` dentro de
  `versao-sistema.json`.
- Quem já tem o app/programa instalado confere a cada abertura e continua se
  atualizando sozinho pelo mecanismo que já existia — o build novo só entra
  em jogo quando sai um `.apk`/`.exe` realmente novo (mudança na casca, não
  no conteúdo do dia a dia).

## 4. Primeira instalação

Isso tudo cuida das atualizações seguintes — a primeira instalação, alguém
baixa uma vez:
- **Android**: baixe `Funchal-Mobile.apk` pelo link do GitHub Pages,
  permita "instalar de fontes desconhecidas" na primeira vez, instale.
- **Windows**: baixe `Funchal-Setup.exe`, rode o instalador.

## 5. Rodando localmente (pra testar antes de publicar)

```bash
# App Android
cd funchal-mobile && npm install
npx cap sync android
npx cap run android         # precisa do Android Studio/emulador instalado

# Software Windows
cd funchal-desktop && npm install
npm start                   # abre a janela igual ao instalado
```
