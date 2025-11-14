# Chapter 3 - Flutter Navegação e Rotas 

## 📋 Índice

1. [Introdução](#introdução)
2. [Navegação Básica (Navigator 1.0)](#navegação-básica-navigator-10)
3. [Navegação Avançada (Navigator 2.0)](#navegação-avançada-navigator-20)
4. [Go Router - Solução Declarativa](#go-router---solução-declarativa)
5. [Exemplos Práticos](#exemplos-práticos)
6. [Melhores Práticas](#melhores-práticas)
7. [Recursos Adicionais](#recursos-adicionais)

## 🚀 Introdução

A navegação é um aspecto fundamental no desenvolvimento de aplicações Flutter. Este guia abrange desde conceitos básicos até implementações avançadas usando o pacote **go_router**, que é a solução recomendada pelo time do Flutter para navegação declarativa.

### Tipos de Navegação no Flutter

1. **Navigator 1.0** - API imperativa tradicional
2. **Navigator 2.0** - API declarativa mais poderosa
3. **go_router** - Pacote que simplifica o Navigator 2.0

## 🧭 Navegação Básica (Navigator 1.0)

### Conceitos Fundamentais

O Navigator 1.0 usa uma pilha (stack) de rotas para gerenciar as telas da aplicação.

#### Navegação Simples

```dart
// Navegar para uma nova tela
Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => SecondScreen()),
);

// Voltar para a tela anterior
Navigator.pop(context);

// Substituir a tela atual
Navigator.pushReplacement(
  context,
  MaterialPageRoute(builder: (context) => NewScreen()),
);

// Limpar pilha e navegar
Navigator.pushAndRemoveUntil(
  context,
  MaterialPageRoute(builder: (context) => HomeScreen()),
  (route) => false, // Remove todas as rotas anteriores
);
```

#### Rotas Nomeadas

```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Navigation Demo',
      initialRoute: '/',
      routes: {
        '/': (context) => HomeScreen(),
        '/second': (context) => SecondScreen(),
        '/third': (context) => ThirdScreen(),
      },
    );
  }
}

// Navegação usando rotas nomeadas
Navigator.pushNamed(context, '/second');
Navigator.pushReplacementNamed(context, '/third');
Navigator.pushNamedAndRemoveUntil(context, '/', (route) => false);
```

#### Passando Dados Entre Telas

```dart
// Enviando dados
Navigator.push(
  context,
  MaterialPageRoute(
    builder: (context) => DetailScreen(data: 'Hello World'),
  ),
);

// Recebendo dados na tela de destino
class DetailScreen extends StatelessWidget {
  final String data;

  const DetailScreen({Key? key, required this.data}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Detalhes')),
      body: Center(child: Text(data)),
    );
  }
}

// Retornando dados
// Na tela de origem
final result = await Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => InputScreen()),
);
print('Resultado: $result');

// Na tela de destino
Navigator.pop(context, 'Dados de retorno');
```

#### Exemplo Completo - Navigator 1.0

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Navigator 1.0 Demo',
      theme: ThemeData(primarySwatch: Colors.blue),
      home: HomeScreen(),
    );
  }
}

class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Home')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            ElevatedButton(
              onPressed: () async {
                final result = await Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => SecondScreen(message: 'Olá da Home!'),
                  ),
                );
                if (result != null) {
                  ScaffoldMessenger.of(context).showSnackBar(
                    SnackBar(content: Text('Retorno: $result')),
                  );
                }
              },
              child: Text('Ir para Segunda Tela'),
            ),
            SizedBox(height: 16),
            ElevatedButton(
              onPressed: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(builder: (context) => ProfileScreen()),
                );
              },
              child: Text('Ver Perfil'),
            ),
          ],
        ),
      ),
    );
  }
}

class SecondScreen extends StatelessWidget {
  final String message;

  const SecondScreen({Key? key, required this.message}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Segunda Tela')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(
              message,
              style: TextStyle(fontSize: 18),
            ),
            SizedBox(height: 20),
            ElevatedButton(
              onPressed: () {
                Navigator.pop(context, 'Dados da segunda tela');
              },
              child: Text('Voltar com Dados'),
            ),
          ],
        ),
      ),
    );
  }
}

class ProfileScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Perfil')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            CircleAvatar(
              radius: 50,
              child: Icon(Icons.person, size: 50),
            ),
            SizedBox(height: 16),
            Text('João Silva', style: TextStyle(fontSize: 24)),
            Text('joao@email.com', style: TextStyle(fontSize: 16)),
            SizedBox(height: 20),
            ElevatedButton(
              onPressed: () => Navigator.pop(context),
              child: Text('Voltar'),
            ),
          ],
        ),
      ),
    );
  }
}
```

## 🔄 Navegação Avançada (Navigator 2.0)

O Navigator 2.0 introduz uma abordagem declarativa para navegação, oferecendo melhor controle sobre deep linking e estado da aplicação.

### Conceitos Principais

- **Router**: Gerencia a configuração de rotas
- **RouteInformationParser**: Converte URLs em objetos de configuração
- **RouterDelegate**: Constrói o Navigator baseado na configuração

```dart
class MyApp extends StatefulWidget {
  @override
  _MyAppState createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  final MyRouterDelegate _routerDelegate = MyRouterDelegate();
  final MyRouteInformationParser _routeInformationParser = MyRouteInformationParser();

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'Navigator 2.0 Demo',
      routerDelegate: _routerDelegate,
      routeInformationParser: _routeInformationParser,
    );
  }
}
```

## 🎯 Go Router - Solução Declarativa

O **go_router** é o pacote oficial recomendado pelo Flutter para navegação declarativa. Ele simplifica significativamente a implementação do Navigator 2.0.

### Instalação

```yaml
dependencies:
  flutter:
    sdk: flutter
  go_router: ^16.2.4
```

### Configuração Básica

```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'Go Router Demo',
      routerConfig: _router,
    );
  }

  final GoRouter _router = GoRouter(
    initialLocation: '/',
    routes: [
      GoRoute(
        path: '/',
        builder: (context, state) => HomeScreen(),
      ),
      GoRoute(
        path: '/details/:id',
        builder: (context, state) {
          final id = state.pathParameters['id']!;
          return DetailsScreen(id: id);
        },
      ),
      GoRoute(
        path: '/profile',
        builder: (context, state) => ProfileScreen(),
      ),
    ],
  );
}
```

### Navegação com Go Router

```dart
// Navegação simples
context.go('/profile');

// Navegação com parâmetros
context.go('/details/123');

// Push (adiciona à pilha)
context.push('/details/456');

// Pop (volta)
context.pop();

// Substituir rota atual
context.pushReplacement('/new-screen');
```

### Rotas Aninhadas (Sub-rotas)

```dart
final GoRouter _router = GoRouter(
  initialLocation: '/',
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => HomeScreen(),
      routes: [
        GoRoute(
          path: 'settings',
          builder: (context, state) => SettingsScreen(),
          routes: [
            GoRoute(
              path: 'profile',
              builder: (context, state) => ProfileSettingsScreen(),
            ),
          ],
        ),
      ],
    ),
  ],
);

// Navegação: context.go('/settings/profile');
```

### Shell Routes (Navegação com Layout Persistente)

```dart
final GoRouter _router = GoRouter(
  initialLocation: '/home',
  routes: [
    ShellRoute(
      builder: (context, state, child) {
        return ScaffoldWithNavBar(child: child);
      },
      routes: [
        GoRoute(
          path: '/home',
          builder: (context, state) => HomeScreen(),
        ),
        GoRoute(
          path: '/search',
          builder: (context, state) => SearchScreen(),
        ),
        GoRoute(
          path: '/profile',
          builder: (context, state) => ProfileScreen(),
        ),
      ],
    ),
  ],
);

class ScaffoldWithNavBar extends StatelessWidget {
  final Widget child;

  const ScaffoldWithNavBar({Key? key, required this.child}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: child,
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _calculateSelectedIndex(context),
        onTap: (index) => _onItemTapped(index, context),
        items: [
          BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Home'),
          BottomNavigationBarItem(icon: Icon(Icons.search), label: 'Search'),
          BottomNavigationBarItem(icon: Icon(Icons.person), label: 'Profile'),
        ],
      ),
    );
  }

  int _calculateSelectedIndex(BuildContext context) {
    final location = GoRouterState.of(context).location;
    if (location.startsWith('/home')) return 0;
    if (location.startsWith('/search')) return 1;
    if (location.startsWith('/profile')) return 2;
    return 0;
  }

  void _onItemTapped(int index, BuildContext context) {
    switch (index) {
      case 0:
        context.go('/home');
        break;
      case 1:
        context.go('/search');
        break;
      case 2:
        context.go('/profile');
        break;
    }
  }
}
```

### Redirecionamento e Guards

```dart
final GoRouter _router = GoRouter(
  initialLocation: '/',
  redirect: (context, state) {
    final isLoggedIn = AuthService.isLoggedIn();
    final isLoggingIn = state.location == '/login';

    if (!isLoggedIn && !isLoggingIn) {
      return '/login';
    }

    if (isLoggedIn && isLoggingIn) {
      return '/';
    }

    return null; // Não redirecionar
  },
  routes: [
    GoRoute(
      path: '/login',
      builder: (context, state) => LoginScreen(),
    ),
    GoRoute(
      path: '/',
      builder: (context, state) => HomeScreen(),
    ),
    GoRoute(
      path: '/admin',
      redirect: (context, state) {
        if (!AuthService.isAdmin()) {
          return '/';
        }
        return null;
      },
      builder: (context, state) => AdminScreen(),
    ),
  ],
);
```

### Tratamento de Erros

```dart
final GoRouter _router = GoRouter(
  initialLocation: '/',
  errorBuilder: (context, state) => ErrorScreen(error: state.error),
  routes: [
    // suas rotas aqui
  ],
);

class ErrorScreen extends StatelessWidget {
  final Exception? error;

  const ErrorScreen({Key? key, this.error}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Erro')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(Icons.error, size: 64, color: Colors.red),
            SizedBox(height: 16),
            Text('Página não encontrada'),
            SizedBox(height: 16),
            ElevatedButton(
              onPressed: () => context.go('/'),
              child: Text('Voltar ao Início'),
            ),
          ],
        ),
      ),
    );
  }
}
```

## 🎯 Exemplos Práticos

### Exemplo 1: App de E-commerce com Go Router

```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

void main() {
  runApp(ECommerceApp());
}

class ECommerceApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'E-commerce App',
      routerConfig: _router,
    );
  }

  final GoRouter _router = GoRouter(
    initialLocation: '/',
    routes: [
      ShellRoute(
        builder: (context, state, child) {
          return MainLayout(child: child);
        },
        routes: [
          GoRoute(
            path: '/',
            builder: (context, state) => HomeScreen(),
          ),
          GoRoute(
            path: '/categories',
            builder: (context, state) => CategoriesScreen(),
            routes: [
              GoRoute(
                path: ':categoryId',
                builder: (context, state) {
                  final categoryId = state.pathParameters['categoryId']!;
                  return CategoryScreen(categoryId: categoryId);
                },
              ),
            ],
          ),
          GoRoute(
            path: '/product/:productId',
            builder: (context, state) {
              final productId = state.pathParameters['productId']!;
              return ProductScreen(productId: productId);
            },
          ),
          GoRoute(
            path: '/cart',
            builder: (context, state) => CartScreen(),
          ),
          GoRoute(
            path: '/profile',
            builder: (context, state) => ProfileScreen(),
          ),
        ],
      ),
      GoRoute(
        path: '/checkout',
        builder: (context, state) => CheckoutScreen(),
      ),
    ],
  );
}

class MainLayout extends StatelessWidget {
  final Widget child;

  const MainLayout({Key? key, required this.child}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: child,
      bottomNavigationBar: BottomNavigationBar(
        type: BottomNavigationBarType.fixed,
        currentIndex: _calculateSelectedIndex(context),
        onTap: (index) => _onItemTapped(index, context),
        items: [
          BottomNavigationBarItem(icon: Icon(Icons.home), label: 'Home'),
          BottomNavigationBarItem(icon: Icon(Icons.category), label: 'Categorias'),
          BottomNavigationBarItem(icon: Icon(Icons.shopping_cart), label: 'Carrinho'),
          BottomNavigationBarItem(icon: Icon(Icons.person), label: 'Perfil'),
        ],
      ),
    );
  }

  int _calculateSelectedIndex(BuildContext context) {
    final location = GoRouterState.of(context).location;
    if (location == '/') return 0;
    if (location.startsWith('/categories')) return 1;
    if (location.startsWith('/cart')) return 2;
    if (location.startsWith('/profile')) return 3;
    return 0;
  }

  void _onItemTapped(int index, BuildContext context) {
    switch (index) {
      case 0:
        context.go('/');
        break;
      case 1:
        context.go('/categories');
        break;
      case 2:
        context.go('/cart');
        break;
      case 3:
        context.go('/profile');
        break;
    }
  }
}

class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('E-commerce')),
      body: GridView.builder(
        padding: EdgeInsets.all(16),
        gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 2,
          crossAxisSpacing: 16,
          mainAxisSpacing: 16,
        ),
        itemCount: 10,
        itemBuilder: (context, index) {
          return GestureDetector(
            onTap: () => context.go('/product/\${index + 1}'),
            child: Card(
              child: Column(
                children: [
                  Expanded(
                    child: Container(
                      color: Colors.grey[300],
                      child: Icon(Icons.shopping_bag, size: 50),
                    ),
                  ),
                  Padding(
                    padding: EdgeInsets.all(8),
                    child: Text('Produto \${index + 1}'),
                  ),
                ],
              ),
            ),
          );
        },
      ),
    );
  }
}

class ProductScreen extends StatelessWidget {
  final String productId;

  const ProductScreen({Key? key, required this.productId}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Produto $productId')),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Container(
              height: 200,
              width: double.infinity,
              color: Colors.grey[300],
              child: Icon(Icons.shopping_bag, size: 100),
            ),
            SizedBox(height: 16),
            Text(
              'Produto $productId',
              style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 8),
            Text(
              'R\$ 99,99',
              style: TextStyle(fontSize: 20, color: Colors.green),
            ),
            SizedBox(height: 16),
            Text(
              'Descrição detalhada do produto $productId. Este é um excelente produto com ótima qualidade.',
              style: TextStyle(fontSize: 16),
            ),
            Spacer(),
            SizedBox(
              width: double.infinity,
              child: ElevatedButton(
                onPressed: () {
                  ScaffoldMessenger.of(context).showSnackBar(
                    SnackBar(content: Text('Produto adicionado ao carrinho!')),
                  );
                },
                child: Text('Adicionar ao Carrinho'),
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class CategoriesScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final categories = ['Eletrônicos', 'Roupas', 'Casa', 'Esportes'];

    return Scaffold(
      appBar: AppBar(title: Text('Categorias')),
      body: ListView.builder(
        itemCount: categories.length,
        itemBuilder: (context, index) {
          return ListTile(
            leading: Icon(Icons.category),
            title: Text(categories[index]),
            trailing: Icon(Icons.arrow_forward_ios),
            onTap: () => context.go('/categories/\${index + 1}'),
          );
        },
      ),
    );
  }
}

class CategoryScreen extends StatelessWidget {
  final String categoryId;

  const CategoryScreen({Key? key, required this.categoryId}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Categoria $categoryId')),
      body: GridView.builder(
        padding: EdgeInsets.all(16),
        gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
          crossAxisCount: 2,
          crossAxisSpacing: 16,
          mainAxisSpacing: 16,
        ),
        itemCount: 6,
        itemBuilder: (context, index) {
          return GestureDetector(
            onTap: () => context.go('/product/cat$categoryId-\${index + 1}'),
            child: Card(
              child: Column(
                children: [
                  Expanded(
                    child: Container(
                      color: Colors.grey[300],
                      child: Icon(Icons.shopping_bag, size: 50),
                    ),
                  ),
                  Padding(
                    padding: EdgeInsets.all(8),
                    child: Text('Produto \${index + 1}'),
                  ),
                ],
              ),
            ),
          );
        },
      ),
    );
  }
}

class CartScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Carrinho')),
      body: Column(
        children: [
          Expanded(
            child: ListView.builder(
              itemCount: 3,
              itemBuilder: (context, index) {
                return ListTile(
                  leading: Container(
                    width: 50,
                    height: 50,
                    color: Colors.grey[300],
                    child: Icon(Icons.shopping_bag),
                  ),
                  title: Text('Produto \${index + 1}'),
                  subtitle: Text('R\$ 99,99'),
                  trailing: IconButton(
                    icon: Icon(Icons.remove_circle),
                    onPressed: () {},
                  ),
                );
              },
            ),
          ),
          Container(
            padding: EdgeInsets.all(16),
            child: Column(
              children: [
                Row(
                  mainAxisAlignment: MainAxisAlignment.spaceBetween,
                  children: [
                    Text('Total:', style: TextStyle(fontSize: 18)),
                    Text('R\$ 299,97', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
                  ],
                ),
                SizedBox(height: 16),
                SizedBox(
                  width: double.infinity,
                  child: ElevatedButton(
                    onPressed: () => context.go('/checkout'),
                    child: Text('Finalizar Compra'),
                  ),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}

class CheckoutScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Checkout')),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            Card(
              child: Padding(
                padding: EdgeInsets.all(16),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text('Resumo do Pedido', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
                    SizedBox(height: 8),
                    Text('3 itens'),
                    Text('Total: R\$ 299,97'),
                  ],
                ),
              ),
            ),
            SizedBox(height: 16),
            Card(
              child: Padding(
                padding: EdgeInsets.all(16),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text('Endereço de Entrega', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
                    SizedBox(height: 8),
                    Text('Rua das Flores, 123'),
                    Text('São Paulo, SP - 01234-567'),
                  ],
                ),
              ),
            ),
            Spacer(),
            SizedBox(
              width: double.infinity,
              child: ElevatedButton(
                onPressed: () {
                  showDialog(
                    context: context,
                    builder: (context) => AlertDialog(
                      title: Text('Pedido Confirmado!'),
                      content: Text('Seu pedido foi realizado com sucesso.'),
                      actions: [
                        TextButton(
                          onPressed: () {
                            Navigator.of(context).pop();
                            context.go('/');
                          },
                          child: Text('OK'),
                        ),
                      ],
                    ),
                  );
                },
                child: Text('Confirmar Pedido'),
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class ProfileScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Perfil')),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          children: [
            CircleAvatar(
              radius: 50,
              child: Icon(Icons.person, size: 50),
            ),
            SizedBox(height: 16),
            Text('João Silva', style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold)),
            Text('joao@email.com', style: TextStyle(fontSize: 16)),
            SizedBox(height: 32),
            ListTile(
              leading: Icon(Icons.shopping_bag),
              title: Text('Meus Pedidos'),
              trailing: Icon(Icons.arrow_forward_ios),
              onTap: () {},
            ),
            ListTile(
              leading: Icon(Icons.favorite),
              title: Text('Favoritos'),
              trailing: Icon(Icons.arrow_forward_ios),
              onTap: () {},
            ),
            ListTile(
              leading: Icon(Icons.settings),
              title: Text('Configurações'),
              trailing: Icon(Icons.arrow_forward_ios),
              onTap: () {},
            ),
            ListTile(
              leading: Icon(Icons.help),
              title: Text('Ajuda'),
              trailing: Icon(Icons.arrow_forward_ios),
              onTap: () {},
            ),
          ],
        ),
      ),
    );
  }
}
```

### Exemplo 2: Autenticação com Go Router

```dart
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

class AuthService {
  static bool _isLoggedIn = false;

  static bool isLoggedIn() => _isLoggedIn;

  static void login() => _isLoggedIn = true;

  static void logout() => _isLoggedIn = false;
}

void main() {
  runApp(AuthApp());
}

class AuthApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'Auth Demo',
      routerConfig: _router,
    );
  }

  final GoRouter _router = GoRouter(
    initialLocation: '/',
    redirect: (context, state) {
      final isLoggedIn = AuthService.isLoggedIn();
      final isLoggingIn = state.location == '/login';

      // Se não estiver logado e não estiver na tela de login
      if (!isLoggedIn && !isLoggingIn) {
        return '/login';
      }

      // Se estiver logado e estiver na tela de login
      if (isLoggedIn && isLoggingIn) {
        return '/';
      }

      return null; // Não redirecionar
    },
    routes: [
      GoRoute(
        path: '/login',
        builder: (context, state) => LoginScreen(),
      ),
      GoRoute(
        path: '/',
        builder: (context, state) => HomeScreen(),
      ),
      GoRoute(
        path: '/profile',
        builder: (context, state) => ProfileScreen(),
      ),
    ],
  );
}

class LoginScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Login')),
      body: Padding(
        padding: EdgeInsets.all(16),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            TextField(
              decoration: InputDecoration(
                labelText: 'Email',
                border: OutlineInputBorder(),
              ),
            ),
            SizedBox(height: 16),
            TextField(
              decoration: InputDecoration(
                labelText: 'Senha',
                border: OutlineInputBorder(),
              ),
              obscureText: true,
            ),
            SizedBox(height: 24),
            SizedBox(
              width: double.infinity,
              child: ElevatedButton(
                onPressed: () {
                  AuthService.login();
                  context.go('/');
                },
                child: Text('Entrar'),
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Home'),
        actions: [
          IconButton(
            icon: Icon(Icons.logout),
            onPressed: () {
              AuthService.logout();
              context.go('/login');
            },
          ),
        ],
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Bem-vindo!', style: TextStyle(fontSize: 24)),
            SizedBox(height: 16),
            ElevatedButton(
              onPressed: () => context.go('/profile'),
              child: Text('Ver Perfil'),
            ),
          ],
        ),
      ),
    );
  }
}

class ProfileScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Perfil')),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            CircleAvatar(
              radius: 50,
              child: Icon(Icons.person, size: 50),
            ),
            SizedBox(height: 16),
            Text('João Silva', style: TextStyle(fontSize: 24)),
            Text('joao@email.com', style: TextStyle(fontSize: 16)),
          ],
        ),
      ),
    );
  }
}
```

## 💡 Melhores Práticas

### 1. Estrutura de Rotas

```dart
// ✅ Bom - Rotas organizadas e hierárquicas
final GoRouter _router = GoRouter(
  routes: [
    ShellRoute(
      builder: (context, state, child) => MainLayout(child: child),
      routes: [
        GoRoute(path: '/', builder: (context, state) => HomeScreen()),
        GoRoute(
          path: '/products',
          builder: (context, state) => ProductsScreen(),
          routes: [
            GoRoute(
              path: ':id',
              builder: (context, state) => ProductDetailScreen(
                id: state.pathParameters['id']!,
              ),
            ),
          ],
        ),
      ],
    ),
  ],
);
```

### 2. Gerenciamento de Estado

```dart
// Use providers ou outros gerenciadores de estado
class AppRouter {
  static GoRouter createRouter(AuthNotifier authNotifier) {
    return GoRouter(
      refreshListenable: authNotifier,
      redirect: (context, state) {
        // Lógica de redirecionamento baseada no estado
      },
      routes: [
        // suas rotas
      ],
    );
  }
}
```

### 3. Tipagem Segura

```dart
// Use extensões para navegação tipada
extension AppNavigation on BuildContext {
  void goToProduct(String productId) {
    go('/products/$productId');
  }

  void goToCategory(String categoryId) {
    go('/categories/$categoryId');
  }
}

// Uso
context.goToProduct('123');
```

### 4. Tratamento de Deep Links

```dart
final GoRouter _router = GoRouter(
  routes: [
    GoRoute(
      path: '/product/:id',
      builder: (context, state) {
        final productId = state.pathParameters['id'];
        final fromShare = state.queryParameters['from'] == 'share';

        return ProductScreen(
          productId: productId!,
          showShareBanner: fromShare,
        );
      },
    ),
  ],
);
```

### 5. Performance

```dart
// Use lazy loading para telas pesadas
GoRoute(
  path: '/heavy-screen',
  builder: (context, state) {
    return FutureBuilder(
      future: loadHeavyData(),
      builder: (context, snapshot) {
        if (snapshot.connectionState == ConnectionState.waiting) {
          return LoadingScreen();
        }
        return HeavyScreen(data: snapshot.data);
      },
    );
  },
),
```

## 📚 Recursos Adicionais

### Documentação Oficial
- [Flutter Navigation](https://docs.flutter.dev/ui/navigation)
- [Go Router Package](https://pub.dev/packages/go_router)
- [Go Router Documentation](https://pub.dev/documentation/go_router/latest/)

### Funcionalidades Avançadas

#### Animações Personalizadas

```dart
GoRoute(
  path: '/custom-transition',
  pageBuilder: (context, state) {
    return CustomTransitionPage(
      key: state.pageKey,
      child: CustomScreen(),
      transitionsBuilder: (context, animation, secondaryAnimation, child) {
        return SlideTransition(
          position: animation.drive(
            Tween(begin: Offset(1.0, 0.0), end: Offset.zero),
          ),
          child: child,
        );
      },
    );
  },
),
```

#### Rotas Condicionais

```dart
GoRoute(
  path: '/admin',
  redirect: (context, state) {
    if (!UserService.isAdmin()) {
      return '/unauthorized';
    }
    return null;
  },
  builder: (context, state) => AdminScreen(),
),
```

#### Query Parameters

```dart
GoRoute(
  path: '/search',
  builder: (context, state) {
    final query = state.queryParameters['q'] ?? '';
    final category = state.queryParameters['category'];

    return SearchScreen(
      query: query,
      category: category,
    );
  },
),

// Navegação: context.go('/search?q=flutter&category=mobile');
```

### Packages Complementares

- **go_router_builder**: Geração de código para rotas tipadas
- **routemaster**: Alternativa ao go_router
- **auto_route**: Geração automática de rotas
- **fluro**: Router simples e leve

### Dicas de Debug

```dart
// Habilitar logs do go_router
import 'package:logging/logging.dart';

void main() {
  Logger.root.level = Level.ALL;
  Logger.root.onRecord.listen((record) {
    print('${record.level.name}: ${record.time}: ${record.message}');
  });

  runApp(MyApp());
}
```

---

## 🎉 Conclusão

A navegação no Flutter evoluiu significativamente com o Navigator 2.0 e o go_router. O go_router oferece uma API declarativa poderosa que simplifica:

- **Deep linking** nativo
- **Navegação tipada** e segura
- **Redirecionamento** baseado em estado
- **Layouts persistentes** com ShellRoute
- **Tratamento de erros** robusto

Para novos projetos, recomenda-se usar o **go_router** pela sua simplicidade e recursos avançados. Para projetos existentes, a migração gradual é possível mantendo compatibilidade com Navigator 1.0.

**Happy Navigation! 🚀**
