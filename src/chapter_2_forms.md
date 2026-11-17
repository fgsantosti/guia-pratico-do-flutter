# 2.2 - Formulários e Validação

## 📋 Índice

1. [Introdução](#-introdução)
2. [Passo 1: TextField e TextEditingController](#-passo-1-textfield-e-texteditingcontroller)
3. [Passo 2: Personalizando o Campo](#-passo-2-personalizando-o-campo)
4. [Passo 3: Form e TextFormField](#-passo-3-form-e-textformfield)
5. [Passo 4: Validadores Reutilizáveis](#-passo-4-validadores-reutilizáveis)
6. [Passo 5: Foco e Teclado](#-passo-5-foco-e-teclado)
7. [Passo 6: Máscaras (CPF, Telefone, CEP)](#-passo-6-máscaras-cpf-telefone-cep)
8. [Passo 7: Outros Campos de Formulário](#-passo-7-outros-campos-de-formulário)
9. [Projeto Prático: Tela de Cadastro](#-projeto-prático-tela-de-cadastro)
10. [Melhores Práticas](#-melhores-práticas)
11. [Recursos Adicionais](#-recursos-adicionais)

## 🚀 Introdução

Login, cadastro, busca, checkout: quase todo app tem formulários. Neste capítulo você vai aprender a capturar o que o usuário digita, **validar** os dados antes de enviar e oferecer uma boa experiência com máscaras, foco e teclado adequados.

## ⌨ Passo 1: TextField e TextEditingController

O `TextField` é o campo de texto básico. Para ler ou alterar o que foi digitado, usamos um `TextEditingController`:

```dart
import 'package:flutter/material.dart';

class NomePage extends StatefulWidget {
  const NomePage({super.key});

  @override
  State<NomePage> createState() => _NomePageState();
}

class _NomePageState extends State<NomePage> {
  final _controller = TextEditingController();
  String _saudacao = '';

  @override
  void dispose() {
    _controller.dispose(); // Sempre libere o controller!
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Saudação')),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            TextField(
              controller: _controller,
              decoration: const InputDecoration(labelText: 'Seu nome'),
              onChanged: (texto) => print('Digitando: $texto'),
            ),
            const SizedBox(height: 16),
            ElevatedButton(
              onPressed: () => setState(() => _saudacao = 'Olá, ${_controller.text}!'),
              child: const Text('Cumprimentar'),
            ),
            const SizedBox(height: 16),
            Text(_saudacao, style: const TextStyle(fontSize: 24)),
          ],
        ),
      ),
    );
  }
}
```

### Conceitos aprendidos:
- `TextEditingController`: guarda o texto do campo (`controller.text`)
- `controller.clear()`: apaga o conteúdo; `controller.text = 'x'`: define um valor
- `onChanged`: chamado a cada tecla digitada
- `dispose()`: controllers ocupam memória e precisam ser liberados

## 🎨 Passo 2: Personalizando o Campo

O `InputDecoration` controla a aparência do campo:

```dart
TextField(
  decoration: InputDecoration(
    labelText: 'E-mail',                    // Rótulo que "sobe" ao focar
    hintText: 'nome@exemplo.com',           // Exemplo dentro do campo
    helperText: 'Usaremos para o login',    // Texto de ajuda embaixo
    prefixIcon: const Icon(Icons.email),    // Ícone à esquerda
    suffixIcon: IconButton(                 // Ícone à direita
      icon: const Icon(Icons.clear),
      onPressed: () {},
    ),
    border: const OutlineInputBorder(),     // Borda em volta
    filled: true,                           // Fundo preenchido
  ),
  keyboardType: TextInputType.emailAddress, // Teclado com @
  textCapitalization: TextCapitalization.none,
  maxLength: 100,                           // Mostra o contador 0/100
)
```

### Tipos de teclado

| `keyboardType` | Uso |
|----------------|-----|
| `TextInputType.text` | Texto comum (padrão) |
| `TextInputType.emailAddress` | E-mail (mostra o @) |
| `TextInputType.number` | Apenas números |
| `TextInputType.phone` | Telefone |
| `TextInputType.multiline` | Texto com várias linhas (use com `maxLines: null`) |
| `TextInputType.url` | Endereços web |

### Campo de senha com botão de mostrar

```dart
class CampoSenha extends StatefulWidget {
  const CampoSenha({super.key, this.controller});
  final TextEditingController? controller;

  @override
  State<CampoSenha> createState() => _CampoSenhaState();
}

class _CampoSenhaState extends State<CampoSenha> {
  bool _oculta = true;

  @override
  Widget build(BuildContext context) {
    return TextField(
      controller: widget.controller,
      obscureText: _oculta,
      decoration: InputDecoration(
        labelText: 'Senha',
        prefixIcon: const Icon(Icons.lock),
        border: const OutlineInputBorder(),
        suffixIcon: IconButton(
          icon: Icon(_oculta ? Icons.visibility : Icons.visibility_off),
          onPressed: () => setState(() => _oculta = !_oculta),
        ),
      ),
    );
  }
}
```

## ✅ Passo 3: Form e TextFormField

Para validar vários campos de uma vez, use um `Form` com `TextFormField`. Cada campo recebe uma função `validator` que devolve:

- `null` → o valor é **válido**
- uma `String` → a **mensagem de erro** exibida embaixo do campo

```dart
class LoginPage extends StatefulWidget {
  const LoginPage({super.key});

  @override
  State<LoginPage> createState() => _LoginPageState();
}

class _LoginPageState extends State<LoginPage> {
  // A chave dá acesso ao estado do formulário
  final _formKey = GlobalKey<FormState>();
  final _email = TextEditingController();
  final _senha = TextEditingController();

  @override
  void dispose() {
    _email.dispose();
    _senha.dispose();
    super.dispose();
  }

  void _entrar() {
    // Executa todos os validators. Retorna true se todos devolverem null.
    if (_formKey.currentState!.validate()) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Entrando como ${_email.text}...')),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Login')),
      body: Form(
        key: _formKey,
        child: ListView(
          padding: const EdgeInsets.all(16),
          children: [
            TextFormField(
              controller: _email,
              keyboardType: TextInputType.emailAddress,
              decoration: const InputDecoration(labelText: 'E-mail'),
              validator: (valor) {
                if (valor == null || valor.trim().isEmpty) {
                  return 'Informe o e-mail';
                }
                if (!valor.contains('@')) {
                  return 'E-mail inválido';
                }
                return null;
              },
            ),
            const SizedBox(height: 12),
            TextFormField(
              controller: _senha,
              obscureText: true,
              decoration: const InputDecoration(labelText: 'Senha'),
              validator: (valor) =>
                  (valor == null || valor.length < 6) ? 'Mínimo de 6 caracteres' : null,
            ),
            const SizedBox(height: 24),
            FilledButton(onPressed: _entrar, child: const Text('Entrar')),
          ],
        ),
      ),
    );
  }
}
```

### Conceitos aprendidos:
- `Form`: agrupa os campos e coordena a validação
- `GlobalKey<FormState>`: permite chamar `validate()`, `save()` e `reset()` no formulário
- `TextFormField`: um `TextField` que sabe participar de um `Form`
- `validator`: devolve `null` (válido) ou a mensagem de erro

### Quando validar?

```dart
Form(
  key: _formKey,
  // Valida enquanto o usuário digita, mas só depois da primeira interação
  autovalidateMode: AutovalidateMode.onUserInteraction,
  child: ...,
)
```

| `AutovalidateMode` | Comportamento |
|--------------------|---------------|
| `disabled` (padrão) | Só valida ao chamar `validate()` |
| `onUserInteraction` | Valida a cada alteração depois que o usuário mexer no campo |
| `always` | Valida sempre, mesmo antes de o usuário digitar (mostra erros cedo demais) |

### Comparando dois campos (confirmar senha)

```dart
TextFormField(
  obscureText: true,
  decoration: const InputDecoration(labelText: 'Confirmar senha'),
  validator: (valor) => valor != _senha.text ? 'As senhas não conferem' : null,
)
```

## ♻ Passo 4: Validadores Reutilizáveis

Em vez de repetir as regras em cada tela, crie uma classe com validadores:

```dart
// lib/utils/validadores.dart
class Validadores {
  static String? obrigatorio(String? valor, {String campo = 'Este campo'}) {
    if (valor == null || valor.trim().isEmpty) return '$campo é obrigatório';
    return null;
  }

  static String? email(String? valor) {
    final erro = obrigatorio(valor, campo: 'O e-mail');
    if (erro != null) return erro;

    final regex = RegExp(r'^[\w.+-]+@[\w-]+\.[\w.-]+$');
    return regex.hasMatch(valor!.trim()) ? null : 'E-mail inválido';
  }

  static String? senha(String? valor) {
    if (valor == null || valor.length < 8) return 'Mínimo de 8 caracteres';
    if (!valor.contains(RegExp(r'[A-Z]'))) return 'Inclua uma letra maiúscula';
    if (!valor.contains(RegExp(r'[0-9]'))) return 'Inclua um número';
    return null;
  }

  static String? cpf(String? valor) {
    final numeros = (valor ?? '').replaceAll(RegExp(r'\D'), '');
    if (numeros.length != 11) return 'CPF deve ter 11 dígitos';
    if (RegExp(r'^(\d)\1{10}$').hasMatch(numeros)) return 'CPF inválido';

    // Calcula os dois dígitos verificadores
    int digito(int tamanho) {
      var soma = 0;
      for (var i = 0; i < tamanho; i++) {
        soma += int.parse(numeros[i]) * (tamanho + 1 - i);
      }
      final resto = (soma * 10) % 11;
      return resto == 10 ? 0 : resto;
    }

    final valido = digito(9) == int.parse(numeros[9]) &&
        digito(10) == int.parse(numeros[10]);
    return valido ? null : 'CPF inválido';
  }

  /// Combina vários validadores: devolve o primeiro erro encontrado
  static String? Function(String?) combinar(List<String? Function(String?)> regras) {
    return (valor) {
      for (final regra in regras) {
        final erro = regra(valor);
        if (erro != null) return erro;
      }
      return null;
    };
  }
}
```

Usando:

```dart
TextFormField(validator: Validadores.email)
TextFormField(validator: Validadores.cpf)
TextFormField(
  validator: Validadores.combinar([
    (v) => Validadores.obrigatorio(v, campo: 'O nome'),
    (v) => v!.trim().split(' ').length < 2 ? 'Informe nome e sobrenome' : null,
  ]),
)
```

> 💡 Validadores são funções puras e simples de testar. Veja o Capítulo 7 para escrever testes de unidade para eles.

## 🎯 Passo 5: Foco e Teclado

### Ir para o próximo campo ao apertar "Enter"

```dart
TextFormField(
  textInputAction: TextInputAction.next, // Botão "próximo" no teclado
  decoration: const InputDecoration(labelText: 'Nome'),
),
TextFormField(
  textInputAction: TextInputAction.next,
  decoration: const InputDecoration(labelText: 'E-mail'),
),
TextFormField(
  textInputAction: TextInputAction.done, // Botão "concluído" no último campo
  onFieldSubmitted: (_) => _salvar(),
  decoration: const InputDecoration(labelText: 'Senha'),
),
```

Com `TextInputAction.next`, o Flutter move o foco para o próximo campo automaticamente.

### Controlando o foco manualmente

```dart
final _focoSenha = FocusNode();

// No campo de e-mail:
onFieldSubmitted: (_) => _focoSenha.requestFocus(),

// No campo de senha:
focusNode: _focoSenha,

// Lembre-se de liberar no dispose():
_focoSenha.dispose();
```

### Fechar o teclado

```dart
FocusScope.of(context).unfocus();
```

Para fechar ao tocar fora dos campos, envolva a tela em um `GestureDetector`:

```dart
GestureDetector(
  onTap: () => FocusScope.of(context).unfocus(),
  child: Scaffold(...),
)
```

## 🎭 Passo 6: Máscaras (CPF, Telefone, CEP)

Máscaras formatam o texto enquanto o usuário digita (`12345678900` → `123.456.789-00`). O pacote [`mask_text_input_formatter`](https://pub.dev/packages/mask_text_input_formatter) facilita:

```bash
flutter pub add mask_text_input_formatter
```

```dart
import 'package:flutter/services.dart';
import 'package:mask_text_input_formatter/mask_text_input_formatter.dart';

final mascaraCpf = MaskTextInputFormatter(mask: '###.###.###-##', filter: {'#': RegExp(r'[0-9]')});
final mascaraCep = MaskTextInputFormatter(mask: '#####-###', filter: {'#': RegExp(r'[0-9]')});
final mascaraTelefone = MaskTextInputFormatter(mask: '(##) #####-####', filter: {'#': RegExp(r'[0-9]')});

TextFormField(
  decoration: const InputDecoration(labelText: 'CPF'),
  keyboardType: TextInputType.number,
  inputFormatters: [mascaraCpf],
  validator: Validadores.cpf,
)

// Valor sem a máscara, para enviar à API
final cpfSemMascara = mascaraCpf.getUnmaskedText(); // "12345678900"
```

### Formatadores nativos do Flutter

Sem pacotes, o Flutter oferece formatadores simples:

```dart
TextField(
  inputFormatters: [
    FilteringTextInputFormatter.digitsOnly,  // Só números
    LengthLimitingTextInputFormatter(4),     // Máximo de 4 caracteres
  ],
)
```

## 🗳 Passo 7: Outros Campos de Formulário

### Dropdown

```dart
String? _estado;

DropdownButtonFormField<String>(
  value: _estado,
  decoration: const InputDecoration(labelText: 'Estado'),
  items: const ['SP', 'RJ', 'MG', 'PI', 'BA']
      .map((uf) => DropdownMenuItem(value: uf, child: Text(uf)))
      .toList(),
  onChanged: (valor) => setState(() => _estado = valor),
  validator: (valor) => valor == null ? 'Selecione o estado' : null,
)
```

> 💡 Nas versões mais recentes do Flutter, o parâmetro `value` do `DropdownButtonFormField` foi renomeado para `initialValue`. Se o editor mostrar um aviso de "deprecated", basta trocar o nome.

### Checkbox, Switch e Radio

```dart
bool _aceitouTermos = false;
bool _notificacoes = true;

CheckboxListTile(
  value: _aceitouTermos,
  onChanged: (v) => setState(() => _aceitouTermos = v ?? false),
  title: const Text('Li e aceito os termos de uso'),
  controlAffinity: ListTileControlAffinity.leading,
)

SwitchListTile(
  value: _notificacoes,
  onChanged: (v) => setState(() => _notificacoes = v),
  title: const Text('Receber notificações'),
)
```

### Seletor de data

```dart
DateTime? _nascimento;

Future<void> _escolherData() async {
  final data = await showDatePicker(
    context: context,
    initialDate: DateTime(2000),
    firstDate: DateTime(1900),
    lastDate: DateTime.now(),
  );
  if (data != null) setState(() => _nascimento = data);
}

ListTile(
  leading: const Icon(Icons.cake),
  title: Text(_nascimento == null
      ? 'Data de nascimento'
      : '${_nascimento!.day}/${_nascimento!.month}/${_nascimento!.year}'),
  onTap: _escolherData,
)
```

> 💡 Para o calendário aparecer em português, configure a internacionalização do app (veja o Capítulo 10).

## 📝 Projeto Prático: Tela de Cadastro

Juntando tudo em uma tela completa (usa a classe `Validadores` do Passo 4 e o pacote de máscaras do Passo 6):

```dart
import 'package:flutter/material.dart';
import 'package:mask_text_input_formatter/mask_text_input_formatter.dart';

void main() => runApp(const MaterialApp(home: CadastroPage()));

class CadastroPage extends StatefulWidget {
  const CadastroPage({super.key});

  @override
  State<CadastroPage> createState() => _CadastroPageState();
}

class _CadastroPageState extends State<CadastroPage> {
  final _formKey = GlobalKey<FormState>();
  final _nome = TextEditingController();
  final _email = TextEditingController();
  final _senha = TextEditingController();
  final _mascaraTelefone = MaskTextInputFormatter(
    mask: '(##) #####-####',
    filter: {'#': RegExp(r'[0-9]')},
  );

  bool _senhaOculta = true;
  bool _aceitouTermos = false;
  bool _enviando = false;

  @override
  void dispose() {
    _nome.dispose();
    _email.dispose();
    _senha.dispose();
    super.dispose();
  }

  Future<void> _cadastrar() async {
    FocusScope.of(context).unfocus();

    if (!_formKey.currentState!.validate()) return;

    if (!_aceitouTermos) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('Você precisa aceitar os termos')),
      );
      return;
    }

    setState(() => _enviando = true);
    await Future.delayed(const Duration(seconds: 2)); // Simula o envio à API
    if (!mounted) return;
    setState(() => _enviando = false);

    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text('Bem-vindo(a), ${_nome.text.split(' ').first}! 🎉')),
    );
    _formKey.currentState!.reset();
    _nome.clear();
    _email.clear();
    _senha.clear();
    setState(() => _aceitouTermos = false);
  }

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: () => FocusScope.of(context).unfocus(),
      child: Scaffold(
        appBar: AppBar(title: const Text('Criar conta')),
        body: Form(
          key: _formKey,
          autovalidateMode: AutovalidateMode.onUserInteraction,
          child: ListView(
            padding: const EdgeInsets.all(16),
            children: [
              TextFormField(
                controller: _nome,
                textCapitalization: TextCapitalization.words,
                textInputAction: TextInputAction.next,
                decoration: const InputDecoration(
                  labelText: 'Nome completo',
                  prefixIcon: Icon(Icons.person),
                  border: OutlineInputBorder(),
                ),
                validator: (v) => Validadores.obrigatorio(v, campo: 'O nome'),
              ),
              const SizedBox(height: 16),
              TextFormField(
                controller: _email,
                keyboardType: TextInputType.emailAddress,
                textInputAction: TextInputAction.next,
                decoration: const InputDecoration(
                  labelText: 'E-mail',
                  prefixIcon: Icon(Icons.email),
                  border: OutlineInputBorder(),
                ),
                validator: Validadores.email,
              ),
              const SizedBox(height: 16),
              TextFormField(
                keyboardType: TextInputType.phone,
                textInputAction: TextInputAction.next,
                inputFormatters: [_mascaraTelefone],
                decoration: const InputDecoration(
                  labelText: 'Celular',
                  hintText: '(11) 91234-5678',
                  prefixIcon: Icon(Icons.phone),
                  border: OutlineInputBorder(),
                ),
                validator: (_) => _mascaraTelefone.getUnmaskedText().length == 11
                    ? null
                    : 'Celular incompleto',
              ),
              const SizedBox(height: 16),
              TextFormField(
                controller: _senha,
                obscureText: _senhaOculta,
                textInputAction: TextInputAction.done,
                onFieldSubmitted: (_) => _cadastrar(),
                decoration: InputDecoration(
                  labelText: 'Senha',
                  helperText: 'Mínimo de 8 caracteres, com maiúscula e número',
                  prefixIcon: const Icon(Icons.lock),
                  border: const OutlineInputBorder(),
                  suffixIcon: IconButton(
                    icon: Icon(_senhaOculta ? Icons.visibility : Icons.visibility_off),
                    onPressed: () => setState(() => _senhaOculta = !_senhaOculta),
                  ),
                ),
                validator: Validadores.senha,
              ),
              const SizedBox(height: 8),
              CheckboxListTile(
                value: _aceitouTermos,
                onChanged: (v) => setState(() => _aceitouTermos = v ?? false),
                title: const Text('Li e aceito os termos de uso'),
                controlAffinity: ListTileControlAffinity.leading,
                contentPadding: EdgeInsets.zero,
              ),
              const SizedBox(height: 16),
              SizedBox(
                height: 48,
                child: FilledButton(
                  onPressed: _enviando ? null : _cadastrar,
                  child: _enviando
                      ? const SizedBox(
                          width: 24,
                          height: 24,
                          child: CircularProgressIndicator(strokeWidth: 2, color: Colors.white),
                        )
                      : const Text('Cadastrar'),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

### Conceitos aprendidos:
- `ListView` no lugar de `Column` garante a rolagem quando o teclado aparece
- `onPressed: null` desabilita o botão durante o envio, evitando cadastros duplicados
- `reset()` limpa os erros do formulário; `clear()` limpa o texto de cada controller

## ✅ Melhores Práticas

- **Valide no app e no servidor.** A validação no app melhora a experiência; a do servidor garante a segurança.
- **Mensagens claras:** diga como corrigir ("Mínimo de 8 caracteres"), não só que está errado ("Inválido").
- **Teclado certo para cada campo** (`emailAddress`, `number`, `phone`).
- **Use `textInputAction`** para o usuário navegar entre campos sem fechar o teclado.
- **Desabilite o botão** enquanto envia e mostre um indicador de carregamento.
- **Libere controllers e FocusNodes** no `dispose()`.
- **Envie os dados sem máscara** para a API.

## 📖 Recursos Adicionais

- [Cookbook: Construir um formulário com validação](https://docs.flutter.dev/cookbook/forms/validation)
- [Cookbook: Recuperar o valor de um campo](https://docs.flutter.dev/cookbook/forms/retrieve-input)
- [Cookbook: Foco e campos de texto](https://docs.flutter.dev/cookbook/forms/focus)
- [Pacote mask_text_input_formatter](https://pub.dev/packages/mask_text_input_formatter)
