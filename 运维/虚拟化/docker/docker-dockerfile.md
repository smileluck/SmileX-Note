[toc]

---

# jar 

## 使用application.yml启动

```
FROM java:8
EXPOSE 8080

ENV APP_JVM="-server -Xms2g -Xmx2g -Xmn1g -Duser.timezone=GMT+08"

ENV SPRING_YML="--spring.config.location=/application.yml"

VOLUME /tmp
ADD smilex-admin.jar  /admin.jar
ADD application.yml  /application.yml
RUN bash -c 'touch /admin.jar'
ENTRYPOINT java ${APP_JVM} -jar /admin.jar ${SPRING_YML}
```

