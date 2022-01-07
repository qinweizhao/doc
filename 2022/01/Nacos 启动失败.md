# Nacos 启动失败

## 一、环境

1. CentOS 7 
2. JDK 11

## 二、报错信息

![2022-01-07_163904](https:img.qinweizhao.com/2022/01/2022-01-07_163904.png)

## 三、解决方案

修改 bin 目录下的 startup.sh 文件

需要更改的配置

1. ```sh
   JAVA_OPT_EXT_FIX="-Djava.ext.dirs=${JAVA_HOME}/jre/lib/ext:${JAVA_HOME}/lib/ext"
   ```

2. ```sh
   echo "$JAVA $JAVA_OPT_EXT_FIX ${JAVA_OPT}"
   ```

3. ```sh
   echo "$JAVA $JAVA_OPT_EXT_FIX ${JAVA_OPT}" > ${BASE_DIR}/logs/start.out 2>&1 &
   ```

4. ```sh
   nohup "$JAVA" "$JAVA_OPT_EXT_FIX" ${JAVA_OPT} nacos.nacos >> ${BASE_DIR}/logs/start.out 2>&1 &
   ```

更改为

1. ```sh
   JAVA_OPT="${JAVA_OPT} -Djava.ext.dirs=${JAVA_HOME}/jre/lib/ext:${JAVA_HOME}/lib/ext"
   ```

2. ```sh
   echo "$JAVA ${JAVA_OPT}"
   ```

3. ```sh
    echo "$JAVA ${JAVA_OPT}" > ${BASE_DIR}/logs/start.out 2>&1 &
   ```

4. ```sh
   nohup $JAVA ${JAVA_OPT} nacos.nacos >> ${BASE_DIR}/logs/start.out 2>&1 &
   ```

   