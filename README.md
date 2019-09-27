# kurento 설치
```bash
$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 5AFA7A83
$ sudo tee "/etc/apt/sources.list.d/kurento.list" > /dev/null << EOF
# Kurento Media Server - Release packages
def [arch=amd64] http://ubuntu.openvidu.io/6.11.0 bionic kms6
EOF
$ sudo apt-get update
$ sudo apt-get install --yes kurento-media-server
```

# Kurento Media Server Start/Stop

```bash
$ sudo service kurento-media-server start
$ sudo service kurento-media-server stop
```

# Kurento Media Server Client Creator

```bash
$ sudo apt-get install kurento-module-creator
```

```bash
$ kurento-module-creator -c <CODEGEN_DIR> -r <ROM_FILE> -r <TEMPLATES_DIR>
```

#### kurento-module-creator Command Option Description


* CODEGEN_DIR: Destination directory for generated files.
* ROM_FILE: A space-separated list of Kurento Media Element Description (kmd files), or folders containing these files. For example, you can take a look to the kmd files within the Kurento Media Server source code.
* TEMPLATES_DIR: Directory that contains template files. As an example, you can take a look to the internal Java templates and JavaScript templates directories.

* CODEGEN_DIR: 생성 된 파일의 대상 디렉토리.
* ROM_FILE: 공백으로 구분 된 Kurento 미디어 요소 설명 (kmd 파일) 또는 이러한 파일이 포함 된 폴더의 목록입니다 . 예를 들어 Kurento Media Server 소스 코드 내의 kmd 파일을 살펴볼 수 있습니다 .
* TEMPLATES_DIR: 템플리트 파일이 포함 된 디렉토리. 예를 들어, 내부 Java 템플리트 및 JavaScript 템플리트 디렉토리를 살펴볼 수 있습니다 .

## Kurento Module Creator Java, Javascript Templates

* https://github.com/Kurento/kurento-client-js/tree/master/templates
* https://github.com/Kurento/kurento-java/tree/master/kurento-client/src/main/resources/templates

## Kurento Media Server KMD Files Git

* https://github.com/Kurento/kms-elements/blob/master/src/server/interface
* https://github.com/Kurento/kms-core/blob/master/src/server/interface
* https://github.com/Kurento/kms-filters/tree/master/src/server/interface


