🍷 Adega do Pai — REST API
API REST para gerenciamento de uma adega, desenvolvida com foco em arquitetura modular, autenticação segura, controle de acesso, gerenciamento de estoque, pedidos e pagamentos.







📌 Sobre o projeto
O Adega do Pai é uma API REST desenvolvida para representar o backend de uma plataforma de vendas e gerenciamento de produtos de uma adega.

A aplicação centraliza operações como:

autenticação e gerenciamento de usuários;

controle de acesso por perfil;

cadastro e gerenciamento de produtos;

categorização de produtos;

gerenciamento de fornecedores;

controle de estoque e histórico de movimentações;

gerenciamento de endereços;

carrinho de compras;

criação e acompanhamento de pedidos;

pagamentos;

avaliações de produtos e pedidos;

notificações;

geração e armazenamento de relatórios.

A aplicação foi estruturada utilizando a arquitetura modular do NestJS, separando responsabilidades por domínio e mantendo regras de negócio isoladas em seus respectivos serviços.

🎯 Objetivos técnicos
O projeto foi desenvolvido com foco em demonstrar conhecimentos práticos de desenvolvimento backend, incluindo:

construção de APIs REST;

arquitetura modular;

TypeScript;

NestJS;

modelagem relacional;

PostgreSQL;

Prisma ORM;

autenticação baseada em JWT;

autorização baseada em roles;

hashing seguro de senhas;

validação de DTOs;

proteção contra brute force;

rate limiting;

proteção HTTP com Helmet;

isolamento de recursos por usuário;

migrations;

testes automatizados;

organização e manutenção de código.

🏗️ Stack tecnológica
Backend
Tecnologia	Utilização
Node.js	Runtime da aplicação
NestJS 11	Framework backend
TypeScript 5.9	Linguagem principal
Express	HTTP platform
Prisma 7	ORM
PostgreSQL	Banco de dados relacional
Passport	Estratégia de autenticação
JWT	Autenticação stateless
bcrypt	Hash de senhas
class-validator	Validação dos dados de entrada
class-transformer	Transformação dos DTOs
Helmet	Headers de segurança HTTP
Throttler	Rate limiting
Jest	Testes automatizados
Supertest	Testes HTTP/E2E

As dependências e scripts estão definidos no package.json do projeto. 
G
GitHub

🧱 Arquitetura
A aplicação utiliza uma arquitetura modular baseada nos recursos/domínios do sistema.

src/
├── auth/
│   ├── dto/
│   ├── auth.controller.ts
│   ├── auth.service.ts
│   ├── jwt-auth.guard.ts
│   ├── jwt.strategy.ts
│   └── roles.guard.ts
│
├── prisma/
│   ├── prisma.module.ts
│   └── prisma.service.ts
│
├── modulos/
│   ├── address/
│   ├── cart/
│   ├── cartitem/
│   ├── category/
│   ├── notification/
│   ├── order/
│   ├── orderitem/
│   ├── orderreview/
│   ├── payment/
│   ├── paymentmethod/
│   ├── product/
│   ├── report/
│   ├── review/
│   ├── stockhistory/
│   ├── supplier/
│   └── user/
│
├── app.module.ts
├── app.controller.ts
└── main.ts

prisma/
├── migrations/
└── schema.prisma

Cada módulo segue, de forma geral, uma separação entre:

Controller
    ↓
Service
    ↓
Prisma
    ↓
PostgreSQL

Essa divisão facilita manutenção, testes e evolução independente dos diferentes domínios da aplicação.

🔐 Autenticação e autorização
A API utiliza JWT Bearer Token para autenticação.

As rotas são protegidas globalmente pelo JwtAuthGuard, enquanto determinadas rotas podem ser explicitamente marcadas como públicas.

Authorization: Bearer <access_token>

A estratégia JWT extrai o token do header Authorization, valida sua assinatura e consulta o usuário no banco antes de disponibilizá-lo para os controllers. 
G
GitHub
+1

Perfis
Atualmente o sistema trabalha principalmente com:

user
admin

O controle de autorização é realizado através de RolesGuard e do decorator @Roles().

Exemplo:

@Roles('admin')
@Post()
create() {
  // ...
}

Isso permite restringir operações administrativas sem duplicar lógica de autorização dentro dos controllers. 
G
GitHub

🛡️ Segurança
Segurança foi considerada em diferentes camadas da aplicação.

Hash de senhas
As senhas não são armazenadas em texto puro.

O cadastro utiliza bcrypt:

const hashedPassword = await bcrypt.hash(password, 10);

A autenticação compara a senha informada com o hash armazenado utilizando bcrypt.compare. 
G
GitHub

Proteção contra brute force
Após 5 tentativas consecutivas de login inválidas, a conta é temporariamente bloqueada por 15 minutos.

5 tentativas inválidas
        ↓
bloqueio temporário
        ↓
15 minutos

Após um login bem-sucedido, o contador de tentativas é zerado. 
G
GitHub

Rate limiting
A aplicação utiliza @nestjs/throttler com:

60 requisições
por
1 minuto

O ThrottlerGuard é registrado globalmente. 
G
GitHub

HTTP Security Headers
A aplicação utiliza Helmet:

app.use(helmet());

Também existe configuração de CORS para permitir comunicação com o frontend. 
G
GitHub

Validação
Todos os requests passam pelo ValidationPipe global:

new ValidationPipe({
  whitelist: true,
  transform: true,
})

Isso permite validar DTOs e remover propriedades que não fazem parte do contrato esperado pela API. 
G
GitHub

🔑 Autenticação
Registrar usuário
POST /auth/register

Request
{
  "email": "usuario@email.com",
  "password": "SenhaForte123!",
  "name": "João Silva",
  "phone": "11999999999"
}

O endpoint valida o email, exige uma senha forte e cria o usuário com senha protegida por hash. 
G
GitHub
+1

Response
{
  "access_token": "<jwt>",
  "refresh_token": "<jwt>",
  "user": {
    "id": 1,
    "email": "usuario@email.com",
    "name": "João Silva",
    "role": "user",
    "phone": "11999999999"
  }
}

Login
POST /auth/login

Request
{
  "email": "usuario@email.com",
  "password": "SenhaForte123!"
}

Response
{
  "access_token": "<jwt>",
  "refresh_token": "<jwt>",
  "user": {
    "id": 1,
    "email": "usuario@email.com",
    "name": "João Silva",
    "role": "user",
    "phone": "11999999999"
  }
}

O access token possui validade configurada de 1 hora, enquanto o refresh token gerado durante login possui validade de 7 dias. 
G
GitHub
+1

Usuário autenticado
GET /auth/me
Authorization: Bearer <access_token>

Retorna os dados básicos do usuário autenticado.

Atualizar senha
PATCH /auth/change-password
Authorization: Bearer <access_token>

Exemplo:

{
  "currentPassword": "SenhaAtual123!",
  "newPassword": "NovaSenha456!"
}

Renovar access token
POST /auth/refresh

Request
{
  "refreshToken": "<refresh_token>"
}

Response
{
  "access_token": "<novo_access_token>"
}

Os endpoints de autenticação pública incluem registro, login e refresh. 
G
GitHub
+1

📚 Endpoints
A API está organizada por domínio.

👤 Usuários
Método	Endpoint	Acesso
POST	/user	Admin
GET	/user	Admin
GET	/user/:id	Usuário/Admin
PATCH	/user/:id	Usuário/Admin
DELETE	/user/:id	Admin

A implementação também verifica o usuário autenticado para evitar acesso indevido a recursos de outros usuários. 
G
GitHub

🏷️ Categorias
Método	Endpoint	Acesso
POST	/category	Admin
GET	/category	Público
GET	/category/:id	Público
PATCH	/category/:id	Admin
DELETE	/category/:id	Admin


G
GitHub
🍾 Produtos
Método	Endpoint	Acesso
POST	/product	Admin
GET	/product	Público
GET	/product/:id	Público
PATCH	/product/:id	Admin
DELETE	/product/:id	Admin

Criar produto
POST /product
Authorization: Bearer <admin_token>
Content-Type: application/json

{
  "name": "Whisky Example",
  "description": "Whisky premium",
  "price": 199.90,
  "stock": 20,
  "categoryId": 1,
  "supplierId": 1,
  "imageUrl": "https://example.com/product.jpg",
  "isActive": true
}

O DTO valida nome, preço, estoque, categoria, fornecedor e campos opcionais como imagem e status. 
G
GitHub
+1

🚚 Fornecedores
Método	Endpoint	Acesso
POST	/supplier	Admin
GET	/supplier	Público
GET	/supplier/:id	Público
PATCH	/supplier/:id	Admin
DELETE	/supplier/:id	Admin


G
GitHub
📍 Endereços
Método	Endpoint	Acesso
POST	/address	Autenticado
GET	/address	Admin
GET	/address/mine	Autenticado
GET	/address/:id	Proprietário
PATCH	/address/:id	Proprietário
DELETE	/address/:id	Proprietário

Ao criar um endereço, o userId é obtido diretamente do JWT, evitando que o cliente consiga criar um endereço em nome de outro usuário. 
G
GitHub

🛒 Carrinho
Método	Endpoint	Acesso
POST	/cart	Autenticado
GET	/cart	Admin
GET	/cart/mine	Autenticado
GET	/cart/:id	Proprietário/Admin
PATCH	/cart/:id	Proprietário/Admin
DELETE	/cart/:id	Proprietário/Admin


G
GitHub
Itens do carrinho
Método	Endpoint	Acesso
POST	/cartitem	Autenticado
GET	/cartitem	Admin
GET	/cartitem/:id	Proprietário/Admin
PATCH	/cartitem/:id	Proprietário/Admin
DELETE	/cartitem/:id	Proprietário/Admin


G
GitHub
📦 Pedidos
Método	Endpoint	Acesso
POST	/order	Autenticado
GET	/order	Admin
GET	/order/my	Autenticado
GET	/order/:id	Proprietário/Admin
PATCH	/order/:id	Proprietário/Admin
DELETE	/order/:id	Admin

Um ponto importante da implementação é que o userId do pedido é sobrescrito com o usuário obtido através do JWT:

createOrderDto.userId = user.userId;

Dessa maneira, o cliente não consegue simplesmente alterar o userId enviado no body para criar um pedido associado a outra conta. 
G
GitHub

💳 Pagamentos
Método	Endpoint	Acesso
POST	/payment	Autenticado
GET	/payment	Admin
GET	/payment/:id	Proprietário/Admin
PATCH	/payment/:id	Proprietário/Admin
DELETE	/payment/:id	Admin


G
GitHub
A entidade de pagamento suporta:

valor;

pedido relacionado;

usuário;

método de pagamento;

status;

identificador da transação.

💰 Métodos de pagamento
Método	Endpoint	Acesso
POST	/paymentmethod	Admin
GET	/paymentmethod	Público
GET	/paymentmethod/:id	Público
PATCH	/paymentmethod/:id	Admin
DELETE	/paymentmethod/:id	Admin


G
GitHub
⭐ Avaliações de produtos
Método	Endpoint	Acesso
POST	/review	Autenticado
GET	/review	Público
GET	/review/:id	Público
PATCH	/review/:id	Proprietário/Admin
DELETE	/review/:id	Admin

As avaliações possuem nota de 1 a 5, comentário opcional, usuário e produto relacionados. 
G
GitHub
+1

⭐ Avaliações de pedidos
Método	Endpoint	Acesso
POST	/orderreview	Autenticado
GET	/orderreview	Admin
GET	/orderreview/:id	Proprietário/Admin
PATCH	/orderreview/:id	Proprietário/Admin
DELETE	/orderreview/:id	Admin


G
GitHub
📦 Histórico de estoque
Método	Endpoint	Acesso
POST	/stockhistory	Admin
GET	/stockhistory	Admin
GET	/stockhistory/:id	Admin
PATCH	/stockhistory/:id	Admin
DELETE	/stockhistory/:id	Admin

Cada movimentação registra:

productId
change
reason
createdAt

O campo change representa entradas e saídas:

+10 → entrada de estoque

-3  → saída de estoque


G
GitHub
+1
🔔 Notificações
Método	Endpoint	Acesso
POST	/notification	Admin
GET	/notification	Admin
GET	/notification/:id	Usuário/Admin
PATCH	/notification/:id	Admin
DELETE	/notification/:id	Admin


G
GitHub
📊 Relatórios
Método	Endpoint	Acesso
POST	/report	Admin
GET	/report	Admin
GET	/report/:id	Admin
PATCH	/report/:id	Admin
DELETE	/report/:id	Admin

Os relatórios possuem um campo JSON para armazenar os dados gerados. 
G
GitHub
+1

🗄️ Modelo de dados
O banco de dados é PostgreSQL e utiliza Prisma ORM.

As principais entidades são:

User
 ├── Address
 ├── Cart
 │    └── CartItem
 ├── Order
 │    ├── OrderItem
 │    ├── Payment
 │    └── OrderReview
 ├── Review
 └── Notification

Product
 ├── Category
 ├── Supplier
 ├── CartItem
 ├── OrderItem
 ├── Review
 └── StockHistory

Payment
 └── PaymentMethod

Report

O schema Prisma define relacionamentos, chaves estrangeiras, índices de unicidade e regras de onDelete entre os principais recursos. 
G
GitHub

🧩 Principais entidades
User
Responsável pela identidade e autenticação do cliente.

id
email
password
name
phone
role
failedAttempts
lockUntil
createdAt
updatedAt

O email é único e o role padrão é user. 
G
GitHub

Product
Representa os produtos comercializados.

id
name
description
price
stock
categoryId
supplierId
imageUrl
isActive
createdAt
updatedAt

Cada produto pertence a uma categoria e a um fornecedor. 
G
GitHub

Order
Representa uma compra realizada pelo usuário.

id
userId
totalAmount
status
addressId
createdAt
updatedAt

Os pedidos possuem itens, pagamentos e avaliações relacionadas. 
G
GitHub

Status
A implementação documenta os seguintes estados:

pending
processing
shipped
delivered
cancelled

🔄 Fluxo principal
Um fluxo típico da aplicação pode ser representado da seguinte maneira:

                 ┌──────────────┐
                 │   Usuário    │
                 └──────┬───────┘
                        │
                        ▼
                 ┌──────────────┐
                 │   Register   │
                 │    / Login   │
                 └──────┬───────┘
                        │
                        ▼
                  ┌───────────┐
                  │    JWT    │
                  └─────┬─────┘
                        │
            ┌───────────┼───────────┐
            ▼           ▼           ▼
        Produtos     Carrinho    Endereço
            │           │           │
            └───────────┼───────────┘
                        ▼
                     Pedido
                        │
                        ▼
                    Pagamento
                        │
                        ▼
                  Pedido concluído
                        │
              ┌─────────┴─────────┐
              ▼                   ▼
          Avaliação           Notificação

🚀 Como executar
Pré-requisitos
Certifique-se de possuir instalado:

Node.js;

npm;

PostgreSQL;

Git.

1. Clonar o projeto
git clone https://github.com/Mart1nelli/adega-backend.git

cd adega-backend

2. Instalar dependências
npm install

O projeto possui scripts específicos para desenvolvimento, build, produção, Prisma e testes. 
G
GitHub

3. Configurar variáveis de ambiente
Crie o arquivo:

.env

A partir do:

.env.example

Exemplo:

DATABASE_URL="postgresql://USER:PASSWORD@HOST:PORT/DATABASE?schema=public"

JWT_SECRET=uma_chave_secreta_forte

O DATABASE_URL é utilizado pelo Prisma para conexão com PostgreSQL e o JWT_SECRET é utilizado para assinatura dos tokens. 
G
GitHub
+1

⚠️ Nunca versionar o arquivo .env contendo credenciais reais.

🗃️ Banco de dados
Depois de configurar o PostgreSQL e o DATABASE_URL, execute:

npx prisma generate

Depois aplique as migrations:

npx prisma migrate dev

O projeto utiliza prisma.config.ts para configurar o schema e o caminho das migrations. 
G
GitHub

▶️ Executando a API
Desenvolvimento
npm run start:dev

Por padrão:

http://localhost:3000

A porta pode ser alterada através da variável:

PORT=3000

A aplicação lê essa variável durante o bootstrap. 
G
GitHub

Produção
Compile:

npm run build

Execute:

npm run start:prod

O script de produção executa:

node dist/main


G
GitHub
🔬 Prisma Studio
Para visualizar e manipular os dados através da interface do Prisma:

npm run prisma:studio

ou:

npx prisma studio

🧪 Testes
Testes unitários
npm run test

Testes em modo watch
npm run test:watch

Testes E2E
npm run test:e2e

Coverage
npm run test:cov

Os scripts de testes estão configurados com Jest e Supertest. 
G
GitHub

🧹 Qualidade de código
ESLint
npm run lint

Prettier
npm run format

A configuração do projeto inclui ESLint e Prettier para padronização e manutenção do código. 
G
GitHub

📁 Organização dos módulos
O projeto possui uma separação explícita por domínio:

Módulo	Responsabilidade
auth	Autenticação e autorização
user	Usuários
category	Categorias
product	Produtos
supplier	Fornecedores
address	Endereços
cart	Carrinho
cartitem	Itens do carrinho
order	Pedidos
orderitem	Itens dos pedidos
payment	Pagamentos
paymentmethod	Métodos de pagamento
review	Avaliações de produtos
orderreview	Avaliações de pedidos
stockhistory	Histórico de estoque
notification	Notificações
report	Relatórios
prisma	Persistência e acesso ao banco

Esses módulos são registrados no AppModule, mantendo a aplicação organizada por responsabilidades de negócio. 
G
GitHub

🧠 Decisões arquiteturais
Modularização
Em vez de concentrar toda a lógica em poucos arquivos, cada domínio possui seu próprio módulo:

product/
├── dto/
├── product.controller.ts
├── product.service.ts
└── product.module.ts

Isso facilita:

manutenção;

testes;

evolução da aplicação;

separação de responsabilidades;

reutilização de serviços;

onboarding de novos desenvolvedores.

DTOs
A entrada de dados é definida através de DTOs.

Exemplo:

export class CreateProductDto {
  name: string;
  description?: string;
  price: number;
  stock?: number;
  categoryId: number;
  supplierId: number;
  imageUrl?: string;
  isActive?: boolean;
}

Os DTOs utilizam decorators do class-validator para validar os dados recebidos pela API. 
G
GitHub

Autorização centralizada
A aplicação utiliza Guards do NestJS para centralizar regras de segurança:

Request
   │
   ▼
JwtAuthGuard
   │
   ▼
RolesGuard
   │
   ▼
Controller
   │
   ▼
Service

Isso evita espalhar verificações de autenticação e autorização por toda a aplicação. 
G
GitHub
+1

🔒 Isolamento de dados por usuário
Além do controle por roles, determinados recursos verificam o proprietário do recurso.

Por exemplo:

GET /order/:id

Um usuário comum somente pode acessar seus próprios pedidos, enquanto um administrador possui acesso global.

A mesma abordagem é utilizada em recursos como:

pedidos;

carrinho;

pagamentos;

avaliações;

endereços;

notificações.


G
GitHub
+3
📈 Possibilidades de evolução
Algumas evoluções naturais para uma próxima versão do projeto seriam:

documentação interativa com Swagger/OpenAPI;

paginação padronizada;

filtros avançados de produtos;

ordenação e busca textual;

refresh token com rotação e revogação;

auditoria de operações administrativas;

integração real com gateway de pagamento;

filas para notificações;

Redis para cache;

observabilidade com logs estruturados;

métricas e tracing;

CI/CD;

Dockerização completa do ambiente;

deploy automatizado;

testes de integração mais abrangentes;

versionamento da API (/api/v1);

tratamento padronizado de erros;

utilização de Decimal para valores monetários em vez de Float.

🧪 Exemplo de utilização
Depois de iniciar a API:

npm run start:dev

Você pode testar o fluxo básico:

1. Registrar
POST http://localhost:3000/auth/register

{
  "email": "joao@email.com",
  "password": "SenhaForte123!",
  "name": "João",
  "phone": "11999999999"
}

2. Fazer login
POST http://localhost:3000/auth/login

3. Copiar o token
access_token

4. Utilizar nas rotas protegidas
Authorization: Bearer <access_token>

5. Consultar o usuário
GET http://localhost:3000/auth/me

6. Consultar produtos
GET http://localhost:3000/product

7. Criar um pedido
POST http://localhost:3000/order
Authorization: Bearer <access_token>

O backend utiliza o usuário autenticado do JWT para associar o pedido à conta correta. 
G
GitHub

📊 Visão geral da API
                    ADEGA DO PAI API
                           │
          ┌────────────────┴────────────────┐
          │                                 │
     Autenticação                       Domínios
          │                                 │
      ┌───┴───┐        ┌───────────────────┼───────────────────┐
      │       │        │                   │                   │
     JWT    RBAC    Catálogo            Vendas             Usuários
      │       │        │                   │                   │
      │       │    ┌───┼───┐          ┌────┼────┐          ┌────┼────┐
      │       │    │   │   │          │    │    │          │    │    │
      │       │ Produto Categoria  Carrinho Pedido Pagamento User Address
      │       │    │
      │       │  Supplier
      │       │
      └───────┴───────────────┐
                              │
                        PostgreSQL
                              │
                           Prisma

💡 Principais pontos técnicos
Este projeto demonstra, na prática, conhecimentos em:

NestJS

TypeScript

REST APIs

PostgreSQL

Prisma ORM

JWT

Passport

RBAC

bcrypt

DTO + ValidationPipe

Guards

Rate limiting

Helmet

Relacionamentos SQL

Migrations

Jest

Supertest

Arquitetura modular

Separação de responsabilidades

👨‍💻 Sobre o projeto
Projeto desenvolvido como backend de uma plataforma de gerenciamento e comercialização de produtos para uma adega.

O principal objetivo técnico é aplicar conceitos de desenvolvimento backend moderno, segurança, persistência relacional, arquitetura modular e boas práticas de construção de APIs REST.

📄 Licença
Este projeto atualmente está configurado como UNLICENSED no package.json.

Consulte o proprietário do projeto antes de reutilizar ou redistribuir o código. 
G
GitHub

🔗 Repositório
GitHub:
https://github.com/Mart1nelli/adega-backend
