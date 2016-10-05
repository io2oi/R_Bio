이 문서는 [원본연결](http://googlegenomics.readthedocs.io/en/latest/use_cases/setup_gridengine_cluster_on_compute_engine/preemptible_vms.html#monitoring-your-job) 의 문서를 간략하게 정리한 것이다.

# Create a Grid Engine cluster with Preemptible VM workers
Preemptible VM worker는 일반 instance와 달리 가격이 매우 싸다는 장점이 있다. 대신해서 최대 24시간까지만 사용 가능하다.  
| Note |
|--------|
| Preemtible VM은 최대 20개 정도까지만 생성 하는 것을 추천한다. 그 이상을 사용하고 싶으면 정규 VM을 사용하라.|

## Toolset
Premptible VM을 사용하기 위해서는 다음의 세가지 도구가 필요하다
* [Elasticluster](https://elasticluster.readthedocs.org/) - to create, configure, and destroy the cluster
* [cluster_monitor.sh](https://github.com/googlegenomics/grid-computing-tools/blob/master/bin/cluster_monitor.sh) - to replace TERMINATED virtual machine instances
* [array_job_monitor.sh](https://github.com/googlegenomics/grid-computing-tools/blob/master/tools/array_job_monitor.sh) - to requeue failed Grid Engine tasks

## Steps

### Setting up your cluster
먼저 생성 방법은 간단하다. 앞에서 살펴보았던 gridengine cluster 설정 파일에 아래의 두줄을 추가한다.
frontend는 preemtible 하지 않아야 한다. compute 노드만 preemtible 하게 설정한다.
`~/.elasticluster/config`: 
```
[cluster/gridengine/compute]
scheduling=preemptible
```

### Monitoring your cluster
24시간이 지나 terminate가 된 노드들에 대해서 제거하고 새 노드로 교제하는 작업이 필요하다. 이 일을 `cluster_monitor.sh` 스크립트가 해 준다.

#### Downloading the grid-computing-tools
[grid-computing-tools github](https://github.com/googlegenomics/grid-computing-tools) 클론하기
```
git clone https://github.com/googlegenomics/grid-computing-tools.git
```
`elasticluster` 디렉토리와 같은 수준에서 `grid-computing-tools` 를 clone하는 것을 추천한다.

#### Running cluster_monitor.sh
elasticluster 를 PATH에 잡도록 virtualenv를 활성화 한다. 
```
source elasticluster/bin/active
```
Then run the cluster monitor script:
그리고 나서 cluster monitor script를 실행한다. 
```
grid-computing-tools/bin/cluster_monitor.sh gridengine
```
`Ctrl-C`를 눌러서 종료할 수 있다. 

기본적으로는 10분에 한번씩 클러스터 상태를 확인한다. 간격을 바꾸고 싶으면 추가 인자를 지정하여 스크립트를 실행하면 된다. 예를 들어
```
grid-computing-tools/bin/cluster_monitor.sh gridengine 5
```
이렇게 하면 5분에 한번씩 확인한다.

#### To grow your cluster
compute 노드의 갯수를 늘리고 싶으면 `~/.elasticluster/config` 의 내용을 바꿈으로서 가능하다. 예를 들어 3개에서 10개로 늘리고 싶으면 아래와 같이 파일 내용을 바꾸면 모니터링이 다시 확인할 때 노드를 추가 하게 된다. 
```
[cluster/gridengine]
...
compute_nodes=10
...
```

#### To shrink your cluster
반대로 갯수를 줄이고 싶다면 같은 방법을 이용한다.

preemptible VM 이 종료되면 모니터가 해당 클러스터를 제거하고 설정파일의 최대 갯수에 맞추어서 인스턴스를 교체한다. 수동으로 인스턴스를 종료하고 싶을 때는 [여기](https://cloud.google.com/compute/docs/instances/stopping-or-deleting-an-instance) 를 참조하라. 

### Monitoring your cluster

#### Downloading the grid-computing-tools
#### Running cluster_monitor.sh
#### To grow your cluster
#### To shrink your cluster

### Monitoring your job
#### Upload the job monitor script
#### Run the job monitor script

