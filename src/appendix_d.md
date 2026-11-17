# Apêndice D - Glossário

Termos usados ao longo do livro, em ordem alfabética. Entre parênteses, o capítulo onde o assunto é abordado.

## A

- **AAB (Android App Bundle)**: Formato de pacote exigido pela Google Play. A loja gera, a partir dele, APKs otimizados para cada aparelho. (Cap. 9)

- **AnimationController**: Objeto que controla uma animação explícita: duração, início, pausa e repetição. (Cap. 10)

- **AOT (Ahead-of-Time)**: Compilação feita antes da execução, usada no modo release. Gera código nativo rápido.

- **APK**: Arquivo de instalação de apps Android. (Cap. 9)

- **API (Application Programming Interface)**: Conjunto de endereços e regras para um programa conversar com outro, normalmente via HTTP. (Cap. 6)

- **Asset**: Arquivo incluído no app (imagens, fontes, JSON), declarado no `pubspec.yaml`.

- **async / await**: Palavras-chave para escrever código assíncrono como se fosse sequencial. `await` espera um `Future` terminar sem travar a tela. (Apêndice F)

## B

- **BLoC (Business Logic Component)**: Padrão de gerenciamento de estado que transforma **eventos** em **estados**. (Cap. 5)

- **build()**: Método que descreve como o widget deve aparecer. Pode ser chamado muitas vezes, por isso deve ser rápido.

- **BuildContext**: A "localização" de um widget na árvore. Usado para acessar tema, navegação, providers e o tamanho da tela.

## C

- **ChangeNotifier**: Classe que avisa seus ouvintes quando algo muda, chamando `notifyListeners()`. Base do Provider. (Cap. 4)

- **Constraints (restrições)**: Limites mínimos e máximos de largura e altura que um widget pai passa ao filho. (Seção 2.1)

- **const**: Valor conhecido em tempo de compilação. Widgets `const` são criados uma vez e não são reconstruídos. (Cap. 10)

- **Controller**: Objeto que controla um widget de fora, como `TextEditingController`, `ScrollController` e `AnimationController`. Deve ser liberado com `dispose()`.

- **CRUD**: As quatro operações básicas com dados: *Create*, *Read*, *Update* e *Delete*. (Cap. 8)

## D

- **Dart**: Linguagem de programação usada pelo Flutter, criada pelo Google. (Apêndice F)

- **Deep link**: Link que abre uma tela específica dentro do app. (Cap. 3)

- **DevTools**: Conjunto de ferramentas de depuração e performance do Flutter. (Apêndice B)

- **dispose()**: Método chamado quando um `State` é removido da tela. Usado para liberar controllers e cancelar assinaturas.

## E

- **Emulador**: Aparelho virtual que roda no computador. (Apêndice A)

- **Equatable**: Pacote que permite comparar objetos pelos valores dos campos, e não pela referência. (Cap. 5)

## F

- **Firebase**: Plataforma do Google com serviços prontos de back-end: login, banco de dados, notificações. (Seção 10.1)

- **Flavor**: Variante do app (ex.: desenvolvimento e produção) com configurações próprias. (Cap. 9)

- **Future**: Valor que estará disponível no futuro, como a resposta de uma API. (Apêndice F)

- **FutureBuilder**: Widget que reconstrói a tela conforme o estado de um `Future`: carregando, erro ou dados. (Cap. 6)

## G

- **GestureDetector**: Widget que detecta toques, arrastos e outros gestos. (Cap. 1)

- **GlobalKey**: Chave única no app inteiro que dá acesso ao estado de um widget, como o `FormState`. (Seção 2.2)

- **Golden test**: Teste que compara a aparência de um widget com uma imagem de referência. (Cap. 7)

## H

- **Hero**: Widget que anima um elemento entre duas telas durante a navegação. (Cap. 10)

- **Hot Reload**: Recarrega o código alterado em cerca de um segundo, **mantendo o estado** do app. (Apêndice B)

- **Hot Restart**: Reinicia o app com o novo código, **perdendo o estado**. (Apêndice B)

## I

- **i18n (internacionalização)**: Preparar o app para vários idiomas. "i18n" porque há 18 letras entre o "i" e o "n" de *internationalization*. (Cap. 10)

- **InheritedWidget**: Widget que disponibiliza dados para toda a sua subárvore. Base do Provider. (Cap. 4)

- **initState()**: Método chamado uma única vez quando um `StatefulWidget` é criado. Local para inicializações. (Cap. 1)

- **Injeção de dependência**: Técnica em que uma classe recebe suas dependências de fora, em vez de criá-las. Facilita testes e trocas. (Cap. 10)

- **Interceptor**: Código executado antes ou depois de toda requisição HTTP, como para adicionar um token. (Cap. 6)

- **Isolate**: Linha de execução separada do Dart, usada para tarefas pesadas sem travar a interface. (Cap. 10)

## J

- **JIT (Just-in-Time)**: Compilação durante a execução, usada no modo debug. Permite o Hot Reload.

- **JSON**: Formato de texto para trocar dados, baseado em chaves e valores. (Cap. 6)

## K

- **Key**: Identificador de um widget. Ajuda o Flutter a saber quais widgets mudaram e facilita os testes. (Cap. 7)

- **Keystore**: Arquivo com a chave usada para assinar o app Android. (Cap. 9)

## L

- **late**: Modificador que indica que a variável será inicializada depois, mas antes de ser usada. (Apêndice F)

- **Lint**: Regra de análise de código que aponta problemas e más práticas. (Apêndice B)

## M

- **Material Design**: Sistema de design do Google, seguido pelos widgets Material do Flutter. (Cap. 2)

- **Mock**: Objeto falso usado em testes para simular uma dependência real. (Cap. 7)

- **mounted**: Propriedade que indica se o `State` ainda está na tela. Verifique antes de usar `setState` após um `await`.

## N

- **Navigator**: Gerencia a pilha de telas do app (`push` empilha, `pop` desempilha). (Cap. 3)

- **Null safety**: Recurso do Dart que impede, em tempo de compilação, o uso acidental de valores nulos. (Apêndice F)

## P

- **Pacote (package)**: Biblioteca reutilizável, publicada no [pub.dev](https://pub.dev).

- **Platform channel**: Canal de comunicação entre o código Dart e o código nativo (Kotlin/Swift). (Cap. 10)

- **Plugin**: Pacote que contém código nativo para acessar recursos do aparelho, como câmera e GPS.

- **Provider**: Pacote de gerenciamento de estado baseado em `InheritedWidget`. (Cap. 4)

- **pubspec.yaml**: Arquivo de configuração do projeto: nome, versão, dependências e assets.

## R

- **Release**: Modo de build otimizado, usado para publicar o app. (Cap. 9)

- **Repository (repositório)**: Classe que esconde de onde vêm os dados (API, banco) e oferece operações simples para o resto do app. (Cap. 6 e 8)

- **Rota**: Uma tela do app, no contexto da navegação. (Cap. 3)

## S

- **Scaffold**: Estrutura básica de uma tela Material: `AppBar`, corpo, `FloatingActionButton`, menu lateral etc.

- **SDK (Software Development Kit)**: Conjunto de ferramentas para desenvolver para uma plataforma. (Apêndice A)

- **setState()**: Avisa ao Flutter que o estado mudou e que o widget precisa ser reconstruído. (Cap. 1)

- **Splash screen**: Tela exibida enquanto o app carrega. (Seção 9.1)

- **StatefulWidget**: Widget com estado que pode mudar ao longo do tempo. (Cap. 1)

- **StatelessWidget**: Widget sem estado interno; depende apenas dos parâmetros recebidos. (Cap. 1)

- **Stream**: Sequência de valores que chegam ao longo do tempo, como mensagens de um chat. (Apêndice F)

- **StreamBuilder**: Widget que reconstrói a tela a cada novo valor de um `Stream`. (Seção 10.1)

## T

- **Tema (ThemeData)**: Configuração central de cores, fontes e estilos do app. (Cap. 10)

- **Tween**: Define o valor inicial e final de uma animação. (Cap. 10)

## V

- **Validator**: Função que verifica o valor de um campo e devolve `null` (válido) ou uma mensagem de erro. (Seção 2.2)

- **vsync**: Sincroniza a animação com a taxa de atualização da tela, evitando desperdício. (Cap. 10)

## W

- **Widget**: A unidade básica da interface no Flutter. Tudo é widget: textos, botões, layouts e até o próprio app. (Cap. 2)

- **Widget tree (árvore de widgets)**: A estrutura hierárquica formada pelos widgets aninhados.
