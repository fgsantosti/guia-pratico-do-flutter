# Apêndice E - Aplicativos Exemplo

Este apêndice reúne os aplicativos completos construídos ao longo do livro, com sugestões de evolução para praticar.

> 💡 Cada exemplo pode ser colado em `lib/main.dart` de um projeto novo (`flutter create nome_do_app`), instalando antes os pacotes indicados.

## 📱 Apps do Livro

| App | Onde está | Conceitos praticados | Pacotes |
|-----|-----------|----------------------|---------|
| Olá Mundo interativo | [Capítulo 1](./chapter_1.md) | Stateless, Stateful, `setState`, gestos | – |
| Catálogo de widgets | [Capítulo 2](./chapter_2.md) | Material, Cupertino, widgets básicos | – |
| Cartão de perfil | [Seção 2.1](./chapter_2_layouts.md#-projeto-prático-cartão-de-perfil) | `Row`, `Column`, `Stack`, `Wrap`, `Expanded` | – |
| Tela de cadastro | [Seção 2.2](./chapter_2_forms.md#-projeto-prático-tela-de-cadastro) | `Form`, validação, máscaras, foco | `mask_text_input_formatter` |
| Navegação entre telas | [Capítulo 3](./chapter_3.md) | Navigator, rotas nomeadas, go_router | `go_router` |
| Envio de dados entre telas | [Seção 3.1](./chapter_3sub.md) | Argumentos, retorno de dados | – |
| App de tarefas (Provider) | [Capítulo 4](./chapter_4.md#-exemplo-prático-completo---app-de-tarefas) | `ChangeNotifier`, `Consumer`, `context.watch` | `provider` |
| Contador com BLoC | [Capítulo 5](./chapter_5.md) | Eventos, estados, `BlocBuilder` | `flutter_bloc`, `equatable` |
| Lista de posts | [Capítulo 6](./chapter_6.md) | HTTP, JSON, `FutureBuilder`, paginação | `http`, `dio` |
| Buscador de CEP | [Capítulo 6](./chapter_6.md#-projeto-prático-buscador-de-cep) | API real, tratamento de erros | `http` |
| Testes do contador e do BLoC | [Capítulo 7](./chapter_7.md) | Testes de unidade, widget e integração | `mocktail`, `bloc_test` |
| CRUD de usuários | [Capítulo 8](./chapter_8.md) | SQLite, repositório, Provider | `sqflite`, `provider` |
| Configurações persistentes | [Seção 8.1](./chapter_8_persistencia.md) | Preferências, tema salvo, onboarding | `shared_preferences` |
| Splash animada | [Seção 9.1](./chapter_9_splash.md) | Splash nativa e animada | `flutter_native_splash` |
| Tema claro/escuro | [Capítulo 10](./chapter_10.md#-temas-e-modo-escuro) | `ThemeData`, Material 3, Provider | `provider` |
| Tarefas na nuvem | [Seção 10.1](./chapter_10_firebase.md) | Login, Firestore em tempo real, regras | `firebase_auth`, `cloud_firestore` |

## 🚀 Projetos para Praticar

Depois de terminar o livro, tente construir estes apps do zero. Eles combinam vários capítulos.

### 1. 📝 Lista de Compras (iniciante)

- Adicionar, marcar e remover itens
- Salvar a lista no aparelho
- Agrupar por categoria

**Capítulos:** 1, 2.1, 2.2, 4 e 8.1

### 2. 💰 Controle Financeiro (intermediário)

- Cadastro de receitas e despesas com valor, data e categoria
- Saldo do mês e gráfico de gastos por categoria
- Filtro por período

**Capítulos:** 2.2, 4 ou 5, 8 e 10 (formatação de moeda com `intl`) · **Pacotes extras:** `fl_chart`

### 3. 🎬 Catálogo de Filmes (intermediário)

- Lista de filmes populares com paginação
- Tela de detalhes com animação `Hero`
- Busca e lista de favoritos salva no aparelho

**Capítulos:** 3, 5, 6, 8.1 e 10 · **API:** [The Movie Database](https://developer.themoviedb.org)

### 4. 🌦 App de Clima (intermediário)

- Clima atual pela localização do GPS
- Previsão para os próximos dias
- Busca por cidade e tema que muda conforme o clima

**Capítulos:** 6 e 10 (recursos do aparelho) · **API:** [OpenWeather](https://openweathermap.org/api) · **Pacotes extras:** `geolocator`

### 5. 💬 Chat em Tempo Real (avançado)

- Login com e-mail
- Salas de conversa com mensagens em tempo real
- Envio de fotos e notificações de novas mensagens

**Capítulos:** 2.2, 5 e 10.1 · **Pacotes extras:** `firebase_storage`, `firebase_messaging`, `image_picker`

### 6. 🛒 Loja Virtual (avançado)

- Catálogo com categorias e busca
- Carrinho de compras com Provider ou BLoC
- Login, histórico de pedidos e tema claro/escuro
- Layout responsivo para celular, tablet e web
- Testes do carrinho e publicação na Play Store

**Capítulos:** todos! · **API:** [Fake Store API](https://fakestoreapi.com)

## 📂 Compartilhe seu Projeto

Terminou um projeto? Publique no GitHub com um bom `README.md` contendo:

- Capturas de tela ou um GIF do app funcionando
- Lista de funcionalidades
- Tecnologias e pacotes usados
- Como rodar o projeto (`flutter pub get` e `flutter run`)

Um portfólio com 2 ou 3 projetos bem feitos vale mais que dezenas de exercícios inacabados. 🚀
