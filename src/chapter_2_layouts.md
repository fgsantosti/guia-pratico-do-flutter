# 2.1 - Layouts: Organizando Widgets na Tela

## 📋 Índice

1. [Introdução](#-introdução)
2. [A Regra de Ouro do Layout](#-a-regra-de-ouro-do-layout)
3. [Container, Padding e SizedBox](#-container-padding-e-sizedbox)
4. [Row e Column](#-row-e-column)
5. [Expanded e Flexible](#-expanded-e-flexible)
6. [Stack e Positioned](#-stack-e-positioned)
7. [Wrap](#-wrap)
8. [ListView e GridView](#-listview-e-gridview)
9. [Projeto Prático: Cartão de Perfil](#-projeto-prático-cartão-de-perfil)
10. [Erros Comuns de Layout](#-erros-comuns-de-layout)
11. [Recursos Adicionais](#-recursos-adicionais)

## 🚀 Introdução

No Flutter, **tudo é widget**, inclusive o layout. Não existe um arquivo separado de estilos: você posiciona os elementos **combinando widgets**, um dentro do outro.

Dominar meia dúzia de widgets de layout (`Row`, `Column`, `Stack`, `Expanded`, `Padding` e `SizedBox`) permite construir praticamente qualquer tela.

> 💡 Ative o **Flutter Inspector** (no VS Code: "Open DevTools" → aba Inspector) e use o botão "Show Guidelines" para ver as bordas e espaçamentos de cada widget. Isso ajuda muito a entender o layout.

## 📏 A Regra de Ouro do Layout

Todo o sistema de layout do Flutter segue uma frase:

> **Restrições descem. Tamanhos sobem. O pai define a posição.**

1. O widget **pai** diz ao filho: "você pode ter entre X e Y de largura e altura" (as *constraints*).
2. O **filho** escolhe o próprio tamanho dentro desses limites e avisa ao pai.
3. O **pai** decide onde posicionar o filho.

Por isso, um `Container(width: 100)` nem sempre fica com 100 de largura: se o pai obrigar o filho a ocupar a tela toda, o pai vence. Quando algo não fica do tamanho esperado, pergunte: **"que restrições o pai está passando?"**

## 📦 Container, Padding e SizedBox

### Container: a "caixa" com estilo

```dart
Container(
  width: 200,
  height: 100,
  margin: const EdgeInsets.all(16),        // Espaço por fora
  padding: const EdgeInsets.all(12),       // Espaço por dentro
  alignment: Alignment.center,             // Posição do filho
  decoration: BoxDecoration(
    color: Colors.blue.shade50,
    borderRadius: BorderRadius.circular(12),
    border: Border.all(color: Colors.blue),
    boxShadow: const [
      BoxShadow(color: Colors.black26, blurRadius: 8, offset: Offset(0, 4)),
    ],
  ),
  child: const Text('Sou um Container'),
)
```

> ⚠️ Não use `color` e `decoration` ao mesmo tempo: coloque a cor dentro do `BoxDecoration`.

### EdgeInsets: definindo espaçamentos

```dart
EdgeInsets.all(16)                               // Todos os lados
EdgeInsets.symmetric(horizontal: 16, vertical: 8)
EdgeInsets.only(left: 8, top: 4)
EdgeInsets.fromLTRB(8, 4, 8, 4)                  // Esquerda, cima, direita, baixo
```

### Padding e SizedBox: mais leves que o Container

Se você só precisa de espaçamento ou tamanho, prefira widgets específicos:

```dart
// Apenas espaço interno
const Padding(
  padding: EdgeInsets.all(16),
  child: Text('Com padding'),
)

// Espaço fixo entre widgets
const SizedBox(height: 16)

// Tamanho fixo para o filho
const SizedBox(width: 200, height: 48, child: ElevatedButton(onPressed: null, child: Text('OK')))

// Ocupar toda a largura disponível
SizedBox(width: double.infinity, child: ElevatedButton(onPressed: () {}, child: const Text('Entrar')))
```

## ↔ Row e Column

`Row` coloca os filhos **lado a lado** (horizontal) e `Column` coloca **um embaixo do outro** (vertical). Eles têm exatamente as mesmas propriedades.

```dart
Column(
  mainAxisAlignment: MainAxisAlignment.center,   // Eixo principal (vertical na Column)
  crossAxisAlignment: CrossAxisAlignment.start,  // Eixo cruzado (horizontal na Column)
  children: const [
    Text('Primeiro'),
    Text('Segundo'),
    Text('Terceiro'),
  ],
)
```

### Os dois eixos

| | Eixo principal (*main axis*) | Eixo cruzado (*cross axis*) |
|---|-----------------------------|-----------------------------|
| `Row` | Horizontal ↔️ | Vertical ↕️ |
| `Column` | Vertical ↕️ | Horizontal ↔️ |

### MainAxisAlignment

| Valor | Resultado (em uma Row) |
|-------|------------------------|
| `start` | `[A B C        ]` |
| `center` | `[    A B C    ]` |
| `end` | `[        A B C]` |
| `spaceBetween` | `[A     B     C]` |
| `spaceAround` | `[ A   B   C ]` (meio espaço nas pontas) |
| `spaceEvenly` | `[  A  B  C  ]` (espaços iguais) |

### CrossAxisAlignment

| Valor | Resultado |
|-------|-----------|
| `start` | Alinha no início do eixo cruzado |
| `center` | Centraliza (padrão) |
| `end` | Alinha no fim |
| `stretch` | Estica os filhos para ocupar todo o eixo cruzado |

### Espaçamento entre filhos

Desde o Flutter 3.27, `Row` e `Column` têm a propriedade `spacing`:

```dart
Row(
  spacing: 12, // 12 pixels entre cada filho
  children: const [Icon(Icons.star), Icon(Icons.star), Icon(Icons.star)],
)
```

Em versões anteriores, use `SizedBox(width: 12)` entre os filhos.

### Exemplo: barra de ações

```dart
Row(
  mainAxisAlignment: MainAxisAlignment.spaceEvenly,
  children: const [
    Column(children: [Icon(Icons.call), Text('Ligar')]),
    Column(children: [Icon(Icons.message), Text('Mensagem')]),
    Column(children: [Icon(Icons.share), Text('Compartilhar')]),
  ],
)
```

## 📐 Expanded e Flexible

Dentro de `Row` ou `Column`, o `Expanded` faz um filho **ocupar todo o espaço que sobrou**.

```dart
Row(
  children: [
    const Icon(Icons.person),
    const SizedBox(width: 8),
    // O texto ocupa todo o espaço entre o ícone e o botão
    const Expanded(
      child: Text('Um nome bem comprido que não cabe na linha', overflow: TextOverflow.ellipsis),
    ),
    IconButton(icon: const Icon(Icons.more_vert), onPressed: () {}),
  ],
)
```

### Dividindo o espaço com `flex`

```dart
Row(
  children: [
    Expanded(flex: 2, child: Container(height: 80, color: Colors.red)),    // 2/6
    Expanded(flex: 3, child: Container(height: 80, color: Colors.green)),  // 3/6
    Expanded(flex: 1, child: Container(height: 80, color: Colors.blue)),   // 1/6
  ],
)
```

### Expanded x Flexible

| | `Expanded` | `Flexible` |
|---|------------|------------|
| Ocupa todo o espaço disponível? | ✅ Sempre | ❌ Só o que precisa (até o limite) |
| Uso típico | Preencher espaço | Permitir que o filho encolha sem estourar |

```dart
Row(
  children: [
    // Ocupa só o tamanho do texto, mas pode encolher se faltar espaço
    Flexible(child: Text(nomeDoUsuario, overflow: TextOverflow.ellipsis)),
    const Icon(Icons.verified, color: Colors.blue),
  ],
)
```

### Spacer: empurrando widgets

```dart
Row(
  children: const [
    Text('Total'),
    Spacer(),         // Ocupa o espaço do meio
    Text('R\$ 99,90'),
  ],
)
```

## 🥞 Stack e Positioned

O `Stack` empilha widgets **um sobre o outro**, como camadas. O primeiro da lista fica no fundo.

```dart
Stack(
  children: [
    // Camada 1: imagem de fundo
    Container(width: 300, height: 200, color: Colors.indigo),

    // Camada 2: texto no canto inferior esquerdo
    const Positioned(
      left: 16,
      bottom: 16,
      child: Text('Promoção', style: TextStyle(color: Colors.white, fontSize: 24)),
    ),

    // Camada 3: selo no canto superior direito
    Positioned(
      top: 8,
      right: 8,
      child: Container(
        padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 4),
        decoration: BoxDecoration(
          color: Colors.red,
          borderRadius: BorderRadius.circular(12),
        ),
        child: const Text('-50%', style: TextStyle(color: Colors.white)),
      ),
    ),
  ],
)
```

### Exemplo clássico: ícone com contador (badge)

```dart
Stack(
  clipBehavior: Clip.none, // Permite que o contador "saia" da área do ícone
  children: [
    const Icon(Icons.shopping_cart, size: 32),
    Positioned(
      right: -6,
      top: -6,
      child: CircleAvatar(
        radius: 10,
        backgroundColor: Colors.red,
        child: Text('3', style: const TextStyle(fontSize: 12, color: Colors.white)),
      ),
    ),
  ],
)
```

> 💡 O Material 3 já tem o widget `Badge` pronto: `Badge(label: Text('3'), child: Icon(Icons.shopping_cart))`.

## 🔁 Wrap

Quando os itens podem não caber em uma linha, o `Wrap` quebra automaticamente para a próxima, como texto em um parágrafo. Ideal para *tags* e filtros:

```dart
Wrap(
  spacing: 8,     // Espaço horizontal entre itens
  runSpacing: 8,  // Espaço entre as linhas
  children: ['Flutter', 'Dart', 'Mobile', 'Android', 'iOS', 'Web', 'Desktop']
      .map((tag) => Chip(label: Text(tag)))
      .toList(),
)
```

## 📜 ListView e GridView

`Row` e `Column` **não rolam**. Para conteúdo maior que a tela, use widgets roláveis.

### SingleChildScrollView: rolar uma Column

```dart
SingleChildScrollView(
  padding: const EdgeInsets.all(16),
  child: Column(
    children: [
      // Formulário longo, texto de termos de uso etc.
    ],
  ),
)
```

### ListView.builder: listas longas

```dart
ListView.builder(
  itemCount: 100,
  itemBuilder: (context, index) => ListTile(
    leading: CircleAvatar(child: Text('${index + 1}')),
    title: Text('Item ${index + 1}'),
  ),
)
```

### GridView: grade

```dart
GridView.count(
  crossAxisCount: 3,        // 3 colunas
  mainAxisSpacing: 8,
  crossAxisSpacing: 8,
  padding: const EdgeInsets.all(8),
  children: List.generate(
    12,
    (i) => Container(color: Colors.teal[100 * ((i % 8) + 1)]),
  ),
)
```

## 👤 Projeto Prático: Cartão de Perfil

Vamos combinar tudo em uma tela completa:

```dart
import 'package:flutter/material.dart';

void main() => runApp(const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: PerfilPage(),
    ));

class PerfilPage extends StatelessWidget {
  const PerfilPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.grey.shade100,
      body: SingleChildScrollView(
        child: Column(
          children: [
            // ---------- Capa + foto sobreposta (Stack) ----------
            Stack(
              clipBehavior: Clip.none,
              alignment: Alignment.center,
              children: [
                Container(height: 180, color: Colors.indigo),
                const Positioned(
                  bottom: -50,
                  child: CircleAvatar(
                    radius: 54,
                    backgroundColor: Colors.white,
                    child: CircleAvatar(
                      radius: 50,
                      child: Icon(Icons.person, size: 60),
                    ),
                  ),
                ),
              ],
            ),
            const SizedBox(height: 60),

            // ---------- Nome e cargo (Column) ----------
            const Text(
              'Maria Silva',
              style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
            ),
            const Text('Desenvolvedora Flutter', style: TextStyle(color: Colors.grey)),
            const SizedBox(height: 16),

            // ---------- Estatísticas (Row + Expanded) ----------
            Padding(
              padding: const EdgeInsets.symmetric(horizontal: 16),
              child: Card(
                child: Padding(
                  padding: const EdgeInsets.symmetric(vertical: 16),
                  child: Row(
                    children: const [
                      _Estatistica(valor: '128', rotulo: 'Projetos'),
                      _Estatistica(valor: '2,4k', rotulo: 'Seguidores'),
                      _Estatistica(valor: '312', rotulo: 'Seguindo'),
                    ],
                  ),
                ),
              ),
            ),
            const SizedBox(height: 16),

            // ---------- Habilidades (Wrap) ----------
            Padding(
              padding: const EdgeInsets.symmetric(horizontal: 16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  const Text('Habilidades', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
                  const SizedBox(height: 8),
                  Wrap(
                    spacing: 8,
                    runSpacing: 8,
                    children: ['Flutter', 'Dart', 'Firebase', 'Provider', 'BLoC', 'Git']
                        .map((h) => Chip(label: Text(h)))
                        .toList(),
                  ),
                ],
              ),
            ),
            const SizedBox(height: 24),

            // ---------- Botões (Row + Expanded) ----------
            Padding(
              padding: const EdgeInsets.symmetric(horizontal: 16),
              child: Row(
                children: [
                  Expanded(
                    child: FilledButton.icon(
                      onPressed: () {},
                      icon: const Icon(Icons.person_add),
                      label: const Text('Seguir'),
                    ),
                  ),
                  const SizedBox(width: 12),
                  Expanded(
                    child: OutlinedButton.icon(
                      onPressed: () {},
                      icon: const Icon(Icons.message),
                      label: const Text('Mensagem'),
                    ),
                  ),
                ],
              ),
            ),
            const SizedBox(height: 24),
          ],
        ),
      ),
    );
  }
}

class _Estatistica extends StatelessWidget {
  const _Estatistica({required this.valor, required this.rotulo});

  final String valor;
  final String rotulo;

  @override
  Widget build(BuildContext context) {
    return Expanded(
      child: Column(
        children: [
          Text(valor, style: const TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
          Text(rotulo, style: const TextStyle(color: Colors.grey)),
        ],
      ),
    );
  }
}
```

### Conceitos aprendidos:
- `Stack` com `clipBehavior: Clip.none` permite a foto "vazar" da capa
- Um widget próprio (`_Estatistica`) evita repetir código
- `Expanded` divide a largura igualmente entre as estatísticas e entre os botões
- `SingleChildScrollView` garante a rolagem em telas pequenas

## 🐛 Erros Comuns de Layout

| Erro | Causa | Solução |
|------|-------|---------|
| Faixa amarela e preta: **"A RenderFlex overflowed by N pixels"** | O conteúdo da `Row`/`Column` é maior que o espaço | Use `Expanded`/`Flexible`, `TextOverflow.ellipsis` ou envolva em `SingleChildScrollView` |
| **"Vertical viewport was given unbounded height"** | `ListView` dentro de uma `Column` sem limite de altura | Envolva o `ListView` em `Expanded` |
| **"RenderBox was not laid out"** | Normalmente a mesma causa acima | Dê um tamanho ao widget rolável (`Expanded` ou `SizedBox`) |
| **"Incorrect use of ParentDataWidget"** | `Expanded` fora de `Row`/`Column`/`Flex`, ou `Positioned` fora de `Stack` | Coloque-os como filhos **diretos** do widget correto |
| O teclado cobre o campo de texto | A tela não rola | Use `SingleChildScrollView` no corpo do `Scaffold` |
| `Container(width: 100)` ocupa a tela toda | O pai impõe restrições "apertadas" | Envolva em `Center` ou `Align`, que liberam o filho para escolher o tamanho |

### Exemplo: corrigindo o erro de ListView dentro de Column

```dart
// ❌ Erro: o ListView não sabe qual altura ter
Column(
  children: [
    const Text('Contatos'),
    ListView.builder(itemCount: 50, itemBuilder: (_, i) => Text('Contato $i')),
  ],
)

// ✅ Correto: o Expanded dá ao ListView o espaço que sobrou
Column(
  children: [
    const Text('Contatos'),
    Expanded(
      child: ListView.builder(itemCount: 50, itemBuilder: (_, i) => Text('Contato $i')),
    ),
  ],
)
```

## 📖 Recursos Adicionais

- [Documentação oficial: Layouts no Flutter](https://docs.flutter.dev/ui/layout)
- [Documentação oficial: Entendendo as constraints](https://docs.flutter.dev/ui/layout/constraints)
- [Catálogo de widgets de layout](https://docs.flutter.dev/ui/widgets/layout)
- [Flutter Inspector](https://docs.flutter.dev/tools/devtools/inspector)
