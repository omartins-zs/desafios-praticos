# Desafio de Programação – Notificações de Pagamento

## Objetivo  
Desenvolver um sistema que permita a um vendedor emitir lembretes de pagamento para seus clientes, selecionando o canal de notificação desejado (e-mail ou SMS) para cada cobrança. Ao final, o vendedor deverá ver, em uma lista, quais lembretes já foram enviados e em qual canal.

## Problema  
Atualmente, o sistema de vendas não possui funcionalidade para notificar o cliente sobre pagamentos pendentes de forma personalizada. Todas as notificações são enviadas automaticamente por e-mail, sem dar opção de escolher SMS ou outro canal. Isso causa baixa efetividade, pois alguns clientes não conferem e-mail com frequência e podem perder prazos de pagamento.

## Solução Proposta  
1. **Seleção de Canal de Notificação**  
   - No momento de gerar o lembrete, o vendedor poderá escolher entre “E-mail” ou “SMS” para cada cobrança.

2. **Estrutura de Banco de Dados**  
   - Tabela `Vendedores` (já existente)  
   - Tabela `Clientes`  
   - Tabela `Cobranças` (onde ficam cada fatura a receber)  
   - Tabela `NotificacoesPagamento` (novo recurso que relaciona cobrança, vendedor, canal e data de envio)  

3. **Service Layer**  
   - `NotificacaoService`  
     - Recebe `cobranca_id`, `vendedor_id` e `canal` (e-mail ou SMS)  
     - Cria registro em `NotificacoesPagamento` com status “pendente”  
     - Enfileira `EnviarNotificacaoJob` para envio em segundo plano

4. **Job (Fila)**  
   - `EnviarNotificacaoJob`  
     - Recebe `$notificacaoId`  
     - Consulta dados de cliente, cobrança e canal  
     - Chama `ServicoEmail` ou `ServicoSMS` para enviar a mensagem  
     - Atualiza `NotificacoesPagamento` com `status = enviado`, ou `falha` em caso de erro, e `data_envio`

5. **API para Testes**  
   - Endpoint `POST /api/notificacoes`  
     - Body JSON:  
       ```json
       {
         "cobranca_id": 10,
         "canal": "sms"
       }
       ```  
     - Valida se `cobranca_id` existe, pertence ao vendedor autenticado e `canal` é “sms” ou “email”  
     - Retorna JSON com dados da notificação criada (status “pendente”)

6. **Front-End Blade**  
   - **Listagem de Cobranças Pendentes** (`/cobrancas`): fatura, valor, vencimento e botão “Notificar”  
   - **Formulário de Notificação**: escolha do canal (drop-down: E-mail/SMS)  
   - **Histórico de Notificações** (`/notificacoes`): tabela com ID, Cobrança, Cliente, Canal, Status e Data de Envio

7. **Documentação de Uso**  
   - Instruções de setup (migrations, seeders)  
   - Exemplos de requisição à API (curl)  
   - Comandos principais:  
     ```
     php artisan migrate:fresh --seed
     php artisan serve
     npm run dev
     php artisan queue:work --queue=notificacoes
     ```

## Estrutura do Banco de Dados  

```sql
-- Clientes
CREATE TABLE Clientes (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100),
    email VARCHAR(100),
    telefone VARCHAR(20),
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

-- Cobranças
CREATE TABLE Cobrancas (
    id INT AUTO_INCREMENT PRIMARY KEY,
    vendedor_id INT NOT NULL,
    cliente_id INT NOT NULL,
    descricao VARCHAR(200),
    valor DECIMAL(10,2),
    data_vencimento DATE,
    status ENUM('pendente','paga','atrasada') DEFAULT 'pendente',
    created_at TIMESTAMP,
    updated_at TIMESTAMP,
    FOREIGN KEY (vendedor_id) REFERENCES Vendedores(id),
    FOREIGN KEY (cliente_id) REFERENCES Clientes(id)
);

-- NotificacoesPagamento
CREATE TABLE NotificacoesPagamento (
    id INT AUTO_INCREMENT PRIMARY KEY,
    cobranca_id INT NOT NULL,
    vendedor_id INT NOT NULL,
    canal ENUM('email','sms') NOT NULL,
    status ENUM('pendente','enviado','falha') DEFAULT 'pendente',
    data_envio DATETIME NULL,
    erro TEXT NULL,
    created_at TIMESTAMP,
    updated_at TIMESTAMP,
    FOREIGN KEY (cobranca_id) REFERENCES Cobrancas(id),
    FOREIGN KEY (vendedor_id) REFERENCES Vendedores(id)
);
```

## Componentes Principais

- **Models**  
  - `Cliente`, `Cobranca`, `NotificacaoPagamento`  

- **Migrations & Seeders**  
  - Criar tabelas e povoar com dados de exemplo (vendedor, clientes e cobranças)

- **Services** (`app/Services/`)  
  - `NotificacaoService.php`  
  - `ServicoEmail.php` (envio de e-mail)  
  - `ServicoSMS.php` (envio de SMS, pode ser simulado)

- **Job** (`app/Jobs/EnviarNotificacaoJob.php`)  
  - Processa a notificação em fila, chama o serviço respectivo e atualiza status

- **Controllers Web (Blade)**  
  - `CobrancaController` → lista cobranças e botão “Notificar”  
  - `NotificacaoController` → histórico de notificações  

- **Controller API**  
  - `Api/NotificacaoController` → método `store` para receber JSON

- **FormRequest** (`NotificacaoRequest.php`)  
  - Validação:  
    ```php
    return [
      'cobranca_id' => 'required|integer|exists:cobrancas,id',
      'canal'       => 'required|in:email,sms',
    ];
    ```

- **Rotas**  
  - **Web**:  
    ```php
    Route::get('/cobrancas', [CobrancaController::class, 'index'])->name('cobrancas.index');
    Route::post('/cobrancas/{cobranca}/notificar', [NotificacaoController::class, 'store'])->name('notificacoes.store');
    Route::get('/notificacoes', [NotificacaoController::class, 'index'])->name('notificacoes.index');
    ```
  - **API**:  
    ```php
    Route::post('/notificacoes', [App\Http\Controllers\Api\NotificacaoController::class, 'store']);
    ```

## Comandos Úteis

- **Migrations & Seeders**  
  ```bash
  php artisan migrate:fresh --seed
  ```

- **Servidor Laravel**  
  ```bash
  php artisan serve
  ```

- **Compilar Assets (Tailwind/Vite)**  
  ```bash
  npm run dev
  ```

- **Rodar fila de Jobs**  
  ```bash
  php artisan queue:work --queue=notificacoes
  ```

---

### Observações

- A lógica de negócios está separada em **Services** e **Jobs**, garantindo organização e manutenibilidade  
- A fila (`queue`) é configurada com driver `database` por padrão  
- As views podem ser estilizadas com Tailwind (ou outro framework CSS)

---

**Feito com PHP, Laravel e MySQL**  
