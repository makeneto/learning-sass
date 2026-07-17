# Sass — Guia Completo (do 0 ao expert)

> Baseado em **Dart Sass 1.101**, a implementação oficial e de referência do Sass (a única mantida ativamente; Ruby Sass e LibSass estão descontinuados).

---

## Índice

1. [O que é o Sass](#1-o-que-é-o-sass)
2. [Instalação e compilação](#2-instalação-e-compilação)
3. [Sintaxe: SCSS vs Sass indentado](#3-sintaxe-scss-vs-sass-indentado)
4. [Variáveis](#4-variáveis)
5. [Nesting (aninhamento)](#5-nesting-aninhamento)
6. [Partials e o sistema de módulos (`@use` / `@forward`)](#6-partials-e-o-sistema-de-módulos-use--forward)
7. [Mixins (`@mixin` / `@include`)](#7-mixins-mixin--include)
8. [Funções (`@function`)](#8-funções-function)
9. [Herança (`@extend`)](#9-herança-extend)
10. [Operadores](#10-operadores)
11. [Estruturas de controlo (`@if`, `@each`, `@for`, `@while`)](#11-estruturas-de-controlo-if-each-for-while)
12. [Maps](#12-maps)
13. [Módulos nativos (`sass:*`)](#13-módulos-nativos-sass)
14. [Placeholders (`%`)](#14-placeholders)
15. [Interpolação](#15-interpolação)
16. [Arquitetura de projeto (padrão 7-1)](#16-arquitetura-de-projeto-padrão-7-1)
17. [Sass + BEM](#17-sass--bem)
18. [Novidades recentes do Dart Sass](#18-novidades-recentes-do-dart-sass)
19. [Boas práticas e erros comuns](#19-boas-práticas-e-erros-comuns)

---

## 1. O que é o Sass

Sass (*Syntactically Awesome Style Sheets*) é um pré-processador CSS: escreve-se código Sass/SCSS, que depois é **compilado** para CSS puro que o browser entende.

| Conceito       | Descrição                                                                 |
| -------------- | -------------------------------------------------------------------------- |
| Pré-processador | Ferramenta que transforma uma linguagem (Sass) noutra (CSS)               |
| Dart Sass      | Implementação oficial, escrita em Dart, distribuída via `npm`             |
| `.scss`        | Sintaxe com chavetas e ponto-e-vírgula, 100% compatível com CSS            |
| `.sass`        | Sintaxe indentada, sem chavetas nem ponto-e-vírgula                        |

O Sass não muda o CSS em runtime — tudo é resolvido **em tempo de compilação**. Isto significa zero custo de performance no browser.

> 💡 **Dica**: Se já sabes CSS, já sabes 90% do SCSS. O SCSS é um sobreconjunto de CSS — qualquer ficheiro `.css` válido é também um `.scss` válido.

---

## 2. Instalação e compilação

```bash
# Instalação global (executável standalone, mais rápido)
npm install -g sass

# Instalação local num projeto (recomendado)
pnpm add -D sass
```

Compilar um ficheiro:

```bash
sass input.scss output.css
```

Modo *watch* (recompila a cada alteração):

```bash
sass --watch input.scss:output.css
```

Compilar uma pasta inteira:

```bash
sass --watch scss:css
```

| Flag              | Função                                                        |
| ----------------- | -------------------------------------------------------------- |
| `--style=compressed` | Gera CSS minificado                                          |
| `--source-map`    | Gera source maps (ativo por defeito)                          |
| `--no-source-map` | Desativa source maps                                           |
| `--watch`         | Recompila automaticamente ao gravar                            |

> ⚠️ **Atenção**: Em produção, o mais comum é não chamar `sass` diretamente, mas sim através do bundler (Vite, Webpack, Next.js) que já integra o Dart Sass como dependência de dev.

---

## 3. Sintaxe: SCSS vs Sass indentado

**SCSS** (recomendado, e o que se usa em 95% dos projetos reais):

```scss
.card {
  padding: 16px;
  &:hover {
    opacity: 0.8;
  }
}
```

**Sass indentado** (sem chavetas, sem `;`, a indentação define o bloco):

```sass
.card
  padding: 16px
  &:hover
    opacity: 0.8
```

> **Regra de ouro**: usa sempre SCSS. É a sintaxe oficialmente recomendada pela equipa do Sass para projetos novos, é a que tem melhor suporte de ferramentas (linters, formatters, IDEs) e é a que aparece em toda a documentação moderna. Este guia usa SCSS do início ao fim.

---

## 4. Variáveis

```scss
$cor-primaria: #1a1a1a;
$espacamento-base: 8px;
$fonte-titulo: "Inter", sans-serif;

.botao {
  background: $cor-primaria;
  padding: $espacamento-base * 2;
  font-family: $fonte-titulo;
}
```

### Âmbito (scope)

Variáveis declaradas dentro de um bloco `{ }` só existem dentro dele, a não ser que uses `!global`:

```scss
$tema: "claro";

.dark-mode {
  $tema: "escuro" !global; // agora altera a variável global
}
```

### `!default`

Permite definir um valor por defeito que só é usado se a variável ainda não tiver sido definida — essencial para bibliotecas e ficheiros configuráveis:

```scss
$raio-borda: 4px !default;
```

Se quem importa o ficheiro já tiver definido `$raio-borda` antes, esse valor prevalece.

> 💡 **Dica**: `!default` é a base de toda a configuração de bibliotecas Sass (Bootstrap, por exemplo, usa centenas de variáveis `!default`).

---

## 5. Nesting (aninhamento)

```scss
.navbar {
  display: flex;

  .nav-item {
    padding: 8px;

    &.active {
      font-weight: bold;
    }

    &:hover {
      color: red;
    }
  }
}
```

Compila para:

```css
.navbar { display: flex; }
.navbar .nav-item { padding: 8px; }
.navbar .nav-item.active { font-weight: bold; }
.navbar .nav-item:hover { color: red; }
```

O `&` representa o **seletor pai**. É usado para:

| Uso                  | Exemplo             | Resultado             |
| --------------------- | -------------------- | ----------------------- |
| Pseudo-classes        | `&:hover`            | `.card:hover`           |
| Modificadores BEM     | `&--active`          | `.card--active`         |
| Elemento filho BEM    | `&__title`           | `.card__title`          |
| Seletor composto      | `.dark &`            | `.dark .card`           |

### Nesting de propriedades

Propriedades com prefixo comum também podem ser aninhadas:

```scss
.text {
  font: {
    family: sans-serif;
    size: 14px;
    weight: bold;
  }
}
```

> ⚠️ **Atenção**: nesting excessivo (mais de 3 níveis) gera seletores CSS muito específicos e difíceis de sobrepor. Mantém o aninhamento raso.

---

## 6. Partials e o sistema de módulos (`@use` / `@forward`)

### Partials

Um ficheiro que começa por `_` (ex.: `_variaveis.scss`) é um **partial** — o Sass não o compila isoladamente, só quando é importado noutro ficheiro.

### `@use` (substitui o `@import`)

```scss
// _variaveis.scss
$cor-primaria: #1a1a1a;
$espacamento: 8px;
```

```scss
// styles.scss
@use "variaveis";

.card {
  color: variaveis.$cor-primaria;
  padding: variaveis.$espacamento;
}
```

Regras principais do `@use`:

| Regra                                          | Explicação                                                         |
| ------------------------------------------------ | --------------------------------------------------------------------- |
| Namespace automático                             | O nome do ficheiro (`variaveis`) torna-se o prefixo de acesso        |
| Só carrega o ficheiro **uma vez**, mesmo que seja usado em vários sítios | Evita CSS duplicado                              |
| Namespace personalizado                          | `@use "variaveis" as v;` → `v.$cor-primaria`                        |
| Sem namespace                                    | `@use "variaveis" as *;` (acesso direto, usar com moderação)         |
| Ordem importa                                    | O que é usado primeiro é compilado primeiro                          |

### `@forward` — reexportar módulos

Usado tipicamente num `_index.scss` central que agrega vários partials:

```scss
// abstracts/_index.scss
@forward "variaveis";
@forward "mixins";
@forward "funcoes";
```

```scss
// styles.scss
@use "abstracts" as *;

.card {
  color: $cor-primaria;
}
```

`@forward` também permite prefixar ou esconder membros:

```scss
@forward "botoes" as btn-*;        // prefixa tudo com btn-
@forward "botoes" show $cor-botao; // só expõe esta variável
@forward "botoes" hide $interno;   // esconde uma variável interna
```

E permite configurar valores `!default` de quem consome o módulo:

```scss
@forward "botoes" with (
  $raio-borda: 8px !default
);
```

> **Regra de ouro**: `@import` está **descontinuado** desde 2023 e será removido nas próximas versões principais do Dart Sass. Todo o código novo deve usar `@use` e `@forward`. Se herdares um projeto com `@import`, migra-o assim que possível.

---

## 7. Mixins (`@mixin` / `@include`)

Um mixin é um bloco de estilos reutilizável, com ou sem parâmetros.

```scss
@mixin flex-center($direcao: row) {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: $direcao;
}

.header {
  @include flex-center;
}

.sidebar {
  @include flex-center(column);
}
```

### Parâmetros nomeados e valores por defeito

```scss
@mixin botao($cor: black, $tamanho: 16px) {
  background: $cor;
  font-size: $tamanho;
}

.btn-danger {
  @include botao($cor: red);
}
```

### Argumentos variádicos (`...`)

```scss
@mixin sombra($sombras...) {
  box-shadow: $sombras;
}

.card {
  @include sombra(0 1px 2px black, 0 2px 4px grey);
}
```

### `@content` — injetar bloco de estilos no mixin

```scss
@mixin responsive($ponto-quebra) {
  @media (min-width: $ponto-quebra) {
    @content;
  }
}

.container {
  width: 100%;

  @include responsive(768px) {
    width: 750px;
  }
}
```

> 💡 **Dica**: `@content` é o mecanismo por trás de praticamente todos os sistemas de *breakpoints* em Sass (incluindo os usados por frameworks como o Bootstrap).

---

## 8. Funções (`@function`)

Ao contrário de um mixin (que gera estilos), uma função **devolve um valor**.

```scss
@function rem($px, $base: 16px) {
  @return math.div($px, $base) * 1rem;
}

.titulo {
  font-size: rem(24px); // 1.5rem
}
```

```scss
@function contraste($cor) {
  @if lightness($cor) > 50% {
    @return black;
  } @else {
    @return white;
  }
}

.badge {
  background: $cor-primaria;
  color: contraste($cor-primaria);
}
```

> ⚠️ **Atenção**: desde o Dart Sass 1.33, o operador `/` para divisão está **descontinuado**. Usa sempre `math.div($a, $b)` do módulo `sass:math` em vez de `$a / $b`.

---

## 9. Herança (`@extend`)

Permite que um seletor herde as regras de outro.

```scss
%botao-base {
  padding: 8px 16px;
  border-radius: 4px;
  border: none;
}

.btn-primario {
  @extend %botao-base;
  background: blue;
}

.btn-secundario {
  @extend %botao-base;
  background: grey;
}
```

Compila para:

```css
.btn-primario, .btn-secundario {
  padding: 8px 16px;
  border-radius: 4px;
  border: none;
}
.btn-primario { background: blue; }
.btn-secundario { background: grey; }
```

| `@extend` vs `@mixin`        | Quando usar                                                       |
| ------------------------------ | -------------------------------------------------------------------- |
| `@extend %placeholder`         | Quando queres **agrupar** seletores no CSS final (menos repetição)  |
| `@mixin` + `@include`          | Quando precisas de **parâmetros** ou de repetir o bloco por seletor |

> **Regra de ouro**: prefere sempre `@extend` com placeholders (`%nome`), nunca com classes reais (`.classe`). Estender uma classe real cria acoplamento inesperado entre componentes não relacionados.

---

## 10. Operadores

| Categoria     | Operadores                              | Exemplo                          |
| --------------- | ------------------------------------------ | ----------------------------------- |
| Aritméticos    | `+` `-` `*` `%`, e `math.div()` para divisão | `$a + $b`, `math.div($a, $b)`      |
| Comparação      | `==` `!=` `<` `>` `<=` `>=`               | `$a > $b`                           |
| Lógicos         | `and` `or` `not`                          | `$a and $b`                         |
| Concatenação    | interpolação `#{}`                        | `"btn-#{$tamanho}"`                 |

```scss
$largura-total: 1200px;
$colunas: 12;
$largura-coluna: math.div($largura-total, $colunas);

.col-4 {
  width: $largura-coluna * 4;
}
```

Strings também podem ser concatenadas com `+`:

```scss
$prefixo: "icon";
.#{$prefixo}-user { content: "\e001"; }
```

---

## 11. Estruturas de controlo (`@if`, `@each`, `@for`, `@while`)

### `@if` / `@else if` / `@else`

```scss
@mixin texto($peso) {
  @if $peso == "bold" {
    font-weight: 700;
  } @else if $peso == "medium" {
    font-weight: 500;
  } @else {
    font-weight: 400;
  }
}
```

### `@each` — iterar listas ou maps

```scss
$cores: (primaria: blue, sucesso: green, erro: red);

@each $nome, $valor in $cores {
  .texto-#{$nome} {
    color: $valor;
  }
}
```

```scss
$tamanhos: small, medium, large;

@each $tamanho in $tamanhos {
  .btn-#{$tamanho} {
    padding: 4px * index($tamanhos, $tamanho);
  }
}
```

### `@for` — repetir um número fixo de vezes

```scss
@for $i from 1 through 12 {
  .col-#{$i} {
    width: math.div(100%, 12) * $i;
  }
}
```

`through` inclui o último valor; `to` exclui-o.

### `@while` — repetir enquanto a condição for verdadeira

```scss
$i: 1;
@while $i <= 5 {
  .item-#{$i} { order: $i; }
  $i: $i + 1;
}
```

> 💡 **Dica**: na prática, `@each` e `@for` cobrem 95% dos casos. `@while` raramente é necessário em CSS — usa-o só quando o incremento não é linear.

---

## 12. Maps

Um map é uma estrutura chave-valor, essencial para *design tokens*, temas e breakpoints.

```scss
$breakpoints: (
  "mobile": 375px,
  "tablet": 768px,
  "desktop": 1280px,
);

@mixin acima-de($chave) {
  @media (min-width: map.get($breakpoints, $chave)) {
    @content;
  }
}

.container {
  @include acima-de("tablet") {
    max-width: 720px;
  }
}
```

Funções mais usadas do módulo `sass:map`:

| Função                       | Função                                              |
| ------------------------------ | ------------------------------------------------------ |
| `map.get($map, $chave)`        | Obtém o valor de uma chave                             |
| `map.has-key($map, $chave)`    | Verifica se a chave existe                             |
| `map.keys($map)`               | Lista todas as chaves                                  |
| `map.values($map)`             | Lista todos os valores                                 |
| `map.merge($map1, $map2)`      | Junta dois maps (útil para configurar temas)           |
| `map.set($map, $chave, $valor)` | Devolve um novo map com a chave atualizada             |

---

## 13. Módulos nativos (`sass:*`)

Desde o novo sistema de módulos, as funções globais antigas (`lighten()`, `darken()`, `percentage()`, etc.) foram organizadas em módulos nativos que precisam de `@use`.

```scss
@use "sass:math";
@use "sass:color";
@use "sass:string";
@use "sass:list";
@use "sass:map";
@use "sass:meta";
```

| Módulo        | Exemplos de funções                                                    |
| --------------- | -------------------------------------------------------------------------- |
| `sass:math`    | `math.div()`, `math.round()`, `math.min()`, `math.max()`, `math.abs()`   |
| `sass:color`   | `color.adjust()`, `color.scale()`, `color.mix()`, `color.channel()`      |
| `sass:string`  | `string.length()`, `string.slice()`, `string.to-upper-case()`             |
| `sass:list`    | `list.length()`, `list.nth()`, `list.append()`, `list.index()`            |
| `sass:map`     | ver secção anterior                                                       |
| `sass:meta`    | `meta.type-of()`, `meta.call()`, `meta.feature-exists()`                  |
| `sass:selector`| `selector.nest()`, `selector.is-superselector()`                          |

```scss
@use "sass:color";

.btn:hover {
  background: color.adjust($cor-primaria, $lightness: -10%);
}
```

> ⚠️ **Atenção**: as funções globais de cor (`lighten()`, `darken()`, `saturate()`) estão **descontinuadas** desde o Dart Sass 1.79 a favor de `color.adjust()` e `color.scale()`, que lidam corretamente com espaços de cor modernos (`oklch`, `lab`, etc.). Evita usá-las em código novo.

---

## 14. Placeholders

Já vistos na secção de `@extend`, os placeholders (`%nome`) são seletores que **nunca são emitidos sozinhos** para o CSS final — só existem para serem estendidos.

```scss
%visualmente-escondido {
  position: absolute;
  width: 1px;
  height: 1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
}

.sr-only {
  @extend %visualmente-escondido;
}
```

Se `%visualmente-escondido` nunca for estendido em lado nenhum, **não gera qualquer CSS** — o que os torna ideais para bibliotecas de utilitários internos.

---

## 15. Interpolação

A sintaxe `#{}` insere o valor de uma expressão Sass dentro de contexto CSS (seletores, nomes de propriedades, strings, valores).

```scss
$propriedade: margin;
$lado: top;

.card {
  #{$propriedade}-#{$lado}: 16px;
}
```

```scss
@each $tema in "claro", "escuro" {
  .tema-#{$tema} {
    // ...
  }
}
```

> 💡 **Dica**: só precisas de interpolação quando misturas Sass com texto literal do CSS (nomes de seletores, propriedades dinâmicas). Em atribuições simples de variável (`color: $cor;`) não é necessária.

---

## 16. Arquitetura de projeto (padrão 7-1)

O padrão 7-1 organiza os ficheiros Sass em 7 pastas + 1 ficheiro principal:

```
scss/
├── abstracts/      # variáveis, funções, mixins, configuração (sem output CSS direto)
│   ├── _variaveis.scss
│   ├── _mixins.scss
│   ├── _funcoes.scss
│   └── _index.scss
├── base/           # reset, tipografia, estilos base
│   ├── _reset.scss
│   └── _tipografia.scss
├── components/     # botões, cards, modais, badges
│   ├── _botao.scss
│   └── _card.scss
├── layout/         # header, footer, grid, navbar
│   ├── _header.scss
│   └── _grid.scss
├── pages/          # estilos específicos de uma página
│   └── _home.scss
├── themes/         # temas alternativos (dark mode, marcas)
│   └── _dark.scss
├── vendors/        # overrides de bibliotecas de terceiros
│   └── _bootstrap.scss
└── main.scss       # ponto de entrada — só @use/@forward
```

`main.scss` fica assim, minimalista:

```scss
@use "abstracts";
@use "base";
@use "components";
@use "layout";
@use "pages";
```

> **Regra de ouro**: nenhum CSS deve ser escrito diretamente em `main.scss` — o ficheiro serve apenas para orquestrar a ordem de carregamento dos módulos.

---

## 17. Sass + BEM

Sass e BEM (Block, Element, Modifier) combinam-se naturalmente graças ao `&`:

```scss
.card {
  padding: 16px;

  &__titulo {
    font-size: 20px;
  }

  &__descricao {
    color: grey;
  }

  &--destacado {
    border: 2px solid gold;
  }

  &--compacto {
    padding: 8px;
  }
}
```

Compila para as classes BEM planas: `.card`, `.card__titulo`, `.card__descricao`, `.card--destacado`, `.card--compacto`.

Um mixin auxiliar comum para elementos e modificadores:

```scss
@mixin elemento($nome) {
  &__#{$nome} {
    @content;
  }
}

@mixin modificador($nome) {
  &--#{$nome} {
    @content;
  }
}

.card {
  @include elemento("titulo") {
    font-size: 20px;
  }

  @include modificador("destacado") {
    border: 2px solid gold;
  }
}
```

> 💡 **Dica**: este padrão de mixins `elemento()`/`modificador()` é opcional — para a maioria dos casos, `&__` e `&--` diretos já são suficientemente legíveis. Só vale a pena introduzir os mixins em bases de código muito grandes, onde a consistência automática compensa a indireção extra.

---

## 18. Novidades recentes do Dart Sass

| Funcionalidade                          | Descrição                                                                 |
| ------------------------------------------ | ------------------------------------------------------------------------------ |
| Sistema de módulos (`@use`/`@forward`)     | Substitui `@import`, que está descontinuado                                  |
| `math.div()`                               | Substitui o operador `/` para divisão                                       |
| `color.adjust()` / `color.scale()`         | Substituem `lighten()`/`darken()`, com suporte a espaços de cor modernos     |
| Suporte nativo a `calc()`, `clamp()`, `min()`/`max()` | O Sass já não tenta resolver estas funções — passam diretamente para o CSS, permitindo misturar valores CSS dinâmicos com Sass |
| Espaços de cor CSS4 (`oklch`, `lab`, `lch`) | Suporte para as novas sintaxes de cor do CSS moderno                        |
| `@debug`, `@warn`, `@error`                | Diretivas para depuração e validação de mixins/funções                      |
| Compilação mais rápida                     | Dart Sass tem vindo a fechar sucessivas otimizações de performance a cada release |

```scss
@mixin validar-cor($cor) {
  @if meta.type-of($cor) != "color" {
    @error "Esperava-se uma cor, recebi #{meta.type-of($cor)}.";
  }
}
```

> ⚠️ **Atenção**: com `@import` a caminho da remoção definitiva, qualquer projeto novo ou biblioteca partilhada deve ser escrito exclusivamente com `@use`/`@forward` desde o primeiro dia.

---

## 19. Boas práticas e erros comuns

| Boa prática                                                     | Porquê                                                             |
| ------------------------------------------------------------------ | ----------------------------------------------------------------------- |
| Usar `@use`/`@forward`, nunca `@import`                            | `@import` está descontinuado e será removido                       |
| Estender apenas placeholders (`%`), nunca classes                  | Evita acoplamento indesejado entre seletores não relacionados      |
| Limitar o nesting a 3 níveis                                       | Seletores CSS mais simples e fáceis de sobrepor                    |
| Centralizar tokens de design em `abstracts/`                       | Um único ponto de verdade para cores, espaçamentos e tipografia    |
| Usar `math.div()` em vez de `/`                                    | `/` está descontinuado para divisão                                |
| Nomear variáveis e mixins em português ou inglês, mas nunca misturar | Consistência facilita a leitura em equipa                        |
| Evitar magic numbers — usar maps de breakpoints/espaçamentos       | Facilita manutenção e escalabilidade                                |

> **Regra de ouro**: se uma regra de estilo se repete em três ou mais sítios, é sinal de que deve tornar-se um mixin, uma função ou um placeholder — nunca copiar e colar.
