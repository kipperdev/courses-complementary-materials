# Curso de Java Spring | Fundamentos do Framework

Spring é o framework mais usado para construir aplicações backend em Java. O **Spring Boot** é a porta de entrada moderna: ele configura quase tudo para você (servidor embutido, conexões, defaults sensatos), permitindo focar na lógica da aplicação em vez de configuração.

## Criando um projeto

A forma mais simples é pelo [Spring Initializr](https://start.spring.io/): escolha Maven, Java, e adicione as dependências (Spring Web, Spring Data JPA, driver do banco). Ele gera um projeto pronto para rodar.

```bash
./mvnw spring-boot:run   # roda a aplicação
```

A classe principal:

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

## Inversão de Controle e Injeção de Dependência

O coração do Spring é o **IoC Container**. Em vez de você criar objetos manualmente com `new`, o Spring cria e gerencia essas instâncias (chamadas **beans**) e as **injeta** onde forem necessárias.

```java
@Service
public class PedidoService {
    private final PedidoRepository repository;

    // o Spring injeta o repository automaticamente pelo construtor
    public PedidoService(PedidoRepository repository) {
        this.repository = repository;
    }
}
```

> 📌 Prefira **injeção via construtor** (como acima) a injeção via `@Autowired` em campo: deixa as dependências explícitas, facilita testes e permite campos `final`.

### Principais anotações (estereótipos)

O Spring detecta e gerencia automaticamente classes marcadas com:

| Anotação | Papel |
| --- | --- |
| `@RestController` | recebe requisições HTTP e devolve dados (JSON) |
| `@Service` | contém a lógica de negócio |
| `@Repository` | acesso a dados |
| `@Component` | bean genérico gerenciado pelo Spring |
| `@Configuration` | classe que define beans manualmente |

## Camadas de uma aplicação Spring

Uma organização comum separa responsabilidades em camadas:

```
Controller  ->  Service  ->  Repository  ->  Banco de dados
(HTTP)          (regras)     (persistência)
```

### Controller

Expõe os endpoints da API:

```java
@RestController
@RequestMapping("/produtos")
public class ProdutoController {

    private final ProdutoService service;

    public ProdutoController(ProdutoService service) {
        this.service = service;
    }

    @GetMapping
    public List<Produto> listar() {
        return service.listarTodos();
    }

    @GetMapping("/{id}")
    public Produto buscar(@PathVariable Long id) {
        return service.buscarPorId(id);
    }

    @PostMapping
    public ResponseEntity<Produto> criar(@RequestBody ProdutoDTO dto,
                                         UriComponentsBuilder uriBuilder) {
        Produto criado = service.criar(dto);
        var uri = uriBuilder.path("/produtos/{id}")
                            .buildAndExpand(criado.getId()).toUri();
        return ResponseEntity.created(uri).body(criado);
    }
}
```

Anotações de mapeamento:

- `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping` — verbos HTTP.
- `@PathVariable` — injeta um valor da URL (`/produtos/5`).
- `@RequestBody` — injeta o corpo JSON da requisição.
- `@RequestParam` — injeta query params (`?nome=café`).

## Persistência com Spring Data JPA

O **JPA** mapeia classes Java para tabelas do banco (ORM). O **Spring Data** elimina o código repetitivo de acesso a dados.

### Entidade

```java
@Entity
@Table(name = "produtos")
public class Produto {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nome;
    private double preco;

    // getters e setters
}
```

### Repository

Basta estender `JpaRepository` e você ganha `save`, `findById`, `findAll`, `delete` etc. de graça:

```java
public interface ProdutoRepository extends JpaRepository<Produto, Long> {
    // queries derivadas do nome do método
    List<Produto> findByNomeContaining(String nome);
    List<Produto> findByPrecoGreaterThan(double preco);
}
```

### Configuração do banco (`application.properties`)

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/meudb
spring.datasource.username=postgres
spring.datasource.password=senha
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

## DTOs

Evite expor suas entidades diretamente na API. Use **DTOs** (Data Transfer Objects) para controlar o que entra e sai, geralmente como `record`:

```java
public record ProdutoDTO(String nome, double preco) {}
```

## Resumo

- **Spring Boot** configura o essencial automaticamente; comece pelo Spring Initializr.
- O **IoC Container** cria e injeta os beans; prefira injeção por construtor.
- Organize em camadas: **Controller → Service → Repository**.
- **Spring Data JPA** mapeia entidades e gera os métodos de acesso a dados.
- Use **DTOs** para não expor entidades diretamente.

> Para temas além dos fundamentos (tratamento de exceções, Spring Security e migrations), veja o guia **Conhecendo o framework Spring** (`spring-advanced-guide.md`).
