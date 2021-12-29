# Maven 自定义 Archetype

## 一、目标项目结构（用来做模板的项目）

![2021-12-29_134513](https://img.qinweizhao.com/2021/12/2021-12-29_134513.png)

## 二、创建 archetype

### 1、进入到项目根目录下执行（ pom.xml 同级目录）

```bash
mvn archetype:create-from-project
```

ERROR:

```txt
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  2.431 s
[INFO] Finished at: 2021-12-29T14:26:01+08:00
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-archetype-plugin:3.2.0:create-from-project (default-cli) on project qwz-mall: Invoker process ended with result different t
han 0! -> [Help 1]
[ERROR]
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR]
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoFailureException
```

SOLVE:将 Maven 安装目录的 conf 文件夹中的 setting.xml 复制一份到 用户目录的 .m2 文件夹中

![2021-12-29_142657](https://img.qinweizhao.com/2021/12/2021-12-29_142657.png)

此时会在项目 target 下生成文件：

![2021-12-29_135650](https://img.qinweizhao.com/2021/12/2021-12-29_135650.png)

### 2、如果是多模块需要进行修改

![2021-12-29_143101](https://img.qinweizhao.com/2021/12/2021-12-29_143101.png)

如图所示，需要更改的地方有两处：

1. 将 mall 替换为 `__rootArtifactId__`
2. 打开 archetype-metadata.xml 文件将 除了 dir 处的 mall 替换为 `${rootArtifactId}`，dir 除的 mall 替换为 `__rootArtifactId__`

![2021-12-29_143725](https://img.qinweizhao.com/2021/12/2021-12-29_143725.png)

### 3、将 target 目录下的 archetype 安装到本地仓库

```bash
 mvn install
```

### 4、生成骨架配置文件

```bash
mvn archetype:crawl
```

执行成功后，在本地仓库的根目录生成`archetype-catalog.xml`骨架配置文件:

![2021-12-29_140330](https://img.qinweizhao.com/2021/12/2021-12-29_140330.png)

## 三、使用 archetype 模板

```bash
 mvn archetype:generate -DarchetypeCatalog=local
```

> 执行`mvn archetype:generate -DarchetypeCatalog=local`从本地 archeType 模板中创建项目。

![2021-12-29_144711](https://img.qinweizhao.com/2021/12/2021-12-29_144711.png)

选择模板，指定 groupId、artifactId、version 和 package 信息。

![2021-12-29_145030](https://img.qinweizhao.com/2021/12/2021-12-29_145030.png)
