# Apêndice C - Recursos Úteis

## 📋 Índice

1. [Documentação Oficial](#-documentação-oficial)
2. [Pacotes Recomendados](#-pacotes-recomendados)
3. [Ferramentas](#-ferramentas)
4. [APIs Gratuitas para Praticar](#-apis-gratuitas-para-praticar)
5. [Comunidades](#-comunidades)
6. [Onde Continuar Aprendendo](#-onde-continuar-aprendendo)

## 📚 Documentação Oficial

| Recurso | Endereço | Para quê |
|---------|----------|----------|
| Documentação do Flutter | [docs.flutter.dev](https://docs.flutter.dev) | Referência principal |
| Cookbook | [docs.flutter.dev/cookbook](https://docs.flutter.dev/cookbook) | Receitas curtas para tarefas comuns |
| Catálogo de widgets | [docs.flutter.dev/ui/widgets](https://docs.flutter.dev/ui/widgets) | Todos os widgets por categoria |
| API do Flutter | [api.flutter.dev](https://api.flutter.dev) | Detalhes de cada classe e método |
| Documentação do Dart | [dart.dev/guides](https://dart.dev/guides) | A linguagem Dart |
| DartPad | [dartpad.dev](https://dartpad.dev) | Rodar Dart e Flutter no navegador, sem instalar nada |
| Canal oficial no YouTube | [youtube.com/@flutterdev](https://www.youtube.com/@flutterdev) | Série "Widget of the Week" e novidades |
| Material Design 3 | [m3.material.io](https://m3.material.io) | Guia de design dos componentes Material |

## 📦 Pacotes Recomendados

Todos disponíveis no [pub.dev](https://pub.dev). Antes de escolher um pacote, confira a pontuação (**likes**, **pub points** e **popularidade**) e a data da última atualização.

### Gerenciamento de estado

| Pacote | Descrição |
|--------|-----------|
| [`provider`](https://pub.dev/packages/provider) | Simples e oficialmente recomendado (Capítulo 4) |
| [`flutter_bloc`](https://pub.dev/packages/flutter_bloc) | Padrão BLoC (Capítulo 5) |
| [`riverpod`](https://pub.dev/packages/flutter_riverpod) | Evolução do Provider, sem depender do `context` |
| [`get_it`](https://pub.dev/packages/get_it) | Injeção de dependência (Capítulo 10) |

### Rede e dados

| Pacote | Descrição |
|--------|-----------|
| [`http`](https://pub.dev/packages/http) | Requisições HTTP simples (Capítulo 6) |
| [`dio`](https://pub.dev/packages/dio) | Cliente HTTP completo com interceptors (Capítulo 6) |
| [`json_serializable`](https://pub.dev/packages/json_serializable) | Gera `fromJson`/`toJson` |
| [`freezed`](https://pub.dev/packages/freezed) | Classes imutáveis com `copyWith` e igualdade |
| [`connectivity_plus`](https://pub.dev/packages/connectivity_plus) | Verifica se há conexão |

### Armazenamento

| Pacote | Descrição |
|--------|-----------|
| [`shared_preferences`](https://pub.dev/packages/shared_preferences) | Chave-valor simples (seção 8.1) |
| [`flutter_secure_storage`](https://pub.dev/packages/flutter_secure_storage) | Dados sensíveis criptografados (seção 8.1) |
| [`sqflite`](https://pub.dev/packages/sqflite) | Banco SQLite (Capítulo 8) |
| [`drift`](https://pub.dev/packages/drift) | SQLite com tipagem e consultas reativas |
| [`path_provider`](https://pub.dev/packages/path_provider) | Pastas do aparelho (seção 8.1) |

### Navegação

| Pacote | Descrição |
|--------|-----------|
| [`go_router`](https://pub.dev/packages/go_router) | Rotas declarativas, deep links e Web (Capítulo 3) |

### Interface

| Pacote | Descrição |
|--------|-----------|
| [`google_fonts`](https://pub.dev/packages/google_fonts) | Centenas de fontes gratuitas |
| [`cached_network_image`](https://pub.dev/packages/cached_network_image) | Imagens da internet com cache |
| [`lottie`](https://pub.dev/packages/lottie) | Animações em JSON (Capítulo 10) |
| [`flutter_svg`](https://pub.dev/packages/flutter_svg) | Exibir imagens SVG |
| [`shimmer`](https://pub.dev/packages/shimmer) | Efeito de "carregando" em esqueleto |
| [`fl_chart`](https://pub.dev/packages/fl_chart) | Gráficos |
| [`mask_text_input_formatter`](https://pub.dev/packages/mask_text_input_formatter) | Máscaras de CPF, telefone, CEP (seção 2.2) |

### Recursos do aparelho

| Pacote | Descrição |
|--------|-----------|
| [`image_picker`](https://pub.dev/packages/image_picker) | Câmera e galeria |
| [`geolocator`](https://pub.dev/packages/geolocator) | Localização GPS |
| [`permission_handler`](https://pub.dev/packages/permission_handler) | Solicitar e verificar permissões |
| [`url_launcher`](https://pub.dev/packages/url_launcher) | Abrir links, telefone, e-mail, WhatsApp |
| [`share_plus`](https://pub.dev/packages/share_plus) | Compartilhar conteúdo com outros apps |
| [`flutter_local_notifications`](https://pub.dev/packages/flutter_local_notifications) | Notificações locais e agendadas |

### Utilidades

| Pacote | Descrição |
|--------|-----------|
| [`intl`](https://pub.dev/packages/intl) | Datas, moedas e números formatados |
| [`equatable`](https://pub.dev/packages/equatable) | Comparação de objetos por valor (Capítulo 5) |
| [`uuid`](https://pub.dev/packages/uuid) | Gerar identificadores únicos |

### Publicação

| Pacote | Descrição |
|--------|-----------|
| [`flutter_launcher_icons`](https://pub.dev/packages/flutter_launcher_icons) | Ícones do app (Capítulo 9) |
| [`flutter_native_splash`](https://pub.dev/packages/flutter_native_splash) | Splash screen (seção 9.1) |

## 🛠 Ferramentas

| Ferramenta | Para quê |
|------------|----------|
| [Flutter DevTools](https://docs.flutter.dev/tools/devtools) | Depuração, performance e inspeção de widgets |
| [Figma](https://www.figma.com) | Design e protótipos de telas |
| [Material Theme Builder](https://material-foundation.github.io/material-theme-builder/) | Gerar paletas de cores do Material 3 |
| [quicktype](https://app.quicktype.io) | Gerar classes Dart a partir de um JSON |
| [Postman](https://www.postman.com) / [Insomnia](https://insomnia.rest) | Testar APIs antes de codificar |
| [Firebase Console](https://console.firebase.google.com) | Back-end pronto (seção 10.1) |
| [Codemagic](https://codemagic.io) | Build e publicação automatizados na nuvem |
| [LottieFiles](https://lottiefiles.com) | Animações gratuitas |
| [Google Fonts](https://fonts.google.com) | Escolher fontes |
| [Material Symbols](https://fonts.google.com/icons) | Buscar ícones do Material |

## 🌐 APIs Gratuitas para Praticar

| API | Conteúdo | Precisa de chave? |
|-----|----------|-------------------|
| [JSONPlaceholder](https://jsonplaceholder.typicode.com) | Posts, usuários e comentários falsos | ❌ |
| [ViaCEP](https://viacep.com.br) | Endereços pelo CEP | ❌ |
| [BrasilAPI](https://brasilapi.com.br) | CEP, CNPJ, bancos, feriados e mais | ❌ |
| [PokéAPI](https://pokeapi.co) | Dados de Pokémon | ❌ |
| [The Movie Database](https://developer.themoviedb.org) | Filmes e séries | ✅ (gratuita) |
| [OpenWeather](https://openweathermap.org/api) | Previsão do tempo | ✅ (gratuita) |
| [IBGE](https://servicodados.ibge.gov.br/api/docs) | Estados, municípios e dados do Brasil | ❌ |
| [Rick and Morty API](https://rickandmortyapi.com) | Personagens da série | ❌ |
| [Fake Store API](https://fakestoreapi.com) | Produtos para simular uma loja | ❌ |

## 👥 Comunidades

| Comunidade | Onde |
|------------|------|
| Flutterando | Maior comunidade brasileira, com canal no YouTube e Discord |
| Flutter Brasil | Publicações no Medium e grupos |
| Stack Overflow | Perguntas com a tag [`flutter`](https://stackoverflow.com/questions/tagged/flutter) |
| Reddit | [r/FlutterDev](https://www.reddit.com/r/FlutterDev) |
| Discord oficial | Link em [flutter.dev/community](https://flutter.dev/community) |
| Meetups e GDGs | Procure o Google Developer Group (GDG) da sua cidade |

### Como pedir ajuda (e conseguir resposta)

1. Descreva **o que esperava** e **o que aconteceu**.
2. Cole a **mensagem de erro completa** (como texto, não como foto da tela).
3. Mostre o **menor trecho de código** que reproduz o problema.
4. Informe a saída de `flutter doctor` e a versão dos pacotes envolvidos.

## 🎓 Onde Continuar Aprendendo

- **Codelabs oficiais:** [docs.flutter.dev/codelabs](https://docs.flutter.dev/codelabs), tutoriais guiados com projetos completos.
- **Widget of the Week:** vídeos de 1 minuto sobre um widget cada.
- **Código aberto:** estude apps de exemplo no repositório [flutter/samples](https://github.com/flutter/samples).
- **Pratique:** recrie telas de apps que você usa no dia a dia. É o melhor exercício de layout.
- **Contribua:** abra issues e sugestões neste livro no GitHub!
