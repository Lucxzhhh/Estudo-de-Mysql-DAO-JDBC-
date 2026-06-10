# Estudo de Mysql-DAO-JDBC

# Estudo de MySQL-DAO-JDBC

---

## O que é o DAO (Data Access Object)
DAO (Data Access Object) é um padrão de projeto *(design pattern)*, onde ocorre uma divisão da lógica de acessos a dados da lógica de negócio em uma aplicação. Ele encapsula todas as interações com o banco de dados, fazendo com que a lógica de negócio não precise lidar diretamente com os comandos de SQL.

Em um projeto organizado em DAO, a classe DAO é a responsável por realizar as operações **CRUD** *(Create, Read, Update, Delete)* sobre uma entidade do projeto, obtendo uma camada de abstração entre o banco de dados e a lógica da aplicação.

---

## Quais problemas o DAO resolve?
O padrão DAO surgiu na programação orientada a objetos como uma forma de solução para reduzir o acoplamento entre a camada de negócio e a camada de persistência.

### A introdução do DAO resolveu esses problemas:
- Criar uma camada intermediária entre o banco de dados e a lógica de negócio.
- Permitir a reutilização de código para operações comuns no banco de dados.
- Facilitar a troca de fornecedores de banco de dados sem grandes impactos na lógica da aplicação.
- Melhorar a testabilidade, pois permite substituir DAOs reais por implementações simuladas *(mock)* durante os testes.

---

## Arquitetura das camadas
É uma forma de organizar o código separando responsabilidades. Cada camada tem uma função específica e não precisa saber como as outras funcionam por dentro.

### Model
- Contém as entidades do sistema (objetos que representam dados).
- Exemplos: classe *Cliente*, *Produto*, *Pedido*.
- São classes simples com atributos, *getters* (métodos de acesso) e *setters* (métodos de modificação).
- Funciona como um container de dados, sem lógica de negócio.

### dao
- Contém as interfaces (contratos).
- Define os métodos que existem, mas não como eles funcionam.
- A interface define os métodos **CRUD** e mantém o código modular, permitindo trocar implementações sem mudar a lógica de negócio.
- Exemplo: *ClienteDao* com métodos *inserir()*, *buscarPorId()*, *listarTodos()*, *atualizar()*, *deletar()*.

### dao.impl
- Contém as implementações das interfaces.
- É aqui que fica o SQL de verdade, o JDBC, o *PreparedStatement*.
- É onde ficam as implementações concretas dos DAOs usando JDBC para interagir com o banco MySQL.
- Exemplo: *ClienteDaoJDBC* que implementa *ClienteDao*.

### db
- Contém a infraestrutura de conexão.
- Inclui a classe utilitária de conexão com o banco, a exceção customizada *DbException* e a *DaoFactory* para instanciar os DAOs.

### app
- Contém o ponto de entrada do sistema.
- É o *Main.java* com o menu e a orquestração das ações do usuário.
- Chama os DAOs através das interfaces, sem saber nada do SQL.

### Por que separar assim?
Essa separação permite que as duas partes evoluam de forma independente. Se a lógica de negócio mudar, ela continua dependendo da interface DAO. Se a lógica de persistência mudar, os clientes DAO não serão afetados.

---

## Conceitos básicos de banco de dados
Antes de trabalhar com JDBC, é importante entender alguns conceitos básicos de banco de dados.

- **INT** → números inteiros (ex: idade, id, quantidade).
- **VARCHAR** → textos (ex: nome, email).
- **DECIMAL** → números com casas decimais (ex: preço).
- **DATETIME** → data e hora.
- **PK** *(Primary Key)* → identifica unicamente cada registro na tabela.
- **FK** *(Foreign Key)* → cria relacionamento entre tabelas.
- **NOT NULL** → o campo não pode ficar vazio.
- **UNIQUE** → o valor não pode se repetir na tabela.

---

## JDBC — Como funciona a conexão Java com banco de dados
JDBC é semelhante ao ODBC, e no princípio utilizava o ODBC para conectar-se com o banco de dados. A partir de um código nativo, as aplicações Java podiam usar qualquer banco de dados que tivesse um driver ODBC disponível. Desta forma ajudou muito a popularizar o JDBC, uma vez que existe um driver ODBC para praticamente qualquer banco de dados de mercado.

Assim como ODBC, JDBC também funciona através de drivers que são responsáveis pela conexão com o banco e execução das instruções SQL. Esses drivers foram divididos em quatro tipos.

<img width="642" height="740" alt="image" src="https://github.com/user-attachments/assets/c0900ef1-8ee6-4dff-b199-718b7b898e15" />

Esses drivers são implementações das interfaces do pacote *java.sql*. Geralmente estão disponibilizados em arquivos **JAR** *(Java ARchive)* pelos fabricantes do banco de dados ou terceiros. Você pode encontrar esses drivers JDBC em www.oracle.com, www.dev.mysql.com, www.microsoft.com, www.ibm.com.

Após fazer o download do driver, basta incluí-lo ao **CLASSPATH**. A partir desse momento está tudo pronto para você acessar o banco de dados via Java.

> **Nota:** Na maioria dos IDEs a variável de ambiente **CLASSPATH** é ignorada (utilizam uma forma própria de gerenciar as classes usadas pelo projeto). Caso isso ocorra, você deve adicionar o driver no projeto.

---

## DriverManager e getConnection
*DriverManager* é uma classe totalmente implementada que conecta um aplicativo à fonte de dados, usando a URL específica do banco de dados. No momento em que essa classe tenta fazer o primeiro contato, ela carrega automaticamente quaisquer drivers JDBC 4.0 encontrados no *classpath*. Lembrando que seu aplicativo tem que carregar manualmente quaisquer drivers JDBC anteriores à versão 4.0.

### Utilizando a classe DriverManager
A conexão com o seu SGBD usando a classe *DriverManager* envolve a chamada do método *DriverManager.getConnection()*. O seguinte método estabelece uma conexão com o banco de dados:

```java
public Connection getConnection() throws SQLException {

    Connection conn = null;
    Properties connectionProps = new Properties();
    connectionProps.put("user", this.userName);
    connectionProps.put("password", this.password);

    if (this.dbms.equals("mysql")) {
        conn = DriverManager.getConnection(
                   "jdbc:" + this.dbms + "://" +
                   this.serverName +
                   ":" + this.portNumber + "/",
                   connectionProps);
    }
    System.out.println("Conectado ao banco de dados");
    return conn;
}
```

Utilizando o método *DriverManager.getConnection()* ele estabelece uma conexão com o banco de dados. Ele precisa da URL do banco de dados, que varia dependendo do seu SGBD.

#### Observações
- Normalmente a URL do banco de dados utiliza o nome específico do banco ao qual você deseja conectar. Por exemplo, `jdbc:mysql://localhost:3306/mysql` representa a URL do banco de dados MySQL chamado *mysql*.
- Em versões anteriores do JDBC, era necessário inicializar o driver JDBC chamando o método *Class.forName()*, que exige um objeto do tipo *java.sql.Driver*.

---

## Configuração de conexão (db.properties)
O arquivo *db.properties* serve para guardar as informações de conexão com o banco de dados separadas do código Java. Isso evita deixar dados sensíveis como senha diretamente no código, facilitando manutenção e organização.

Exemplo:
```properties
db.url=jdbc:mysql://localhost:3306/teste
db.user=root
db.password=
```

---

## ConnectionFactory
A *ConnectionFactory* é uma classe responsável por centralizar a criação de conexões com o banco. Ao invés de abrir conexão em vários lugares do código, você tem um único ponto que faz isso, lendo as informações do *db.properties*.

Exemplo:
```java
public class ConnectionFactory {
    public static Connection getConnection() {
        try {
            return DriverManager.getConnection(
                "jdbc:mysql://localhost:3306/teste",
                "root",
                ""
            );
        } catch (SQLException e) {
            throw new DbException(e.getMessage());
        }
    }
}
```

---

## Statement vs PreparedStatement
Quando já há uma conexão com o banco de dados, as interfaces JDBC *Statement*, *PreparedStatement* e *CallableStatement* permitem enviar comandos SQL ou PL/SQL e receber dados do banco de dados. Elas também definem métodos que ajudam a converter diferenças entre tipos de dados Java e SQL.

### O que é Statement
É uma interface simples que permite executar comandos SQL fixos, sem parâmetros dinâmicos, como um *SELECT*, *INSERT*, *UPDATE* ou *DELETE* simples.

Exemplo:
```java
Statement stmt = conn.createStatement();
stmt.executeQuery("SELECT * FROM usuarios");
```

### O que é PreparedStatement
O *PreparedStatement* é uma subinterface de *Statement* que trabalha com **SQL parametrizado** (usando `?`). Ele "prepara" uma consulta no banco, permitindo reutilizá-la várias vezes com diferentes valores, deixando o código mais rápido e seguro contra SQL Injection.

Todos os parâmetros JDBC são representados pelo símbolo `?`, conhecido como marcador de parâmetro. Você deve fornecer valores para cada parâmetro antes de executar as instruções SQL.

Os métodos **setXXX()** vinculam valores aos parâmetros, onde **XXX** representa o tipo de dado Java. Caso você esqueça de fornecer os valores, receberá uma *SQLException*.

Cada marcador de parâmetro é referido por sua posição ordinal. O primeiro marcador representa a posição 1, o segundo posição 2, e assim por diante. Esse método difere dos índices de array Java, que começam em 0.

Todos os métodos do objeto *Statement* para interagir com o banco de dados — *execute()*, *executeQuery()* e *executeUpdate()* — também funcionam com o objeto *PreparedStatement*. No entanto, os métodos são modificados para usar instruções SQL que podem receber parâmetros.

Exemplo:
```java
PreparedStatement ps = conn.prepareStatement(
    "INSERT INTO usuarios (nome, idade) VALUES (?, ?)"
);
ps.setString(1, "Pedro");
ps.setInt(2, 20);
ps.executeUpdate();
```

---

## executeUpdate() vs executeQuery()
O *executeQuery()* é usado com *SELECT* e retorna um *ResultSet* com os dados buscados. Já o *executeUpdate()* é usado com *INSERT*, *UPDATE* ou *DELETE* e retorna um *int* com o número de linhas afetadas.

**executeQuery:**
```java
ResultSet rs = stmt.executeQuery("SELECT * FROM usuarios");
```

**executeUpdate:**
```java
int linhasAfetadas = stmt.executeUpdate("INSERT INTO usuarios VALUES (1, 'João')");
```

---

## ResultSet
Um objeto *ResultSet* é uma tabela de dados que representa um conjunto de resultados de banco de dados, gerado pela execução de uma instrução que consulta o banco. Você pode acessá-lo através do cursor. Esse cursor é um ponteiro que aponta para uma linha de dados do *ResultSet*. Inicialmente, o cursor é posicionado antes da primeira linha.

O cursor funciona da seguinte forma: o método *next()* move o cursor para a próxima linha. Esse método retorna *false* se o cursor estiver posicionado após a última linha.

A interface *ResultSet* usa métodos *getter* para recuperar os valores das colunas da linha atual, como *getString()* e *getLong()*. Você pode recuperar os valores utilizando o índice numérico da coluna ou o nome da coluna.

Exemplo:
```java
ResultSet rs = stmt.executeQuery(query);
while (rs.next()) {
    String nome = rs.getString("COF_NAME");
    int id = rs.getInt("SUP_ID");
    float preco = rs.getFloat("PRICE");
    System.out.println(nome + ", " + id + ", " + preco);
}
```

---

## SQL Injection e PreparedStatement como proteção
SQL Injection é um tipo de ataque que se inicia quando são injetadas instruções SQL maliciosas nas consultas para tentar retirar informações sensíveis do banco de dados. Ele aproveita dados incorretamente sanitizados para inserir características extras e, no final, consegue burlar a autenticação ou obter acesso a dados.

Um exemplo de ataque é o de `1=1`, onde sempre retorna verdadeiro e `LIMIT 1` retorna apenas 1 linha. O hacker fará login com o primeiro *user_id* da tabela *MsUser*, não importa qual seja o nome de usuário.

```java
Statement statement = conn.createStatement();
statement.execute("SELECT * FROM MsUser WHERE username='" + username + "'" + " AND password = '" + password + "'");
```

`SELECT * FROM MsUser WHERE username='aaa' or 1=1 limit 1`

### Como PreparedStatement previne
Para prevenir o SQL Injection você precisa executar a consulta em instruções preparadas. Quando utilizamos *PreparedStatement*, não concatenamos a entrada do usuário na instrução SQL, pois isso força o desenvolvedor a primeiro definir todas as consultas SQL e depois passar cada parâmetro separadamente. Resumindo, nada dentro da entrada do usuário é tratado como instrução SQL.

O *PreparedStatement* é compilado uma única vez e fica salvo no cache. Quando os dados do usuário chegam, os *placeholders* são substituídos sem recompilar a *query*. Por isso qualquer SQL que o usuário digitar é tratado como dado puro e não como instrução SQL.

```java
String login_query = "SELECT * FROM MsUser WHERE username = ? AND password = ?";
PreparedStatement query = conn.prepareStatement(login_query);
query.setString(1, username);
query.setString(2, password);
ResultSet result = query.executeQuery();
```

`SELECT * FROM MsUser WHERE username = "aaa' or 1=1 limit 1 --" AND password = "aaa"`

### Passos para usar o PreparedStatement
- Declare a variável SQL com `?` como *placeholder*.
- Crie o *PreparedStatement* com *conn.prepareStatement(query)*.
- Use os métodos *setter* para preencher os *placeholders*.
- Execute com *executeQuery()*.
- Processe o resultado normalmente.
- Feche o *PreparedStatement* e o *ResultSet*.

### Benefícios extras do PreparedStatement
Caso você precise executar uma consulta 10 ou 100 vezes, ela não vai passar por todas as fases novamente. Ela pode rodar rapidamente pois simplesmente substitui os marcadores de posição na consulta pré-compilada. Esse recurso oferece melhor desempenho.

---

## try-with-resources
É um recurso do Java (desde o Java 7) que serve para fechar automaticamente recursos após o uso e evitar ter que usar *finally* manualmente. Seus recursos comuns no JDBC são *Connection*, *PreparedStatement* e *ResultSet*.

### Como funciona o fechamento automático
Qualquer objeto que implementa **AutoCloseable** pode ser usado. Quando o bloco *try* termina, o Java chama automaticamente *.close()*.

**Ordem de fechamento:**
- *ResultSet*
- *Statement* / *PreparedStatement*
- *Connection*

### Como usar no JDBC

```java
try (Connection conn = DriverManager.getConnection(url, user, pass);
     PreparedStatement ps = conn.prepareStatement("SELECT * FROM usuarios");
     ResultSet rs = ps.executeQuery()) {

    while (rs.next()) {
        System.out.println(rs.getString("nome"));
    }

} catch (SQLException e) {
    e.printStackTrace();
}
```

> **Você não precisa fazer:**
> ```java
> rs.close();
> ps.close();
> conn.close();
> ```

Caso não feche, haverá vazamento de conexão. As conexões do banco ficarão abertas e o banco tem limite de conexões simultâneas. Pode acontecer:
- Erro **"Too many connections"**
- Sistema parar de responder
- Queda de performance

Isso é chamado de **Connection Leak** *(Vazamento de conexão)*.

---

## SQLException
O *SQLException* é uma exceção do Java usada para **erros relacionados ao banco de dados**.

Quando ela é lançada:
- Erro de conexão com o banco.
- SQL inválido.
- Tabela não existe.
- Tipo de dado errado.
- Falha na execução de *query*.

### Informações que ela carrega

```java
catch (SQLException e) {
    System.out.println(e.getMessage());    // mensagem do erro
    System.out.println(e.getSQLState());   // código padrão SQL
    System.out.println(e.getErrorCode());  // código específico do banco
}
```

- *getMessage()* → descrição do erro.
- *getSQLState()* → padrão internacional.
- *getErrorCode()* → código específico do MySQL.

### Como tratar (try-catch)

```java
try {
    Connection conn = DriverManager.getConnection(url, user, pass);
} catch (SQLException e) {
    System.out.println("Erro ao conectar: " + e.getMessage());
}
```

---

## DbException (exceção customizada)
O *DbException* é uma classe criada por você. Ela serve para padronizar erros do banco de dados no seu projeto, além de substituir o uso direto de *SQLException* (pois ele deixa o código poluído, dificulta a padronização de mensagens e deixa o código muito dependente do JDBC). A solução é criar o *DbException*, que geralmente estende *RuntimeException*, facilitando o uso porque não precisa de *throws*.

### O que é RuntimeException
O *RuntimeException* é uma **exceção não obrigatória** *(unchecked)*, ou seja, você não precisa usar *throws*.

```java
public class DbException extends RuntimeException {
    public DbException(String msg) {
        super(msg);
    }
}
```

### Exemplo de código SEM DbException
```java
public void inserir() throws SQLException {
    PreparedStatement ps = conn.prepareStatement("...");
}
```

### Exemplo de código COM DbException
```java
try {
    PreparedStatement ps = conn.prepareStatement("...");
} catch (SQLException e) {
    throw new DbException(e.getMessage());
}
```

---

## Transações
Uma transação é um conjunto de operações que devem ser executadas juntas. Se uma falhar, todas as outras são desfeitas. Isso garante que o banco nunca fique com dados pela metade.

Um exemplo prático é inserir um pedido e os itens desse pedido ao mesmo tempo. Se a inserção dos itens falhar, o pedido também precisa ser desfeito.

Por padrão o JDBC confirma cada operação automaticamente. Para controlar isso manualmente você usa:
- *setAutoCommit(false)* → desativa a confirmação automática.
- *commit()* → confirma todas as operações.
- *rollback()* → desfaz tudo caso ocorra erro.

### Exemplo:
```java
try {
    conn.setAutoCommit(false);
    // operações aqui
    conn.commit();
} catch (SQLException e) {
    conn.rollback();
    throw new DbException(e.getMessage());
}
```

---
## Mapa conceitual

```
projeto-dao-base/
├─ pom.xml
├─ README.md
└─ src/
   ├─ main/
   │  ├─ java/
   │  │  └─ br/escola/dao_base/
   │  │     ├─ app/
   │  │     │  └─ Main.java
   │  │     ├─ model/
   │  │     │  └─ (entidades do tema entram depois)
   │  │     ├─ dao/
   │  │     │  └─ (interfaces entram depois)
   │  │     ├─ dao/impl/
   │  │     │  └─ (implementações JDBC entram depois)
   │  │     └─ db/
   │  │        ├─ ConnectionFactory.java
   │  │        └─ DbException.java
   │  └─ resources/
   │     ├─ db.properties
   │     └─ sql/
   │        ├─ schema.sql
   │        └─ seed.sql
   └─ test/
      └─ java/ (opcional)
```

# Fontes utilizadas

**Fontes DAO:**
- https://www.dio.me/articles/o-que-e-dao-ba9c73921265

**Fontes Conceitos Básicos de Banco:**
- https://dev.mysql.com/doc/refman/8.0/en/data-types.html
- https://dev.mysql.com/doc/refman/8.0/en/constraint-primary-key.html

**Fontes JDBC:**
- https://www.devmedia.com.br/jdbc-tutorial/6638#1

**Fontes DriverManager e getConnection:**
- https://docs.oracle.com/javase/tutorial/jdbc/basics/connecting.html
- https://dev.mysql.com/doc/connector-j/en/connector-j-usagenotes-connect-drivermanager.html

**Fontes Configuração de conexão (db.properties):**
- https://docs.oracle.com/javase/tutorial/jdbc/basics/connecting.html
- https://www.baeldung.com/java-jdbc-connection-factory

**Fontes ConnectionFactory:**
- https://docs.oracle.com/javase/tutorial/jdbc/basics/connecting.html
- https://www.baeldung.com/java-jdbc-connection-factory

**Fontes Statement vs PreparedStatement:**
- https://www.tutorialspoint.com/jdbc/jdbc-statements.htm
- https://www.guj.com.br/t/diferenca-entre-preparecall-e-preparestatement/284216/
- https://www.ramon.pro.br/statement-vs-preparedstatement-quais-as-diferencas/

**Fontes executeUpdate() vs executeQuery():**
- https://docs.oracle.com/javase/tutorial/jdbc/basics/prepared.html
- https://www.tutorialspoint.com/jdbc/jdbc-statements.htm

**Fontes ResultSet:**
- https://docs.oracle.com/javase/tutorial/jdbc/basics/retrieving.html

**Fontes SQL Injection e PreparedStatement como proteção:**
- https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html
- https://medium.com/swlh/preventing-sql-injection-attack-with-java-prepared-statement-259611281e4d
- https://javabypatel.blogspot.com/2015/09/how-prepared-statement-in-java-prevents-sql-injection.html

**Fontes try-with-resources:**
- https://www.tutorialspoint.com/java/java_try_with_resources.htm

**Fontes SQLException:**
- https://docs.oracle.com/javase/tutorial/jdbc/basics/sqlexception.html
- https://www.tutorialspoint.com/jdbc/jdbc-exceptions.htm

**Fontes DbException (Exceção Customizada):**
- https://www.tutorialspoint.com/jdbc/jdbc-exceptions.htm
- https://www.baeldung.com/java-new-custom-exception

**Fontes Transações:**
- https://docs.oracle.com/javase/tutorial/jdbc/basics/transactions.html
- https://www.tutorialspoint.com/jdbc/jdbc-transactions.htm
  
## 👨‍💻 Desenvolvedor

<i>Projeto desenvolvido por estudantes do 2º ano de Desenvolvimento de Sistemas - ETEC</i>

</div>
<br>

<table align="center">
  <tr>
    <!-- Lucas -->
    <td align="center">
      <a href="https://github.com/Lucxzhhh">
        <img src="https://github.com/user-attachments/assets/b0f08778-8ee4-4370-a426-2c43b95f3b1a" width="140"><br>
        <b>Lucxzhhh</b>
      </a>
    </td>
  </tr>
</table>

<br>
