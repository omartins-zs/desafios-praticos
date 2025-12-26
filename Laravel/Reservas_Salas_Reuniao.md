# Teste de Programação – Sistema de Reservas de Salas de Reunião

## Objetivo  
Desenvolver um sistema que permita a um usuário (funcionário) de uma empresa reservar salas de reunião, escolhendo o período desejado e verificando a disponibilidade em tempo real.

---

## Problema  
O sistema atual não possui controle de reservas de salas: todos os funcionários tentam reservar via e-mail ou grupos de chat, o que causa conflitos de horário, dupla reserva e falta de transparência sobre quais salas estão disponíveis ou ocupadas em determinado horário.

---

## Solução Proposta  
1. **Verificação de Disponibilidade**  
   - Antes de confirmar a reserva, o sistema deve checar se a sala está livre no intervalo solicitado (data e horário de início/fim).

2. **Cadastro de Salas e Reservas**  
   - Permitir ao administrador cadastrar novas salas (nome, capacidade, localização).  
   - Permitir ao usuário reservar uma sala informando sala_id, usuário_id, data, horário de início e de término.

3. **Conflito de Horário**  
   - Impedir reservas sobrepostas: antes de inserir no banco, validar se não existe outra reserva para a mesma sala cujo período conflite com o solicitado.

4. **Listagem de Reservas**  
   - Listar todas as reservas de cada sala, mostrando data, horário de início/fim e usuário responsável.

5. **Documentação de Uso**  
   - Entregar instruções básicas em texto, explicando como usar as rotas (web ou API) e como rodar as migrations de banco.

---

## Tecnologias Utilizadas  
- **Linguagem de Programação:** PHP, Laravel  
- **Banco de Dados:** MySQL  

---

## Estrutura do Banco de Dados  

```sql
-- Tabela: Usuarios
CREATE TABLE Usuarios (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    departamento VARCHAR(100),
    telefone VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Tabela: Salas
CREATE TABLE Salas (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    capacidade INT NOT NULL,
    localizacao VARCHAR(200),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Tabela: Reservas
CREATE TABLE Reservas (
    id INT AUTO_INCREMENT PRIMARY KEY,
    usuario_id INT NOT NULL,
    sala_id INT NOT NULL,
    data_reserva DATE NOT NULL,
    horario_inicio TIME NOT NULL,
    horario_fim TIME NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (usuario_id) REFERENCES Usuarios(id),
    FOREIGN KEY (sala_id) REFERENCES Salas(id),
    CONSTRAINT chk_horarios CHECK (horario_inicio < horario_fim)
);

-- Índice composto para acelerar verificação de conflito
CREATE INDEX idx_sala_data ON Reservas (sala_id, data_reserva);
```
