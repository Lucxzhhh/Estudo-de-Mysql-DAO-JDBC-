# Estudo de Mysql-DAO-JDBC

## O que é o DAO (Data Access Object).
DAO (Data Access Object), é um padrão de de projeto (design pattern), onde ocorre uma divisão da logica de acessos a dados da logia de negocio em uma aplicação. Ele encapsula todo as interações com o banco de dados, fazendo como que a logica de negocio não precise lidar diretamento com os comandos de SQL.

Em um projeto organizado em DAO, a classe DAO é a resposável por realizar as operações CRUD (Create, Read, Update, Delete) sobre um entidade do projeto, obtendo uma camada de abstração enstre o banco de dados e a lógica da aplicação.


## Quais problemas o DAO resolve ?
O padrão DAO surgiu na programação orientadaa a objetos como uma forma de solução para reduzir o acoplamento entre camda de negocio e a camda de persistencia. 

A introdução do DAO resolveru esse problemas :
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
- A interface define os métodos CRUD () e mantém o código modular, permitindo trocar impremnteções sem mudar a lógica de negócio.
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

Por que separar assim?
Com esse tipo de separação permite que as duas partes evoluam de forma independente, se a lógica de negócio mudar, ela continua dependendo da interface DAO. Se a lógica de persistÊncia mudar, os clientes DAO não vão ser afetados.
