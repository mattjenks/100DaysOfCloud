# A consistent cloud dev environment

## Introduction

I have a work computer (Late 2019 MBP) and a personal computer (Late 2013 MBP). My work computer is currently being repaired and besides that, I switch between them depending ont he project I am working. Sometimes there is overlap. When working in a cloud environment and using CLI programs, I find I like to have consistency across my laptops.

## Prerequisite

Admittedly, I have  DevSecOps experience, where I have done projects using:
- [zsh](http://zsh.sourceforge.net/Doc/Release/index.html#Top)
- [zsh functions](http://zsh.sourceforge.net/Doc/Release/Functions.html#Functions)
- [ohmyz.sh](https://ohmyz.sh/)
- [docker](https://www.docker.com/get-started)
- [Github Actions](https://github.com/features/actions)
- It would also be nice to have some utilities always at the ready, such as
  - [aws](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
  - wget
  - jq
  - curl
  - gnupg
  - [keybase](https://keybase.io/)
- and I want to learn about [Terraform](https://www.terraform.io/)

## Use Case

- I'd like to be able to know I am running the same command and will get the same output no matter which computer I use. This means I have to keep versions synchronized on each laptop. I am going to accomplish this with docker and creating functions to wrap the docker commands.

## Cloud Research/ Try yourself

### Step 1 download and test container
- previously, I had created a base dev environment, which is available in [GitHub](https://github.com/GigaTech-net/dev)
- I updated the [Dockerfile](https://github.com/GigaTech-net/dev/blob/master/Dockerfile) to add consistent versions of the above utilities.
- I had previously added a simple [actions workflow](https://github.com/GigaTech-net/dev/blob/master/.github/workflows/workflow.yml) to build and publish this on the master branch
  - Hiding the secrets (for Docker publishing) of course!
  - This publishes the Docker image to my (GigaTECH Docker Hub](https://hub.docker.com/repository/docker/gigatech/dev)
  
```zsh
-> docker pull gigatech/dev
-> docker container run --rm gigatech/dev aws --version
aws-cli/2.0.31 Python/3.7.3 Linux/4.19.76-linuxkit botocore/2.0.0dev35
```

This will work the same on any of my linux based laptops

### Step 2 create zsh functions 

I don't want to have to type this "docker container run --rm gigatech/dev aws" each time. So alias these out with zsh functions ...

In my ~/.zshrc

```zsh
# development commands via docker

function gtdev () {
  if [ $# -eq 0 ]; then
    docker container run --rm -it -w /home/gigatech/workdir \
      -v "$(pwd)":/home/gigatech/workdir \
      -v "${HOME}":/home/gigatech \
      gigatech/dev zsh
  else
    docker container run --rm -it -w /home/gigatech/workdir \
      -v "$(pwd)":/home/gigatech/workdir \
      -v "${HOME}":/home/gigatech \
      gigatech/dev $@
  fi

}

function tf () {
   gtdev terraform $@
}

function aws () {
   gtdev aws $@
}
```

Note that I set the working dir via the -w parameter as well as set my current directory to the working dictory and overlay my home directory into the container, both via the -v parameters, respectively. This will allow me to use my zsh config and my aws configuration (profile configuration): "aws configure --profile my_profile"
These shortform commands now work ...

```zsh
-> aws --version
aws-cli/2.0.31 Python/3.7.3 Linux/4.19.76-linuxkit botocore/2.0.0dev35
-> tf --version
Terraform v0.12.28
```

### Step 3 create aws profiles for personal and employer cloud environments

```zsh
-> aws configure --profile my_personal_cloud
accesskey
secretkey
default region
output format
```

repeat for employer account(s)

and now access via:

```zsh
-> aws cmd --profile my_personal_cloud
```

## ☁️ Cloud Outcome

✍️ I know have a consistent development setup no matter what laptop I am working on.

## Next Steps

✍️ Use [IaC](https://en.wikipedia.org/wiki/Infrastructure_as_code) to put AWS setup in Github ... Hello [Terraform](https://www.terraform.io/)!

## Social Proof

✍️ Show that you shared your process on Twitter or LinkedIn

[Consistent cloud dev environment](link)
