# Securing Solr (jetty)

### HashLoginService

Add 
```xml
    <Call name="addBean">
      <Arg>
        <New class="org.eclipse.jetty.security.HashLoginService">
          <Set name="name">Solr Admin Access</Set>
          <Set name="config"><SystemProperty name="jetty.home" default="."/>/etc/realm.properties</Set>
          <Set name="refreshInterval">0</Set>
        </New>
      </Arg>
   </Call>
```
to jetty.xml (/opt/solr-6.1.0/server/etc/jetty.xml)

### jetty users

Generate password hash
```bash
java -cp /opt/solr-6.1.0/server/lib/jetty-util-9.3.8.v20160314.jar org.eclipse.jetty.util.security.Password myusername mysecret

2017-03-27 15:37:34.752:INFO::main: Logging initialized @173ms
mypassword
OBF:1uh41zly1x8g1vu11ym71ym71vv91x8e1zlk1ugm
MD5:34819d7beeabb9260a5c854bc85b3e44
CRYPT:myylAylKPNtmw
```

Create realm.properties (as configured in HashLoginService, eg /etc/realm.properties points to /opt/solr-6.1.0/server/etc/realm.properties). Use the hash (eg OBF) as generated before. The syntax is
username: hash[,role]
```java
myusername: OBF:1uh41zly1x8g1vu11ym71ym71vv91x8e1zlk1ugm,admin
```

### web.xml

Add security-constraint with auth-constraint for everything and a security-constraint without auth-constraint for stuff which should be accessed publicly.

```xml
  <security-constraint>
    <web-resource-collection>
      <web-resource-name>mysolrcore-select</web-resource-name>
      <url-pattern>/mysolrcore/select/*</url-pattern>
    </web-resource-collection>
    <web-resource-collection>
      <web-resource-name>mysolrcore-suggest</web-resource-name>
      <url-pattern>/mysolrcore/suggest/*</url-pattern>
    </web-resource-collection>
  </security-constraint>

  <security-constraint>
    <web-resource-collection>
      <web-resource-name>Solr authenticated application</web-resource-name>
      <url-pattern>/*</url-pattern>
    </web-resource-collection>
    <auth-constraint>
      <role-name>admin</role-name>
    </auth-constraint>
  </security-constraint>

  <login-config>
    <auth-method>BASIC</auth-method>
    <realm-name>Solr Admin Access</realm-name>
  </login-config>
```

### restart solr service
