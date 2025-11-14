# Enviando Dados para uma Nova Tela

## 📋 Índice

1. [Introdução](#introdução)
2. [Conceitos Fundamentais](#conceitos-fundamentais)
3. [Método 1: Passagem Direta via Construtor](#método-1-passagem-direta-via-construtor)
4. [Método 2: Usando RouteSettings](#método-2-usando-routesettings)
5. [Método 3: Named Routes com Argumentos](#método-3-named-routes-com-argumentos)
6. [Tipos de Dados Suportados](#tipos-de-dados-suportados)
7. [Exemplos Práticos Avançados](#exemplos-práticos-avançados)
8. [Melhores Práticas](#melhores-práticas)
9. [Tratamento de Erros](#tratamento-de-erros)
10. [Recursos Adicionais](#recursos-adicionais)

## 🚀 Introdução

A navegação entre telas é uma funcionalidade essencial em aplicações Flutter. Frequentemente, precisamos não apenas navegar para uma nova tela, mas também **passar dados** para ela. Este guia abrange todas as formas de enviar dados entre telas no Flutter.

### Cenários Comuns

- 📱 Passar informações de um item selecionado em uma lista
- 👤 Enviar dados do usuário para telas de perfil
- 🛒 Transferir itens do carrinho para checkout
- 📝 Compartilhar dados de formulários entre etapas
- 🔍 Passar parâmetros de busca e filtros

## 🧠 Conceitos Fundamentais

### Navigator e Routes

No Flutter, a navegação funciona como uma **pilha (stack)** de telas:

```dart
// Pilha de navegação
[HomeScreen] ← Tela atual (topo da pilha)
[LoginScreen] ← Tela anterior
[SplashScreen] ← Primeira tela (base da pilha)
```

### Tipos de Navegação

1. **Push Navigation** - Adiciona nova tela à pilha
2. **Pop Navigation** - Remove tela atual da pilha
3. **Replace Navigation** - Substitui tela atual

### Formas de Passar Dados

1. **Construtor direto** - Mais simples e type-safe
2. **RouteSettings** - Flexível, mas requer casting
3. **Named Routes** - Organizado para apps grandes
4. **Global State** - Para dados compartilhados

## 📱 Método 1: Passagem Direta via Construtor

### Vantagens
- ✅ **Type-safe** - Verificação de tipos em tempo de compilação
- ✅ **Simples** - Fácil de entender e implementar
- ✅ **IDE Support** - Autocompletar e refatoração
- ✅ **Menos propenso a erros**

### Exemplo Básico - Lista de Tarefas

#### 1. Definindo o Modelo de Dados

```dart
class Todo {
  final String id;
  final String title;
  final String description;
  final bool isCompleted;
  final DateTime createdAt;
  final Priority priority;

  const Todo({
    required this.id,
    required this.title,
    required this.description,
    this.isCompleted = false,
    required this.createdAt,
    this.priority = Priority.medium,
  });

  Todo copyWith({
    String? id,
    String? title,
    String? description,
    bool? isCompleted,
    DateTime? createdAt,
    Priority? priority,
  }) {
    return Todo(
      id: id ?? this.id,
      title: title ?? this.title,
      description: description ?? this.description,
      isCompleted: isCompleted ?? this.isCompleted,
      createdAt: createdAt ?? this.createdAt,
      priority: priority ?? this.priority,
    );
  }
}

enum Priority { low, medium, high }
```

#### 2. Tela de Lista (Origem)

```dart
class TodoListScreen extends StatelessWidget {
  const TodoListScreen({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    final todos = _generateTodos();

    return Scaffold(
      appBar: AppBar(
        title: const Text('Lista de Tarefas'),
        backgroundColor: Colors.blue,
      ),
      body: ListView.builder(
        padding: const EdgeInsets.all(8.0),
        itemCount: todos.length,
        itemBuilder: (context, index) {
          final todo = todos[index];
          return Card(
            margin: const EdgeInsets.symmetric(vertical: 4.0),
            child: ListTile(
              leading: CircleAvatar(
                backgroundColor: _getPriorityColor(todo.priority),
                child: Text(
                  todo.id,
                  style: const TextStyle(
                    color: Colors.white,
                    fontWeight: FontWeight.bold,
                  ),
                ),
              ),
              title: Text(
                todo.title,
                style: TextStyle(
                  decoration: todo.isCompleted 
                    ? TextDecoration.lineThrough 
                    : null,
                ),
              ),
              subtitle: Text(
                _formatDate(todo.createdAt),
                style: TextStyle(color: Colors.grey[600]),
              ),
              trailing: Icon(
                todo.isCompleted ? Icons.check_circle : Icons.circle_outlined,
                color: todo.isCompleted ? Colors.green : Colors.grey,
              ),
              onTap: () {
                // Navegação com passagem de dados via construtor
                Navigator.push(
                  context,
                  MaterialPageRoute<void>(
                    builder: (context) => TodoDetailScreen(todo: todo),
                  ),
                );
              },
            ),
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          Navigator.push(
            context,
            MaterialPageRoute<void>(
              builder: (context) => const AddTodoScreen(),
            ),
          );
        },
        child: const Icon(Icons.add),
      ),
    );
  }

  List<Todo> _generateTodos() {
    return List.generate(
      15,
      (i) => Todo(
        id: (i + 1).toString(),
        title: 'Tarefa ${i + 1}',
        description: 'Descrição detalhada da tarefa ${i + 1}. '
            'Esta tarefa precisa ser concluída com atenção aos detalhes.',
        isCompleted: i % 3 == 0,
        createdAt: DateTime.now().subtract(Duration(days: i)),
        priority: Priority.values[i % 3],
      ),
    );
  }

  Color _getPriorityColor(Priority priority) {
    switch (priority) {
      case Priority.low:
        return Colors.green;
      case Priority.medium:
        return Colors.orange;
      case Priority.high:
        return Colors.red;
    }
  }

  String _formatDate(DateTime date) {
    return '${date.day}/${date.month}/${date.year}';
  }
}
```

#### 3. Tela de Detalhes (Destino)

```dart
class TodoDetailScreen extends StatelessWidget {
  final Todo todo;

  const TodoDetailScreen({
    Key? key,
    required this.todo,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(todo.title),
        backgroundColor: _getPriorityColor(todo.priority),
        actions: [
          IconButton(
            icon: const Icon(Icons.edit),
            onPressed: () {
              Navigator.push(
                context,
                MaterialPageRoute<void>(
                  builder: (context) => EditTodoScreen(todo: todo),
                ),
              );
            },
          ),
        ],
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Status Card
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16.0),
                child: Row(
                  children: [
                    Icon(
                      todo.isCompleted 
                        ? Icons.check_circle 
                        : Icons.circle_outlined,
                      color: todo.isCompleted ? Colors.green : Colors.grey,
                      size: 32,
                    ),
                    const SizedBox(width: 12),
                    Text(
                      todo.isCompleted ? 'Concluída' : 'Pendente',
                      style: TextStyle(
                        fontSize: 18,
                        fontWeight: FontWeight.bold,
                        color: todo.isCompleted ? Colors.green : Colors.orange,
                      ),
                    ),
                  ],
                ),
              ),
            ),

            const SizedBox(height: 16),

            // Priority Card
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16.0),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    const Text(
                      'Prioridade',
                      style: TextStyle(
                        fontSize: 16,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                    const SizedBox(height: 8),
                    Container(
                      padding: const EdgeInsets.symmetric(
                        horizontal: 12,
                        vertical: 6,
                      ),
                      decoration: BoxDecoration(
                        color: _getPriorityColor(todo.priority),
                        borderRadius: BorderRadius.circular(16),
                      ),
                      child: Text(
                        _getPriorityText(todo.priority),
                        style: const TextStyle(
                          color: Colors.white,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                    ),
                  ],
                ),
              ),
            ),

            const SizedBox(height: 16),

            // Description Card
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16.0),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    const Text(
                      'Descrição',
                      style: TextStyle(
                        fontSize: 16,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                    const SizedBox(height: 8),
                    Text(
                      todo.description,
                      style: const TextStyle(fontSize: 14),
                    ),
                  ],
                ),
              ),
            ),

            const SizedBox(height: 16),

            // Date Info Card
            Card(
              child: Padding(
                padding: const EdgeInsets.all(16.0),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    const Text(
                      'Informações',
                      style: TextStyle(
                        fontSize: 16,
                        fontWeight: FontWeight.bold,
                      ),
                    ),
                    const SizedBox(height: 8),
                    Row(
                      mainAxisAlignment: MainAxisAlignment.spaceBetween,
                      children: [
                        const Text('ID:'),
                        Text(
                          todo.id,
                          style: const TextStyle(fontWeight: FontWeight.bold),
                        ),
                      ],
                    ),
                    const SizedBox(height: 4),
                    Row(
                      mainAxisAlignment: MainAxisAlignment.spaceBetween,
                      children: [
                        const Text('Criada em:'),
                        Text(
                          _formatDate(todo.createdAt),
                          style: const TextStyle(fontWeight: FontWeight.bold),
                        ),
                      ],
                    ),
                    const SizedBox(height: 4),
                    Row(
                      mainAxisAlignment: MainAxisAlignment.spaceBetween,
                      children: [
                        const Text('Dias desde criação:'),
                        Text(
                          '${DateTime.now().difference(todo.createdAt).inDays}',
                          style: const TextStyle(fontWeight: FontWeight.bold),
                        ),
                      ],
                    ),
                  ],
                ),
              ),
            ),

            const SizedBox(height: 24),

            // Action Buttons
            Row(
              children: [
                Expanded(
                  child: ElevatedButton.icon(
                    onPressed: () {
                      _toggleTodoStatus(context);
                    },
                    icon: Icon(
                      todo.isCompleted ? Icons.undo : Icons.check,
                    ),
                    label: Text(
                      todo.isCompleted ? 'Marcar Pendente' : 'Marcar Concluída',
                    ),
                    style: ElevatedButton.styleFrom(
                      backgroundColor: todo.isCompleted 
                        ? Colors.orange 
                        : Colors.green,
                      padding: const EdgeInsets.symmetric(vertical: 12),
                    ),
                  ),
                ),
                const SizedBox(width: 12),
                Expanded(
                  child: ElevatedButton.icon(
                    onPressed: () {
                      _deleteTodo(context);
                    },
                    icon: const Icon(Icons.delete),
                    label: const Text('Excluir'),
                    style: ElevatedButton.styleFrom(
                      backgroundColor: Colors.red,
                      padding: const EdgeInsets.symmetric(vertical: 12),
                    ),
                  ),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }

  Color _getPriorityColor(Priority priority) {
    switch (priority) {
      case Priority.low:
        return Colors.green;
      case Priority.medium:
        return Colors.orange;
      case Priority.high:
        return Colors.red;
    }
  }

  String _getPriorityText(Priority priority) {
    switch (priority) {
      case Priority.low:
        return 'Baixa';
      case Priority.medium:
        return 'Média';
      case Priority.high:
        return 'Alta';
    }
  }

  String _formatDate(DateTime date) {
    return '${date.day}/${date.month}/${date.year}';
  }

  void _toggleTodoStatus(BuildContext context) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Text(
          todo.isCompleted 
            ? 'Tarefa marcada como pendente' 
            : 'Tarefa marcada como concluída',
        ),
        backgroundColor: todo.isCompleted ? Colors.orange : Colors.green,
      ),
    );
  }

  void _deleteTodo(BuildContext context) {
    showDialog(
      context: context,
      builder: (BuildContext context) {
        return AlertDialog(
          title: const Text('Confirmar Exclusão'),
          content: Text('Deseja realmente excluir a tarefa "${todo.title}"?'),
          actions: [
            TextButton(
              onPressed: () => Navigator.of(context).pop(),
              child: const Text('Cancelar'),
            ),
            TextButton(
              onPressed: () {
                Navigator.of(context).pop();
                Navigator.of(context).pop();
                ScaffoldMessenger.of(context).showSnackBar(
                  const SnackBar(
                    content: Text('Tarefa excluída com sucesso'),
                    backgroundColor: Colors.red,
                  ),
                );
              },
              child: const Text('Excluir'),
            ),
          ],
        );
      },
    );
  }
}
```

## 🔧 Método 2: Usando RouteSettings

### Vantagens
- ✅ **Flexível** - Pode passar qualquer tipo de objeto
- ✅ **Desacoplado** - Tela de destino não precisa conhecer o tipo exato
- ✅ **Reutilizável** - Mesma tela pode receber diferentes tipos de dados

### Desvantagens
- ❌ **Não é type-safe** - Requer casting manual
- ❌ **Propenso a erros** - Pode quebrar em runtime
- ❌ **Menos suporte da IDE**

### Exemplo com RouteSettings

#### Tela de Destino

```dart
class TodoDetailScreen extends StatelessWidget {
  const TodoDetailScreen({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    // Extraindo argumentos via RouteSettings
    final todo = ModalRoute.of(context)!.settings.arguments as Todo;

    return Scaffold(
      appBar: AppBar(
        title: Text(todo.title),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              'ID: ${todo.id}',
              style: const TextStyle(fontSize: 16),
            ),
            const SizedBox(height: 8),
            Text(
              todo.description,
              style: const TextStyle(fontSize: 14),
            ),
            const SizedBox(height: 16),
            Text(
              'Status: ${todo.isCompleted ? "Concluída" : "Pendente"}',
              style: TextStyle(
                fontSize: 16,
                color: todo.isCompleted ? Colors.green : Colors.orange,
                fontWeight: FontWeight.bold,
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

#### Navegação com RouteSettings

```dart
onTap: () {
  Navigator.push(
    context,
    MaterialPageRoute<void>(
      builder: (context) => const TodoDetailScreen(),
      settings: RouteSettings(
        arguments: todos[index],
        name: '/todo-detail',
      ),
    ),
  );
},
```

### Versão Mais Segura com Verificação de Tipos

```dart
class TodoDetailScreen extends StatelessWidget {
  const TodoDetailScreen({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    final arguments = ModalRoute.of(context)?.settings.arguments;

    // Verificação de tipo mais segura
    if (arguments == null || arguments is! Todo) {
      return Scaffold(
        appBar: AppBar(title: const Text('Erro')),
        body: const Center(
          child: Text(
            'Dados inválidos ou não fornecidos',
            style: TextStyle(fontSize: 16),
          ),
        ),
      );
    }

    final todo = arguments as Todo;

    return Scaffold(
      appBar: AppBar(title: Text(todo.title)),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Text(todo.description),
      ),
    );
  }
}
```

## 🗺️ Método 3: Named Routes com Argumentos

### Configuração de Rotas Nomeadas

```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Navigation Demo',
      initialRoute: '/',
      routes: {
        '/': (context) => const TodoListScreen(),
        '/todo-detail': (context) => const TodoDetailScreen(),
        '/add-todo': (context) => const AddTodoScreen(),
        '/edit-todo': (context) => const EditTodoScreen(),
      },
      // Tratamento de rotas não encontradas
      onUnknownRoute: (settings) {
        return MaterialPageRoute(
          builder: (context) => const NotFoundScreen(),
        );
      },
    );
  }
}
```

### Navegação com Named Routes

```dart
// Navegação simples
Navigator.pushNamed(context, '/todo-detail');

// Navegação com argumentos
Navigator.pushNamed(
  context,
  '/todo-detail',
  arguments: todo,
);

// Navegação com substituição
Navigator.pushReplacementNamed(
  context,
  '/todo-detail',
  arguments: todo,
);

// Navegação removendo todas as rotas anteriores
Navigator.pushNamedAndRemoveUntil(
  context,
  '/home',
  (route) => false,
);
```

### Gerador de Rotas Avançado

```dart
class RouteGenerator {
  static Route<dynamic> generateRoute(RouteSettings settings) {
    switch (settings.name) {
      case '/':
        return MaterialPageRoute(
          builder: (_) => const TodoListScreen(),
        );

      case '/todo-detail':
        if (settings.arguments is Todo) {
          return MaterialPageRoute(
            builder: (_) => TodoDetailScreen(
              todo: settings.arguments as Todo,
            ),
          );
        }
        return _errorRoute('Argumentos inválidos para /todo-detail');

      case '/add-todo':
        return MaterialPageRoute(
          builder: (_) => const AddTodoScreen(),
        );

      case '/edit-todo':
        if (settings.arguments is Todo) {
          return MaterialPageRoute(
            builder: (_) => EditTodoScreen(
              todo: settings.arguments as Todo,
            ),
          );
        }
        return _errorRoute('Argumentos inválidos para /edit-todo');

      default:
        return _errorRoute('Rota não encontrada: ${settings.name}');
    }
  }

  static Route<dynamic> _errorRoute(String message) {
    return MaterialPageRoute(
      builder: (_) => Scaffold(
        appBar: AppBar(title: const Text('Erro')),
        body: Center(
          child: Text(
            message,
            style: const TextStyle(fontSize: 16),
          ),
        ),
      ),
    );
  }
}

// Uso no MaterialApp
MaterialApp(
  onGenerateRoute: RouteGenerator.generateRoute,
  initialRoute: '/',
)
```

## 📊 Tipos de Dados Suportados

### Tipos Primitivos

```dart
// String
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (context) => DetailScreen(title: 'Meu Título'),
  ),
);

// Números
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (context) => CounterScreen(initialValue: 42),
  ),
);

// Boolean
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (context) => SettingsScreen(isDarkMode: true),
  ),
);
```

### Objetos Complexos

```dart
// Classe personalizada
class User {
  final String id;
  final String name;
  final String email;
  final List<String> roles;

  const User({
    required this.id,
    required this.name,
    required this.email,
    required this.roles,
  });
}

Navigator.push(
  context,
  MaterialPageRoute(
    builder: (context) => ProfileScreen(user: currentUser),
  ),
);
```

### Listas e Maps

```dart
// Lista
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (context) => ItemListScreen(
      items: ['Item 1', 'Item 2', 'Item 3'],
    ),
  ),
);

// Map
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (context) => ConfigScreen(
      settings: {
        'theme': 'dark',
        'language': 'pt-BR',
        'notifications': true,
      },
    ),
  ),
);
```

### Funções Callback

```dart
class EditScreen extends StatelessWidget {
  final String initialText;
  final Function(String) onSave;

  const EditScreen({
    Key? key,
    required this.initialText,
    required this.onSave,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    final controller = TextEditingController(text: initialText);

    return Scaffold(
      appBar: AppBar(
        title: const Text('Editar'),
        actions: [
          IconButton(
            icon: const Icon(Icons.save),
            onPressed: () {
              onSave(controller.text);
              Navigator.pop(context);
            },
          ),
        ],
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: TextField(
          controller: controller,
          decoration: const InputDecoration(
            labelText: 'Texto',
            border: OutlineInputBorder(),
          ),
        ),
      ),
    );
  }
}

// Uso
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (context) => EditScreen(
      initialText: currentText,
      onSave: (newText) {
        setState(() {
          currentText = newText;
        });
      },
    ),
  ),
);
```

## 🎯 Exemplos Práticos Avançados

### 1. Sistema de E-commerce

```dart
// Modelos
class Product {
  final String id;
  final String name;
  final double price;
  final String imageUrl;
  final String description;
  final List<String> categories;

  const Product({
    required this.id,
    required this.name,
    required this.price,
    required this.imageUrl,
    required this.description,
    required this.categories,
  });
}

class CartItem {
  final Product product;
  final int quantity;

  const CartItem({
    required this.product,
    required this.quantity,
  });
}

// Tela de Produto
class ProductScreen extends StatelessWidget {
  final Product product;
  final Function(CartItem) onAddToCart;

  const ProductScreen({
    Key? key,
    required this.product,
    required this.onAddToCart,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text(product.name)),
      body: Column(
        children: [
          Image.network(product.imageUrl),
          Padding(
            padding: const EdgeInsets.all(16.0),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  product.name,
                  style: const TextStyle(
                    fontSize: 24,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                Text(
                  'R\$ ${product.price.toStringAsFixed(2)}',
                  style: const TextStyle(
                    fontSize: 20,
                    color: Colors.green,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                const SizedBox(height: 16),
                Text(product.description),
              ],
            ),
          ),
        ],
      ),
      bottomNavigationBar: Padding(
        padding: const EdgeInsets.all(16.0),
        child: ElevatedButton(
          onPressed: () {
            onAddToCart(CartItem(product: product, quantity: 1));
            ScaffoldMessenger.of(context).showSnackBar(
              const SnackBar(
                content: Text('Produto adicionado ao carrinho'),
              ),
            );
          },
          child: const Text('Adicionar ao Carrinho'),
        ),
      ),
    );
  }
}
```

### 2. Sistema de Formulário Multi-etapas

```dart
class FormData {
  String? name;
  String? email;
  String? phone;
  String? address;
  List<String> interests;

  FormData({
    this.name,
    this.email,
    this.phone,
    this.address,
    this.interests = const [],
  });

  FormData copyWith({
    String? name,
    String? email,
    String? phone,
    String? address,
    List<String>? interests,
  }) {
    return FormData(
      name: name ?? this.name,
      email: email ?? this.email,
      phone: phone ?? this.phone,
      address: address ?? this.address,
      interests: interests ?? this.interests,
    );
  }
}

// Etapa 1 - Dados Pessoais
class PersonalInfoScreen extends StatefulWidget {
  final FormData formData;
  final Function(FormData) onNext;

  const PersonalInfoScreen({
    Key? key,
    required this.formData,
    required this.onNext,
  }) : super(key: key);

  @override
  State<PersonalInfoScreen> createState() => _PersonalInfoScreenState();
}

class _PersonalInfoScreenState extends State<PersonalInfoScreen> {
  final _formKey = GlobalKey<FormState>();
  late TextEditingController _nameController;
  late TextEditingController _emailController;

  @override
  void initState() {
    super.initState();
    _nameController = TextEditingController(text: widget.formData.name);
    _emailController = TextEditingController(text: widget.formData.email);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Dados Pessoais')),
      body: Form(
        key: _formKey,
        child: Padding(
          padding: const EdgeInsets.all(16.0),
          child: Column(
            children: [
              TextFormField(
                controller: _nameController,
                decoration: const InputDecoration(labelText: 'Nome'),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Nome é obrigatório';
                  }
                  return null;
                },
              ),
              TextFormField(
                controller: _emailController,
                decoration: const InputDecoration(labelText: 'Email'),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Email é obrigatório';
                  }
                  if (!value.contains('@')) {
                    return 'Email inválido';
                  }
                  return null;
                },
              ),
              const Spacer(),
              ElevatedButton(
                onPressed: () {
                  if (_formKey.currentState!.validate()) {
                    final updatedData = widget.formData.copyWith(
                      name: _nameController.text,
                      email: _emailController.text,
                    );
                    widget.onNext(updatedData);
                  }
                },
                child: const Text('Próximo'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

### 3. Sistema de Filtros Avançados

```dart
class FilterOptions {
  final List<String> categories;
  final double minPrice;
  final double maxPrice;
  final bool inStock;
  final String sortBy;

  const FilterOptions({
    this.categories = const [],
    this.minPrice = 0.0,
    this.maxPrice = 1000.0,
    this.inStock = false,
    this.sortBy = 'name',
  });

  FilterOptions copyWith({
    List<String>? categories,
    double? minPrice,
    double? maxPrice,
    bool? inStock,
    String? sortBy,
  }) {
    return FilterOptions(
      categories: categories ?? this.categories,
      minPrice: minPrice ?? this.minPrice,
      maxPrice: maxPrice ?? this.maxPrice,
      inStock: inStock ?? this.inStock,
      sortBy: sortBy ?? this.sortBy,
    );
  }
}

class FilterScreen extends StatefulWidget {
  final FilterOptions initialFilters;
  final Function(FilterOptions) onApplyFilters;

  const FilterScreen({
    Key? key,
    required this.initialFilters,
    required this.onApplyFilters,
  }) : super(key: key);

  @override
  State<FilterScreen> createState() => _FilterScreenState();
}

class _FilterScreenState extends State<FilterScreen> {
  late FilterOptions _currentFilters;

  @override
  void initState() {
    super.initState();
    _currentFilters = widget.initialFilters;
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Filtros'),
        actions: [
          TextButton(
            onPressed: () {
              setState(() {
                _currentFilters = const FilterOptions();
              });
            },
            child: const Text('Limpar'),
          ),
        ],
      ),
      body: Column(
        children: [
          Expanded(
            child: ListView(
              padding: const EdgeInsets.all(16.0),
              children: [
                // Categorias
                const Text(
                  'Categorias',
                  style: TextStyle(
                    fontSize: 18,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                ...['Eletrônicos', 'Roupas', 'Casa', 'Esportes'].map(
                  (category) => CheckboxListTile(
                    title: Text(category),
                    value: _currentFilters.categories.contains(category),
                    onChanged: (bool? value) {
                      setState(() {
                        if (value == true) {
                          _currentFilters = _currentFilters.copyWith(
                            categories: [..._currentFilters.categories, category],
                          );
                        } else {
                          _currentFilters = _currentFilters.copyWith(
                            categories: _currentFilters.categories
                                .where((c) => c != category)
                                .toList(),
                          );
                        }
                      });
                    },
                  ),
                ),

                const SizedBox(height: 16),

                // Faixa de Preço
                const Text(
                  'Faixa de Preço',
                  style: TextStyle(
                    fontSize: 18,
                    fontWeight: FontWeight.bold,
                  ),
                ),
                RangeSlider(
                  values: RangeValues(
                    _currentFilters.minPrice,
                    _currentFilters.maxPrice,
                  ),
                  min: 0,
                  max: 1000,
                  divisions: 20,
                  labels: RangeLabels(
                    'R\$ ${_currentFilters.minPrice.toInt()}',
                    'R\$ ${_currentFilters.maxPrice.toInt()}',
                  ),
                  onChanged: (RangeValues values) {
                    setState(() {
                      _currentFilters = _currentFilters.copyWith(
                        minPrice: values.start,
                        maxPrice: values.end,
                      );
                    });
                  },
                ),

                // Apenas em Estoque
                SwitchListTile(
                  title: const Text('Apenas em Estoque'),
                  value: _currentFilters.inStock,
                  onChanged: (bool value) {
                    setState(() {
                      _currentFilters = _currentFilters.copyWith(inStock: value);
                    });
                  },
                ),
              ],
            ),
          ),

          // Botão Aplicar
          Padding(
            padding: const EdgeInsets.all(16.0),
            child: SizedBox(
              width: double.infinity,
              child: ElevatedButton(
                onPressed: () {
                  widget.onApplyFilters(_currentFilters);
                  Navigator.pop(context);
                },
                child: const Text('Aplicar Filtros'),
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

## 🏆 Melhores Práticas

### 1. Escolha do Método Adequado

```dart
// ✅ Use construtor direto para dados simples e type-safety
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (context) => DetailScreen(item: item),
  ),
);

// ✅ Use RouteSettings para flexibilidade
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (context) => GenericDetailScreen(),
    settings: RouteSettings(arguments: item),
  ),
);

// ✅ Use Named Routes para apps grandes
Navigator.pushNamed(context, '/detail', arguments: item);
```

### 2. Validação de Argumentos

```dart
class DetailScreen extends StatelessWidget {
  const DetailScreen({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    final arguments = ModalRoute.of(context)?.settings.arguments;

    // Validação robusta
    if (arguments == null) {
      return _buildErrorScreen('Nenhum argumento fornecido');
    }

    if (arguments is! Todo) {
      return _buildErrorScreen('Tipo de argumento inválido');
    }

    final todo = arguments as Todo;
    return _buildDetailScreen(todo);
  }

  Widget _buildErrorScreen(String message) {
    return Scaffold(
      appBar: AppBar(title: const Text('Erro')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.error, size: 64, color: Colors.red),
            const SizedBox(height: 16),
            Text(message, style: const TextStyle(fontSize: 16)),
          ],
        ),
      ),
    );
  }

  Widget _buildDetailScreen(Todo todo) {
    return Scaffold(
      appBar: AppBar(title: Text(todo.title)),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Text(todo.description),
      ),
    );
  }
}
```

### 3. Imutabilidade de Dados

```dart
// ✅ Use classes imutáveis
class Todo {
  final String title;
  final String description;

  const Todo({required this.title, required this.description});

  // Método copyWith para modificações
  Todo copyWith({String? title, String? description}) {
    return Todo(
      title: title ?? this.title,
      description: description ?? this.description,
    );
  }
}

// ❌ Evite classes mutáveis
class BadTodo {
  String title;
  String description;

  BadTodo({required this.title, required this.description});
}
```

### 4. Separação de Responsabilidades

```dart
// ✅ Separe lógica de navegação
class NavigationService {
  static void navigateToDetail(BuildContext context, Todo todo) {
    Navigator.push(
      context,
      MaterialPageRoute(
        builder: (context) => TodoDetailScreen(todo: todo),
      ),
    );
  }

  static void navigateToEdit(BuildContext context, Todo todo) {
    Navigator.push(
      context,
      MaterialPageRoute(
        builder: (context) => EditTodoScreen(todo: todo),
      ),
    );
  }
}

// Uso
onTap: () => NavigationService.navigateToDetail(context, todo),
```

### 5. Tratamento de Estados de Loading

```dart
class DetailScreen extends StatefulWidget {
  final String itemId;

  const DetailScreen({Key? key, required this.itemId}) : super(key: key);

  @override
  State<DetailScreen> createState() => _DetailScreenState();
}

class _DetailScreenState extends State<DetailScreen> {
  bool _isLoading = true;
  Item? _item;
  String? _error;

  @override
  void initState() {
    super.initState();
    _loadItem();
  }

  Future<void> _loadItem() async {
    try {
      final item = await ItemService.getById(widget.itemId);
      setState(() {
        _item = item;
        _isLoading = false;
      });
    } catch (e) {
      setState(() {
        _error = e.toString();
        _isLoading = false;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    if (_isLoading) {
      return const Scaffold(
        body: Center(child: CircularProgressIndicator()),
      );
    }

    if (_error != null) {
      return Scaffold(
        appBar: AppBar(title: const Text('Erro')),
        body: Center(child: Text(_error!)),
      );
    }

    return Scaffold(
      appBar: AppBar(title: Text(_item!.title)),
      body: Text(_item!.description),
    );
  }
}
```

## ⚠️ Tratamento de Erros

### 1. Argumentos Nulos ou Inválidos

```dart
class SafeDetailScreen extends StatelessWidget {
  const SafeDetailScreen({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    try {
      final arguments = ModalRoute.of(context)?.settings.arguments;

      if (arguments == null) {
        return _buildErrorScreen(
          context,
          'Dados não fornecidos',
          'Nenhum dado foi passado para esta tela.',
        );
      }

      if (arguments is! Todo) {
        return _buildErrorScreen(
          context,
          'Dados inválidos',
          'Os dados fornecidos não são do tipo esperado.',
        );
      }

      final todo = arguments as Todo;
      return _buildSuccessScreen(context, todo);

    } catch (e) {
      return _buildErrorScreen(
        context,
        'Erro inesperado',
        'Ocorreu um erro ao processar os dados: $e',
      );
    }
  }

  Widget _buildErrorScreen(BuildContext context, String title, String message) {
    return Scaffold(
      appBar: AppBar(
        title: Text(title),
        backgroundColor: Colors.red,
      ),
      body: Center(
        child: Padding(
          padding: const EdgeInsets.all(16.0),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              const Icon(
                Icons.error_outline,
                size: 64,
                color: Colors.red,
              ),
              const SizedBox(height: 16),
              Text(
                title,
                style: const TextStyle(
                  fontSize: 20,
                  fontWeight: FontWeight.bold,
                ),
              ),
              const SizedBox(height: 8),
              Text(
                message,
                textAlign: TextAlign.center,
                style: const TextStyle(fontSize: 16),
              ),
              const SizedBox(height: 24),
              ElevatedButton(
                onPressed: () => Navigator.pop(context),
                child: const Text('Voltar'),
              ),
            ],
          ),
        ),
      ),
    );
  }

  Widget _buildSuccessScreen(BuildContext context, Todo todo) {
    return Scaffold(
      appBar: AppBar(title: Text(todo.title)),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Text(todo.description),
      ),
    );
  }
}
```

### 2. Wrapper para Navegação Segura

```dart
class SafeNavigator {
  static Future<T?> pushSafely<T extends Object?>(
    BuildContext context,
    Route<T> route,
  ) async {
    try {
      return await Navigator.push(context, route);
    } catch (e) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text('Erro na navegação: $e'),
          backgroundColor: Colors.red,
        ),
      );
      return null;
    }
  }

  static Future<T?> pushNamedSafely<T extends Object?>(
    BuildContext context,
    String routeName, {
    Object? arguments,
  }) async {
    try {
      return await Navigator.pushNamed(
        context,
        routeName,
        arguments: arguments,
      );
    } catch (e) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text('Erro na navegação: $e'),
          backgroundColor: Colors.red,
        ),
      );
      return null;
    }
  }
}

// Uso
SafeNavigator.pushSafely(
  context,
  MaterialPageRoute(
    builder: (context) => DetailScreen(todo: todo),
  ),
);
```

## 🔗 Recursos Adicionais

### Documentação Oficial
- [Flutter Navigation and Routing](https://docs.flutter.dev/development/ui/navigation)
- [Navigator Class](https://api.flutter.dev/flutter/widgets/Navigator-class.html)
- [Route Class](https://api.flutter.dev/flutter/widgets/Route-class.html)
- [RouteSettings Class](https://api.flutter.dev/flutter/widgets/RouteSettings-class.html)

### Packages Úteis
- [go_router](https://pub.dev/packages/go_router) - Roteamento declarativo
- [auto_route](https://pub.dev/packages/auto_route) - Geração automática de rotas
- [fluro](https://pub.dev/packages/fluro) - Roteamento avançado

### Tutoriais e Exemplos
- [Flutter Navigation Examples](https://github.com/flutter/samples/tree/master/navigation_and_routing)
- [Navigation Cookbook](https://docs.flutter.dev/cookbook/navigation)

### Comparação de Métodos

| Método | Type Safety | Flexibilidade | Complexidade | Recomendado Para |
|--------|-------------|---------------|--------------|------------------|
| Construtor Direto | ✅ Alta | ❌ Baixa | ✅ Baixa | Apps pequenos/médios |
| RouteSettings | ❌ Baixa | ✅ Alta | ⚠️ Média | Casos específicos |
| Named Routes | ⚠️ Média | ✅ Alta | ⚠️ Média | Apps grandes |
| go_router | ✅ Alta | ✅ Alta | ❌ Alta | Apps complexos |

---

## 📝 Conclusão

A passagem de dados entre telas é fundamental no desenvolvimento Flutter. Cada método tem suas vantagens:

- **Construtor direto**: Ideal para a maioria dos casos, oferece type safety
- **RouteSettings**: Útil para casos específicos que precisam de flexibilidade
- **Named Routes**: Melhor para aplicações grandes e organizadas

**Recomendações:**
1. **Comece simples** - Use construtor direto
2. **Valide sempre** os argumentos recebidos
3. **Mantenha imutabilidade** dos dados
4. **Trate erros** adequadamente
5. **Considere packages** como go_router para apps complexos

---

*Documentação criada para auxiliar desenvolvedores Flutter no domínio da navegação e passagem de dados entre telas.*