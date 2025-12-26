# Teste de Programação — Sistema de Controle de Acesso a Ambientes

## 📌 Contexto

Uma empresa possui diversos ambientes restritos, como:

- Sala de servidores
- Laboratórios
- Almoxarifado
- Áreas administrativas
- Datacenters

Atualmente, o controle de entrada e saída de pessoas é feito de forma manual, o que gera falhas de segurança, ausência de rastreabilidade e dificuldade em auditorias internas e externas.

---

## 🎯 Objetivo

Desenvolver um Sistema de Controle de Acesso a Ambientes capaz de:

- Registrar entrada e saída de usuários
- Controlar permissões por nível de acesso
- Impedir acessos não autorizados
- Manter histórico completo para auditoria
- Fornecer dados confiáveis de quem acessou cada ambiente

---

## ❗ Problema

O modelo atual apresenta diversos riscos:

- Não há registro confiável de acessos
- Não existe controle de permissões
- Não é possível saber quem acessou, quando acessou e por quanto tempo permaneceu
- Dificuldade em auditorias e investigações internas
- Possível descumprimento de normas de segurança

---

## 💡 Solução Proposta

Criar um sistema que:

- Cadastre usuários com níveis de acesso
- Cadastre ambientes com nível mínimo exigido
- Registre automaticamente data e hora de entrada e saída
- Bloqueie tentativas de acesso indevidas
- Permita consultar histórico de acessos
- Gere dados auditáveis para segurança

---

## 🧩 Regras de Negócio

1. Um usuário só pode acessar ambientes compatíveis com seu nível de acesso
2. Um usuário não pode entrar duas vezes no mesmo ambiente sem registrar a saída
3. Todo acesso deve conter data e hora de entrada e saída
4. Acessos sem saída registrada são considerados ativos
5. Usuários com nível ADMIN podem visualizar todos os acessos
6. Usuários comuns visualizam apenas seus próprios registros
7. Tentativas de acesso não autorizado devem ser registradas para auditoria

---

## 🛠 Tecnologias Sugeridas

- Linguagem: PHP
- Framework: Laravel
- Banco de Dados: MySQL
- API REST (JSON)
- Autenticação por sessão ou token
- Front-end simples (Blade, Bootstrap ou Tailwind)

---

## 🗄 Estrutura do Banco de Dados

    CREATE TABLE usuarios (
        id INT PRIMARY KEY AUTO_INCREMENT,
        nome VARCHAR(100) NOT NULL,
        email VARCHAR(100) UNIQUE NOT NULL,
        nivel_acesso ENUM('ADMIN', 'GERENTE', 'USUARIO') NOT NULL,
        created_at DATETIME,
        updated_at DATETIME
    );

    CREATE TABLE ambientes (
        id INT PRIMARY KEY AUTO_INCREMENT,
        nome VARCHAR(100) NOT NULL,
        nivel_minimo_acesso ENUM('ADMIN', 'GERENTE', 'USUARIO') NOT NULL,
        created_at DATETIME,
        updated_at DATETIME
    );

    CREATE TABLE acessos (
        id INT PRIMARY KEY AUTO_INCREMENT,
        usuario_id INT NOT NULL,
        ambiente_id INT NOT NULL,
        data_hora_entrada DATETIME NOT NULL,
        data_hora_saida DATETIME NULL,
        FOREIGN KEY (usuario_id) REFERENCES usuarios(id),
        FOREIGN KEY (ambiente_id) REFERENCES ambientes(id)
    );

---

## 🔌 Endpoints Esperados

### Registrar entrada

    POST /api/acessos/entrada

    {
      "usuario_id": 1,
      "ambiente_id": 2
    }

### Registrar saída

    POST /api/acessos/saida

    {
      "acesso_id": 10
    }

### Listar acessos

    GET /api/acessos

---

## 📊 Funcionalidades Extras (Opcional)

- Dashboard com usuários atualmente em ambientes
- Total de acessos ativos
- Logs de tentativas de acesso negadas
- Exportação de relatórios em CSV ou PDF
- Uso de filas para registrar acessos de forma assíncrona
- Auditoria de acessos indevidos

---

## 🧪 Critérios de Avaliação

- Organização e clareza do código
- Separação de responsabilidades
- Implementação correta das regras de negócio
- Qualidade da modelagem do banco de dados
- Clareza da documentação
- Boas práticas de API REST
