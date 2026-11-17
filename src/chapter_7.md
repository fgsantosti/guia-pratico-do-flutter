# Capítulo 7 - Testes em Flutter

## 📋 Índice

1. [Introdução](#-introdução)
2. [Tipos de Teste](#-tipos-de-teste)
3. [Preparando o Projeto](#-preparando-o-projeto)
4. [Passo 1: Primeiro Teste de Unidade](#-passo-1-primeiro-teste-de-unidade)
5. [Passo 2: Organizando com group e setUp](#-passo-2-organizando-com-group-e-setup)
6. [Passo 3: Testando Código Assíncrono](#-passo-3-testando-código-assíncrono)
7. [Passo 4: Testes de Widget](#-passo-4-testes-de-widget)
8. [Passo 5: Mocks com mocktail](#-passo-5-mocks-com-mocktail)
9. [Passo 6: Testando Provider](#-passo-6-testando-provider)
10. [Passo 7: Testando BLoC](#-passo-7-testando-bloc)
11. [Passo 8: Golden Tests](#-passo-8-golden-tests)
12. [Passo 9: Testes de Integração](#-passo-9-testes-de-integração)
13. [Cobertura e Integração Contínua](#-cobertura-e-integração-contínua)
14. [Melhores Práticas](#-melhores-práticas)
15. [Recursos Adicionais](#-recursos-adicionais)

## 🚀 Introdução

Testes automatizados são pequenos programas que verificam se o seu código faz o que deveria. Eles parecem trabalho extra no começo, mas trazem benefícios enormes:

- **Confiança para mudar:** você refatora sem medo de quebrar algo escondido.
- **Bugs encontrados cedo:** o teste falha no seu computador, não no celular do usuário.
- **Documentação viva:** um bom teste mostra como o código deve ser usado.

Neste capítulo vamos testar exemplos dos capítulos anteriores: o modelo `Usuario` (Capítulo 8), o `HomeBloc` (Capítulo 5) e o repositório de posts (Capítulo 6).

## 🔺 Tipos de Teste

O Flutter tem três tipos principais de teste, organizados na chamada **pirâmide de testes**:

```
          ▲
         ╱ ╲        Integração
        ╱   ╲       Poucos, lentos, testam o app inteiro
       ╱─────╲
      ╱       ╲     Widget
     ╱         ╲    Vários, testam uma tela ou componente
    ╱───────────╲
   ╱             ╲  Unidade
  ╱               ╲ Muitos, rápidos, testam funções e classes
 ╱─────────────────╲
```

| Tipo | Testa | Velocidade | Precisa de emulador? |
|------|-------|------------|----------------------|
| Unidade | Uma função, classe ou BLoC | ⚡ Muito rápido | ❌ Não |
| Widget | Um widget e suas interações | ⚡ Rápido | ❌ Não |
| Integração | O app completo, em execução real | 🐢 Lento | ✅ Sim |

## 📦 Preparando o Projeto

Todo projeto Flutter já vem com o pacote `flutter_test` e uma pasta `test/`. Vamos adicionar as ferramentas extras usadas no capítulo:

```bash
flutter pub add dev:mocktail dev:bloc_test
flutter pub add 'dev:integration_test:{"sdk":"flutter"}'
```

O `pubspec.yaml` ficará assim:

```yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  integration_test:
    sdk: flutter
  mocktail: ^1.0.4
  bloc_test: ^9.1.7
```

### Estrutura de pastas

Os arquivos de teste espelham a pasta `lib/` e **devem** terminar com `_test.dart`:

```
lib/
├── model/usuario.dart
└── features/home/bloc/home_bloc.dart
test/
├── model/usuario_test.dart
└── features/home/bloc/home_bloc_test.dart
integration_test/
└── app_test.dart
```

### Rodando os testes

```bash
flutter test                               # Todos os testes da pasta test/
flutter test test/model/usuario_test.dart  # Um arquivo específico
flutter test --name "converte"             # Só testes cujo nome contém "converte"
```

> 💡 No VS Code e no Android Studio aparecem botões **Run | Debug** acima de cada teste.

## 🧪 Passo 1: Primeiro Teste de Unidade

Vamos começar com uma função simples para entender a estrutura:

```dart
// lib/utils/calculadora.dart
class Calculadora {
  int somar(int a, int b) => a + b;

  double dividir(int a, int b) {
    if (b == 0) throw ArgumentError('Divisão por zero');
    return a / b;
  }
}
```

```dart
// test/utils/calculadora_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:meu_app/utils/calculadora.dart';

void main() {
  test('somar 2 + 3 deve resultar em 5', () {
    // Arrange (preparar)
    final calculadora = Calculadora();

    // Act (agir)
    final resultado = calculadora.somar(2, 3);

    // Assert (verificar)
    expect(resultado, 5);
  });

  test('dividir por zero deve lançar ArgumentError', () {
    final calculadora = Calculadora();

    expect(() => calculadora.dividir(10, 0), throwsArgumentError);
  });
}
```

Saída no terminal:

```
00:01 +2: All tests passed!
```

### Conceitos aprendidos:
- `test('descrição', () { ... })`: define um teste
- `expect(valorReal, valorEsperado)`: a verificação; se não bater, o teste falha
- **Arrange, Act, Assert**: o padrão para organizar todo teste
- `throwsArgumentError`: um **matcher** que verifica se uma exceção foi lançada

### Matchers mais usados

| Matcher | Verifica |
|---------|----------|
| `equals(x)` ou só `x` | Igualdade |
| `isTrue` / `isFalse` | Booleanos |
| `isNull` / `isNotNull` | Nulos |
| `isA<Tipo>()` | Tipo do objeto |
| `contains(x)` | Lista ou texto contém `x` |
| `hasLength(n)` | Tamanho de lista ou texto |
| `greaterThan(n)` / `lessThan(n)` | Comparação numérica |
| `throwsA(isA<Erro>())` | Lança uma exceção específica |

## 🗂 Passo 2: Organizando com group e setUp

Vamos testar o modelo `Usuario` do Capítulo 8:

```dart
// test/model/usuario_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:meu_app/model/usuario.dart';

void main() {
  group('Usuario', () {
    late Usuario usuario;

    // Executa antes de CADA teste do grupo
    setUp(() {
      usuario = Usuario(id: 1, nome: 'Ana', email: 'ana@email.com', idade: 30);
    });

    test('toMap converte todos os campos', () {
      final map = usuario.toMap();

      expect(map, {
        'id': 1,
        'nome': 'Ana',
        'email': 'ana@email.com',
        'idade': 30,
      });
    });

    test('fromMap cria um usuário com os mesmos dados', () {
      final map = {'id': 2, 'nome': 'Bruno', 'email': 'b@email.com', 'idade': 25};

      final resultado = Usuario.fromMap(map);

      expect(resultado.id, 2);
      expect(resultado.nome, 'Bruno');
      expect(resultado.idade, 25);
    });

    test('toMap seguido de fromMap preserva os dados (ida e volta)', () {
      final copia = Usuario.fromMap(usuario.toMap());

      expect(copia.toString(), usuario.toString());
    });

    test('id pode ser nulo para usuários ainda não salvos', () {
      final novo = Usuario(nome: 'Caio', email: 'c@email.com', idade: 20);

      expect(novo.id, isNull);
    });
  });
}
```

### Conceitos aprendidos:
- `group()`: agrupa testes relacionados; o nome aparece no relatório
- `setUp()`: roda antes de cada teste, garantindo que cada um comece "limpo"
- `tearDown()`: o oposto, roda depois de cada teste (para fechar recursos)
- `setUpAll()` / `tearDownAll()`: rodam uma única vez, antes e depois do grupo
- Testes de "ida e volta" são ótimos para conversões como `toMap`/`fromMap`

## ⏳ Passo 3: Testando Código Assíncrono

Basta marcar o teste como `async` e usar `await`:

```dart
Future<String> buscarSaudacao() async {
  await Future.delayed(const Duration(milliseconds: 100));
  return 'Olá!';
}

void main() {
  test('buscarSaudacao retorna Olá!', () async {
    final resultado = await buscarSaudacao();
    expect(resultado, 'Olá!');
  });

  // Alternativa: o matcher "completion" espera o Future terminar
  test('buscarSaudacao com completion', () {
    expect(buscarSaudacao(), completion('Olá!'));
  });

  // Streams: emitsInOrder verifica a sequência de valores
  test('stream emite 1, 2 e 3', () {
    final stream = Stream.fromIterable([1, 2, 3]);
    expect(stream, emitsInOrder([1, 2, 3, emitsDone]));
  });
}
```

## 📱 Passo 4: Testes de Widget

Testes de widget renderizam um widget em um ambiente simulado, sem emulador. Vamos testar um contador parecido com o do Capítulo 1:

```dart
// lib/contador_page.dart
import 'package:flutter/material.dart';

class ContadorPage extends StatefulWidget {
  const ContadorPage({super.key});

  @override
  State<ContadorPage> createState() => _ContadorPageState();
}

class _ContadorPageState extends State<ContadorPage> {
  int _contador = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Contador')),
      body: Center(
        child: Text('$_contador', key: const Key('texto_contador')),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => setState(() => _contador++),
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

```dart
// test/contador_page_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:meu_app/contador_page.dart';

void main() {
  testWidgets('contador começa em 0 e incrementa ao tocar no botão', (tester) async {
    // Renderiza o widget (precisa de um MaterialApp em volta)
    await tester.pumpWidget(const MaterialApp(home: ContadorPage()));

    // Verifica o estado inicial
    expect(find.text('0'), findsOneWidget);
    expect(find.text('1'), findsNothing);

    // Toca no botão
    await tester.tap(find.byIcon(Icons.add));

    // Redesenha a tela depois do setState
    await tester.pump();

    // Verifica o novo estado
    expect(find.text('0'), findsNothing);
    expect(find.text('1'), findsOneWidget);
  });

  testWidgets('encontra o texto pela Key', (tester) async {
    await tester.pumpWidget(const MaterialApp(home: ContadorPage()));

    final texto = tester.widget<Text>(find.byKey(const Key('texto_contador')));
    expect(texto.data, '0');
  });
}
```

### Conceitos aprendidos:
- `testWidgets`: versão de `test` que fornece um `WidgetTester`
- `pumpWidget()`: renderiza o widget pela primeira vez
- `pump()`: avança um frame (necessário depois de `tap`, `setState` etc.)
- `pumpAndSettle()`: avança frames até as animações terminarem
- `find`: localiza widgets (`find.text`, `find.byIcon`, `find.byType`, `find.byKey`)
- `findsOneWidget`, `findsNothing`, `findsNWidgets(n)`: matchers de quantidade

### Interações disponíveis

```dart
await tester.tap(find.byType(ElevatedButton));          // Tocar
await tester.longPress(find.text('Item'));              // Toque longo
await tester.enterText(find.byType(TextField), 'Ana');  // Digitar
await tester.drag(find.byType(ListView), const Offset(0, -300)); // Arrastar
await tester.scrollUntilVisible(find.text('Item 50'), 100);      // Rolar até achar
```

### Testando um formulário

```dart
testWidgets('mostra erro quando o nome está vazio', (tester) async {
  await tester.pumpWidget(const MaterialApp(home: CadastroPage()));

  await tester.tap(find.text('Salvar'));
  await tester.pump();

  expect(find.text('Informe o nome'), findsOneWidget);
});
```

> 💡 Use `Key` nos widgets importantes. Ela torna os testes mais estáveis do que procurar por texto, que pode mudar com a tradução do app.

## 🎭 Passo 5: Mocks com mocktail

Testes de unidade não devem acessar a internet: seriam lentos e falhariam sem conexão. A solução é usar **mocks**, objetos falsos que fingem ser a dependência real.

Vamos testar o `PostRepository` do Capítulo 6 simulando o `ApiClient`:

```dart
// test/repositories/post_repository_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';
import 'package:meu_app/repositories/post_repository.dart';
import 'package:meu_app/services/api_client.dart';

// 1. Cria a classe mock
class MockApiClient extends Mock implements ApiClient {}

void main() {
  late MockApiClient api;
  late PostRepository repository;

  setUp(() {
    api = MockApiClient();
    repository = PostRepository(api);
  });

  test('listar converte o JSON em uma lista de Post', () async {
    // 2. Define o comportamento: "quando chamarem get, devolva isto"
    when(() => api.get(any())).thenAnswer((_) async => [
          {'id': 1, 'userId': 1, 'title': 'Primeiro', 'body': 'A'},
          {'id': 2, 'userId': 1, 'title': 'Segundo', 'body': 'B'},
        ]);

    final posts = await repository.listar();

    expect(posts, hasLength(2));
    expect(posts.first.title, 'Primeiro');

    // 3. Verifica se a dependência foi chamada corretamente
    verify(() => api.get('/posts?_page=1&_limit=20')).called(1);
  });

  test('listar repassa o erro da API', () async {
    when(() => api.get(any())).thenThrow(Exception('Sem internet'));

    await expectLater(repository.listar(), throwsA(isA<Exception>()));
  });
}
```

### Conceitos aprendidos:
- `class MockX extends Mock implements X`: cria um mock de qualquer classe
- `when(...).thenAnswer(...)`: define a resposta de um método assíncrono
- `when(...).thenReturn(...)`: o mesmo, para métodos síncronos
- `when(...).thenThrow(...)`: simula um erro
- `any()`: aceita qualquer argumento
- `verify(...).called(n)`: confirma que o método foi chamado `n` vezes

> 💡 É por isso que no Capítulo 6 o `PostRepository` recebe o `ApiClient` no construtor: essa **injeção de dependência** é o que permite trocar o real pelo mock.

## 🎯 Passo 6: Testando Provider

Um `ChangeNotifier` é uma classe Dart comum, então pode ser testado com testes de unidade. Usando o `PostsProvider` do Capítulo 6:

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';

class MockPostRepository extends Mock implements PostRepository {}

void main() {
  late MockPostRepository repository;
  late PostsProvider provider;

  setUp(() {
    repository = MockPostRepository();
    provider = PostsProvider(repository);
  });

  test('status começa como inicial', () {
    expect(provider.status, Status.inicial);
  });

  test('carregar com sucesso preenche a lista', () async {
    when(() => repository.listar()).thenAnswer(
      (_) async => [const Post(id: 1, userId: 1, title: 'T', body: 'B')],
    );

    await provider.carregar();

    expect(provider.status, Status.sucesso);
    expect(provider.posts, hasLength(1));
  });

  test('carregar com erro muda o status para erro', () async {
    when(() => repository.listar()).thenThrow(Exception('falhou'));

    await provider.carregar();

    expect(provider.status, Status.erro);
    expect(provider.erro, contains('falhou'));
  });

  test('notifica os ouvintes duas vezes (carregando e resultado)', () async {
    when(() => repository.listar()).thenAnswer((_) async => []);
    var notificacoes = 0;
    provider.addListener(() => notificacoes++);

    await provider.carregar();

    expect(notificacoes, 2);
  });
}
```

## 🛡 Passo 7: Testando BLoC

O pacote `bloc_test` simplifica muito o teste de BLoCs. Vamos testar o `HomeBloc` do Capítulo 5:

```dart
// test/features/home/bloc/home_bloc_test.dart
import 'package:bloc_test/bloc_test.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:meu_app/features/home/bloc/home_bloc.dart';
import 'package:meu_app/features/home/bloc/home_event.dart';
import 'package:meu_app/features/home/bloc/home_state.dart';

void main() {
  group('HomeBloc', () {
    test('estado inicial tem contador 0', () {
      expect(HomeBloc().state.counter, 0);
    });

    blocTest<HomeBloc, HomeState>(
      'emite contador 1 quando recebe IncrementEvent',
      build: () => HomeBloc(),
      act: (bloc) => bloc.add(IncrementEvent()),
      expect: () => [
        isA<HomeState>().having((s) => s.counter, 'counter', 1),
      ],
    );

    blocTest<HomeBloc, HomeState>(
      'emite 1, 2 e 3 com três incrementos',
      build: () => HomeBloc(),
      act: (bloc) => bloc
        ..add(IncrementEvent())
        ..add(IncrementEvent())
        ..add(IncrementEvent()),
      expect: () => [
        isA<HomeState>().having((s) => s.counter, 'counter', 1),
        isA<HomeState>().having((s) => s.counter, 'counter', 2),
        isA<HomeState>().having((s) => s.counter, 'counter', 3),
      ],
    );
  });
}
```

### Conceitos aprendidos:
- `blocTest`: cria o BLoC (`build`), envia eventos (`act`) e verifica os estados emitidos (`expect`)
- `isA<T>().having(...)`: verifica um campo específico do estado
- `seed:` (opcional): define um estado inicial diferente do padrão

> 💡 Se o estado estender `Equatable` (como na versão avançada do Capítulo 5), dá para comparar diretamente: `expect: () => [const HomeLoaded(1)]`. Sem `Equatable`, dois objetos com os mesmos valores **não** são considerados iguais, e por isso usamos `having`.

### Testando um BLoC com dependências

Para o `PostsBloc` do Capítulo 6, combinamos `bloc_test` com `mocktail`:

```dart
class MockPostRepository extends Mock implements PostRepository {}

void main() {
  late MockPostRepository repository;

  setUp(() => repository = MockPostRepository());

  blocTest<PostsBloc, PostsState>(
    'emite [Carregando, Sucesso] quando a API responde',
    setUp: () => when(() => repository.listar()).thenAnswer((_) async => []),
    build: () => PostsBloc(repository),
    act: (bloc) => bloc.add(PostsCarregados()),
    expect: () => [isA<PostsCarregando>(), isA<PostsSucesso>()],
    verify: (_) => verify(() => repository.listar()).called(1),
  );

  blocTest<PostsBloc, PostsState>(
    'emite [Carregando, Erro] quando a API falha',
    setUp: () => when(() => repository.listar()).thenThrow(Exception('offline')),
    build: () => PostsBloc(repository),
    act: (bloc) => bloc.add(PostsCarregados()),
    expect: () => [isA<PostsCarregando>(), isA<PostsErro>()],
  );
}
```

## 🖼 Passo 8: Golden Tests

Um **golden test** tira uma "foto" do widget e compara com uma imagem de referência salva no projeto. Se algo mudar visualmente, o teste falha.

```dart
testWidgets('tela do contador tem a aparência esperada', (tester) async {
  await tester.pumpWidget(const MaterialApp(home: ContadorPage()));

  await expectLater(
    find.byType(ContadorPage),
    matchesGoldenFile('goldens/contador_page.png'),
  );
});
```

```bash
# Gera (ou atualiza) as imagens de referência
flutter test --update-goldens

# Execuções seguintes comparam com as imagens salvas
flutter test
```

> ⚠️ Fontes e renderização variam entre sistemas operacionais. Gere os goldens no mesmo sistema usado na integração contínua para evitar falhas falsas.

## 🤖 Passo 9: Testes de Integração

Testes de integração rodam o **app de verdade** em um emulador ou celular, simulando um usuário:

```dart
// integration_test/app_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:integration_test/integration_test.dart';
import 'package:meu_app/main.dart' as app;

void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  testWidgets('fluxo completo: incrementar o contador três vezes', (tester) async {
    app.main();
    await tester.pumpAndSettle();

    for (var i = 0; i < 3; i++) {
      await tester.tap(find.byIcon(Icons.add));
      await tester.pumpAndSettle();
    }

    expect(find.text('3'), findsOneWidget);
  });
}
```

```bash
# Com um emulador aberto ou celular conectado
flutter test integration_test/app_test.dart
```

### Conceitos aprendidos:
- `IntegrationTestWidgetsFlutterBinding.ensureInitialized()`: obrigatório no início
- `app.main()`: inicia o app real, com banco, rotas e tudo mais
- A API (`tap`, `find`, `expect`) é a mesma dos testes de widget

## 📊 Cobertura e Integração Contínua

### Cobertura de código

A cobertura mostra quais linhas do código foram executadas pelos testes:

```bash
flutter test --coverage
```

O resultado fica em `coverage/lcov.info`. Para ver um relatório em HTML (Linux/macOS, requer o pacote `lcov`):

```bash
genhtml coverage/lcov.info -o coverage/html
# Abra coverage/html/index.html no navegador
```

> 💡 Cobertura de 100% não é o objetivo. Priorize testar regras de negócio, conversões de dados e fluxos críticos (login, pagamento).

### Rodando os testes no GitHub Actions

Crie `.github/workflows/testes.yml` no projeto do app:

```yaml
name: Testes

on: [push, pull_request]

jobs:
  testes:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          channel: stable
      - run: flutter pub get
      - run: flutter analyze
      - run: flutter test --coverage
```

A cada `push`, o GitHub roda a análise de código e os testes, e avisa se algo quebrou.

## ✅ Melhores Práticas

- **Um comportamento por teste:** se o teste falhar, o nome dele já diz o que quebrou.
- **Nomes descritivos:** `'retorna erro quando o CEP tem menos de 8 dígitos'` é melhor que `'teste 3'`.
- **Siga Arrange, Act, Assert.**
- **Testes independentes:** um teste nunca deve depender do resultado de outro. Use `setUp`.
- **Nada de internet ou banco real** em testes de unidade: use mocks.
- **Escreva o teste ao corrigir um bug:** primeiro um teste que reproduz o bug, depois a correção. Assim ele nunca volta.
- **Rode os testes sempre**, de preferência automaticamente no GitHub Actions.

## 📖 Recursos Adicionais

- [Documentação oficial: Testes no Flutter](https://docs.flutter.dev/testing/overview)
- [Cookbook: Testes de unidade](https://docs.flutter.dev/cookbook/testing/unit/introduction)
- [Cookbook: Testes de widget](https://docs.flutter.dev/cookbook/testing/widget/introduction)
- [Testes de integração](https://docs.flutter.dev/testing/integration-tests)
- [Pacote mocktail](https://pub.dev/packages/mocktail)
- [Pacote bloc_test](https://pub.dev/packages/bloc_test)
