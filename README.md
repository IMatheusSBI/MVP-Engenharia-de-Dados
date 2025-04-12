# MVP Engenharia de Dados

### 1. Objetivo

O objetivo deste projeto é desenvolver um pipeline de dados na nuvem que permita o monitoramento contínuo dos pluviômetros instalados em uma ferrovia. Atualmente, os alertas de precipitação e falhas dos dispositivos são enviados exclusivamente por e-mail, dificultando a análise integrada e a tomada de decisões rápidas. O projeto busca:

Coletar e persistir de forma incremental os dados dos dispositivos (pluviômetros) e seus sensores, que medem a quantidade de chuva (em mm) a cada 15 minutos.

Construir um modelo semântico que relacione os dispositivos e as medições, facilitando a criação de dashboards interativos no Power BI.

Responder às seguintes perguntas de negócio:

  1. Existe correlação entre eventos de chuva intensa e falhas no funcionamento dos dispositivos?

  2. Qual foi a quantidade de precipitação registrada em cada ponto da ferrovia nas últimas 24 horas e nos últimos 7 dias?

  3. Quantos gatilhos de alerta diário e semanal foram atingidos em determinado período?

  4.Quais pluviômetros estão atualmente offline ou apresentaram falhas de comunicação?

  5. Onde estão localizados os pluviômetros que registraram as maiores quantidades de precipitação?

  6. Existe correlação entre falhas nos dispositivos e eventos de chuva intensa?

  7. Qual o histórico de funcionamento dos dispositivos em termos de disponibilidade de dados?

### 2. Coleta

A coleta dos dados foi realizada a partir de uma API fornecida por uma empresa terceirizada responsável pelos pluviômetros. As etapas de coleta incluem:

Autenticação e Requisições:
Pelo PySpark foi criado uma classe para gerenciamento da autenticação (obtendo o token de acesso) e para realizar as requisições HTTP necessárias para obter:

  1. A lista de dispositivos (pluviômetros) e suas informações.

  2. As medições dos sensores físicos e dos soft sensors.

Persistência dos Dados:
Os dados coletados são integrados e armazenados de forma incremental em um Lakehouse (utilizando Delta Lake), garantindo que novos dados sejam mesclados sem sobrescrever o histórico. Essa operação é realizada através dos métodos overwriteData (para cargas completas, como na atualização dos dispositivos) e mergeData (para operações incrementais diárias).

A coleta é documentada tanto pela definição dos métodos na classe DataCollectionService quanto pela estratégia de coletar os dados em intervalos diários.

### 3. Modelagem
Todos os campos foram tipados de forma rigorosa utilizando schemas do PySpark (exemplo: TimestampType para datas, DoubleType para valores, etc.).

Foram definidos valores mínimos e máximos esperados para as medições (precipitação em mm) e categorizado os tipos de sensores (físicos e soft).

Linhagem dos Dados:

Origem: API fornecida pela empresa terceirizada.

Método de composição: Requisições via classe OrionAPI para obtenção dos dados, seguindo a transformação e persistência incremental realizada na classe DataCollectionService.

A modelagem é rigorosamente documentada por meio dos schemas definidos no dicionário configTables e pela organização das classes que manipulam os dados, o que garante a consistência e a integração entre as tabelas no ambiente do Lakehouse e no Power BI.

Para facilitar a análise e visualização dos dados, foi construído um modelo semântico no ambiente do Power BI. O modelo adota uma estrutura inspirada em um Esquema Estrela (Star Schema) com as seguintes principais tabelas:

Tabela Fato – Medições de Sensor (Fato_Chuva):

sensorId

softSensorId

readingDate (data/hora da leitura)

sensorValue (valor medido – precipitação em mm)

deviceName

deviceId

sensorType

Tabelas Dimensão – Dispositivos e Datas:

Dim_Dispositivo:

deviceId, deviceName, latitude, longitude, status, último upload, etc.

Dim_Tempo:

Data, Mês, Ano, Dia da Semana, Hora (para facilitar análises temporais)


### 4. Carga

A carga dos dados foi implementada para garantir que os dados coletados sejam armazenados de forma consistente e incremental na nuvem. As abordagens adotadas são:

Armazenamento Inicial e Atualização:

  1. overwriteData: Utilizado para a atualização completa de dados, como na carga inicial da tabela de dispositivos.

  2. mergeData: Método de merge incremental que verifica se os dados existem e atualiza os registros correspondentes ou insere novos registros, sem a necessidade de sobrescrever todo o conjunto de dados diariamente. Essa estratégia preserva o histórico e mantém a performance da consulta.

Tecnologia Utilizada:

  Delta Lake: Suporta operações de merge, atualização e manutenção de versões dos dados, proporcionando confiabilidade e performance no Lakehouse do Microsoft Fabric.

Essa etapa está bem documentada no código, onde as classes Repository e DataCollectionService executam as operações de carga, assegurando a persistência e a integridade dos dados na nuvem.

### 5. Análise

#### 5.1 Análise de Qualidade dos Dados:
Tratamento e Validação:

  Conversão de strings para datetime para os campos de data.

  Remoção de duplicatas nos DataFrames através de dropDuplicates().

  Verificação dos sensores, ignorando aqueles classificados como "Unallocated" para assegurar a relevância dos dados.

#### 5.2 Garantia de Consistência:
O processo de parsing e a utilização dos schemas definidos garantem que os dados estejam sempre consistentes, possibilitando análises precisas sem interferência de dados mal formatados.

Solução do Problema e Discussão:
A solução implementada permite que os dados dos pluviômetros sejam coletados de forma automática e integrada, agregando informações de medição e status dos dispositivos em um único repositório.

#### 5.3 Visualizações e Dashboards:
No Power BI, foram criados dashboards que possibilitam:

  1. Visualização por mapa (exibindo a localização dos dispositivos);

  2. Gráficos mostrando os gatilhos dos alertas diários e semanais de chuva;

  3. Tabelas detalhadas com informações operacionais dos dispositivos (status, último upload, bateria, etc.);

Análises temporais diárias de precipitação para cada sensor.

Benefícios:
A solução possibilita identificar de forma rápida eventos críticos e falhas nos dispositivos, permitindo ações preventivas e otimizando a gestão dos ativos da ferrovia.

A análise final dos dados, juntamente com as visualizações desenvolvidas, demonstra que o problema identificado (dependência exclusiva de e-mails para alertas e a dificuldade de monitoramento centralizado) foi solucionado com uma abordagem integrada e automatizada.

### 6. Autoavaliação 
Reflexão sobre o Projeto:

Pontos Fortes:

Foi um otimo desafio, apesar de trabalhar diretamente com PowerBI a alguns anos nunca havia ultilizado diversos recursos do Fabric, como o DataLake, Notebook, Papiline... Com isso juntei o util ao agradavel desenvolvendo um projeto para atender o MVP proposto quanto meu trabalho profissional. 

Desafios Encontrados:

Ajustar os formatos de data e validação dos dados provenientes da API.

Garantir que as operações de merge incremental atualizem os dados sem afetar a performance do Lakehouse.

Criar uma modelagem de dados que atendesse às necessidades de visualização e análise sem perder a integridade dos dados.

Lições e Trabalhos Futuros:

Explorar análises preditivas para antecipar eventos críticos.

Ampliar o período de análise para obter insights históricos mais detalhados.

### 7. Capricho 

  1.  Utilização de boas práticas de programação, modularização e documentação das classes e funções.

  2. Desde a coleta e persistência dos dados até a análise final via dashboards, cada etapa foi implementada visando manter a consistência, performance e integridade dos dados.

  3. O modelo semântico e a organização dos dados facilitam a criação de visualizações intuitivas e interativas no Power BI, o que potencializa a análise dos dados e a identificação de problemas e oportunidades.

  4. O pipeline foi desenhado de forma a permitir a expansão futura (ex: incorporação de novos sensores ou ajustes na modelagem de dados) e reproduzir o processo em outros contextos, mantendo os benefícios do uso incremental e da plataforma de nuvem.

## Conclusão
O projeto atendeu aos requisitos propostos, demonstrando a viabilidade de monitorar em tempo real os dados dos pluviômetros e integrá-los a um dashboard interativo. Através de um pipeline robusto e incremental, foi possível melhorar o controle operacional e a gestão dos alertas, possibilitando ações mais rápidas e precisas no monitoramento dos ativos ferroviários.
