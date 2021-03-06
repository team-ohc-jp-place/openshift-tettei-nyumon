# 18章 Jenkinsパイプラインサンプル
Spring BootとPostgreSQLで動作するアプリケーションと、そのJenkinsパイプラインのサンプルコードです（Java 8およびJava 11での動作を確認）。アプリケーションの概要は以下の図の通りで、アプリケーションはSpring Bootで動作し、データベースとしてPostgreSQLを利用します。アプリケーションは、4つのAPIエンドポイントを持ちます。

![overview](images/freelancer-overview.png)

## ファイルの説明

| ファイル | 説明 |
:-----------|:------------|
`Jenkinsfile` | Jenkinsのパイプラインの実行定義 |
`etc/Dockerfile_jenkins_agent` | カスタムのJenkins agentを作成するためのDockerfile |
`etc/testdata.sql` | アプリケーションがテストに利用するテストデータ|
`openshift/application-build.yaml` | アプリケーションコンテナイメージの作成に必要なBuildConfigなどのマニフェストファイル |
`openshift/application-deploy.yaml` | アプリケーションをデプロイするためのマニフェストファイル|
`openshift/custom-jenkins-agent.yaml` | カスタムのJenkins agentを作成するために利用するBuildConfigなどのマニフェストファイル|
`pox.xml` | アプリケーションの依存ライブラリ等を記述したファイル（本書サンプルとして内容の理解は不要）|
`src/*` | アプリケーションのコード（本書サンプルとして内容の理解は不要）|


## ローカル環境でアプリケーションを実行する
本書のJenkinsパイプラインを動作させる上では、アプリケーションをローカル環境で実行する必要はありませんが、もし手元で動作させたい場合は下記のように実行できます。PostgreSQLをDockerを用いて起動します。

```
// Java version
$ java -version
java version "1.8.0_211"
Java(TM) SE Runtime Environment (build 1.8.0_211-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.211-b12, mixed mode)

$ mvn -version
Apache Maven 3.6.3 (cecedd343002696d0abb50b32b541b8a6ba2883f)
Maven home: /usr/local/Cellar/maven/3.6.3_1/libexec
Java version: 1.8.0_211, vendor: Oracle Corporation, runtime: /Library/Java/JavaVirtualMachines/jdk1.8.0_211.jdk/Contents/Home/jre
Default locale: ja_JP, platform encoding: UTF-8
OS name: "mac os x", version: "10.16", arch: "x86_64", family: "mac"

// Start PostgreSQL with Docker
$ docker run --name my-pg -e POSTGRES_USER=freelancer -e POSTGRES_PASSWORD=password -e POSTGRES_DB=freelancerdb_test -d -p 5432:5432 postgres:12

// Connect PostgreSQL and load data
// You can load test data by using etc/testdata.sql
$ psql -f etc/testdata.sql -h localhost -U freelancer -d freelancerdb_test
Password for user postgres:
CREATE TABLE
INSERT 0 1
INSERT 0 1
INSERT 0 1

// Test
$ mvn clean test
...
Results :

Tests run: 5, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  38.428 s
[INFO] Finished at: 2021-05-31T00:04:59+09:00
[INFO] ------------------------------------------------------------------------
```

## エンドポイントと仕様
| Method | Endpoint |
:-----------|:------------|
 GET | /freelancers |
 GET | /freelancers/{freelancerId} |

 ```
$ export FREELANCER_URL=http://$(oc get route freelancer-service -n $FREELANCER4J_PRJ -o template --template='{{.spec.host}}')

$ curl -X GET "$FREELANCER_URL/freelancers"
[
  {
    "freelancerId": "1",
    "firstName": "Ken",
    "lastName": "Yasuda",
    "email": "ken.yasuda@example.com",
    "skills": [
      "ruby",
      "php",
      "mysql"
    ]
  },
  {
    "freelancerId": "2",
    "firstName": "Tadashi",
    "lastName": "Komiya",
    "email": "tadashi.komiya@example.com",
    "skills": [
      "c#",
      "windows",
      "sqlserver"
    ]
  },
  {
    "freelancerId": "3",
    "firstName": "Taro",
    "lastName": "Goto",
    "email": "taro.goto@example.com",
    "skills": [
      "ruby",
      "postgresql",
      "java"
    ]
  }
]


$ curl -X GET "$FREELANCER_URL/freelancers/1"
{
  "freelancerId": "1",
  "firstName": "Ken",
  "lastName": "Yasuda",
  "email": "ken.yasuda@example.com",
  "skills": [
    "ruby",
    "php",
    "mysql"
  ]
}
 ```
