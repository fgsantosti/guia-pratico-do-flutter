# 10.1 - Firebase

## 📋 Índice

1. [Introdução](#-introdução)
2. [Passo 1: Configurando o Firebase](#-passo-1-configurando-o-firebase)
3. [Passo 2: Autenticação com E-mail e Senha](#-passo-2-autenticação-com-e-mail-e-senha)
4. [Passo 3: Controlando a Sessão](#-passo-3-controlando-a-sessão)
5. [Passo 4: Banco de Dados com Cloud Firestore](#-passo-4-banco-de-dados-com-cloud-firestore)
6. [Passo 5: Lista em Tempo Real](#-passo-5-lista-em-tempo-real)
7. [Passo 6: Regras de Segurança](#-passo-6-regras-de-segurança)
8. [Passo 7: Notificações Push](#-passo-7-notificações-push)
9. [Melhores Práticas](#-melhores-práticas)
10. [Recursos Adicionais](#-recursos-adicionais)

## 🚀 Introdução

O **Firebase** é uma plataforma do Google que oferece um "back-end pronto": login de usuários, banco de dados em tempo real, armazenamento de arquivos, notificações e muito mais, sem que você precise criar e manter um servidor.

| Serviço | Para que serve | Pacote |
|---------|----------------|--------|
| Authentication | Login (e-mail, Google, Apple, telefone) | `firebase_auth` |
| Cloud Firestore | Banco de dados NoSQL em tempo real | `cloud_firestore` |
| Cloud Storage | Upload de fotos e arquivos | `firebase_storage` |
| Cloud Messaging | Notificações push | `firebase_messaging` |
| Crashlytics | Relatórios de erros do app | `firebase_crashlytics` |
| Analytics | Estatísticas de uso | `firebase_analytics` |

> 💡 O plano gratuito (**Spark**) é suficiente para estudar e para apps pequenos.

Neste capítulo vamos criar um app de **tarefas na nuvem**: o usuário faz login e vê apenas as próprias tarefas, sincronizadas em tempo real entre aparelhos.

## 🔧 Passo 1: Configurando o Firebase

### 1. Crie o projeto no console

Acesse [console.firebase.google.com](https://console.firebase.google.com), clique em **Adicionar projeto** e siga os passos.

### 2. Instale as ferramentas de linha de comando

```bash
# Firebase CLI (requer Node.js)
npm install -g firebase-tools
firebase login

# FlutterFire CLI
dart pub global activate flutterfire_cli
```

> ⚠️ Se o comando `flutterfire` não for encontrado, adicione a pasta `~/.pub-cache/bin` (Linux/macOS) ou `%LOCALAPPDATA%\Pub\Cache\bin` (Windows) ao PATH.

### 3. Conecte o app ao projeto

Na pasta do seu app Flutter:

```bash
flutterfire configure
```

O comando pergunta qual projeto usar e para quais plataformas, registra os apps no Firebase e gera o arquivo `lib/firebase_options.dart` com as configurações.

### 4. Adicione os pacotes e inicialize

```bash
flutter pub add firebase_core firebase_auth cloud_firestore
```

```dart
// lib/main.dart
import 'package:firebase_core/firebase_core.dart';
import 'package:flutter/material.dart';
import 'firebase_options.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  runApp(const MeuApp());
}
```

> 💡 Sempre que adicionar um novo serviço do Firebase ou uma nova plataforma, rode `flutterfire configure` de novo.

## 🔑 Passo 2: Autenticação com E-mail e Senha

No console do Firebase, vá em **Authentication → Método de login** e ative **E-mail/senha**.

Vamos criar um serviço que concentra as operações de login:

```dart
// lib/services/auth_service.dart
import 'package:firebase_auth/firebase_auth.dart';

class AuthService {
  final _auth = FirebaseAuth.instance;

  /// Usuário logado no momento (ou null)
  User? get usuarioAtual => _auth.currentUser;

  /// Avisa sempre que o usuário entra ou sai
  Stream<User?> get mudancas => _auth.authStateChanges();

  Future<void> cadastrar(String email, String senha) async {
    try {
      await _auth.createUserWithEmailAndPassword(email: email, password: senha);
    } on FirebaseAuthException catch (e) {
      throw _traduzir(e);
    }
  }

  Future<void> entrar(String email, String senha) async {
    try {
      await _auth.signInWithEmailAndPassword(email: email, password: senha);
    } on FirebaseAuthException catch (e) {
      throw _traduzir(e);
    }
  }

  Future<void> recuperarSenha(String email) =>
      _auth.sendPasswordResetEmail(email: email);

  Future<void> sair() => _auth.signOut();

  String _traduzir(FirebaseAuthException e) {
    return switch (e.code) {
      'invalid-email' => 'E-mail inválido.',
      'email-already-in-use' => 'Este e-mail já está cadastrado.',
      'weak-password' => 'A senha deve ter pelo menos 6 caracteres.',
      'invalid-credential' || 'wrong-password' || 'user-not-found' =>
        'E-mail ou senha incorretos.',
      'too-many-requests' => 'Muitas tentativas. Aguarde e tente novamente.',
      'network-request-failed' => 'Sem conexão com a internet.',
      _ => 'Erro inesperado (${e.code}).',
    };
  }
}
```

### Tela de login

```dart
// lib/pages/login_page.dart
import 'package:flutter/material.dart';
import '../services/auth_service.dart';

class LoginPage extends StatefulWidget {
  const LoginPage({super.key});

  @override
  State<LoginPage> createState() => _LoginPageState();
}

class _LoginPageState extends State<LoginPage> {
  final _auth = AuthService();
  final _email = TextEditingController();
  final _senha = TextEditingController();
  bool _carregando = false;

  Future<void> _executar(Future<void> Function() acao) async {
    setState(() => _carregando = true);
    try {
      await acao();
      // Não é preciso navegar: o AuthGate (Passo 3) troca a tela sozinho
    } catch (e) {
      if (!mounted) return;
      ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text('$e')));
    } finally {
      if (mounted) setState(() => _carregando = false);
    }
  }

  @override
  void dispose() {
    _email.dispose();
    _senha.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Entrar')),
      body: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            TextField(
              controller: _email,
              keyboardType: TextInputType.emailAddress,
              decoration: const InputDecoration(labelText: 'E-mail'),
            ),
            const SizedBox(height: 12),
            TextField(
              controller: _senha,
              obscureText: true,
              decoration: const InputDecoration(labelText: 'Senha'),
            ),
            const SizedBox(height: 24),
            if (_carregando)
              const CircularProgressIndicator()
            else ...[
              FilledButton(
                onPressed: () => _executar(
                  () => _auth.entrar(_email.text.trim(), _senha.text),
                ),
                child: const Text('Entrar'),
              ),
              TextButton(
                onPressed: () => _executar(
                  () => _auth.cadastrar(_email.text.trim(), _senha.text),
                ),
                child: const Text('Criar conta'),
              ),
            ],
          ],
        ),
      ),
    );
  }
}
```

> 💡 Para validar os campos antes de enviar (e-mail vazio, senha curta), use um `Form` como mostrado na seção [2.2 - Formulários](./chapter_2_forms.md).

## 🚪 Passo 3: Controlando a Sessão

O Firebase **lembra o usuário logado** mesmo depois de fechar o app. Usamos o stream `authStateChanges()` para decidir qual tela mostrar:

```dart
// lib/auth_gate.dart
import 'package:firebase_auth/firebase_auth.dart';
import 'package:flutter/material.dart';
import 'pages/login_page.dart';
import 'pages/tarefas_page.dart';

class AuthGate extends StatelessWidget {
  const AuthGate({super.key});

  @override
  Widget build(BuildContext context) {
    return StreamBuilder<User?>(
      stream: FirebaseAuth.instance.authStateChanges(),
      builder: (context, snapshot) {
        if (snapshot.connectionState == ConnectionState.waiting) {
          return const Scaffold(body: Center(child: CircularProgressIndicator()));
        }
        // Tem usuário? Mostra as tarefas. Senão, o login.
        return snapshot.hasData ? const TarefasPage() : const LoginPage();
      },
    );
  }
}

class MeuApp extends StatelessWidget {
  const MeuApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(home: AuthGate());
  }
}
```

### Conceitos aprendidos:
- `authStateChanges()`: um `Stream` que emite o usuário ao entrar e `null` ao sair
- `StreamBuilder`: como o `FutureBuilder`, mas reage a **cada** novo valor do stream
- Ao chamar `signOut()`, o stream emite `null` e o app volta ao login automaticamente

## 🗄 Passo 4: Banco de Dados com Cloud Firestore

No console, vá em **Firestore Database → Criar banco de dados**, escolha uma região próxima (ex.: `southamerica-east1`, São Paulo) e comece no **modo de produção**.

### Como o Firestore organiza os dados

Diferente do SQLite (tabelas e linhas), o Firestore usa **coleções** e **documentos**:

```
usuarios (coleção)
└── abc123 (documento: id do usuário)
    └── tarefas (subcoleção)
        ├── t1 (documento) → { titulo: "Estudar Flutter", concluida: false, criadaEm: ... }
        └── t2 (documento) → { titulo: "Ler capítulo 10", concluida: true,  criadaEm: ... }
```

### Modelo

```dart
// lib/models/tarefa.dart
import 'package:cloud_firestore/cloud_firestore.dart';

class Tarefa {
  final String? id;
  final String titulo;
  final bool concluida;
  final DateTime? criadaEm;

  const Tarefa({this.id, required this.titulo, this.concluida = false, this.criadaEm});

  factory Tarefa.fromDoc(DocumentSnapshot<Map<String, dynamic>> doc) {
    final dados = doc.data()!;
    return Tarefa(
      id: doc.id,
      titulo: dados['titulo'] as String,
      concluida: dados['concluida'] as bool? ?? false,
      criadaEm: (dados['criadaEm'] as Timestamp?)?.toDate(),
    );
  }

  Map<String, dynamic> toMap() => {
        'titulo': titulo,
        'concluida': concluida,
        'criadaEm': criadaEm == null ? FieldValue.serverTimestamp() : Timestamp.fromDate(criadaEm!),
      };
}
```

### Repositório (CRUD)

```dart
// lib/repositories/tarefa_repository.dart
import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:firebase_auth/firebase_auth.dart';
import '../models/tarefa.dart';

class TarefaRepository {
  final _db = FirebaseFirestore.instance;

  /// Coleção de tarefas do usuário logado
  CollectionReference<Map<String, dynamic>> get _colecao {
    final uid = FirebaseAuth.instance.currentUser!.uid;
    return _db.collection('usuarios').doc(uid).collection('tarefas');
  }

  /// Lista em tempo real, mais recentes primeiro
  Stream<List<Tarefa>> observar() {
    return _colecao
        .orderBy('criadaEm', descending: true)
        .snapshots()
        .map((snap) => snap.docs.map(Tarefa.fromDoc).toList());
  }

  Future<void> adicionar(String titulo) =>
      _colecao.add(Tarefa(titulo: titulo).toMap());

  Future<void> alternar(Tarefa tarefa) =>
      _colecao.doc(tarefa.id).update({'concluida': !tarefa.concluida});

  Future<void> remover(String id) => _colecao.doc(id).delete();
}
```

### Conceitos aprendidos:
- `collection()` / `doc()`: navegam pela estrutura de dados
- `add()`: cria um documento com id automático
- `update()`: altera apenas os campos informados
- `snapshots()`: um `Stream` que emite a lista **sempre que algo mudar** no servidor
- `FieldValue.serverTimestamp()`: usa a hora do servidor, e não a do celular

## 📡 Passo 5: Lista em Tempo Real

```dart
// lib/pages/tarefas_page.dart
import 'package:flutter/material.dart';
import '../models/tarefa.dart';
import '../repositories/tarefa_repository.dart';
import '../services/auth_service.dart';

class TarefasPage extends StatefulWidget {
  const TarefasPage({super.key});

  @override
  State<TarefasPage> createState() => _TarefasPageState();
}

class _TarefasPageState extends State<TarefasPage> {
  final _repo = TarefaRepository();
  late final Stream<List<Tarefa>> _stream = _repo.observar();

  Future<void> _novaTarefa() async {
    final controller = TextEditingController();
    final titulo = await showDialog<String>(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('Nova tarefa'),
        content: TextField(controller: controller, autofocus: true),
        actions: [
          TextButton(onPressed: () => Navigator.pop(context), child: const Text('Cancelar')),
          FilledButton(
            onPressed: () => Navigator.pop(context, controller.text.trim()),
            child: const Text('Salvar'),
          ),
        ],
      ),
    );
    if (titulo != null && titulo.isNotEmpty) {
      await _repo.adicionar(titulo);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Minhas tarefas'),
        actions: [
          IconButton(icon: const Icon(Icons.logout), onPressed: AuthService().sair),
        ],
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _novaTarefa,
        child: const Icon(Icons.add),
      ),
      body: StreamBuilder<List<Tarefa>>(
        stream: _stream,
        builder: (context, snapshot) {
          if (snapshot.hasError) {
            return Center(child: Text('Erro: ${snapshot.error}'));
          }
          if (!snapshot.hasData) {
            return const Center(child: CircularProgressIndicator());
          }

          final tarefas = snapshot.data!;
          if (tarefas.isEmpty) {
            return const Center(child: Text('Nenhuma tarefa. Toque em + para criar.'));
          }

          return ListView.builder(
            itemCount: tarefas.length,
            itemBuilder: (context, i) {
              final tarefa = tarefas[i];
              return Dismissible(
                key: ValueKey(tarefa.id),
                direction: DismissDirection.endToStart,
                background: Container(
                  color: Colors.red,
                  alignment: Alignment.centerRight,
                  padding: const EdgeInsets.only(right: 16),
                  child: const Icon(Icons.delete, color: Colors.white),
                ),
                onDismissed: (_) => _repo.remover(tarefa.id!),
                child: CheckboxListTile(
                  value: tarefa.concluida,
                  onChanged: (_) => _repo.alternar(tarefa),
                  title: Text(
                    tarefa.titulo,
                    style: TextStyle(
                      decoration: tarefa.concluida ? TextDecoration.lineThrough : null,
                    ),
                  ),
                ),
              );
            },
          );
        },
      ),
    );
  }
}
```

Abra o app em dois aparelhos (ou no emulador e na web) com o mesmo usuário: uma tarefa criada em um aparece **instantaneamente** no outro. ✨

> 💡 O Firestore tem **cache offline** ativado por padrão no Android e iOS: o app continua funcionando sem internet e sincroniza quando a conexão voltar.

## 🔒 Passo 6: Regras de Segurança

As regras definem quem pode ler e escrever cada dado. Elas ficam no **servidor**, então não podem ser burladas alterando o app.

No console, vá em **Firestore Database → Regras**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Cada usuário só acessa as próprias tarefas
    match /usuarios/{uid}/tarefas/{tarefaId} {
      allow read, write: if request.auth != null && request.auth.uid == uid;
    }

    // Todo o resto é bloqueado
    match /{document=**} {
      allow read, write: if false;
    }
  }
}
```

### Validando os dados recebidos

```
match /usuarios/{uid}/tarefas/{tarefaId} {
  allow read, delete: if request.auth.uid == uid;
  allow create, update: if request.auth.uid == uid
    && request.resource.data.titulo is string
    && request.resource.data.titulo.size() > 0
    && request.resource.data.titulo.size() <= 200;
}
```

> ⚠️ **Nunca** publique um app com a regra `allow read, write: if true;`. Isso deixa todo o banco aberto para qualquer pessoa na internet.

## 🔔 Passo 7: Notificações Push

O **Firebase Cloud Messaging (FCM)** envia notificações para os aparelhos, mesmo com o app fechado.

```bash
flutter pub add firebase_messaging
```

```dart
import 'package:firebase_messaging/firebase_messaging.dart';

// Precisa ser uma função de nível superior (fora de classes)
@pragma('vm:entry-point')
Future<void> _mensagemEmSegundoPlano(RemoteMessage mensagem) async {
  await Firebase.initializeApp(options: DefaultFirebaseOptions.currentPlatform);
  print('Mensagem em segundo plano: ${mensagem.messageId}');
}

Future<void> configurarNotificacoes() async {
  final messaging = FirebaseMessaging.instance;

  // Pede permissão (obrigatório no iOS e no Android 13+)
  await messaging.requestPermission();

  // Token que identifica este aparelho. Envie ao seu servidor para notificações individuais.
  final token = await messaging.getToken();
  print('Token FCM: $token');

  // App fechado ou em segundo plano
  FirebaseMessaging.onBackgroundMessage(_mensagemEmSegundoPlano);

  // App aberto na tela
  FirebaseMessaging.onMessage.listen((mensagem) {
    print('Recebida com o app aberto: ${mensagem.notification?.title}');
  });

  // Usuário tocou na notificação
  FirebaseMessaging.onMessageOpenedApp.listen((mensagem) {
    print('Abriu pela notificação: ${mensagem.data}');
  });
}
```

Chame `configurarNotificacoes()` no `main`, depois do `Firebase.initializeApp`.

### Testando

No console, vá em **Messaging → Criar primeira campanha → Notificações do Firebase**, escreva título e texto e envie para o app, ou para um aparelho específico usando o token impresso no console.

> 💡 Com o app **aberto**, o Android não exibe a notificação na barra automaticamente. Para isso, combine com o pacote `flutter_local_notifications`. No iOS, é necessário configurar uma chave APNs no console do Firebase e ativar "Push Notifications" no Xcode.

## ✅ Melhores Práticas

- **Regras de segurança desde o primeiro dia.** O app não é a proteção: as regras são.
- **Estruture os dados pensando nas consultas.** No Firestore, você paga por leitura de documento; evite baixar coleções inteiras sem `limit()`.
- **Use `serverTimestamp()`** para datas: o relógio do celular pode estar errado.
- **Esconda o Firebase atrás de repositórios.** Assim a tela não depende diretamente dele e os testes ficam mais simples (Capítulo 7).
- **Monitore o uso** no console para não ter surpresas ao ultrapassar o plano gratuito.
- **Ative o App Check** antes de publicar, para que só o seu app acesse o seu projeto.

## 📖 Recursos Adicionais

- [FlutterFire: documentação oficial](https://firebase.flutter.dev)
- [Firebase: adicionar o Firebase a um app Flutter](https://firebase.google.com/docs/flutter/setup)
- [Firebase Authentication](https://firebase.google.com/docs/auth/flutter/start)
- [Cloud Firestore](https://firebase.google.com/docs/firestore/quickstart)
- [Regras de segurança do Firestore](https://firebase.google.com/docs/firestore/security/get-started)
- [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging/flutter/client)
