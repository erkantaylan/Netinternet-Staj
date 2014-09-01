Elasticsearch-Logstash-Kibana
=============================

Logstash logları almak, işlemek ve bunları sunmak için bir programdır. Her türden logları kullanabilir. Sistem logları, web sunucu logları, hata logları ve uygulama loglarını tutar, istediğiniz tür log gönderebilirsiniz.

Logları kaydetmek için elasticsearch ve raporlama için kibanayı kullanacağız. Logstash her ne kadar çok etkili bir tool olsada elasticsearch, kibana gibi yan uygulamar olmadan pek kullanışlı değildir.

Logstash: gelen logları inceleyen server bileşeni
Elasticsearch: logları tutmak için
Kibana: Web arayüz
Logstash Forwarder: servera yüklendiğinde logları logstash'e gönderir. lumberjack network protokoünü kullanarak logstashle iletişime geçer. 
Not: Logstash Forwarder'ı logstashle aynı makineye kurmak zorunda değiliz. Logları alacağımız makineye bunu kurup logları başka yere gönderebiliriz.

Kurulumu ubuntu 12.04 x64 üzerinde gerçekleştirdim.
Elasticsearch-Logstash-Kibana (ELK) Kurulum
===========================================
Logstash JRuby ile yazıldığı için java istemektedir ama ruby kurmanıza gerek yoktur.

Java 7 Kurulum
=============
Java 7 olmasının sebebe elasticsearch için önerilen java versiyonu olmasıdır.

Depoya javayı ekle

> sudo add-apt-repository -y ppa:webupd8team/java

Servera kurarken üstteki kod çalışmıyor. Bunun için

> sudo apt-get update 

> sudo apt-get install apt-file 

> sudo apt-file update 

> sudo apt-get install python-software-properties 

Depoyu güncelle

> sudo apt-get update

Java indir ve yükle

> sudo apt-get -y install oracle-java7-installer

Elasticsearch Kurulum
=====================
Note: Logstash 1.4.2 de önerilen Elasticsearch 1.1.1

> wget -O - http://packages.elasticsearch.org/GPG-KEY-elasticsearch | sudo apt-key add -

> echo 'deb http://packages.elasticsearch.org/elasticsearch/1.1/debian stable main' | sudo tee /etc/apt/sources.list.d/elasticsearch.list

> sudo apt-get update

> sudo apt-get -y install elasticsearch=1.1.1

Elasticsearchü kurduk ve şimdi editleyelim.

> sudo nano /etc/elasticsearch/elasticsearch.yml

Dinamik scriptleri disable edelim. (Boş bir yere ekleyiniz.)
> script.disable_dynamic: true

Aşağıdakini bulup yorum satırını kaldıralım. (# network.host: localhost)


> network.host: localhost

localhost bazen çalışmaybiliyor, onun yerine 127.0.0.1 de yazabiliriz yerine. Kaydedip çıkabiliriz.

> sudo service elasticsearch restart

Elasticsearch'ü başlatalım açılışta otomatik başlatmak için:

> sudo update-rc.d elasticsearch defaults 95 10

Kibana kurulumu
===============
Logstash 1.4.2 için kibana sürümü 3.0.1

> cd ~; wget https://download.elasticsearch.org/kibana/kibana/kibana-3.0.1.tar.gz
tar xvf kibana-3.0.1.tar.gz
Kibana configi düzenleyelim.
sudo nano ~/kibana-3.0.1/config.js
Kibananın default portunu değiştirelim. Aşağıdaki satırı bulup 9200 yerine 80 yapalım ki tarayıcıdan girdiğimizde doğrudan olaşabilelim (localhost:9200 yerine).


elasticsearch: "http://"+window.location.hostname+":80",


Kibanayı kurduk ve ayarladık ama kibananın çalışabilmesi için nginx'i kurmamız lazım. Aşağıdaki kodları sırayla çalıştıralım.
sudo mkdir -p /var/www/kibana3


sudo cp -R ~/kibana-3.0.1/* /var/www/kibana3/
Install Nginx
sudo apt-get install nginx
cd ~; wget https://github.com/elasticsearch/kibana/raw/master/sample/nginx.conf
sudo nano nginx.conf



server_name i bulup FQDN yerine localhost yapalım. (varsa domain adı)

server_name FQDN;

root /var/www/kibana3;

Üstteki değişikliği de yaptıktan sonra kaydedip çıkabiliriz.



Şimdi apache2-utils kuracağız ki doğrudan serverımıza herkes ulaşıp kibanamızı bozmasın.

sudo service nginx restart
Install Logstash
Sonunda geldik logstash kurulumuna.

Logstash source list yapalım önce

sudo apt-get update
sudo apt-get install logstash=1.4.2-1-2c0f5a1
SSL Sertifikası Oluşturma
Logstash Forwarderten logları alırken SSL sertifikası oluşturmamız lazım.

Aşağıdaki kodları takip edin.

sudo mkdir -p /etc/pki/tls/certs
sudo mkdir /etc/pki/tls/private
cd /etc/pki/tls; sudo openssl req -x509 -batch -nodes -days 3650 -newkey rsa:2048 -keyout private/logstash-forwarder.key -out certs/logstash-forwarder.crt
logstash-forwarder.crt bütün serverlara kopyalandı ve log göndermeye hazır ama bunu biraz sonra yapacağız. Şimdi logstashımızın ayarlarımı yapalım.


logstash forwarder protokol için

yazıp kaydedelim. lumberjack 5000 numaralı portu dinleyecek ve bizim SSL sertifikamızı kullanacak.
Syslog mesajlarımızı almak ve filtrelemek için;
sudo nano /etc/logstash/conf.d/10-syslog.conf

Bunların dışında config dosyası oluşturacağımız zaman dosya isimlerinin 01 ile 30 arasında olmasına dikkat etmeliyiz.
Logstashe bir restart atalım
Logstash Forwarder Kurulumu
SSL sertifikamızı Logstash Forwarder Paketine kopyalayım.

olusturduğumuz kullanıcı adı(user) ve (server_private_ip)server ip sini girmeyi unutmayalım

Logstashi paket liste ekleyelim

Eğer 32 bitse ubuntunuz hata alacaksınız. bunun içinde aşağıdaki kodları kullanabilirsiniz. (Kendim denemedim.)
wget https://assets.digitalocean.com/articles/logstash/logstash-forwarder_0.3.1_i386.deb
sudo dpkg -i logstash-forwarder_0.3.1_i386.deb
Buraya kadar başarıyla geldiysek. Logstash Forwarderımızın rebootten sonra tekrar otomatik çalışması için de aşağıdaki kodları çalıştıralım.
cd /etc/init.d/; sudo wget https://raw.github.com/elasticsearch/logstash-forwarder/master/logstash-forwarder.init -O logstash-forwarder
sudo chmod +x logstash-forwarder
sudo update-rc.d logstash-forwarder defaults
SSL sertifikayı /etc/pki/tls/certs e kopyalalım
sudo mkdir -p /etc/pki/tls/certs
sudo cp /tmp/logstash-forwarder.crt /etc/pki/tls/certs/
Şimdi sıra Logstash Forwarder'ızın config dosyasına geldi
sudo nano /etc/logstash-forwarder 
Logstash private ip mizi ilgili yere yazıp kaydedelim

{
  "network": {
    "servers": [ "logstash_server_private_IP:5000" ],
    "timeout": 15,
    "ssl ca": "/etc/pki/tls/certs/logstash-forwarder.crt"
  },
  "files": [
    {
      "paths": [
        "/var/log/syslog",
        "/var/log/auth.log"
       ],
      "fields": { "type": "syslog" }
    }
   ]
}
Bu config Logstash  Forwarderı 5000 nolu portu kullarak server'ımıza bağlar.

sudo service logstash-forwarder restart
servise restart attıktan sonra loglarımız düşmeye başlamış olması lazım. web browserdan localhost veya adresimizi yazdıktan sonra kibana sayfamızı görmemiz lazım. Buradan Logstash Dashboard a tıklayınca 






echo 'deb http://packages.elasticsearch.org/logstashforwarder/debian stable main' | sudo tee /etc/apt/sources.list.d/logstashforwarder.list
ve kuralım
sudo apt-get update
sudo apt-get install logstash-forwarder
scp /etc/pki/tls/certs/logstash-forwarder.crt user@server_private_IP:/tmp
sudo service logstash restart
filter {
  if [type] == "syslog" {
    grok {
      match => { "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:syslog_hostname} %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:syslog_message}" }
      add_field => [ "received_at", "%{@timestamp}" ]
      add_field => [ "received_from", "%{host}" ]
    }
    syslog_pri { }
    date {
      match => [ "syslog_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
    }
  }
}
yazıp kaydedelim. Bu ayar dosyası gelen syslog mesajını grok standartına göre filtreleyelip parse eder ve daha sorgulanabilir hale getirir.
Son olarak;

sudo nano /etc/logstash/conf.d/30-lumberjack-output.conf

output {
  elasticsearch { host => localhost }
  stdout { codec => rubydebug }
}
bu ayarda gelen filtrelenmiş logların çıktısını elasticsearche gönderir.
sudo nano /etc/logstash/conf.d/01-lumberjack-input.conf
içine;

input {
  lumberjack {
    port => 5000
    type => "logs"
    ssl_certificate => "/etc/pki/tls/certs/logstash-forwarder.crt"
    ssl_key => "/etc/pki/tls/private/logstash-forwarder.key"
  }
}
echo 'deb http://packages.elasticsearch.org/logstash/1.4/debian stable main' | sudo tee /etc/apt/sources.list.d/logstash.list
sudo apt-get install apache2-utils
user yazan yere kullanıcı adı yazınız
sudo htpasswd -c /etc/nginx/conf.d/kibana.myhost.org.htpasswd user

