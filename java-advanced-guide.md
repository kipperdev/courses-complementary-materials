# Java Tópicos Avançados | Se aprofundando na linguagem

Este guia cobre os temas que separam quem usa Java de quem entende Java: programação assíncrona e paralela com threads e lambdas, os recursos modernos da linguagem e como a JVM gerencia memória através do Garbage Collector.

## Programação assíncrona com Threads

Uma **thread** é uma linha de execução. Por padrão seu programa roda em uma única thread (a `main`), mas você pode criar outras para executar tarefas em paralelo, sem bloquear o fluxo principal.

```java
// criando uma thread com lambda (Runnable é uma interface funcional)
Thread t = new Thread(() -> {
    System.out.println("Rodando em outra thread");
});
t.start();   // dispara a execução
t.join();    // espera a thread terminar
```

> 📌 `start()` cria uma nova thread; chamar `run()` diretamente executaria no mesmo fluxo, sem paralelismo.

### Lambdas e interfaces funcionais

Uma **interface funcional** tem exatamente um método abstrato e pode ser implementada por um **lambda**. É o que torna o código assíncrono e a Stream API concisos.

```java
Runnable tarefa = () -> System.out.println("oi");
Supplier<String> nome = () -> "Kipper";
Function<Integer, Integer> dobro = n -> n * 2;
Predicate<Integer> ehPar = n -> n % 2 == 0;
```

## Programação paralela

Criar threads na mão é trabalhoso e arriscado. O Java oferece abstrações de mais alto nível.

### ExecutorService (pool de threads)

Em vez de criar e destruir threads manualmente, você submete tarefas a um **pool** que as reaproveita.

```java
ExecutorService pool = Executors.newFixedThreadPool(4);

pool.submit(() -> processar(arquivo));

Future<Integer> futuro = pool.submit(() -> calcular()); // retorna resultado
Integer resultado = futuro.get(); // bloqueia até ficar pronto

pool.shutdown();
```

### CompletableFuture (assíncrono encadeável)

Permite encadear tarefas assíncronas sem bloquear:

```java
CompletableFuture
    .supplyAsync(() -> buscarUsuario(id))
    .thenApply(usuario -> usuario.getEmail())
    .thenAccept(email -> enviar(email));
```

### Streams paralelas

Para processar grandes coleções dividindo o trabalho entre os núcleos da CPU:

```java
long pares = numeros.parallelStream()
    .filter(n -> n % 2 == 0)
    .count();
```

> ⚠️ Paralelismo só vale a pena com volume grande de dados e operações independentes. Para listas pequenas, o custo de coordenar as threads pode ser maior que o ganho.

### Cuidados com concorrência

Quando várias threads acessam o mesmo dado, podem ocorrer **race conditions**. Proteja o estado compartilhado:

```java
// synchronized garante que só uma thread execute o bloco por vez
public synchronized void incrementar() { contador++; }

// ou use estruturas thread-safe
AtomicInteger contador = new AtomicInteger(0);
contador.incrementAndGet();
```

## Recursos do Java moderno

### Records

Classes imutáveis de dados sem boilerplate (construtor, getters, `equals`, `hashCode`, `toString` gerados):

```java
public record Ponto(int x, int y) {}
```

### Text Blocks

Strings multi-linha legíveis, sem concatenação:

```java
String json = """
    {
        "nome": "Kipper",
        "ativo": true
    }
    """;
```

### Switch expressions

O `switch` que retorna valor, sem `break`, sem "fall-through" acidental:

```java
String tipo = switch (codigo) {
    case 1, 2 -> "básico";
    case 3 -> "premium";
    default -> "desconhecido";
};
```

### var (inferência de tipo local)

```java
var lista = new ArrayList<String>(); // o tipo é inferido
```

## Garbage Collector (GC)

Em Java você **não libera memória manualmente**. O **Garbage Collector** é o processo da JVM que identifica objetos que não são mais referenciados por ninguém e libera a memória que eles ocupavam.

### Como ele decide o que remover

Um objeto vira "lixo" quando **não há mais nenhuma referência alcançável** a ele:

```java
Produto p = new Produto("Café", 25); // objeto criado
p = null; // a referência sumiu -> o objeto fica elegível para o GC
```

### Gerações da memória (Heap)

A JVM divide a memória em gerações, partindo da observação de que a maioria dos objetos morre jovem:

- **Young Generation:** onde objetos novos nascem. A maioria é coletada rápido aqui (Minor GC, barato).
- **Old Generation:** objetos que sobreviveram a várias coletas "envelhecem" e vão para cá (Major GC, mais caro).

### Pontos importantes

- O GC roda automaticamente; você não controla *quando*.
- `System.gc()` é apenas uma **sugestão**, não uma garantia, evite depender dele.
- Vazamentos de memória em Java acontecem quando você mantém referências vivas sem querer (ex.: coleções estáticas que só crescem).
- A JVM oferece diferentes coletores (G1, ZGC, Parallel) ajustáveis conforme a necessidade de latência ou throughput.

## Resumo

- **Threads** executam tarefas em paralelo; prefira `ExecutorService` e `CompletableFuture` a criar threads na mão.
- **Lambdas** implementam interfaces funcionais e deixam o código assíncrono conciso.
- Proteja estado compartilhado (`synchronized`, classes `Atomic`) para evitar race conditions.
- Java moderno: **Records, Text Blocks, switch expressions, var**.
- O **Garbage Collector** libera automaticamente objetos sem referências; entender Young/Old Generation ajuda a escrever código que não vaza memória.
