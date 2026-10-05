# 🍷 Adega do Pai — REST API

Um backend completo e robusto para uma plataforma de vendas e gerenciamento de adegas. Desenvolvido com **NestJS** e **TypeScript**, este projeto aplica conceitos modernos de arquitetura modular, segurança, persistência relacional e testes automatizados.

---

## 🎯 Visão Geral

A **Adega do Pai API** centraliza todas as operações de um e-commerce de bebidas, incluindo:
- Autenticação de usuários e controle de acesso por perfis (RBAC: `admin` e `user`).
- Catálogo de produtos, categorias e controle de fornecedores.
- Gestão de estoque com histórico de movimentações.
- Carrinho de compras, criação de pedidos e simulação de pagamentos.
- Sistema de avaliações (produtos e pedidos) e notificações.

## 🚀 Tecnologias Utilizadas

A stack foi escolhida para garantir tipagem estática, escalabilidade e manutenibilidade:

- **Core:** Node.js, NestJS 11, TypeScript 5.9, Express
- **Banco de Dados:** PostgreSQL, Prisma ORM 7
- **Segurança:** Passport (JWT stateless), bcrypt, Helmet, Throttler (Rate Limiting)
- **Validação:** class-validator, class-transformer
- **Testes:** Jest, Supertest

## 🛡 Segurança e Arquitetura

O projeto segue uma arquitetura baseada em **módulos de domínio** (`Controller → Service → Prisma → DB`), promovendo o isolamento das regras de negócio.

- **Autenticação JWT:** Tokens de acesso e refresh com proteção de rotas via `JwtAuthGuard`.
- **Autorização (RBAC):** Uso de `RolesGuard` (`@Roles('admin')`) para restringir operações sensíveis.
- **Isolamento de Dados:** Validação do proprietário do recurso (ex: um usuário só acessa seu próprio carrinho/pedido extraindo o ID diretamente do token JWT).
- **Proteções Ativas:** Bloqueio de conta após 5 tentativas falhas de login (Brute Force) e Rate Limiting global (60 req/min).

## ⚙️ Como Executar o Projeto

### Pré-requisitos
- [Node.js](https://nodejs.org/) e npm
- [PostgreSQL](https://www.postgresql.org/)
- [Git](https://git-scm.com/)

### Instalação

**1. Clone o repositório**
```bash
git clone https://github.com/Mart1nelli/adega-backend.git
cd adega-backend
```

**2. Instale as dependências**
```bash
npm install
```

**3. Configure as variáveis de ambiente**
Crie um arquivo `.env` na raiz do projeto baseado no `.env.example`:
```env
DATABASE_URL="postgresql://USER:PASSWORD@HOST:PORT/DATABASE?schema=public"
JWT_SECRET="sua_chave_secreta_jwt_aqui"
PORT=3000
```

**4. Configure o Banco de Dados**
```bash
npx prisma generate
npx prisma migrate dev
```

### Inicialização

```bash
# Modo de desenvolvimento
npm run start:dev

# Build e Produção
npm run build
npm run start:prod
```

A API estará disponível por padrão em `http://localhost:3000`.

## 🧪 Testes e Qualidade

O projeto conta com uma suíte de testes unitários e E2E, além de ferramentas de padronização de código.

```bash
# Executar testes unitários
npm run test

# Executar testes E2E
npm run test:e2e

# Verificar cobertura (Coverage)
npm run test:cov

# Padronização de código
npm run lint
npm run format
```

## 📚 Estrutura de Endpoints

A API é segmentada nos seguintes domínios principais (protegidos conforme o nível de acesso):

| Domínio | Endpoint Base | Acesso Principal |
| :--- | :--- | :--- |
| **Autenticação** | `/auth` | Público (Login/Registro) / Autenticado (Me) |
| **Usuários** | `/user` | Admin / Proprietário |
| **Produtos & Catálogo** | `/product`, `/category` | Público (Leitura) / Admin (Escrita) |
| **Carrinho & Pedidos** | `/cart`, `/order` | Autenticado / Proprietário |
| **Estoque & Relatórios**| `/stockhistory`, `/report` | Admin |

> **Dica:** Após gerar o token no `/auth/login`, utilize o header `Authorization: Bearer <seu_token>` para acessar as rotas protegidas.

---

**Licença:** `UNLICENSED` - Consulte o proprietário do repositório para permissões de uso e distribuição. Desenvolvido por [Mart1nelli](https://github.com/Mart1nelli).
