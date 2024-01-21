# Dockerfile best practices

## A. DevOps best practices for the creation of Dockerfiles

Containers are great as a method to create and deploy software, but unless certain guidelines are followed, the quality and security of these resources can impact the business. Remember, containers deployed to the DEV environment usually pass testing and make their way to PROD via UAT.

While a lot of care is usually taken to make sure that the software functions properly and passes the required tests, it can be easy to forget about first principles. Sometimes a base image is selected quickly, and the requisite software is installed into the container to get coding started ASAP. This approach coupled with a lack of established procedures can result in the following issues:

1. Security: The final runtime has the remnants of previous compilation tools, etc that increase the available attack surface.
2. Security: Container is running as root. Use a non-root user.
3. Security: Base image is not the latest. ECR images should be screened for critical vulnerabilities.
4. Performance: Container size is too large. The image should definitely be under 1 GB and ideally under 500MB.
5. Performance: Docker Build stage takes too long. Use caching & multistage builds

### Further reading
[Docker Security 101](https://stackrox.io/blog/docker-security-101/)

**TL;DR: Neglecting the basics in the haste to get a MVP often builds up technical debt & security issues that make their way to production. Spend the extra 20-30 minutes optimising the Dockerfile and checking for security issues. Avoid being penny-wise, but pound foolish!**


## B. Notes on CI/CD resources
DevOps resources that build and test software products are finite and are shared across several teams. Please make sure that any docker build step is performant so that non-optimised builds do not hog resources and slow down other people's  work. Putting a badly performing Dockerfile into the Cl system and thinking that caching will speed-up subsequent builds is an example of fallacious thinking because:
1. There are multiple Bamboo build agents and these agents will not have cached copies of your docker layers.
2. The bamboo agent cache gets cleared quite aggressively. So your layers will be removed every few days
3. In the future someone else is very likely to modify the Dockerfile. It is unlikely to stay static forever! Don't push the problem onto someone else to deal with. It's not cool.
4. Use ECR caching and multistage builds to avoid these performance issues (detailed below).

## C. Some important docker guidelines

1. Keep image sizes small Try to use slim stretch/buster/jessie/bullseye images vs alpine because alpine does not support all package dependencies. It is also hard to debug, and not everyone has familliarity  with it. In contrast, most people in the field understand debian/ubuntu-like commands. Further reading:

* [Moving away from Alpine](https://dev.to/asyazwan/moving-away-from-alpine-30n4)
* [Comparison of linux container images](https://crunchtools.com/comparison-linux-container-images)
* [Alpine docker Python](https://pythonspeed.com/articles/alpine-docker-python)
* [The problem with docker and Alpines package pinning](https://stschindler.medium.com/the-problem-with-docker-and-alpines-package-pinning-18346593e891)
* [Base image Python docker images](https://pythonspeed.com/articles/base-image-python-docker-images)

2. Don't create a Python virtual environment (venv) within the container because the container is already an isolated environment that is discrete from the host OS.
3. Use multistage builds to save time and reduce the size of your container. If your build has lots of steps that involve downloading source code for compilation, these files should not be present in the final image that will eventually make it's way to Production. Compile as a last resource. You can usually copy pre-compiled programs from publically available dockerfiles.
4. Don't use the latest tag. This makes it hard to debug issues as we have to track down which "latest" was causing teh error and over time the tracking 1complexity compounds.
5. Don't run more than one process in a container. Each container should be a specialised, discrete unit.
6. **Do NOT use root** to run your application because this is a security issue. A specialised non-privileged user should be running the container. Use Docker build arguments to set this up. Compromised root-containers allow:
  * Attackers to modify the container system so that they can edit the host filesystem, install system packages at runtime, etc.
  * Containers to bind privileged ports under 1024.

7. Containers can be spun up to execute various tests and then destroyed after the tests are finished. In the course of this process, a bind-mount volume can be created between the host machine & the container. In this case it is imoportant not to write files to the host as root. If this is done on a build agent, it breaks the build because the bamboo user cannot delete root-owned files. To avoid this, setup a non-root user as per point 6. Use the following BASH code to build the container so that the container-user shares the same UID/GID that the host-user has.

```bash
## Find user id of currently logged in user assign that to the container user

### That way, files written to bind mounted volumes will not have permission issues
USER_ID-$(id -u)

docker build \
  --build-arg USER_ID=SUSER_ID \
  --file ${PATH_TO_DOCKER_FILE} \
  --tag 1.0.3 .

```

8. Screen the base image for critical vulnerabilities. Using the latest patched image is essential. After uploading the image to ECR, check the vulnerabilities and take action if any issues are found. ECR repos should be set to 'scan on push' to enable automatic security scanning for every image committed to the repo.

9. Use our DevOps script to cache layers in ECR (even for test containers). This will increase build speeds dramatically. Briefly, this involves using the `docker build --cache-from` option
```bash
## login to ECR & setup buildkit
eval $(aws ecr get-login --no-include-email --profile ${AWS_PROFILE} --region eu-west-1 | sed 's|https://||')
export DOCKER_BUILDKIT=1 #<------ This needs to be set

## Find last tag
LAST_TAG=$(aws ecr describe-images --repository-name ${ECR_REPO_NAME} --query 'sort_by(imageDetails,& imagePushedAt)[-1].imageTags[0]' | sed 's|"||g')

## Build with ECR cache
docker build \
  --cache-from 012345678901.dkr.ecr.eu-west-1.amazonaws.com/my-amazing-app:${LAST_TAG} \
  --tag my-amazing-app:${NEW_TAG}
  --build-arg BUILDKIT_INLINE_CACHE=1 \
  $PATH_TO_DOCKERFILE

```

10. Use multistage builds to improve speed, reduce container size and increase security by removing unnecessary dev tools. Use in conjunction with ECR caching to help reduce costs.

### Further reading:
* [Why non-root containers are important for security](https://docs.bitnami.com/tutorials/why-non-root-containers-are-important-for-security)
* [Run docker nginx as non-root user](https://www.rockyourcode.com/run-docker-nginx-as-non-root-user)
* [Advanced dockerfiles faster builds and smaller images using buildkit and multistage builds](https://www.docker.com/blog/advanced-dockerfiles-faster-builds-and-smaller-images-using-buildkit-and-multistage-builds)
* [Caching parallelism in docker multistage builds](https://kgrz.io/caching-parallelism-in-docker-multi-stage-builds.html)
* [Caching docker layers on serverless buildhosts with multistage builds --target, and --cache-from](https://andrewlock.net/caching-docker-layers-on-serverless-build-hosts-with-multi-stage-builds---target,-and---cache-from)
