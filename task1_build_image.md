# Build Minecraft runtime image and upload it to ECR

## Agenda

In this lab we will do some preliminary work: create two images: one for **master** pod, second for **worker** pod, create Elastic Container Registary and push the images into it.<br>

We will use [open source project for build highly avaliable Minecraft server called "MultiPaper"](https://github.com/MultiPaper/MultiPaper).<br> All tasks were tested with version 1.92.2.

```
Please be vigilant! We are not responsible for the resources used in the project and their changes. Use project data only as part of our labs
```

## 1. Create Elastic Container Registry
First create AWS ECR registries with the following parameters:
1) `name`=`eks-minecraft-master`
2) `name`=`eks-minecraft-worker`


## 2. Create docker images

Lets build runtime images!

There are two ways to complete this task: you can do and upload the images by yourself with the requirements provided in [2.1] (as an advanced task).<br>

Or you can skip this optional step and use the script [2.2] to buid and upload the images.

### 2.1 (Optional) Requirements for advanced task: "Craft your images"

<details>

- You need to build two images one for *worker* and one for *master* using `openjdk:17` image as a base for both.

- You need to download 2 JAR-files: one for worker and one for master from https://multipaper.io/download.html.<br> Or you can build them from the [sources](https://github.com/MultiPaper/MultiPaper). We would recommend to use pre-builded binaries.<br>

Those 2 jar files should be copied to `/opt/mc_worker/` folder.

- Also you need to download the [Java edition of the Minecraft](https://www.minecraft.net/en-us/download/server) itself<br> Multipaper worker required it to start but cannot do this as we are running containers in a private network.<br> This file should be renamed to `mojang_1.19.2.jar` and pre-copied to `/opt/mc_worker/cache/` folder of the Worker container.<br>
Also you need eula.txt file to run Multipaper successfully: just create empty file with name `eula.txt` and add the following string to it `"eula=true"`. Copy `eula.txt` to `/opt/mc_worker/` folder of the Worker container.

- Master container will use port `35353` and Worker - `25565`.

- Master will be started inside the container with the following command: ```java -Dallow.multiple.connections -jar MultiPaper-Master-2.10.1-all.jar 35353```

- Worker will be started inside the container with the following command: ```java -DmultipaperMasterAddress=minecraft-service:35353 -Dproperties.online-mode=false -Deula=true -jar multipaper-1.19.2-37.jar```
 
- Build and test containers. Upload them to ECR reposirtories with appropriate tags, e.g. use build version as tag.

</details>

### 2.2 Example script to build worker and master images in automated way.

<details>


**Note**: You must change **AWS_ACCOUNT** parameter in the script to your AWS account number.
**Note**: You have to be logged in AWS account before uploading Docker images to ECR.

```
#!/usr/bin/env bash

export URL_PORTAL='https://multipaper.io/api/v2/projects/multipaper/versions/1.19.2/builds/37/downloads'
export MASTER_TAG="2.10.1-all"
export WORKER_TAG="1.19.2-37"
export MASTER_BUILD="MultiPaper-Master-"${MASTER_TAG}".jar"
export WORKER_BUILD="multipaper-"${WORKER_TAG}".jar"
export URL_MASTER="${URL_PORTAL}/${MASTER_BUILD}"
export URL_WORKER="${URL_PORTAL}/${WORKER_BUILD}"
export URL_MINECRAFT="https://piston-data.mojang.com/v1/objects/f69c284232d7c7580bd89a5a4931c3581eae1378/server.jar" #link taken forom official site https://www.minecraft.net/en-us/download/server
export MASTER_PORT='35353'
export WORKER_PORT='25565'
export REGION='eu-west-1'
export AWS_ACCOUNT="123456789012"
export AWS_REPO="${AWS_ACCOUNT}.dkr.ecr.${REGION}.amazonaws.com"
export MASTER_REPO="${AWS_REPO}/eks-minecraft-master"
export WORKER_REPO="${AWS_REPO}/eks-minecraft-worker"

cd $(mktemp -d) && pwd

touch Dockerfile-master Dockerfile-worker eula.txt
echo "eula=true" >> eula.txt

cat > Dockerfile-master <<EOF
FROM openjdk:17

COPY ${MASTER_BUILD} /opt/mc_master/${MASTER_BUILD}
WORKDIR /opt/mc_master
EXPOSE ${MASTER_PORT}

ENTRYPOINT [ "java", "-Dallow.multiple.connections", "-jar", "${MASTER_BUILD}", "${MASTER_PORT}" ]
EOF

cat > Dockerfile-worker <<EOF
FROM openjdk:17

COPY ${WORKER_BUILD} /opt/mc_worker/${WORKER_BUILD}
COPY server.jar /opt/mc_worker/cache/mojang_1.19.2.jar
COPY eula.txt /opt/mc_worker/eula.txt
WORKDIR /opt/mc_worker
EXPOSE ${WORKER_PORT}

ENTRYPOINT [ "java", "-DmultipaperMasterAddress=minecraft-service:${MASTER_PORT}", "-Dproperties.online-mode=false", "-Deula=true", "-jar", "${WORKER_BUILD}" ]
EOF

curl ${URL_MASTER} -O -: ${URL_WORKER} -O -: ${URL_MINECRAFT} -O

docker build -f Dockerfile-master -t ${MASTER_REPO}:v${MASTER_TAG} .
docker build -f Dockerfile-worker -t ${WORKER_REPO}:v${WORKER_TAG} .

aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${AWS_REPO}
docker push ${MASTER_REPO}:v${MASTER_TAG}
docker push ${WORKER_REPO}:v${WORKER_TAG} 
```

</details>
</details>

## Definition of done

Your two images placed to created Elastic Container Registary.

## Clean-up

Do not forget to stop and delete your resources on the end of practice. You can use Tags to locate required resources.

