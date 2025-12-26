# Teste de Programação – Sistema de Gerenciamento de Estoque com Alertas

## Objetivo  
Desenvolver um sistema que permita a uma equipe de vendas e logística gerenciar produtos em múltiplos armazéns, registrar entradas e saídas de estoque e gerar alertas automáticos quando o nível de um produto em qualquer armazém atingir um limite mínimo definido.

---

## Problema  
Atualmente, a empresa controla inventário por planilhas manuais ou ERP genérico sem alertas específicos. Isso causa falta de visibilidade em tempo real sobre quantos itens restam em cada armazém, levando a rupturas de estoque (produto esgotado) ou excessos (capital parado). Além disso, não há rastreio automático de movimentações de entrada/saída em cada local.

---

## Solução Proposta  
### 1. Cadastro de Produtos e Armazéns  
- **Produtos:** cada item possui código, nome, descrição, preço unitário e quantidade mínima de segurança (estoque mínimo).  
- **Armazéns:** cada local de armazenamento (e.g., “Matriz”, “Filial Sul”, “Filial Norte”) possui nome e endereço.

### 2. Controle de Estoque por Armazém  
- **EstoqueAtual:** para cada par (produto_id, armazem_id), mantemos a quantidade disponível.  
- No momento de movimentação (entrada ou saída), o sistema atualiza o estoque naquele armazém.

### 3. Registro de Movimentações  
- **Movimentacoes:** ao fazer entrada (compra, reposição) ou saída (venda, transferência), registra‐se um log contendo produto_id, armazem_id origem (para saídas) ou destino (para entradas), tipo de movimentação (enum: “entrada”, “saida”), quantidade, data e, opcionalmente, documento fiscal.

### 4. Alertas Automáticos de Baixo Estoque  
- Sempre que uma **saída** for processada, após debitar o estoque, verifica‐se se a quantidade em estoque daquele produto no armazém ficou abaixo (ou igual) ao nível mínimo.  
- Se sim, cria‐se automaticamente um registro em **AlertasEstoque** com timestamp, produto_id, armazem_id e quantidade atual. O sistema pode usar esse histórico para dashboard ou envio de notificações.

### 5. Interface ou API (para testes)  
- **Front‐end (Blade ou SPA):**  
  - Página de listagem de produtos com filtro por armazém e busca por nome/código.  
  - Formulário de movimentação (entrada/saída): seleciona produto, armazém, tipo e quantidade.  
  - Dashboard de alertas: exibe produtos em alerta por armazém.  
- **API JSON (opcional):** endpoints para listar estoques, processar movimentações e consultar alertas.

### 6. Documentação de Uso  
- Instruções breves para configurar o banco (migrations), seeders de exemplo (produtos, armazéns), rodar o Laravel e a fila (caso queira notificar via e-mail quando houver alerta), além de exemplificar chamadas à API.

---

## Tecnologias Utilizadas  
- **Linguagem de Programação:** PHP, Laravel  
- **Banco de Dados:** MySQL  
- **(Opcional) Fila/Job:** para notificar e-mail quando criado alerta, usando driver `database` ou `redis`  
- **Front‐end:** Blade + Tailwind CSS (ou Vue/React, se preferir)

---

## Estrutura do Banco de Dados  

```sql
-- Tabela: Produtos
CREATE TABLE Produtos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    codigo VARCHAR(50) NOT NULL UNIQUE,
    nome VARCHAR(150) NOT NULL,
    descricao TEXT NULL,
    preco_unitario DECIMAL(10,2) NOT NULL,
    estoque_minimo INT NOT NULL DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Tabela: Armazens
CREATE TABLE Armazens (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    endereco VARCHAR(200) NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Tabela: EstoqueAtual
CREATE TABLE EstoqueAtual (
    id INT AUTO_INCREMENT PRIMARY KEY,
    produto_id INT NOT NULL,
    armazem_id INT NOT NULL,
    quantidade INT NOT NULL DEFAULT 0,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (produto_id) REFERENCES Produtos(id),
    FOREIGN KEY (armazem_id) REFERENCES Armazens(id),
    UNIQUE KEY uq_produto_armazem (produto_id, armazem_id)
);

-- Tabela: Movimentacoes
CREATE TABLE Movimentacoes (
    id INT AUTO_INCREMENT PRIMARY KEY,
    produto_id INT NOT NULL,
    armazem_id INT NOT NULL,
    tipo ENUM('entrada','saida') NOT NULL,
    quantidade INT NOT NULL,
    data_movimentacao DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    documento VARCHAR(100) NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (produto_id) REFERENCES Produtos(id),
    FOREIGN KEY (armazem_id) REFERENCES Armazens(id)
);

-- Tabela: AlertasEstoque
CREATE TABLE AlertasEstoque (
    id INT AUTO_INCREMENT PRIMARY KEY,
    produto_id INT NOT NULL,
    armazem_id INT NOT NULL,
    quantidade_atual INT NOT NULL,
    data_alerta DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    registrado BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (produto_id) REFERENCES Produtos(id),
    FOREIGN KEY (armazem_id) REFERENCES Armazens(id)
);

-- Índices auxiliares
CREATE INDEX idx_mov_prod_armazem ON Movimentacoes(produto_id, armazem_id);
CREATE INDEX idx_alerta_prod_armazem ON AlertasEstoque(produto_id, armazem_id);
```
