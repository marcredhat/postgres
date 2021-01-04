
# TLS Enablement on Postgres

```text
Prepare Database Server for SSL Authentication
$  sudo su
#  cd /apps/pgsql/pgdata01

Copy the files server.pem, rootca_cert.pem and ca_certs.pem to the data directory.
# vi postgresql.conf

And set parameters in postgresql.conf file like below.
ssl = on ssl_ciphers = 'HIGH:MEDIUM:+3DES:!aNULL'
sl_prefer_server_ciphers = on 
ssl_cert_file = ' /apps/pgsql/pgdata01/ server.pem'
ssl_key_file = ' /apps/pgsql/pgdata01/ server.key'
ssl_ca_file = ' /apps/pgsql/pgdata01/ rootca_cert.pem'
ssl_crl_file = ''

Add following entry for the client machine in pg_hba.conf file:
# vi pg_hba.conf
Add below parameter to connect with client machine 
hostssl      all             all            127.0.0.1/32               cert               clientcert=1
Restart the server:
/apps/pgsql/as9.6/bin/pg_ctl -D /apps/pgsql/pgdata01 stop
/apps/pgsql/as9.6/bin/pg_ctl -D /apps/pgsql/pgdata01 start
Test SSL connection from client
/apps/pgsql/as9.6/bin/psql -p 5432 postgres -h 127.0.0.1
/apps/pgsql/as9.6/bin/psql -p 5432 edb -h 127.0.0.1

Above commands will connect to Postgres database as SSL connection TLSv1.2.
```



# Cloudera Manager parameters (to enable TLS on Postgres)


```text
Cloudera Manager:
1. In db.properties:
com.cloudera.cmf.orm.hibernate.connection.url=jdbc:postgresql://<db_host>:<port>/<scm_db>?ssl=true
com.cloudera.cmf.orm.hibernate.connection.username=<scm username> com.cloudera.cmf.orm.hibernate.connection.password=<scm password> 2. In /etc/default/cloudera-scm-server add the following to export CMF_JAVA_OPTS:
-Djavax.net.ssl.trustStore=/opt/cloudera/security/jks/truststore.jks
 
3. Restart the Cloudera Manager server.
 
AMON:
1. In Activity Monitor Advanced Configuration Snippet (Safety Valve) for cmon.conf:

<property>
    <name>db.hibernate.connection.url</name>
    <value>jdbc:postgresql://<db_host>:<db_port>/<amon_db>?ssl=true</value>
</property>
2. In Java Configuration Options for Activity Monitor:
-Djavax.net.ssl.trustStore=/opt/cloudera/security/jks/truststore.jks
​3. Restart AMON.

 RMAN:
1. In Java Configuration Options for Reports Manager:

-Djavax.net.ssl.trustStore=/opt/cloudera/security/jks/truststore.jks
2. In Reports Manager Advanced Configuration Snippet (Safety Valve) for headlamp.db.properties:
com.cloudera.headlamp.db.name=<rman_db>?ssl=true
 
3. Restart RMAN
 
NAVIGATOR AUDIT:

1. In Navigator Audit Server Database Name (navigator.db.name):
<nav_db>?ssl=true
 
2. In Java Configuration Options for Navigator Audit:
-Djavax.net.ssl.trustStore=/opt/cloudera/security/jks/truststore.jks

3. Restart Navigator Audit.
 
NAVIGATOR METADATA SERVER:
1. In Java Configuration Options for Navigator Metadata Server:

-Djavax.net.ssl.trustStore=/opt/cloudera/security/jks/truststore.jks
 
2. In Navigator Metadata Server Advanced Configuration Snippet (Safety Valve) for cloudera-navigator.properties :

navms.db.url=jdbc:postgresql://<postgresql host>:<postgresql pot>/<navms db>?ssl=true

3. Restart Navigator Metadata Server.
 
HIVE:
You can follow below document for hive database https://docs.cloudera.com/documentation/enterprise/6/6.2/topics/cdh_ig_hive_metastore_configure.html#jdbc_url_override 

OOZIE:
1. In Oozie Server Advanced Configuration Snippet (Safety Valve) for oozie-site.xml:

Name: oozie.service.JPAService.jdbc.url
Value: jdbc:postgresql://<host>:<port>/<oozie_db>?ssl=true
 
2. In Java Configuration Options for Oozie Server:

-Djavax.net.ssl.trustStore=/opt/cloudera/security/jks/truststore.jks
 
3. Restart Oozie

HUE:
1. In Hue Server Advanced Configuration Snippet (Safety Valve) for hue_safety_valve_server.ini:

[desktop]
[[database]]
options='{"sslmode": "verify-full"}'
 
2. You need to copy the root.crt file into a default folder /usr/lib/hue/.postgresql and set it to 755. 
     You will get the error below if you don’t: 
          OperationalError: root certificate file "/usr/lib/hue/.postgresql/root.crt" does not exist

$ mkdir /usr/lib/hue/.postgresql
$ cp /var/lib/pgsql/data/root.crt /usr/lib/hue/.postgresql/

3. Restart Hue Service

 
SENTRY:
1. In Sentry Service Advanced Configuration Snippet (Safety Valve) for sentry-xite.xml:
 
Name: sentry.store.jdbc.url
Value: jdbc:postgresql:/<host>:<port>/<sentry_db>?ssl=true
Final: checked
2. In Java Configuration Options for Sentry Server:
 

-Djavax.net.ssl.trustStore=/opt/cloudera/security/jks/truststore.jks
3. Restart Sentry.
```



