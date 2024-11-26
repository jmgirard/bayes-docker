# rocker-bayes
- Dockerfile for rocker-bayes (v3.3)
- Built on top of [rocker/rstudio:4.4.1](https://github.com/rocker-org/rocker-versioned2/blob/master/dockerfiles/rstudio_4.4.1.Dockerfile) (Ubuntu 22.04)
- Multi-architecture support for AMD64 and ARM64
- Installs CmdStan 2.35.0
- Installs packages: brms cmdstanr easystats effects ggeffects patchwork rstan rstanarm tidyverse

# How to use

## Option 1: Pull and run image
Most users will want to just install Docker Desktop, pull the image, and run it.<br />
Then navigate to <http://localhost:8787> and enter "rstudio" and "pass".

```
docker pull jmgirard/rocker-bayes
docker run -e PASSWORD=pass -p 8787:8787 jmgirard/rocker-bayes
```

## Option 2: Build image locally
You could also download the Dockerfile from GitHub and build it yourself.<br />
Note that rstanarm is slow to build, so skip that if you don't plan to use it.<br />
Then navigate to <http://localhost:8787> and enter "rstudio" and "pass".<br />
You can also customize the rstudio port and password in `.env`.

```
git clone https://github.com/jmgirard/rocker-bayes.git
cd rocker-bayes
docker-compose up -d
```

# Test between-and-within-chain parallelization
Note that this small model won't get much benefit from within-chain parallelization. <br />
It's just used to quickly test that everything is working.

```r
library(brms)
fit_serial <- brm(
  count ~ zAge + zBase * Trt + (1|patient),
  data = epilepsy, family = poisson(),
  chains = 4, cores = 4, backend = "cmdstanr"
)
fit_parallel <- update(
  fit_serial, chains = 2, cores = 2,
  backend = "cmdstanr", threads = threading(2)
)
```

# How to build multi-architecture manifest
These notes are more for me, but perhaps others can learn from them.

## Build on Windows for linux/amd64

```
# git clone https://github.com/jmgirard/rocker-bayes.git
# cd rocker-bayes
docker buildx build --platform linux/amd64 --load --push \
  -f bayes_4.4.1.Dockerfile -t jmgirard/rocker-bayes:amd64 .
```

## Build on Mac (Apple Silicon) for linux/arm64

```
# git clone https://github.com/jmgirard/rocker-bayes.git
# cd rocker-bayes
docker buildx build --platform linux/arm64 --load --push \
  -f bayes_4.4.1.Dockerfile -t jmgirard/rocker-bayes:arm64 .
```

## Create multi-architecture manifest list

```
docker manifest create jmgirard/rocker-bayes:latest \
  --amend jmgirard/rocker-bayes:amd64 \
  --amend jmgirard/rocker-bayes:arm64

docker manifest annotate jmgirard/rocker-bayes:latest \
  jmgirard/rocker-bayes:amd64 --arch amd64

docker manifest annotate jmgirard/rocker-bayes:latest \
  jmgirard/rocker-bayes:arm64 --arch arm64

docker manifest push jmgirard/rocker-bayes:latest
```

