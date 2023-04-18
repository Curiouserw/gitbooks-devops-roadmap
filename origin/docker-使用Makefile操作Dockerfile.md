
```bash

IMAGE_BASE = docker-registry-default.apps.okd311.curiouser.com/openshift
IMAGE_NAME = demo-springboot2
IMAGE_VERSION = latest
IMAGE_TAGVERSION = $(GIT_COMMIT)

all: build tag push

build:
  docker build --rm -f Dockerfile -t ${IMAGE_BASE}/${IMAGE_NAME}:${IMAGE_VERSION} .

tag:
  docker tag ${IMAGE_BASE}/${IMAGE_NAME}:${IMAGE_VERSION} ${IMAGE_BASE}/${IMAGE_NAME}:${IMAGE_TAGVERSION}

push:
  docker push ${IMAGE_BASE}/${IMAGE_NAME}:${IMAGE_VERSION}
  docker push ${IMAGE_BASE}/${IMAGE_NAME}:${IMAGE_TAGVERSION}
```

**makefile中的命令必须以tab作为开头(分隔符),不能用扩展的tab即用空格代替的tab。(如果是vim编辑的话,执行 set noexpandtab)。否则会报如下错误：`Makefile:10: *** multiple target patterns. Stop.`**