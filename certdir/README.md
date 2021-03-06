To run the SSL tests, the following properties are used:

* certdir: directory where the certificates and keys are store
* enable_ssl_tests: enables SSL tests

In order to configure PostgreSQL for SSL tests, the following changes should be applied:

* Copy server/server.crt, server/server.key, and server/root.crt to $PGDATA directory
* In $PGDATA directory: chmod 0600 server.crt server.key root.crt
* Set ssl=on in postgresql.conf
* Set ssl_cert_file=server.crt in postgresql.conf
* Set ssl_key_file=server.key in postgresql.conf
* Set ssl_ca_file=root.crt in postgresql.conf
* Add databases for SSL tests. Note: sslinfo extension is used in tests to tell if connection is using SSL or not

      for db in hostdb hostssldb hostnossldb certdb hostsslcertdb; do
        createdb $db
        psql $db -c "create extension sslinfo"
      done
* Add test databases to pg_hba.conf. If you do not overwrite the pg_hba.conf then remember to comment out all lines
  starting with "host all".
* Uncomment enable_ssl_tests=true in ssltests.properties
* The username for connecting to postgres as specified in build.local.properties tests has to be "test".

This directory contains example certificates generated by the following
commands:

openssl req -x509 -newkey rsa:1024 -days 3650 -keyout goodclient.key -out goodclient.crt
#Common name is test, password is sslpwd

openssl req -x509 -newkey rsa:1024 -days 3650 -keyout badclient.key -out badclient.crt
#Common name is test, password is sslpwd

openssl req -x509 -newkey rsa:1024 -days 3650 -nodes -keyout badroot.key -out badroot.crt
#Common name is localhost
rm badroot.key

openssl pkcs8 -topk8 -in goodclient.key -out goodclient.pk8 -outform DER -v1 PBE-MD5-DES

openssl pkcs8 -topk8 -in badclient.key -out badclient.pk8 -outform DER -v1 PBE-MD5-DES

cp goodclient.crt server/root.crt

cd server

openssl req -x509 -newkey rsa:1024 -nodes -days 3650 -keyout server.key -out server.crt

cp server.crt ../goodroot.crt

#Common name is localhost, no password

#PKCS12

Create the goodclient.p12 file with

openssl pkcs12 -export -in goodclient.crt -inkey goodclient.key -out goodclient.p12 -name local -CAfile client_ca.crt -caname local
