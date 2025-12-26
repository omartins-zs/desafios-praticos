# Teste de Programação — Sistema de Agendamento de Serviços

## Objetivo
Desenvolver um sistema que permita clientes agendarem serviços com prestadores, escolhendo data, horário e tipo de serviço.

## Problema
Os agendamentos são feitos manualmente, causando conflitos de horários, falta de histórico e dificuldade de gestão.

## Solução Proposta
Criar um sistema que permita visualizar horários disponíveis, evitar conflitos e manter histórico de agendamentos.

## Estrutura do Banco de Dados

```sql
CREATE TABLE Prestadores (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100),
    especialidade VARCHAR(100),
    email VARCHAR(100),
    telefone VARCHAR(20)
);

CREATE TABLE Clientes (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100),
    email VARCHAR(100),
    telefone VARCHAR(20)
);

CREATE TABLE Agendamentos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    prestador_id INT,
    cliente_id INT,
    data DATE,
    horario TIME,
    status VARCHAR(50),
    FOREIGN KEY (prestador_id) REFERENCES Prestadores(id),
    FOREIGN KEY (cliente_id) REFERENCES Clientes(id)
);
```
