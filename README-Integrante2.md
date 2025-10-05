# NexusCart — E-commerce Full-Stack (MERN)

**Objetivo (comum ao grupo)**  
Construir um e-commerce full-stack com catálogo de produtos, autenticação, carrinho, checkout e pedidos, usando stack **MERN**, boas práticas de API REST e versionamento por **organização + forks** no GitHub.

**Arquitetura (comum ao grupo)**  
- **Front-end:** React + Vite + React Router (consumo de API REST)  
- **Back-end:** Node.js + Express (ESM)  
- **Banco:** MongoDB (Mongoose)  
- **Criptografia / Segurança:** Bcrypt (hash de senha), JWT (auth), middlewares `protect`/`admin`  
- **Containerização:** (opcional) Docker  
- **IA:** (opcional para extensões futuras)  
- **Configuração:** variáveis em `.env` (não versionar) + `.env.example` para referência

---

## 👤 Minha contribuição — Integrante 2 (Back-end: Usuários, Carrinho & Pedidos)

### Visão geral
Responsável por **autenticação**, **endereços do usuário**, **carrinho** e **checkout/pedidos** (cálculo de totais, status e idempotência), garantindo integração com o front na jornada de compra.

### ✅ Principais entregas

#### 1) Autenticação (JWT) e segurança
- **Rotas**:
  - `POST /api/users/register` — cadastro com hash de senha (**bcrypt**)
  - `POST /api/users/login` — login com emissão de **JWT**
  - `GET /api/users/me` — perfil do usuário autenticado
- **Middleware**:
  - `protect` — valida `Authorization: Bearer <token>`
- **Modelo `User`**:
  - Campos: `name`, `email` (único), `password` (hash), `isAdmin`
  - Hooks/métodos: `pre('save')` + `matchPassword()`

#### 2) Endereços do usuário (CRUD)
- **User.addresses[]** com subdocumentos: `label`, `street`, `number`, `complement`, `district`, `city`, `state`, `zip`
- **Rotas**:
  - `GET /api/users/me/addresses`
  - `POST /api/users/me/addresses`
  - `PUT /api/users/me/addresses/:addrId`
  - `DELETE /api/users/me/addresses/:addrId`

#### 3) Carrinho
- **Rotas**:
  - `POST /api/cart` (adiciona `{ productId, qty }`)
  - `PATCH /api/cart/:productId` (ajusta quantidade)
  - `GET /api/cart/summary` (subtotal/contagem — usado no badge do header)
  - `DELETE /api/cart` (limpa carrinho)

#### 4) Checkout & Pedidos
- **Modelo `Order`**:
  - `items[]` (snapshot de produto), `subtotal`, `shipping`, `discount`, `total`
  - `shippingAddress`, `status: PLACED|PAID|CANCELED`, `paidAt`
  - `idempotencyKey` (evita duplicação de pedido) + índice único `{ user, idempotencyKey }` (`sparse`)
- **Cálculo de totais** em `utils/checkout.js`:
  - Frete grátis ≥ **200**
  - Cupom `NEXUS10` = **10%**
  - Cupom `FRETEGRATIS` = frete **0**
- **Rotas**:
  - `GET /api/cart/checkout-preview?cep=...&coupon=...` — prévia de `subtotal`, `frete`, `desconto`, `total`
  - `POST /api/orders` — cria pedido; aceita **`Idempotency-Key`**
  - `GET /api/orders?status&from&to&page&limit` — filtro/paginação
  - `GET /api/orders/:id` — detalhe
  - `PATCH /api/orders/:id/pay` — paga e **abate estoque**
  - `PATCH /api/orders/:id/cancel` — cancela e **devolve estoque** (se pago)
  - `GET /api/orders/summary` — resumo por status `{ placed, paid, canceled, total }`

#### 5) Upload (apoio ao front)
- `POST /api/upload` (multer, pasta `/uploads`) + `GET /uploads/*` (estático)

### 🗓️ Commits distribuídos
- **Dia 2:** Auth (JWT), `protect`, base de Cart/Orders; ajustes de `server.js`
- **Dia 3:** Evolução de Cart/Orders; limpeza de carrinho após criar pedido
- **Dia 4:** Checkout Preview; `Order` com endereço/totais; `GET /api/orders/summary`
- **Dia 5:** Endereços do usuário (CRUD); idempotência em `POST /api/orders`; filtros por período/status

---

## ⚙️ Como rodar (minha parte — back-end)

### `backend/.env` (exemplo — **não versionar**)
```env
PORT=4000
MONGO_URI=mongodb+srv://<user>:<pass>@<cluster>.mongodb.net/NexusCart?retryWrites=true&w=majority
JWT_SECRET=um-segredo-forte
CORS_ORIGIN=http://localhost:5173
UPLOAD_DIR=uploads
```

### Instalar e iniciar
```bash
cd backend
npm i
npm run dev   # ou: npm start
# Servidor: http://localhost:4000
```

### Testes rápidos
```bash
# Registrar
curl -X POST http://localhost:4000/api/users/register   -H "Content-Type: application/json"   -d '{"name":"Cliente","email":"user@nexusc.art","password":"user123"}'

# Login (pegar token)
curl -X POST http://localhost:4000/api/users/login   -H "Content-Type: application/json"   -d '{"email":"user@nexusc.art","password":"user123"}'
```
> Use o **token** para acessar endereços, carrinho, checkout e pedidos.

---

## 🔁 Regras de Git do Projeto (Organização + Forks)

### Remotos
- `origin` → **seu fork**  
- `upstream` → **repositório da organização**
```bash
git remote -v
git remote add upstream https://github.com/<ORGANIZACAO>/<REPO>.git   # se ainda não existir
```

### Fluxo de trabalho
1. **Sincronizar sua `main` com a organização**
   ```bash
   git fetch --all --prune
   git checkout main
   git pull --rebase upstream main
   git push origin main
   ```
2. **Criar branch por feature/dia**
   ```bash
   git checkout -b feat/integrante2-dia5-orders
   ```
3. **Commits — Conventional Commits**
   - Tipos: `feat`, `fix`, `refactor`, `chore`, `docs`, `test`, `build`, `ci`
   - Exemplos:
     - `feat(orders): idempotência no POST /api/orders`
     - `fix(cart): corrige cálculo do subtotal em summary`
   ```bash
   git add -A
   git commit -m "feat(orders): filtros por status e período em GET /api/orders"
   ```
4. **Push para o seu fork**
   ```bash
   git push -u origin feat/integrante2-dia5-orders
   ```
5. **Pull Request (fork → organização)**
   - **base:** `ORGANIZACAO/REPO` — `main`
   - **compare:** `SEU_USUARIO/REPO` — `feat/integrante2-dia5-orders`
   - **Checklist do PR**
     - [ ] Descrição clara do que foi feito  
     - [ ] Como testar (passos/endpoints)  
     - [ ] Sem arquivos sensíveis (`.env`, chaves, dumps)  
     - [ ] Build/lint ok (se houver CI)
6. **Rebase antes de finalizar**
   ```bash
   git fetch upstream
   git checkout feat/integrante2-dia5-orders
   git rebase upstream/main
   # resolver conflitos → git add <arquivo> → git rebase --continue
   git push --force-with-lease
   ```

### Política de arquivos sensíveis
- **Nunca** commitar `.env` ou credenciais.
- Adicionar ao `.gitignore`:
  ```
  .env
  backend/.env
  uploads/
  node_modules/
  ```
- Manter `backend/.env.example` com placeholders.

### Branch naming
- `feat/integrante2-diaX-<escopo>` (ex.: `feat/integrante2-dia5-orders`)
- `fix/integrante2-<bug-curto>` (ex.: `fix/integrante2-orders-calc`)
- `docs/integrante2-<tema>` (ex.: `docs/integrante2-readme`)

### Granularidade dos commits
- Commits **pequenos e temáticos**; cada commit deve **buildar e rodar**.

### Atualizar seu fork após merge na org
```bash
git checkout main
git pull --rebase upstream main
git push origin main
```

### Dica para `.env` local não atrapalhar
```bash
git update-index --assume-unchanged backend/.env
# voltar a rastrear: git update-index --no-assume-unchanged backend/.env
```

---

## 📁 Arquivos/pastas que trabalhei
- `src/routes/userRoutes.js` — auth, `/me`, `/me/addresses`  
- `src/models/User.js` — `addresses[]`, hash de senha etc.  
- `src/routes/cartRoutes.js` — add/patch/delete/summary, **checkout-preview**  
- `src/utils/checkout.js` — regras de frete/cupom/totais  
- `src/models/Order.js` — schema com **idempotencyKey**  
- `src/routes/orderRoutes.js` — criar/listar/filtrar/detalhar/pagar/cancelar/summary  
- `src/routes/uploadRoutes.js` + `server.js` — upload/estáticos `/uploads`

---

## 📝 Observações finais
- **Segurança:** JWT em headers, senhas nunca retornadas, validações básicas.  
- **Evolução:** regras de frete/cupom isoladas (`utils/checkout.js`) para futuras tabelas por CEP.  
- **Integração:** endpoints e payloads prontos para o front (paginações, filtros e summaries).
