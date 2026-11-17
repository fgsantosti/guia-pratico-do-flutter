# Capítulo 6 - Consumo de APIs

## 📋 Índice

1. [Introdução](#-introdução)
2. [Fundamentos de HTTP](#-fundamentos-de-http)
3. [Preparando o Projeto](#-preparando-o-projeto)
4. [Passo 1: Primeira Requisição GET](#-passo-1-primeira-requisição-get)
5. [Passo 2: Convertendo JSON em Objetos](#-passo-2-convertendo-json-em-objetos)
6. [Passo 3: Exibindo Dados com FutureBuilder](#-passo-3-exibindo-dados-com-futurebuilder)
7. [Passo 4: POST, PUT e DELETE](#-passo-4-post-put-e-delete)
8. [Passo 5: Tratamento de Erros](#-passo-5-tratamento-de-erros)
9. [Passo 6: Organizando em Camadas](#-passo-6-organizando-em-camadas)
10. [Passo 7: Usando o Dio](#-passo-7-usando-o-dio)
11. [Passo 8: Integrando com Provider](#-passo-8-integrando-com-provider)
12. [Passo 9: Integrando com BLoC](#-passo-9-integrando-com-bloc)
13. [Passo 10: Paginação e Pull-to-Refresh](#-passo-10-paginação-e-pull-to-refresh)
14. [Projeto Prático: Buscador de CEP](#-projeto-prático-buscador-de-cep)
15. [Melhores Práticas](#-melhores-práticas)
16. [Problemas Comuns](#-problemas-comuns)
17. [Recursos Adicionais](#-recursos-adicionais)

## 🚀 Introdução

Quase todo aplicativo real conversa com um servidor: busca notícias, envia um cadastro, faz login, consulta preços. Essa conversa acontece por meio de **APIs** (Application Programming Interfaces), normalmente usando o protocolo **HTTP** e trocando dados no formato **JSON**.

Neste capítulo você vai aprender a consumir APIs no Flutter, desde a primeira requisição até uma arquitetura organizada, integrada com **Provider** (Capítulo 4) e **BLoC** (Capítulo 5).

### APIs usadas nos exemplos

Todas são gratuitas e não exigem cadastro:

| API | Endereço | Uso no capítulo |
|-----|----------|-----------------|
| JSONPlaceholder | `https://jsonplaceholder.typicode.com` | Posts falsos para treinar GET, POST, PUT e DELETE |
| ViaCEP | `https://viacep.com.br` | Consulta de endereço pelo CEP |

## 🌐 Fundamentos de HTTP

Uma requisição HTTP tem um **método** (o que você quer fazer), uma **URL** (onde), **cabeçalhos** (informações extras) e, às vezes, um **corpo** (os dados enviados).

### Métodos principais

| Método | Ação | Exemplo |
|--------|------|---------|
| `GET` | Buscar dados | Listar posts |
| `POST` | Criar um recurso | Cadastrar um post |
| `PUT` / `PATCH` | Atualizar (inteiro / parcial) | Editar o título |
| `DELETE` | Remover | Apagar um post |

### Códigos de status

| Faixa | Significado | Exemplos |
|-------|-------------|----------|
| `2xx` | Sucesso | `200 OK`, `201 Created`, `204 No Content` |
| `4xx` | Erro do cliente (do app) | `400 Bad Request`, `401 Unauthorized`, `404 Not Found` |
| `5xx` | Erro do servidor | `500 Internal Server Error`, `503 Service Unavailable` |

### Como é um JSON

```json
{
  "userId": 1,
  "id": 1,
  "title": "Meu primeiro post",
  "body": "Conteúdo do post"
}
```

No Dart, um objeto JSON vira um `Map<String, dynamic>` e uma lista JSON vira uma `List<dynamic>`.

## 🛠 Preparando o Projeto

### Adicionando o pacote http

```bash
flutter pub add http
```

### Permissão de internet no Android

Em modo debug a internet já funciona, mas o **app de release** precisa da permissão. Abra `android/app/src/main/AndroidManifest.xml` e adicione, antes da tag `<application>`:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <uses-permission android:name="android.permission.INTERNET" />

    <application
        ...
```

> ⚠️ Esquecer essa linha é a causa número 1 de "funciona no emulador, mas não funciona no app publicado".

### No macOS

Para apps desktop macOS, adicione em `macos/Runner/DebugProfile.entitlements` **e** `Release.entitlements`:

```xml
<key>com.apple.security.network.client</key>
<true/>
```

## 📥 Passo 1: Primeira Requisição GET

Vamos buscar um único post e imprimir o resultado no console:

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

Future<void> buscarPost() async {
  final url = Uri.parse('https://jsonplaceholder.typicode.com/posts/1');

  final resposta = await http.get(url);

  if (resposta.statusCode == 200) {
    final Map<String, dynamic> json = jsonDecode(resposta.body);
    print('Título: ${json['title']}');
  } else {
    print('Erro: ${resposta.statusCode}');
  }
}
```

### Conceitos aprendidos:
- `Uri.parse()`: converte o texto da URL em um objeto `Uri`
- `http.get()`: faz a requisição e devolve um `Future<Response>`
- `await`: espera a resposta chegar sem travar a tela
- `resposta.body`: o corpo da resposta, como texto
- `jsonDecode()`: transforma o texto JSON em `Map` ou `List`

> 💡 Usamos `import ... as http` para deixar claro de onde vêm as funções (`http.get`, `http.post`), evitando conflito de nomes.

## 🧱 Passo 2: Convertendo JSON em Objetos

Trabalhar com `json['title']` espalhado pelo código é frágil: um erro de digitação só aparece em tempo de execução. O ideal é criar uma **classe modelo**, como fizemos com `Usuario` no Capítulo 8.

```dart
class Post {
  final int? id;
  final int userId;
  final String title;
  final String body;

  const Post({
    this.id,
    required this.userId,
    required this.title,
    required this.body,
  });

  /// Cria um Post a partir do JSON recebido da API
  factory Post.fromJson(Map<String, dynamic> json) {
    return Post(
      id: json['id'] as int?,
      userId: json['userId'] as int,
      title: json['title'] as String,
      body: json['body'] as String,
    );
  }

  /// Converte o Post em JSON para enviar à API
  Map<String, dynamic> toJson() {
    return {
      if (id != null) 'id': id,
      'userId': userId,
      'title': title,
      'body': body,
    };
  }
}
```

Agora a função de busca devolve objetos tipados:

```dart
Future<List<Post>> buscarPosts() async {
  final url = Uri.parse('https://jsonplaceholder.typicode.com/posts');
  final resposta = await http.get(url);

  if (resposta.statusCode != 200) {
    throw Exception('Falha ao carregar posts (${resposta.statusCode})');
  }

  final List<dynamic> lista = jsonDecode(resposta.body);
  return lista.map((item) => Post.fromJson(item)).toList();
}
```

### Conceitos aprendidos:
- `factory`: construtor que pode conter lógica antes de criar o objeto
- `fromJson` / `toJson`: convenção para converter entre JSON e objeto
- `if (id != null) 'id': id`: "collection if", só inclui a chave quando existe valor
- `.map(...).toList()`: transforma cada item da lista JSON em um `Post`

### Gerando o código automaticamente (opcional)

Em projetos com muitos modelos, escrever `fromJson` à mão cansa. O pacote `json_serializable` gera esse código:

```bash
flutter pub add json_annotation
flutter pub add dev:build_runner dev:json_serializable
```

```dart
import 'package:json_annotation/json_annotation.dart';

part 'post.g.dart'; // Arquivo que será gerado

@JsonSerializable()
class Post {
  final int? id;
  final int userId;
  final String title;
  final String body;

  const Post({this.id, required this.userId, required this.title, required this.body});

  factory Post.fromJson(Map<String, dynamic> json) => _$PostFromJson(json);
  Map<String, dynamic> toJson() => _$PostToJson(this);
}
```

```bash
dart run build_runner build --delete-conflicting-outputs
```

> 💡 Para aprender, escreva à mão primeiro. Use geração de código quando já entender o que ela faz por você.

## 📱 Passo 3: Exibindo Dados com FutureBuilder

O `FutureBuilder` reconstrói a tela de acordo com o estado de um `Future`: carregando, com erro ou com dados.

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(const MaterialApp(home: PostsPage()));

class PostsPage extends StatefulWidget {
  const PostsPage({super.key});

  @override
  State<PostsPage> createState() => _PostsPageState();
}

class _PostsPageState extends State<PostsPage> {
  late Future<List<Post>> _futurePosts;

  @override
  void initState() {
    super.initState();
    // Cria o Future UMA vez. Nunca chame a API dentro do build()!
    _futurePosts = buscarPosts();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Posts')),
      body: FutureBuilder<List<Post>>(
        future: _futurePosts,
        builder: (context, snapshot) {
          // 1. Carregando
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const Center(child: CircularProgressIndicator());
          }

          // 2. Erro
          if (snapshot.hasError) {
            return Center(
              child: Column(
                mainAxisSize: MainAxisSize.min,
                children: [
                  Text('Erro: ${snapshot.error}'),
                  const SizedBox(height: 8),
                  ElevatedButton(
                    onPressed: () => setState(() => _futurePosts = buscarPosts()),
                    child: const Text('Tentar novamente'),
                  ),
                ],
              ),
            );
          }

          // 3. Sucesso
          final posts = snapshot.data ?? [];
          if (posts.isEmpty) {
            return const Center(child: Text('Nenhum post encontrado'));
          }

          return ListView.separated(
            itemCount: posts.length,
            separatorBuilder: (_, __) => const Divider(height: 1),
            itemBuilder: (context, index) {
              final post = posts[index];
              return ListTile(
                leading: CircleAvatar(child: Text('${post.id}')),
                title: Text(post.title),
                subtitle: Text(
                  post.body,
                  maxLines: 2,
                  overflow: TextOverflow.ellipsis,
                ),
              );
            },
          );
        },
      ),
    );
  }
}

// Cole aqui a classe Post e a função buscarPosts() do Passo 2
```

### Conceitos aprendidos:
- `FutureBuilder`: widget que reage aos estados de um `Future`
- `snapshot.connectionState`: indica se ainda está carregando
- `snapshot.hasError` / `snapshot.error`: indicam se houve falha
- `snapshot.data`: os dados, quando a requisição termina com sucesso
- Os **4 estados de uma tela com dados remotos**: carregando, erro, vazio e sucesso

> ⚠️ **Erro clássico:** escrever `future: buscarPosts()` diretamente no `build`. Como o `build` roda muitas vezes, o app faria uma requisição nova a cada reconstrução.

## ✏ Passo 4: POST, PUT e DELETE

Para enviar dados, precisamos informar o **corpo** em JSON e o cabeçalho `Content-Type`:

```dart
const _baseUrl = 'https://jsonplaceholder.typicode.com';
const _headers = {'Content-Type': 'application/json; charset=UTF-8'};

/// Cria um post (POST)
Future<Post> criarPost(Post post) async {
  final resposta = await http.post(
    Uri.parse('$_baseUrl/posts'),
    headers: _headers,
    body: jsonEncode(post.toJson()),
  );

  if (resposta.statusCode == 201) {
    return Post.fromJson(jsonDecode(resposta.body));
  }
  throw Exception('Erro ao criar post (${resposta.statusCode})');
}

/// Atualiza um post inteiro (PUT)
Future<Post> atualizarPost(Post post) async {
  final resposta = await http.put(
    Uri.parse('$_baseUrl/posts/${post.id}'),
    headers: _headers,
    body: jsonEncode(post.toJson()),
  );

  if (resposta.statusCode == 200) {
    return Post.fromJson(jsonDecode(resposta.body));
  }
  throw Exception('Erro ao atualizar post (${resposta.statusCode})');
}

/// Remove um post (DELETE)
Future<void> removerPost(int id) async {
  final resposta = await http.delete(Uri.parse('$_baseUrl/posts/$id'));

  if (resposta.statusCode != 200 && resposta.statusCode != 204) {
    throw Exception('Erro ao remover post (${resposta.statusCode})');
  }
}
```

Usando em um botão:

```dart
ElevatedButton(
  onPressed: () async {
    final novo = await criarPost(
      const Post(userId: 1, title: 'Olá API', body: 'Criado pelo Flutter'),
    );
    if (!context.mounted) return;
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text('Post criado com id ${novo.id}')),
    );
  },
  child: const Text('Criar post'),
)
```

### Conceitos aprendidos:
- `jsonEncode()`: o inverso de `jsonDecode`, transforma `Map` em texto JSON
- `headers`: avisam ao servidor que estamos enviando JSON
- `201 Created`: status padrão de sucesso para criação
- `context.mounted`: verifica se a tela ainda existe depois de um `await`

> 💡 O JSONPlaceholder **simula** as operações: ele responde como se tivesse salvo, mas os dados não mudam de verdade. É perfeito para treinar.

## 🧯 Passo 5: Tratamento de Erros

Em um app real, muita coisa pode dar errado: sem internet, servidor lento, resposta inesperada. Vamos criar uma exceção própria com mensagens amigáveis:

```dart
import 'dart:async';
import 'dart:convert';
import 'dart:io';
import 'package:http/http.dart' as http;

class ApiException implements Exception {
  final String mensagem;
  final int? statusCode;

  const ApiException(this.mensagem, {this.statusCode});

  @override
  String toString() => mensagem;
}

Future<dynamic> getJson(String url) async {
  try {
    final resposta = await http
        .get(Uri.parse(url))
        .timeout(const Duration(seconds: 10));

    switch (resposta.statusCode) {
      case >= 200 && < 300:
        return jsonDecode(resposta.body);
      case 401:
        throw const ApiException('Sessão expirada. Faça login novamente.', statusCode: 401);
      case 404:
        throw const ApiException('Recurso não encontrado.', statusCode: 404);
      case >= 500:
        throw ApiException('Servidor indisponível. Tente mais tarde.',
            statusCode: resposta.statusCode);
      default:
        throw ApiException('Erro inesperado (${resposta.statusCode}).',
            statusCode: resposta.statusCode);
    }
  } on SocketException {
    throw const ApiException('Sem conexão com a internet.');
  } on TimeoutException {
    throw const ApiException('O servidor demorou demais para responder.');
  } on FormatException {
    throw const ApiException('Resposta do servidor em formato inválido.');
  }
}
```

### Conceitos aprendidos:
- `.timeout()`: cancela a espera depois de um tempo e lança `TimeoutException`
- `SocketException`: falha de rede (sem internet, DNS, servidor fora do ar)
- `FormatException`: o corpo não era um JSON válido
- `switch` com padrões (`>= 200 && < 300`): recurso do Dart 3
- Exceções próprias deixam a tela exibir mensagens claras para o usuário

> ⚠️ `dart:io` (e portanto `SocketException`) **não existe na Web**. Se o app também roda no navegador, capture `http.ClientException`, que funciona em todas as plataformas.

## 🗂 Passo 6: Organizando em Camadas

Chamar a API diretamente na tela funciona, mas mistura responsabilidades. Vamos usar a mesma ideia do Capítulo 8 (Model, Repository e View):

```
lib/
├── models/
│   └── post.dart              # Classe Post (fromJson/toJson)
├── services/
│   └── api_client.dart        # Faz as requisições HTTP "cruas"
├── repositories/
│   └── post_repository.dart   # Converte JSON em Post e expõe operações
├── providers/
│   └── posts_provider.dart    # Estado da tela (Passo 8)
└── pages/
    └── posts_page.dart        # Interface
```

### services/api_client.dart

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

class ApiClient {
  ApiClient({http.Client? client, this.baseUrl = 'https://jsonplaceholder.typicode.com'})
      : _client = client ?? http.Client();

  final http.Client _client;
  final String baseUrl;

  Future<dynamic> get(String path) async {
    final resposta = await _client.get(Uri.parse('$baseUrl$path'));
    _verificar(resposta);
    return jsonDecode(resposta.body);
  }

  Future<dynamic> post(String path, Map<String, dynamic> body) async {
    final resposta = await _client.post(
      Uri.parse('$baseUrl$path'),
      headers: {'Content-Type': 'application/json; charset=UTF-8'},
      body: jsonEncode(body),
    );
    _verificar(resposta);
    return jsonDecode(resposta.body);
  }

  void _verificar(http.Response resposta) {
    if (resposta.statusCode < 200 || resposta.statusCode >= 300) {
      throw Exception('Erro HTTP ${resposta.statusCode}');
    }
  }
}
```

### repositories/post_repository.dart

```dart
import '../models/post.dart';
import '../services/api_client.dart';

class PostRepository {
  PostRepository(this._api);

  final ApiClient _api;

  Future<List<Post>> listar({int pagina = 1, int limite = 20}) async {
    final List<dynamic> json = await _api.get('/posts?_page=$pagina&_limit=$limite');
    return json.map((e) => Post.fromJson(e)).toList();
  }

  Future<Post> criar(Post post) async {
    final json = await _api.post('/posts', post.toJson());
    return Post.fromJson(json);
  }
}
```

### Conceitos aprendidos:
- **ApiClient**: sabe falar HTTP, mas não conhece `Post`
- **Repository**: sabe o que é um `Post`, mas não sabe como o HTTP funciona
- Receber o `http.Client` no construtor permite trocar por um **mock** nos testes (Capítulo 7)

## 🚀 Passo 7: Usando o Dio

O [`dio`](https://pub.dev/packages/dio) é um cliente HTTP mais completo. Ele converte JSON automaticamente e oferece **interceptors**, que executam código antes ou depois de toda requisição.

```bash
flutter pub add dio
```

```dart
import 'package:dio/dio.dart';

class DioClient {
  DioClient() {
    dio.interceptors.addAll([
      // Adiciona o token em todas as requisições
      InterceptorsWrapper(
        onRequest: (options, handler) {
          const token = 'meu-token'; // Em um app real, leia do armazenamento seguro
          options.headers['Authorization'] = 'Bearer $token';
          handler.next(options);
        },
        onError: (erro, handler) {
          if (erro.response?.statusCode == 401) {
            // Ex.: redirecionar para o login
          }
          handler.next(erro);
        },
      ),
      // Mostra as requisições no console (útil em desenvolvimento)
      LogInterceptor(requestBody: true, responseBody: true),
    ]);
  }

  final dio = Dio(
    BaseOptions(
      baseUrl: 'https://jsonplaceholder.typicode.com',
      connectTimeout: const Duration(seconds: 10),
      receiveTimeout: const Duration(seconds: 10),
    ),
  );

  Future<List<Post>> buscarPosts() async {
    try {
      final resposta = await dio.get<List<dynamic>>('/posts');
      return resposta.data!.map((e) => Post.fromJson(e)).toList();
    } on DioException catch (e) {
      throw Exception(_mensagem(e));
    }
  }

  String _mensagem(DioException e) {
    return switch (e.type) {
      DioExceptionType.connectionTimeout ||
      DioExceptionType.receiveTimeout => 'Tempo de conexão esgotado',
      DioExceptionType.connectionError => 'Sem conexão com a internet',
      DioExceptionType.badResponse => 'Erro do servidor (${e.response?.statusCode})',
      _ => 'Erro inesperado',
    };
  }
}
```

### http x dio

| Recurso | `http` | `dio` |
|---------|--------|-------|
| Simplicidade | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| JSON automático | ❌ | ✅ |
| Interceptors | ❌ | ✅ |
| Timeout global | ❌ | ✅ |
| Upload com progresso | Limitado | ✅ |
| Cancelar requisição | Limitado | ✅ (`CancelToken`) |

> 💡 Comece com `http` para entender o básico. Migre para `dio` quando precisar de token em todas as requisições, logs ou uploads.

## 🎯 Passo 8: Integrando com Provider

Vamos mover a lógica de carregamento para um `ChangeNotifier`, como no Capítulo 4:

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

enum Status { inicial, carregando, sucesso, erro }

class PostsProvider extends ChangeNotifier {
  PostsProvider(this._repository);

  final PostRepository _repository;

  Status status = Status.inicial;
  List<Post> posts = [];
  String? erro;

  Future<void> carregar() async {
    status = Status.carregando;
    notifyListeners();

    try {
      posts = await _repository.listar();
      status = Status.sucesso;
    } catch (e) {
      erro = e.toString();
      status = Status.erro;
    }
    notifyListeners();
  }
}

void main() {
  runApp(
    ChangeNotifierProvider(
      create: (_) => PostsProvider(PostRepository(ApiClient()))..carregar(),
      child: const MaterialApp(home: PostsPage()),
    ),
  );
}

class PostsPage extends StatelessWidget {
  const PostsPage({super.key});

  @override
  Widget build(BuildContext context) {
    final provider = context.watch<PostsProvider>();

    return Scaffold(
      appBar: AppBar(title: const Text('Posts (Provider)')),
      body: switch (provider.status) {
        Status.inicial || Status.carregando =>
          const Center(child: CircularProgressIndicator()),
        Status.erro => Center(
            child: TextButton(
              onPressed: provider.carregar,
              child: Text('${provider.erro}\nToque para tentar de novo'),
            ),
          ),
        Status.sucesso => ListView.builder(
            itemCount: provider.posts.length,
            itemBuilder: (_, i) => ListTile(title: Text(provider.posts[i].title)),
          ),
      },
    );
  }
}
```

### Conceitos aprendidos:
- Um `enum` de status deixa claro em que situação a tela está
- `..carregar()`: operador cascata, cria o provider e já dispara o carregamento
- `switch` como expressão escolhe o widget certo para cada status

## 🛡 Passo 9: Integrando com BLoC

A mesma tela, agora com o padrão do Capítulo 5:

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

// ---------- Eventos ----------
sealed class PostsEvent {}

final class PostsCarregados extends PostsEvent {}

// ---------- Estados ----------
sealed class PostsState {}

final class PostsCarregando extends PostsState {}

final class PostsSucesso extends PostsState {
  PostsSucesso(this.posts);
  final List<Post> posts;
}

final class PostsErro extends PostsState {
  PostsErro(this.mensagem);
  final String mensagem;
}

// ---------- BLoC ----------
class PostsBloc extends Bloc<PostsEvent, PostsState> {
  PostsBloc(this._repository) : super(PostsCarregando()) {
    on<PostsCarregados>((event, emit) async {
      emit(PostsCarregando());
      try {
        final posts = await _repository.listar();
        emit(PostsSucesso(posts));
      } catch (e) {
        emit(PostsErro(e.toString()));
      }
    });
  }

  final PostRepository _repository;
}

// ---------- Tela ----------
class PostsBlocPage extends StatelessWidget {
  const PostsBlocPage({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (_) => PostsBloc(PostRepository(ApiClient()))..add(PostsCarregados()),
      child: Scaffold(
        appBar: AppBar(title: const Text('Posts (BLoC)')),
        body: BlocBuilder<PostsBloc, PostsState>(
          builder: (context, state) => switch (state) {
            PostsCarregando() => const Center(child: CircularProgressIndicator()),
            PostsErro(:final mensagem) => Center(
                child: TextButton(
                  onPressed: () => context.read<PostsBloc>().add(PostsCarregados()),
                  child: Text('$mensagem\nToque para tentar de novo'),
                ),
              ),
            PostsSucesso(:final posts) => ListView.builder(
                itemCount: posts.length,
                itemBuilder: (_, i) => ListTile(title: Text(posts[i].title)),
              ),
          },
        ),
      ),
    );
  }
}
```

### Conceitos aprendidos:
- `sealed class`: o compilador obriga o `switch` a tratar **todos** os estados
- `PostsErro(:final mensagem)`: desestruturação, extrai o campo direto no `switch`
- Handlers `async` no `on<Evento>` permitem usar `await` e emitir vários estados

> 💡 Repare que o `PostRepository` é **o mesmo** nos Passos 8 e 9. Separar camadas permite trocar a gerência de estado sem reescrever o acesso à API.

## 📜 Passo 10: Paginação e Pull-to-Refresh

Listas grandes não devem ser baixadas de uma vez. A ideia é carregar uma **página** por vez e buscar a próxima quando o usuário chega ao fim da lista:

```dart
class PostsPaginadosPage extends StatefulWidget {
  const PostsPaginadosPage({super.key});

  @override
  State<PostsPaginadosPage> createState() => _PostsPaginadosPageState();
}

class _PostsPaginadosPageState extends State<PostsPaginadosPage> {
  final _repository = PostRepository(ApiClient());
  final _scrollController = ScrollController();
  final List<Post> _posts = [];

  int _pagina = 1;
  bool _carregando = false;
  bool _temMais = true;

  @override
  void initState() {
    super.initState();
    _carregarMais();
    _scrollController.addListener(() {
      // Faltando 200 pixels para o fim, busca a próxima página
      final posicao = _scrollController.position;
      if (posicao.pixels >= posicao.maxScrollExtent - 200) {
        _carregarMais();
      }
    });
  }

  Future<void> _carregarMais() async {
    if (_carregando || !_temMais) return;
    setState(() => _carregando = true);

    final novos = await _repository.listar(pagina: _pagina, limite: 15);

    setState(() {
      _posts.addAll(novos);
      _pagina++;
      _temMais = novos.isNotEmpty;
      _carregando = false;
    });
  }

  Future<void> _recarregar() async {
    _posts.clear();
    _pagina = 1;
    _temMais = true;
    await _carregarMais();
  }

  @override
  void dispose() {
    _scrollController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Posts paginados')),
      body: RefreshIndicator(
        onRefresh: _recarregar,
        child: ListView.builder(
          controller: _scrollController,
          itemCount: _posts.length + (_temMais ? 1 : 0),
          itemBuilder: (context, index) {
            if (index == _posts.length) {
              // Último item: indicador de carregamento
              return const Padding(
                padding: EdgeInsets.all(16),
                child: Center(child: CircularProgressIndicator()),
              );
            }
            final post = _posts[index];
            return ListTile(
              leading: CircleAvatar(child: Text('${post.id}')),
              title: Text(post.title),
            );
          },
        ),
      ),
    );
  }
}
```

### Conceitos aprendidos:
- `ScrollController`: permite saber a posição da rolagem
- `_page` e `_limit`: parâmetros de paginação do JSONPlaceholder (cada API tem os seus)
- `_carregando`: evita disparar várias requisições ao mesmo tempo
- `RefreshIndicator`: o gesto de "puxar para atualizar"

## 📮 Projeto Prático: Buscador de CEP

Vamos juntar tudo em um app útil: o usuário digita um CEP e o app mostra o endereço usando a API **ViaCEP**.

Resposta da API para `https://viacep.com.br/ws/01001000/json/`:

```json
{
  "cep": "01001-000",
  "logradouro": "Praça da Sé",
  "bairro": "Sé",
  "localidade": "São Paulo",
  "uf": "SP"
}
```

Quando o CEP não existe, a API responde `200` com `{"erro": true}`.

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: CepPage(),
    ));

// ---------- Modelo ----------
class Endereco {
  final String cep;
  final String logradouro;
  final String bairro;
  final String cidade;
  final String uf;

  const Endereco({
    required this.cep,
    required this.logradouro,
    required this.bairro,
    required this.cidade,
    required this.uf,
  });

  factory Endereco.fromJson(Map<String, dynamic> json) => Endereco(
        cep: json['cep'] ?? '',
        logradouro: json['logradouro'] ?? '',
        bairro: json['bairro'] ?? '',
        cidade: json['localidade'] ?? '',
        uf: json['uf'] ?? '',
      );
}

// ---------- Serviço ----------
class CepService {
  Future<Endereco> buscar(String cep) async {
    final somenteNumeros = cep.replaceAll(RegExp(r'\D'), '');
    if (somenteNumeros.length != 8) {
      throw const FormatException('O CEP deve ter 8 dígitos');
    }

    final resposta = await http
        .get(Uri.parse('https://viacep.com.br/ws/$somenteNumeros/json/'))
        .timeout(const Duration(seconds: 10));

    if (resposta.statusCode != 200) {
      throw Exception('Erro ao consultar CEP (${resposta.statusCode})');
    }

    final Map<String, dynamic> json = jsonDecode(resposta.body);
    if (json['erro'] != null) {
      throw Exception('CEP não encontrado');
    }
    return Endereco.fromJson(json);
  }
}

// ---------- Tela ----------
class CepPage extends StatefulWidget {
  const CepPage({super.key});

  @override
  State<CepPage> createState() => _CepPageState();
}

class _CepPageState extends State<CepPage> {
  final _controller = TextEditingController();
  final _service = CepService();

  Endereco? _endereco;
  String? _erro;
  bool _carregando = false;

  Future<void> _buscar() async {
    FocusScope.of(context).unfocus(); // Fecha o teclado
    setState(() {
      _carregando = true;
      _erro = null;
      _endereco = null;
    });

    try {
      final endereco = await _service.buscar(_controller.text);
      setState(() => _endereco = endereco);
    } on FormatException catch (e) {
      setState(() => _erro = e.message);
    } catch (e) {
      setState(() => _erro = e.toString().replaceFirst('Exception: ', ''));
    } finally {
      setState(() => _carregando = false);
    }
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('📮 Buscador de CEP')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            TextField(
              controller: _controller,
              keyboardType: TextInputType.number,
              maxLength: 9,
              decoration: InputDecoration(
                labelText: 'CEP',
                hintText: '01001-000',
                border: const OutlineInputBorder(),
                suffixIcon: IconButton(
                  icon: const Icon(Icons.search),
                  onPressed: _carregando ? null : _buscar,
                ),
              ),
              onSubmitted: (_) => _buscar(),
            ),
            const SizedBox(height: 16),
            if (_carregando) const CircularProgressIndicator(),
            if (_erro != null)
              Text(_erro!, style: const TextStyle(color: Colors.red, fontSize: 16)),
            if (_endereco != null) _CartaoEndereco(endereco: _endereco!),
          ],
        ),
      ),
    );
  }
}

class _CartaoEndereco extends StatelessWidget {
  const _CartaoEndereco({required this.endereco});

  final Endereco endereco;

  @override
  Widget build(BuildContext context) {
    return Card(
      child: ListTile(
        leading: const Icon(Icons.location_on, size: 40),
        title: Text(endereco.logradouro.isEmpty ? 'CEP geral da cidade' : endereco.logradouro),
        subtitle: Text('${endereco.bairro}\n${endereco.cidade} - ${endereco.uf}\nCEP: ${endereco.cep}'),
        isThreeLine: true,
      ),
    );
  }
}
```

### Desafios
1. Salve as últimas 5 buscas com `shared_preferences` (veja a seção 8.1).
2. Adicione uma máscara de CEP enquanto o usuário digita (veja a seção 2.2).
3. Reescreva a tela usando Provider ou BLoC.

## ✅ Melhores Práticas

- **Nunca chame a API dentro do `build()`.** Use `initState`, um Provider ou um BLoC.
- **Trate os 4 estados:** carregando, erro, vazio e sucesso.
- **Use classes modelo** em vez de acessar `Map` espalhado pelo código.
- **Separe em camadas** (cliente HTTP, repositório, estado, tela).
- **Defina timeouts:** sem eles, o usuário pode ficar esperando para sempre.
- **Não coloque chaves secretas no app.** Qualquer string no código pode ser extraída do APK. Chaves sensíveis devem ficar no seu servidor.
- **Use HTTPS.** O Android bloqueia HTTP sem criptografia por padrão.
- **Verifique `mounted`** antes de usar `context` ou `setState` depois de um `await`.

## 🐛 Problemas Comuns

| Problema | Causa provável | Solução |
|----------|----------------|---------|
| Funciona no debug, falha no release | Falta a permissão `INTERNET` | Adicione no `AndroidManifest.xml` |
| `Cleartext HTTP traffic not permitted` | URL com `http://` | Use `https://` |
| Não acessa `localhost` no emulador Android | O emulador tem rede própria | Use `10.0.2.2` no lugar de `localhost` |
| `type 'List<dynamic>' is not a subtype of 'Map'` | O JSON é uma lista, não um objeto | Confira a estrutura da resposta antes de converter |
| Requisições repetidas sem parar | Future criado dentro do `build` | Crie o Future no `initState` |
| Erro de CORS na Web | O servidor não permite o domínio | Precisa ser liberado no servidor da API |
| Acentos aparecendo errados | Corpo decodificado em outra codificação | Use `utf8.decode(resposta.bodyBytes)` |

## 📖 Recursos Adicionais

- [Documentação oficial: Buscar dados da internet](https://docs.flutter.dev/cookbook/networking/fetch-data)
- [Documentação oficial: Serialização JSON](https://docs.flutter.dev/data-and-backend/serialization/json)
- [Pacote http](https://pub.dev/packages/http)
- [Pacote dio](https://pub.dev/packages/dio)
- [JSONPlaceholder](https://jsonplaceholder.typicode.com)
- [ViaCEP](https://viacep.com.br)
