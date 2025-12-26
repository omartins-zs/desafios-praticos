# Teste de Programação – Sistema de Agendamento de Consultas Médicas

## Objetivo  
Desenvolver um sistema que permita a uma clínica gerenciar o agendamento de consultas médicas, garantindo que não haja sobreposição de horários entre pacientes e que seja possível informar especialidade, médico e sala de atendimento para cada consulta.

---

## Problema  
Atualmente, a clínica faz agendamentos manualmente por telefone ou planilha, o que gera conflitos de horário (dois pacientes marcados para o mesmo horário com o mesmo médico ou na mesma sala), dificuldade de acompanhar atendimentos cancelados ou remarcados, e falta de visibilidade em tempo real sobre a agenda de cada profissional.

---

## Solução Proposta  
1. **Cadastro de Médicos, Pacientes e Salas**  
   - **Médico:** nome, CRM, especialidade (por exemplo, Cardiologia, Ortopedia, Pediatria).  
   - **Paciente:** nome, CPF, data de nascimento, telefone, e-mail.  
   - **Sala:** número ou código, andar, tipo de consultório (individual, sala de procedimentos, etc.).

2. **Agendamento de Consultas**  
   - O usuário (secretária ou recepcionista) escolhe: paciente_id, médico_id, sala_id, data_consulta e horário_inicio/hora_fim.  
   - Antes de confirmar, o sistema valida:  
     - Se o médico já não está ocupado em outro agendamento que sobreponha esse intervalo.  
     - Se a sala já não está ocupada no mesmo período.  
     - Limites mínimos de antecedência (por exemplo, não permitir agendar menos de 2 horas antes).

3. **Cancelamento e Remarcação**  
   - Permitir ao usuário cancelar uma consulta, informando motivo.  
   - Remarcar consiste em criar um novo registro de consulta vinculado ao anterior (relacionamento opcional “remarcada_de”) ou editar a data e horários, mas sempre revalidando conflitos.

4. **Visualização da Agenda**  
   - Tela onde se possa filtrar por médico, data e exibir blocos horários (por hora ou meia hora) com cor diferente para cada consulta.  
   - Histórico de consultas canceladas ou remarcadas.

5. **Alertas Automáticos**  
   - Um Job agendado diariamente que envie e-mail ou SMS para pacientes com consultas marcadas para o dia seguinte, como lembrete.  
   - Registrar em tabela de **LembretesEnviados** para não duplicar envios.

6. **Documentação de Uso**  
   - Instruções para configuração do banco, execução de migrations e seeders de exemplo (médicos, pacientes, salas).  
   - Exemplos de endpoints de API (se houver) ou rotas web.

---

## Tecnologias Sugeridas  
- **Linguagem de Programação:** PHP, Laravel  
- **Banco de Dados:** MySQL  
- **Front-end:** Blade + Tailwind CSS (ou framework de sua preferência)  
- **Fila de Jobs:** driver `database` ou `redis` para envio de lembretes  

---

## Estrutura do Banco de Dados  

```sql
-- Tabela: Medicos
CREATE TABLE Medicos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    crm VARCHAR(20) NOT NULL UNIQUE,
    especialidade VARCHAR(100) NOT NULL,
    telefone VARCHAR(20),
    email VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Tabela: Pacientes
CREATE TABLE Pacientes (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    cpf CHAR(11) NOT NULL UNIQUE,
    data_nascimento DATE NOT NULL,
    telefone VARCHAR(20),
    email VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Tabela: Salas
CREATE TABLE Salas (
    id INT AUTO_INCREMENT PRIMARY KEY,
    codigo VARCHAR(50) NOT NULL UNIQUE,
    andar VARCHAR(20),
    tipo VARCHAR(50), -- ex: "Consultório", "Sala de Procedimentos"
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Tabela: Consultas
CREATE TABLE Consultas (
    id INT AUTO_INCREMENT PRIMARY KEY,
    paciente_id INT NOT NULL,
    medico_id INT NOT NULL,
    sala_id INT NOT NULL,
    data_consulta DATE NOT NULL,
    horario_inicio TIME NOT NULL,
    horario_fim TIME NOT NULL,
    status ENUM('agendada','cancelada','realizada','remarcada') NOT NULL DEFAULT 'agendada',
    motivo_cancelamento VARCHAR(255) NULL,
    remarcada_de INT NULL, -- referência a outra consulta
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (paciente_id) REFERENCES Pacientes(id),
    FOREIGN KEY (medico_id) REFERENCES Medicos(id),
    FOREIGN KEY (sala_id) REFERENCES Salas(id),
    FOREIGN KEY (remarcada_de) REFERENCES Consultas(id),
    CONSTRAINT chk_horarios CHECK (horario_inicio < horario_fim)
);

-- Índices compostos para verificar conflitos rapidamente
CREATE INDEX idx_consulta_medico_data ON Consultas (medico_id, data_consulta);
CREATE INDEX idx_consulta_sala_data ON Consultas (sala_id, data_consulta);

-- Tabela: LembretesEnviados
CREATE TABLE LembretesEnviados (
    id INT AUTO_INCREMENT PRIMARY KEY,
    consulta_id INT NOT NULL,
    data_envio DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    canal ENUM('email','sms') NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (consulta_id) REFERENCES Consultas(id)
);
```
