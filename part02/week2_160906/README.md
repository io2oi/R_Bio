# Run workflows and common tasks in parallel
[연결](http://googlegenomics.readthedocs.io/en/latest/use_cases/run_pipelines_in_the_cloud/index.html)

우선 이 작업을 하기 위해서는 자신의 project id 와 bucket id를 알고 있어야 한다.

bucket id 확인: [https://console.cloud.google.com/storage](https://console.cloud.google.com/storage)
    또는 
```bash
]$ gsutil ls
```
project id 확인: [https://console.cloud.google.com/iam-admin](https://console.cloud.google.com/storage) 에서 Setting

## Overview
파이프라인(pipeline) 이라는 것은 아래의 일을 포함하는 것이다.
- Path(s) of input files to read from Cloud Storage
- Path(s) of output files/directories to write to Cloud Storage
- A Docker image to run
- A command to run in the Docker image
- Cloud resources to use (number of CPUs, amount of memory, disk size and type)

Pipelines API 는 다음의 일들을 한다.
- Create a Compute Engine virtual machine
- Download the Docker image
- Download the input files
- Run a new Docker container with the specified image and command
- Upload the output files
- Destroy the Compute Engine virtual machine
- Log files are uploaded periodically to Cloud Storage.

실제의 예제를 보면서 파악해 보자
*이 Pipeline API는 실행할 동안에만 존재하는 가상머신이나 파일들에 기반하여 작동한다. 고정되어 있는 가상머신등을 사용하고 싶을 때는 Grid Engine 을 사용하자*

## 실제 예제
### 시작하기 전에 필요한 것
- Clone or fork [this repository](https://github.com/googlegenomics/pipelines-api-examples/).
- If you plan to create your own Docker images, then install docker: https://docs.docker.com/engine/installation/#installation
- Follow the Google Genomics [getting started instructions](https://cloud.google.com/genomics/install-genomics-tools#create-project-and-authenticate) to set up your Google Cloud Project. The Pipelines API requires that the following are enabled in your project:
 - [Genomics API](https://console.cloud.google.com/project/_/apis/api/genomics)
 - [Cloud Storage API](https://console.cloud.google.com/project/_/apis/api/storage_api)
 - [Compute Engine API](https://console.cloud.google.com/project/_/apis/api/compute_component)
- Follow the Google Genomics [getting started instructions](https://cloud.google.com/genomics/install-genomics-tools#install-genomics-tools) to install and authorize the Google Cloud SDK.
- Install or update the python client via ```pip install --upgrade google-api-python-client```. For more detail see [https://cloud.google.com/genomics/v1/libraries]( gs://genomics-public-data/ftp-trace.ncbi.nih.gov/1000genomes/ftp/technical/working/20140123_NA12878_Illumina_Platinum/**.vcf.gz).

### Compress or Decompress Files
[원본연결](https://github.com/googlegenomics/pipelines-api-examples/tree/master/compress)

시작하기 전에 아래와 같이 git repository를 clone 하고 compress 예제 디렉토리로 working directory를 옮긴다.
```bash
]$ cd <your git directory> 
]$ git clone https://github.com/googlegenomics/pipelines-api-examples.git
]$ cd <your git directory>/pipelines-api-examples/compress
```
여기서는  ```gs://genomics-public-data/ftp-trace.ncbi.nih.gov/1000genomes/ftp/technical/working/20140123_NA12878_Illumina_Platinum/**.vcf.gz``` 에 있는 파일을 docker ubuntu 이미지의 gzip 과 gunzip 을 이용해서 압축하고 풀면서 수행자가 가지고 있는 bucket에 결과를 복사하는 과정이다.

위의 원본에 들어가면 실행하기 위한 명령어가 나와 있다.
```bash
PYTHONPATH=.. python ./run_compress.py \
  --project YOUR-PROJECT-ID \
  --zones "us-*" \
  --disk-size 200 \
  --operation "gunzip" \
  --input gs://genomics-public-data/ftp-trace.ncbi.nih.gov/1000genomes/ftp/technical/working/20140123_NA12878_Illumina_Platinum/**.vcf.gz \
  --output gs://YOUR-BUCKET/pipelines-api-examples/compress/output \
  --logging gs://YOUR-BUCKET/pipelines-api-examples/compress/logging \
  --poll-interval 20
```
먼저 python 코드를 보면 어떤 일을 하는지 대략적으로 파악 가능함

파악 했으면 위의 명령어 실행
실행 뒤에 자신의 bucket을 확인하면 결과물이 생성된 것을 볼 수 있다.






***
# Create a Grid Engine cluster on Compute Engine
## Run workflows and common tasks in parallel
[연결](http://googlegenomics.readthedocs.io/en/latest/use_cases/run_pipelines_in_the_cloud/index.html)

## Create a Grid Engine cluster on Compute Engine
[연결](http://googlegenomics.readthedocs.io/en/latest/use_cases/setup_gridengine_cluster_on_compute_engine/index.html)