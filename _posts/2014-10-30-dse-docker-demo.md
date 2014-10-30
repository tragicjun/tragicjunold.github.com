*Build docker image for dse*
```text
FROM tegdsf/centos
RUN svn checkout http://tc-svn.tencent.com/doss/doss_openapi_rep/openapi_proj/trunk/commons/DSE /root/dse-trunk
RUN cd /root/dse-trunk; mvn package
RUN mv /root/dse-trunk/release/dse-1.0.3 /root
RUN rm -r /root/dse-trunk
RUN rm -r /tmp/mavenRepository
```
