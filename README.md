# Estudo de Mysql-DAO-JDBC

# O que é o DAO (Data Access Object).
DAO (Data Access Object), é um padrão de de projeto (design pattern), onde ocorre uma divisão da logica de acessos a dados da logia de negocio em uma aplicação. Ele encapsula todo as interações com o banco de dados, fazendo como que a logica de negocio não precise lidar diretamento com os comandos de SQL.

Em um projeto organizado em DAO, a classe DAO é a resposável por realizar as operações CRUD (Create, Read, Update, Delete) sobre um entidade do projeto, obtendo uma camada de abstração enstre o banco de dados e a lógica da aplicação.


## Quais problemas o DAO resolve ?
O padrão DAO surgiu na programação orientada a objetos como uma forma de solução para reduzir o acoplamento entre camda de negocio e a camda de persistencia. 

#### A introdução do DAO resolveu esse problemas :
- Criar uma camada intermediária entre o banco de dados e a lógica de negócio;
  
- Permitir a reutilização de código para operações comuns no banco de dados;
Facilitar a troca de fornecedores de banco de dados sem grandes impactos na lógica da aplicação;

- Melhorar a testabilidade, pois permite substituir DAOs reais por implementações simuladas (mock) durante os testes.

## Arquitetura das camadas 
É uma forma de organizar o codifo separando responsavilidades. CAda camada tem um função especifica e não preisa saber como as outras funcionam por dentro.

  ### Model
- Contém as entidades do sistemas (Objectos que representam dados)
- Exemplos:  classe CLIENTE, PRODUTO, PEDIDO.
- São classes simples com atribustos, getters (métodos de acesso) e setters(médotos de modificação).
- Funciona como um container de dados, sem lógica de negócio.
  
  ### dao
- contém as interfaces (contratos).
- Define os métodos qye existe, mas não como eles funcionam.
- A interface define os métodos CRUD (Create, Read, Update, Delete) e mantém o código modular, permitindo trocar impremnteções sem mudar a lógica de negócio.
- Exemplo: ClienteDao com métodos inserir(), buscarPorID(), listarTodos(), atualizar(), deletar()

  ### dao.impl
- Contém as implementações das interfaces.
- É aqui que fica o SQL de verdade, o JDBC, o PreparedStatement.
- É onde ficam as imprementações concretas dis DAOs usando JDBC para interagir com o banco MySQL.
- Exemplo: ClienteDaoJDBC que implementa CLienteDao.

  ### db
- contém a infrestrutura de conexão.
- Inclui a classe utilitária de conexão com o banco, a exceção customuzada DbExcption, e a DaoFactory para instalar os DAOs.

  ### app
- contém o ponto de entrada do sistema.
- É o main.java com o menu e a orquestração das ações do usuário.
- Chamada os DAos através das interfaces, sem saber nada do SQL.

## Por que separar assim?
Com esse tipo de separação permite que as duas partes evoluam de forma independente, se a lógica de negócio mudar, ela continua dependendo da interface DAO. Se a lógica de persistÊncia mudar, os clientes DAO não vão ser afetados.

# JDBC como Funciona (a conexão java com banco de dados).
JDBC é semelhante ao ODBC,e no principio utilizava o ODBC para conectar-se com o banco de dados. A partir de um codigo nativo as aplicações de java podiam usar qualquer banco de dados que tivesse um driver ODBC dísponivel. Desta forma ajudou muito a popularizar o JDBC uma vez que existe ym driverOBDC para praticamente qualquer banco de dados de mercado.

Assim como ODBC, JDBC também funciona através de drivers que são responsáveis pela conexão com o banco e execução das intruções SQL. Esse drivers foran divididos em quatro tipos.

<img width="642" height="740" alt="image" src="https://github.com/user-attachments/assets/c0900ef1-8ee6-4dff-b199-718b7b898e15" />

Esses drivers são implementações das interfaces do pacote java.sql. Geralmente eles estão disponibilizados em arquivos JAR (java ARchive) pelos frabricantes do banco de dados ou terceiros. Você pode encontrar esses drivers JDBC em m www.oracle.com, www.dev.mysql.com, www.microsoft.com, www.ibm.com. Ou você pode consultar a base de dados de drivers certificados da Sun em http://developers.sun.com/product/jdbc/drivers.

Após fazer a download do driver, basta incluir ao **CLASSPATH**. A partir desse momento está tudo pronto para você acessar o banco de dados via java.

**Nota :** Na maoria dos IDE's ignoram a variavel ambiente **CLASSPATH** (utilizam uma forma própria de gerenciar as classes usadas pelo projeto), caso isso ocorra você deve adicionar o driver no projeto.

# DriverManager e getConnection
DriberManager é uma classe que totalmente implementada onde ela conecta um aplicativo a fonte de dados, usando a url especifica de banco de dados. No momento em que essa classe tenta fazer o primeiro contado ela carrega automaticamente quaisquer drivers JDBC 4.0 encontrados no classpath. Lembrando que seu aplicativo tem que carregar manualmente quaisquer drivers JDBC anteriores à versão 4.0

## Utilizando a classe DriverManager
A conexão com o seu SGBD usando o _DriverManager_ classe envolve a chamada do método _DriverManager.getConeection._ O seguinte método JDBCTutorialUtilities.getConnection, estabelece uma conexão com o banco de dados:

````
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
````

Utilizando esse método  _DriverManager.getConeection_ ele estabelece uma conexão com o banco de dados. Ele precisa da url do banco de dados, varia dependendo do seu SGBD. Seguem os exemplo abaixo :

#### Java DB :
onde é o nome do banco de dados ao qual se conectar e instrui o SGBD a criar o banco de dados _jdbc:derby:testdb;create=truetestdbcreate=true_

##### Observações 
- Normalmente, a url do banco de dados, se utiliza o nome especifico do banco de dados existente ao qual você deseja conctar. Por exemplo, URL jdbc:mysql://localhost:3306/mysql onde ela representa o URL do banco de dados MySQL chamado mysql. Esse exemplo usam a URL que não especifica um banco de dados especifico porque os exemplos criam um novo banco de dados.

- Em versoões anteriores do JDBC para conseguir se conectar, era necessario se inicializar o driver JDBC que precisa ser chamadi o método 'initialize' _Class.forName_, e esse método exige um objeto do tipo 'Connection' _java.sql.Driver_

## Statement vs PreparedStatement 
Quando já à uma conexão com o banco de dados. A interfaces JDBC Statement, PreparedStatement e CallableStatement, eles permitem enviar os comandos de SQL ou PL/SQL, e receber dados do seu banco de dados. Eles também difine métodos que ajuda a conectar diferenças entre tipos de dados Java e SQL. 

#### O que é Statement
É uma interface simples que permite executar comandos SQL "fixo", sem parâmetros dinamicos, como um _SELECT, INSERT UPDATE ou DELETE simples. Exemplo :

``
Statement stmt = conn.createStatement();
stmt.executeQuery("SELECT * FROM usuarios");
``
#### O que é PreparedStatement
O PreparedStatement é uma subinterface de _Statement_ que trabalha com **SQL parametrizado** (usando ?).Ele "prepara" uma consulta no seu banco, uma vez que o seu banco permite reutiliza-lo varias vezes com diferentes valores, isso acaba deixando mais rápido e seguro contra o SQL injection.
Todos parametros JBDC são representados pelo simbolo **?**, que é conhecido como marcador de parametros. Onde você deve fornecer valores para cada parametro antes de executar instruções SQL.

Os métodos **setXXX()** vinculam aos parametros, onde **XXX** representa o tipo de dado Java do valor que você deja vincular ao parametro de entrada. Caso você esquecer de fornecer os valores, receberá uma SQLException.

Cada marcador de parametro é referido por sua posição ordinal. O primeiro marcador representa a posição 1, e a próxima posição 2, e assim por diante. esse método difere do dos indices de arry Java, que começam em 0.

Todos oss métodos **do objeto Statement** para interagir com o banco de daodos (a) execute (), (b) escuteQuery () e (c) executeUpdate () também funcionam com o objeto PreparedStatement. No entanto, os métodos são modificados para usar instruções SQL que podem inseriri os parametros.
Exemplo :
``
PreparedStatement ps = conn.prepareStatement(
    "INSERT INTO usuarios (nome, idade) VALUES (?, ?)"
);
ps.setString(1, "pedro");
ps.setInt(2, 20);
ps.executeUpdate();
``


### Objetos da Afirmação (criando o objeto de Instrução)
Para usar o obejto Statement para executar uma instrução SQL, é necessesario  criar um método _createStatement( )_ do objeto Connection. Exemplo abaixo :

``
Statement stmt = null;
try {
   stmt = conn.createStatement( );
   . . .
}
catch (SQLException e) {
   . . .
}
finally {
   . . .
}
``

##  executeUpdate() vs executeQuery()
O executeQuery() usado com SELECT, retorna um ResultSet com os dados buscados, já o executeUpdate ele é usado com ISERT, UPDATE ou DELETE, retorna um int com o número de linhas afetaddas. Exemplos: 

**executeQuery**
``executeQuery → SELECT
ResultSet rs = stmt.executeQuery("SELECT * FROM usuarios");``

**executeUpdate**
`` INSERT
int linhasAfetadas = stmt.executeUpdate("INSERT INTO usuarios VALUES (1, 'João')");``

## ResultSet
Um objeto _ResultSet_ é uma tabela de dados que geralmente é conjunto de resultados (de uma banco de dados) de execuções de uma instrução que consulta o banco de dados. Você pode acessar ele atraves do cursor. Esse cursor é o ponterio que aponta para uma linha de dados ResultSet incialmente, o cursor é posicionado antes da primeira linha.

O cursor funciona da seguinte forma: O método next() move o cursor para a proxima linha. esse método retorna false se o curso estiver posicionado após a última linha.

A interface ResultSet usa o método getter para recuperar os valores das colunas da linha atual, como getString  e getLong. Você pode recuparar os valores utilizando o índice nimerioco da coluna ou o nome da coluna.

**Exemplo:**

``
ResultSet rs = stmt.executeQuery(query);
while (rs.next()) {
    String nome = rs.getString("COF_NAME");
    int id = rs.getInt("SUP_ID");
    float preco = rs.getFloat("PRICE");
    System.out.println(nome + ", " + id + ", " + preco);
}
``
##  SQL Injection e PreparedStatement como proteção
SQL Injection é um tipo de ataque, onde ele se inicia quando intam instruções de SQL maliciosas nas consultas para tentar retirar informações sensíveis do banco de dados, eles aproveita dados incorretamente sanitizados para inserir algumas caracteristicas extras, no final eles conseguem burlar a autenticação ou obter acesso a dados.

Um exemplo de ataque é o de 1=1, onde sempre retorna o verdadeiro e limite 1 retorna apenas 1 linha o hacker fara login com o primeiro user_id tabela MsUser, não importa qual seja o nome de usuario.

````
package com.minghong;

import java.sql.*;
import java.util.Scanner;

public class Select {
    public static void main(String[] args) {
        try {
            //establishing connection and use database with the name market in the specified folder
            Connection conn = DriverManager.getConnection("jdbc:sqlite:D:\\Java Course\\SQLiteCRUD\\market.db");

            //scan user's input
            Scanner scan = new Scanner(System.in);
            System.out.println("Username : ");
            String username = scan.nextLine();

            System.out.println("Password : ");
            String password = scan.nextLine();

            //sql statement that vulnerable to sql injection
            Statement statement = conn.createStatement();
            statement.execute("SELECT * FROM MsUser WHERE username='" + username + "'" + " AND password = '" + password + "'");
            System.out.println("SELECT * FROM MsUser WHERE username='" + username + "'" + " AND password = '" + password + "'");

            //get result
            ResultSet result = statement.getResultSet();
            if (result.next()) {
                System.out.println("Successfully login as " + result.getString("username"));
            } else {
                System.out.println("Wrong username or password");
            }

            result.close();

            //close statement and connection
            statement.close();
            conn.close();

            System.out.println("Command successful");
        } catch (SQLException e) {
            //error handling & print error message
            System.out.println("Error : " + e.getMessage());
        }
    }
}
````

``SELECT * FROM MsUser WHERE username='aaa' or 1=1 limit 1``

#### Como PreparedStatement previne
Para prevenir a SQL Injection você precisa tomar algumas precauções, como execultar a consulta em instruções preparadas ("quando utilizamos elas não concatenamos a entrada do usuario na instruções SQL, que está executnado, porque isso forçará o desenvolvedor a primeiro definir todas as consultas SQL e depois passar cada parametro ou seja a entrada do usuarip, para a consulta depois"). Resumindo nada dentro da entrada do usuario é tratado como instrução SQL.

````
package com.minghong;

import java.sql.*;
import java.util.Scanner;

public class Login {

    public static void main(String[] args) {
        try {
            //establishing connection and use database with the name market in the specified folder
            Connection conn = DriverManager.getConnection("jdbc:sqlite:D:\\Java Course\\SQLiteCRUD\\market.db");

            //scan user's input
            Scanner scan = new Scanner(System.in);
            System.out.println("Username : ");
            String username = scan.nextLine();

            System.out.println("Password : ");
            String password = scan.nextLine();

            //declare a variable for the sql statement
            String login_query = "SELECT * FROM MsUser WHERE username = ? AND password = ?";

            //we call conn.prepareStatement method to create the instance of prepared statement
            PreparedStatement query = conn.prepareStatement(login_query);

            //pass user's input to the prepared statement query
            query.setString(1,username);
            query.setString(2,password);

            //execute query
            ResultSet result = query.executeQuery();
            System.out.println(result.getStatement().toString());
            //close prepared statement
            query.close();

            //check result
            if (result.next()) {
                System.out.println("Successfully login as " + result.getString("username"));
            } else {
                System.out.println("Wrong username or password");
            }

            result.close();

            //close connection
            conn.close();
        } catch (SQLException e) {
            //error handling & print error message
            System.out.println("Error : " + e.getMessage());
        }
    }
}
````

``SELECT * FROM MsUser WHERE username = "aaa' or 1=1 limit 1 --" AND password = "aaa"``

#### Passo para usar o PreparedStatement
- Declare a variavel SQL com como placeholder **?**.
- Crie o PreparedStatement com conn.prepareStatement(query).
- Use os métodos setter para preencher os placeholders
- Executte com 

# Fontes utilizadas

fontes JDBC: https://www.devmedia.com.br/jdbc-tutorial/6638#1. 

fontes DriverManager e getConnection :
- https://docs.oracle.com/javase/tutorial/jdbc/basics/connecting.html.
- https://dev.mysql.com/doc/connector-j/en/connector-j-usagenotes-connect-drivermanager.html.

Fontes Statement vs PreparedStatement  : 
- https://www.tutorialspoint.com/jdbc/jdbc-statements.htm
- https://www.guj.com.br/t/diferenca-entre-preparecall-e-preparestatement/284216/
- https://www.ramon.pro.br/statement-vs-preparedstatement-quais-as-diferencas/

Fontes executeUpdate() vs executeQuery():
- https://docs.oracle.com/javase/tutorial/jdbc/basics/prepared.html
- https://www.tutorialspoint.com/jdbc/jdbc-statements.htm

Fontes ResultSet :
- https://docs.oracle.com/javase/tutorial/jdbc/basics/retrieving.html

