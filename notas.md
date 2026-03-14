Conhecendo um pouco mais sobre os tipos de dados do Prometheus

Ao criamos o exporter, pecisamos saber qual o tipo de dado a ser usado;

Temos quatro:
 
gauge: Medidor
O tipo de dado gauge é o tipo de dado utilizado para criar métricas que podem ter seus valores alterados para cima ou para baixo, por exemplo, a 
ultilização de memória ou cpu. Se quiser trazer para exemplos da vida real, podemos falar que aquelas filas que você odeia é o tipo de dado gauge, ou 
então a temperatura da sua cidade, ela pode ser alterada para cima ou para baixo, ou seja, é um medidor, é um gauge! :D Um exemplo de métrica do tipo 
gauge é a métrica memory_usage, que é uma métrica que mostra a utilização de memória.
memory_usage{instance="localhost:8899",job="Primeiro Exporter"}
 


counter: Contador
O tipo de dado counter é o tipo de dado utilizado que vai ser incrementado no decorrer do tempo, por exemplo, quando eu quero contar os erros em uma 
aplicação no decorrer da última hora. O valor atual do counter quase nunca é importante, pois o que queremos dele são os valores durante uma janela de tempo, por 
exemplo, quantas vezes a minha aplicação falhou durante o final de semana. Normalmente as métricas counter possuem o sufixo _total para indicar que é o total de 
valores que foram contados, por exemplo:
requests_total{instance="localhost:8899",job="Primeiro Exporter"}
 


histogram: Histograma
O tipo de dado histogram é o tipo de dado que te permite especificar o seu valor através de buckets predefinidos, por exemplo, o tempo de execução de uma 
aplicação. Com o histogram eu consigo contar todas as requisições que minha aplicação respondeu entre 0 e 0,5 segundos, ou então as requisições que tiveram 
respostas entre 1,0 e 2,5 e assim por diante. Por padrão, os buckets predefinidos são até no máximo 10 segundos, se você quiser mais, você pode criar seus 
próprios buckets personalizados. Um coisa super importante, o Prometheus irá contar cada item em cada bucket, e também a soma dos valores. Uma métrica do tipo 
histogram inclui alguns itens importantes são adicionados ao final do nome da métrica para indicar o tipo de dado e o tamanho do bucket, por exemplo:
requests_duration_seconds_bucket{le="0.5"}
Onde le é o valor limite do bucket, o valor 0.5 indica que o valor do bucket é até 0,5 segundos, ou seja, aqui nesse bucket poderiam estar os requisições que 
tiveram respostas entre 0 e 0,5 segundos.
O _bucket é um sufixo que indica que o valor é um bucket.

Ainda temos alguns sufixos que são importantes e que podem ser úteis para nós:

_count: Contador
O sufixo _count indica que o valor é um contador, ou seja, o valor é incrementado a cada vez que a métrica é atualizada.

_sum: Soma
O sufixo _sum indica que o valor é uma soma, ou seja, o valor é somado a cada vez que a métrica é atualizada.

_bucket: Bucket
O sufixo _bucket indica que o valor é um bucket, ou seja, o valor é um bucket.
O ponto alto do histogram é a excelente flexibilidades, pois percentuais e as janelas de tempos podem definidas durante a criação das queries, o ponto 
negativo é a precisão é um pouco inferior quando comparado com o summary.
 

summary: Resumo
O tipo de dado summary é bem parecido com o histogram, com a diferença que os buckets, aqui chamados de quantiles, são definidos por um valor entre 0 e 1, ou 
seja, o valor do bucket é o valor que está entre os quantiles.
Da mesma forma como no histogram, podemos criar métricas do tipo summary com alguns itens importantes adicionados ao final do nome da métrica, por exemplo:
requests_duration_seconds_sum{instance="localhost:8899",job="Primeiro Exporter"}

Utilizamos o sufixo _sum indica que o valor é uma soma, ou seja, o valor é somado a cada vez que a métrica é atualizada e o 

sufixo _count para indicar que o valor é um contador, ou seja, o valor é incrementado a cada vez que a métrica é atualizada.
O ponto alto do summary é a excelente precisão e o ponto baixo é a baixa flexibilidades, pois percentuais e as janelas de tempos precisam ser definidos 
durante a criação da métrica e não é possível agregar métricas do tipo summary com outras métricas do tipo summary durante a criação das queries.

<br>

Criei um container do Prometheus para realizar o monitoramento, expondo a interface web na porta 9090. Utilizei um arquivo de configuração (prometheus.yml) armazenado na minha máquina e configurei um volume persistente (prometheus_data) para garantir que os dados coletados não sejam perdidos caso o container seja reiniciado ou recriado.

Como estou utilizando os exporters e o Prometheus em containers separados, precisei criar uma rede Docker dedicada (monitor-net) para permitir a comunicação entre eles. Após criar a rede, conectei manualmente os containers do Prometheus e dos exporters a essa rede, possibilitando que o Prometheus acesse os endpoints de métricas de cada serviço pelo nome dos containers.


# Criação do container do prometheus
docker run -d \
    -p 9090:9090 \
    -v /home/tamiris/go/exportes/exporter-python/prometheus.yml:/etc/prometheus/prometheus.yml \
    -v prometheus_data:/prometheus \
    prom/prometheus \
    --config.file=/etc/prometheus/prometheus.yml \
    --storage.tsdb.path=/prometheus

Como estou utilizando containers separados tive que criar uma rede e conectar os conteiners nessa rede

# criando a rede
docker network create monitor-net

# conectando os containers na rede criada
docker network connect monitor-net distracted_mclaren
docker network connect monitor-net segundo-exporter
docker network connect monitor-net primeiro-exporter


# Funções

A função rate
A função rate representa a taxa de crescimento por segundo de uma determinada métrica como média, durante um intervalo de tempo.

rate(metrica)[5m]
 Onde metrica é a métrica que você deseja calcular a taxa de crescimento durante um intervalo de tempo de 5 minutos. Você pode utilizar a função rate para trabalhar com métricas do tipo gauge e counter.

Vamos para um exemplo real:

rate(prometheus_http_requests_total{job="prometheus",handler="/api/v1/query"}[5m])
 

Aqui estou calculando a média da taxa de crescimento por segundo da métrica prometheus_http_requests_total, filtrando por job e handler e durante um intervalo de tempo de 5 minutos. Nesse caso eu quero saber o crescimento nas queries que estão sendo feitas no Prometheus.

  


A função irate
A função irate representa a taxa de crescimento por segundo de uma determinada métrica, mas diferentemente da função rate, a função irate não faz a média dos valores, ela pega os dois últimos pontos e calcula a taxa de crescimento. Quando representado em um gráfico, é possível ver a diferença entre a função rate e a função irate, enquanto o gráfico com o rate é mais suave, o gráfico com o irate é mais "pontiagudo", você consegue ver quedas e subidas mais nítidas.

irate(metrica)[5m]
 

Onde metrica é a métrica que você deseja calcular a taxa de crescimento, considerando somente os dois últimos pontos, durante um intervalo de tempo de 5 minutos.

Vamos para um exemplo real:

irate(prometheus_http_requests_total{job="prometheus",handler="/api/v1/query"}[5m])
 

Aqui estou calculando a taxa de crescimento por segundo da métrica prometheus_http_requests_total, considerando somente os dois últimos pontos, filtrando por job e handler e durante um intervalo de tempo de 5 minutos. Nesse caso eu quero saber o crescimento nas queries que estão sendo feitas no Prometheus.

<br>

A função delta
A função delta representa a diferença entre o valor atual e o valor anterior de uma métrica. Quando estamos falando de delta estamos falando por exemplo do consumo de um disco. Vamos imaginar que eu queira saber o quando eu usei de disco em um determinado intervalo de tempo, eu posso utilizar a função delta para calcular a diferença entre o valor atual e o valor anterior.

delta(metrica[5m])
 

Onde metrica é a métrica que você deseja calcular a diferença entre o valor atual e o valor anterior, durante um intervalo de tempo de 5 minutos.

 

Vamos para um exemplo real:

delta(prometheus_http_response_size_bytes_count{job="prometheus",handler="/api/v1/query"}[5m])
Agora estou calculando a diferença entre o valor atual e o valor anterior da métrica prometheus_http_response_size_bytes_count, filtrando por job e handler e durante um intervalo de tempo de 5 minutos. Nesse caso eu quero saber o quanto de bytes eu estou consumindo nas queries que estão sendo feitas no Prometheus.

  


A função increase
Da mesma forma que a função delta, a função increase representa a diferença entre o primeiro e último valor durante um intervalo de tempo, porém a diferença é que a função increase considera que o valor é um contador, ou seja, o valor é incrementado a cada vez que a métrica é atualizada. Ela começa com o valor 0 e vai somando o valor da métrica a cada atualização. Você já pode imaginar qual o tipo de métrica que ela trabalha, certo? Qual? Counter!

increase(metrica[5m])
 

Onde metrica é a métrica que você deseja calcular a diferença entre o primeiro e último valor durante um intervalo de tempo de 5 minutos.

 

Vamos para um exemplo real:

increase(prometheus_http_requests_total{job="prometheus",handler="/api/v1/query"}[5m])
 

Aqui estou calculando a diferença entre o primeiro e último valor da métrica prometheus_http_requests_total, filtrando por job e handler e durante um intervalo de tempo de 5 minutos.

Você pode acompanhar o resultado dessa query clicando em Graph e depois em Execute, assim você vai ver o gráfico com o resultado da query fazendo mais sentindo.

<br>


A função sum
A função sum representa a soma de todos os valores de uma métrica. Você pode utilizar a função sum nos tipos de dados counter, gauge, histogram e summary. Um exemplo de uso da função sum é quando você quer saber o quanto de memória está sendo utilizada por todos os seus containers, ou o quanto de memória está sendo utilizada por todos os seus pods.

sum(metrica)
 

Onde metrica é a métrica que você deseja somar.

 

Vamos para um exemplo real:

sum(go_memstats_alloc_bytes{job="prometheus"})
 

Aqui estou somando todos os valores da métrica go_memstats_alloc_bytes, filtrando por job e durante um intervalo de tempo de 5 minutos.

  


A função count
Outra função bem utilizada é função count representa o contador de uma métrica. Você pode utilizar a função count nos tipos de dados counter, gauge, histogram e summary. Um exemplo de uso da função count é quando você quer saber quantos containers estão rodando em um determinado momento ou quantos de seus pods estão em execução.

count(metrica)
 

Onde metrica é a métrica que você deseja contar.

 

Vamos para um exemplo real:

count(prometheus_http_requests_total)
 

Teremos como resultado o número de valores que a métrica prometheus_http_requests_total possui.


A função avg
A função avg representa o valor médio de uma métrica. Você pode utilizar a função avg nos tipos de dados counter, gauge, histogram e summary. Essa é uma das funções mais utilizadas, pois é muito comum você querer saber o valor médio de uma métrica, por exemplo, o valor médio de memória utilizada por um container.

avg(metrica)
 

Onde metrica é a métrica que você deseja calcular a média.

  


A função min
A função min representa o valor mínimo de uma métrica. Você pode utilizar a função min nos tipos de dados counter, gauge, histogram e summary. Um exemplo de uso da função min é quando você quer saber qual o menor valor de memória utilizada por um container.

min(metrica)
 

Onde metrica é a métrica que você deseja calcular o mínimo.

  


A função max
A função max representa o valor máximo de uma métrica. Um exemplo de uso da função max é quando você quer saber qual o maior valor de memória pelos nodes de um cluster Kubernetes.

max(metrica)
 

Onde metrica é a métrica que você deseja calcular o máximo.




