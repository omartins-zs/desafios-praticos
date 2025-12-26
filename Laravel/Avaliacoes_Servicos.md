# Teste de Programação — Sistema Web de Avaliação de Serviços

## 📌 Contexto

Uma empresa presta diversos tipos de serviços aos seus clientes, como manutenção, suporte técnico, limpeza, consultoria ou atendimento especializado.

Atualmente, a empresa **não possui um sistema estruturado para coletar avaliações**, fazendo com que o feedback dos clientes seja informal, perdido ou não utilizado para melhoria contínua.

---

## 🎯 Objetivo

Desenvolver um **Sistema Web de Avaliação de Serviços**, permitindo que:

- Clientes avaliem serviços prestados
- Sejam atribuídas notas e comentários
- A empresa visualize métricas de satisfação
- Seja calculada a média de avaliações por serviço
- O histórico de avaliações fique armazenado e acessível

---

## ❗ Problema

A ausência de um sistema de avaliações gera os seguintes problemas:

- Falta de métricas claras de satisfação do cliente
- Dificuldade em identificar serviços mal avaliados
- Ausência de histórico de feedbacks
- Decisões baseadas apenas em percepção, não em dados
- Pouca visibilidade da qualidade dos serviços prestados

---

## 💡 Solução Proposta

Criar um sistema web que:

- Cadastre serviços e seus respectivos prestadores
- Permita que clientes realizem avaliações
- Registre nota e comentário para cada avaliação
- Calcule automaticamente a média de notas por serviço
- Exiba avaliações anteriores
- Ofereça visão gerencial da qualidade dos serviços

---

## 🧩 Regras de Negócio

1. Cada avaliação deve estar vinculada a um serviço
2. A nota deve variar de 1 a 5
3. Um serviço pode possuir várias avaliações
4. A média do serviço deve ser recalculada a cada nova avaliação
5. Comentários são opcionais, mas recomendados
6. Avaliações não podem ser editadas após o envio
7. Serviços sem avaliações devem exibir média zero ou “Sem avaliações”

---

## 🛠 Tecnologias Sugeridas

- Linguagem: PHP
- Framework: Laravel
- Banco de Dados: MySQL
- Views com Blade
- Estilização com Bootstrap ou Tailwind CSS
- Autenticação simples (opcional)

---

## 🗄 Estrutura do Banco de Dados

### Tabela: servicos

    CREATE TABLE servicos (
        id INT PRIMARY KEY AUTO_INCREMENT,
        nome VARCHAR(100) NOT NULL,
        prestador VARCHAR(100) NOT NULL,
        created_at DATETIME,
        updated_at DATETIME
    );

### Tabela: avaliacoes

    CREATE TABLE avaliacoes (
        id INT PRIMARY KEY AUTO_INCREMENT,
        servico_id INT NOT NULL,
        nota INT NOT NULL,
        comentario VARCHAR(255),
        data_avaliacao DATE NOT NULL,
        created_at DATETIME,
        updated_at DATETIME,
        FOREIGN KEY (servico_id) REFERENCES servicos(id)
    );

---

## 🖥 Funcionalidades Web Esperadas

### Tela de listagem de serviços

- Exibir:
  - Nome do serviço
  - Prestador
  - Média das avaliações
  - Quantidade de avaliações
- Botão para visualizar detalhes do serviço

### Tela de detalhes do serviço

- Exibir:
  - Informações do serviço
  - Média atual das avaliações
  - Lista de avaliações já realizadas
- Botão para adicionar nova avaliação

### Tela de avaliação

- Formulário contendo:
  - Seleção de nota (1 a 5)
  - Campo de comentário
- Botão para enviar avaliação

---

## 📊 Funcionalidades Extras (Opcional)

- Ranking de serviços mais bem avaliados
- Filtro por nota mínima
- Destaque visual para serviços mal avaliados
- Dashboard simples com métricas gerais
- Paginação de avaliações
- Moderação de comentários

---

## 🧪 Critérios de Avaliação

- Organização do código
- Uso correto do MVC
- Separação de responsabilidades
- Qualidade da modelagem do banco de dados
- Usabilidade das telas
- Clareza da documentação
- Boas práticas de desenvolvimento web
