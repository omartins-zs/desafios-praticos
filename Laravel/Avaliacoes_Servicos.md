# Teste de Programação — Sistema de Avaliação de Serviços

## Objetivo
Permitir que clientes avaliem serviços prestados.

## Problema
Falta de métricas de satisfação do cliente.

## Solução Proposta
Sistema de avaliações com notas, comentários e média por serviço.

## Estrutura do Banco de Dados

```sql
CREATE TABLE Servicos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100),
    prestador VARCHAR(100)
);

CREATE TABLE Avaliacoes (
    id INT PRIMARY KEY AUTO_INCREMENT,
    servico_id INT,
    nota INT,
    comentario VARCHAR(255),
    data_avaliacao DATE,
    FOREIGN KEY (servico_id) REFERENCES Servicos(id)
);
```
