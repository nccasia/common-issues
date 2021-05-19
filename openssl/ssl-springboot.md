# SSL with springboot

## Solution 1: configuration

1. Generating a certificate for your domain (e.g. example.com)

        certbot-auto certonly -d example.com
        
Things are generated in /etc/letsencrypt/live/example.com. Spring Boot expects PKCS#12 formatted file. It means that you must convert the keys to a PKCS#12 keystore (e.g. using OpenSSL). As follows:

- Open /etc/letsencrypt/live/example.com directory

        openssl pkcs12 -export -in fullchain.pem -inkey privkey.pem -out keystore.p12 -name tomcat -CAfile chain.pem -caname root
        
- The file keystore.p12 with PKCS12 is now generated in /etc/letsencrypt/live/example.com.
  
  It's time to configure your Spring Boot application. Open the application.properties file and put following properties there:
  
        server.port=8443
        security.require-ssl=true
        server.ssl.key-store=/etc/letsencrypt/live/example.com/keystore.p12
        server.ssl.key-store-password=<your-password>
        server.ssl.keyStoreType=PKCS12
        server.ssl.keyAlias=tomcat


## Solution 2: proxy pass from nginx

        upstream app_dev_webserver_node_server {
          server 127.0.0.1:8080;
        }
        
        server {            
            location /api/ {
                proxy_pass              http://app_dev_webserver_node_server;
            }
        }