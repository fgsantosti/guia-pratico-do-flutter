# Capítulo 8 - Aplicativo CRUD com SQLite

## 📁 Estrutura do Projeto

```dart
lib/
├── db_helper/
│   ├── database_conexao.dart    # Configuração e conexão com SQLite
│   └── repositorio.dart         # Operações CRUD no banco
├── model/
│   └── usuario.dart             # Modelo de dados do usuário
├── screens/
│   ├── form_usuario.dart        # Tela única para criar/editar usuário
│   └── listar_usuarios.dart     # Tela principal com lista de usuários
├── services/
│   └── usuario_service.dart     # Provider para gerenciamento de estado
└── main.dart                    # Arquivo principal da aplicação
```

## 📦 Dependências (pubspec.yaml)
```dart
dependencies:
  flutter:
    sdk: flutter
  sqflite: ^2.3.0      # Banco de dados SQLite local
  provider: ^6.1.1     # Gerenciamento de estado
  path: ^1.8.3         # Manipulação de caminhos de arquivo
```

## 🔧 Arquivos do Projeto


### 1. main.dart

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'screens/listar_usuarios.dart';
import 'services/usuario_service.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      // Fornece o UsuarioService para toda a aplicação
      create: (context) => UsuarioService(),
      child: MaterialApp(
        title: 'CRUD Usuários',
        theme: ThemeData(
          primarySwatch: Colors.blue,
          visualDensity: VisualDensity.adaptivePlatformDensity,
        ),
        home: ListarUsuarios(),
        debugShowCheckedModeBanner: false,
      ),
    );
  }
}
```

### 2. model/usuario.dart

```dart
/**
 * Modelo de dados do Usuário
 * 
 * Esta classe representa a estrutura de um usuário no banco de dados.
 * Contém métodos para converter entre objeto Dart e Map (para SQLite).
 */
class Usuario {
  int? id;           // ID único do usuário (auto increment)
  String nome;       // Nome completo do usuário
  String email;      // Email do usuário
  int idade;         // Idade do usuário

  // Construtor da classe
  Usuario({
    this.id,
    required this.nome,
    required this.email,
    required this.idade,
  });

  /**
   * Converte um Map (vindo do banco) para objeto Usuario
   * Usado quando recuperamos dados do SQLite
   */
  factory Usuario.fromMap(Map<String, dynamic> map) {
    return Usuario(
      id: map['id'],
      nome: map['nome'],
      email: map['email'],
      idade: map['idade'],
    );
  }

  /**
   * Converte o objeto Usuario para Map
   * Usado quando salvamos dados no SQLite
   */
  Map<String, dynamic> toMap() {
    return {
      'id': id,
      'nome': nome,
      'email': email,
      'idade': idade,
    };
  }

  /**
   * Método para facilitar debug e logs
   */
  @override
  String toString() {
    return 'Usuario{id: $id, nome: $nome, email: $email, idade: $idade}';
  }
}
```

### 3. db_helper/database_conexao.dart

```dart
import 'package:sqflite/sqflite.dart';
import 'package:path/path.dart';

/**
 * Classe responsável pela conexão e configuração do banco SQLite
 * 
 * Implementa o padrão Singleton para garantir uma única instância
 * do banco de dados em toda a aplicação.
 */
class DatabaseConexao {
  static final DatabaseConexao _instance = DatabaseConexao._internal();
  static Database? _database;

  // Construtor privado (Singleton)
  DatabaseConexao._internal();

  // Factory constructor que retorna sempre a mesma instância
  factory DatabaseConexao() {
    return _instance;
  }

  /**
   * Getter que retorna a instância do banco
   * Se não existir, cria uma nova conexão
   */
  Future<Database> get database async {
    if (_database != null) return _database!;
    _database = await _initDatabase();
    return _database!;
  }

  /**
   * Inicializa o banco de dados
   * Cria o arquivo do banco e as tabelas necessárias
   */
  Future<Database> _initDatabase() async {
    // Obtém o caminho padrão para bancos de dados no dispositivo
    String databasesPath = await getDatabasesPath();
    String path = join(databasesPath, 'usuarios.db');

    // Abre/cria o banco de dados
    return await openDatabase(
      path,
      version: 1,
      onCreate: _onCreate,
    );
  }

  /**
   * Cria as tabelas do banco quando ele é criado pela primeira vez
   */
  Future<void> _onCreate(Database db, int version) async {
    await db.execute('''
      CREATE TABLE usuarios (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        nome TEXT NOT NULL,
        email TEXT NOT NULL UNIQUE,
        idade INTEGER NOT NULL
      )
    ''');
  
    print('Tabela usuarios criada com sucesso!');
  }

  /**
   * Fecha a conexão com o banco
   * Útil para limpeza de recursos
   */
  Future<void> closeDatabase() async {
    if (_database != null) {
      await _database!.close();
      _database = null;
    }
  }
}
```

### 4. db_helper/repositorio.dart

```dart
import 'package:sqflite/sqflite.dart';
import '../model/usuario.dart';
import 'database_conexao.dart';

/**
 * Repositório para operações CRUD com usuários
 * 
 * Esta classe implementa o padrão Repository, centralizando
 * todas as operações de banco de dados relacionadas aos usuários.
 */
class Repositorio {
  final DatabaseConexao _databaseConexao = DatabaseConexao();

  /**
   * CREATE - Insere um novo usuário no banco
   * Retorna o ID do usuário inserido
   */
  Future<int> inserirUsuario(Usuario usuario) async {
    try {
      final db = await _databaseConexao.database;
      int id = await db.insert(
        'usuarios',
        usuario.toMap(),
        conflictAlgorithm: ConflictAlgorithm.replace,
      );
      print('Usuário inserido com ID: $id');
      return id;
    } catch (e) {
      print('Erro ao inserir usuário: $e');
      throw Exception('Erro ao inserir usuário: $e');
    }
  }

  /**
   * READ - Busca todos os usuários
   * Retorna uma lista de objetos Usuario
   */
  Future<List<Usuario>> buscarTodosUsuarios() async {
    try {
      final db = await _databaseConexao.database;
      final List<Map<String, dynamic>> maps = await db.query(
        'usuarios',
        orderBy: 'nome ASC', // Ordena por nome
      );

      // Converte cada Map em objeto Usuario
      return List.generate(maps.length, (i) {
        return Usuario.fromMap(maps[i]);
      });
    } catch (e) {
      print('Erro ao buscar usuários: $e');
      throw Exception('Erro ao buscar usuários: $e');
    }
  }

  /**
   * READ - Busca um usuário específico por ID
   * Retorna null se não encontrar
   */
  Future<Usuario?> buscarUsuarioPorId(int id) async {
    try {
      final db = await _databaseConexao.database;
      final List<Map<String, dynamic>> maps = await db.query(
        'usuarios',
        where: 'id = ?',
        whereArgs: [id],
      );

      if (maps.isNotEmpty) {
        return Usuario.fromMap(maps.first);
      }
      return null;
    } catch (e) {
      print('Erro ao buscar usuário por ID: $e');
      throw Exception('Erro ao buscar usuário: $e');
    }
  }

  /**
   * UPDATE - Atualiza os dados de um usuário existente
   * Retorna o número de linhas afetadas
   */
  Future<int> atualizarUsuario(Usuario usuario) async {
    try {
      final db = await _databaseConexao.database;
      int linhasAfetadas = await db.update(
        'usuarios',
        usuario.toMap(),
        where: 'id = ?',
        whereArgs: [usuario.id],
      );
      print('Usuário atualizado. Linhas afetadas: $linhasAfetadas');
      return linhasAfetadas;
    } catch (e) {
      print('Erro ao atualizar usuário: $e');
      throw Exception('Erro ao atualizar usuário: $e');
    }
  }

  /**
   * DELETE - Remove um usuário do banco
   * Retorna o número de linhas afetadas
   */
  Future<int> deletarUsuario(int id) async {
    try {
      final db = await _databaseConexao.database;
      int linhasAfetadas = await db.delete(
        'usuarios',
        where: 'id = ?',
        whereArgs: [id],
      );
      print('Usuário deletado. Linhas afetadas: $linhasAfetadas');
      return linhasAfetadas;
    } catch (e) {
      print('Erro ao deletar usuário: $e');
      throw Exception('Erro ao deletar usuário: $e');
    }
  }

  /**
   * Busca usuários por nome (busca parcial)
   * Útil para implementar funcionalidade de pesquisa
   */
  Future<List<Usuario>> buscarUsuariosPorNome(String nome) async {
    try {
      final db = await _databaseConexao.database;
      final List<Map<String, dynamic>> maps = await db.query(
        'usuarios',
        where: 'nome LIKE ?',
        whereArgs: ['%$nome%'],
        orderBy: 'nome ASC',
      );

      return List.generate(maps.length, (i) {
        return Usuario.fromMap(maps[i]);
      });
    } catch (e) {
      print('Erro ao buscar usuários por nome: $e');
      throw Exception('Erro ao buscar usuários: $e');
    }
  }
}
```

### 5. services/usuario_service.dart

```dart
import 'package:flutter/foundation.dart';
import '../model/usuario.dart';
import '../db_helper/repositorio.dart';

/**
 * Service Provider para gerenciamento de estado dos usuários
 * 
 * Esta classe utiliza o padrão Provider do Flutter para gerenciar
 * o estado da aplicação. Ela atua como uma camada intermediária
 * entre a UI e o repositório de dados.
 */
class UsuarioService extends ChangeNotifier {
  final Repositorio _repositorio = Repositorio();

  // Lista de usuários carregados na memória
  List<Usuario> _usuarios = [];

  // Estados de carregamento e erro
  bool _isLoading = false;
  String? _errorMessage;

  // Getters para acessar os dados privados
  List<Usuario> get usuarios => _usuarios;
  bool get isLoading => _isLoading;
  String? get errorMessage => _errorMessage;

  /**
   * Carrega todos os usuários do banco de dados
   * Atualiza a UI automaticamente através do notifyListeners()
   */
  Future<void> carregarUsuarios() async {
    _setLoading(true);
    _clearError();
  
    try {
      _usuarios = await _repositorio.buscarTodosUsuarios();
      print('${_usuarios.length} usuários carregados');
    } catch (e) {
      _setError('Erro ao carregar usuários: $e');
      print('Erro no service ao carregar usuários: $e');
    } finally {
      _setLoading(false);
    }
  }

  /**
   * Adiciona um novo usuário
   * Recarrega a lista automaticamente após inserir
   */
  Future<bool> adicionarUsuario(Usuario usuario) async {
    _setLoading(true);
    _clearError();
  
    try {
      int id = await _repositorio.inserirUsuario(usuario);
      if (id > 0) {
        await carregarUsuarios(); // Recarrega a lista
        return true;
      }
      return false;
    } catch (e) {
      _setError('Erro ao adicionar usuário: $e');
      print('Erro no service ao adicionar usuário: $e');
      return false;
    } finally {
      _setLoading(false);
    }
  }

  /**
   * Atualiza um usuário existente
   * Recarrega a lista automaticamente após atualizar
   */
  Future<bool> atualizarUsuario(Usuario usuario) async {
    _setLoading(true);
    _clearError();
  
    try {
      int linhasAfetadas = await _repositorio.atualizarUsuario(usuario);
      if (linhasAfetadas > 0) {
        await carregarUsuarios(); // Recarrega a lista
        return true;
      }
      return false;
    } catch (e) {
      _setError('Erro ao atualizar usuário: $e');
      print('Erro no service ao atualizar usuário: $e');
      return false;
    } finally {
      _setLoading(false);
    }
  }

  /**
   * Remove um usuário
   * Recarrega a lista automaticamente após deletar
   */
  Future<bool> deletarUsuario(int id) async {
    _setLoading(true);
    _clearError();
  
    try {
      int linhasAfetadas = await _repositorio.deletarUsuario(id);
      if (linhasAfetadas > 0) {
        await carregarUsuarios(); // Recarrega a lista
        return true;
      }
      return false;
    } catch (e) {
      _setError('Erro ao deletar usuário: $e');
      print('Erro no service ao deletar usuário: $e');
      return false;
    } finally {
      _setLoading(false);
    }
  }

  /**
   * Busca um usuário específico por ID
   */
  Future<Usuario?> buscarUsuarioPorId(int id) async {
    try {
      return await _repositorio.buscarUsuarioPorId(id);
    } catch (e) {
      _setError('Erro ao buscar usuário: $e');
      return null;
    }
  }

  /**
   * Busca usuários por nome
   */
  Future<void> buscarUsuariosPorNome(String nome) async {
    _setLoading(true);
    _clearError();
  
    try {
      if (nome.isEmpty) {
        await carregarUsuarios(); // Se busca vazia, carrega todos
      } else {
        _usuarios = await _repositorio.buscarUsuariosPorNome(nome);
      }
    } catch (e) {
      _setError('Erro ao buscar usuários: $e');
    } finally {
      _setLoading(false);
    }
  }

  // Métodos privados para gerenciar estado interno
  void _setLoading(bool loading) {
    _isLoading = loading;
    notifyListeners(); // Notifica a UI sobre mudanças
  }

  void _setError(String error) {
    _errorMessage = error;
    notifyListeners();
  }

  void _clearError() {
    _errorMessage = null;
    notifyListeners();
  }

  /**
   * Limpa todos os dados em memória
   * Útil para logout ou reset da aplicação
   */
  void limparDados() {
    _usuarios.clear();
    _clearError();
    notifyListeners();
  }
}
```


### 6. screens/listar_usuarios.dart

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../services/usuario_service.dart';
import '../model/usuario.dart';
import 'form_usuario.dart';

/**
 * Tela principal que exibe a lista de usuários
 * 
 * Esta tela consome o UsuarioService através do Provider
 * e exibe os dados em uma ListView. Permite navegar para
 * a tela de formulário para criar/editar usuários.
 */
class ListarUsuarios extends StatefulWidget {
  @override
  _ListarUsuariosState createState() => _ListarUsuariosState();
}

class _ListarUsuariosState extends State<ListarUsuarios> {
  final TextEditingController _searchController = TextEditingController();

  @override
  void initState() {
    super.initState();
    // Carrega os usuários assim que a tela é inicializada
    WidgetsBinding.instance.addPostFrameCallback((_) {
      Provider.of<UsuarioService>(context, listen: false).carregarUsuarios();
    });
  }

  @override
  void dispose() {
    _searchController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Lista de Usuários'),
        backgroundColor: Colors.blue,
        foregroundColor: Colors.white,
        actions: [
          // Botão para recarregar a lista
          IconButton(
            icon: Icon(Icons.refresh),
            onPressed: () {
              Provider.of<UsuarioService>(context, listen: false)
                  .carregarUsuarios();
            },
          ),
        ],
      ),
      body: Column(
        children: [
          // Campo de busca
          Padding(
            padding: EdgeInsets.all(16.0),
            child: TextField(
              controller: _searchController,
              decoration: InputDecoration(
                labelText: 'Buscar usuário',
                prefixIcon: Icon(Icons.search),
                border: OutlineInputBorder(),
                suffixIcon: IconButton(
                  icon: Icon(Icons.clear),
                  onPressed: () {
                    _searchController.clear();
                    Provider.of<UsuarioService>(context, listen: false)
                        .carregarUsuarios();
                  },
                ),
              ),
              onChanged: (value) {
                // Busca em tempo real conforme o usuário digita
                Provider.of<UsuarioService>(context, listen: false)
                    .buscarUsuariosPorNome(value);
              },
            ),
          ),
        
          // Lista de usuários
          Expanded(
            child: Consumer<UsuarioService>(
              builder: (context, usuarioService, child) {
                // Exibe loading
                if (usuarioService.isLoading) {
                  return Center(
                    child: CircularProgressIndicator(),
                  );
                }

                // Exibe erro se houver
                if (usuarioService.errorMessage != null) {
                  return Center(
                    child: Column(
                      mainAxisAlignment: MainAxisAlignment.center,
                      children: [
                        Icon(Icons.error, size: 64, color: Colors.red),
                        SizedBox(height: 16),
                        Text(
                          usuarioService.errorMessage!,
                          style: TextStyle(color: Colors.red),
                          textAlign: TextAlign.center,
                        ),
                        SizedBox(height: 16),
                        ElevatedButton(
                          onPressed: () => usuarioService.carregarUsuarios(),
                          child: Text('Tentar Novamente'),
                        ),
                      ],
                    ),
                  );
                }

                // Exibe mensagem se não há usuários
                if (usuarioService.usuarios.isEmpty) {
                  return Center(
                    child: Column(
                      mainAxisAlignment: MainAxisAlignment.center,
                      children: [
                        Icon(Icons.people_outline, size: 64, color: Colors.grey),
                        SizedBox(height: 16),
                        Text(
                          'Nenhum usuário encontrado',
                          style: TextStyle(fontSize: 18, color: Colors.grey),
                        ),
                        SizedBox(height: 16),
                        ElevatedButton.icon(
                          onPressed: () => _navegarParaFormulario(),
                          icon: Icon(Icons.add),
                          label: Text('Adicionar Primeiro Usuário'),
                        ),
                      ],
                    ),
                  );
                }

                // Exibe a lista de usuários
                return ListView.builder(
                  itemCount: usuarioService.usuarios.length,
                  itemBuilder: (context, index) {
                    final usuario = usuarioService.usuarios[index];
                    return _buildUsuarioCard(usuario, usuarioService);
                  },
                );
              },
            ),
          ),
        ],
      ),
    
      // Botão flutuante para adicionar usuário
      floatingActionButton: FloatingActionButton(
        onPressed: _navegarParaFormulario,
        child: Icon(Icons.add),
        backgroundColor: Colors.blue,
        foregroundColor: Colors.white,
      ),
    );
  }

  /**
   * Constrói o card de cada usuário na lista
   */
  Widget _buildUsuarioCard(Usuario usuario, UsuarioService usuarioService) {
    return Card(
      margin: EdgeInsets.symmetric(horizontal: 16, vertical: 8),
      elevation: 2,
      child: ListTile(
        leading: CircleAvatar(
          backgroundColor: Colors.blue,
          child: Text(
            usuario.nome.isNotEmpty ? usuario.nome[0].toUpperCase() : '?',
            style: TextStyle(color: Colors.white, fontWeight: FontWeight.bold),
          ),
        ),
        title: Text(
          usuario.nome,
          style: TextStyle(fontWeight: FontWeight.bold),
        ),
        subtitle: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(usuario.email),
            Text('${usuario.idade} anos'),
          ],
        ),
        trailing: PopupMenuButton<String>(
          onSelected: (value) {
            if (value == 'editar') {
              _navegarParaFormulario(usuario: usuario);
            } else if (value == 'deletar') {
              _confirmarDelecao(usuario, usuarioService);
            }
          },
          itemBuilder: (context) => [
            PopupMenuItem(
              value: 'editar',
              child: Row(
                children: [
                  Icon(Icons.edit, color: Colors.blue),
                  SizedBox(width: 8),
                  Text('Editar'),
                ],
              ),
            ),
            PopupMenuItem(
              value: 'deletar',
              child: Row(
                children: [
                  Icon(Icons.delete, color: Colors.red),
                  SizedBox(width: 8),
                  Text('Deletar'),
                ],
              ),
            ),
          ],
        ),
        onTap: () => _navegarParaFormulario(usuario: usuario),
      ),
    );
  }

  /**
   * Navega para a tela de formulário
   * Se usuario for null, será modo criação
   * Se usuario for fornecido, será modo edição
   */
  void _navegarParaFormulario({Usuario? usuario}) {
    Navigator.push(
      context,
      MaterialPageRoute(
        builder: (context) => FormUsuario(usuario: usuario),
      ),
    );
  }

  /**
   * Exibe dialog de confirmação antes de deletar
   */
  void _confirmarDelecao(Usuario usuario, UsuarioService usuarioService) {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: Text('Confirmar Exclusão'),
        content: Text('Deseja realmente excluir ${usuario.nome}?'),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: Text('Cancelar'),
          ),
          TextButton(
            onPressed: () async {
              Navigator.pop(context);
              bool sucesso = await usuarioService.deletarUsuario(usuario.id!);
            
              if (sucesso) {
                ScaffoldMessenger.of(context).showSnackBar(
                  SnackBar(
                    content: Text('Usuário excluído com sucesso!'),
                    backgroundColor: Colors.green,
                  ),
                );
              } else {
                ScaffoldMessenger.of(context).showSnackBar(
                  SnackBar(
                    content: Text('Erro ao excluir usuário!'),
                    backgroundColor: Colors.red,
                  ),
                );
              }
            },
            child: Text('Excluir', style: TextStyle(color: Colors.red)),
          ),
        ],
      ),
    );
  }
}
```

### 7. screens/form_usuario.dart

```dart
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../services/usuario_service.dart';
import '../model/usuario.dart';

/**
 * Tela de formulário para criar/editar usuários
 * 
 * Esta tela serve tanto para criar novos usuários quanto para
 * editar usuários existentes. O modo é determinado pelo parâmetro
 * 'usuario' passado no construtor.
 */
class FormUsuario extends StatefulWidget {
  final Usuario? usuario; // null = modo criação, objeto = modo edição

  const FormUsuario({Key? key, this.usuario}) : super(key: key);

  @override
  _FormUsuarioState createState() => _FormUsuarioState();
}

class _FormUsuarioState extends State<FormUsuario> {
  final _formKey = GlobalKey<FormState>();

  // Controladores para os campos de texto
  final _nomeController = TextEditingController();
  final _emailController = TextEditingController();
  final _idadeController = TextEditingController();

  // Determina se está em modo edição
  bool get _isEditMode => widget.usuario != null;

  @override
  void initState() {
    super.initState();
  
    // Se está editando, preenche os campos com os dados existentes
    if (_isEditMode) {
      _nomeController.text = widget.usuario!.nome;
      _emailController.text = widget.usuario!.email;
      _idadeController.text = widget.usuario!.idade.toString();
    }
  }

  @override
  void dispose() {
    // Libera os recursos dos controladores
    _nomeController.dispose();
    _emailController.dispose();
    _idadeController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(_isEditMode ? 'Editar Usuário' : 'Novo Usuário'),
        backgroundColor: Colors.blue,
        foregroundColor: Colors.white,
      ),
      body: Consumer<UsuarioService>(
        builder: (context, usuarioService, child) {
          return Padding(
            padding: EdgeInsets.all(16.0),
            child: Form(
              key: _formKey,
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.stretch,
                children: [
                  // Ícone e título
                  Icon(
                    _isEditMode ? Icons.edit : Icons.person_add,
                    size: 64,
                    color: Colors.blue,
                  ),
                  SizedBox(height: 16),
                  Text(
                    _isEditMode ? 'Editar dados do usuário' : 'Adicionar novo usuário',
                    style: Theme.of(context).textTheme.headlineSmall,
                    textAlign: TextAlign.center,
                  ),
                  SizedBox(height: 32),

                  // Campo Nome
                  TextFormField(
                    controller: _nomeController,
                    decoration: InputDecoration(
                      labelText: 'Nome Completo',
                      prefixIcon: Icon(Icons.person),
                      border: OutlineInputBorder(),
                    ),
                    validator: (value) {
                      if (value == null || value.trim().isEmpty) {
                        return 'Por favor, informe o nome';
                      }
                      if (value.trim().length < 2) {
                        return 'Nome deve ter pelo menos 2 caracteres';
                      }
                      return null;
                    },
                  ),
                  SizedBox(height: 16),

                  // Campo Email
                  TextFormField(
                    controller: _emailController,
                    keyboardType: TextInputType.emailAddress,
                    decoration: InputDecoration(
                      labelText: 'Email',
                      prefixIcon: Icon(Icons.email),
                      border: OutlineInputBorder(),
                    ),
                    validator: (value) {
                      if (value == null || value.trim().isEmpty) {
                        return 'Por favor, informe o email';
                      }
                      // Validação básica de email
                      if (!RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$').hasMatch(value)) {
                        return 'Por favor, informe um email válido';
                      }
                      return null;
                    },
                  ),
                  SizedBox(height: 16),

                  // Campo Idade
                  TextFormField(
                    controller: _idadeController,
                    keyboardType: TextInputType.number,
                    decoration: InputDecoration(
                      labelText: 'Idade',
                      prefixIcon: Icon(Icons.cake),
                      border: OutlineInputBorder(),
                    ),
                    validator: (value) {
                      if (value == null || value.trim().isEmpty) {
                        return 'Por favor, informe a idade';
                      }
                      int? idade = int.tryParse(value);
                      if (idade == null) {
                        return 'Por favor, informe uma idade válida';
                      }
                      if (idade < 1 || idade > 120) {
                        return 'Idade deve estar entre 1 e 120 anos';
                      }
                      return null;
                    },
                  ),
                  SizedBox(height: 32),

                  // Botão Salvar
                  ElevatedButton(
                    onPressed: usuarioService.isLoading ? null : _salvarUsuario,
                    style: ElevatedButton.styleFrom(
                      backgroundColor: Colors.blue,
                      foregroundColor: Colors.white,
                      padding: EdgeInsets.symmetric(vertical: 16),
                    ),
                    child: usuarioService.isLoading
                        ? Row(
                            mainAxisAlignment: MainAxisAlignment.center,
                            children: [
                              SizedBox(
                                width: 20,
                                height: 20,
                                child: CircularProgressIndicator(
                                  strokeWidth: 2,
                                  valueColor: AlwaysStoppedAnimation<Color>(Colors.white),
                                ),
                              ),
                              SizedBox(width: 12),
                              Text('Salvando...'),
                            ],
                          )
                        : Text(
                            _isEditMode ? 'Atualizar Usuário' : 'Salvar Usuário',
                            style: TextStyle(fontSize: 16),
                          ),
                  ),
                  SizedBox(height: 16),

                  // Botão Cancelar
                  OutlinedButton(
                    onPressed: usuarioService.isLoading ? null : () => Navigator.pop(context),
                    child: Text('Cancelar'),
                  ),
                ],
              ),
            ),
          );
        },
      ),
    );
  }

  /**
   * Salva ou atualiza o usuário
   */
  Future<void> _salvarUsuario() async {
    // Valida o formulário
    if (!_formKey.currentState!.validate()) {
      return;
    }

    final usuarioService = Provider.of<UsuarioService>(context, listen: false);

    // Cria objeto Usuario com os dados do formulário
    final usuario = Usuario(
      id: _isEditMode ? widget.usuario!.id : null,
      nome: _nomeController.text.trim(),
      email: _emailController.text.trim(),
      idade: int.parse(_idadeController.text.trim()),
    );

    bool sucesso;
    String mensagem;

    try {
      if (_isEditMode) {
        // Modo edição
        sucesso = await usuarioService.atualizarUsuario(usuario);
        mensagem = sucesso 
            ? 'Usuário atualizado com sucesso!' 
            : 'Erro ao atualizar usuário!';
      } else {
        // Modo criação
        sucesso = await usuarioService.adicionarUsuario(usuario);
        mensagem = sucesso 
            ? 'Usuário criado com sucesso!' 
            : 'Erro ao criar usuário!';
      }

      // Exibe mensagem de feedback
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text(mensagem),
          backgroundColor: sucesso ? Colors.green : Colors.red,
        ),
      );

      // Se sucesso, volta para a tela anterior
      if (sucesso) {
        Navigator.pop(context);
      }
    } catch (e) {
      // Trata erros não capturados
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: Text('Erro inesperado: $e'),
          backgroundColor: Colors.red,
        ),
      );
    }
  }
}
```


## 🔍 Explicação dos Conceitos
### SQLite (sqflite)
- Banco local: Armazena dados diretamente no dispositivo
- Operações CRUD: Create, Read, Update, Delete
- SQL nativo: Permite consultas complexas
- Persistência: Dados mantidos entre sessões do app

### Provider
- Gerenciamento de estado: Centraliza o estado da aplicação
- Reatividade: UI atualiza automaticamente quando dados mudam
- ChangeNotifier: Notifica widgets sobre mudanças
- Consumer: Widget que escuta mudanças do Provider

### Arquitetura Utilizada
- Model: Representa os dados (Usuario)
- Repository: Acesso aos dados (operações de banco)
- Service: Lógica de negócio e gerenciamento de estado
- View: Interface do usuário (telas)

## 🚀 Como Usar
- Adicione as dependências no pubspec.yaml
- Copie todos os arquivos para suas respectivas pastas
- Execute `flutter pub get`
- Execute o aplicativo com `flutter run`

O aplicativo permite criar, listar, editar e excluir usuários com persistência local usando SQLite.
