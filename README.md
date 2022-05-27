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
