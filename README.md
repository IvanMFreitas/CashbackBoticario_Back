# Cashback Solution - Grupo Boticário

![](https://i.ibb.co/Ydf2JTM/GBLogo.jpg)

# 0. [TL;DR]

* Abra um Powershell com diretiva de administrador local :information_desk_person: (Nunca se sabe né?) :see_no_evil: :hear_no_evil: :speak_no_evil:
* Acesse o diretório onde está a Solução/GBCashback.API
* Execute o comando "dotnet run", e pronto. Sua API já está sendo executada no endereço: https://localhost:5001
* Caso deseje ver a documentação, acessar: https://localhost:5001/swagger

* ***Código disponibilizado apenas para os avaliadores***

![](https://i.ibb.co/TH1rfCy/5h-Wp-FDj8s8.gif)

# 1. Introdução

Esse repositório contém a solução desenvolvida para atender a demanda do teste de Desenvolvimento proposto pelo Grupo Boticário.
A demanda refere-se ao desenvolvimento de uma solução para Calcular o Cashback de cada venda realizada por um revendedor.
As premissas desses teste estão no documento enviado, ao iniciar o processo seletivo.

# 2. Pré Requisitos

Para a aplicação ser executada, o ambiente deve conter a versão .Net Core 3.1x ou superior, que pode ser baixada em:
https://dotnet.microsoft.com/download/dotnet-core/3.1


# 3. Arquitetura

A arquitetura usada na solução, assemelha-se muito com a Arquitetura MVC, onde cada projeto tem sua responsabilidade.

![](https://i.ibb.co/5njDw7F/Solution.png)

As responsabilidades dos projetos são descritas a seguir:

* **GBCashback.API:** A API própriamente dita, com settings, acesso ao banco de dados e injeção de dependências que serão utilizadas nos outros projetos;

* **GBCashback.Controllers:** Para facilitar o desenvolvimento, futuras implementações e melhoras, e supote, as controllers são utilizadas a partir de outro projeto. Essas, consomem injeções realizadas nos services da API;

* **GBCashback.Data:** O projeto Data contém toda a parte de comunicação com o Banco de dados, utilizando-se do EntityFramework Core para SQLite. Também inclui a classe DataContext, que é responsável por mapear as tabelas do DB;

* **GBCashback.Helpers:** Classes helpers são utilizadas nesse projeto para formatação de string, data, e Criptografia de Password (utilizando Salted Hash);

* **GBCashback.Models:** Nesse projeto estão todas as Models e Enums que serão acessadas pelas Controllers, Services e também banco de dados;

* **GBCashback.Services:** E finalmente, o projeto Services armazena as classes de Services, que contém toda a regra de negócio envolvida na API e consumo de Endpoints;

# 4. Banco de dados

O banco de dados utilizado foi o SQLite devido a sua "simplicidade" e ao mesmo tempo, sua "robustes" e rapidez no desenvolvimento.

SQLite utiliza uma tecnologia de relacionamento entre entidades semelhante a outros bancos mais parrudos, como SQL Server, Oracle, Postgres entre outros. No entanto ele é gravado diretamente no diretório da aplicação, o que facilita a abertura dele em ferramentas externas e o torna mais leve.

A ferramenta que o Deev. pode utilizar é o "DB Browser", que pode ser baixado no: https://sqlitebrowser.org/. A ferramenta é free para uso pessoal.

O diagrama da Database encontra-se abaixo, apenas para consulta. 

![](https://i.ibb.co/37NwnSG/databse.png)

# 5. Segurança

A proposta utilizada para segurança é a utilização de Autenticação usando JWT Tokens.
**JSon Web Token** pode trafegar informações de usuário, as Roles que ele tem acesso entre outras.
Também cria uma espécie de "assinatura" entre Client e Server, não permitindo o acesso de terceiros sem que o mesmo esteja logado.

Quanto ao login, também para segurança de senha, foi utilizado a classe CryptoHelpers. Essa classe contém todos os métodos herdados da classe "System.Web.Helpers", no entanto essa classe é desenvolvida para .Net Framework.
Por isso a criação de uma classe "CryptoHelpers" com os métodos necessários, porém portados para .Net Core.
A classe original pode ser encontrada em: https://github.com/aspnet/AspNetWebStack/blob/master/src/System.Web.Helpers/Crypto.cs

# 6. Endpoints

Como toda a API que segue boas práticas, toda a documentação de Endpoints pode ser acessada em:
https://localhost:5001/swagger.

No entanto, deixamos, no link a seguir, o arquivo POSTMAN com a definição de endpoints.

Documentação da API: https://documenter.getpostman.com/view/8428011/T1LPD7Kh?version=latest

Para automatizar a Autenticação nos testes, criar como "Environment Variables" no Postman, as variáveis "token" e "currentToken".
Dessa maneira, ao clicar no endpoint de "Login", o token gerado já é adicionado a variáveis de Environment e assim os outros métodos já são autorizados com o token.

*Passo 1*

![](https://i.ibb.co/9TzyJLV/Environment1.png)

*Passo 2*

![](https://i.ibb.co/r586gM8/Environment2.png)

**Lista de ENDPOINTS**

![](https://i.ibb.co/SJYpRxV/Endpoints.png)


***Dados de Login para Usuário Administrador***

*CPF:* ***153.509.460-56***

*Senha:* ***12345***

Também abaixo segue a descrição de cada Endpoint.

## 6.1. (POST) Revendedor/CriarRevendedor

Esse Endpoint é encarregado de criar um Revendedor (Usuário) na base de dados.
O Endpoint espera um objeto Json simples, que contém as seguintes propriedades:

```json
{
	"Nome": "Ivan Freitas",
	"CPF": "123.456.789-00",
	"Senha": "12345"
}
```
E o retorno desse Endpoint deve retornar o usuário criado com sucesso, ou, no caso de criar um usuário com CPF Existente, o sistema avisa que o CPF já existe.

## 6.2. (POST) Login

O Endpoint responsável por fazer o login do usuário na aplicação.
Recebe um objeto JSon:

```json
{
    "CPF" : "123.456.789-00",
    "Senha": "12345"
}
```

E irá retornar um objeto contendo um jwtToken, que será utilizado para autenticar futuros consumos dos Endpoints.

## 6.3. (POST)Compra/CriarCompra

Endpoint responsável por Criar (cadastrar) uma compra no sistema.
Já no ato do cadastro, o cashback é calculado de acordo com o total de vendas cadastradas para o usuário no mês.
Recebe:

```json
{
    "Codigo": 1240,
    "Valor": 50.00,
    "DtVenda": "2020-08-13T20:40",
    "CPF": "1345678900"
}
```
E retorna o objeto de compra preenchido, já com o Cashback calculado.

## 6.4. (GET) Compra/RetornaComprasRealizadas

Endpoint que retornará todas as compras realizadas pelo usuário logado, sem nenhum filtro aplicado.
No caso do usuário logado ser um Administrador, esse endpoint retornará todas as compras contidas no sistema.
Por ser um método GET, não possúi Objeto Json enviado no body.

## 6.5. (GET) Compra/RetornaComprasRealizadasPorCPF

Endpoint que retornará todas as compras realizadas pelo usuário logado, com CPF como filtro aplicado.
Esse Endpoint **só pode ser acessado por um Administrador do Sistema.**
Por ser um método GET, não possúi Objeto Json enviado no body, no entanto, deve receber 1 parâmetro:
```json
"CPF" = "12345678900" 
```

## 6.6. (GET) Compra/RetornaComprasPorPeriodo

Endpoint que retornará todas as compras realizadas pelo usuário logado, com o filtro de Data inicial e final aplicado.
No caso do usuário logado ser um Administrador, esse endpoint retornará todas as compras contidas no sistema com o filtro descrito acima.
Por ser um método GET, não possúi Objeto Json enviado no body, no entanto, deve receber 2 parâmetros:
"dtInicial" = "11/08/2020"
"dtFinal" = "12/08/2020"

## 6.7. (GET) Compra/RetornaComprasPorPeriodoCpf

Endpoint que retornará todas as compras realizadas, com o filtro de Data inicial e final e também CPF aplicado.
Esse Endpoint **só pode ser acessado por um Administrador do Sistema.**
Por ser um método GET, não possúi Objeto Json enviado no body, no entanto, deve receber 3 parâmetros:
```json
"dtInicial" = "11/08/2020"
"dtFinal" = "12/08/2020"
"cpf" = "12345678900"
```

## 6.8. (DELETE) Compra/RemoveCompra

Endpoint responsável por excluir uma compra da Base de dados, bem como seu Cashback calculado.
Esse Endpoint **só pode ser acessado por um Administrador do Sistema.**
Por ser um método DELETE, não possúi Objeto Json enviado no body, no entanto, deve receber 1 parâmetro:
```json
"numCompra" = "1236"
```

## 6.9. (PUT) Compra/EditarStatusDeCompra

Endpoint responsável por alterar o Status de uma compra (Aprovada ou Em análise)
Esse Endpoint **só pode ser acessado por um Administrador do Sistema.**
Por ser um método PUT, não possúi Objeto Json enviado no body, no entanto, deve receber 2 parâmetros:
```json
"numCompra" = "1236"
"status" = "0" (para "Validação") ou "1" (para "Aprovado")
```

## 6.10. (GET) Cashback/CashbackAcumulado

Endpoint que consulta a API Externa disponibilizada pelo Grupo Boticário, para o teste, e traz o Cashback Acumulado de acordo com o CPF logado.
Endpoint vai retornar o total de Cashback Acumulado.

## 6.11. (GET) Cashback/CashbackAcumuladoPorCPF

Endpoint que consulta a API Externa disponibilizada pelo Grupo Boticário, para o teste, e traz o Cashback Acumulado de acordo com um CPF enviado por parâmetro.
Esse Endpoint **só pode ser acessado por um Administrador do Sistema.**
Endpoint vai retornar o total de Cashback Acumulado, e deve receber como parâmetro o CPF a ser consultado:
```json
"cpf" = "12345678900"
```

# 7. Logs

O Approach utilizado para Logar informações na aplicação é a utilização da classe ILogger.

No entanto, visando manter um histórico, foi desenvolvido uma LogService, que além de imprimir o Log no console da aplicação (PRINT de LOG), a Classe LOG também é responsável por adicionar o LOG recebido no Banco de dados, na tabela LOG.

![](https://i.ibb.co/k9T8ncX/Logs.png)

# 8. Executando a API

Para executar a aplicação, basta que:
* os pré requisitos sejam atendidos;
* seja utilizado a ultima versão de commit disponível nesse GitHub;
* e seja executado o comando "dotnet run" dentro do diretório do projeto "GBCashback.API".

**A primeira execução será um pouco mais lenta**, devido a restauração de pacotes externos utilizados, e sendo restaudados automaticamente pelo Nuget Package Manager.

No entanto, as próximas consultas serão feitas em realtime (ou próximo a isso).

# 9. Testes

Foi implementado na solution um projeto de testes (GBCashback.UnitTests), para atender requisitos de testes das Services, operações de DB e algumas consultas na API Externa do Boticário.

Nesse projeto foram utilizados:

* O SDK de testes padrão da Microsoft
* XUnit
* Fluent Assertions (Esse para melhorar os Asserts de retorno)

E, para fins de estatística, o código da solution e seus projetos adjacentes agora tem 93.91% de Code Coverage (convenções padrões sugerem no mínimo 80% de Code Coverage).

![](https://i.ibb.co/dDCGMbd/code-Coverage.png)

*As Controllers não foram testadas, pois a ideia é implementar Testes de Integração em uma versão futura, como melhoria.*

# 10. Melhorias futuras

Há algumas melhorias sugeridas na aplicação que podem ser feitas na sequência, no entanto, fica para quem quiser se aventurar nesse branch:

* Implementação de Projeto para Integration Tests;

# 11. Agradecimentos

A Deus, por todo o conhecimento e sabedoria dada até aqui.

Aos familiares e amigos que apoiaram e ajudaram em todas as horas.

E finalmente, mas sem esquecer, agradeço a oportunidade dada para realizar esse teste, pelo conhecimento adquirido no desenvolvimento (sim, vc sempre aprende algo novo).

Sendo assim, me coloco a disposição para qualquer esclarecimento que possa haver.

# 12. Contato

Ivan Freitas

Senior Developer

https://github.com/IvanMFreitas
