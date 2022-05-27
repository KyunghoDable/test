# Prepare
Node 내 AmazonKinesisFullAccess Policy


# Install
```console
kubectl create ns fluent-bit
kubectl -n fluent-bit create sa fluent-bit
kubectl -n fluent-bit apply -f fluent-bit
```


# Configure
## service
|Parameter|Description|Default|
|---|---|---|
|`storage.path`|스트림 및 데이터 청크를 저장할 위치 설정| |
|`storage.checksum`|파일 시스템에서 데이터를 쓰고 읽을 때 데이터 무결성 검사를 활성화 여부|off|
|`storage.max_chunks_up`|메모리에 올릴 수 있는 최대 청크 수를 설정|128|
|`storage.backlog.mem_limit`|백로그 데이터(전달되지 않고 여전히 스토리지 계층에 있는 데이터 청크를 찾는 과정) 레코드를 처리할 때 사용할 최대 메모리 값|5M|
|`storage.sync`|데이터를 파일 시스템에 저장하는 데 사용되는 동기화 모드|normal|

## FILTER
### kubernetes
|Parameter|Description|Default|
|---|---|---|
|`Merge_Log`|특수문자 제거(디코딩)| |
|`K8S-Logging.Parser`|map 형태의 key 를 한단계 낮춘다  ex) foo : {var:"", fee:""} --> var:"", fee:""| |


### nest
|Parameter|Description|Default|
|---|---|---|
|`Nested_under`|일치하는 Map 안에 Value 들을 뺀다| |

### record_modifier
|Parameter|Description|Default|
|---|---|---|
|`Remove_key`|일치하는 Key 제거| |



## INPUT
|Parameter|Description|Default|
|---|---|---|
|`Buffer_Chunk_Size`|파일 데이터를 읽을 초기 버퍼 크기|32k|
|`Rotate_Wait`|일부 보류 데이터가 플러시되는 경우 한 번 회전된 파일을 모니터링하는 추가 시간(초)을 지정|5|
|`Mem_Buf_Limit`|Tail 플러그인이 사용할 수 있는 메모리 제한| |
|`Buffer_Max_Size`|모니터링되는 파일당 버퍼 크기 제한을 설정|32k|
|`Tag`|태그(regex-extract 필드 포함)를 설정합니다. ex) kube.<namespace_name>.<pod_name>.<container_name>| |
|`Parser`|구문 분석기 종류||
|`DB`|모니터링되는 파일 및 오프셋을 추적할 데이터베이스 파일을 지정||
|`Skip_Long_Lines`|Fluent Bit가 긴 줄을 건너뛰고 버퍼 크기에 맞는 다른 줄을 계속 처리하도록 지시|Off|
|`Refresh_Interval`|감시된 파일 목록을 새로 고치는 간격(초)|60|
|`Path`|로그 파일을 지정하는 패턴||
|`storage.type`|메모리/파일시스템 선택|memory|


# OUTPUT
Match : Kubernetes 의 Tag 는 kube.var.log.containers.<namespace_name>_<pod_name>_<container_name> 와 같이 설정되므로 Container 이름으로 Ouput Match
stream : Kinesis DataStream Name
region : Kinesis DataStream Region
storage.total_limit_size : storage.type 이 filesystem 인 경우 저장될 file 의 최대 크기



# 신규 INPUT/OUPUT 추가
```console
[INPUT]
    Name              tail
    Tag               kube.*
    Buffer_Chunk_Size 512K
    Rotate_Wait       30
    Mem_Buf_Limit     100M
    Buffer_Max_Size   5M
    DB                /var/log/flb_kube.db
    Skip_Long_Lines   On
    Refresh_Interval  10
    storage.type      filesystem
    Path              /var/log/containers/*`_<EXAMPLE>`*.log

[OUTPUT]
    Name kinesis_streams
    storage.total_limit_size  1G
    Match kube.var.log.containers.*`_<EXAMPLE>-`*
    stream app-log-reco-api
    region ap-northeast-2
```
