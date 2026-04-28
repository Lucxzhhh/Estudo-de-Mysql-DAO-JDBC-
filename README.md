# Estudo de Mysql-DAO-JDBC

# O que é o DAO (Data Access Object).
DAO (Data Access Object), é um padrão de de projeto (design pattern), onde ocorre uma divisão da logica de acessos a dados da logia de negocio em uma aplicação. Ele encapsula todo as interações com o banco de dados, fazendo como que a logica de negocio não precise lidar diretamento com os comandos de SQL.

Em um projeto organizado em DAO, a classe DAO é a resposável por realizar as operações CRUD (Create, Read, Update, Delete) sobre um entidade do projeto, obtendo uma camada de abstração enstre o banco de dados e a lógica da aplicação.


## Quais problemas o DAO resolve ?
O padrão DAO surgiu na programação orientada a objetos como uma forma de solução para reduzir o acoplamento entre camda de negocio e a camda de persistencia. 

A introdução do DAO resolveu esse problemas :
- Criar uma camada intermediária entre o banco de dados e a lógica de negócio;
  
- Permitir a reutilização de código para operações comuns no banco de dados;
Facilitar a troca de fornecedores de banco de dados sem grandes impactos na lógica da aplicação;

- Melhorar a testabilidade, pois permite substituir DAOs reais por implementações simuladas (mock) durante os testes.

## Arquitetura das camadas 
É uma forma de organizar o codifo separando responsavilidades. CAda camada tem um função especifica e não preisa saber como as outras funcionam por dentro.

  Model
- Contém as entidades do sistemas (Objectos que representam dados)
- Exemplos:  classe CLIENTE, PRODUTO, PEDIDO.
- São classes simples com atribustos, getters (métodos de acesso) e setters(médotos de modificação).
- Funciona como um container de dados, sem lógica de negócio.
  
  dao
- contém as interfaces (contratos).
- Define os métodos qye existe, mas não como eles funcionam.
- A interface define os métodos CRUD (Create, Read, Update, Delete) e mantém o código modular, permitindo trocar impremnteções sem mudar a lógica de negócio.
- Exemplo: ClienteDao com métodos inserir(), buscarPorID(), listarTodos(), atualizar(), deletar()

  dao.impl
- Contém as implementações das interfaces.
- É aqui que fica o SQL de verdade, o JDBC, o PreparedStatement.
- É onde ficam as imprementações concretas dis DAOs usando JDBC para interagir com o banco MySQL.
- Exemplo: ClienteDaoJDBC que implementa CLienteDao.

  db
- contém a infrestrutura de conexão.
- Inclui a classe utilitária de conexão com o banco, a exceção customuzada DbExcption, e a DaoFactory para instalar os DAOs.

  app
- contém o ponto de entrada do sistema.
- É o main.java com o menu e a orquestração das ações do usuário.
- Chamada os DAos através das interfaces, sem saber nada do SQL.

## Por que separar assim?
Com esse tipo de separação permite que as duas partes evoluam de forma independente, se a lógica de negócio mudar, ela continua dependendo da interface DAO. Se a lógica de persistÊncia mudar, os clientes DAO não vão ser afetados.

# JDBC como Funciona (a conexão java com banco de dados).
JDBC é semelhante ao ODBC,e no principio utilizava o ODBC para conectar-se com o banco de dados. A partir de um codigo nativo as aplicações de java podiam usar qualquer banco de dados que tivesse um driver ODBC dísponivel. Desta forma ajudou muito a popularizar o JDBC uma vez que existe ym driverOBDC para praticamente qualquer banco de dados de mercado.

Assim
































Padrão DAO — o que é e qual problema resolve

Arquitetura em camadas — o que vai em cada pacote (model, dao, dao.impl, db, app)

JDBC — como funciona a conexão Java com banco de dados

DriverManager — como abrir uma conexão com getConnection()

Statement vs PreparedStatement — diferença e quando usar cada um

executeUpdate() vs executeQuery() — qual usar pra cada tipo de SQL

ResultSet — como ler os dados retornados do banco

SQL Injection — o que é e como acontece

PreparedStatement como proteção — como ele previne SQL Injection

try-with-resources — como fechar conexões automaticamente

Vazamento de conexão — o que é e por que é um problema

SQLException — o que é essa exceção

DbException (exceção customizada) — por que criar uma classe própria de erro

Chaves no banco (PK, FK, NOT NULL, UNIQUE) — conceito de modelagem

Tipos de dados MySQL (INT, VARCHAR, DECIMAL, DATETIME) — quando usar cada um

Índices no banco — o que são e quando importam

Transações — o que é uma transação e por que usar

setAutoCommit, commit, rollback — como controlar transações no JDBC

db.properties — como configurar a conexão por arquivo externo

ConnectionFactory — como criar uma classe que gerencia a conexão

MySQL Connector/J — o driver que conecta Java ao MySQL

URL de conexão JDBC — como montar a string de conexão corretamente
