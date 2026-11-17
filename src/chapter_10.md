# Capítulo 10 - Recursos Avançados

## 📋 Índice

1. [Introdução](#-introdução)
2. [Animações Implícitas](#-animações-implícitas)
3. [Animações Explícitas](#-animações-explícitas)
4. [Hero e Lottie](#-hero-e-lottie)
5. [Temas e Modo Escuro](#-temas-e-modo-escuro)
6. [Layout Responsivo](#-layout-responsivo)
7. [Internacionalização (i18n)](#-internacionalização-i18n)
8. [Performance](#-performance)
9. [Recursos do Aparelho](#-recursos-do-aparelho)
10. [Platform Channels](#-platform-channels)
11. [Arquitetura e Injeção de Dependência](#-arquitetura-e-injeção-de-dependência)
12. [Recursos Adicionais](#-recursos-adicionais)

> 💡 A integração com o **Firebase** (login, banco de dados e notificações) está na seção [10.1 - Firebase](./chapter_10_firebase.md).

## 🚀 Introdução

Você já sabe criar telas, navegar, gerenciar estado, consumir APIs, testar e publicar. Este capítulo reúne recursos que transformam um app que "funciona" em um app **polido, rápido e pronto para crescer**.

Cada seção é independente: leia na ordem que fizer mais sentido para o seu projeto.

## ✨ Animações Implícitas

As animações **implícitas** são as mais fáceis: você muda um valor e o Flutter anima a transição sozinho. São os widgets que começam com `Animated...`.

```dart
import 'package:flutter/material.dart';

class CaixaAnimada extends StatefulWidget {
  const CaixaAnimada({super.key});

  @override
  State<CaixaAnimada> createState() => _CaixaAnimadaState();
}

class _CaixaAnimadaState extends State<CaixaAnimada> {
  bool _expandida = false;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Animações implícitas')),
      body: Center(
        child: GestureDetector(
          onTap: () => setState(() => _expandida = !_expandida),
          child: AnimatedContainer(
            duration: const Duration(milliseconds: 600),
            curve: Curves.easeInOut,
            width: _expandida ? 250 : 100,
            height: _expandida ? 250 : 100,
            decoration: BoxDecoration(
              color: _expandida ? Colors.orange : Colors.blue,
              borderRadius: BorderRadius.circular(_expandida ? 40 : 8),
            ),
            alignment: Alignment.center,
            child: AnimatedOpacity(
              duration: const Duration(milliseconds: 600),
              opacity: _expandida ? 1 : 0,
              child: const Text('Olá! 👋', style: TextStyle(fontSize: 24)),
            ),
          ),
        ),
      ),
    );
  }
}
```

### Widgets implícitos mais usados

| Widget | Anima |
|--------|-------|
| `AnimatedContainer` | Tamanho, cor, borda, margem, padding |
| `AnimatedOpacity` | Transparência |
| `AnimatedAlign` / `AnimatedPositioned` | Posição |
| `AnimatedPadding` | Espaçamento |
| `AnimatedDefaultTextStyle` | Estilo do texto |
| `AnimatedSwitcher` | Troca de um widget por outro |
| `AnimatedCrossFade` | Transição entre dois widgets |
| `TweenAnimationBuilder` | Qualquer valor numérico ou cor |

### AnimatedSwitcher: trocando widgets com animação

```dart
AnimatedSwitcher(
  duration: const Duration(milliseconds: 300),
  transitionBuilder: (child, animation) =>
      ScaleTransition(scale: animation, child: child),
  child: Text(
    '$_contador',
    // A Key avisa que é um widget NOVO, e isso dispara a animação
    key: ValueKey(_contador),
    style: const TextStyle(fontSize: 48),
  ),
)
```

### Conceitos aprendidos:
- `duration` e `curve`: quanto tempo dura e qual o "ritmo" da animação
- Basta mudar o valor no `setState`: o widget interpola do valor antigo para o novo
- No `AnimatedSwitcher`, a `Key` diferente indica ao Flutter que o filho mudou

## 🎬 Animações Explícitas

Use animações **explícitas** quando precisar de controle total: repetir, pausar, inverter ou sincronizar várias animações. Elas usam um `AnimationController` (já visto na [seção 9.1](./chapter_9_splash.md)).

Exemplo: um coração "pulsando" sem parar.

```dart
class CoracaoPulsante extends StatefulWidget {
  const CoracaoPulsante({super.key});

  @override
  State<CoracaoPulsante> createState() => _CoracaoPulsanteState();
}

class _CoracaoPulsanteState extends State<CoracaoPulsante>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller = AnimationController(
    vsync: this,
    duration: const Duration(milliseconds: 800),
  )..repeat(reverse: true); // Vai e volta para sempre

  late final Animation<double> _escala = Tween<double>(begin: 1, end: 1.3).animate(
    CurvedAnimation(parent: _controller, curve: Curves.easeInOut),
  );

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return ScaleTransition(
      scale: _escala,
      child: const Icon(Icons.favorite, color: Colors.red, size: 80),
    );
  }
}
```

### Controlando a animação

```dart
_controller.forward();          // Executa do início ao fim
_controller.reverse();          // Executa de trás para frente
_controller.repeat();           // Repete para sempre
_controller.stop();             // Pausa
_controller.reset();            // Volta ao início
_controller.animateTo(0.5);     // Vai até a metade
```

### AnimatedBuilder: animando qualquer propriedade

```dart
AnimatedBuilder(
  animation: _controller,
  builder: (context, child) {
    return Transform.rotate(
      angle: _controller.value * 2 * 3.1415, // Uma volta completa
      child: child,
    );
  },
  child: const FlutterLogo(size: 80), // Não é reconstruído a cada frame
)
```

### Implícita ou explícita?

| Situação | Use |
|----------|-----|
| Animar quando um valor muda | Implícita |
| Animação que repete ou fica em loop | Explícita |
| Controlar pausa, velocidade ou direção | Explícita |
| Coordenar várias animações em sequência | Explícita |

## 🦸 Hero e Lottie

### Hero: transição entre telas

O `Hero` faz um widget "voar" de uma tela para outra durante a navegação. Basta usar a **mesma `tag`** nas duas telas:

```dart
// Tela de lista
ListTile(
  leading: Hero(
    tag: 'produto-${produto.id}',
    child: Image.network(produto.foto, width: 56),
  ),
  title: Text(produto.nome),
  onTap: () => Navigator.push(
    context,
    MaterialPageRoute(builder: (_) => DetalhePage(produto: produto)),
  ),
)

// Tela de detalhe
Hero(
  tag: 'produto-${produto.id}',
  child: Image.network(produto.foto, width: double.infinity),
)
```

### Lottie: animações prontas de designers

O [Lottie](https://lottiefiles.com) permite usar animações criadas no After Effects, exportadas como JSON. Ótimo para telas de carregamento, sucesso ou lista vazia.

```bash
flutter pub add lottie
```

```dart
import 'package:lottie/lottie.dart';

// De um arquivo local (declare a pasta assets/ no pubspec.yaml)
Lottie.asset('assets/animacoes/sucesso.json', width: 200, repeat: false)

// Direto da internet
Lottie.network('https://exemplo.com/animacao.json')
```

## 🎨 Temas e Modo Escuro

Em vez de definir cores e fontes em cada widget, defina um **tema** central. Assim, mudar a identidade visual do app inteiro é alterar um único lugar.

### Tema com Material 3

```dart
MaterialApp(
  theme: ThemeData(
    colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
    useMaterial3: true,
  ),
  darkTheme: ThemeData(
    colorScheme: ColorScheme.fromSeed(
      seedColor: Colors.indigo,
      brightness: Brightness.dark,
    ),
    useMaterial3: true,
  ),
  themeMode: ThemeMode.system, // Segue a configuração do celular
  home: const HomePage(),
)
```

O `ColorScheme.fromSeed` gera uma paleta completa e harmoniosa a partir de **uma única cor**.

### Usando as cores do tema

```dart
// ❌ Cor fixa: fica errada no modo escuro
Container(color: Colors.white)

// ✅ Cor do tema: se adapta automaticamente
Container(color: Theme.of(context).colorScheme.surface)

Text(
  'Título',
  style: Theme.of(context).textTheme.headlineMedium,
)
```

### Personalizando componentes

```dart
ThemeData(
  colorScheme: ColorScheme.fromSeed(seedColor: Colors.indigo),
  appBarTheme: const AppBarTheme(centerTitle: true),
  elevatedButtonTheme: ElevatedButtonThemeData(
    style: ElevatedButton.styleFrom(
      padding: const EdgeInsets.symmetric(horizontal: 32, vertical: 16),
      shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
    ),
  ),
  inputDecorationTheme: const InputDecorationTheme(
    border: OutlineInputBorder(),
    filled: true,
  ),
)
```

### Botão para trocar o tema (com Provider)

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

class TemaProvider extends ChangeNotifier {
  ThemeMode modo = ThemeMode.system;

  void alternar() {
    modo = modo == ThemeMode.dark ? ThemeMode.light : ThemeMode.dark;
    notifyListeners();
  }
}

void main() {
  runApp(
    ChangeNotifierProvider(
      create: (_) => TemaProvider(),
      child: const MeuApp(),
    ),
  );
}

class MeuApp extends StatelessWidget {
  const MeuApp({super.key});

  @override
  Widget build(BuildContext context) {
    final tema = context.watch<TemaProvider>();

    return MaterialApp(
      theme: ThemeData(colorScheme: ColorScheme.fromSeed(seedColor: Colors.teal)),
      darkTheme: ThemeData(
        colorScheme: ColorScheme.fromSeed(
          seedColor: Colors.teal,
          brightness: Brightness.dark,
        ),
      ),
      themeMode: tema.modo,
      home: Scaffold(
        appBar: AppBar(
          title: const Text('Temas'),
          actions: [
            IconButton(
              icon: Icon(tema.modo == ThemeMode.dark ? Icons.light_mode : Icons.dark_mode),
              onPressed: tema.alternar,
            ),
          ],
        ),
        body: const Center(child: Text('Toque no ícone para trocar o tema')),
      ),
    );
  }
}
```

> 💡 Para lembrar a escolha do usuário ao reabrir o app, salve o modo com `shared_preferences` (seção [8.1](./chapter_8_persistencia.md)).

## 📐 Layout Responsivo

Um app Flutter pode rodar em celulares, tablets, web e desktop. Um layout responsivo se adapta ao espaço disponível.

### MediaQuery: informações da tela

```dart
final tamanho = MediaQuery.sizeOf(context);
final largura = tamanho.width;
final retrato = MediaQuery.orientationOf(context) == Orientation.portrait;
```

### LayoutBuilder: espaço disponível para o widget

```dart
class ProdutosResponsivo extends StatelessWidget {
  const ProdutosResponsivo({super.key});

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) {
        // Define quantas colunas cabem
        final colunas = switch (constraints.maxWidth) {
          < 600 => 2,   // Celular
          < 1000 => 3,  // Tablet
          _ => 5,       // Desktop
        };

        return GridView.builder(
          padding: const EdgeInsets.all(8),
          gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
            crossAxisCount: colunas,
            mainAxisSpacing: 8,
            crossAxisSpacing: 8,
          ),
          itemCount: 30,
          itemBuilder: (_, i) => Card(child: Center(child: Text('Produto $i'))),
        );
      },
    );
  }
}
```

### Layout de lista e detalhe

Em telas largas, mostre a lista e o detalhe lado a lado; em telas estreitas, use navegação:

```dart
LayoutBuilder(
  builder: (context, constraints) {
    if (constraints.maxWidth >= 840) {
      return Row(
        children: [
          SizedBox(width: 320, child: ListaEmails(onSelecionar: _selecionar)),
          const VerticalDivider(width: 1),
          Expanded(child: DetalheEmail(email: _selecionado)),
        ],
      );
    }
    return ListaEmails(
      onSelecionar: (email) => Navigator.push(
        context,
        MaterialPageRoute(builder: (_) => DetalheEmail(email: email)),
      ),
    );
  },
)
```

### Tamanhos de referência (Material Design)

| Largura | Tipo | Navegação sugerida |
|---------|------|--------------------|
| < 600 | Celular | `NavigationBar` (barra inferior) |
| 600 a 840 | Tablet / celular deitado | `NavigationRail` (barra lateral compacta) |
| > 840 | Tablet grande / desktop | `NavigationDrawer` fixo |

> 💡 `MediaQuery.sizeOf(context)` é preferível a `MediaQuery.of(context).size`: reconstrói o widget só quando o tamanho muda, e não a cada mudança de teclado ou brilho.

## 🌍 Internacionalização (i18n)

Internacionalizar é preparar o app para vários idiomas. O Flutter faz isso com arquivos `.arb` e geração de código.

### 1. Dependências

```bash
flutter pub add flutter_localizations --sdk=flutter
flutter pub add intl:any
```

No `pubspec.yaml`, habilite a geração:

```yaml
flutter:
  generate: true
```

### 2. Arquivo de configuração `l10n.yaml` (na raiz do projeto)

```yaml
arb-dir: lib/l10n
template-arb-file: app_pt.arb
output-localization-file: app_localizations.dart
```

### 3. Textos em cada idioma

`lib/l10n/app_pt.arb`:

```json
{
  "titulo": "Minhas Tarefas",
  "ola": "Olá, {nome}!",
  "@ola": {
    "placeholders": { "nome": { "type": "String" } }
  },
  "tarefas": "{quantidade, plural, =0{Nenhuma tarefa} =1{1 tarefa} other{{quantidade} tarefas}}",
  "@tarefas": {
    "placeholders": { "quantidade": { "type": "int" } }
  }
}
```

`lib/l10n/app_en.arb`:

```json
{
  "titulo": "My Tasks",
  "ola": "Hello, {nome}!",
  "tarefas": "{quantidade, plural, =0{No tasks} =1{1 task} other{{quantidade} tasks}}"
}
```

### 4. Gerar o código e configurar o app

```bash
flutter gen-l10n
```

O comando também roda sozinho no `flutter run`.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_localizations/flutter_localizations.dart';
import 'l10n/app_localizations.dart';

void main() => runApp(const MeuApp());

class MeuApp extends StatelessWidget {
  const MeuApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      localizationsDelegates: AppLocalizations.localizationsDelegates,
      supportedLocales: AppLocalizations.supportedLocales,
      home: const HomePage(),
    );
  }
}

class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    final t = AppLocalizations.of(context)!;

    return Scaffold(
      appBar: AppBar(title: Text(t.titulo)),
      body: Column(
        children: [
          Text(t.ola('Maria')),
          Text(t.tarefas(3)), // "3 tarefas" ou "3 tasks"
        ],
      ),
    );
  }
}
```

> 💡 Em versões do Flutter anteriores à 3.32, o código era gerado em um pacote sintético e o import era `package:flutter_gen/gen_l10n/app_localizations.dart`.

### Formatando datas e números

```dart
import 'package:intl/intl.dart';

final preco = NumberFormat.currency(locale: 'pt_BR', symbol: r'R$').format(1234.5);
// R$ 1.234,50

final data = DateFormat('dd/MM/yyyy', 'pt_BR').format(DateTime.now());
// 16/11/2026
```

> ⚠️ Para usar `DateFormat` com nomes de meses em português (ex.: `'d MMMM'`), inicialize os dados do idioma com `initializeDateFormatting('pt_BR')` (de `package:intl/date_symbol_data_local.dart`) no `main`.

## ⚡ Performance

O Flutter é rápido por padrão, mas alguns hábitos fazem diferença em telas complexas.

### 1. Use `const` sempre que possível

```dart
// Um widget const é criado uma única vez e nunca é reconstruído
const Text('Título fixo')
const SizedBox(height: 16)
const Icon(Icons.home)
```

> 💡 Ative a regra `prefer_const_constructors` do `flutter_lints`: o editor avisa onde falta `const`.

### 2. Listas grandes com `.builder`

```dart
// ❌ Cria os 10.000 itens de uma vez
ListView(children: itens.map((i) => ListTile(title: Text(i))).toList())

// ✅ Cria só os itens visíveis na tela
ListView.builder(
  itemCount: itens.length,
  itemBuilder: (_, i) => ListTile(title: Text(itens[i])),
)
```

### 3. Reconstrua só o necessário

Divida telas grandes em widgets menores. Com Provider, use `Selector` ou `context.select` para escutar apenas um campo:

```dart
// Só reconstrói quando "total" mudar, e não a cada mudança no carrinho
final total = context.select<CarrinhoProvider, double>((c) => c.total);
```

### 4. Não faça trabalho pesado no `build`

O `build` pode rodar dezenas de vezes por segundo. Ordenações, filtros e cálculos devem ficar no estado (Provider, BLoC ou `initState`). Para processamento muito pesado (ex.: decodificar um JSON gigante), use um *isolate*:

```dart
final dados = await Isolate.run(() => jsonDecode(textoGigante));
```

### 5. Imagens no tamanho certo

```dart
Image.network(
  url,
  cacheWidth: 300, // Decodifica na resolução exibida, economizando memória
)
```

Para cache em disco de imagens da internet, use o pacote [`cached_network_image`](https://pub.dev/packages/cached_network_image).

### Ferramentas de diagnóstico

- **Flutter DevTools:** abra pelo VS Code ("Open DevTools") para ver o desempenho de cada frame, a memória e a árvore de widgets.
- **Modo profile:** meça a performance sempre com `flutter run --profile` em um celular real. O modo debug é muito mais lento e engana.
- **Performance overlay:** `MaterialApp(showPerformanceOverlay: true)` mostra gráficos de frames na tela.

## 📷 Recursos do Aparelho

### Câmera e galeria

```bash
flutter pub add image_picker
```

```dart
import 'dart:io';
import 'package:image_picker/image_picker.dart';

final picker = ImagePicker();

Future<File?> escolherFoto({bool camera = false}) async {
  final XFile? arquivo = await picker.pickImage(
    source: camera ? ImageSource.camera : ImageSource.gallery,
    maxWidth: 1080,      // Reduz o tamanho da imagem
    imageQuality: 85,
  );
  return arquivo == null ? null : File(arquivo.path);
}

// Exibindo
if (_foto != null) Image.file(_foto!, height: 200)
```

No iOS, adicione ao `ios/Runner/Info.plist` os textos que explicam o uso:

```xml
<key>NSCameraUsageDescription</key>
<string>Usamos a câmera para você tirar a foto do perfil.</string>
<key>NSPhotoLibraryUsageDescription</key>
<string>Usamos a galeria para você escolher a foto do perfil.</string>
```

### Localização (GPS)

```bash
flutter pub add geolocator
```

```dart
import 'package:geolocator/geolocator.dart';

Future<Position> obterLocalizacao() async {
  if (!await Geolocator.isLocationServiceEnabled()) {
    throw Exception('Ative a localização do aparelho');
  }

  var permissao = await Geolocator.checkPermission();
  if (permissao == LocationPermission.denied) {
    permissao = await Geolocator.requestPermission();
  }
  if (permissao == LocationPermission.denied ||
      permissao == LocationPermission.deniedForever) {
    throw Exception('Permissão de localização negada');
  }

  return Geolocator.getCurrentPosition();
}
```

No Android, adicione ao `AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
```

No iOS, adicione `NSLocationWhenInUseUsageDescription` ao `Info.plist`.

### Abrindo links, e-mail e telefone

```bash
flutter pub add url_launcher
```

```dart
import 'package:url_launcher/url_launcher.dart';

await launchUrl(Uri.parse('https://flutter.dev'));
await launchUrl(Uri.parse('tel:+5511999999999'));
await launchUrl(Uri.parse('mailto:contato@exemplo.com?subject=Olá'));
await launchUrl(Uri.parse('https://wa.me/5511999999999'));
```

### Boas práticas com permissões

- Peça a permissão **no momento do uso**, não ao abrir o app.
- Explique antes **por que** precisa dela.
- Trate a recusa: o app deve continuar funcionando, só sem aquele recurso.

## 🔌 Platform Channels

Às vezes você precisa de algo que só existe no código nativo (Kotlin/Java no Android, Swift no iOS) e que nenhum pacote oferece. Os **platform channels** permitem que o Dart chame esse código.

Exemplo: obter o nível da bateria.

### Lado Dart

```dart
import 'package:flutter/services.dart';

class Bateria {
  static const _canal = MethodChannel('br.com.meuapp/bateria');

  static Future<int?> nivel() async {
    try {
      return await _canal.invokeMethod<int>('obterNivel');
    } on PlatformException catch (e) {
      print('Erro: ${e.message}');
      return null;
    }
  }
}
```

### Lado Android (`android/app/src/main/kotlin/.../MainActivity.kt`)

```kotlin
import android.content.Context
import android.os.BatteryManager
import io.flutter.embedding.android.FlutterActivity
import io.flutter.embedding.engine.FlutterEngine
import io.flutter.plugin.common.MethodChannel

class MainActivity : FlutterActivity() {
    private val canal = "br.com.meuapp/bateria"

    override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
        super.configureFlutterEngine(flutterEngine)

        MethodChannel(flutterEngine.dartExecutor.binaryMessenger, canal)
            .setMethodCallHandler { chamada, resultado ->
                if (chamada.method == "obterNivel") {
                    val bm = getSystemService(Context.BATTERY_SERVICE) as BatteryManager
                    val nivel = bm.getIntProperty(BatteryManager.BATTERY_PROPERTY_CAPACITY)
                    resultado.success(nivel)
                } else {
                    resultado.notImplemented()
                }
            }
    }
}
```

### Conceitos aprendidos:
- O **nome do canal** deve ser idêntico nos dois lados
- `invokeMethod`: chama um método nativo pelo nome e espera a resposta
- `resultado.success()` / `resultado.error()` / `resultado.notImplemented()`: as respostas possíveis do lado nativo

> 💡 Antes de escrever código nativo, procure no [pub.dev](https://pub.dev): para bateria, por exemplo, já existe o pacote `battery_plus`.

## 🏛 Arquitetura e Injeção de Dependência

Conforme o app cresce, organizar o código se torna essencial. Uma estrutura muito usada é a **organização por funcionalidade** (feature-first), com camadas dentro de cada uma:

```
lib/
├── core/                    # Código compartilhado
│   ├── network/api_client.dart
│   ├── theme/app_theme.dart
│   └── di/injecao.dart
├── features/
│   ├── auth/
│   │   ├── data/            # Acesso a dados: repositórios, APIs, modelos
│   │   ├── domain/          # Regras de negócio (entidades, casos de uso)
│   │   └── presentation/    # Telas, widgets, BLoCs/Providers
│   └── tarefas/
│       ├── data/
│       ├── domain/
│       └── presentation/
└── main.dart
```

### A regra principal

As camadas de dentro **não conhecem** as de fora:

```
presentation  →  domain  ←  data
  (telas)        (regras)     (API, banco)
```

A tela pede ao domínio "liste as tarefas"; o domínio não sabe se elas vêm da internet ou do SQLite. Isso permite trocar a fonte de dados e testar cada parte isoladamente (Capítulo 7).

### Injeção de dependência com get_it

Em vez de cada classe criar as próprias dependências (`PostRepository(ApiClient())`), registramos tudo em um lugar central:

```bash
flutter pub add get_it
```

```dart
// lib/core/di/injecao.dart
import 'package:get_it/get_it.dart';

final getIt = GetIt.instance;

void configurarDependencias() {
  // Uma única instância para o app inteiro
  getIt.registerLazySingleton<ApiClient>(() => ApiClient());

  // O repositório recebe o ApiClient já registrado
  getIt.registerLazySingleton<PostRepository>(
    () => PostRepository(getIt<ApiClient>()),
  );

  // BLoCs: uma instância nova a cada vez que for pedida
  getIt.registerFactory<PostsBloc>(() => PostsBloc(getIt<PostRepository>()));
}
```

```dart
void main() {
  configurarDependencias();
  runApp(const MeuApp());
}

// Em qualquer lugar do app
BlocProvider(
  create: (_) => getIt<PostsBloc>()..add(PostsCarregados()),
  child: const PostsView(),
)
```

### Tipos de registro

| Método | Comportamento |
|--------|---------------|
| `registerSingleton` | Cria agora; sempre devolve a mesma instância |
| `registerLazySingleton` | Cria no primeiro uso; depois devolve a mesma |
| `registerFactory` | Cria uma instância nova a cada chamada |

> 💡 Nos testes, basta registrar mocks no lugar das classes reais: `getIt.registerSingleton<PostRepository>(MockPostRepository())`.

### Quando usar essa estrutura?

- **Apps pequenos ou de estudo:** a estrutura simples do Capítulo 8 é suficiente.
- **Apps com várias telas, equipe ou longa duração:** organize por funcionalidade e use injeção de dependência desde cedo.

## 📖 Recursos Adicionais

- [Documentação oficial: Animações](https://docs.flutter.dev/ui/animations)
- [Documentação oficial: Temas](https://docs.flutter.dev/cookbook/design/themes)
- [Documentação oficial: Layout adaptativo e responsivo](https://docs.flutter.dev/ui/adaptive-responsive)
- [Documentação oficial: Internacionalização](https://docs.flutter.dev/ui/accessibility-and-internationalization/internationalization)
- [Documentação oficial: Boas práticas de performance](https://docs.flutter.dev/perf/best-practices)
- [Documentação oficial: Platform channels](https://docs.flutter.dev/platform-integration/platform-channels)
- [Guia de arquitetura de apps Flutter](https://docs.flutter.dev/app-architecture)
- [LottieFiles: animações gratuitas](https://lottiefiles.com)
