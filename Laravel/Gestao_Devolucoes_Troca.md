# Teste de Programação – Sistema de Gestão de Devoluções e Trocas em E-commerce

## Objetivo  
Desenvolver um sistema que permita a uma loja online processar devoluções e trocas de produtos de forma organizada, registrando o motivo, status e conectando devoluções ao pedido original, garantindo que o cliente tenha feedback em tempo real e que o estoque seja ajustado corretamente.

---

## Problema  
No momento, a loja recebe pedidos de devolução e troca por e-mail ou telefone, sem registro centralizado. Isso gera várias dificuldades:

- Não há rastreamento automático de quais pedidos já tiveram devoluções abertas ou finalizadas.  
- O estoque não é atualizado imediatamente ao registrar uma devolução, causando inconsistência (produtos podem ficar indisponíveis ou duplicados).  
- Falta de visibilidade sobre o motivo das devoluções, dificultando análises de qualidade e atendimento.  
- Clientes não recebem status em tempo real (pêndente, aprovado, recusado, concluído) durante o processo.

---

## Solução Proposta  
1. **Cadastro de Entidades Principais**  
   - **Clientes**: armazena informações básicas (id, nome, e-mail, telefone).  
   - **Pedidos**: já existente, relaciona-se a cliente e contém itens pedidos.  
   - **Produtos**: já existente, com informações de SKU, nome, preço.  
   - **Estoque**: registro atual de cada produto.

2. **Tabela de Devoluções**  
   - Cada registro de devolução é vinculado a um item de pedido específico (pedido_id e produto_id).  
   - Campos: motivo, data_solicitacao, status (enum: “pendente”, “aprovada”, “recusada”, “concluída”), quantidade, observações do atendente.

3. **Fluxo de Status**  
   - **Pendente**: cliente solicitou devolução; aguardando análise.  
   - **Aprovada** ou **Recusada**: gestor decide.  
   - **Concluída**: produto retornou ao estoque e (se necessário) troca foi enviada ou reembolso processado.  
   - Todas as transições devem ser registradas em tabela de histórico de status.

4. **Ajuste de Estoque**  
   - Quando devolução for marcada como **concluída** (e aprovada), incrementar a quantidade de produto em estoque.  
   - Caso seja troca, criar nova saída de estoque do produto de troca e entrada do produto devolvido.

5. **Histórico e Notificações**  
   - Registrar cada alteração de status em `DevolucaoHistorico` (com timestamp e usuário responsável).  
   - Opcional: Job em fila para enviar e-mail ao cliente quando status mudar (por exemplo, de “pendente” para “aprovada” etc).

6. **Interface ou API**  
   - **Front-end Blade**:  
     - Listagem de devoluções abertas (status “pendente”).  
     - Formulário de análise: aprovar/recusar, adicionar observações.  
     - Tela de histórico de cada devolução, mostrando datas e responsáveis.  
   - **API JSON** (opcional):  
     - `POST /api/devolucoes` para criar nova solicitação (cliente informa pedido_id, produto_id, quantidade, motivo).  
     - `GET /api/devolucoes` para listar todas (ou filtrar por status).  
     - `PUT /api/devolucoes/{id}` para atualizar status.

7. **Documentação de Uso**  
   - Instruções para rodar migrations, seeders de exemplo (clientes, produtos, pedidos).  
   - Exemplos de requisição à API (curl) para criação e atualização de devoluções.  
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

-- Tabela: Pedidos (simplificada para contexto)
CREATE TABLE Pedidos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    cliente_id INT NOT NULL,
    data_pedido DATE NOT NULL,
    total DECIMAL(10,2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (cliente_id) REFERENCES Clientes(id)
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

-- Tabela: Devolucoes
CREATE TABLE Devolucoes (
    id INT AUTO_INCREMENT PRIMARY KEY,
    pedido_item_id INT NOT NULL,
    cliente_id INT NOT NULL,
    produto_id INT NOT NULL,
    quantidade INT NOT NULL,
    motivo TEXT NOT NULL,
    status ENUM('pendente','aprovada','recusada','concluida') NOT NULL DEFAULT 'pendente',
    data_solicitacao DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    data_status DATETIME NULL,
    observacoes TEXT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (pedido_item_id) REFERENCES PedidoItems(id),
    FOREIGN KEY (cliente_id) REFERENCES Clientes(id),
    FOREIGN KEY (produto_id) REFERENCES Produtos(id)
);

-- Tabela: DevolucaoHistorico
CREATE TABLE DevolucaoHistorico (
    id INT AUTO_INCREMENT PRIMARY KEY,
    devolucao_id INT NOT NULL,
    status_old ENUM('pendente','aprovada','recusada','concluida') NOT NULL,
    status_new ENUM('pendente','aprovada','recusada','concluida') NOT NULL,
    alterado_por INT NOT NULL,
    data_alteracao DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    observacoes TEXT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (devolucao_id) REFERENCES Devolucoes(id)
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
```
