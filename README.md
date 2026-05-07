# banco-de-dados-nosql
Repositório para alocar o projeto da disciplina de Banco de Dados NoSQL. 

LogShield: Auditoria Imutável de Acessos a Dados Sensíveis (Zero Trust)
Projeto acadêmico desenvolvido para a disciplina de Banco de Dados NoSQL
Curso de Gestão da Informação (GI) — Universidade Federal de Uberlândia (UFU)

Sobre o Projeto
O LogShield é um sistema distribuído de armazenamento de logs de segurança construído para auditar, em tempo real, todas as interações com dados sensíveis (PII - Personally Identifiable Information) dentro de um ecossistema corporativo. Baseado no princípio de Zero Trust (Confiança Zero), o sistema assume que nenhuma requisição de dados é inerentemente segura.
A proposta não é gerenciar os usuários, mas sim atuar como um "Cofre Negro" (Black Box) inalterável: toda vez que um dado crítico é lido, modificado ou exportado em qualquer sistema da empresa, o LogShield registra o evento.

O Problema: O Risco Oculto e a Rigidez Relacional
Com a consolidação de leis como a LGPD e a GDPR, o vazamento de dados ou o acesso indevido resulta em multas milionárias e perda de reputação. Quando um incidente ocorre, os auditores fazem uma pergunta simples: "Quem acessou o banco de dados de clientes no dia 15 de março?"
Muitas empresas não conseguem responder porque enfrentam dois gargalos estruturais:
1.	Gargalo de Performance (SQL): Salvar um log de auditoria para cada SELECT ou visualização de tela em um banco relacional derruba a performance do sistema principal. Bancos SQL não foram desenhados para suportar tempestades contínuas de gravações (Write-Heavy Workloads).
2.	Fragilidade de Auditoria: Logs salvos em tabelas SQL padrão ou arquivos de texto (.txt / .csv) podem ser facilmente editados ou apagados por um administrador de banco de dados (DBA) mal-intencionado, quebrando a cadeia de custódia da auditoria.

A Solução e o Fluxo de Funcionamento
O sistema resolve esse problema separando a auditoria da aplicação principal e utilizando uma arquitetura orientada a eventos Append-Only (apenas inserção).
O fluxo ocorre em 3 etapas:
1.	Interceptação (O Gatilho): Um funcionário abre o cadastro de um cliente no CRM da empresa ou uma API solicita um dado.
2.	Ingestão Assíncrona: A aplicação principal envia rapidamente um evento JSON para o LogShield (via mensageria, como Kafka/RabbitMQ) contendo:
• timestamp: Data e hora exata (milissegundos)
• actor_id: Matrícula do funcionário ou ID do serviço
• action: READ, EXPORT, UPDATE, DELETE
• data_accessed: CPF, Cartão de Crédito, Histórico Médico, etc.
• ip_address: Origem da requisição
• justification: Motivo do acesso (ex: "Atendimento ao ticket #450")
3.	Armazenamento e Alerta: O NoSQL absorve o dado instantaneamente. Um motor de regras avalia o padrão e, se for anômalo, dispara um alerta para a equipe de segurança.

Exemplo Prático (O Cenário de Risco)
• Sexta-feira, 03:15 da manhã: A credencial de um analista de marketing, que normalmente acessa 50 perfis de clientes por dia em horário comercial, começa a exportar 10.000 registros contendo e-mail e telefone de clientes VIPs.
• Ação do LogShield: O banco Wide-Column recebe os 10.000 logs em menos de um segundo sem engasgar. O motor de anomalias detecta a quebra de padrão (horário atípico + volume massivo) e bloqueia a credencial automaticamente, alertando o CISO (Chief Information Security Officer) antes que o vazamento se concretize.

Por que Banco de Dados NoSQL? (A Defesa Técnica)
Para este desafio, a escolha ideal recai sobre a família de bancos de dados Wide-Column (como Apache Cassandra ou ScyllaDB), fundamentada em três pilares:
• Write-Optimized (Otimizado para Escrita): Diferente do SQL, a arquitetura de armazenamento do Cassandra é desenhada primariamente para absorver milhões de inserções por segundo. Inserir um log tem custo de performance quase zero.
• Escalabilidade Horizontal Absoluta: Conforme a empresa cresce e gera mais logs corporativos, basta adicionar novos nós (servidores) ao cluster. Os dados de log são particionados automaticamente com base no tempo (time-window) e no usuário.
• Time-to-Live (TTL) Nativo: Leis de proteção de dados exigem que logs sejam mantidos por X anos e depois destruídos. No Cassandra, é possível configurar um TTL no momento da inserção; o banco expurga o log automaticamente no fim do prazo, eliminando rotinas pesadas de exclusão manual (DELETE).
