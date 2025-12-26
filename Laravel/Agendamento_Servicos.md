# Teste de Programação — Sistema de Agendamento de Serviços

## 📌 Contexto

Uma empresa presta serviços sob agendamento, como por exemplo:

- Salão de beleza
- Clínica de estética
- Consultório
- Assistência técnica
- Serviços domiciliares

Atualmente, os agendamentos são feitos de forma manual (WhatsApp, telefone ou planilhas), o que gera conflitos de horários, falhas de comunicação e dificuldade de gestão.

---

## 🎯 Objetivo

Desenvolver um **Sistema de Agendamento de Serviços** que permita:

- Clientes agendarem serviços com prestadores
- Escolher data, horário e tipo de serviço
- Visualizar horários disponíveis
- Evitar conflitos de agenda
- Manter histórico completo de agendamentos
- Facilitar a gestão dos prestadores e atendimentos

---

## ❗ Problema

O modelo atual apresenta diversos problemas:

- Conflitos de horários entre atendimentos
- Falta de controle sobre disponibilidade dos prestadores
- Não há histórico confiável de agendamentos
- Dificuldade para reagendamentos e cancelamentos
- Falta de indicadores para gestão (quantidade de atendimentos, horários mais usados, etc.)

---

## 💡 Solução Proposta

Criar um sistema que:

- Cadastre clientes, prestadores e serviços
- Permita ao cliente escolher:
  - Prestador
  - Serviço
  - Data
  - Horário disponível
- Valide automaticamente conflitos de agenda
- Permita cancelamento e reagendamento
- Registre status do agendamento
- Gere histórico para clientes e prestadores
- Forneça dados para gestão do negócio

---

## 🧩 Regras de Negócio

1. Um prestador não pode ter dois agendamentos no mesmo horário
2. Um cliente pode ter múltiplos agendamentos, desde que não conflitem
3. Agendamentos possuem status:
   - PENDENTE
   - CONFIRMADO
   - CANCELADO
   - FINALIZADO
4. Apenas agendamentos CONFIRMADOS contam como ocupação de horário
5. Cancelamentos liberam o horário automaticamente
6. Um prestador pode definir seus horários de atendimento
7. Agendamentos no passado não podem ser criados
8. O sistema deve impedir sobreposição de horários

---

## 🛠 Tecnologias Sugeridas

- Linguagem: PHP
- Framework: Laravel
- Banco de Dados: MySQL
- API REST (JSON)
- Autenticação (token ou sessão)
- Front-end simples (Blade, Bootstrap ou Tailwind)

---

## 🗄 Estrutura do Banco de Dados

### Tabela: prestadores

    CREATE TABLE prestadores (
        id INT PRIMARY KEY AUTO_INCREMENT,
        nome VARCHAR(100) NOT NULL,
        especialidade VARCHAR(100),
        email VARCHAR(100),
        telefone VARCHAR(20),
        created_at DATETIME,
        updated_at DATETIME
    );

### Tabela: clientes

    CREATE TABLE clientes (
        id INT PRIMARY KEY AUTO_INCREMENT,
        nome VARCHAR(100) NOT NULL,
        email VARCHAR(100),
        telefone VARCHAR(20),
        created_at DATETIME,
        updated_at DATETIME
    );

### Tabela: servicos

    CREATE TABLE servicos (
        id INT PRIMARY KEY AUTO_INCREMENT,
        nome VARCHAR(100) NOT NULL,
        duracao_minutos INT NOT NULL,
        created_at DATETIME,
        updated_at DATETIME
    );

### Tabela: agendamentos

    CREATE TABLE agendamentos (
        id INT PRIMARY KEY AUTO_INCREMENT,
        prestador_id INT NOT NULL,
        cliente_id INT NOT NULL,
        servico_id INT NOT NULL,
        data DATE NOT NULL,
        horario TIME NOT NULL,
        status ENUM('PENDENTE','CONFIRMADO','CANCELADO','FINALIZADO') NOT NULL,
        created_at DATETIME,
        updated_at DATETIME,
        FOREIGN KEY (prestador_id) REFERENCES prestadores(id),
        FOREIGN KEY (cliente_id) REFERENCES clientes(id),
        FOREIGN KEY (servico_id) REFERENCES servicos(id)
    );

---

## 🔌 Endpoints Esperados

### Criar agendamento

    POST /api/agendamentos

    {
      "cliente_id": 1,
      "prestador_id": 2,
      "servico_id": 3,
      "data": "2025-08-10",
      "horario": "14:00"
    }

### Listar agendamentos

    GET /api/agendamentos

### Cancelar agendamento

    POST /api/agendamentos/cancelar

    {
      "agendamento_id": 10
    }

### Reagendar

    POST /api/agendamentos/reagendar

    {
      "agendamento_id": 10,
      "nova_data": "2025-08-12",
      "novo_horario": "16:00"
    }

---

## 📊 Funcionalidades Extras (Opcional)

- Agenda visual (calendário diário/semanal)
- Dashboard de gestão
- Filtros por prestador, data e status
- Envio de notificações (email ou fila)
- Uso de filas para confirmação de agendamento
- Relatórios de atendimentos por período

---

## 🧪 Critérios de Avaliação

- Organização e clareza do código
- Separação de responsabilidades (Controllers, Services, etc.)
- Validação correta de conflitos de horário
- Modelagem adequada do banco de dados
- Clareza da documentação
- Boas práticas de API REST
