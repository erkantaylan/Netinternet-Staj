Elasticsearch-Logstash-Kibana
=============================

![alt tag](http://logstash.net/images/logstash.png)

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
```
 sudo add-apt-repository -y ppa:webupd8team/java
```

Servera kurarken üstteki kod çalışmıyor. Bunun için
```
sudo apt-get update 
sudo apt-get install apt-file 
sudo apt-file update 
sudo apt-get install python-software-properties 
```
Depoyu güncelle
```
sudo apt-get update
```
Java indir ve yükle
```
sudo apt-get -y install oracle-java7-installer
```
Elasticsearch Kurulum
=====================

Note: Logstash 1.4.2 de önerilen Elasticsearch 1.1.1
```
wget -O - http://packages.elasticsearch.org/GPG-KEY-elasticsearch | sudo apt-key add -
```
```
echo 'deb http://packages.elasticsearch.org/elasticsearch/1.1/debian stable main' | sudo tee /etc/apt/sources.list.d/elasticsearch.list
```
```
sudo apt-get update
```
```
sudo apt-get -y install elasticsearch=1.1.1
```
Elasticsearchü kurduk ve şimdi editleyelim.
```
sudo nano /etc/elasticsearch/elasticsearch.yml
```
Dinamik scriptleri disable edelim. (Boş bir yere ekleyiniz.)

```
script.disable_dynamic: true
```
Aşağıdakini bulup yorum satırını kaldıralım. (# network.host: localhost)

```
network.host: localhost
```
localhost bazen çalışmaybiliyor, onun yerine 127.0.0.1 de yazabiliriz yerine. Kaydedip çıkabiliriz.
```
sudo service elasticsearch restart
```
Elasticsearch'ü başlatalım açılışta otomatik başlatmak için:
```
sudo update-rc.d elasticsearch defaults 95 10
```
Kibana kurulumu
===============
Logstash 1.4.2 için kibana sürümü 3.0.1
```
cd ~; wget https://download.elasticsearch.org/kibana/kibana/kibana-3.0.1.tar.gz
```
```
tar xvf kibana-3.0.1.tar.gz
```
Kibana configi düzenleyelim.
```
sudo nano ~/kibana-3.0.1/config.js
```
Kibananın default portunu değiştirelim. Aşağıdaki satırı bulup 9200 yerine 80 yapalım ki tarayıcıdan girdiğimizde doğrudan olaşabilelim (localhost:9200 yerine).

```
elasticsearch: "http://"+window.location.hostname+":80",
```

Kibanayı kurduk ve ayarladık ama kibananın çalışabilmesi için nginx'i kurmamız lazım. Aşağıdaki kodları sırayla çalıştıralım.

Nginx için uygun bir yere klasör açalım:
```
sudo mkdir -p /var/www/kibana3
```
Dosyalarımızı buraya kopyalayalım:
```
sudo cp -R ~/kibana-3.0.1/* /var/www/kibana3/
```
Install Nginx
=============

Nginx kurulumu
```
sudo apt-get install nginx
```
Githubtan nginx.conf dosyamızı çekelim:
```
cd ~; wget https://github.com/elasticsearch/kibana/raw/master/sample/nginx.conf
```
Nginx conf edit:
```
sudo nano nginx.conf
```
server_name i bulup FQDN yerine localhost yapalım. (varsa domain adı)
```
server_name FQDN;
```
```
root /var/www/kibana3;
```
Üstteki değişikliği de yaptıktan sonra kaydedip çıkabiliriz. Şimdi ayar dosyasını nginx in içine kopyalayalım:
```
sudo cp nginx.conf /etc/nginx/sites-available/default
```
Şimdi apache2-utils kuracağız ki doğrudan serverımıza herkes ulaşıp kibanamızı bozmasın.
```
sudo apt-get install apache2-utils
```
user yazan yere kullanıcı adı yazınız ( kullanıcıyı yazıp enter'a bastıktan sonra bir sizden şifre girmenizi isteyecek)
```
sudo htpasswd -c /etc/nginx/conf.d/kibana.myhost.org.htpasswd user
```
```
sudo service nginx restart
```
Install Logstash
================

Sonunda geldik logstash kurulumuna.

Logstash'ı source'a ekleyelim
```
echo 'deb http://packages.elasticsearch.org/logstash/1.4/debian stable main' | sudo tee /etc/apt/sources.list.d/logstash.list
```
```
sudo apt-get update
```
```
sudo apt-get install logstash=1.4.2-1-2c0f5a1
```
Logstash kurulumu bitti ama ayarını daha yapmadık.

SSL Sertifikası Oluşturma
=========================
Logstash Forwarderten logları alırken SSL sertifikası oluşturmamız lazım.

Aşağıdaki kodları takip edin.
```
sudo mkdir -p /etc/pki/tls/certs
```
```
sudo mkdir /etc/pki/tls/private
```
```
cd /etc/pki/tls; sudo openssl req -x509 -batch -nodes -days 3650 -newkey rsa:2048 -keyout private/logstash-forwarder.key -out certs/logstash-forwarder.crt
```
logstash-forwarder.crt bütün serverlara kopyalandı ve log göndermeye hazır ama bunu biraz sonra yapacağız. Şimdi logstashımızın ayarlarımı yapalım.
```
sudo nano /etc/logstash/conf.d/01-lumberjack-input.conf
```
içine;

```
input {
  lumberjack {
    port => 5000
    type => "logs"
    ssl_certificate => "/etc/pki/tls/certs/logstash-forwarder.crt"
    ssl_key => "/etc/pki/tls/private/logstash-forwarder.key"
  }
}
```

Syslog mesajlarımızı almak ve filtrelemek için;
```
sudo nano /etc/logstash/conf.d/10-syslog.conf
```
içine :
```
filter {
 if [type] == "syslog" {
   grok {
     match => { "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:syslog_hostname} 
%{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:syslog_message}" }
     add_field => [ "received_at", "%{@timestamp}" ]
     add_field => [ "received_from", "%{host}" ]
   }
   syslog_pri { }
   date {
     match => [ "syslog_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
   }
 }
}
```
yazıp kaydedelim. Bu ayar dosyası gelen syslog mesajını grok standartına göre filtreleyelip parse eder ve daha sorgulanabilir hale getirir.
Son olarak;
```
sudo nano /etc/logstash/conf.d/30-lumberjack-output.conf
```
içine :
```
output {
   elasticsearch { host => localhost }
   stdout { codec => rubydebug }
}
```

bu ayarda gelen filtrelenmiş logların çıktısını elasticsearche gönderir.
```
sudo service logstash restart
```
Logstsah serverımız şu anda hazır, şimdi Logstasth Forwarderımızı da hazırlayalım.

Set Up Logstash Forwarder
=========================

Daha önce oluşturduğumuz SSL Sertifikasını servera kopyalayalım
(User yerine kendi kullanıcı adınızı, diğerinede server ip'sini girmelisiniz.)
```
scp /etc/pki/tls/certs/logstash-forwarder.crt user@server_private_IP:/tmp
```
Install Logstash Forwarder Package
==================================
```
echo 'deb http://packages.elasticsearch.org/logstashforwarder/debian stable main' | sudo tee /etc/apt/sources.list.d/logstashforwarder.list
```
```
sudo apt-get update
```
```
sudo apt-get install logstash-forwarder
```
Eğer 32 bit ubuntu kullanıyorsanız aşağıdaki adımları yapmalısınız:
```
wget https://assets.digitalocean.com/articles/logstash/logstash-forwarder_0.3.1_i386.deb
```
```
sudo dpkg -i logstash-forwarder_0.3.1_i386.deb
```
Şimdi Logstash Forwarderimizi açılışta çalışmazsını sağlayalım:
```
cd /etc/init.d/; sudo wget https://raw.github.com/elasticsearch/logstash-forwarder/master/logstash-forwarder.init -O logstash-forwarder
```
```
sudo chmod +x logstash-forwarder
```
```
sudo update-rc.d logstash-forwarder defaults
```
Sertifikalarımızı uygun yere kopyalayım.
```
sudo mkdir -p /etc/pki/tls/certs
```
```
sudo cp /tmp/logstash-forwarder.crt /etc/pki/tls/certs/
```
Logstash Forwarderımız için config dosyasını düzenleyelim
```
sudo nano /etc/logstash-forwarder
```
logstash_server_private_IP yazan yere serverımızın ip'sini yazacağız.
içine;
```
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
```
kaydedip çıkalım. Bu dosya Logstash Forwarderımızla Logstashımızı 5000 nolu portu kullanarak logların aktarımını sağlar. (Bunu yaparkende bizim oluşturup kopyaladığımız SSL Sertifikasını kullanır)
```
sudo service logstash-forwarder restart
```
Şu anda Logstash Forwarder'ımız syslog ve auth.log mesajlarını göndermeye başladı bile. Logstash forwarder kurulumunu log göndermek istediğiniz başka yerlerede tekrar edederek loglamaya başlayabilirsiniz.

Connect to Kibana
=================

Bütün ayarlamarımız bittiğine göre artık loglarınızı inceleyebilirsiniz. Logstash server ip nizi web browserdan girdikten sonra Kibana ana sayfasını göreceksiniz ve aşağıdaki Logstash Dasboard'a tıkladığınız zaman 

![alt tag](https://assets.digitalocean.com/articles/logstash/dashboard.png)
