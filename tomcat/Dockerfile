FROM tomcat:10.1-jdk17

# Restore default webapps
RUN cp -r /usr/local/tomcat/webapps.dist/* /usr/local/tomcat/webapps/

# Configure users
COPY tomcat-users.xml /usr/local/tomcat/conf/tomcat-users.xml

# Configure Manager access
COPY manager-context.xml /usr/local/tomcat/webapps/manager/META-INF/context.xml

EXPOSE 8080

CMD ["catalina.sh","run"]
