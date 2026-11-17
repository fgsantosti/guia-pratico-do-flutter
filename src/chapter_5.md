# Capítulo 5 - Gerenciamento de Estado com BLoC

## 📖 Sobre Este Tutorial

Este tutorial é baseado no excelente artigo do **Gabriel Caires** publicado no Medium da comunidade Flutter Brasil. Aqui você aprenderá a implementar o padrão BLoC (Business Logic Component) no Flutter de forma prática e organizada.

**Artigo Original:** [Flutter: Gerenciamento de estado com BloC 🛡️](https://medium.com/brasilflutter/flutter-gerenciamento-de-estado-com-bloc-%EF%B8%8F-4e23bd4955bd)

---

## 🎯 O que Você Vai Aprender

- ✅ Conceitos fundamentais do padrão BLoC
- ✅ Diferença entre BLoC (conceito) e flutter_bloc (pacote)
- ✅ Vantagens e desvantagens do BLoC
- ✅ Implementação prática passo a passo
- ✅ Estrutura de arquivos recomendada
- ✅ Exemplo completo de contador com BLoC

---

## 🤔 O que é BLoC?

**BLoC** é a sigla para **Business Logic Component**.

É um padrão de arquitetura usado no Flutter para **separar a lógica da interface (UI)**.

### 🔄 Como Funciona

Em vez de misturar tudo em um único widget com `setState()`, o BLoC organiza seu código em:

- **📥 Eventos**: o que o usuário faz (ex: clicar em um botão)
- **📤 Estados**: o que deve ser exibido na tela depois de processar aquele evento
- **⚙️ BLoC**: a ponte entre os dois — ele recebe o evento, processa a lógica e emite um novo estado

### 🎯 Benefícios da Separação

- Testar sua lógica separadamente
- Reutilizar o mesmo BLoC em várias telas
- Manter o código limpo e bem organizado

---

## 📦 O que é o Pacote `flutter_bloc`?

O `flutter_bloc` é um **pacote oficial mantido pela equipe do BLoC** que fornece tudo que você precisa para aplicar esse padrão no Flutter de forma prática.

### 🛠 O que Ele Oferece

- **Widgets prontos**: `BlocBuilder`, `BlocProvider`, `BlocListener`
- Integração com `Stream` e `InheritedWidget`
- Suporte para testes
- Estrutura recomendada para organizar arquivos

> **💡 Resumo:** 
> - **BLoC** = conceito/padrão
> - **flutter_bloc** = pacote que implementa o conceito

---

## ✅ Vantagens do BLoC

### 🎯 **Separação entre UI e Lógica**
A interface (UI) fica separada da lógica de negócio. Código mais limpo e organizado.

### 📈 **Código Organizado e Escalável**
Estrutura clara: eventos, estados e lógica bem separados. Facilita manutenção e crescimento.

### ♻ **Lógica Reutilizável**
Reaproveite o mesmo BLoC em várias partes do app ou outros projetos.

### 🧪 **Fácil de Testar**
Lógica fora da interface permite testes automatizados simples.

### 🔄 **Fluxo Previsível**
Ciclo claro: evento → lógica → novo estado. Comportamento confiável.

### 👥 **Comunidade Ativa**
Pacote oficial, bem documentado e usado profissionalmente.

---

## ❌ Desvantagens do BLoC

### 📁 **Mais Arquivos**
Exige arquivos separados para eventos, estados e bloco.

### 📚 **Curva de Aprendizado**
Conceitos de eventos, estados e Streams podem confundir no início.

### 📝 **Código Verboso**
Mais código repetitivo para estruturar a lógica.

---

## 🚀 Implementação Passo a Passo

### 1⃣ **Adicionar Dependência**

Adicione no `pubspec.yaml`:

```yaml
dependencies:
  flutter_bloc: ^9.1.1
```

### 2⃣ **Estrutura de Arquivos Recomendada**

```
lib/
├── main.dart
├── app.dart
├── features/
│   └── home/
│       ├── bloc/
│       │   ├── home_bloc.dart
│       │   ├── home_event.dart
│       │   └── home_state.dart
│       └── view/
│           └── home_page.dart
```

### 3⃣ **Definindo os Eventos**

Crie `features/home/bloc/home_event.dart`:

```dart
// Define uma classe selada que serve como base para todos os eventos do HomeBloc
sealed class HomeEvent {} // Usar `sealed` ajuda o compilador a garantir que todos os eventos são tratados

// Evento específico que representa a ação de incrementar o contador
final class IncrementEvent extends HomeEvent {}
```

**💡 Por que `sealed class`?**
- Garante que todos os eventos sejam tratados
- Melhora a análise estática do código
- Previne criação de subclasses não intencionais

### 4⃣ **Definindo os Estados**

Crie `features/home/bloc/home_state.dart`:

```dart
// Define uma classe imutável para representar o estado do HomeBloc
final class HomeState {
  // Valor do contador atual
  final int counter;

  // Construtor constante (permite comparar por identidade em tempo de compilação)
  const HomeState(this.counter);
}
```

**💡 Por que `final class`?**
- Impede herança desnecessária
- Melhora performance
- Deixa a intenção do código mais clara

### 5⃣ **Implementando o BLoC**

Crie `features/home/bloc/home_bloc.dart`:

```dart
// Importa o pacote flutter_bloc para usar a classe Bloc
import 'package:flutter_bloc/flutter_bloc.dart';

// Importa os eventos e estados do HomeBloc
import 'home_event.dart';
import 'home_state.dart';

// HomeBloc: responsável por mapear eventos para estados
class HomeBloc extends Bloc<HomeEvent, HomeState> {
  // Construtor: define o estado inicial como HomeState(0)
  HomeBloc() : super(const HomeState(0)) {
    // Define o que acontece quando um IncrementEvent é recebido
    on<IncrementEvent>((event, emit) {
      emit(
        HomeState(state.counter + 1),
      ); // Incrementa o valor atual e emite novo estado
    });
  }
}
```

**🔍 Explicação do Código:**
- `super(const HomeState(0))`: Define estado inicial com contador = 0
- `on<IncrementEvent>()`: Registra handler para o evento
- `emit()`: Emite novo estado para a UI

### 6⃣ **Criando a Interface**

Crie `features/home/view/home_page.dart`:

```dart
// Importações essenciais
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

// Importações locais do BLoC
import '../bloc/home_bloc.dart';
import '../bloc/home_event.dart';
import '../bloc/home_state.dart';

// Widget de tela inicial (HomePage)
class HomePage extends StatelessWidget {
  const HomePage({
    super.key,
    required this.title,
  }); // Construtor com título obrigatório

  final String title; // Título da página, exibido no AppBar

  @override
  Widget build(BuildContext context) {
    // Recupera o HomeBloc do contexto (injeção feita no main.dart)
    final bloc = context.read<HomeBloc>();

    return Scaffold(
      appBar: AppBar(
        // Usa a cor do tema como fundo da AppBar
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        title: Text(title), // Exibe o título no topo
      ),
      body: Center(
        // Centraliza a coluna na tela
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center, // Centraliza verticalmente
          children: [
            const Text('You have pushed the button this many times:'),

            // Usa BlocBuilder para escutar o estado do HomeBloc
            BlocBuilder<HomeBloc, HomeState>(
              builder: (context, state) {
                return Text(
                  '${state.counter}', // Exibe o valor do contador
                  style: Theme.of(context).textTheme.headlineMedium, // Estilo do texto
                );
              },
            ),
          ],
        ),
      ),
      // Botão flutuante que dispara o evento de incremento
      floatingActionButton: FloatingActionButton(
        onPressed: () => bloc.add(IncrementEvent()), // Adiciona evento ao BLoC
        tooltip: 'Increment',
        child: const Icon(Icons.add), // Ícone de adição no botão
      ),
    );
  }
}
```

**🔍 Widgets Importantes:**
- `context.read<HomeBloc>()`: Obtém instância do BLoC
- `BlocBuilder`: Reconstrói UI quando estado muda
- `bloc.add()`: Adiciona evento ao BLoC

### 7⃣ **Configurando o App**

Crie `app.dart`:

```dart
// Importa os widgets do Flutter
import 'package:flutter/material.dart';

// Importa a página inicial da aplicação
import 'features/home/view/home_page.dart';

// MyApp é o widget raiz da aplicação, configurando o tema e a rota inicial
class MyApp extends StatelessWidget {
  const MyApp({
    super.key,
  }); // Construtor constante e com chave, bom para performance

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter BLoC App', // Título do app (usado em algumas plataformas)
      theme: ThemeData(
        // Define o tema visual da aplicação
        colorScheme: ColorScheme.fromSeed(
          seedColor: Colors.deepPurple,
        ), // Geração de esquema de cores baseado em uma cor base
      ),
      home: const HomePage(
        title: 'Flutter Demo Home Page',
      ), // Página inicial do app
    );
  }
}
```

### 8⃣ **Configurando o Main**

Crie/edite `main.dart`:

```dart
// Importa os widgets e componentes visuais do Flutter
import 'package:flutter/material.dart';

// Importa a biblioteca flutter_bloc, que permite usar o padrão BLoC com o Flutter
import 'package:flutter_bloc/flutter_bloc.dart';

// Importa o HomeBloc, responsável pela lógica de negócios da tela inicial
import 'features/home/bloc/home_bloc.dart';

// Importa o widget principal do app, que contém a configuração visual e as rotas
import 'app.dart';

void main() {
  // Função principal que inicializa o aplicativo
  runApp(
    // Injetando o HomeBloc no contexto da aplicação com BlocProvider
    MultiBlocProvider(
      providers: [
        // Cada item da lista é um BlocProvider responsável por fornecer um BLoC específico
        BlocProvider<HomeBloc>(
          create: (_) => HomeBloc(), // Cria e fornece uma instância de HomeBloc
        ),

        // Exemplos adicionais poderiam ser adicionados aqui, como:
        // BlocProvider<AuthenticationBloc>(create: (_) => AuthenticationBloc()),
        // BlocProvider<ThemeBloc>(create: (_) => ThemeBloc()),
      ],
      child: const MyApp(), // Widget principal da aplicação
    ),
  );
}
```

**🔍 Injeção de Dependência:**
- `MultiBlocProvider`: Fornece múltiplos BLoCs
- `BlocProvider`: Disponibiliza BLoC para widgets filhos
- `create: (_) => HomeBloc()`: Factory function para criar instância

---

## 🎯 Fluxo de Funcionamento

### 1. **Usuário Interage** 👆
```dart
FloatingActionButton(
  onPressed: () => bloc.add(IncrementEvent()),
  child: const Icon(Icons.add),
)
```

### 2. **Evento é Processado** ⚙
```dart
on<IncrementEvent>((event, emit) {
  emit(HomeState(state.counter + 1));
});
```

### 3. **Estado é Atualizado** 🔄
```dart
BlocBuilder<HomeBloc, HomeState>(
  builder: (context, state) {
    return Text('${state.counter}');
  },
)
```

### 4. **UI é Reconstruída** 🎨
A interface atualiza automaticamente com o novo valor.

---

## 🧪 Testando o BLoC

### Exemplo de Teste Unitário

```dart
import 'package:bloc_test/bloc_test.dart';
import 'package:flutter_test/flutter_test.dart';

import '../lib/features/home/bloc/home_bloc.dart';
import '../lib/features/home/bloc/home_event.dart';
import '../lib/features/home/bloc/home_state.dart';

void main() {
  group('HomeBloc', () {
    late HomeBloc homeBloc;

    setUp(() {
      homeBloc = HomeBloc();
    });

    tearDown(() {
      homeBloc.close();
    });

    test('estado inicial deve ser HomeState(0)', () {
      expect(homeBloc.state, const HomeState(0));
    });

    blocTest<HomeBloc, HomeState>(
      'deve emitir [HomeState(1)] quando IncrementEvent é adicionado',
      build: () => homeBloc,
      act: (bloc) => bloc.add(IncrementEvent()),
      expect: () => [const HomeState(1)],
    );

    blocTest<HomeBloc, HomeState>(
      'deve emitir [HomeState(1), HomeState(2)] quando IncrementEvent é adicionado duas vezes',
      build: () => homeBloc,
      act: (bloc) {
        bloc.add(IncrementEvent());
        bloc.add(IncrementEvent());
      },
      expect: () => [const HomeState(1), const HomeState(2)],
    );
  });
}
```

---

## 🚀 Executando o Projeto

### 1. **Clone o Repositório de Exemplo**
```bash
git clone https://github.com/GabrielCairesDev/flutter_exemplo_bloc
cd flutter_exemplo_bloc
```

### 2. **Instale as Dependências**
```bash
flutter pub get
```

### 3. **Execute o App**
```bash
flutter run
```

### 4. **Execute os Testes**
```bash
flutter test
```

---

## 📚 Conceitos Avançados

### 🎯 **BlocListener vs BlocBuilder**

```dart
// BlocBuilder: Reconstrói UI quando estado muda
BlocBuilder<HomeBloc, HomeState>(
  builder: (context, state) {
    return Text('${state.counter}');
  },
)

// BlocListener: Executa ação quando estado muda (sem reconstruir)
BlocListener<HomeBloc, HomeState>(
  listener: (context, state) {
    if (state.counter == 10) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Chegou a 10!')),
      );
    }
  },
  child: MyWidget(),
)

// BlocConsumer: Combina Builder + Listener
BlocConsumer<HomeBloc, HomeState>(
  listener: (context, state) {
    // Lógica de side effects
  },
  builder: (context, state) {
    // Constrói UI
    return Text('${state.counter}');
  },
)
```

### 🔄 **Estados Mais Complexos**

```dart
sealed class HomeState extends Equatable {
  const HomeState();

  @override
  List<Object> get props => [];
}

final class HomeInitial extends HomeState {}

final class HomeLoading extends HomeState {}

final class HomeLoaded extends HomeState {
  final int counter;

  const HomeLoaded(this.counter);

  @override
  List<Object> get props => [counter];
}

final class HomeError extends HomeState {
  final String message;

  const HomeError(this.message);

  @override
  List<Object> get props => [message];
}
```

### 📥 **Eventos Mais Complexos**

```dart
sealed class HomeEvent extends Equatable {
  const HomeEvent();

  @override
  List<Object> get props => [];
}

final class IncrementEvent extends HomeEvent {}

final class DecrementEvent extends HomeEvent {}

final class ResetEvent extends HomeEvent {}

final class SetCounterEvent extends HomeEvent {
  final int value;

  const SetCounterEvent(this.value);

  @override
  List<Object> get props => [value];
}
```

---

## 🎯 Quando Usar BLoC?

### ✅ **Use BLoC Quando:**
- Projeto grande e complexo
- Equipe com múltiplos desenvolvedores
- Necessidade de testes extensivos
- Lógica de negócio complexa
- Reutilização de lógica entre telas

### ❌ **Evite BLoC Quando:**
- Projeto simples e pequeno
- Prototipagem rápida
- Equipe pequena ou desenvolvedor solo
- Lógica simples de UI

### 🔄 **Alternativas Mais Simples:**
- `setState()` para estado local
- `ValueNotifier` para estado reativo simples
- `ChangeNotifier` com Provider
- `Riverpod` para casos intermediários

---

## 🛠 Ferramentas Úteis

### 📦 **Pacotes Complementares**
```yaml
dependencies:
  flutter_bloc: ^9.1.1
  equatable: ^2.0.5  # Para comparação de estados

dev_dependencies:
  bloc_test: ^9.1.4   # Para testes de BLoC
  mocktail: ^1.0.0    # Para mocks em testes
```

### 🔧 **Extensões VS Code**
- **Bloc**: Snippets e templates
- **Flutter Bloc Snippets**: Código boilerplate
- **Awesome Flutter Snippets**: Snippets gerais

### 🎯 **Bloc DevTools**
```dart
import 'package:bloc/bloc.dart';

class SimpleBlocObserver extends BlocObserver {
  @override
  void onEvent(BlocBase bloc, Object? event) {
    super.onEvent(bloc, event);
    print('${bloc.runtimeType} $event');
  }

  @override
  void onTransition(BlocBase bloc, Transition transition) {
    super.onTransition(bloc, transition);
    print(transition);
  }

  @override
  void onError(BlocBase bloc, Object error, StackTrace stackTrace) {
    super.onError(bloc, error, stackTrace);
    print('${bloc.runtimeType} $error $stackTrace');
  }
}

void main() {
  Bloc.observer = SimpleBlocObserver();
  runApp(MyApp());
}
```

---

## 📖 Recursos Adicionais

### 📚 **Documentação Oficial**
- [BLoC Library](https://bloclibrary.dev/)
- [Flutter BLoC Package](https://pub.dev/packages/flutter_bloc)
- [BLoC Test Package](https://pub.dev/packages/bloc_test)

### 🎥 **Vídeos Recomendados**
- [BLoC Pattern - Official Flutter Channel](https://www.youtube.com/watch?v=PLHln7wHgPE)
- [Flutter BLoC Tutorial - ResoCoder](https://www.youtube.com/watch?v=LeLrsnHeCZY)

### 📱 **Projetos de Exemplo**
- [Repositório Oficial BLoC](https://github.com/felangel/bloc/tree/master/examples)
- [Flutter Samples](https://github.com/flutter/samples)
- [Exemplo do Tutorial](https://github.com/GabrielCairesDev/flutter_exemplo_bloc)

---

## 🎯 Conclusão

O BLoC é uma ferramenta poderosa para gerenciamento de estado em Flutter, especialmente útil em projetos grandes e complexos. Embora tenha uma curva de aprendizado inicial, os benefícios de organização, testabilidade e escalabilidade fazem valer o investimento.

### 💡 **Principais Takeaways:**

1. **BLoC separa lógica de UI** - Código mais limpo e testável
2. **Fluxo previsível** - Eventos → Lógica → Estados
3. **Ótimo para projetos grandes** - Escalabilidade e manutenibilidade
4. **Comunidade ativa** - Suporte e recursos abundantes
5. **Não é sempre necessário** - Avalie a complexidade do projeto

### 🚀 **Próximos Passos:**

1. Pratique com o exemplo do contador
2. Implemente um CRUD simples com BLoC
3. Explore BlocListener e BlocConsumer
4. Aprenda sobre Cubit (versão simplificada)
5. Estude arquiteturas como Clean Architecture com BLoC

---

## 👨‍💻 Créditos

**Tutorial baseado no artigo de:** Gabriel Caires  
**Artigo original:** [Flutter: Gerenciamento de estado com BloC 🛡️](https://medium.com/brasilflutter/flutter-gerenciamento-de-estado-com-bloc-%EF%B8%8F-4e23bd4955bd)  
**Comunidade:** [Flutter Brasil](https://medium.com/brasilflutter)  
**Repositório de exemplo:** [flutter_exemplo_bloc](https://github.com/GabrielCairesDev/flutter_exemplo_bloc)

---

**Versão:** 1.0  
**Última Atualização:** Novembro 2024  
**Flutter Version:** 3.16+  
**Dart Version:** 3.2+

---

*Comece sua jornada com BLoC hoje mesmo! 🚀*