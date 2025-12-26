# Teste de Programação – Sistema de Gestão de Devoluções, Trocas e Reembolsos em E-commerce

## Objetivo  
Desenvolver um sistema completo que permita a uma loja online processar devoluções, trocas e reembolsos de produtos de forma organizada, registrando o motivo, status e conectando devoluções ao pedido original, garantindo que o cliente tenha feedback em tempo real, que o estoque seja ajustado corretamente e que os processos financeiros sejam rastreados adequadamente.

---

## Problema  
No momento, a loja recebe pedidos de devolução e troca por e-mail ou telefone, sem registro centralizado. Isso gera várias dificuldades:

- Não há rastreamento automático de quais pedidos já tiveram devoluções abertas ou finalizadas.  
- O estoque não é atualizado imediatamente ao registrar uma devolução, causando inconsistência (produtos podem ficar indisponíveis ou duplicados).  
- Falta de visibilidade sobre o motivo das devoluções, dificultando análises de qualidade e atendimento.  
- Clientes não recebem status em tempo real (pêndente, aprovado, recusado, concluído) durante o processo.
- Não há diferenciação clara entre devoluções (com reembolso) e trocas (com novo pedido).
- Reembolsos não são rastreados de forma organizada, dificultando o controle financeiro.
- Não há registro de métodos de pagamento utilizados para reembolsos.

---

## Solução Proposta  

### 1. **Cadastro de Entidades Principais**  
   - **Clientes**: armazena informações básicas (id, nome, e-mail, telefone).  
   - **Pedidos**: relaciona-se a cliente e contém itens pedidos, com suporte para pedidos de troca.  
   - **Produtos**: com informações de SKU, nome, preço.  
   - **Estoque**: registro atual de cada produto.
   - **Usuários**: sistema de autenticação para gestores que processam devoluções e reembolsos.

### 2. **Tabela de Devoluções**  
   - Cada registro de devolução é vinculado a um item de pedido específico (pedido_id e produto_id).  
   - Campos principais:
     - `tipo`: ENUM('devolucao', 'troca') - diferencia devoluções de trocas
     - `produto_troca_id`: ID do produto desejado na troca (obrigatório quando tipo = 'troca')
     - `motivo`: texto livre explicando o motivo
     - `status`: ENUM('pendente', 'aprovada', 'recusada', 'concluida')
     - `quantidade`: quantidade de itens devolvidos/trocados
     - `codigo_rastreamento`: código único para rastreamento
     - `data_solicitacao`, `data_status`, `data_envio`: timestamps importantes
     - `observacoes`: notas do atendente

### 3. **Sistema de Trocas**
   - Quando `tipo = 'troca'`, o sistema:
     - Valida que o produto de troca é diferente do produto devolvido
     - Valida estoque suficiente do produto de troca
     - Ao concluir: incrementa estoque do produto devolvido e decrementa do produto de troca
     - **Cria automaticamente um novo pedido** vinculado à devolução (pedido de troca)
     - O pedido de troca é marcado com `eh_pedido_troca = true`

### 4. **Sistema de Reembolsos**
   - Quando `tipo = 'devolucao'` (não troca) e status = 'concluida':
     - **Cria automaticamente um registro de reembolso** com status `pendente`
     - Calcula valor baseado no preço unitário × quantidade do item do pedido
     - Vincula à devolução e cliente
   - Campos do reembolso:
     - `valor`: valor calculado automaticamente
     - `status`: ENUM('pendente', 'processado', 'cancelado')
     - `metodo`: ENUM('credito_estorno', 'transferencia', 'boleto', 'pix')
     - `autorizado`: boolean indicando se foi autorizado
     - `autorizado_por`, `data_autorizacao`: rastreamento de autorização
     - `processado_por`, `data_processamento`: rastreamento de processamento
     - `observacoes`: notas sobre o processamento

### 5. **Fluxo de Status de Devoluções**  
   - **Pendente**: cliente solicitou devolução/troca; aguardando análise.  
   - **Aprovada** ou **Recusada**: gestor decide.  
   - **Concluída**: 
     - Produto retornou ao estoque (incremento automático)
     - Se for troca: produto de troca foi enviado (decremento automático) e pedido de troca criado
     - Se for devolução: reembolso criado automaticamente (status pendente)
   - Todas as transições devem ser registradas em tabela de histórico de status.

### 6. **Fluxo de Reembolso**
   - **Criação automática**: quando devolução é concluída (tipo = 'devolucao')
   - **Status pendente**: aguardando processamento
   - **Processamento**: gestor seleciona método de pagamento e processa
   - **Status processado**: reembolso foi efetivado
   - **Status cancelado**: reembolso foi cancelado (caso necessário)

### 7. **Ajuste de Estoque**  
   - Quando devolução/troca for marcada como **concluída**:
     - **Sempre incrementa** a quantidade do produto devolvido em estoque
     - **Se for troca**: decrementa a quantidade do produto de troca em estoque
     - Valida estoque suficiente antes de processar trocas
   - Transações garantem consistência (tudo ou nada)

### 8. **Histórico e Notificações**  
   - Registrar cada alteração de status em `DevolucaoHistorico` (com timestamp e usuário responsável).  
   - Job em fila para enviar e-mail ao cliente quando status mudar (pendente → aprovada, aprovada → concluída, etc).
   - Job separado para notificações de reembolso processado.
   - Tabela `LembretesEmail` registra todos os e-mails enviados.

### 9. **Interface ou API**  
   - **Front-end Blade**:  
     - Listagem de devoluções abertas (status "pendente").  
     - Formulário de análise: aprovar/recusar, adicionar observações.  
     - Tela de histórico de cada devolução, mostrando datas e responsáveis.
     - **Listagem de reembolsos** com filtros por status
     - **Detalhes do reembolso** com formulário para processar (método de pagamento)
     - **Visualização de pedidos de troca** vinculados às devoluções
   - **API JSON**:  
     - `POST /api/devolucoes` para criar nova solicitação (suporta tipo 'devolucao' ou 'troca')
     - `GET /api/devolucoes` para listar todas (ou filtrar por status, tipo, cliente, produto)
     - `GET /api/devolucoes/{id}` para visualizar detalhes
     - `PUT /api/devolucoes/{id}` para atualizar status
     - `GET /api/reembolsos` para listar reembolsos
     - `GET /api/reembolsos/{id}` para visualizar reembolso
     - `POST /api/reembolsos/{id}/processar` para processar reembolso

### 10. **Documentação de Uso**  
   - Instruções para rodar migrations, seeders de exemplo (clientes, produtos, pedidos, devoluções, reembolsos).  
   - Exemplos de requisição à API (curl) para criação e atualização de devoluções e reembolsos.  
   - Comandos principais:  
     ```
     php artisan migrate:fresh --seed
     php artisan serve
     php artisan queue:work --queue=emails
     npm run dev
     ```

---

## Estrutura do Banco de Dados  

```sql
-- Tabela: Clientes
CREATE TABLE Clientes (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    telefone VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Tabela: Produtos
CREATE TABLE Produtos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    sku VARCHAR(50) NOT NULL UNIQUE,
    nome VARCHAR(150) NOT NULL,
    preco DECIMAL(10,2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Tabela: Pedidos (com suporte para pedidos de troca)
CREATE TABLE Pedidos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    cliente_id INT NOT NULL,
    devolucao_id INT NULL,  -- FK para devolução (quando é pedido de troca)
    data_pedido DATE NOT NULL,
    total DECIMAL(10,2) NOT NULL,
    eh_pedido_troca BOOLEAN DEFAULT FALSE,  -- Indica se é pedido gerado por troca
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (cliente_id) REFERENCES Clientes(id),
    FOREIGN KEY (devolucao_id) REFERENCES Devolucoes(id) ON DELETE SET NULL
);

-- Tabela: PedidoItems (itens de cada pedido)
CREATE TABLE PedidoItems (
    id INT AUTO_INCREMENT PRIMARY KEY,
    pedido_id INT NOT NULL,
    produto_id INT NOT NULL,
    quantidade INT NOT NULL,
    preco_unitario DECIMAL(10,2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (pedido_id) REFERENCES Pedidos(id),
    FOREIGN KEY (produto_id) REFERENCES Produtos(id)
);

-- Tabela: EstoqueAtual
CREATE TABLE EstoqueAtual (
    id INT AUTO_INCREMENT PRIMARY KEY,
    produto_id INT NOT NULL,
    quantidade INT NOT NULL DEFAULT 0,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (produto_id) REFERENCES Produtos(id),
    UNIQUE (produto_id)
);

-- Tabela: Devolucoes (com suporte para trocas)
CREATE TABLE Devolucoes (
    id INT AUTO_INCREMENT PRIMARY KEY,
    pedido_item_id INT NOT NULL,
    cliente_id INT NOT NULL,
    produto_id INT NOT NULL,
    produto_troca_id INT NULL,  -- FK para produto de troca (obrigatório quando tipo = 'troca')
    quantidade INT NOT NULL,
    motivo TEXT NOT NULL,
    motivo_troca TEXT NULL,  -- Motivo específico da troca (opcional)
    tipo ENUM('devolucao', 'troca') NOT NULL DEFAULT 'devolucao',
    status ENUM('pendente','aprovada','recusada','concluida') NOT NULL DEFAULT 'pendente',
    data_solicitacao DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    data_status DATETIME NULL,
    data_envio DATETIME NULL,  -- Data de envio do produto de troca
    codigo_rastreamento VARCHAR(50) NULL UNIQUE,  -- Código único para rastreamento
    observacoes TEXT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (pedido_item_id) REFERENCES PedidoItems(id),
    FOREIGN KEY (cliente_id) REFERENCES Clientes(id),
    FOREIGN KEY (produto_id) REFERENCES Produtos(id),
    FOREIGN KEY (produto_troca_id) REFERENCES Produtos(id) ON DELETE SET NULL
);

-- Tabela: DevolucaoHistorico
CREATE TABLE DevolucaoHistorico (
    id INT AUTO_INCREMENT PRIMARY KEY,
    devolucao_id INT NOT NULL,
    status_old ENUM('pendente','aprovada','recusada','concluida') NOT NULL,
    status_new ENUM('pendente','aprovada','recusada','concluida') NOT NULL,
    alterado_por INT NOT NULL,  -- FK para users
    data_alteracao DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    observacoes TEXT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (devolucao_id) REFERENCES Devolucoes(id),
    FOREIGN KEY (alterado_por) REFERENCES Users(id)
);

-- Tabela: Reembolsos (novo)
CREATE TABLE Reembolsos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    devolucao_id INT NOT NULL UNIQUE,  -- Uma devolução gera apenas um reembolso
    cliente_id INT NOT NULL,
    valor DECIMAL(10,2) NOT NULL,
    status ENUM('pendente', 'processado', 'cancelado') NOT NULL DEFAULT 'pendente',
    autorizado BOOLEAN DEFAULT FALSE,
    autorizado_por INT NULL,  -- FK para users
    data_autorizacao DATETIME NULL,
    metodo ENUM('credito_estorno', 'transferencia', 'boleto', 'pix') NULL,
    observacoes TEXT NULL,
    data_processamento DATETIME NULL,
    processado_por INT NULL,  -- FK para users
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (devolucao_id) REFERENCES Devolucoes(id) ON DELETE CASCADE,
    FOREIGN KEY (cliente_id) REFERENCES Clientes(id) ON DELETE CASCADE,
    FOREIGN KEY (autorizado_por) REFERENCES Users(id) ON DELETE SET NULL,
    FOREIGN KEY (processado_por) REFERENCES Users(id) ON DELETE SET NULL
);

-- Tabela: LembretesEmail (opcional)
CREATE TABLE LembretesEmail (
    id INT AUTO_INCREMENT PRIMARY KEY,
    devolucao_id INT NOT NULL,
    data_envio DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    canal VARCHAR(20) NOT NULL DEFAULT 'email',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (devolucao_id) REFERENCES Devolucoes(id)
);

-- Tabela: Users (sistema de autenticação)
CREATE TABLE Users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    remember_token VARCHAR(100) NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

---

## Funcionalidades Implementadas

### ✅ Devoluções
- Criação de solicitações de devolução vinculadas a itens de pedido
- Fluxo de status: pendente → aprovada/recusada → concluída
- Histórico completo de alterações de status
- Código de rastreamento único
- Validação de quantidade (não pode exceder quantidade do pedido)

### ✅ Trocas
- Diferenciação entre devolução e troca através do campo `tipo`
- Validação: produto de troca obrigatório e diferente do produto devolvido
- Validação de estoque suficiente do produto de troca
- Ajuste automático de estoque (incrementa devolvido, decrementa troca)
- Criação automática de pedido de troca quando concluída
- Rastreamento de data de envio do produto de troca

### ✅ Reembolsos
- Criação automática quando devolução (não troca) é concluída
- Cálculo automático do valor (preço unitário × quantidade)
- Fluxo de processamento: pendente → processado/cancelado
- Métodos de pagamento: Crédito/Estorno, Transferência, Boleto, PIX
- Rastreamento de autorização e processamento (quem, quando)
- Interface web para processar reembolsos

### ✅ Estoque
- Ajuste automático ao concluir devoluções/trocas
- Validação de estoque antes de processar trocas
- Transações garantem consistência

### ✅ Notificações
- E-mails assíncronos via Jobs em fila
- Notificações para mudanças de status de devolução
- Notificações para processamento de reembolso
- Registro de todos os e-mails enviados

### ✅ Interface Web
- Listagem de devoluções com filtros
- Detalhes de devolução com histórico
- Listagem de reembolsos com filtros
- Formulário para processar reembolsos
- Visualização de pedidos de troca vinculados

### ✅ API RESTful
- Endpoints completos para devoluções (CRUD)
- Endpoints para reembolsos (listar, visualizar, processar)
- Filtros avançados (status, tipo, cliente, produto)
- Respostas padronizadas em JSON
- Validação através de Form Requests

---

## Fluxos de Trabalho

### Fluxo de Devolução com Reembolso

1. **Cliente solicita devolução** via API ou interface
   - Tipo: `devolucao`
   - Status: `pendente`
   - E-mail enviado ao cliente

2. **Gestor analisa e aprova**
   - Status: `aprovada`
   - E-mail enviado ao cliente

3. **Gestor conclui devolução**
   - Status: `concluida`
   - Estoque do produto devolvido incrementado
   - **Reembolso criado automaticamente** (status: `pendente`)
   - E-mail enviado ao cliente

4. **Gestor processa reembolso**
   - Seleciona método de pagamento
   - Adiciona observações
   - Status: `processado`
   - E-mail enviado ao cliente

### Fluxo de Troca

1. **Cliente solicita troca** via API ou interface
   - Tipo: `troca`
   - `produto_troca_id` obrigatório
   - Status: `pendente`
   - E-mail enviado ao cliente

2. **Gestor analisa e aprova**
   - Status: `aprovada`
   - E-mail enviado ao cliente

3. **Gestor conclui troca**
   - Status: `concluida`
   - Estoque do produto devolvido incrementado
   - Estoque do produto de troca decrementado
   - **Pedido de troca criado automaticamente**
   - E-mail enviado ao cliente

---

## Exemplos de Uso da API

### Criar Devolução

```bash
POST /api/devolucoes
Content-Type: application/json

{
  "pedido_item_id": 1,
  "quantidade": 2,
  "motivo": "Produto com defeito na tela",
  "tipo": "devolucao"
}
```

### Criar Troca

```bash
POST /api/devolucoes
Content-Type: application/json

{
  "pedido_item_id": 1,
  "quantidade": 1,
  "motivo": "Produto não corresponde à descrição. Quero trocar por outro modelo.",
  "tipo": "troca",
  "produto_troca_id": 2
}
```

### Atualizar Status de Devolução

```bash
PUT /api/devolucoes/1
Content-Type: application/json

{
  "status": "aprovada",
  "observacoes": "Devolução aprovada. Cliente deve enviar o produto."
}
```

### Processar Reembolso

```bash
POST /api/reembolsos/1/processar
Content-Type: application/json

{
  "metodo": "pix",
  "observacoes": "Reembolso processado via PIX. Código de rastreamento: PIX123456"
}
```

---

## Requisitos Técnicos

### Tecnologias
- **Laravel 12** (Framework PHP)
- **PHP 8.2+**
- **MySQL/MariaDB** ou SQLite
- **Tailwind CSS** (para interface)
- **Sistema de Filas** (para e-mails assíncronos)

### Arquitetura
- **Services**: Lógica de negócio isolada (DevolucaoService, ReembolsoService, EstoqueService)
- **Form Requests**: Validação isolada
- **Jobs**: Processamento assíncrono de e-mails
- **Models**: Relacionamentos Eloquent bem definidos
- **Controllers**: Separação entre API e Web

### Boas Práticas
- Transações de banco para garantir consistência
- Logs estruturados para debugging
- Tratamento de erros padronizado
- Validações completas
- Código limpo e organizado (SOLID)

---

## Comandos de Instalação e Uso

```bash
# Instalar dependências
composer install
npm install

# Configurar ambiente
cp .env.example .env
php artisan key:generate

# Configurar banco de dados no .env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=nome_do_banco
DB_USERNAME=seu_usuario
DB_PASSWORD=sua_senha

# Executar migrations e seeders
php artisan migrate:fresh --seed

# Iniciar servidor
php artisan serve

# Processar filas (e-mails)
php artisan queue:work

# Ou executar tudo junto (servidor + filas + vite)
composer dev
```

---

## Dados de Exemplo

Os seeders criam:
- **5 Clientes** de exemplo
- **6 Produtos** com estoque inicial
- **10 Pedidos** com itens aleatórios
- **5 Devoluções** com diferentes status (incluindo trocas)
- **Reembolsos** criados automaticamente para devoluções concluídas

---

## Observações Importantes

1. **Reembolsos são criados automaticamente** quando uma devolução (não troca) é concluída
2. **Pedidos de troca são criados automaticamente** quando uma troca é concluída
3. **Estoque é ajustado automaticamente** ao concluir devoluções/trocas
4. **E-mails são enviados de forma assíncrona** via sistema de filas
5. **Todas as transições de status são registradas** no histórico
6. **Validações garantem integridade** dos dados (quantidade, estoque, etc)

---

Este documento descreve o desafio completo e atualizado, incluindo todas as funcionalidades de devoluções, trocas e reembolsos implementadas no sistema.

