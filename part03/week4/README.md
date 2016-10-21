이 문서는 [여기](http://googlegenomics.readthedocs.io/en/latest/use_cases/run_samtools_over_many_files/index.html)의 문서를 간략하게 한글로 번역 정리 한 것이다.
# 클라우드 스토리지에서 SAMtools 를 이용하여 BAM 파일 인덱스 만들기.
만약 수천개의 BAM 파일이 있고 이 파일들 모두를 구글 클라우드 저장소에서 인덱스 파일 (BAI) 을 만들어야 한다면?

[Grid Computing Tools github](https://github.com/googlegenomics/grid-computing-tools) 에서는 이를 위한 코드들을 제공하고 있다.

## Overview
아래의 두개의 기술 덕분에 많은 수의 파일들을 처리할 수 가 있다.
* [Google Compute Engine](https://cloud.google.com/compute/)
* [Grid Engine (SGE)](http://gridengine.info/)

Google Compute Engine 을 통해서는 VM을 사용해서 수십에서 수백개의 인스턴스를 사용할 수 있다.

Grid Engine 을 통해서는 생성한 인스턴스 위에서 각종 작업들을 할 수 있다.

## Directory structure
이 도구들을 사용하기 위해서 내 컴퓨터에 [Grid Computing Tools github](https://github.com/googlegenomics/grid-computing-tools) 와 [Elasticluster repo](https://github.com/gc3-uzh-ch/elasticluster)를 내려받아야 한다.

그리고 아래의 두개의 디렉토리는 workspace root(`WS_ROOT`)에 같이 있는 것으로 가정한다.

* `grid-computing-tools`
* `elasticluster`

## Running the samples

아래의 예에서는 3개의 worker 인스턴스에서 6개의 파일을 가지고 `samtools` 실습을 한다. (BAM 파일은 public repo에 있는 BAM 파일 사용)

1. grid engine을 사용하는 google compute engine 클러스터를 생성한다.
    shell 에서 :
    1. cd ${WS_ROOT}
    1. [Create a Grid Engine cluster on Compute Engine](http://googlegenomics.readthedocs.io/en/latest/use_cases/setup_gridengine_cluster_on_compute_engine/index.html) 의 링크의 내용을 따라서 클러스터를 생성한다.
1. `grid-computing-tools` repository 를 내려받는다.
```
cd ${WS_ROOT}
git clone https://github.com/googlegenomics/grid-computing-tools.git
```
1. `src` and `samples`  디렉토리들을 Grid Engine master instance 에 올린다.
```
cd grid-computing-tools
elasticluster sftp gridengine << 'EOF'
mkdir src
mkdir src/common
mkdir src/samtools
put src/common/* src/common/
put src/samtools/* src/samtools/
mkdir samples
mkdir samples/samtools
put samples/samtools/* samples/samtools/
EOF
```
1. 마스터 인스턴스에 SSH 한다.
```
elasticluster ssh gridengine
```
1. 샘플들을 작업하기 위한 설정 파일을 만든다. (참조: `grid-computing-tools/samples/samtools/samtools_index_config.sh`)
The syntax for running the sample is:
```
./src/samtools/launch_samtools.sh [config_file]
```
`config_file` 에는 두가지 종류의 인자가 정의 되어 있어야 한다.
    * 실행하려는 operation 종류
    * 원본 파일들과 생성 파일들이 저장될 공간
operation 종류는 아래와 같이 정하면 된다.
    * SAMTOOLS_OPERATION: `index`
공간의 위치는 다음과 같이 정의 한다.
    * INPUT_LIST_FILE: file containing a list of GCS paths to the input files to process
    * OUTPUT_PATH: GCS path indicating where to upload the output files. If set to source, the output will be written to the same path as the source file (with the extension .bai appended)
    * OUTPUT_LOG_PATH: (optional) GCS path indicating where to upload log files
1. Run the sample:
아래와 같이 실행한다.
```
./src/samtools/launch_samtools.sh ./samples/samtools/samtools_index_config.sh
```
성공적으로 launch되면 아래와 같은 메시지가 출력 된다. 
```
Your job-array 1.1-6:1 ("samtools") has been submitted
This message tells you that the submitted job is a gridengine array job. The above message indicates that the job id is 1 and that the tasks are numbered 1 through 6. The name of the job samtools is also indicated.
```
1. Monitoring the status of your job
`qstat`이라는 명령어를 사용해서 클러스터에 제출된 작업들의 상태를 확인 할 수 있다.
1. Destroying the cluster
```
elasticluster stop gridengine
```

## Running your own job
이제는 자신의 BAM 파일을 작업할 차례이다. 그 때 추가로 필요한 사항들은 다음과 같다.
1. Create an input list file
1. Create a job config file
1. Create a gridengine cluster with sufficient disk space attached to each compute node
1. Upload input list file, config file, and grid-computing-tools source to the gridengine cluster master
1. Do a “dry run” (optional)
1. Do a “test run” (optional)
1. Launch the job

이중에서 가장 중요해 보이는 충분한 디스크 확보에 대해서 살펴본다. 나머지는 [원본](http://googlegenomics.readthedocs.io/en/latest/use_cases/run_samtools_over_many_files/index.html) 문서를 참조하면 쉽게 따라할 수 있다.

** Create a gridengine cluster with sufficient disk space attached to each compute node **

Determine disk size requirements

Each compute node will require sufficient disk space to hold the input and output files for its current task. Determine the largest file in your input list and estimate the total space you will need. It may be necessary to download the file and perform the operation manually to get a maximum combined input and output size.

Persistent disk performance also scales with the size of the volume. Independent of storage requirements, for consistent throughput on long running jobs, use a standard persistent disk of at least 1TB, or use SSD persistent disk. More documentation is available for selecting the right persistent disk.


Verify or increase quota

Your choice for number of nodes and disk size must take into account your Compute Engine resource quota for the region of your cluster.

Quota limits and current usage can be viewed with gcloud compute:

gcloud compute regions describe *region*
or in Cloud Platform Console:

https://console.cloud.google.com/project/_/compute/quotas
Important quota limits include CPUs, in-use IP addresses, and disk size.

To request additional quota, submit the Compute Engine quota request form.

