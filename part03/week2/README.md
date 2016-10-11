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
24시간이 지나서 instance 가 terminate 되었을 때 다음과 같은 일을 할 수 있다.

TERMINATED instances 를 클러스터에서 제거하고 새로운 인스턴스를 생성하여 TERMINATED 인스턴스를 대체한다.
-> 이 일은 [cluster_monitor.sh](https://github.com/googlegenomics/grid-computing-tools/blob/master/bin/cluster_monitor.sh) 스크립트를 이용하여 할 수 있다.

이 스크립트는 다음 사이트에서 얻을 수 있다. [grid-computing-tools github project](https://github.com/googlegenomics/grid-computing-tools)

#### Downloading the grid-computing-tools
[grid-computing-tools github project](https://github.com/googlegenomics/grid-computing-tools) 을 클론하기 위해서 다음을 따라한다.
```git clone https://github.com/googlegenomics/grid-computing-tools.git```
`elasticluster` directory 의 sibling directory 에 클론하는 것을 추천한다.

#### Running cluster_monitor.sh
먼저 PATH 에 `elasticluster` 실행파일들이 포함되어있는지 확인해야 함.
PATH에 포함해서 export 하거나 `virtualenv`를 이용하면 된다. 
```
source elasticluster/bin/active
```
그런 다음 스크립트를 실행한다. 
```
grid-computing-tools/bin/cluster_monitor.sh gridengine
```
스크립트는 계속 실행상태를 유지하므로 스크립트를 종료하고 싶으면 `Ctrl-C`를 누른다.

기본적으로 모니터링은 클러스터 상태를 확인 하고 10분 동안 sleep모드를 유지한다. sleep interval을 바꾸고 싶으면 추가 아규먼트를 스크립트에 넘겨야 한다. 예를 들어
```
grid-computing-tools/bin/cluster_monitor.sh gridengine 5
```
이렇게 하면 5분 동안 sleep 하게 된다.

#### To grow your cluster
클러스터내에서 worker 노드의 수를 늘리기 위해서는 `~/.elasticluster/config`의 `compute_nodes`값을 변경해야 한다. 예를 들어 [Grid Engine cluster](http://googlegenomics.readthedocs.io/en/latest/use_cases/setup_gridengine_cluster_on_compute_engine/index.html) 를 생성할 때 3개에서 10개로 늘리고 싶으면 다음과 같이 세팅한다. 
```
[cluster/gridengine]
...
compute_nodes=10
...
```
이렇게 하면 다음번 클러스터 모니터가 작동 할 때 클러스터 워커 갯수를 늘리게 된다.

#### To shrink your cluster
To reduce the number of workers in your cluster while it is running, update the compute_nodes value in ~/.elasticluster/config.

As the preemptible VMs are terminated, the cluster monitor will remove them from the cluster, and will only replace instances if the total number in the cluster is less than the configured value. You can also manually terminate instances if desired.

### Monitoring your job
컴퓨팅 노드가 종료되는 것과는 별개로 그 위에서 돌고 있는 작업들에 대한 모니터링에 대한 방법이 필요하다.
노드가 종료되었다가 다시 실행 되었다고 해서 그 위에서 돌던 작업이 자동으로 실행 되는 것이 아닐 뿐더러, 실행 프로그램의 자체 문제 등으로 인해 노드와 상관 없이 프로그램이 종료 되어 있을 수도 있다. 
이를 위해  [array_job_monitor.sh](https://github.com/googlegenomics/grid-computing-tools/blob/master/tools/array_job_monitor.sh) script 를 [grid-computing-tools github project](https://github.com/googlegenomics/grid-computing-tools)에서 제공하고 있다.

- For each task allocated to a node:
 - Get the associated node’s uptime
   - Restart the task if
     - the node is down
     - the node’s uptime is less than the task’s running time (meaning that the node has been replaced since the task started)
     - the task runtime is longer than a configurable timeout interval (optional)
    
Note: when you launch your job on the Grid Engine cluster, be sure to mark the job as “restartable”. This can be done by passing the flag `-r y` to the `qsub` command.

#### Upload the job monitor script
job monitor 스크립트는 클러스터의 frontend 노드에서 실행해야만 한다. `array_job_monitor.sh`를 올리기 위해서는

```
elasticluster sftp gridengine << EOF
mkdir tools
put tools/array_job_monitor tools/
EOF
```
를 실행하면 된다.

#### Run the job monitor script

`array_job_monitor.sh` 를 실행하기 위해서 frontend 인스턴스에 ssh로 접근 이 필요하다.
```
elasticluster ssh gridengine
```
`array_job_monitor.sh`에 대한 파라미터들은 아래와 같다.

**job_id**
    Grid Engine job ID to monitor
**monitor_interval**
    Minutes to sleep between checks of running tasks

    Default: 15 minutes

**task_timeout**
    Number of minutes a task may run before it is considered stalled, and is eligible to be resubmitted.

    Default: None

**queue_name**
    Grid Engine job queue the job_id is associated with

    Default: all.q

For example, to monitor job 1, every 5 minutes, for jobs that should not take more than 10 minutes:
```
./tools/array_job_monitor.sh 1 5 10
```