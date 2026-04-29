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

`asda
public Connection getConnection() throws SQLException {

    Conexão conn = nula;
    Propriedades connectionProps = nova Propriedades();
    connectionProps.put("user", this.userName);
    connectionProps.put("senha", this.senha);

    se (this.dbms.equals("mysql")) {
        conexão = DriverManager.getConnection(
                   "jdbc:" + this.dbms + "://" +
                   this.serverName +
                   ":" + this.portNumber + "/",
                   connectionProps);
    } else if (this.dbms.equals("derby")) {
        conexão = DriverManager.getConnection(
                   "jdbc:" + this.dbms + ":" +
                   this.dbName +
                   ";criar=verdadeiro",
                   connectionProps);
    }
    System.out.println("Conectado ao banco de dados");
    retornar conexão;
}
`



fontes JDBC: https://www.devmedia.com.br/jdbc-tutorial/6638#1. 

