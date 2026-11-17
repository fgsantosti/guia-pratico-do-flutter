# 📘 Guia Prático do Flutter

Guia em português para aprender Flutter, do primeiro "Olá Mundo" até a publicação nas lojas, com exemplos de código comentados.

**📖 Leia online:** <https://fgsantosti.github.io/guia-pratico-do-flutter/>

## 📚 Sumário

- [Introdução](src/introducao.md)

### Fundamentos
1. [Introdução ao Flutter](src/chapter_1.md)
2. [Catálogo de Widgets](src/chapter_2.md)
   - 2.1 [Layouts](src/chapter_2_layouts.md)
   - 2.2 [Formulários e Validação](src/chapter_2_forms.md)
3. [Navegação e Rotas](src/chapter_3.md)
   - 3.1 [Enviando Dados para uma Nova Tela](src/chapter_3sub.md)

### Gerenciamento de Estado
4. [Provider](src/chapter_4.md)
5. [BLoC](src/chapter_5.md)

### Dados e Qualidade
6. [Consumo de APIs](src/chapter_6.md)
7. [Testes em Flutter](src/chapter_7.md)
8. [Aplicativo CRUD com SQLite](src/chapter_8.md)
   - 8.1 [Persistência Local Simples](src/chapter_8_persistencia.md)

### Indo para Produção
9. [Publicação de Apps](src/chapter_9.md)
   - 9.1 [Splash Screen](src/chapter_9_splash.md)
10. [Recursos Avançados](src/chapter_10.md)
    - 10.1 [Firebase](src/chapter_10_firebase.md)

### Apêndices
- A. [Configuração do Ambiente](src/appendix_a.md)
- B. [Dicas e Truques](src/appendix_b.md)
- C. [Recursos Úteis](src/appendix_c.md)
- D. [Glossário](src/appendix_d.md)
- E. [Aplicativos Exemplo](src/appendix_e.md)
- F. [Dart Essencial](src/appendix_f.md)

## 🛠 Gerando o livro localmente

O livro usa o [mdBook](https://rust-lang.github.io/mdBook/) (versão 0.4 ou superior).

```bash
# Instalar (requer Rust) ou baixe o binário em github.com/rust-lang/mdBook/releases
cargo install mdbook

# Servir com recarregamento automático em http://localhost:3000
mdbook serve --open

# Gerar a versão estática na pasta book/
mdbook build
```

## 🚀 Publicação

A cada `push` na branch `main`, o GitHub Actions gera o livro e publica no GitHub Pages (veja [.github/workflows/deploy.yml](.github/workflows/deploy.yml)).
