# パイプラインベースのCI/CDツール Concourse CI入門

- 以下のサイトを参考とした

    https://ik.am/entries/379

- 以前作成中の以下のREADMEも参考としている

    https://github.com/moriyamaES/ConcourseCI-install#readme

## 環境

- 環境は以下

    ```
    # uname -a
    Linux control-plane.minikube.internal 3.10.0-1160.95.1.el7.x86_64 #1 SMP Mon Jul 24 13:59:37 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
    ```

- Concourse CI を起動

    ```
    # docker-compose -f ~/ConcourseCI-install/docker-compose.yml up -d
    [+] Running 3/3
    ✔ Container concourseci-install-db-1      Started                                                                                                                                                                             0.0s 
    ✔ Container concourseci-install-web-1     Started                                                                                                                                                                             0.0s 
    ✔ Container concourseci-install-worker-1  Started   
    ```

- Concourse CI　のWebにアクセス
- 以下のコマンドを実行

    ```
    # fly --target tutorial login --concourse-url http://localhost:8080
    ```

 - 表示されたURL（`http://10.1.1.200:8080/login?fly_port=45953` ） でブラウザにアクセスし、ユーザ：`test` パスワード`test` でログインし 表示されたトークンを`(input hidden):`の後ろにコピーする


        ```
        logging in to team 'main'

        navigate to the following URL in your browser:

        http://10.1.1.200:8080/login?fly_port=45953

        or enter token manually (input hidden): 
        ```

- 既存のパイプラインを削除する

```
# fly -t tutorial destroy-pipeline -p publishing-outputs
```

## はじめてのConcourse CI

- 以下を作成

```
jobs:
- name: hello-world
  plan:
  - task: say-hello
    config:
      platform: linux
      image_resource:
        type: docker-image
        source:
          repository: ubuntu
      run:
        path: bash
        args: 
        - -c
        - |
          echo "Hello, world!"
```

- 以下のコマンドを実行し、パイプラインを設定

```
# fly -t tutorial set-pipeline -p hello -c hello.yml
```

- 以下のコマンドを実行し、パイプラインを有効にする

```
# fly -t tutorial unpause-pipeline -p hello
```

- 以下のコマンドで、ジョブを実行する

```
#  fly -t tutorial trigger-job -j hello/hello-world --watch
```

- 結果　→ 過去を同じように以下のエラーが発生

```
# fly -t tutorial trigger-job -j hello/hello-world --watch
started hello/hello-world #1

initializing
initializing check: image
selected worker: df635a5eda68
run check: find or create container on worker df635a5eda68: failed to retrieve kernel parameter "net.ipv4.tcp_retries1": open /proc/sys/net/ipv4/tcp_retries1: no such file or directory
run check: find or create container on worker df635a5eda68: failed to retrieve kernel parameter "net.ipv4.tcp_retries1": open /proc/sys/net/ipv4/tcp_retries1: no such file or directory
errored
```

- ここでも、イメージを `centos` に変える


```
jobs:
- name: hello-world
  plan:
  - task: say-hello
    config:
      platform: linux
      image_resource:
        type: docker-image
        source:
          repository: centos
      run:
        path: bash
        args: 
        - -c
        - |
          echo "Hello, world!"
```


- 以下のコマンドを実行し、パイプラインを設定

```
# fly -t tutorial set-pipeline -p hello -c hello.yml
```

- 以下のコマンドを実行し、パイプラインを有効にする

```
# fly -t tutorial unpause-pipeline -p hello
```

- 以下のコマンドで、ジョブを実行する

```
#  fly -t tutorial trigger-job -j hello/hello-world --watch
```



```
# fly -t tutorial set-pipeline -p centos -c task_centos-uname.yml
```


```
# fly -t tutorial unpause-pipeline -p centos
```

```
#  fly -t tutorial trigger-job -j hello/hello-world --watch
```



hello.yml