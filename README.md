# Robo Financeiro Pessoal - Telegram + n8n

Um bot inteligente para controle financeiro pessoal via Telegram, alimentado por Inteligencia Artificial (Agentes de IA) e automatizado com n8n.

---

## Como funciona

O bot recebe mensagens em linguagem natural pelo Telegram. Nao e necessario digitar comandos exatos - basta escrever como voce falaria normalmente.

O fluxo interno e o seguinte:

  Telegram -> Filter My ID -> IA (Agente) -> Parse JSON -> Switch Intent -> Acao correta -> Resposta no Telegram

1. Voce manda uma mensagem no Telegram.
2. O bot filtra para garantir que so o seu ID processa os dados.
3. A Inteligencia Artificial (Agente) analisa a mensagem e identifica a intencao.
4. O fluxo e direcionado para a acao correta (salvar despesa, consultar saldo, etc.).
5. O resultado e salvo no banco de dados (n8n Datatable) e voce recebe a resposta no Telegram.

---

## Arquitetura

| Componente          | Tecnologia                     |
|---------------------|-------------------------------|
| Automacao           | n8n (self-hosted)             |
| Inteligencia Artif. | FreeLLMAPI                    |
| Mensageria          | Telegram Bot API              |
| Banco de Dados      | n8n Datatable (teste finan)   |

---

## Comandos disponiveis

O bot entende linguagem natural e reconhece 8 intencoes diferentes:

---

### 1 - Registrar Despesa (add_expense)
Salva um gasto no banco de dados.

Palavras-chave reconhecidas:
  gastei, comprei, despesa, saida, paguei algo especifico

Exemplo:
  Voce: "gastei 50 reais em almoco"
  Bot:  "Despesa salva com sucesso: R$ 50.00"

---

### 2 - Registrar Receita (add_income)
Adiciona uma entrada de dinheiro ao saldo.

Palavras-chave reconhecidas:
  adicione ao saldo, recebi, ganhei, entrou dinheiro, salario, pix recebido, deposito

Exemplo:
  Voce: "recebi meu salario de 3000 reais"
  Bot:  "Receita adicionada com sucesso: R$ 3000.00"

---

### 3 - Consultar Saldo (query_balance)
Retorna o saldo atual (receitas totais menos despesas totais).

Palavras-chave reconhecidas:
  qual meu saldo, saldo atual, quanto tenho

Exemplo:
  Voce: "qual meu saldo atual?"
  Bot:  "Seu saldo atual e R$ 2950.00
         Receitas: R$ 3000.00
         Despesas: R$ 50.00"

---

### 4 - Consultar Despesas do Mes (query_debts)
Mostra o total de despesas de um mes especifico ou do mes atual.

Palavras-chave reconhecidas:
  quanto devo, quanto gastei este mes, minhas despesas do mes

Exemplo:
  Voce: "quanto gastei em setembro?"
  Bot:  "Suas despesas em 2026-09 somam R$ 50.00."

---

### 5 - Zerar Tudo (reset_all)
Marca um ponto de reinicio geral: saldo, receitas e despesas comecam do zero a partir desse momento.

ATENCAO: Nao apaga os dados do banco. Insere um marcador de tempo que faz o calculo ignorar tudo anterior a ele.

Palavras-chave reconhecidas:
  zerar tudo, zerar meu saldo, limpar tudo, resetar financeiro

Exemplo:
  Voce: "zerar tudo"
  Bot:  "Tudo zerado com sucesso. A partir de agora seu saldo, receitas e despesas comecam do zero."

---

### 6 - Zerar Despesas (reset_expenses)
Marca um reinicio apenas para as despesas.

Palavras-chave reconhecidas:
  zerar despesas, limpar despesas, apagar gastos

---

### 7 - Zerar Receitas (reset_income)
Marca um reinicio apenas para as receitas.

Palavras-chave reconhecidas:
  zerar receitas, limpar receitas, apagar entradas

---

### 8 - Marcar Despesas como Pagas (mark_expenses_paid)
Insere um marcador de que as despesas foram quitadas. As despesas anteriores a esse ponto nao entram mais no calculo de query_debts.

Palavras-chave reconhecidas:
  paguei minhas despesas, marcar despesas como pagas, quitar despesas, registre que as despesas estao pagas

---

## Categorias disponiveis

A IA classifica automaticamente cada transacao em uma das categorias abaixo:

| Categoria     | Exemplos                              |
|---------------|---------------------------------------|
| Moradia       | Aluguel, condominio, luz, agua        |
| Transporte    | Gasolina, Uber, onibus                |
| Alimentacao   | Mercado, restaurante, delivery        |
| Lazer         | Cinema, jogos, viagens                |
| Educacao      | Cursos, livros, mensalidade           |
| Saude         | Farmacia, consulta, plano de saude    |
| Outros        | Tudo que nao se encaixa acima         |

---

## Configuracao do ambiente

### Pre-requisitos
- n8n instalado localmente:  npm install -g n8n
- Conta no Telegram e bot criado via @BotFather
- Chave de API da OpenAI

### Credenciais necessarias no n8n

| Credencial   | Onde obter                              |
|--------------|-----------------------------------------|
| Telegram API | Token gerado pelo @BotFather            |
| OpenAI API   | https://platform.openai.com/api-keys   |

### Como iniciar o n8n
  n8n

Acesse em: http://localhost:5678

---

## Estrutura do banco de dados

Nome da tabela: teste finan

| Coluna      | Tipo     | Descricao                                                                                       |
|-------------|----------|-------------------------------------------------------------------------------------------------|
| type        | String   | Tipo: expense, income, reset_all, reset_expenses, reset_income, mark_expenses_paid             |
| amount      | Number   | Valor da transacao (0 para marcadores)                                                          |
| category    | String   | Categoria da transacao                                                                          |
| description | String   | Descricao breve da transacao                                                                    |
| date        | DateTime | Data da transacao                                                                               |
| status      | String   | "paid" para transacoes normais, "marker" para marcadores de reset                              |

O arquivo schema_tabela_n8n.csv na pasta do projeto contem a estrutura da tabela para backup e reimportacao.

---

## Observacoes importantes

- O bot usa linguagem natural: nao e necessario digitar comandos exatos.
- O reset nao apaga dados: ele apenas insere um marcador de tempo. O historico completo fica salvo no banco.
- O Filter My ID e um filtro de seguranca para que apenas voce possa usar o bot. Configure com o seu Telegram ID.
- O calculo de saldo e dinamico: sempre considera os resets mais recentes na hora de calcular receitas e despesas.
- O bot foi construido com n8n e testado com OpenAI GPT-4o.

---

## Arquivos do projeto

| Arquivo                                        | Descricao                              |
|------------------------------------------------|----------------------------------------|
| Nck0h1ENZurB7HPS-Telegram_Bot_Financeiro.json  | Workflow do n8n para importacao        |
| schema_tabela_n8n.csv                          | Schema do banco de dados para backup   |
| README.md                                      | Este arquivo de documentacao           |
