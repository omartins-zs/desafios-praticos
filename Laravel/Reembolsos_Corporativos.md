# Teste de Programação — Sistema de Reembolsos Corporativos

## 📌 Contexto

Em empresas de médio e grande porte, colaboradores realizam despesas corporativas como:

- Alimentação
- Transporte
- Hospedagem
- Compra de materiais
- Serviços emergenciais

Atualmente, muitas dessas solicitações de reembolso são feitas por **e-mail ou planilhas**, o que gera confusão, atrasos, falta de padronização e dificuldade de auditoria.

---

## 🎯 Objetivo

Desenvolver um **Sistema de Reembolsos Corporativos** capaz de:

- Centralizar solicitações de reembolso
- Controlar fluxo de aprovação
- Garantir rastreabilidade das solicitações
- Manter histórico financeiro organizado
- Facilitar auditorias internas e externas

---

## ❗ Problema

O modelo atual apresenta diversos problemas:

- Solicitações feitas por e-mail se perdem facilmente
- Não existe controle claro de status
- Dificuldade para saber:
  - Quem solicitou
  - Quem aprovou
  - Quando foi aprovado
- Falta de histórico consolidado
- Risco de pagamentos duplicados ou indevidos
- Baixa transparência para colaboradores e gestores

---

## 💡 Solução Proposta

Criar um sistema que:

- Cadastre colaboradores
- Permita abertura de solicitações de reembolso
- Controle status da solicitação
- Implemente fluxo de aprovação
- Registre datas e responsáveis por cada etapa
- Permita consulta ao histórico completo
- Gere dados confiáveis para controle financeiro

---

## 🧩 Regras de Negócio

1. Todo reembolso deve estar vinculado a um colaborador
2. Uma solicitação inicia com status **PENDENTE**
3. Apenas usuários com perfil **GESTOR** ou **FINANCEIRO** podem aprovar ou reprovar
4. Status possíveis:
   - PENDENTE
   - APROVADO
   - REPROVADO
   - PAGO
5. Um reembolso só pode ser marcado como **PAGO** após aprovação
6. A data de aprovação só deve ser preenchida quando o status for APROVADO
7. Solicitações reprovadas devem conter justificativa
8. Todo o histórico deve ser auditável

---

## 🛠 Tecnologias Sugeridas

- Linguagem: PHP
- Framework: Laravel
- Banco de Dados: MySQL
- API REST (JSON)
- Autenticação por sessão ou token
- Uso de filas para notificações e processamento assíncrono

---

## 🗄 Estrutura do Banco de Dados

### Tabela: colaboradores

    CREATE TABLE colaboradores (
        id INT PRIMARY KEY AUTO_INCREMENT,
        nome VARCHAR(100) NOT NULL,
        email VARCHAR(100) UNIQUE NOT NULL,
        cargo VARCHAR(100) NOT NULL,
        created_at DATETIME,
        updated_at DATETIME
    );

### Tabela: reembolsos

    CREATE TABLE reembolsos (
        id INT PRIMARY KEY AUTO_INCREMENT,
        colaborador_id INT NOT NULL,
        descricao VARCHAR(255) NOT NULL,
        valor DECIMAL(10,2) NOT NULL,
        status ENUM('PENDENTE','APROVADO','REPROVADO','PAGO') NOT NULL,
        data_solicitacao DATE NOT NULL,
        data_aprovacao DATE NULL,
        justificativa_reprovacao TEXT NULL,
        created_at DATETIME,
        updated_at DATETIME,
        FOREIGN KEY (colaborador_id) REFERENCES colaboradores(id)
    );

---

## 🔌 Endpoints Esperados

### Criar solicitação de reembolso

    POST /api/reembolsos

    {
      "colaborador_id": 1,
      "descricao": "Almoço com cliente",
      "valor": 85.90
    }

### Aprovar reembolso

    POST /api/reembolsos/aprovar

    {
      "reembolso_id": 10
    }

### Reprovar reembolso

    POST /api/reembolsos/reprovar

    {
      "reembolso_id": 10,
      "justificativa": "Despesa fora da política"
    }

### Marcar reembolso como pago

    POST /api/reembolsos/pagar

    {
      "reembolso_id": 10
    }

### Listar reembolsos

    GET /api/reembolsos

---

## ⚙️ Processamento Assíncrono (Fila)

- Envio de notificações ao colaborador quando:
  - Solicitação for criada
  - Solicitação for aprovada ou reprovada
  - Reembolso for pago
- Uso de filas para evitar impacto na performance
- Pode ser utilizado:
  - Database Queue
  - Redis
  - RabbitMQ

---

## 📊 Funcionalidades Extras (Opcional)

- Upload de comprovantes fiscais
- Limite de valor por cargo
- Relatórios financeiros por período
- Dashboard para área financeira
- Exportação para CSV ou PDF
- Logs de auditoria

---

## 🧪 Critérios de Avaliação

- Organização do código
- Separação de responsabilidades (Controllers, Services, Jobs)
- Implementação correta das regras de negócio
- Qualidade da modelagem do banco de dados
- Uso adequado de filas
- Clareza da documentação
- Boas práticas de API REST
