# Curso Descomplicando Prometheus 

Minha jornada no aprendizado do sistema de monitoramento prometheus sendo realizado com a monitoria do Jeferson Fernando da Linuxtips.

## Porque devemos Monitorar?

Devemos monitorar para saber o que acontece na infraestrutura, na aplicação, no serviço em tempo real. Para caso ocorra alguma falha ou anomalia
as pessoas responsáveis, sejam notificadas com o máximo de informação para avaliar e fazer o troubleshooting para resolução adequada.

Além de podemos ir além do monitoramento tradicional e expandir para o conceito de observabilidade.

O que seria a OBSERVABILIDADE é uma capacidade que tem de analizar, explorar e coletar métricas de ambientes descentralizados ou em nuvem. 
Entendendo assim os padrões e extraindo dados sobre o funcionamento, gerando alertas, dashboards que ajudam a solucionar e resolver incidentes 
de performance, disponibilidade e segurança, melhorando a experiência do usuário final.


A Observabilidade esta apoiada em três pilares: LOGS, MÉTRICAS E TRACES

LOGS são os registros brutos de alguma ocorrẽncia dentro do sistema, ele é um facilitador na identificação e na investigação.

MÉTRICAS são informações retiradas dos logs que juntos é possivel mensurar atráves de relatórios ou dashboards

TRACES é a maneira de rastrear todo o fluxo de informações,coletando parâmetros e agregando os mesmos. 

Podemos incluir também os EVENTOS para sabemos o que esta acontecendo e os DASHBOARDS para a visualização.

![meio-2](https://user-images.githubusercontent.com/13388615/190480508-bde6ef5a-700a-41e9-be78-2a58b0016c21.jpg)


## Introdução

Prometheus é um sistema de captura e armazenamento de métricas, com a facilidade da integração com outros sistemas.
Ele foi inspirado no sistema de monitoramento do cluster kubernetes da google o BORG.

Ele surgiu em 2012 e foi escrito na linguagem GO, pelos engenheiros da SoundCloud.

Uma dos principais ponto do prometheus é que ele vai até o alvo para coletar as métricas,e depois ele guarda no banco de dados timeseries. 
Mas também é possivel que ele receba essas informações atráves do PUSH GATEWAY.



## Prometheus Monitoring Architecture

Temos três componentes principais no prometheus que são: Retrieval, Storage e PromQL

RETRIEVAL é o que coleta as métricas e armazena no Storage é também o que conversa diretamente com o Service Discovery para descoberta de novos serviços
disponiveis que ainda não estão sendo coletados.

STORAGE é o banco de dados de séries temporais que armazena as métricas coletadas.

Banco de Dados Temporais
Um banco de dados de séries temporais(Time series database) é um sistema usado para armazenar séries temporais através de 
pares de tempo e valor associados.  Em alguns campos, as séries temporais podem ser chamadas de perfis, curvas, traços ou ainda, tendências. 

PROMQL é a linguagem para consulta responsável por executar as queries do usuário. Ela não é parecida com o SQL.


![Prometheus](https://user-images.githubusercontent.com/13388615/190471394-c5fbb82a-bcc8-486f-a2e9-9c72993874d9.png)




