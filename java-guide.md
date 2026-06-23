# Curso de Java | Fundamentos da Linguagem + Programação Orientada a Objetos

Java é uma linguagem **compilada e fortemente tipada**, orientada a objetos, que roda sobre a **JVM (Java Virtual Machine)**. Você escreve o código `.java`, ele é compilado para **bytecode** (`.class`), e a JVM executa esse bytecode em qualquer sistema operacional, daí o lema "write once, run anywhere".

```bash
javac Main.java   # compila para bytecode
java Main         # executa na JVM
```

## Estrutura básica

Todo programa Java começa em um método `main` dentro de uma classe:

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Olá, mundo!");
    }
}
```

## Tipos primitivos x objetos

Java tem **8 tipos primitivos** (guardam o valor diretamente) e os demais são objetos (guardam uma referência).

```java
int idade = 25;
double preco = 19.90;
boolean ativo = true;
char letra = 'A';
long grande = 10_000_000L;

// Tipos por referência (objetos)
String nome = "Fernanda";
Integer idadeObj = 25; // wrapper do int
```

> 📌 Para cada primitivo existe uma classe "wrapper" (`int` → `Integer`, `double` → `Double`...). Coleções e generics só trabalham com objetos, por isso os wrappers existem.

## Variáveis, operadores e controle de fluxo

```java
// condicional
if (idade >= 18) {
    System.out.println("Maior de idade");
} else {
    System.out.println("Menor de idade");
}

// laços
for (int i = 0; i < 5; i++) System.out.println(i);

int j = 0;
while (j < 5) { j++; }

// switch
switch (status) {
    case "ATIVO" -> System.out.println("ok");
    default -> System.out.println("desconhecido");
}
```

## Programação Orientada a Objetos (POO)

A POO organiza o código em torno de **objetos**, que combinam dados (atributos) e comportamento (métodos). Uma **classe** é o molde; o **objeto** é a instância criada a partir dele.

```java
public class Conta {
    private double saldo; // atributo

    public void depositar(double valor) { // método
        this.saldo += valor;
    }

    public double getSaldo() {
        return this.saldo;
    }
}

// criando objetos
Conta conta = new Conta();
conta.depositar(100);
```

### Os 4 pilares da POO

**1. Encapsulamento**
Esconder os detalhes internos e expor apenas o necessário. Atributos ficam `private` e o acesso é feito por métodos (`getters`/`setters`), o que protege o estado do objeto.

```java
public class Pessoa {
    private String nome;
    public String getNome() { return nome; }
    public void setNome(String nome) { this.nome = nome; }
}
```

**2. Herança**
Uma classe pode herdar atributos e comportamentos de outra com `extends`, reaproveitando código.

```java
public class Animal {
    public void comer() { System.out.println("comendo"); }
}

public class Cachorro extends Animal {
    public void latir() { System.out.println("au au"); }
}
```

**3. Polimorfismo**
O mesmo método pode se comportar de formas diferentes dependendo do objeto. Conseguimos isso sobrescrevendo métodos (`@Override`).

```java
public class Animal {
    public String som() { return "som genérico"; }
}

public class Gato extends Animal {
    @Override
    public String som() { return "miau"; }
}

Animal a = new Gato();
a.som(); // "miau" - decidido em tempo de execução
```

**4. Abstração**
Expor *o que* um objeto faz, escondendo *como* ele faz. Conseguimos com **classes abstratas** e **interfaces**.

```java
// interface: um contrato, sem implementação
public interface Forma {
    double area();
}

public class Circulo implements Forma {
    private double raio;
    public Circulo(double raio) { this.raio = raio; }

    @Override
    public double area() {
        return Math.PI * raio * raio;
    }
}
```

### Classe abstrata x Interface

- **Classe abstrata (`abstract`):** pode ter métodos com e sem implementação e atributos de estado. Uma classe só pode estender **uma** classe abstrata. Use quando há uma relação "é um tipo de".
- **Interface:** define um contrato (o quê), e uma classe pode implementar **várias** interfaces. Use para definir capacidades ("é capaz de").

## Construtores

São métodos especiais chamados na criação do objeto, usados para inicializar o estado.

```java
public class Produto {
    private String nome;
    private double preco;

    public Produto(String nome, double preco) {
        this.nome = nome;
        this.preco = preco;
    }
}

Produto p = new Produto("Café", 25.0);
```

## Modificadores de acesso

| Modificador | Visível em |
| --- | --- |
| `public` | qualquer lugar |
| `protected` | mesmo pacote + subclasses |
| (padrão) | mesmo pacote |
| `private` | apenas na própria classe |

## Resumo

- Java compila para bytecode e roda na JVM.
- Tipos primitivos guardam valor; objetos guardam referência (com wrappers para os primitivos).
- POO organiza o código em classes e objetos.
- Pilares: **encapsulamento, herança, polimorfismo, abstração**.
- Interfaces definem contratos; classes abstratas definem uma base parcial.
