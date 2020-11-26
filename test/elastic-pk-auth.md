     
 # Elastic PKI authetication
 https://www.elastic.co/blog/elasticsearch-security-configure-tls-ssl-pki-authentication    
     
     
     sh-4.4$ bin/elasticsearch-certutil cert --ca-cert /usr/share/elasticsearch/config/http-certs/ca.crt --ca-key /usr/share/elasticsearch/config/http-certs/tls.key --name "CN=something,OU=Consulting Team,DC=mydomain,DC=com"

