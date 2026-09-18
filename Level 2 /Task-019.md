# AWS Level 2 – Task 019: Creating a Private ECR Repository

## Scenario
The Nautilus DevOps team needs a private Amazon Elastic Container Registry (ECR) repository to store a container image.

The application files and Dockerfile are already available under `/root/pyapp` on the `aws-client` host. The image must be built, tagged as `latest`, and pushed to the private ECR repository.

## Requirements

- **Region:** `us-east-1`
- **ECR Repository:** `datacenter-ecr`
- **Repository Type:** Private
- **Application Directory:** `/root/pyapp`
- **Image Tag:** `latest`
- Build the image from the provided Dockerfile
- Push the image to Amazon ECR

## Initial State

Verify the application files:

```bash
cd /root/pyapp
ls -la
```

The directory contained:

```text
Dockerfile
app.py
requirements.txt
```

Before building the application image, Docker was tested:

```bash
docker run --rm hello-world
```

The container failed to start with:

```text
OCI runtime create failed
setting cgroup config for procHooks process caused:
can't load program: operation not permitted
```

## Troubleshooting Path

The dependency path was investigated as:

```text
Docker CLI
   ↓
Docker Daemon
   ↓
OCI Runtime
   ↓
cgroup v2
   ↓
Container Start
```

The Docker daemon could successfully pull images, but containers could not start.

Further investigation showed that `aws-client` itself was running inside a container and the environment was using an older Docker/runc stack with cgroup v2.

Even a privileged container failed:

```bash
docker run --rm --privileged python:3.8-slim python --version
```

This produced the same OCI/cgroup error.

## First Finding

The failure was not caused by:

- The Dockerfile
- `app.py`
- `requirements.txt`
- Amazon ECR
- AWS authentication
- Image tagging

The failure occurred at the Docker/OCI runtime layer in the nested lab environment.

Rather than modifying the host's Docker runtime, **Buildah** was used as an alternative image builder.

## Fix

### 1. Install Buildah

The Docker APT repository initially caused a GPG error, so the broken repository was temporarily disabled.

The Debian security repository also contained stale package references. The configured Debian snapshot repository was used so the required packages could be installed.

After correcting the package sources:

```bash
apt-get install -y buildah
```

Verify:

```bash
buildah --version
```

Result:

```text
buildah version 1.19.6
```

### 2. Install the Required OCI Runtime

The first Buildah attempt reached the Dockerfile `RUN` instruction but failed because `crun` was missing:

```text
exec: "crun": executable file not found in $PATH
```

Install it:

```bash
apt-get install -y crun
```

Verify:

```bash
crun --version
```

Result:

```text
crun version 0.17
```

### 3. Use the Fully Qualified Base Image

The Dockerfile originally referenced:

```dockerfile
FROM python:3.8-slim
```

Buildah required the registry to be explicitly specified, so it was changed to:

```dockerfile
FROM docker.io/library/python:3.8-slim
```

### 4. Build the Image

From `/root/pyapp`:

```bash
buildah bud -t datacenter-ecr:latest .
```

The build completed successfully.

Verify:

```bash
buildah images
```

Result:

```text
REPOSITORY                 TAG
localhost/datacenter-ecr   latest
```

## Create the Private ECR Repository

Create the repository:

```bash
aws ecr create-repository \
  --repository-name datacenter-ecr \
  --region us-east-1
```

Verify:

```bash
aws ecr describe-repositories \
  --repository-names datacenter-ecr \
  --query 'repositories[0].[repositoryName,repositoryUri]' \
  --output table
```

Repository URI:

```text
115067058098.dkr.ecr.us-east-1.amazonaws.com/datacenter-ecr
```

## Authenticate Buildah with ECR

```bash
aws ecr get-login-password \
  --region us-east-1 | \
buildah login \
  --username AWS \
  --password-stdin 115067058098.dkr.ecr.us-east-1.amazonaws.com
```

Result:

```text
Login Succeeded!
```

## Tag the Image

```bash
buildah tag \
  localhost/datacenter-ecr:latest \
  115067058098.dkr.ecr.us-east-1.amazonaws.com/datacenter-ecr:latest
```

Verify:

```bash
buildah images
```

Both references pointed to the same image:

```text
localhost/datacenter-ecr:latest

115067058098.dkr.ecr.us-east-1.amazonaws.com/datacenter-ecr:latest
```

## Push the Image to ECR

```bash
buildah push \
  115067058098.dkr.ecr.us-east-1.amazonaws.com/datacenter-ecr:latest
```

The image layers and manifest were successfully uploaded to Amazon ECR.

## Validation

Verify the image directly from ECR:

```bash
aws ecr describe-images \
  --repository-name datacenter-ecr \
  --query 'imageDetails[*].[imageTags,imageDigest,imageSizeInBytes]' \
  --output table
```

The repository returned:

```text
Tag:    latest
Digest: sha256:d2617b2ec132311a374dfcf98509c3a5afa029f88e8ad04a42d7839eda394f6b
```

The completed workflow was:

```text
/root/pyapp
    ↓
Dockerfile
    ↓
Buildah + crun
    ↓
datacenter-ecr:latest
    ↓
ECR Authentication
    ↓
ECR Tag
    ↓
Buildah Push
    ↓
Amazon ECR
    ↓
datacenter-ecr:latest ✅
```

## Lessons Learned

- A failed container build does not necessarily mean the Dockerfile is incorrect.
- Testing `docker run hello-world` helped isolate the problem from the application itself.
- Docker could pull images successfully while still being unable to start containers.
- The first failure was at the OCI runtime/cgroup layer of the lab environment.
- Buildah provided an alternative way to build and push the container image without relying on the broken Docker daemon execution path.
- Buildah may require fully qualified image names such as `docker.io/library/python:3.8-slim`.
- ECR authentication, image creation, tagging, and pushing should be validated independently.

## Engineering Insight

The most important troubleshooting lesson was to separate the container workflow into independent layers:

```text
Source Code
    ↓
Dockerfile
    ↓
Container Builder
    ↓
OCI Runtime
    ↓
Local Image
    ↓
Registry Authentication
    ↓
Image Tag
    ↓
Registry Push
    ↓
ECR Verification
```

When the original Docker build failed, rebuilding repeatedly would not have solved the problem because the Dockerfile was not the first failure.

By testing a minimal `hello-world` container, the problem was isolated to the container runtime. Buildah and `crun` were then used as a controlled workaround, allowing the actual ECR requirement to be completed without unnecessary changes to the lab's Docker runtime.

## Result

The private ECR repository `datacenter-ecr` was successfully created in `us-east-1`.

Despite a Docker runtime issue in the lab environment, the application image was successfully built using Buildah, tagged as `latest`, authenticated against Amazon ECR, pushed to the repository, and verified through the AWS CLI.

**Task Status:** ✅ Completed
