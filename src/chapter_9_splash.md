# 9.1 - Splash Screen em Flutter

## 📋 Índice

1. [Introdução](#-introdução)
2. [Splash Nativa x Splash em Flutter](#-splash-nativa-x-splash-em-flutter)
3. [Método 1: flutter_native_splash](#-método-1-flutter_native_splash)
4. [Mantendo a Splash Durante a Inicialização](#-mantendo-a-splash-durante-a-inicialização)
5. [Método 2: Configuração Nativa Manual](#-método-2-configuração-nativa-manual)
6. [Método 3: Splash Animada em Flutter](#-método-3-splash-animada-em-flutter)
7. [Exemplo Completo: Nativa + Animada](#-exemplo-completo-nativa--animada)
8. [Melhores Práticas](#-melhores-práticas)
9. [Problemas Comuns](#-problemas-comuns)

## 🚀 Introdução

A **splash screen** (tela de abertura) é a primeira coisa que o usuário vê ao abrir o app. Ela aparece enquanto o sistema operacional carrega o motor do Flutter e enquanto o app prepara o que precisa para funcionar (configurações, login salvo, banco de dados etc.).

Sem uma splash configurada, o usuário vê uma **tela branca** (ou preta) por alguns instantes, o que passa a impressão de app lento ou mal acabado.

### O que você vai aprender

- A diferença entre splash **nativa** e splash **feita em Flutter**
- Gerar a splash nativa automaticamente com o pacote `flutter_native_splash`
- Manter a splash visível enquanto o app inicializa
- Criar uma splash animada com `AnimationController`
- Combinar as duas abordagens para uma transição suave

## 🔍 Splash Nativa x Splash em Flutter

| Característica | Splash Nativa | Splash em Flutter |
|----------------|---------------|-------------------|
| Quando aparece | Imediatamente, antes do Flutter carregar | Só depois que o Flutter já carregou |
| Onde é configurada | Arquivos Android (XML) e iOS (Storyboard) | Código Dart (um widget comum) |
| Animações | Muito limitadas | Qualquer animação |
| Evita a tela branca | ✅ Sim | ❌ Não, sozinha |

> 💡 **Regra prática:** use **sempre** a splash nativa. Se quiser animação, adicione uma splash em Flutter **depois** dela, com o mesmo fundo, para que a troca seja imperceptível.

## 📦 Método 1: flutter_native_splash

O pacote [`flutter_native_splash`](https://pub.dev/packages/flutter_native_splash) gera todos os arquivos nativos (Android, iOS e Web) a partir de uma configuração simples.

### Passo 1: Adicionar a dependência

```bash
flutter pub add flutter_native_splash
```

### Passo 2: Adicionar a imagem

Coloque o logo em `assets/splash/logo.png`. Prefira um PNG com fundo transparente e pelo menos **1152×1152 px**.

```
meu_app/
├── assets/
│   └── splash/
│       ├── logo.png
│       └── logo_dark.png      (opcional, para modo escuro)
├── lib/
└── pubspec.yaml
```

### Passo 3: Configurar no `pubspec.yaml`

Adicione este bloco no final do `pubspec.yaml` (no mesmo nível de `flutter:`, sem indentação):

```yaml
flutter_native_splash:
  # Cor de fundo e imagem central
  color: "#1E88E5"
  image: assets/splash/logo.png

  # Versões para o modo escuro (opcional)
  color_dark: "#121212"
  image_dark: assets/splash/logo_dark.png

  # Android 12+ usa uma API própria de splash
  android_12:
    color: "#1E88E5"
    image: assets/splash/logo.png
    color_dark: "#121212"
    image_dark: assets/splash/logo_dark.png

  # Plataformas
  android: true
  ios: true
  web: false

  # Esconde a barra de status durante a splash
  fullscreen: true
```

### Passo 4: Gerar os arquivos nativos

```bash
dart run flutter_native_splash:create
```

Pronto! Rode o app com `flutter run` e a splash aparecerá imediatamente ao abrir.

> ⚠️ Sempre que mudar a configuração, rode o comando `create` de novo. A splash nativa **não** é atualizada pelo Hot Reload: é preciso parar o app e executá-lo novamente.

Para remover a splash gerada e voltar ao padrão:

```bash
dart run flutter_native_splash:remove
```

### Sobre o Android 12+

A partir do Android 12, o sistema exibe a splash com o **ícone recortado em círculo**. Por isso:

- A imagem deve ter **1152×1152 px**, com o conteúdo cabendo em um círculo de **768 px** de diâmetro no centro.
- O que ficar fora desse círculo será cortado.
- Use a seção `android_12` para definir uma imagem específica, se necessário.

## ⏳ Mantendo a Splash Durante a Inicialização

Por padrão, a splash nativa some assim que o Flutter desenha o primeiro frame. Se o app precisa carregar dados antes de mostrar a primeira tela, você pode **segurar** a splash até terminar:

```dart
import 'package:flutter/material.dart';
import 'package:flutter_native_splash/flutter_native_splash.dart';

void main() async {
  // Garante que o Flutter está pronto e guarda a referência do binding
  final widgetsBinding = WidgetsFlutterBinding.ensureInitialized();

  // Mantém a splash nativa na tela
  FlutterNativeSplash.preserve(widgetsBinding: widgetsBinding);

  // Faz a inicialização necessária
  await inicializarApp();

  runApp(const MeuApp());

  // Remove a splash: o app já está pronto
  FlutterNativeSplash.remove();
}

Future<void> inicializarApp() async {
  // Exemplos: carregar preferências, abrir banco, verificar login...
  await Future.delayed(const Duration(seconds: 2));
}

class MeuApp extends StatelessWidget {
  const MeuApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        body: Center(child: Text('Bem-vindo! 🎉')),
      ),
    );
  }
}
```

### Conceitos aprendidos:
- `WidgetsFlutterBinding.ensureInitialized()`: necessário antes de usar `await` no `main`
- `FlutterNativeSplash.preserve()`: impede que a splash suma no primeiro frame
- `FlutterNativeSplash.remove()`: esconde a splash quando o app estiver pronto

> ⚠️ Não esqueça de chamar `remove()`. Se a inicialização lançar uma exceção antes dele, o app fica "preso" na splash. Use `try/finally` em inicializações que podem falhar.

## 🛠 Método 2: Configuração Nativa Manual

Se você preferir não usar pacotes, é possível configurar a splash diretamente nos arquivos nativos. Este é exatamente o trabalho que o `flutter_native_splash` automatiza.

### Android

Edite `android/app/src/main/res/drawable/launch_background.xml` (e também `drawable-v21/launch_background.xml`, se existir):

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- Cor de fundo -->
    <item android:drawable="@color/splash_background" />

    <!-- Logo centralizado -->
    <item>
        <bitmap
            android:gravity="center"
            android:src="@drawable/logo" />
    </item>
</layer-list>
```

Defina a cor em `android/app/src/main/res/values/colors.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="splash_background">#1E88E5</color>
</resources>
```

E coloque o arquivo `logo.png` em `android/app/src/main/res/drawable/`.

### iOS

1. Abra `ios/Runner.xcworkspace` no Xcode.
2. Adicione a imagem em `Runner/Assets.xcassets` (por exemplo, com o nome `LaunchImage`).
3. Edite `Runner/LaunchScreen.storyboard`: defina a cor de fundo da view e centralize um `UIImageView` com a imagem.

> 💡 A configuração manual dá mais controle, mas precisa ser refeita para cada plataforma, para o modo escuro e para o Android 12. Para a maioria dos projetos, o Método 1 é mais simples e seguro.

## 🎨 Método 3: Splash Animada em Flutter

Uma splash feita em Flutter é apenas **uma tela comum** que aparece primeiro e, depois de alguns segundos (ou ao terminar de carregar algo), navega para a tela principal.

### Versão simples (com Timer)

```dart
import 'dart:async';
import 'package:flutter/material.dart';

void main() => runApp(const MeuApp());

class MeuApp extends StatelessWidget {
  const MeuApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: SplashPage(),
    );
  }
}

class SplashPage extends StatefulWidget {
  const SplashPage({super.key});

  @override
  State<SplashPage> createState() => _SplashPageState();
}

class _SplashPageState extends State<SplashPage> {
  @override
  void initState() {
    super.initState();
    // Após 3 segundos, vai para a Home e remove a splash da pilha
    Timer(const Duration(seconds: 3), () {
      Navigator.of(context).pushReplacement(
        MaterialPageRoute(builder: (_) => const HomePage()),
      );
    });
  }

  @override
  Widget build(BuildContext context) {
    return const Scaffold(
      backgroundColor: Color(0xFF1E88E5),
      body: Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            FlutterLogo(size: 100),
            SizedBox(height: 24),
            CircularProgressIndicator(color: Colors.white),
          ],
        ),
      ),
    );
  }
}

class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Home')),
      body: const Center(child: Text('Tela principal')),
    );
  }
}
```

### Conceitos aprendidos:
- `Timer`: executa uma ação depois de um tempo
- `pushReplacement`: troca a tela atual pela nova, assim o botão "voltar" não retorna para a splash
- A splash é um `StatefulWidget` porque usa `initState` para agendar a navegação

### Versão animada (fade + escala)

Agora vamos animar o logo, aparecendo suavemente e crescendo:

```dart
import 'package:flutter/material.dart';

class SplashAnimadaPage extends StatefulWidget {
  const SplashAnimadaPage({super.key});

  @override
  State<SplashAnimadaPage> createState() => _SplashAnimadaPageState();
}

class _SplashAnimadaPageState extends State<SplashAnimadaPage>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  late final Animation<double> _opacidade;
  late final Animation<double> _escala;

  @override
  void initState() {
    super.initState();

    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 1500),
    );

    _opacidade = CurvedAnimation(
      parent: _controller,
      curve: Curves.easeIn,
    );

    _escala = Tween<double>(begin: 0.5, end: 1.0).animate(
      CurvedAnimation(parent: _controller, curve: Curves.elasticOut),
    );

    _controller.forward();

    // Quando a animação terminar, navega para a Home
    _controller.addStatusListener((status) {
      if (status == AnimationStatus.completed) {
        _irParaHome();
      }
    });
  }

  void _irParaHome() {
    if (!mounted) return;
    Navigator.of(context).pushReplacement(
      PageRouteBuilder(
        transitionDuration: const Duration(milliseconds: 500),
        pageBuilder: (_, __, ___) => const HomePage(),
        transitionsBuilder: (_, animation, __, child) =>
            FadeTransition(opacity: animation, child: child),
      ),
    );
  }

  @override
  void dispose() {
    _controller.dispose(); // Sempre libere o controller!
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFF1E88E5),
      body: Center(
        child: FadeTransition(
          opacity: _opacidade,
          child: ScaleTransition(
            scale: _escala,
            child: const Column(
              mainAxisSize: MainAxisSize.min,
              children: [
                FlutterLogo(size: 120),
                SizedBox(height: 16),
                Text(
                  'Meu App',
                  style: TextStyle(
                    color: Colors.white,
                    fontSize: 28,
                    fontWeight: FontWeight.bold,
                  ),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}

class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Home')),
      body: const Center(child: Text('Tela principal')),
    );
  }
}
```

### Conceitos aprendidos:
- `SingleTickerProviderStateMixin`: fornece o `vsync` para sincronizar a animação com a tela
- `AnimationController`: controla a duração e o progresso (de 0.0 a 1.0)
- `CurvedAnimation` e `Tween`: definem a curva e o intervalo de valores
- `FadeTransition` e `ScaleTransition`: aplicam opacidade e escala ao widget filho
- `PageRouteBuilder`: permite personalizar a transição para a próxima tela
- `mounted`: evita navegar se a tela já tiver sido descartada

## 🧩 Exemplo Completo: Nativa + Animada

Na prática, a melhor experiência combina as duas abordagens:

1. A **splash nativa** aparece instantaneamente (sem tela branca).
2. Ela é removida assim que o Flutter desenha a **splash animada**, que tem o **mesmo fundo**, então o usuário não percebe a troca.
3. A splash animada roda a animação **enquanto** o app carrega os dados.
4. Só navega quando a animação **e** o carregamento terminaram.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_native_splash/flutter_native_splash.dart';

const corSplash = Color(0xFF1E88E5); // Mesma cor do pubspec.yaml

void main() {
  final binding = WidgetsFlutterBinding.ensureInitialized();
  FlutterNativeSplash.preserve(widgetsBinding: binding);
  runApp(const MeuApp());
}

class MeuApp extends StatelessWidget {
  const MeuApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: SplashPage(),
    );
  }
}

class SplashPage extends StatefulWidget {
  const SplashPage({super.key});

  @override
  State<SplashPage> createState() => _SplashPageState();
}

class _SplashPageState extends State<SplashPage>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller = AnimationController(
    vsync: this,
    duration: const Duration(milliseconds: 1200),
  );

  @override
  void initState() {
    super.initState();
    // A splash em Flutter já está pronta: remove a nativa e anima
    FlutterNativeSplash.remove();
    _controller.forward();
    _iniciar();
  }

  Future<void> _iniciar() async {
    // Espera o carregamento E um tempo mínimo de exibição
    final resultados = await Future.wait([
      _carregarDados(),
      Future.delayed(const Duration(milliseconds: 1500)),
    ]);
    final usuarioLogado = resultados.first as bool;

    if (!mounted) return;
    Navigator.of(context).pushReplacement(
      MaterialPageRoute(
        builder: (_) => usuarioLogado ? const HomePage() : const LoginPage(),
      ),
    );
  }

  /// Simula a verificação de login salvo (ex.: SharedPreferences)
  Future<bool> _carregarDados() async {
    await Future.delayed(const Duration(seconds: 1));
    return false;
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: corSplash,
      body: Center(
        child: ScaleTransition(
          scale: CurvedAnimation(
            parent: _controller,
            curve: Curves.easeOutBack,
          ),
          child: const FlutterLogo(size: 120),
        ),
      ),
    );
  }
}

class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Home')),
        body: const Center(child: Text('Usuário logado')),
      );
}

class LoginPage extends StatelessWidget {
  const LoginPage({super.key});

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Login')),
        body: const Center(child: Text('Faça login para continuar')),
      );
}
```

### Conceitos aprendidos:
- `Future.wait`: espera várias tarefas em paralelo; a splash dura o tempo da **mais lenta**
- Tempo mínimo de exibição: evita que a splash "pisque" em aparelhos rápidos
- Decidir a rota inicial (Home ou Login) é um uso clássico da splash

## ✅ Melhores Práticas

- **Seja rápido:** a splash não é propaganda. Mostre-a só pelo tempo necessário (idealmente menos de 2 a 3 segundos).
- **Mesmo visual:** use a mesma cor de fundo e o mesmo logo na splash nativa e na splash em Flutter.
- **Modo escuro:** configure `color_dark` e `image_dark` para não "cegar" quem usa tema escuro.
- **Use `pushReplacement`:** o usuário não deve conseguir voltar para a splash.
- **Trate erros:** se a inicialização falhar, mostre uma mensagem ou siga para uma tela de erro, mas nunca deixe o app travado na splash.
- **Teste em dispositivo real:** a splash nativa não aparece no Hot Reload, e o Android 12+ tem comportamento próprio.

## 🐛 Problemas Comuns

| Problema | Causa provável | Solução |
|----------|----------------|---------|
| Ainda aparece tela branca | Splash nativa não configurada ou arquivos não regenerados | Rode `dart run flutter_native_splash:create` e reinstale o app |
| Logo cortado no Android 12+ | Imagem maior que o círculo de 768 px | Reduza o conteúdo ou use uma imagem específica em `android_12` |
| App fica preso na splash | `FlutterNativeSplash.remove()` nunca foi chamado | Chame `remove()` em um bloco `finally` |
| Erro `setState() called after dispose()` | Navegação ou `setState` após sair da tela | Verifique `if (!mounted) return;` antes de navegar |
| Botão voltar retorna à splash | Uso de `push` em vez de `pushReplacement` | Troque por `pushReplacement` |

## 📖 Recursos Adicionais

- [Pacote flutter_native_splash (pub.dev)](https://pub.dev/packages/flutter_native_splash)
- [Documentação oficial: Adicionar uma splash screen](https://docs.flutter.dev/platform-integration/android/splash-screen)
- [Android: API de Splash Screens](https://developer.android.com/develop/ui/views/launch/splash-screen)
- [Flutter: Animações (introdução)](https://docs.flutter.dev/ui/animations)
