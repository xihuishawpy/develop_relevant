

1. 新建用户
   ```shell
    useradd trino
   ``` 
2. sz上传java包至/usr/local/
3. 解压、重命名为java11
   ```shell
    tar -zxvf java11.tar.gz
    mv java11 /usr/local/java11
   ```

4. root下授权给trino用户
    ```
    chown -R trino:trino /usr/local/java11     
    ```
5. 配置java环境
    ```shell
    vi ~/.bashrc

    export JAVA_HOME=/usr/local/java11
    export PATH=$JAVA_HOME/bin:$PATH
    CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
    ``` 
6. source生效
    ```shell
    source ~/.bash_profile

    java -version
    ```



