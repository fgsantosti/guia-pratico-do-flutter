# Apêndice A - Configuração do Ambiente

## 📋 Índice

1. [O que Você Vai Precisar](#-o-que-você-vai-precisar)
2. [Instalação no Windows](#-instalação-no-windows)
3. [Instalação no macOS](#-instalação-no-macos)
4. [Instalação no Linux](#-instalação-no-linux)
5. [Android Studio e Android SDK](#-android-studio-e-android-sdk)
6. [Configurando o VS Code](#-configurando-o-vs-code)
7. [Emulador Android](#-emulador-android)
8. [Celular Físico](#-celular-físico)
9. [Verificando com flutter doctor](#-verificando-com-flutter-doctor)
10. [Criando e Rodando o Primeiro Projeto](#-criando-e-rodando-o-primeiro-projeto)
11. [Problemas Comuns](#-problemas-comuns)

## 🧰 O que Você Vai Precisar

| Item | Obrigatório? | Para quê |
|------|--------------|----------|
| Flutter SDK | ✅ | Compilar e executar apps |
| Git | ✅ | O Flutter usa para se atualizar |
| Android Studio | ✅ para Android | Android SDK, emulador e ferramentas |
| VS Code | Recomendado | Editor leve e popular |
| Xcode (somente macOS) | ✅ para iOS | Compilar para iPhone |
| Google Chrome | Opcional | Rodar o app na Web |

**Requisitos mínimos sugeridos:** 8 GB de RAM (16 GB para usar o emulador com folga), 20 GB livres em disco e processador com virtualização ativada.

## 🪟 Instalação no Windows

### 1. Instale o Git

Baixe em [git-scm.com](https://git-scm.com/download/win) e instale com as opções padrão.

### 2. Baixe o Flutter SDK

A forma mais simples é pelo **VS Code** (veja a seção [Configurando o VS Code](#-configurando-o-vs-code)), que baixa e configura o SDK automaticamente.

Para instalar manualmente:

1. Baixe o arquivo `.zip` da versão **stable** em [docs.flutter.dev/get-started/install/windows](https://docs.flutter.dev/get-started/install/windows).
2. Extraia em uma pasta **sem espaços e sem acentos**, por exemplo `C:\dev\flutter`.

> ⚠️ Não extraia em `C:\Program Files\`: essa pasta exige permissão de administrador e causa erros.

### 3. Adicione o Flutter ao PATH

1. No menu Iniciar, pesquise **"variáveis de ambiente"** e abra "Editar as variáveis de ambiente para sua conta".
2. Selecione a variável **Path** → **Editar** → **Novo**.
3. Adicione `C:\dev\flutter\bin`.
4. Clique em OK e **abra um novo terminal**.

### 4. Teste

```powershell
flutter --version
```

## 🍎 Instalação no macOS

### 1. Ferramentas de linha de comando

```bash
xcode-select --install
```

### 2. Flutter SDK

Com o [Homebrew](https://brew.sh):

```bash
brew install --cask flutter
```

Ou manualmente: baixe o `.zip` em [docs.flutter.dev](https://docs.flutter.dev/get-started/install/macos), extraia em `~/development/flutter` e adicione ao PATH no arquivo `~/.zshrc`:

```bash
export PATH="$HOME/development/flutter/bin:$PATH"
```

```bash
source ~/.zshrc
flutter --version
```

### 3. Para desenvolver para iOS

1. Instale o **Xcode** pela App Store.
2. Configure:

   ```bash
   sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
   sudo xcodebuild -runFirstLaunch
   sudo xcodebuild -license accept
   ```

3. Instale o CocoaPods (gerenciador de dependências do iOS):

   ```bash
   brew install cocoapods
   ```

4. Abra o simulador:

   ```bash
   open -a Simulator
   ```

## 🐧 Instalação no Linux

### 1. Dependências do sistema (Ubuntu/Debian)

```bash
sudo apt-get update
sudo apt-get install -y curl git unzip xz-utils zip libglu1-mesa
```

Para também compilar apps **desktop Linux**:

```bash
sudo apt-get install -y clang cmake ninja-build pkg-config libgtk-3-dev
```

### 2. Flutter SDK

Pelo Snap:

```bash
sudo snap install flutter --classic
```

Ou manualmente: baixe o `.tar.xz` em [docs.flutter.dev](https://docs.flutter.dev/get-started/install/linux) e extraia:

```bash
mkdir -p ~/development
tar -xf ~/Downloads/flutter_linux_*-stable.tar.xz -C ~/development/
echo 'export PATH="$HOME/development/flutter/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
flutter --version
```

### 3. Aceleração do emulador (KVM)

```bash
sudo apt-get install -y qemu-kvm
sudo adduser $USER kvm
# Faça logout e login novamente
```

## 🤖 Android Studio e Android SDK

Mesmo que você vá programar no VS Code, o Android Studio é a forma mais fácil de instalar o **Android SDK**.

1. Baixe e instale em [developer.android.com/studio](https://developer.android.com/studio).
2. Na primeira execução, siga o assistente com a opção **Standard**. Ele baixa o SDK e cria um emulador.
3. Abra **More Actions → SDK Manager → SDK Tools** e marque:
   - Android SDK Command-line Tools (latest)
   - Android SDK Build-Tools
   - Android SDK Platform-Tools
   - Android Emulator
4. Instale os plugins **Flutter** e **Dart** em **Plugins → Marketplace** (se for usar o Android Studio como editor).
5. Aceite as licenças:

   ```bash
   flutter doctor --android-licenses
   ```

   Responda `y` para todas.

## 💙 Configurando o VS Code

1. Instale o [VS Code](https://code.visualstudio.com).
2. Abra a aba **Extensões** (`Ctrl+Shift+X`) e instale a extensão **Flutter** (ela instala a extensão Dart junto).
3. Se o Flutter SDK ainda não estiver instalado: abra a paleta de comandos (`Ctrl+Shift+P`), digite **"Flutter: New Project"** e clique em **Download SDK** quando solicitado.

### Extensões úteis

| Extensão | Para quê |
|----------|----------|
| Flutter / Dart | Obrigatórias: autocompletar, depuração, Hot Reload |
| Error Lens | Mostra os erros na própria linha do código |
| Pubspec Assist | Adiciona pacotes ao `pubspec.yaml` pela paleta de comandos |
| Awesome Flutter Snippets | Atalhos para criar widgets comuns |

### Configurações recomendadas (`settings.json`)

```json
{
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll": "explicit"
  },
  "dart.previewFlutterUiGuides": true,
  "[dart]": {
    "editor.rulers": [80],
    "editor.defaultFormatter": "Dart-Code.dart-code"
  }
}
```

## 📱 Emulador Android

### Criando um emulador

1. No Android Studio: **More Actions → Virtual Device Manager → Create Device**.
2. Escolha um aparelho (ex.: Pixel 8) e uma imagem de sistema recente com **Google Play**.
3. Clique em **Finish** e no botão ▶️ para iniciar.

### Pela linha de comando

```bash
flutter emulators                       # Lista os emuladores
flutter emulators --launch <nome>       # Abre um emulador
flutter devices                         # Lista os aparelhos disponíveis
```

> 💡 O emulador consome muita memória. Em computadores mais modestos, um celular físico ou a opção **Chrome** (Web) podem ser mais rápidos para estudar.

## 📲 Celular Físico

### Android via USB

1. No celular, abra **Configurações → Sobre o telefone** e toque **7 vezes** em **Número da versão**. Isso ativa as "Opções do desenvolvedor".
2. Em **Configurações → Sistema → Opções do desenvolvedor**, ative a **Depuração USB**.
3. Conecte o cabo e aceite a mensagem "Permitir depuração USB?" no celular.
4. Confira:

   ```bash
   flutter devices
   ```

> ⚠️ No Windows, alguns aparelhos precisam do driver USB do fabricante. No Linux, se o aparelho aparecer como "unauthorized", desconecte, revogue as autorizações de depuração no celular e conecte de novo.

### Android via Wi-Fi (Android 11+)

1. Celular e computador na **mesma rede Wi-Fi**.
2. Em **Opções do desenvolvedor → Depuração por Wi-Fi**, ative e toque em **Parear dispositivo com código**.
3. No computador:

   ```bash
   adb pair IP:PORTA_DE_PAREAMENTO     # Digite o código mostrado no celular
   adb connect IP:PORTA                # A porta exibida na tela principal da depuração por Wi-Fi
   flutter devices
   ```

### iPhone (somente macOS)

1. Conecte o iPhone e toque em **Confiar neste computador**.
2. Ative o **Modo de Desenvolvedor** em **Ajustes → Privacidade e Segurança**.
3. Abra `ios/Runner.xcworkspace` no Xcode, selecione seu **Team** em **Signing & Capabilities** (uma conta Apple gratuita serve para testes).

## 🩺 Verificando com flutter doctor

O `flutter doctor` verifica se tudo está instalado corretamente:

```bash
flutter doctor
```

Exemplo de saída:

```
Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel stable, 3.x.x, on Ubuntu 24.04 LTS)
[✓] Android toolchain - develop for Android devices (Android SDK version 35.0.0)
[✓] Chrome - develop for the web
[✓] Linux toolchain - develop for Linux desktop
[✓] Android Studio (version 2025.x)
[✓] VS Code (version 1.x)
[✓] Connected device (2 available)
[✓] Network resources

• No issues found!
```

| Símbolo | Significado |
|---------|-------------|
| `[✓]` | Tudo certo |
| `[!]` | Funciona, mas há um aviso |
| `[✗]` | Faltando: siga a instrução mostrada |

> 💡 Você **não** precisa de tudo marcado. Para desenvolver só para Android, por exemplo, os itens de Xcode e Linux toolchain podem ficar com `[✗]`.

## 🎉 Criando e Rodando o Primeiro Projeto

```bash
flutter create meu_primeiro_app
cd meu_primeiro_app
flutter run
```

Se houver mais de um aparelho conectado, o Flutter pergunta qual usar. Para escolher direto:

```bash
flutter run -d chrome          # Navegador
flutter run -d emulator-5554   # Um emulador específico (veja o id em flutter devices)
```

Com o app rodando, no terminal:

| Tecla | Ação |
|-------|------|
| `r` | Hot Reload |
| `R` | Hot Restart |
| `q` | Encerrar |

Agora siga para o [Capítulo 1](./chapter_1.md)!

## 🐛 Problemas Comuns

| Problema | Solução |
|----------|---------|
| `flutter: command not found` / "não é reconhecido como comando" | A pasta `flutter/bin` não está no PATH. Confira e **abra um novo terminal** |
| `Android license status unknown` | Rode `flutter doctor --android-licenses` |
| `cmdline-tools component is missing` | Instale "Android SDK Command-line Tools" no SDK Manager |
| `Unable to locate Android SDK` | `flutter config --android-sdk /caminho/para/Android/Sdk` |
| Emulador muito lento | Ative a virtualização (VT-x/AMD-V) na BIOS e use uma imagem `x86_64` |
| Primeiro `flutter run` demora muito | Normal: o Gradle baixa dependências na primeira vez. As próximas são bem mais rápidas |
| Erro de Gradle ou Java incompatível | `flutter doctor -v` mostra o Java usado; o Flutter usa o JDK do Android Studio. Atualize o Android Studio e o Flutter (`flutter upgrade`) |
| `CocoaPods not installed` (macOS) | `brew install cocoapods` |
| Celular não aparece em `flutter devices` | Confira o cabo (alguns só carregam), a depuração USB e aceite a autorização no celular |

### Mantendo o Flutter atualizado

```bash
flutter upgrade         # Atualiza o SDK
flutter --version       # Mostra a versão atual
flutter channel         # Mostra o canal (use sempre "stable")
```
