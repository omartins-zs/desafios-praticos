# Teste de Programação — Notificações de Pedidos Atrasados

## Objetivo
Identificar pedidos atrasados e notificar automaticamente clientes e administradores.

## Problema
Pedidos atrasados são descobertos tarde, gerando reclamações.

## Solução Proposta
Sistema que detecta atrasos e dispara notificações automáticas.

## Estrutura do Banco de Dados

```sql
CREATE TABLE Clientes (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100),
    email VARCHAR(100)
);

CREATE TABLE Pedidos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    cliente_id INT,
    data_pedido DATE,
    prazo_entrega DATE,
    status VARCHAR(50),
    FOREIGN KEY (cliente_id) REFERENCES Clientes(id)
);
```
