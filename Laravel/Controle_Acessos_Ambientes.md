# Teste de Programação — Controle de Acesso a Ambientes

## Objetivo
Controlar entrada e saída de usuários em ambientes restritos.

## Problema
Controle manual não é seguro nem rastreável.

## Solução Proposta
Sistema de registro de acessos com níveis de permissão e histórico.

## Estrutura do Banco de Dados

```sql
CREATE TABLE Usuarios (
    id INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100),
    email VARCHAR(100),
    nivel_acesso VARCHAR(50)
);

CREATE TABLE Acessos (
    id INT PRIMARY KEY AUTO_INCREMENT,
    usuario_id INT,
    data_hora_entrada DATETIME,
    data_hora_saida DATETIME,
    FOREIGN KEY (usuario_id) REFERENCES Usuarios(id)
);
```
