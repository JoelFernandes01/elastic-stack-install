# Como instalar o Elastic Stack no Ubuntu 24.04 LTS
Neste artigo, aprenderemos como  instalar o Elastic Stack no  Ubuntu 24.04 LTS.</br>
O ELK Stack consiste em Elasticsearch, Logstash e Kibana, com o Filebeat frequentemente usado para enviar logs para o Logstash.</br> Essa combinação poderosa é essencial para registro centralizado, visualização de dados e análise em tempo real.</br> Nós o guiaremos pela instalação e configuração de cada componente do ELK stack e verificaremos sua configuração.</br>
Esse tipo de instalação é apenas para conhecimento e testes da ferramenta, não é recomendável instalar toda a solução em um só servidor.

## Pré-requisitos
Estou usando nesse labotatório o Ubuntu 24.04 LTS.
Pelo menos 2 núcleos x 2 Sockets de CPU e 8 GB de RAM para um desempenho bom/rasoável.

Update da distribuição e logo em seguida, instale o pacote apt-transport-https para acessar o repositório via HTTPS.
```
sudo apt update -y
```
![Descrição da imagem](/pictures/update.PNG)
```
sudo apt install apt-transport-https
```
![Descrição da imagem](/pictures/transports-https.PNG)
## Etapa 1: Instalar Java para Elastic Stack no Ubuntu 24.04 LTS
Os componentes do Elastic Stack exigem Java. Instalaremos o OpenJDK 11, que é uma implementação de código aberto amplamente usada da Plataforma Java.</br>
```
sudo apt install openjdk-11-jdk -y
```
![Descrição da imagem](/pictures/openjdk-11-jdk.PNG)
Após a instalação, verifique se o Java está instalado corretamente verificando sua versão.</br>
```
java -version
```
Vamos configurar a variável de ambiente para o Java, precisamos definir a JAVA_HOMEvariável de ambiente.</br>
Abra o arquivo de ambiente.
```
sudo vim /etc/environment</br>
Adicione a seguinte linha no final do arquivo.</br>
JAVA_HOME="/usr/lib/jvm/java-11-openjdk-amd64"
```
```
source /etc/environment
```
```
echo $JAVA_HOME
```
![Descrição da imagem](/pictures/environment.PNG)
## Etapa 2: Instalar o ElasticSearch no Ubuntu 24.04 LTS
Elasticsearch é o componente principal do ELK Stack, usado para pesquisa e análise.</br>
Precisamos importar a chave de assinatura pública e adicionar o repositório Elasticsearch APT ao seu sistema.
```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
```
Adicione a definição do repositório.
```
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
````
Atualize as listas de pacotes novamente para incluir o novo repositório Elasticsearch.
````
sudo apt update -y
````
Instale o Elasticsearch.
````
![Descrição da imagem](/pictures/elastic-keys.PNG)
sudo apt-get install elasticsearch -y
````
![Descrição da imagem](/pictures/elastic-install.PNG)
Inicie o Elasticsearch e configure-o para ser executado na inicialização do sistema.
````
sudo systemctl enable --now elasticsearch
````
Verifique se o Elasticsearch está em execução.
````
sudo systemctl status elasticsearch
````
![Descrição da imagem](/pictures/elastic-status.PNG)
## Etapa 3: Configurar o Elasticsearch no Ubuntu 24.04 LTS
````
sudo vim /etc/elasticsearch/elasticsearch.yml
````
Encontre dentro do arquivo network.host, descomente-a e defina-a com o "<your-server-ip>" também descomente o discovery para especificar os nós iniciais para formação do cluster "discovery.seed_hosts: []"
![Descrição da imagem](/pictures/elastic-network.PNG)

Para uma configuração básica (não recomendada para produção), desative os recursos de segurança.
![Descrição da imagem](/pictures/elastic-security.PNG)
````
sudo systemctl restart elasticsearch
````
````
curl -X GET "localhost:9200"
````
![Descrição da imagem](/pictures/elastic-get-localhost.PNG)
![Descrição da imagem](/pictures/elastic-http-localhost.PNG)

## Etapa 4: Instalar o Logstash no Ubuntu 24.04 LTS
````
sudo apt-get install logstash -y
````
![Descrição da imagem](/pictures/logstash-install.PNG)
````
sudo systemctl enable --now logstash
````
````
sudo systemctl status logstash
````
````
## Etapa 5: Instalar o Kibana no Ubuntu 24.04 LTS
sudo apt-get install kibana
````
![Descrição da imagem](/pictures/kibana-install.PNG)
````
sudo systemctl enable --now kibana
````
## Etapa 6: Configurar o Kibana no Ubuntu 24.04 LTS
Para configurar o Kibana para acesso externo, edite o arquivo de configuração.
````
sudo vim /etc/kibana/kibana.yml
````
````
sudo systemctl restart kibana
````
````
sudo systemctl status kibana
````
````
sudo systemctl status logstash
````
````
sudo systemctl status elasticsearch
````
## Etapa 7: Instalar o Filebeat no Ubuntu 24.04 LTS
Filebeat é um carregador leve usado para encaminhar e centralizar dados de log.</br>
Instale Filebeat usando o comando a seguir.
````
sudo apt-get install filebeat -y
````
![Descrição da imagem](/pictures/filebeat-install.PNG)

Abra o arquivo de configuração do Filebeat para enviar logs para  o Logstash.</br>
Comente a seção de saída  do Elasticsearch .
````
# saída.elasticsearch: 
# hosts: ["localhost:9200"]
Descomente e configure a seção de saída do Logstash.

saída.logstash: 
  hosts: ["localhost:5044"]
````
````
sudo vim /etc/filebeat/filebeat.yml
````
![Descrição da imagem](/pictures/filebeat-output.PNG)
````
sudo filebeat modules enable system
````
````
sudo filebeat setup --index-management -E output.logstash.enabled=false -E 'output.elasticsearch.hosts=["0.0.0.0:9200"]'
````
![Descrição da imagem](/pictures/filebeat-setup.PNG)
````
sudo systemctl enable --now filebeat
````
Certifique-se de que o Elasticsearch está recebendo dados do Filebeat verificando os índices.</br>
Você deverá ver uma saída indicando a presença de índices criados pelo Filebeat.</br>
````
curl -XGET "localhost:9200/_cat/indices?v"
````
![Descrição da imagem](/pictures/filebeat-curl.PNG)
Você pode acessá-lo usando o navegador http://<your-server-ip>:9200/_cat/indices?v
![Descrição da imagem](/pictures/filebeat-http.PNG)


## Conclusão:
![Descrição da imagem](/pictures/elastic-http.PNG)
![Descrição da imagem](/pictures/elastic-http-management.PNG)

Concluindo, instalamos e configuramos com sucesso o Elastic Stack no Ubuntu 24.04 LTS.</br>
Isso incluiu a configuração do Elasticsearch para pesquisa e análise, Logstash para processamento de dados, Kibana para visualização de dados e Filebeat para envio de logs.</br>
O Elastic Stack fornece uma solução robusta para registro centralizado e análise de dados, tornando-o inestimável para monitorar e analisar o desempenho do sistema e logs de aplicativos.