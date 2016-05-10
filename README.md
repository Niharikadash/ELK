# ELK


For elk setup first we have to install JAVA and export JAVA_HOME and JAVA_PATH variables.

  $ yum install -y java-1.8.0-openjdk
  $ alternative --config java   #To set java-1.8.0-openjdk defalt 

Now install Elasticsearch 

  $ wget https://download.elastic.co/elasticsearch/elasticsearch/elasticsearch-1.7.2.noarch.rpm
  $ yum install elasticsearch-1.7.2.noarch.rpm -y

Configure below configuration on /etc/elasticsearch/elasticerch.yml
  cluster.name: ND-elasticsearch
  node.name: "ip-10-10-77-219.ec2.internal"
  network.bind_host: 0
  network.host: localhost
  transport.tcp.port: 9300
  http.cors.enabled: true
  http.cors.allow-origin: "*"
  
And also install elasticsearch-head plugin which allow us to see overview of elasticsearch on browser using http://52.87.179.184:9200/_plugin/head/ where 52.87.179.184 is the public IP of instance.

  $ cd /usr/share/elasticsearch/
  $ ./bin/plugin -install mobz/elasticsearch-head

Now we will logstash which will take the input and format it and send it to elasticsearch

  $ wget https://download.elastic.co/logstash/logstash/packages/centos/logstash-2.3.2-1.noarch.rpm
  $ yum install logstash-2.3.2-1.noarch.rpm

Then configure logstash to take input from stdin and output to elasticsearch. 
Add following entries to /etc/logstash/conf.d/logstash.conf file :

  input { stdin { } }

  filter {
    grok {
     match => { "message" => "%{COMBINEDAPACHELOG}" }
    }
    date {
      match => [ "timestamp" , "dd/MMM/yyyy:HH:mm:ss Z" ]
   }
  }

  output {
   elasticsearch { hosts => ["localhost:9200"] }
   stdout { codec => rubydebug }
  }
  
Now start the elasticsearch and logstash
  $ service elasticsearch start
  $ service logstash start

And atlast installing Kibana

  $ wget https://download.elastic.co/kibana/kibana/kibana-4.1.2-linux-x64.tar.gz
  $ tar xzf kibana-4.1.2-linux-x64.tar.gz
  $ cd kibana-4.1.2-linux-x64
  $ nohup ./bin/kibana &
  
Verify using netstat command that elasticsearch is listening on port 9200 and 9300 and Kibana on port 5601.
  $ netstat -tunapl |grep LISTEN
      tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1437/sshd           
      tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1124/master         
      tcp        0      0 0.0.0.0:5601            0.0.0.0:*               LISTEN      2789/./../node/bin/ 
      tcp6       0      0 :::9200                 :::*                    LISTEN      5040/java           
      tcp6       0      0 :::9300                 :::*                    LISTEN      5040/java           
      tcp6       0      0 :::9301                 :::*                    LISTEN      765/java            
      tcp6       0      0 :::22                   :::*                    LISTEN      1437/sshd           
      tcp6       0      0 ::1:25                  :::*                    LISTEN      1124/master
  


  

