


-------------------------------------------------------------------------------------------------------------------


## INPUT
|Parameter|Description|Default|
|---|---|---|
|`Buffer_Chunk_Size`|파일 데이터를 읽을 초기 버퍼 크기|32k|
Rotate_Wait : 일부 보류 데이터가 플러시되는 경우 한 번 회전된 파일을 모니터링하는 추가 시간(초)을 지정[default : 5]
Mem_Buf_Limit : Tail 플러그인이 사용할 수 있는 메모리 제한
Buffer_Max_Size : 모니터링되는 파일당 버퍼 크기 제한을 설정 [default : 32k]
Tag : 태그(regex-extract 필드 포함)를 설정합니다. 
      ex) kube.<namespace_name>.<pod_name>.<container_name>
Parser : 구문 분석기 종류
DB : 모니터링되는 파일 및 오프셋을 추적할 데이터베이스 파일을 지정
Skip_Long_Lines : Fluent Bit가 긴 줄을 건너뛰고 버퍼 크기에 맞는 다른 줄을 계속 처리하도록 지시 [default : Off]
Refresh_Interval : 감시된 파일 목록을 새로 고치는 간격(초) [default : 60]
Path : 로그 파일을 지정하는 패턴
storage.type : 메모리/파일시스템 선택


-------------------------------------------------------------------------------------------------------------------


# OUTPUT
Match : Kubernetes 의 Tag 는 kube.var.log.containers.<namespace_name>_<pod_name>_<container_name> 와 같이 설정되므로 Container 이름으로 Ouput Match
stream : Kinesis DataStream Name
region : Kinesis DataStream Region
storage.total_limit_size : storage.type 이 filesystem 인 경우 저장될 file 의 최대 크기


-------------------------------------------------------------------------------------------------------------------



# 신규 INPUT/OUPUT 추가
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
    Path              /var/log/containers/*_<EXAMPLE>*.log

[OUTPUT]
    Name kinesis_streams
    storage.total_limit_size  1G
    Match kube.var.log.containers.*_<EXAMPLE>-*
    stream app-log-reco-api
    region ap-northeast-2
