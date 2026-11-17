# 8.1 - Persistência Local Simples

## 📋 Índice

1. [Introdução](#-introdução)
2. [Qual Opção Escolher?](#-qual-opção-escolher)
3. [Passo 1: shared_preferences](#-passo-1-shared_preferences)
4. [Passo 2: Salvando Objetos e Listas](#-passo-2-salvando-objetos-e-listas)
5. [Passo 3: Lembrando o Tema Escolhido](#-passo-3-lembrando-o-tema-escolhido)
6. [Passo 4: Tela de Boas-Vindas Só na Primeira Vez](#-passo-4-tela-de-boas-vindas-só-na-primeira-vez)
7. [Passo 5: Dados Sensíveis com flutter_secure_storage](#-passo-5-dados-sensíveis-com-flutter_secure_storage)
8. [Passo 6: Arquivos com path_provider](#-passo-6-arquivos-com-path_provider)
9. [Melhores Práticas](#-melhores-práticas)
10. [Recursos Adicionais](#-recursos-adicionais)

## 🚀 Introdução

Ao fechar o app, tudo o que está na memória (variáveis, estado do Provider ou do BLoC) é perdido. Para **lembrar** informações entre uma execução e outra, precisamos salvá-las no armazenamento do aparelho.

No Capítulo 8 usamos o **SQLite**, ideal para dados estruturados com muitos registros. Mas para guardar coisas simples, como "o usuário prefere o modo escuro" ou "já viu a tela de boas-vindas", existem opções bem mais práticas.

## 🤔 Qual Opção Escolher?

| Necessidade | Solução | Exemplo |
|-------------|---------|---------|
| Configurações e valores simples | `shared_preferences` | Tema, idioma, "não mostrar novamente" |
| Senhas, tokens e dados sensíveis | `flutter_secure_storage` | Token de login |
| Muitos registros com consultas | SQLite (`sqflite`, Capítulo 8) | Lista de clientes, histórico |
| Arquivos (textos, imagens, PDFs) | `path_provider` + `dart:io` | Foto de perfil, backup em JSON |
| Dados sincronizados na nuvem | Firestore ([seção 10.1](./chapter_10_firebase.md)) | Tarefas em vários aparelhos |

## 🔑 Passo 1: shared_preferences

O `shared_preferences` funciona como um dicionário **chave → valor** salvo no aparelho.

```bash
flutter pub add shared_preferences
```

### Operações básicas

```dart
import 'package:shared_preferences/shared_preferences.dart';

Future<void> exemplos() async {
  final prefs = SharedPreferencesAsync();

  // Salvar
  await prefs.setString('nome', 'Maria');
  await prefs.setInt('contador', 10);
  await prefs.setBool('modoEscuro', true);
  await prefs.setDouble('volume', 0.8);
  await prefs.setStringList('buscas', ['flutter', 'dart']);

  // Ler (retorna null se a chave não existir)
  final nome = await prefs.getString('nome');            // "Maria"
  final contador = await prefs.getInt('contador') ?? 0;  // 10
  final escuro = await prefs.getBool('modoEscuro') ?? false;

  // Remover uma chave ou tudo
  await prefs.remove('contador');
  await prefs.clear();
}
```

### Conceitos aprendidos:
- Tipos suportados: `String`, `int`, `double`, `bool` e `List<String>`
- `?? valorPadrao`: define o valor quando a chave ainda não existe
- Todas as operações são assíncronas (`await`)

> 💡 O pacote oferece duas APIs: `SharedPreferencesAsync` (recomendada para código novo, sempre lê do disco) e `SharedPreferencesWithCache` (mantém uma cópia em memória para leituras síncronas). A antiga `SharedPreferences.getInstance()` ainda funciona, mas tende a ser descontinuada.

### Exemplo: contador que não zera ao fechar o app

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

void main() => runApp(const MaterialApp(home: ContadorPersistente()));

class ContadorPersistente extends StatefulWidget {
  const ContadorPersistente({super.key});

  @override
  State<ContadorPersistente> createState() => _ContadorPersistenteState();
}

class _ContadorPersistenteState extends State<ContadorPersistente> {
  final _prefs = SharedPreferencesAsync();
  int _contador = 0;

  @override
  void initState() {
    super.initState();
    _carregar();
  }

  Future<void> _carregar() async {
    final valor = await _prefs.getInt('contador') ?? 0;
    setState(() => _contador = valor);
  }

  Future<void> _incrementar() async {
    setState(() => _contador++);
    await _prefs.setInt('contador', _contador);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Contador persistente')),
      body: Center(
        child: Text('$_contador', style: Theme.of(context).textTheme.displayLarge),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementar,
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

Feche o app completamente e abra de novo: o número continua lá. 🎉

## 📦 Passo 2: Salvando Objetos e Listas

O `shared_preferences` não salva objetos diretamente, mas podemos convertê-los em **JSON** (como no Capítulo 6):

```dart
import 'dart:convert';
import 'package:shared_preferences/shared_preferences.dart';

class Configuracoes {
  final String idioma;
  final bool notificacoes;
  final double tamanhoFonte;

  const Configuracoes({
    this.idioma = 'pt',
    this.notificacoes = true,
    this.tamanhoFonte = 16,
  });

  Map<String, dynamic> toJson() => {
        'idioma': idioma,
        'notificacoes': notificacoes,
        'tamanhoFonte': tamanhoFonte,
      };

  factory Configuracoes.fromJson(Map<String, dynamic> json) => Configuracoes(
        idioma: json['idioma'] ?? 'pt',
        notificacoes: json['notificacoes'] ?? true,
        tamanhoFonte: (json['tamanhoFonte'] as num?)?.toDouble() ?? 16,
      );
}

class ConfiguracoesRepository {
  static const _chave = 'configuracoes';
  final _prefs = SharedPreferencesAsync();

  Future<void> salvar(Configuracoes config) =>
      _prefs.setString(_chave, jsonEncode(config.toJson()));

  Future<Configuracoes> carregar() async {
    final texto = await _prefs.getString(_chave);
    if (texto == null) return const Configuracoes(); // Valores padrão
    return Configuracoes.fromJson(jsonDecode(texto));
  }
}
```

### Exemplo: histórico das últimas buscas

```dart
class HistoricoBuscas {
  static const _chave = 'historico_buscas';
  static const _limite = 5;
  final _prefs = SharedPreferencesAsync();

  Future<List<String>> listar() async =>
      await _prefs.getStringList(_chave) ?? [];

  Future<void> adicionar(String termo) async {
    final lista = await listar();
    lista.remove(termo);       // Evita duplicados
    lista.insert(0, termo);    // Mais recente primeiro
    await _prefs.setStringList(_chave, lista.take(_limite).toList());
  }
}
```

> 💡 Use esse histórico para resolver o desafio do Buscador de CEP do Capítulo 6!

## 🌗 Passo 3: Lembrando o Tema Escolhido

Vamos melhorar o `TemaProvider` do Capítulo 10 para que a escolha do usuário sobreviva ao fechamento do app:

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'package:shared_preferences/shared_preferences.dart';

class TemaProvider extends ChangeNotifier {
  static const _chave = 'tema';
  final _prefs = SharedPreferencesAsync();

  ThemeMode modo = ThemeMode.system;

  Future<void> carregar() async {
    final salvo = await _prefs.getString(_chave);
    modo = ThemeMode.values.firstWhere(
      (m) => m.name == salvo,
      orElse: () => ThemeMode.system,
    );
    notifyListeners();
  }

  Future<void> definir(ThemeMode novo) async {
    modo = novo;
    notifyListeners();
    await _prefs.setString(_chave, novo.name); // "light", "dark" ou "system"
  }
}

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  final tema = TemaProvider();
  await tema.carregar(); // Carrega ANTES de desenhar, para não "piscar" o tema errado

  runApp(ChangeNotifierProvider.value(value: tema, child: const MeuApp()));
}

class MeuApp extends StatelessWidget {
  const MeuApp({super.key});

  @override
  Widget build(BuildContext context) {
    final tema = context.watch<TemaProvider>();
    return MaterialApp(
      theme: ThemeData(colorScheme: ColorScheme.fromSeed(seedColor: Colors.teal)),
      darkTheme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.teal, brightness: Brightness.dark),
      ),
      themeMode: tema.modo,
      home: const ConfiguracoesPage(),
    );
  }
}

class ConfiguracoesPage extends StatelessWidget {
  const ConfiguracoesPage({super.key});

  @override
  Widget build(BuildContext context) {
    final tema = context.watch<TemaProvider>();
    return Scaffold(
      appBar: AppBar(title: const Text('Configurações')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: SegmentedButton<ThemeMode>(
          segments: const [
            ButtonSegment(value: ThemeMode.light, icon: Icon(Icons.light_mode), label: Text('Claro')),
            ButtonSegment(value: ThemeMode.system, icon: Icon(Icons.phone_android), label: Text('Sistema')),
            ButtonSegment(value: ThemeMode.dark, icon: Icon(Icons.dark_mode), label: Text('Escuro')),
          ],
          selected: {tema.modo},
          onSelectionChanged: (s) => tema.definir(s.first),
        ),
      ),
    );
  }
}
```

### Conceitos aprendidos:
- `ThemeMode.name`: converte o enum em texto para salvar
- `ThemeMode.values.firstWhere`: converte o texto de volta em enum
- Carregar no `main` (antes do `runApp`) evita que o app abra com o tema errado
- `ChangeNotifierProvider.value`: usa uma instância já criada

## 👋 Passo 4: Tela de Boas-Vindas Só na Primeira Vez

Um uso clássico: mostrar a apresentação do app (*onboarding*) apenas na primeira execução.

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  final jaViu = await SharedPreferencesAsync().getBool('viu_onboarding') ?? false;

  runApp(MaterialApp(
    home: jaViu ? const HomePage() : const OnboardingPage(),
  ));
}

class OnboardingPage extends StatelessWidget {
  const OnboardingPage({super.key});

  Future<void> _concluir(BuildContext context) async {
    await SharedPreferencesAsync().setBool('viu_onboarding', true);
    if (!context.mounted) return;
    Navigator.of(context).pushReplacement(
      MaterialPageRoute(builder: (_) => const HomePage()),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(
        child: Padding(
          padding: const EdgeInsets.all(24),
          child: Column(
            children: [
              const Spacer(),
              const Icon(Icons.rocket_launch, size: 120, color: Colors.indigo),
              const SizedBox(height: 24),
              Text('Bem-vindo!', style: Theme.of(context).textTheme.headlineLarge),
              const SizedBox(height: 8),
              const Text(
                'Organize suas tarefas de forma simples e rápida.',
                textAlign: TextAlign.center,
              ),
              const Spacer(),
              SizedBox(
                width: double.infinity,
                child: FilledButton(
                  onPressed: () => _concluir(context),
                  child: const Text('Começar'),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) =>
      const Scaffold(body: Center(child: Text('Tela principal')));
}
```

> 💡 Para criar várias páginas deslizáveis no onboarding, use um `PageView` com indicadores de página.

## 🔒 Passo 5: Dados Sensíveis com flutter_secure_storage

O `shared_preferences` salva os dados em **texto puro**. Para tokens de login e outras informações sensíveis, use o [`flutter_secure_storage`](https://pub.dev/packages/flutter_secure_storage), que usa o **Keychain** no iOS e criptografia com o **Keystore** no Android.

```bash
flutter pub add flutter_secure_storage
```

```dart
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

class TokenStorage {
  static const _chave = 'token_acesso';
  final _storage = const FlutterSecureStorage();

  Future<void> salvar(String token) => _storage.write(key: _chave, value: token);

  Future<String?> ler() => _storage.read(key: _chave);

  Future<void> apagar() => _storage.delete(key: _chave);
}
```

Combinando com o interceptor do Dio (Capítulo 6):

```dart
InterceptorsWrapper(
  onRequest: (options, handler) async {
    final token = await TokenStorage().ler();
    if (token != null) {
      options.headers['Authorization'] = 'Bearer $token';
    }
    handler.next(options);
  },
)
```

> ⚠️ Nenhum armazenamento no aparelho é 100% inviolável. Nunca guarde senhas do usuário em texto: guarde apenas tokens, que podem ser revogados pelo servidor.

## 📁 Passo 6: Arquivos com path_provider

Para salvar arquivos (um backup em JSON, uma imagem baixada), use o [`path_provider`](https://pub.dev/packages/path_provider) para encontrar a pasta certa do app:

```bash
flutter pub add path_provider
```

```dart
import 'dart:convert';
import 'dart:io';
import 'package:path_provider/path_provider.dart';

class BackupService {
  Future<File> _arquivo() async {
    final pasta = await getApplicationDocumentsDirectory();
    return File('${pasta.path}/backup.json');
  }

  Future<void> salvar(List<Map<String, dynamic>> dados) async {
    final arquivo = await _arquivo();
    await arquivo.writeAsString(jsonEncode(dados));
  }

  Future<List<dynamic>> carregar() async {
    final arquivo = await _arquivo();
    if (!await arquivo.exists()) return [];
    return jsonDecode(await arquivo.readAsString());
  }
}
```

### Pastas disponíveis

| Função | Pasta | Uso |
|--------|-------|-----|
| `getApplicationDocumentsDirectory()` | Documentos do app | Dados do usuário, que devem ser mantidos |
| `getApplicationSupportDirectory()` | Suporte do app | Arquivos internos do app |
| `getTemporaryDirectory()` | Temporária | Cache; o sistema pode apagar a qualquer momento |

> ⚠️ `dart:io` e `path_provider` não funcionam na Web. No navegador, use `shared_preferences` ou um banco como o IndexedDB.

## ✅ Melhores Práticas

- **Escolha a ferramenta certa:** preferências no `shared_preferences`, segredos no `flutter_secure_storage` e listas grandes no SQLite.
- **Centralize as chaves** em constantes (`static const _chave = 'tema'`) para evitar erros de digitação.
- **Sempre tenha um valor padrão** (`?? false`) para a primeira execução.
- **Esconda o armazenamento atrás de uma classe** (`TokenStorage`, `ConfiguracoesRepository`). Assim, trocar a tecnologia depois não afeta as telas.
- **Não salve grandes volumes** no `shared_preferences`: ele não foi feito para isso.

## 📖 Recursos Adicionais

- [Cookbook: Armazenar dados chave-valor](https://docs.flutter.dev/cookbook/persistence/key-value)
- [Cookbook: Ler e escrever arquivos](https://docs.flutter.dev/cookbook/persistence/reading-writing-files)
- [Pacote shared_preferences](https://pub.dev/packages/shared_preferences)
- [Pacote flutter_secure_storage](https://pub.dev/packages/flutter_secure_storage)
- [Pacote path_provider](https://pub.dev/packages/path_provider)
