# Apêndice B - Dicas e Truques

## 📋 Índice

1. [Atalhos do Editor](#-atalhos-do-editor)
2. [Hot Reload x Hot Restart](#-hot-reload-x-hot-restart)
3. [Snippets](#-snippets)
4. [Comandos Úteis do Flutter](#-comandos-úteis-do-flutter)
5. [Organização de Pastas](#-organização-de-pastas)
6. [Análise de Código com flutter_lints](#-análise-de-código-com-flutter_lints)
7. [Boas Práticas de Código Dart](#-boas-práticas-de-código-dart)
8. [Depuração](#-depuração)
9. [Erros Mais Comuns e Como Resolver](#-erros-mais-comuns-e-como-resolver)

## ⌨ Atalhos do Editor

### VS Code

| Atalho (Windows/Linux) | macOS | Ação |
|------------------------|-------|------|
| `Ctrl + .` | `Cmd + .` | **Ações rápidas**: envolver com widget, extrair widget, importar |
| `Ctrl + Espaço` | `Ctrl + Espaço` | Autocompletar |
| `F5` | `F5` | Executar com depuração |
| `Ctrl + F5` | `Ctrl + F5` | Executar sem depuração |
| `Ctrl + S` | `Cmd + S` | Salvar (dispara o Hot Reload) |
| `Shift + Alt + F` | `Shift + Option + F` | Formatar o código |
| `F12` | `F12` | Ir para a definição |
| `Shift + F12` | `Shift + F12` | Ver onde é usado |
| `F2` | `F2` | Renomear em todo o projeto |
| `Ctrl + Shift + P` | `Cmd + Shift + P` | Paleta de comandos |
| `Ctrl + P` | `Cmd + P` | Abrir arquivo pelo nome |

### O atalho mais útil: `Ctrl + .`

Com o cursor sobre um widget, `Ctrl + .` abre um menu com:

- **Wrap with widget**: envolve com qualquer widget
- **Wrap with Padding / Center / Column / Row / Container**
- **Wrap with Builder / StreamBuilder**
- **Extract Widget**: transforma o trecho em uma nova classe
- **Remove this widget**: remove o widget e mantém o filho
- **Move widget up / down**: muda a ordem dentro de uma lista de filhos

### Vírgula final = formatação melhor

Coloque uma **vírgula depois do último parâmetro** e o formatador quebra as linhas de forma legível:

```dart
// Sem vírgula final: tudo em uma linha
Container(padding: const EdgeInsets.all(8), child: const Text('Olá'))

// Com vírgula final, após formatar
Container(
  padding: const EdgeInsets.all(8),
  child: const Text('Olá'),
)
```

## 🔥 Hot Reload x Hot Restart

| | Hot Reload ⚡ | Hot Restart 🔄 | Reiniciar o app 🛑 |
|---|--------------|----------------|--------------------|
| Atalho no terminal | `r` | `R` | `q` e `flutter run` |
| Velocidade | ~1 segundo | Alguns segundos | Mais lento |
| Mantém o estado | ✅ | ❌ | ❌ |
| Quando usar | Mudanças visuais e no `build` | Mudanças no `initState`, em variáveis globais ou no `main` | Mudanças nativas, no `pubspec.yaml`, assets novos, splash, ícone |

## ✂ Snippets

Com a extensão Flutter, digite e pressione `Enter`:

| Snippet | Cria |
|---------|------|
| `stless` | `StatelessWidget` |
| `stful` | `StatefulWidget` com a classe `State` |
| `stanim` | `StatefulWidget` com `AnimationController` |
| `stinit` / `initS` | Método `initState` |
| `dis` | Método `dispose` |

## 🖥 Comandos Úteis do Flutter

```bash
flutter create nome_app --org br.com.suaempresa   # Projeto já com o identificador certo
flutter create --platforms=android,ios nome_app  # Só as plataformas desejadas

flutter pub get          # Baixa as dependências
flutter pub add pacote   # Adiciona um pacote
flutter pub remove pacote
flutter pub outdated     # Mostra pacotes desatualizados
flutter pub upgrade      # Atualiza dentro das versões permitidas

flutter clean            # Apaga arquivos de build (resolve muitos erros estranhos)
flutter analyze          # Procura problemas no código
dart format .            # Formata todos os arquivos
dart fix --apply         # Corrige automaticamente avisos do analisador

flutter run --release    # Roda em modo release
flutter logs             # Mostra os logs do aparelho
flutter devices          # Lista aparelhos
```

> 💡 **A receita mágica** para erros estranhos de build: `flutter clean && flutter pub get` e rode de novo.

## 🗂 Organização de Pastas

### Para projetos pequenos (por tipo)

```
lib/
├── main.dart
├── models/        # Classes de dados
├── services/      # API, banco de dados
├── providers/     # Gerenciamento de estado
├── pages/         # Telas
├── widgets/       # Componentes reutilizáveis
└── utils/         # Funções auxiliares, constantes, validadores
```

### Para projetos médios e grandes (por funcionalidade)

```
lib/
├── main.dart
├── core/                # Compartilhado: tema, rede, injeção de dependência
└── features/
    ├── auth/
    │   ├── data/
    │   ├── domain/
    │   └── presentation/
    └── produtos/
        ├── data/
        ├── domain/
        └── presentation/
```

Mais detalhes no [Capítulo 10](./chapter_10.md#-arquitetura-e-injeção-de-dependência).

### Convenções de nomes

| Item | Padrão | Exemplo |
|------|--------|---------|
| Arquivos e pastas | `snake_case` | `tela_login.dart` |
| Classes, enums, typedefs | `PascalCase` | `TelaLogin`, `StatusPedido` |
| Variáveis, funções, parâmetros | `camelCase` | `nomeUsuario`, `buscarDados()` |
| Constantes | `camelCase` | `const corPrimaria = ...` |
| Membros privados | Prefixo `_` | `_contador`, `_carregar()` |

## 🔍 Análise de Código com flutter_lints

Projetos novos já vêm com o `flutter_lints` e o arquivo `analysis_options.yaml`. Você pode ativar regras extras:

```yaml
include: package:flutter_lints/flutter.yaml

linter:
  rules:
    prefer_const_constructors: true
    prefer_const_declarations: true
    prefer_final_locals: true
    require_trailing_commas: true
    avoid_print: true
    sort_child_properties_last: true
    use_build_context_synchronously: true
```

```bash
flutter analyze      # Lista os problemas
dart fix --apply     # Corrige o que for possível automaticamente
```

## ✨ Boas Práticas de Código Dart

### Prefira `final` e `const`

```dart
final nome = 'Ana';          // Não muda depois de atribuído
const pi = 3.14159;          // Conhecido em tempo de compilação
var contador = 0;            // Só quando realmente vai mudar
```

### Use os operadores de nulo

```dart
final nome = usuario?.nome ?? 'Visitante';   // Acesso seguro + valor padrão
lista ??= [];                                // Atribui só se for nulo
```

### Widgets pequenos e reutilizáveis

Quando o `build` ficar longo, extraia partes em **classes de widget** (e não em métodos):

```dart
// ❌ Método: é reconstruído junto com o pai sempre
Widget _buildCabecalho() => const Text('Título');

// ✅ Classe: pode ser const e o Flutter otimiza as reconstruções
class Cabecalho extends StatelessWidget {
  const Cabecalho({super.key});

  @override
  Widget build(BuildContext context) => const Text('Título');
}
```

### Evite números e textos "mágicos"

```dart
// ❌
padding: const EdgeInsets.all(16),
color: const Color(0xFF1E88E5),

// ✅ Centralize em constantes ou no tema
abstract class Espacos {
  static const pequeno = 8.0;
  static const medio = 16.0;
  static const grande = 24.0;
}

padding: const EdgeInsets.all(Espacos.medio),
color: Theme.of(context).colorScheme.primary,
```

### Use `context` com cuidado após `await`

```dart
onPressed: () async {
  await salvar();
  if (!context.mounted) return;       // A tela pode ter sido fechada
  Navigator.pop(context);
}
```

## 🐞 Depuração

### print x debugPrint x log

```dart
import 'dart:developer';

print('Simples');                         // Pode ser cortado em textos longos no Android
debugPrint('Não é cortado');              // Seguro para textos longos
log('Com categoria', name: 'API');        // Aparece organizado no DevTools
```

### Breakpoints

Clique à esquerda do número da linha no VS Code para criar um **breakpoint**. Rode com `F5`: a execução para ali e você pode inspecionar variáveis, avançar linha a linha (`F10`) e entrar em funções (`F11`).

### Flutter DevTools

Com o app rodando pelo VS Code, abra a paleta de comandos e use **"Flutter: Open DevTools"**:

- **Inspector:** árvore de widgets, tamanhos e espaçamentos
- **Performance:** frames lentos (*jank*)
- **Network:** requisições HTTP feitas pelo app
- **Memory:** uso de memória e vazamentos
- **Logging:** todos os logs em um só lugar

### Ver os limites dos widgets

```dart
import 'package:flutter/rendering.dart';

void main() {
  debugPaintSizeEnabled = true; // Desenha as bordas de todos os widgets
  runApp(const MeuApp());
}
```

## 🚑 Erros Mais Comuns e Como Resolver

| Mensagem | O que significa | Solução |
|----------|-----------------|---------|
| `A RenderFlex overflowed by N pixels` | Conteúdo maior que o espaço da `Row`/`Column` | `Expanded`, `Flexible` ou `SingleChildScrollView` ([seção 2.1](./chapter_2_layouts.md)) |
| `Vertical viewport was given unbounded height` | `ListView` dentro de `Column` | Envolva o `ListView` em `Expanded` |
| `setState() called after dispose()` | `setState` depois que a tela fechou | `if (!mounted) return;` antes do `setState` |
| `Null check operator used on a null value` | Uso de `!` em algo que era `null` | Verifique antes ou use `?.` e `??` |
| `LateInitializationError` | Variável `late` lida antes de receber valor | Inicialize no `initState` ou torne-a anulável |
| `No MaterialLocalizations found` | Widget Material fora de um `MaterialApp` | Envolva o app em `MaterialApp` |
| `Navigator operation requested with a context that does not include a Navigator` | `context` acima do `MaterialApp` | Use o `context` de um widget filho do `MaterialApp` |
| `Could not find the correct Provider<X>` | O widget não está abaixo do `Provider` | Mova o `Provider` para cima na árvore, ou use o `context` correto |
| `Looking up a deactivated widget's ancestor is unsafe` | Uso do `context` depois de um `await` | Verifique `context.mounted` |
| `type 'Null' is not a subtype of type 'String'` | Campo ausente no JSON | Use `json['campo'] ?? ''` ou torne o campo anulável |
| `Gradle task assembleDebug failed` | Problema de build Android | `flutter clean && flutter pub get`; confira a mensagem completa acima do erro |
