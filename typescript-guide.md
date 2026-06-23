# Fundamentos de TypeScript

TypeScript é um superset do JavaScript, ou seja, é o JavaScript com a adição de **tipagem estática**. Todo código JavaScript válido também é um código TypeScript válido, mas o TypeScript adiciona uma camada de verificação de tipos que acontece em tempo de compilação (antes do código rodar).

> 📌 O navegador e o Node.js não executam TypeScript diretamente. O código `.ts` é **compilado** (transpilado) para JavaScript `.js`, e é esse JavaScript que de fato roda.

## Por que usar TypeScript?

- **Pega erros antes de rodar:** muitos bugs (acessar propriedade que não existe, passar o argumento errado) são detectados enquanto você escreve, não em produção.
- **Autocomplete e documentação:** o editor entende seus tipos e sugere métodos/propriedades corretas.
- **Refatoração segura:** renomear ou mudar a forma de um objeto avisa todos os lugares que precisam mudar.
- **Escala melhor:** em projetos grandes e com muitas pessoas, os tipos funcionam como um contrato.

## Configuração

```bash
npm install -g typescript
tsc --init   # cria o tsconfig.json
tsc          # compila o projeto
tsc --watch  # recompila a cada alteração
```

O `tsconfig.json` controla o comportamento do compilador. Opções importantes:

```jsonc
{
  "compilerOptions": {
    "target": "ES2020",      // versão do JS gerado
    "strict": true,          // ativa todas as checagens rígidas (recomendado)
    "outDir": "./dist",      // onde os .js serão gerados
    "rootDir": "./src"
  }
}
```

## Tipos básicos

```typescript
let nome: string = "Fernanda";
let idade: number = 25;
let ativo: boolean = true;
let lista: number[] = [1, 2, 3];
let tupla: [string, number] = ["idade", 25];

// any desliga a checagem de tipos (evite usar)
let qualquerCoisa: any = "pode ser tudo";

// unknown é o "any seguro": exige checagem antes de usar
let valor: unknown = pegarValor();

// union types: pode ser um tipo OU outro
let id: string | number = 10;
```

## Inferência de tipos

Você nem sempre precisa anotar o tipo. O TypeScript **infere** sozinho a partir do valor:

```typescript
let cidade = "Pelotas"; // inferido como string
// cidade = 10;         // ❌ erro: number não é atribuível a string
```

Anote os tipos explicitamente em **parâmetros de função** e em **contratos públicos** (retornos de API, props), e deixe a inferência cuidar do resto.

## Funções

```typescript
function soma(a: number, b: number): number {
  return a + b;
}

// parâmetro opcional (?) e valor padrão
function saudacao(nome: string, sobrenome?: string): string {
  return `Olá, ${nome}`;
}

// arrow function tipada
const dobro = (n: number): number => n * 2;
```

## Interfaces e Types

São as duas formas de descrever a "forma" de um objeto.

```typescript
interface Usuario {
  id: number;
  nome: string;
  email?: string;       // opcional
  readonly criadoEm: Date; // não pode ser alterado depois
}

const user: Usuario = {
  id: 1,
  nome: "Kipper",
  criadoEm: new Date(),
};
```

```typescript
// type alias faz quase o mesmo, e ainda permite unions e interseções
type Status = "ativo" | "inativo" | "pendente";

type Coordenada = { x: number; y: number };
type Ponto3D = Coordenada & { z: number }; // interseção
```

> 📌 Regra prática: use `interface` para descrever objetos e contratos (ela pode ser estendida e "mesclada"); use `type` quando precisar de unions, tuplas ou combinações mais complexas.

## Generics

Generics permitem escrever código reutilizável que funciona com **qualquer tipo**, mantendo a segurança de tipos.

```typescript
function primeiro<T>(lista: T[]): T {
  return lista[0];
}

const n = primeiro([1, 2, 3]);       // T = number
const s = primeiro(["a", "b"]);      // T = string

// interface genérica
interface Resposta<T> {
  dados: T;
  status: number;
}

const resposta: Resposta<Usuario> = { dados: user, status: 200 };
```

## Enums

```typescript
enum Perfil {
  Admin = "ADMIN",
  Editor = "EDITOR",
  Leitor = "LEITOR",
}

const meuPerfil: Perfil = Perfil.Admin;
```

## Narrowing (estreitamento de tipos)

Quando um valor pode ser de vários tipos, o TypeScript te obriga a checar antes de usar:

```typescript
function imprimir(id: string | number) {
  if (typeof id === "string") {
    console.log(id.toUpperCase()); // aqui o TS sabe que é string
  } else {
    console.log(id.toFixed(2));    // aqui sabe que é number
  }
}
```

## Utility Types úteis

```typescript
Partial<Usuario>   // todas as propriedades viram opcionais
Required<Usuario>  // todas viram obrigatórias
Pick<Usuario, "id" | "nome"> // só algumas propriedades
Omit<Usuario, "email">       // todas menos algumas
Record<string, number>       // objeto com chaves string e valores number
```

## Resumo

- TypeScript = JavaScript + tipos checados em tempo de compilação.
- Deixe `strict: true` ligado.
- Use inferência sempre que possível e anote os contratos (parâmetros, retornos públicos).
- `interface`/`type` descrevem objetos; **generics** dão reuso com segurança.
- Prefira `unknown` a `any`, e use **narrowing** para tratar unions com segurança.
