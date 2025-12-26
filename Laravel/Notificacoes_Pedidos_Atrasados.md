# Teste de Programação — Sistema de Notificações de Pedidos Atrasados

## 📌 Contexto

Uma empresa de e-commerce realiza entregas diariamente para seus clientes.  
Atualmente, não existe um controle automatizado para identificar **pedidos atrasados**, fazendo com que clientes e administradores só percebam o problema quando o atraso já gerou reclamações.

A ausência de notificações proativas impacta negativamente a experiência do cliente e dificulta a gestão logística.

---

## 🎯 Objetivo

Desenvolver um **Sistema de Monitoramento e Notificação de Pedidos Atrasados**, capaz de:

- Identificar automaticamente pedidos fora do prazo
- Notificar clientes sobre atrasos
- Alertar administradores para ação corretiva
- Manter histórico de notificações enviadas
- Facilitar o acompanhamento do status das entregas

---

## ❗ Problema

O modelo atual apresenta diversos problemas:

- Pedidos atrasados não são identificados em tempo hábil
- Clientes só percebem o atraso quando não recebem o pedido
- Administradores não possuem visão clara de pedidos críticos
- Falta de histórico de comunicação com o cliente
- Impacto negativo na satisfação e retenção de clientes

---

## 💡 Solução Proposta

Criar um sistema que:

- Cadastre clientes e pedidos
- Controle prazo de entrega por pedido
- Identifique automaticamente pedidos em atraso
- Dispare notificações automáticas para:
  - Cliente
  - Administrador
- Registre todas as notificações enviadas
- Permita acompanhamento do status do pedido
- Gere dados para análise e melhoria logística

---

## 🧩 Regras de Negócio

1. Um pedido é considerado **atrasado** quando a data atual ultrapassa o prazo de entrega
2. Apenas pedidos com status:
   - EM_TRANSITO
   - PROCESSANDO  
   podem ser considerados atrasados
3. Pedidos ENTREGUES ou CANCELADOS não devem gerar notificações
4. Cada pedido atrasado deve gerar:
   - Pelo menos uma notificação para o cliente
   - Pelo menos uma notificação para o administrador
5. O sistema não deve enviar notificações duplicadas para o mesmo atraso
6. Alterações de status devem ser registradas
7. O histórico de notificações deve ser auditável

---

## 🛠 Tecnologias Sugeridas

- Linguagem: PHP
- Framework: Laravel
- Banco de Dados: MySQL
- API REST (JSON)
- Sistema de notificações (email ou log)
- Uso de filas para envio assíncrono de notificações

---

## 🗄 Estrutura do Banco de Dados

### Tabela: clientes

    CREATE TABLE clientes (
        id INT PRIMARY KEY AUTO_INCREMENT,
        nome VARCHAR(100) NOT NULL,
        email VARCHAR(100) NOT NULL,
        created_at DATETIME,
        updated_at DATETIME
    );

### Tabela: pedidos

    CREATE TABLE pedidos (
        id INT PRIMARY KEY AUTO_INCREMENT,
        cliente_id INT NOT NULL,
        data_pedido DATE NOT NULL,
        prazo_entrega DATE NOT NULL,
        status ENUM('PROCESSANDO','EM_TRANSITO','ENTREGUE','CANCELADO') NOT NULL,
        created_at DATETIME,
        updated_at DATETIME,
        FOREIGN KEY (cliente_id) REFERENCES clientes(id)
    );

### Tabela: notificacoes

    CREATE TABLE notificacoes (
        id INT PRIMARY KEY AUTO_INCREMENT,
        pedido_id INT NOT NULL,
        tipo ENUM('CLIENTE','ADMIN') NOT NULL,
        mensagem TEXT NOT NULL,
        data_envio DATETIME NOT NULL,
        created_at DATETIME,
        FOREIGN KEY (pedido_id) REFERENCES pedidos(id)
    );

---

## 🔌 Endpoints Esperados

### Criar pedido

    POST /api/pedidos

    {
      "cliente_id": 1,
      "prazo_entrega": "2025-08-15"
    }

### Atualizar status do pedido

    POST /api/pedidos/status

    {
      "pedido_id": 10,
      "status": "EM_TRANSITO"
    }

### Listar pedidos atrasados

    GET /api/pedidos/atrasados

### Disparar verificação manual de atrasos

    POST /api/pedidos/verificar-atrasos

---

## ⚙️ Processamento Assíncrono (Fila)

- Um Job deve rodar periodicamente para:
  - Verificar pedidos atrasados
  - Disparar notificações
- O uso de filas evita impacto de performance
- O sistema pode usar:
  - Database Queue
  - Redis
  - RabbitMQ

---

## 📊 Funcionalidades Extras (Opcional)

- Dashboard com pedidos atrasados
- Relatórios por período
- Configuração de prazo por tipo de pedido
- Reenvio de notificações
- Integração com e-mail ou SMS
- Logs de tentativas de notificação

---

## 🧪 Critérios de Avaliação

- Organização do código
- Separação de responsabilidades (Controllers, Services, Jobs)
- Implementação correta das regras de negócio
- Uso adequado de filas
- Modelagem do banco de dados
- Clareza da documentação
- Boas práticas de API REST
