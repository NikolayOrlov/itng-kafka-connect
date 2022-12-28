# itng-kafka-connect

## How to run kafka-connect-elasticsearch plugin with kafka connect standalone

Download binary community connect distribution at https://packages.confluent.io/archive/7.3/ (confluent-community-7.3.1.tar.gz or confluent-community-connect-7.3.1-zOS.tar.gz) and unpack it into confluent-community-connect folder.

Download kafka connect elasticsearch plugin at https://www.confluent.io/hub/confluentinc/kafka-connect-elasticsearch (confluentinc-kafka-connect-elasticsearch-14.0.3.zip) and unpack it into kafka-connect-elasticsearch folder.

Generate a new self-signed “http” certificate for Elasticsearch installation:
1. To generate a PKCS12 private key and self-signed certificate which will be used for http communication for the Elasticsearch installation
/usr/share/elasticsearch/bin/elasticsearch-certutil cert --days 3650 --self-signed --dns <elasticsearch_dns> --name <elasticsearch_dns> --ip <elasticsearch_ip> --out /etc/elasticsearch/certs/new_http.p12 --pass <keystore_password>
(where currently set <keystore_password> could be gotten by /usr/share/elasticsearch/bin/elasticsearch-keystore show xpack.security.http.ssl.keystore.secure_password command)

2. To give read-write permissions and change owner group for the PKCS12 file
chmod g+rw /etc/elasticsearch/certs/new_http.p12
chgrp elasticsearch /etc/elasticsearch/certs/new_http.p12

3. To edit elasticsearch config /etc/elasticsearch/elasticsearch.yml
...
xpack.security.http.ssl:
  enabled: true
  keystore.path: certs/new_http.p12
...

4. To restart elastic:
systemctl restart elasticsearch.service

5. To convert PKCS12 into PK and certificate in PKCS8 PEM:
openssl pkcs12 -in /etc/elasticsearch/certs/new_http.p12 -out new_http.key -nodes -nocerts
openssl pkcs12 -in /etc/elasticsearch/certs/new_http.p12 -out new_http.crt -nokeys

6. To test the connection:
curl --key new_http.key --cert new_http.crt --cacert new_http.crt  https://<elasticsearch_dns>:9200

7. To fix Kibana to use the newly generated new_http PKCS12
cp new_http.crt /var/lib/kibana
chown kibana /var/lib/kibana/new_http.crt
chgrp kibana /var/lib/kibana/new_http.crt
edit /etc/kibana/kibana.yml:
...
elasticsearch.ssl.certificateAuthorities: [/var/lib/kibana/new_http.crt]
...

systemctl restart kibana.service

8. To generate keystore and truststore by running corresponding scripts in the certs folder




