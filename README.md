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
