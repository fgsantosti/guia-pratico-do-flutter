# Apêndice F - Dart Essencial

## 📋 Índice

1. [Introdução](#-introdução)
2. [Variáveis e Tipos](#-variáveis-e-tipos)
3. [Null Safety](#-null-safety)
4. [Strings](#-strings)
5. [Operadores e Controle de Fluxo](#-operadores-e-controle-de-fluxo)
6. [Funções](#-funções)
7. [Coleções: List, Set e Map](#-coleções-list-set-e-map)
8. [Classes e Objetos](#-classes-e-objetos)
9. [Herança, Interfaces e Mixins](#-herança-interfaces-e-mixins)
10. [Enums, Records e Pattern Matching](#-enums-records-e-pattern-matching)
11. [Programação Assíncrona](#-programação-assíncrona)
12. [Tratamento de Erros](#-tratamento-de-erros)
13. [Recursos Adicionais](#-recursos-adicionais)

## 🚀 Introdução

O Flutter usa a linguagem **Dart**. Se você já programou em Java, JavaScript, C# ou Kotlin, vai se sentir em casa. Este apêndice é uma referência rápida dos recursos de Dart usados no livro.

> 💡 Teste todos os exemplos no [DartPad](https://dartpad.dev), direto no navegador, sem instalar nada.

Todo programa Dart começa pela função `main`:

```dart
void main() {
  print('Olá, Dart!');
}
```

## 📦 Variáveis e Tipos

```dart
// Tipo inferido automaticamente
var nome = 'Ana';          // String
var idade = 30;            // int

// Tipo explícito
String cidade = 'Teresina';
int ano = 2026;
double altura = 1.68;
bool ativo = true;

// final: atribuído uma única vez (em tempo de execução)
final agora = DateTime.now();

// const: valor fixo conhecido em tempo de compilação
const pi = 3.14159;

// dynamic: desliga a verificação de tipo (evite!)
dynamic qualquerCoisa = 10;
qualquerCoisa = 'texto';
```

### Tipos básicos

| Tipo | Exemplo | Descrição |
|------|---------|-----------|
| `int` | `42` | Números inteiros |
| `double` | `3.14` | Números com casas decimais |
| `num` | `42` ou `3.14` | `int` ou `double` |
| `String` | `'texto'` | Texto |
| `bool` | `true` / `false` | Verdadeiro ou falso |
| `List<T>` | `[1, 2, 3]` | Lista ordenada |
| `Set<T>` | `{1, 2, 3}` | Conjunto sem repetição |
| `Map<K, V>` | `{'a': 1}` | Chave → valor |

### final x const

| | `final` | `const` |
|---|---------|---------|
| Pode mudar depois? | ❌ | ❌ |
| Valor conhecido na compilação? | Não precisa | ✅ Obrigatório |
| Exemplo | `final hoje = DateTime.now();` | `const taxa = 0.1;` |

> 💡 **Regra prática:** use `final` por padrão, `const` quando puder e `var` só quando o valor realmente for mudar.

## 🛡 Null Safety

No Dart, por padrão, **nenhuma variável pode ser nula**. Para permitir `null`, adicione `?` ao tipo:

```dart
String nome = 'Ana';
// nome = null;              // ❌ Erro de compilação

String? apelido;             // ✅ Pode ser nulo (começa como null)
apelido = 'Aninha';
```

### Operadores de nulo

```dart
String? apelido;

// ?. : acesso seguro (retorna null se apelido for null)
print(apelido?.length);              // null

// ?? : valor padrão se for null
print(apelido ?? 'Sem apelido');     // Sem apelido

// ??= : atribui só se for null
apelido ??= 'Padrão';

// ! : "eu garanto que não é null" (lança erro se for)
print(apelido!.length);              // Use com MUITO cuidado
```

### Promoção de tipo

Depois de verificar, o Dart entende que o valor não é nulo:

```dart
void saudar(String? nome) {
  if (nome == null) {
    print('Olá, visitante!');
    return;
  }
  print('Olá, ${nome.toUpperCase()}!'); // Aqui nome já é String
}
```

### late

```dart
class Tela {
  late final String titulo;  // Será inicializado depois, antes do primeiro uso

  void iniciar() {
    titulo = 'Início';
  }
}
```

> ⚠️ Ler uma variável `late` antes de atribuir um valor lança `LateInitializationError`.

## 🔤 Strings

```dart
final nome = 'Maria';
final idade = 25;

// Interpolação
print('Olá, $nome!');                      // Olá, Maria!
print('Ano que vem: ${idade + 1} anos');   // Expressões vão entre chaves

// Várias linhas
final texto = '''
Linha 1
Linha 2
''';

// String "crua" (sem escapes), útil para expressões regulares
final regex = r'\d+';

// Métodos úteis
'  flutter  '.trim();           // 'flutter'
'flutter'.toUpperCase();         // 'FLUTTER'
'flutter dart'.split(' ');       // ['flutter', 'dart']
'flutter'.contains('lut');       // true
'flutter'.startsWith('flu');     // true
'flutter'.replaceAll('t', 'T');  // 'fluTTer'
'42'.padLeft(5, '0');            // '00042'

// Conversões
int.parse('42');                 // 42
double.tryParse('abc');          // null (não lança erro)
42.toString();                   // '42'
3.14159.toStringAsFixed(2);      // '3.14'
```

## 🔀 Operadores e Controle de Fluxo

### Operadores

```dart
10 ~/ 3;   // 3   divisão inteira
10 % 3;    // 1   resto
10 / 4;    // 2.5 divisão (sempre double)

final maior = idade >= 18 ? 'Adulto' : 'Menor';   // Operador ternário

// Verificação e conversão de tipo
if (valor is String) print(valor.length);
final texto = valor as String;
```

### if, for e while

```dart
if (nota >= 7) {
  print('Aprovado');
} else if (nota >= 5) {
  print('Recuperação');
} else {
  print('Reprovado');
}

for (var i = 0; i < 3; i++) {
  print(i);
}

for (final fruta in ['maçã', 'banana']) {
  print(fruta);
}

var contador = 0;
while (contador < 3) {
  contador++;
}
```

### switch moderno (Dart 3)

```dart
final dia = 6;

final tipo = switch (dia) {
  1 || 7 => 'Fim de semana',
  >= 2 && <= 6 => 'Dia útil',
  _ => 'Dia inválido',
};
```

## 🔧 Funções

```dart
// Função comum
int somar(int a, int b) {
  return a + b;
}

// Arrow function (uma única expressão)
int multiplicar(int a, int b) => a * b;

// Parâmetros nomeados (entre chaves): muito usados no Flutter
void criarUsuario({required String nome, int idade = 18, String? email}) {
  print('$nome, $idade anos, ${email ?? "sem e-mail"}');
}

criarUsuario(nome: 'Ana');
criarUsuario(nome: 'Bruno', idade: 30, email: 'b@email.com');

// Parâmetros posicionais opcionais (entre colchetes)
String saudar(String nome, [String saudacao = 'Olá']) => '$saudacao, $nome!';

saudar('Ana');            // Olá, Ana!
saudar('Ana', 'Oi');      // Oi, Ana!
```

### Funções como valores

```dart
// Uma função pode ser guardada em variável e passada como parâmetro
void executar(void Function() acao) => acao();

executar(() => print('Executado!'));

// É exatamente isso que fazemos no Flutter:
// ElevatedButton(onPressed: () => print('Clicou'), ...)
```

## 📚 Coleções: List, Set e Map

### List

```dart
final frutas = ['maçã', 'banana', 'uva'];

frutas.add('laranja');
frutas.remove('banana');
frutas.length;              // 3
frutas.first;               // 'maçã'
frutas.contains('uva');     // true
frutas.indexOf('uva');      // 1
```

### Métodos funcionais (muito usados!)

```dart
final numeros = [1, 2, 3, 4, 5];

numeros.map((n) => n * 2).toList();          // [2, 4, 6, 8, 10]
numeros.where((n) => n.isEven).toList();     // [2, 4]
numeros.any((n) => n > 4);                   // true
numeros.every((n) => n > 0);                 // true
numeros.firstWhere((n) => n > 2);            // 3
numeros.fold(0, (soma, n) => soma + n);      // 15
numeros.reduce((a, b) => a > b ? a : b);     // 5 (maior)
numeros.take(2).toList();                    // [1, 2]
numeros.skip(3).toList();                    // [4, 5]

// Ordenar
final nomes = ['Carla', 'Ana', 'Bruno']..sort();   // [Ana, Bruno, Carla]
```

### Spread e collection if/for

```dart
final base = [1, 2];
final completa = [...base, 3, 4];                    // [1, 2, 3, 4]

final logado = true;
final menu = [
  'Início',
  if (logado) 'Perfil',                              // Só entra se logado
  for (final n in [1, 2]) 'Item $n',                 // Gera itens
];
```

No Flutter, isso permite montar listas de widgets de forma condicional:

```dart
Column(
  children: [
    const Text('Título'),
    if (carregando) const CircularProgressIndicator(),
    ...itens.map((i) => Text(i)),
  ],
)
```

### Set e Map

```dart
final tags = {'flutter', 'dart', 'flutter'};   // {'flutter', 'dart'}: sem repetição

final usuario = {
  'nome': 'Ana',
  'idade': 30,
};

usuario['email'] = 'ana@email.com';             // Adiciona
print(usuario['nome']);                          // Ana
usuario.containsKey('idade');                    // true
usuario.remove('idade');

for (final MapEntry(:key, :value) in usuario.entries) {
  print('$key: $value');
}
```

## 🏗 Classes e Objetos

```dart
class Produto {
  // Atributos
  final String nome;
  final double preco;
  int _estoque; // "_" torna privado ao arquivo

  // Construtor com parâmetros nomeados
  Produto({required this.nome, required this.preco, int estoque = 0})
      : _estoque = estoque;

  // Construtor nomeado
  Produto.gratis(this.nome) : preco = 0, _estoque = 0;

  // Getter
  int get estoque => _estoque;
  bool get disponivel => _estoque > 0;

  // Método
  void vender(int quantidade) {
    if (quantidade > _estoque) throw StateError('Estoque insuficiente');
    _estoque -= quantidade;
  }

  // copyWith: cria uma cópia alterando alguns campos (padrão imutável)
  Produto copyWith({String? nome, double? preco}) =>
      Produto(nome: nome ?? this.nome, preco: preco ?? this.preco, estoque: _estoque);

  @override
  String toString() => 'Produto($nome, R\$ $preco)';
}

void main() {
  final caneta = Produto(nome: 'Caneta', preco: 2.5, estoque: 10);
  caneta.vender(3);
  print(caneta.estoque);       // 7

  final cara = caneta.copyWith(preco: 5);
  print(cara);                 // Produto(Caneta, R$ 5.0)
}
```

### Operador cascata (`..`)

Permite chamar vários métodos no mesmo objeto:

```dart
final lista = <int>[]
  ..add(1)
  ..add(2)
  ..addAll([3, 4]);
```

### factory

```dart
class Usuario {
  final String nome;
  Usuario(this.nome);

  // Construtor que pode ter lógica (ex.: converter JSON)
  factory Usuario.fromJson(Map<String, dynamic> json) => Usuario(json['nome'] ?? '');
}
```

## 🧬 Herança, Interfaces e Mixins

```dart
// Classe abstrata: não pode ser instanciada
abstract class Animal {
  final String nome;
  Animal(this.nome);

  void emitirSom(); // Método abstrato: as filhas devem implementar

  void apresentar() => print('Eu sou $nome');
}

// Herança com extends
class Cachorro extends Animal {
  Cachorro(super.nome);

  @override
  void emitirSom() => print('Au au!');
}

// Interface com implements: deve implementar TUDO
abstract interface class Pagavel {
  double calcularValor();
}

class Servico implements Pagavel {
  @override
  double calcularValor() => 100;
}

// Mixin: "pacote" de comportamentos reutilizáveis
mixin Voador {
  void voar() => print('Voando!');
}

class Passaro extends Animal with Voador {
  Passaro(super.nome);

  @override
  void emitirSom() => print('Piu piu!');
}
```

No Flutter, você vê isso o tempo todo:

```dart
class MeuWidget extends StatelessWidget { ... }                      // Herança
class _TelaState extends State<Tela> with SingleTickerProviderStateMixin { ... } // Mixin
class MockApi extends Mock implements ApiClient {}                   // Interface (testes)
```

## 🏷 Enums, Records e Pattern Matching

### Enums

```dart
enum Status { pendente, enviado, entregue }

// Enum com valores e métodos
enum Prioridade {
  baixa('Baixa', 1),
  media('Média', 2),
  alta('Alta', 3);

  const Prioridade(this.rotulo, this.peso);
  final String rotulo;
  final int peso;
}

print(Prioridade.alta.rotulo);   // Alta
print(Status.enviado.name);      // enviado
```

### Records: retornar vários valores

```dart
(double, double) coordenadas() => (-5.09, -42.80);

({String nome, int idade}) dadosUsuario() => (nome: 'Ana', idade: 30);

final (lat, lng) = coordenadas();
final usuario = dadosUsuario();
print('${usuario.nome} tem ${usuario.idade} anos');
```

### Classes seladas e pattern matching

```dart
sealed class Resultado {}

class Sucesso extends Resultado {
  Sucesso(this.dados);
  final String dados;
}

class Falha extends Resultado {
  Falha(this.erro);
  final String erro;
}

String descrever(Resultado r) => switch (r) {
      Sucesso(:final dados) => 'OK: $dados',
      Falha(:final erro) => 'Erro: $erro',
      // Sem "default": o compilador garante que todos os casos foram tratados
    };
```

É a mesma técnica usada nos estados do BLoC (Capítulo 5 e 6).

## ⏳ Programação Assíncrona

Operações demoradas (internet, banco de dados, arquivos) não podem travar a tela. Por isso, elas retornam um `Future`.

### Future com async/await

```dart
Future<String> buscarNome() async {
  await Future.delayed(const Duration(seconds: 2)); // Simula a espera
  return 'Maria';
}

Future<void> main() async {
  print('Buscando...');
  final nome = await buscarNome();   // Espera sem travar
  print('Nome: $nome');
}
```

### Várias tarefas em paralelo

```dart
final resultados = await Future.wait([
  buscarUsuario(),
  buscarPedidos(),
  buscarNotificacoes(),
]);
// Tempo total = o da tarefa mais lenta, e não a soma de todas
```

### Stream: vários valores ao longo do tempo

```dart
Stream<int> contagem() async* {
  for (var i = 3; i >= 1; i--) {
    await Future.delayed(const Duration(seconds: 1));
    yield i; // Emite um valor
  }
}

Future<void> main() async {
  await for (final n in contagem()) {
    print(n); // 3... 2... 1...
  }

  // Ou escutando:
  contagem().listen((n) => print('Recebi $n'));
}
```

| | `Future` | `Stream` |
|---|----------|----------|
| Quantos valores | Um | Vários, ao longo do tempo |
| Exemplo | Resposta de uma API | Mensagens de chat, localização GPS, Firestore |
| No Flutter | `FutureBuilder` | `StreamBuilder` |

## 🧯 Tratamento de Erros

```dart
Future<void> carregar() async {
  try {
    final dados = await buscarDados();
    print(dados);
  } on FormatException catch (e) {
    print('Formato inválido: ${e.message}');   // Erro específico
  } catch (e, stackTrace) {
    print('Erro inesperado: $e');               // Qualquer outro erro
    print(stackTrace);
  } finally {
    print('Sempre executa');                    // Ex.: esconder o carregamento
  }
}

// Lançando erros
void sacar(double valor, double saldo) {
  if (valor > saldo) {
    throw Exception('Saldo insuficiente');
  }
}

// Exceção própria
class SaldoInsuficienteException implements Exception {
  final double faltando;
  SaldoInsuficienteException(this.faltando);

  @override
  String toString() => 'Faltam R\$ ${faltando.toStringAsFixed(2)}';
}
```

## 📖 Recursos Adicionais

- [Tour da linguagem Dart](https://dart.dev/language)
- [Null safety](https://dart.dev/null-safety)
- [Programação assíncrona: Futures, async e await](https://dart.dev/libraries/async/async-await)
- [Patterns e records](https://dart.dev/language/patterns)
- [Effective Dart: guia de estilo](https://dart.dev/effective-dart)
- [DartPad](https://dartpad.dev)
