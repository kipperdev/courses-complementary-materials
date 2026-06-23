# Curso de Tailwind para Iniciantes

Tailwind CSS é um framework CSS **utility-first** (baseado em utilitários). Em vez de você escrever suas próprias classes CSS e estilizar em arquivos separados, você monta o estilo direto no HTML combinando pequenas classes utilitárias, cada uma responsável por uma única propriedade.

```html
<!-- CSS tradicional -->
<button class="btn-primary">Enviar</button>

<!-- Tailwind: o estilo vive na marcação -->
<button class="bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700">
  Enviar
</button>
```

> 📌 A ideia parece estranha no começo ("não vou poluir meu HTML?"), mas na prática você para de inventar nomes de classe, para de pular entre arquivos, e o CSS final fica pequeno porque o Tailwind remove tudo que você não usa.

## Por que usar?

- **Velocidade:** você estiliza sem sair do HTML e sem nomear classes.
- **Consistência:** as cores, espaçamentos e tamanhos vêm de uma escala pré-definida (design tokens), então tudo fica harmônico.
- **Bundle pequeno:** o Tailwind faz "purge" e gera só o CSS das classes que você realmente usou.
- **Responsivo de forma simples:** breakpoints são prefixos (`md:`, `lg:`).

## Instalação

```bash
npm install tailwindcss @tailwindcss/cli
npx @tailwindcss/cli -i ./src/input.css -o ./dist/output.css --watch
```

No seu CSS de entrada:

```css
@import "tailwindcss";
```

E inclua o CSS gerado no HTML. Em projetos com framework (Vite, Next, Angular), use o plugin oficial correspondente.

## Anatomia de uma classe

Quase toda classe segue o padrão `propriedade-valor`:

| Classe | O que faz |
| --- | --- |
| `p-4` | padding em todos os lados (escala 4 = 1rem) |
| `px-4` `py-2` | padding horizontal / vertical |
| `mt-8` | margin-top |
| `text-lg` | tamanho da fonte |
| `font-bold` | peso da fonte |
| `text-gray-700` | cor do texto |
| `bg-blue-600` | cor de fundo |
| `rounded-lg` | borda arredondada |
| `flex` `grid` | tipo de layout |

A **escala de espaçamento** é numérica: `1 = 0.25rem`, `2 = 0.5rem`, `4 = 1rem`, `8 = 2rem`... Isso garante espaçamentos consistentes.

## Cores

As cores seguem o padrão `cor-intensidade`, indo de `50` (mais clara) a `950` (mais escura):

```html
<div class="bg-red-50 text-red-900 border border-red-200">Alerta suave</div>
<div class="bg-emerald-600 text-white">Sucesso</div>
```

## Layout com Flexbox e Grid

```html
<!-- Flexbox: itens em linha, centralizados, com espaço entre eles -->
<div class="flex items-center justify-between gap-4">
  <span>Logo</span>
  <nav>Menu</nav>
</div>

<!-- Grid de 3 colunas -->
<div class="grid grid-cols-3 gap-6">
  <div>1</div>
  <div>2</div>
  <div>3</div>
</div>
```

## Responsividade

O Tailwind é **mobile-first**: classes sem prefixo valem para todas as telas, e os prefixos aplicam a partir de cada breakpoint para cima.

```html
<!-- 1 coluna no celular, 2 no tablet, 4 no desktop -->
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
  ...
</div>
```

| Prefixo | A partir de |
| --- | --- |
| `sm:` | 640px |
| `md:` | 768px |
| `lg:` | 1024px |
| `xl:` | 1280px |
| `2xl:` | 1536px |

## Estados (hover, focus, etc.)

Basta prefixar a classe com o estado:

```html
<button class="bg-blue-600 hover:bg-blue-700 focus:ring-2 active:scale-95 transition">
  Clique
</button>
```

`transition` deixa a mudança suave. Outros estados úteis: `disabled:`, `group-hover:`, `dark:` (modo escuro).

## Modo escuro

```html
<div class="bg-white text-gray-900 dark:bg-gray-900 dark:text-white">
  Funciona nos dois temas
</div>
```

## Reaproveitando estilos

Quando um conjunto de classes se repete muito, você tem duas opções saudáveis:

1. **Componentizar** (React/Angular/Vue): crie um componente `<Button>` e centralize as classes ali. Esta é a forma recomendada.
2. **`@apply`** no CSS, para casos pontuais:

```css
.btn {
  @apply px-4 py-2 rounded-lg bg-blue-600 text-white hover:bg-blue-700;
}
```

> ⚠️ Evite sair criando classes com `@apply` para tudo, isso desfaz a vantagem do utility-first. Prefira componentes.

## Resumo

- Tailwind = pequenas classes utilitárias compostas direto no HTML.
- Escala consistente de espaçamento e cores (design tokens).
- Mobile-first: prefixos `md:`, `lg:` aplicam de um breakpoint para cima.
- Estados via prefixo: `hover:`, `focus:`, `dark:`.
- Para reuso, prefira **componentes** a `@apply`.
