# Java Intermediário | Estruturas de dados, Stream API, Generics, Packages e Records

Depois de dominar os fundamentos e a POO, o passo intermediário é conhecer as ferramentas que você vai usar todos os dias em código real: as coleções, a Stream API para processá-las, generics para reuso seguro, organização em packages e os recursos modernos da linguagem como Records.

## Packages

**Packages** organizam suas classes em namespaces, evitam conflito de nomes e definem visibilidade. Por convenção usam o domínio invertido da empresa.

```java
package com.kipper.ecommerce.domain;

import java.util.List;            // importando uma classe
import com.kipper.ecommerce.dto.*; // importando um pacote inteiro
```

## Estruturas de dados (Collections Framework)

O Java oferece um conjunto de interfaces e implementações para guardar grupos de objetos. As principais:

### List — coleção ordenada que aceita duplicados

```java
List<String> nomes = new ArrayList<>();
nomes.add("Ana");
nomes.add("Bruno");
nomes.get(0);          // "Ana"
nomes.size();          // 2
```

`ArrayList` é rápido para leitura por índice; `LinkedList` é melhor para muitas inserções/remoções no meio.

### Set — coleção sem duplicados

```java
Set<String> tags = new HashSet<>();
tags.add("java");
tags.add("java"); // ignorado, já existe
```

`HashSet` não garante ordem; `LinkedHashSet` mantém a ordem de inserção; `TreeSet` mantém ordenado.

### Map — pares chave → valor

```java
Map<String, Integer> estoque = new HashMap<>();
estoque.put("café", 10);
estoque.get("café");              // 10
estoque.getOrDefault("chá", 0);   // 0
estoque.containsKey("café");      // true
```

### Quando usar cada uma

| Precisa de... | Use |
| --- | --- |
| Ordem e acesso por índice, aceita repetição | `List` / `ArrayList` |
| Garantir itens únicos | `Set` / `HashSet` |
| Associar uma chave a um valor | `Map` / `HashMap` |
| Coleção sempre ordenada | `TreeSet` / `TreeMap` |

## Generics

Generics permitem escrever classes e métodos que funcionam com qualquer tipo mantendo a segurança em tempo de compilação (sem precisar de `Object` + casting).

```java
public class Caixa<T> {
    private T conteudo;
    public void guardar(T item) { this.conteudo = item; }
    public T pegar() { return conteudo; }
}

Caixa<String> caixa = new Caixa<>();
caixa.guardar("livro");
String item = caixa.pegar(); // sem cast, sem risco

// método genérico
public static <T> T primeiro(List<T> lista) {
    return lista.get(0);
}
```

## Stream API

Streams permitem processar coleções de forma **declarativa** (você diz *o que* quer, não *como* iterar), encadeando operações como filtrar, transformar e reduzir.

```java
List<Produto> produtos = List.of(
    new Produto("Café", 25.0),
    new Produto("Chá", 15.0),
    new Produto("Bolo", 40.0)
);

// nomes dos produtos acima de R$ 20, em maiúsculas, ordenados
List<String> resultado = produtos.stream()
    .filter(p -> p.getPreco() > 20)       // operação intermediária
    .map(p -> p.getNome().toUpperCase())  // operação intermediária
    .sorted()                             // operação intermediária
    .toList();                            // operação terminal
```

Operações comuns:

- **`filter`** — mantém só os elementos que passam na condição.
- **`map`** — transforma cada elemento em outra coisa.
- **`sorted`** — ordena.
- **`reduce`** — combina todos em um único valor.
- **`collect` / `toList`** — materializa o resultado.
- **`count`, `anyMatch`, `findFirst`** — operações terminais úteis.

```java
double total = produtos.stream()
    .mapToDouble(Produto::getPreco)
    .sum();
```

> 📌 As operações intermediárias são **preguiçosas** (lazy): nada é processado até uma operação terminal ser chamada. Isso permite que o Java otimize a execução em uma única passagem.

## Records

`record` é uma forma concisa de criar classes **imutáveis** feitas só para carregar dados (DTOs, value objects). O compilador gera automaticamente construtor, getters, `equals`, `hashCode` e `toString`.

```java
public record Produto(String nome, double preco) {}

Produto p = new Produto("Café", 25.0);
p.nome();   // acesso aos campos (sem "get")
p.preco();
```

Equivale a uma classe com ~40 linhas de boilerplate. Use records sempre que precisar de um objeto imutável que apenas agrupa valores.

## Optional

`Optional<T>` representa um valor que **pode ou não existir**, evitando `NullPointerException` e deixando a ausência explícita.

```java
Optional<Produto> achado = produtos.stream()
    .filter(p -> p.nome().equals("Café"))
    .findFirst();

achado.ifPresent(p -> System.out.println(p.preco()));
Produto p = achado.orElse(new Produto("Padrão", 0));
```

## Resumo

- **Packages** organizam o código; importe o que precisar.
- **Collections:** `List` (ordenada, duplica), `Set` (única), `Map` (chave/valor).
- **Generics** dão reuso com segurança de tipos.
- **Stream API** processa coleções de forma declarativa e preguiçosa.
- **Records** eliminam boilerplate de classes imutáveis de dados.
- **Optional** torna a ausência de valor explícita e segura.
