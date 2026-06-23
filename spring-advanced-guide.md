# Conhecendo o framework Spring | Exceções, Spring Security e Migrations

Este guia aprofunda três temas essenciais para uma API Spring pronta para produção: tratamento centralizado de exceções, autenticação e autorização com Spring Security, e controle de versão do banco de dados com migrations.

> Pré-requisito: os fundamentos do framework (IoC, camadas, JPA). Veja `spring-guide.md`.

## 1. Tratamento de exceções

Sem tratamento, qualquer erro vira um `500` genérico com stack trace exposto. O ideal é capturar as exceções em um ponto central e devolver respostas HTTP claras e consistentes.

### Exceções personalizadas

Crie exceções específicas do seu domínio:

```java
public class ProdutoNaoEncontradoException extends RuntimeException {
    public ProdutoNaoEncontradoException() {
        super("Produto não encontrado");
    }
}
```

E lance no service quando fizer sentido:

```java
public Produto buscarPorId(Long id) {
    return repository.findById(id)
        .orElseThrow(ProdutoNaoEncontradoException::new);
}
```

### Handler global com @RestControllerAdvice

Uma única classe captura as exceções de todos os controllers e transforma cada uma na resposta HTTP adequada:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ProdutoNaoEncontradoException.class)
    public ResponseEntity<ErroDTO> handleNaoEncontrado(ProdutoNaoEncontradoException ex) {
        var erro = new ErroDTO(ex.getMessage(), HttpStatus.NOT_FOUND.value());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(erro);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErroDTO> handleValidacao(MethodArgumentNotValidException ex) {
        var msg = ex.getBindingResult().getFieldError().getDefaultMessage();
        var erro = new ErroDTO(msg, HttpStatus.BAD_REQUEST.value());
        return ResponseEntity.badRequest().body(erro);
    }
}

public record ErroDTO(String mensagem, int status) {}
```

> 📌 Vantagem: os controllers ficam limpos (sem `try/catch` por toda parte) e a API responde de forma padronizada.

## 2. Spring Security (Autenticação e Autorização)

- **Autenticação:** *quem é você?* (login, validar credenciais).
- **Autorização:** *o que você pode fazer?* (permissões, papéis/roles).

### Conceitos

O Spring Security intercepta as requisições através de uma **cadeia de filtros**. Um fluxo comum em APIs REST usa **JWT (JSON Web Token)**: o usuário faz login uma vez, recebe um token assinado, e o envia no header `Authorization: Bearer <token>` nas próximas requisições.

```
Login (usuário/senha) -> servidor valida -> gera JWT -> cliente guarda
Requisições seguintes -> envia o JWT -> filtro valida -> libera o acesso
```

### Configuração da cadeia de segurança

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .csrf(csrf -> csrf.disable())
            .sessionManagement(s -> s.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers(HttpMethod.POST, "/login").permitAll()
                .requestMatchers(HttpMethod.GET, "/produtos").permitAll()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(); // nunca guarde senha em texto puro
    }
}
```

### Pontos importantes

- **Nunca** armazene senhas em texto puro, use um hash forte (`BCrypt`).
- APIs REST normalmente são **stateless** (`SessionCreationPolicy.STATELESS`), sem sessão no servidor; o JWT carrega a identidade.
- **Roles/Authorities** controlam a autorização (`hasRole("ADMIN")`, `hasAuthority("LEITURA")`).
- Para emitir e validar JWTs, uma biblioteca comum é a `java-jwt` (Auth0).

## 3. Migrations

**Migrations** versionam o schema do banco de dados de forma controlada e rastreável. Em vez de alterar tabelas na mão (e perder o histórico), você descreve cada mudança em um arquivo versionado que roda automaticamente. As ferramentas mais usadas no ecossistema Spring são **Flyway** e **Liquibase**.

### Flyway

Adicione a dependência e crie arquivos SQL em `src/main/resources/db/migration`, seguindo a convenção de nome:

```
V<VERSÃO>__<descrição>.sql
```

Exemplos:

```
V1__create_table_produtos.sql
V2__add_column_ativo.sql
V3__create_table_pedidos.sql
```

Conteúdo de uma migration (SQL puro):

```sql
-- V1__create_table_produtos.sql
CREATE TABLE produtos (
    id BIGSERIAL PRIMARY KEY,
    nome VARCHAR(255) NOT NULL,
    preco NUMERIC(10,2) NOT NULL
);
```

```sql
-- V2__add_column_ativo.sql
ALTER TABLE produtos ADD COLUMN ativo BOOLEAN DEFAULT TRUE;
```

### Como funciona

- Na inicialização, o Flyway verifica quais migrations já foram aplicadas (ele guarda isso em uma tabela de controle, `flyway_schema_history`).
- Aplica, **em ordem de versão**, apenas as que ainda não rodaram.
- Migrations já aplicadas **nunca devem ser editadas**, para corrigir, crie uma nova versão.

> ⚠️ Com migrations, desligue o `ddl-auto=update` do Hibernate em produção (`spring.jpa.hibernate.ddl-auto=validate`). Quem manda no schema passam a ser as migrations, não o Hibernate.

## Resumo

- **Exceções:** crie exceções de domínio e centralize o tratamento com `@RestControllerAdvice`, devolvendo status HTTP adequados.
- **Spring Security:** autenticação (quem é) x autorização (o que pode); em APIs REST, padrão **JWT + stateless**, senhas sempre com hash (`BCrypt`).
- **Migrations:** versione o schema com Flyway (`V1__...sql`), nunca edite uma migration já aplicada, e deixe `ddl-auto=validate` em produção.
