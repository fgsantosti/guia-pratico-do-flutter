# Chapter 2 - Flutter Widget Catalog 

## 📋 Índice

1. [Introdução](#introdução)
2. [Design Systems](#design-systems)
3. [Base Widgets](#base-widgets)
4. [Widgets Essenciais por Categoria](#widgets-essenciais-por-categoria)
5. [Exemplos Práticos](#exemplos-práticos)
6. [Melhores Práticas](#melhores-práticas)
7. [Recursos Adicionais](#recursos-adicionais)

## 🚀 Introdução

O Flutter oferece uma rica coleção de widgets visuais, estruturais, de plataforma e interativos que permitem criar aplicações bonitas e funcionais de forma mais rápida. Esta documentação abrange os widgets mais importantes do catálogo oficial do Flutter.

### O que são Widgets?

No Flutter, **tudo é um widget**. Widgets são os blocos de construção fundamentais da interface do usuário. Eles descrevem como a UI deve aparecer com base no estado atual e na configuração.

## 🎨 Design Systems

O Flutter vem com dois sistemas de design principais:

### Material Components
Widgets que implementam as especificações do Material Design 3 do Google.

```dart
import 'package:flutter/material.dart';

class MaterialExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
        useMaterial3: true,
      ),
      home: Scaffold(
        appBar: AppBar(
          title: Text('Material Design'),
          backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        ),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              ElevatedButton(
                onPressed: () {},
                child: Text('Botão Elevado'),
              ),
              SizedBox(height: 16),
              FloatingActionButton(
                onPressed: () {},
                child: Icon(Icons.add),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

### Cupertino (iOS Style)
Widgets que seguem as diretrizes de interface humana da Apple para iOS e macOS.

```dart
import 'package:flutter/cupertino.dart';

class CupertinoExample extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return CupertinoApp(
      home: CupertinoPageScaffold(
        navigationBar: CupertinoNavigationBar(
          middle: Text('iOS Style'),
        ),
        child: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              CupertinoButton.filled(
                onPressed: () {},
                child: Text('Botão iOS'),
              ),
              SizedBox(height: 16),
              CupertinoSwitch(
                value: true,
                onChanged: (value) {},
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

## 🧱 Base Widgets

### Widgets Básicos

#### Container
O widget mais versátil para decoração, posicionamento e dimensionamento.

```dart
Container(
  width: 200,
  height: 100,
  padding: EdgeInsets.all(16),
  margin: EdgeInsets.all(8),
  decoration: BoxDecoration(
    color: Colors.blue,
    borderRadius: BorderRadius.circular(12),
    boxShadow: [
      BoxShadow(
        color: Colors.grey.withOpacity(0.5),
        spreadRadius: 2,
        blurRadius: 5,
        offset: Offset(0, 3),
      ),
    ],
  ),
  child: Text(
    'Container Exemplo',
    style: TextStyle(color: Colors.white),
  ),
)
```

#### Text
Widget fundamental para exibir texto.

```dart
Text(
  'Texto Personalizado',
  style: TextStyle(
    fontSize: 24,
    fontWeight: FontWeight.bold,
    color: Colors.blue,
    letterSpacing: 1.2,
  ),
  textAlign: TextAlign.center,
  overflow: TextOverflow.ellipsis,
  maxLines: 2,
)
```

#### Image
Para exibir imagens de diferentes fontes.

```dart
// Imagem da internet
Image.network(
  'https://picsum.photos/300/200',
  width: 300,
  height: 200,
  fit: BoxFit.cover,
  loadingBuilder: (context, child, loadingProgress) {
    if (loadingProgress == null) return child;
    return CircularProgressIndicator();
  },
)

// Imagem local
Image.asset(
  'assets/images/logo.png',
  width: 100,
  height: 100,
)
```

## 📱 Widgets Essenciais por Categoria

### Layout Widgets

#### Column & Row
Para organizar widgets verticalmente e horizontalmente.

```dart
Column(
  mainAxisAlignment: MainAxisAlignment.spaceEvenly,
  crossAxisAlignment: CrossAxisAlignment.center,
  children: [
    Text('Item 1'),
    Text('Item 2'),
    Text('Item 3'),
  ],
)

Row(
  mainAxisAlignment: MainAxisAlignment.spaceBetween,
  children: [
    Icon(Icons.home),
    Icon(Icons.search),
    Icon(Icons.person),
  ],
)
```

#### Stack
Para sobrepor widgets.

```dart
Stack(
  alignment: Alignment.center,
  children: [
    Container(
      width: 200,
      height: 200,
      color: Colors.blue,
    ),
    Positioned(
      top: 20,
      right: 20,
      child: Icon(
        Icons.favorite,
        color: Colors.red,
        size: 30,
      ),
    ),
    Text(
      'Texto Sobreposto',
      style: TextStyle(
        color: Colors.white,
        fontSize: 18,
        fontWeight: FontWeight.bold,
      ),
    ),
  ],
)
```

#### Expanded & Flexible
Para controlar como widgets ocupam espaço disponível.

```dart
Row(
  children: [
    Expanded(
      flex: 2,
      child: Container(
        height: 100,
        color: Colors.red,
        child: Center(child: Text('2/3 do espaço')),
      ),
    ),
    Expanded(
      flex: 1,
      child: Container(
        height: 100,
        color: Colors.blue,
        child: Center(child: Text('1/3 do espaço')),
      ),
    ),
  ],
)
```

### Input Widgets

#### TextField
Para entrada de texto do usuário.

```dart
class TextFieldExample extends StatefulWidget {
  @override
  _TextFieldExampleState createState() => _TextFieldExampleState();
}

class _TextFieldExampleState extends State<TextFieldExample> {
  final TextEditingController _controller = TextEditingController();
  String _inputText = '';

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        TextField(
          controller: _controller,
          decoration: InputDecoration(
            labelText: 'Digite seu nome',
            hintText: 'Nome completo',
            prefixIcon: Icon(Icons.person),
            border: OutlineInputBorder(
              borderRadius: BorderRadius.circular(12),
            ),
            filled: true,
            fillColor: Colors.grey[100],
          ),
          onChanged: (value) {
            setState(() {
              _inputText = value;
            });
          },
        ),
        SizedBox(height: 16),
        Text('Você digitou: $_inputText'),
      ],
    );
  }
}
```

#### Form & TextFormField
Para formulários com validação.

```dart
class FormExample extends StatefulWidget {
  @override
  _FormExampleState createState() => _FormExampleState();
}

class _FormExampleState extends State<FormExample> {
  final _formKey = GlobalKey<FormState>();
  String _email = '';
  String _password = '';

  @override
  Widget build(BuildContext context) {
    return Form(
      key: _formKey,
      child: Column(
        children: [
          TextFormField(
            decoration: InputDecoration(
              labelText: 'Email',
              prefixIcon: Icon(Icons.email),
            ),
            validator: (value) {
              if (value == null || value.isEmpty) {
                return 'Por favor, insira um email';
              }
              if (!value.contains('@')) {
                return 'Email inválido';
              }
              return null;
            },
            onSaved: (value) => _email = value!,
          ),
          TextFormField(
            decoration: InputDecoration(
              labelText: 'Senha',
              prefixIcon: Icon(Icons.lock),
            ),
            obscureText: true,
            validator: (value) {
              if (value == null || value.length < 6) {
                return 'Senha deve ter pelo menos 6 caracteres';
              }
              return null;
            },
            onSaved: (value) => _password = value!,
          ),
          SizedBox(height: 20),
          ElevatedButton(
            onPressed: () {
              if (_formKey.currentState!.validate()) {
                _formKey.currentState!.save();
                ScaffoldMessenger.of(context).showSnackBar(
                  SnackBar(content: Text('Formulário válido!')),
                );
              }
            },
            child: Text('Enviar'),
          ),
        ],
      ),
    );
  }
}
```

### Scrolling Widgets

#### ListView
Para listas roláveis.

```dart
// ListView básico
ListView(
  children: [
    ListTile(
      leading: Icon(Icons.home),
      title: Text('Home'),
      subtitle: Text('Página inicial'),
      trailing: Icon(Icons.arrow_forward_ios),
      onTap: () {},
    ),
    ListTile(
      leading: Icon(Icons.settings),
      title: Text('Configurações'),
      subtitle: Text('Ajustes do app'),
      trailing: Icon(Icons.arrow_forward_ios),
      onTap: () {},
    ),
  ],
)

// ListView.builder para listas dinâmicas
ListView.builder(
  itemCount: 100,
  itemBuilder: (context, index) {
    return ListTile(
      leading: CircleAvatar(
        child: Text('${index + 1}'),
      ),
      title: Text('Item $index'),
      subtitle: Text('Descrição do item $index'),
    );
  },
)
```

#### GridView
Para layouts em grade.

```dart
GridView.builder(
  gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
    crossAxisCount: 2,
    crossAxisSpacing: 10,
    mainAxisSpacing: 10,
    childAspectRatio: 1,
  ),
  itemCount: 20,
  itemBuilder: (context, index) {
    return Container(
      decoration: BoxDecoration(
        color: Colors.blue[100 * (index % 9)],
        borderRadius: BorderRadius.circular(12),
      ),
      child: Center(
        child: Text(
          'Item $index',
          style: TextStyle(
            fontSize: 18,
            fontWeight: FontWeight.bold,
          ),
        ),
      ),
    );
  },
)
```

### Animation Widgets

#### AnimatedContainer
Para animações implícitas de container.

```dart
class AnimatedContainerExample extends StatefulWidget {
  @override
  _AnimatedContainerExampleState createState() => _AnimatedContainerExampleState();
}

class _AnimatedContainerExampleState extends State<AnimatedContainerExample> {
  bool _isExpanded = false;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        AnimatedContainer(
          duration: Duration(seconds: 1),
          curve: Curves.easeInOut,
          width: _isExpanded ? 200 : 100,
          height: _isExpanded ? 200 : 100,
          decoration: BoxDecoration(
            color: _isExpanded ? Colors.red : Colors.blue,
            borderRadius: BorderRadius.circular(_isExpanded ? 50 : 10),
          ),
          child: Center(
            child: Text(
              _isExpanded ? 'Grande' : 'Pequeno',
              style: TextStyle(color: Colors.white),
            ),
          ),
        ),
        SizedBox(height: 20),
        ElevatedButton(
          onPressed: () {
            setState(() {
              _isExpanded = !_isExpanded;
            });
          },
          child: Text('Animar'),
        ),
      ],
    );
  }
}
```

#### Hero
Para animações de transição entre telas.

```dart
// Tela 1
Hero(
  tag: 'hero-image',
  child: GestureDetector(
    onTap: () {
      Navigator.push(
        context,
        MaterialPageRoute(builder: (context) => DetailScreen()),
      );
    },
    child: Container(
      width: 100,
      height: 100,
      decoration: BoxDecoration(
        color: Colors.blue,
        borderRadius: BorderRadius.circular(50),
      ),
    ),
  ),
)

// Tela 2 (DetailScreen)
class DetailScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Detalhes')),
      body: Center(
        child: Hero(
          tag: 'hero-image',
          child: Container(
            width: 300,
            height: 300,
            decoration: BoxDecoration(
              color: Colors.blue,
              borderRadius: BorderRadius.circular(20),
            ),
          ),
        ),
      ),
    );
  }
}
```

### Interaction Widgets

#### GestureDetector
Para detectar gestos do usuário.

```dart
class GestureExample extends StatefulWidget {
  @override
  _GestureExampleState createState() => _GestureExampleState();
}

class _GestureExampleState extends State<GestureExample> {
  String _gestureType = 'Nenhum gesto detectado';

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        GestureDetector(
          onTap: () => setState(() => _gestureType = 'Tap'),
          onDoubleTap: () => setState(() => _gestureType = 'Double Tap'),
          onLongPress: () => setState(() => _gestureType = 'Long Press'),
          onPanUpdate: (details) => setState(() => _gestureType = 'Pan'),
          child: Container(
            width: 200,
            height: 200,
            decoration: BoxDecoration(
              color: Colors.blue,
              borderRadius: BorderRadius.circular(12),
            ),
            child: Center(
              child: Text(
                'Toque aqui',
                style: TextStyle(
                  color: Colors.white,
                  fontSize: 18,
                ),
              ),
            ),
          ),
        ),
        SizedBox(height: 20),
        Text(
          _gestureType,
          style: TextStyle(fontSize: 16),
        ),
      ],
    );
  }
}
```

#### InkWell
Para efeitos de toque Material Design.

```dart
InkWell(
  onTap: () {
    print('InkWell tocado!');
  },
  borderRadius: BorderRadius.circular(12),
  child: Container(
    padding: EdgeInsets.all(16),
    decoration: BoxDecoration(
      border: Border.all(color: Colors.grey),
      borderRadius: BorderRadius.circular(12),
    ),
    child: Row(
      children: [
        Icon(Icons.touch_app),
        SizedBox(width: 8),
        Text('Toque para ver o efeito ripple'),
      ],
    ),
  ),
)
```

## 🎯 Exemplos Práticos

### Exemplo 1: Card de Perfil

```dart
class ProfileCard extends StatelessWidget {
  final String name;
  final String email;
  final String avatarUrl;

  const ProfileCard({
    Key? key,
    required this.name,
    required this.email,
    required this.avatarUrl,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Card(
      elevation: 8,
      margin: EdgeInsets.all(16),
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(16),
      ),
      child: Padding(
        padding: EdgeInsets.all(20),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            CircleAvatar(
              radius: 50,
              backgroundImage: NetworkImage(avatarUrl),
            ),
            SizedBox(height: 16),
            Text(
              name,
              style: TextStyle(
                fontSize: 24,
                fontWeight: FontWeight.bold,
              ),
            ),
            SizedBox(height: 8),
            Text(
              email,
              style: TextStyle(
                fontSize: 16,
                color: Colors.grey[600],
              ),
            ),
            SizedBox(height: 20),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                ElevatedButton.icon(
                  onPressed: () {},
                  icon: Icon(Icons.message),
                  label: Text('Mensagem'),
                ),
                OutlinedButton.icon(
                  onPressed: () {},
                  icon: Icon(Icons.call),
                  label: Text('Ligar'),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}
```

### Exemplo 2: Lista de Tarefas

```dart
class TodoApp extends StatefulWidget {
  @override
  _TodoAppState createState() => _TodoAppState();
}

class _TodoAppState extends State<TodoApp> {
  List<Todo> _todos = [];
  final TextEditingController _controller = TextEditingController();

  void _addTodo() {
    if (_controller.text.isNotEmpty) {
      setState(() {
        _todos.add(Todo(
          id: DateTime.now().millisecondsSinceEpoch,
          title: _controller.text,
          isCompleted: false,
        ));
        _controller.clear();
      });
    }
  }

  void _toggleTodo(int id) {
    setState(() {
      final index = _todos.indexWhere((todo) => todo.id == id);
      if (index != -1) {
        _todos[index].isCompleted = !_todos[index].isCompleted;
      }
    });
  }

  void _deleteTodo(int id) {
    setState(() {
      _todos.removeWhere((todo) => todo.id == id);
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Lista de Tarefas'),
        backgroundColor: Colors.blue,
        foregroundColor: Colors.white,
      ),
      body: Column(
        children: [
          Padding(
            padding: EdgeInsets.all(16),
            child: Row(
              children: [
                Expanded(
                  child: TextField(
                    controller: _controller,
                    decoration: InputDecoration(
                      hintText: 'Adicionar nova tarefa...',
                      border: OutlineInputBorder(
                        borderRadius: BorderRadius.circular(12),
                      ),
                    ),
                    onSubmitted: (_) => _addTodo(),
                  ),
                ),
                SizedBox(width: 8),
                FloatingActionButton(
                  onPressed: _addTodo,
                  child: Icon(Icons.add),
                  mini: true,
                ),
              ],
            ),
          ),
          Expanded(
            child: ListView.builder(
              itemCount: _todos.length,
              itemBuilder: (context, index) {
                final todo = _todos[index];
                return Dismissible(
                  key: Key(todo.id.toString()),
                  direction: DismissDirection.endToStart,
                  background: Container(
                    color: Colors.red,
                    alignment: Alignment.centerRight,
                    padding: EdgeInsets.only(right: 20),
                    child: Icon(
                      Icons.delete,
                      color: Colors.white,
                    ),
                  ),
                  onDismissed: (_) => _deleteTodo(todo.id),
                  child: ListTile(
                    leading: Checkbox(
                      value: todo.isCompleted,
                      onChanged: (_) => _toggleTodo(todo.id),
                    ),
                    title: Text(
                      todo.title,
                      style: TextStyle(
                        decoration: todo.isCompleted
                            ? TextDecoration.lineThrough
                            : null,
                        color: todo.isCompleted
                            ? Colors.grey
                            : null,
                      ),
                    ),
                    trailing: IconButton(
                      icon: Icon(Icons.delete, color: Colors.red),
                      onPressed: () => _deleteTodo(todo.id),
                    ),
                  ),
                );
              },
            ),
          ),
        ],
      ),
    );
  }
}

class Todo {
  final int id;
  final String title;
  bool isCompleted;

  Todo({
    required this.id,
    required this.title,
    required this.isCompleted,
  });
}
```

## 💡 Melhores Práticas

### 1. Performance
- Use `const` constructors sempre que possível
- Prefira `ListView.builder` para listas grandes
- Evite reconstruções desnecessárias com `setState`

```dart
// ✅ Bom
const Text('Texto constante')

// ❌ Evitar
Text('Texto constante')
```

### 2. Responsividade
- Use `MediaQuery` para obter informações da tela
- Implemente layouts adaptativos com `LayoutBuilder`

```dart
Widget build(BuildContext context) {
  final screenWidth = MediaQuery.of(context).size.width;

  return Container(
    width: screenWidth > 600 ? 400 : screenWidth * 0.9,
    child: YourWidget(),
  );
}
```

### 3. Acessibilidade
- Sempre forneça `semanticsLabel` para widgets importantes
- Use cores com contraste adequado

```dart
Icon(
  Icons.favorite,
  semanticLabel: 'Adicionar aos favoritos',
)
```

### 4. Organização de Código
- Extraia widgets complexos em classes separadas
- Use `StatelessWidget` quando não há estado mutável

```dart
// ✅ Bom - Widget extraído
class CustomButton extends StatelessWidget {
  final String text;
  final VoidCallback onPressed;

  const CustomButton({
    Key? key,
    required this.text,
    required this.onPressed,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: onPressed,
      child: Text(text),
    );
  }
}
```

## 📚 Recursos Adicionais

### Documentação Oficial
- [Flutter Widget Catalog](https://docs.flutter.dev/ui/widgets)
- [Flutter API Documentation](https://api.flutter.dev/)
- [Material Design Components](https://docs.flutter.dev/ui/widgets/material)
- [Cupertino Widgets](https://docs.flutter.dev/ui/widgets/cupertino)

### Ferramentas Úteis
- **Flutter Inspector**: Para debug visual
- **Widget of the Week**: Vídeos semanais sobre widgets
- **DartPad**: Para testar código online

### Packages Populares
- `provider`: Gerenciamento de estado
- `http`: Requisições HTTP
- `shared_preferences`: Armazenamento local
- `image_picker`: Seleção de imagens
- `google_fonts`: Fontes do Google

### Dicas de Estudo
1. Pratique criando pequenos projetos
2. Explore o código fonte dos widgets
3. Participe da comunidade Flutter
4. Mantenha-se atualizado com as releases

---

## 🎉 Conclusão

Este guia abrange os widgets mais importantes do Flutter. Lembre-se de que a prática é fundamental para dominar o desenvolvimento Flutter. Comece com widgets simples e gradualmente construa interfaces mais complexas.

Para mais informações e exemplos atualizados, sempre consulte a [documentação oficial do Flutter](https://docs.flutter.dev/ui/widgets).

**Happy Coding! 🚀**