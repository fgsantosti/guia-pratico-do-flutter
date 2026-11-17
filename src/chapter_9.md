# Capítulo 9 - Publicação de Apps

## 📋 Índice

1. [Introdução](#-introdução)
2. [Checklist Antes de Publicar](#-checklist-antes-de-publicar)
3. [Passo 1: Nome do App](#-passo-1-nome-do-app)
4. [Passo 2: Identificador do Pacote](#-passo-2-identificador-do-pacote)
5. [Passo 3: Ícone do App](#-passo-3-ícone-do-app)
6. [Passo 4: Versionamento](#-passo-4-versionamento)
7. [Passo 5: Assinatura do App Android](#-passo-5-assinatura-do-app-android)
8. [Passo 6: Gerando o Build Android](#-passo-6-gerando-o-build-android)
9. [Passo 7: Publicando na Google Play](#-passo-7-publicando-na-google-play)
10. [Passo 8: Publicando na App Store](#-passo-8-publicando-na-app-store)
11. [Passo 9: Publicando para Web](#-passo-9-publicando-para-web)
12. [Ambientes de Desenvolvimento e Produção](#-ambientes-de-desenvolvimento-e-produção)
13. [Problemas Comuns](#-problemas-comuns)
14. [Recursos Adicionais](#-recursos-adicionais)

> 💡 A configuração da **splash screen** está na seção [9.1 - Splash Screen](./chapter_9_splash.md).

## 🚀 Introdução

Seu app está pronto e funcionando no emulador. Agora é hora de colocá-lo nas mãos dos usuários! Publicar envolve alguns passos que não aparecem no dia a dia do desenvolvimento: dar nome e ícone ao app, assinar digitalmente o pacote, gerar o build de **release** e preencher as informações nas lojas.

### Debug x Release

| | Debug | Release |
|---|-------|---------|
| Comando | `flutter run` | `flutter build ...` |
| Hot Reload | ✅ | ❌ |
| Desempenho | Mais lento | Otimizado (código compilado AOT) |
| Tamanho | Grande | Reduzido |
| Faixa "DEBUG" | Aparece | Não aparece |
| Uso | Desenvolvimento | Usuários finais |

Para testar o desempenho real no celular antes de publicar:

```bash
flutter run --release
```

## ✅ Checklist Antes de Publicar

- [ ] Nome e ícone definitivos
- [ ] Splash screen configurada ([seção 9.1](./chapter_9_splash.md))
- [ ] Identificador do pacote único (ex.: `br.com.suaempresa.app`)
- [ ] Versão atualizada no `pubspec.yaml`
- [ ] Permissões necessárias declaradas (internet, câmera, localização...)
- [ ] `debugShowCheckedModeBanner: false` no `MaterialApp`
- [ ] Nenhum `print` com dados sensíveis
- [ ] `flutter analyze` sem erros
- [ ] Testes passando (`flutter test`, veja o Capítulo 7)
- [ ] App testado em modo release em um celular real
- [ ] Política de privacidade publicada em uma URL (exigida pelas lojas)
- [ ] Capturas de tela e textos de descrição prontos

## 🏷 Passo 1: Nome do App

O nome que aparece abaixo do ícone na tela do celular é configurado separadamente em cada plataforma.

### Android

Em `android/app/src/main/AndroidManifest.xml`:

```xml
<application
    android:label="Minhas Tarefas"
    android:icon="@mipmap/ic_launcher">
```

### iOS

Em `ios/Runner/Info.plist`:

```xml
<key>CFBundleDisplayName</key>
<string>Minhas Tarefas</string>
```

> 💡 Nomes curtos (até cerca de 12 caracteres) evitam ser cortados na tela inicial.

## 🆔 Passo 2: Identificador do Pacote

Todo app tem um identificador único no mundo, no formato de domínio invertido: `br.com.suaempresa.minhastarefas`. O padrão gerado pelo Flutter é `com.example.nome_do_app`, e **as lojas rejeitam `com.example`**.

> ⚠️ Depois que o app é publicado, o identificador **nunca mais** pode ser alterado. Escolha com cuidado.

### Android

Em `android/app/build.gradle.kts` (ou `build.gradle` em projetos antigos):

```kotlin
android {
    namespace = "br.com.suaempresa.minhastarefas"

    defaultConfig {
        applicationId = "br.com.suaempresa.minhastarefas"
        // ...
    }
}
```

Se mudar o `namespace`, mova também o arquivo `MainActivity.kt` para a pasta correspondente (`kotlin/br/com/suaempresa/minhastarefas/`) e ajuste a linha `package` no topo dele.

### iOS

Abra `ios/Runner.xcworkspace` no Xcode, selecione **Runner → Signing & Capabilities** e altere o **Bundle Identifier**.

> 💡 O pacote [`change_app_package_name`](https://pub.dev/packages/change_app_package_name) automatiza a troca no Android e no iOS: `dart run change_app_package_name:main br.com.suaempresa.minhastarefas`.

## 🎨 Passo 3: Ícone do App

Cada plataforma exige o ícone em vários tamanhos. O pacote [`flutter_launcher_icons`](https://pub.dev/packages/flutter_launcher_icons) gera todos a partir de uma única imagem.

```bash
flutter pub add dev:flutter_launcher_icons
```

Crie uma imagem quadrada de **1024×1024 px** em `assets/icon/icon.png` e adicione ao `pubspec.yaml`:

```yaml
flutter_launcher_icons:
  android: true
  ios: true
  image_path: "assets/icon/icon.png"

  # Ícone adaptativo do Android (8.0+): fundo + desenho em primeiro plano
  adaptive_icon_background: "#1E88E5"
  adaptive_icon_foreground: "assets/icon/icon_foreground.png"

  # A App Store não aceita transparência no ícone
  remove_alpha_ios: true
```

```bash
dart run flutter_launcher_icons
```

### Dicas de ícone

- Sem textos pequenos: o ícone é exibido com cerca de 48 px.
- No ícone adaptativo, mantenha o desenho dentro do **círculo central** (cerca de 66% da imagem), pois cada fabricante recorta em formato diferente.
- Teste em fundo claro e escuro.

## 🔢 Passo 4: Versionamento

A versão fica no `pubspec.yaml`:

```yaml
version: 1.2.0+5
```

| Parte | Nome | Para que serve |
|-------|------|----------------|
| `1.2.0` | Nome da versão (`versionName` / `CFBundleShortVersionString`) | O que o usuário vê na loja |
| `5` | Número do build (`versionCode` / `CFBundleVersion`) | Controle interno; **precisa aumentar a cada envio** |

O nome da versão segue o **versionamento semântico** (`MAIOR.MENOR.CORREÇÃO`):

- `1.0.0 → 1.0.1`: correção de bug
- `1.0.1 → 1.1.0`: nova funcionalidade compatível
- `1.1.0 → 2.0.0`: mudança grande ou incompatível

Também é possível sobrescrever na hora do build:

```bash
flutter build appbundle --build-name=1.2.0 --build-number=6
```

> ⚠️ A Google Play recusa um envio com `versionCode` igual ou menor a um já enviado. Sempre aumente o número depois do `+`.

## 🔐 Passo 5: Assinatura do App Android

A Google Play exige que o app seja **assinado digitalmente** com uma chave sua. Ela prova que as atualizações vêm do mesmo autor.

### 1. Criar a chave (keystore)

Linux/macOS:

```bash
keytool -genkey -v -keystore ~/upload-keystore.jks \
  -keyalg RSA -keysize 2048 -validity 10000 -alias upload
```

Windows (PowerShell):

```powershell
keytool -genkey -v -keystore $env:USERPROFILE\upload-keystore.jks `
  -storetype JKS -keyalg RSA -keysize 2048 -validity 10000 -alias upload
```

O comando pede uma senha e alguns dados (nome, organização). **Guarde a senha e faça backup do arquivo `.jks`.**

> 💡 Se o comando `keytool` não for encontrado, rode `flutter doctor -v` e procure a linha "Java binary at". O `keytool` fica na mesma pasta.

### 2. Criar o arquivo `android/key.properties`

```properties
storePassword=SUA_SENHA
keyPassword=SUA_SENHA
keyAlias=upload
storeFile=/home/seu_usuario/upload-keystore.jks
```

> ⚠️ **Nunca** envie `key.properties` nem o `.jks` para o GitHub. Adicione ao `.gitignore`:
>
> ```
> android/key.properties
> *.jks
> *.keystore
> ```

### 3. Configurar o Gradle

Em `android/app/build.gradle.kts`, antes do bloco `android {`:

```kotlin
import java.util.Properties
import java.io.FileInputStream

val keystoreProperties = Properties()
val keystorePropertiesFile = rootProject.file("key.properties")
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(FileInputStream(keystorePropertiesFile))
}
```

E dentro do bloco `android { ... }`:

```kotlin
android {
    // ...

    signingConfigs {
        create("release") {
            keyAlias = keystoreProperties["keyAlias"] as String
            keyPassword = keystoreProperties["keyPassword"] as String
            storeFile = keystoreProperties["storeFile"]?.let { file(it) }
            storePassword = keystoreProperties["storePassword"] as String
        }
    }

    buildTypes {
        release {
            // Troque "debug" por "release"
            signingConfig = signingConfigs.getByName("release")
        }
    }
}
```

<details>
<summary>Projetos antigos com <code>build.gradle</code> (Groovy)</summary>

```groovy
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}

android {
    signingConfigs {
        release {
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
            storeFile keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) : null
            storePassword keystoreProperties['storePassword']
        }
    }
    buildTypes {
        release {
            signingConfig signingConfigs.release
        }
    }
}
```

</details>

## 📦 Passo 6: Gerando o Build Android

### App Bundle (para a Google Play)

```bash
flutter build appbundle
```

Arquivo gerado: `build/app/outputs/bundle/release/app-release.aab`

### APK (para instalar diretamente ou distribuir fora da loja)

```bash
flutter build apk --split-per-abi
```

Arquivos gerados em `build/app/outputs/flutter-apk/`, um para cada tipo de processador (`arm64-v8a`, `armeabi-v7a`, `x86_64`), cada um bem menor que um APK único.

### AAB x APK

| | AAB (App Bundle) | APK |
|---|------------------|-----|
| Google Play | ✅ Obrigatório | ❌ Não aceito para apps novos |
| Instalar direto no celular | ❌ | ✅ |
| Tamanho para o usuário | Otimizado pela loja | Maior |

### Ofuscação (opcional)

Dificulta a engenharia reversa do código Dart:

```bash
flutter build appbundle --obfuscate --split-debug-info=build/debug-info
```

> 💡 Guarde a pasta `build/debug-info`. Ela é necessária para entender relatórios de erro (stack traces) do app ofuscado.

### Instalando o APK no celular conectado

```bash
flutter install
```

## 🤖 Passo 7: Publicando na Google Play

1. **Crie uma conta de desenvolvedor** em [play.google.com/console](https://play.google.com/console). Há uma taxa única de US$ 25.
2. **Crie o app:** "Criar app", informe nome, idioma padrão, se é app ou jogo e se é gratuito ou pago.
3. **Preencha o "Conteúdo do app"** (menu lateral), que é obrigatório:
   - URL da política de privacidade
   - Classificação de conteúdo (questionário)
   - Público-alvo
   - Segurança dos dados (quais dados o app coleta)
4. **Monte a página da loja** (Presença na loja → Versão principal da página):
   - Descrição curta (até 80 caracteres) e completa (até 4.000)
   - Ícone 512×512 px
   - Imagem de destaque 1024×500 px
   - Pelo menos 2 capturas de tela do celular
5. **Envie para teste:** em **Testes → Teste interno**, crie uma versão e faça upload do `.aab`. Adicione os e-mails dos testadores.
6. **Produção:** depois de testar, promova a versão para **Produção** e envie para revisão. A análise costuma levar de algumas horas a alguns dias.

> ⚠️ Contas pessoais criadas recentemente precisam fazer um **teste fechado com um número mínimo de testadores por alguns dias** antes de liberar a produção. Confira as regras atuais no Play Console.

## 🍎 Passo 8: Publicando na App Store

Para publicar para iOS é necessário um **Mac com Xcode** e uma conta no **Apple Developer Program** (US$ 99 por ano).

1. **Registre o Bundle ID** em [developer.apple.com](https://developer.apple.com/account) → Certificates, Identifiers & Profiles → Identifiers.
2. **Crie o app** no [App Store Connect](https://appstoreconnect.apple.com): "Meus apps" → "+" → Novo app, usando o mesmo Bundle ID.
3. **Configure a assinatura no Xcode:** abra `ios/Runner.xcworkspace`, vá em **Runner → Signing & Capabilities**, selecione seu **Team** e marque "Automatically manage signing".
4. **Gere o build:**

   ```bash
   flutter build ipa
   ```

   O arquivo fica em `build/ios/ipa/`.
5. **Envie o build** com o app **Transporter** (da Mac App Store), arrastando o `.ipa`, ou pelo Xcode em **Product → Archive → Distribute App**.
6. **Teste com o TestFlight:** o build aparece na aba TestFlight do App Store Connect. Convide testadores por e-mail.
7. **Envie para revisão:** preencha descrição, palavras-chave, capturas de tela (iPhone 6,7" e 6,5", por exemplo), classificação etária e privacidade, selecione o build e clique em "Enviar para revisão".

> 💡 A revisão da Apple é mais rigorosa: apps que parecem incompletos, que travam ou que pedem permissões sem explicar o motivo costumam ser rejeitados. Preencha sempre os textos de permissão no `Info.plist` (ex.: `NSCameraUsageDescription`).

## 🌐 Passo 9: Publicando para Web

```bash
flutter build web
```

Os arquivos ficam em `build/web/` e podem ser hospedados em qualquer servidor de arquivos estáticos.

### GitHub Pages

Se o site ficar em `https://usuario.github.io/meu_app/`, informe o caminho base:

```bash
flutter build web --base-href /meu_app/
```

Depois publique o conteúdo de `build/web/` em uma branch `gh-pages` ou use um workflow do GitHub Actions (como o que publica este livro).

### Firebase Hosting

```bash
npm install -g firebase-tools
firebase login
firebase init hosting   # Pasta pública: build/web
flutter build web
firebase deploy
```

## 🧪 Ambientes de Desenvolvimento e Produção

É comum o app usar uma API de testes durante o desenvolvimento e a API real em produção. A forma mais simples é passar valores com `--dart-define`:

```dart
// lib/config.dart
class Config {
  static const apiUrl = String.fromEnvironment(
    'API_URL',
    defaultValue: 'https://api-dev.meuapp.com',
  );

  static const ambiente = String.fromEnvironment('AMBIENTE', defaultValue: 'dev');

  static bool get producao => ambiente == 'prod';
}
```

```bash
# Desenvolvimento (usa os valores padrão)
flutter run

# Produção
flutter build appbundle \
  --dart-define=AMBIENTE=prod \
  --dart-define=API_URL=https://api.meuapp.com
```

Com muitas variáveis, use um arquivo JSON, por exemplo `config/prod.json`:

```json
{
  "AMBIENTE": "prod",
  "API_URL": "https://api.meuapp.com"
}
```

```bash
flutter build appbundle --dart-define-from-file=config/prod.json
```

> 💡 Para ter **dois apps instalados lado a lado** (ex.: "MeuApp Dev" e "MeuApp"), com ícones e identificadores diferentes, pesquise sobre **flavors** na documentação oficial.

## 🐛 Problemas Comuns

| Problema | Causa provável | Solução |
|----------|----------------|---------|
| "Version code já foi usado" | Número do build repetido | Aumente o número depois do `+` no `pubspec.yaml` |
| "Você enviou um APK ou AAB assinado no modo de depuração" | `signingConfig` ainda usa `debug` | Configure a assinatura de release (Passo 5) |
| App funciona no debug e trava no release | Falta permissão ou erro só no modo AOT | Teste com `flutter run --release` e verifique o `AndroidManifest.xml` |
| `keytool: command not found` | Java fora do PATH | Use o caminho completo mostrado em `flutter doctor -v` |
| Perdi a chave de upload | Arquivo `.jks` apagado | Com o Play App Signing ativo, solicite a redefinição da chave no Play Console |
| Tela branca no GitHub Pages | `base-href` incorreto | Gere com `--base-href /nome_do_repositorio/` |

## 📖 Recursos Adicionais

- [Documentação oficial: Publicar no Android](https://docs.flutter.dev/deployment/android)
- [Documentação oficial: Publicar no iOS](https://docs.flutter.dev/deployment/ios)
- [Documentação oficial: Publicar para Web](https://docs.flutter.dev/deployment/web)
- [Documentação oficial: Flavors](https://docs.flutter.dev/deployment/flavors)
- [Google Play Console: Central de Ajuda](https://support.google.com/googleplay/android-developer)
- [Pacote flutter_launcher_icons](https://pub.dev/packages/flutter_launcher_icons)
