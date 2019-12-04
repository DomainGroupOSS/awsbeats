# This repo
We are currently using this `awsbeats` repo to build a Docker image for building Elastic's beats with the addition of a custom plugin that allows us to output data to AWS Kinesis, which is in turn pulled by Logstash to push to our Elasticsearch clusters

Elastic provide two versions of beats:
* An open source version covered under Apache License 2.0
* A version supported by Elastic through the Elastic License called `x-pack`

Both are accessible from the `elastic/beats` repo and you can build and compile them from source yourself

# The problem
This repo contains functionality to build the open source version of beats, however the `x-pack` version does not currently work due to the change in build process for the `x-pack` binaries.

The `x-pack` version contains additional, enhanced functionality that can be used when using a premium Elasticsearch license (which we have) so it may be preferable in some cases to use the `x-pack` version. An example of this could be the `enroll` functionality built in Metricbeat and Filebeat which is only accessible from the `x-pack` version.

# Building Yourself
Building the beat on your own will help you get a better understanding of both the repos, you should also use the `golang:<version>` Docker image as `awsbeats` does the same thing

You should also use the proper `golang` way of cloning repos by using `go get` (building x-pack **requires** this)
```
$ go get github.com/elastic/beats
```
When you do this, the `beats` repo will be located in your `GOPATH` already, `cd` into it.
## Building OSS Beats
If you read through the Makefile and Dockerfile of this repo, you can learn how to build it yourself but here are a few simple steps:
1. Change into the folder of the beat you wish to build
2. Run `make <beat-name>` where `<beat-name>` is the name of the beat.
3. A binary should be generated which will be the built open source beat

## Building X-Pack Beats
X-Pack utilise a `Magefile` instead of a `Makefile` to build the binaries, so installing `mage` is required
1. Get `mage` by running `go get github.com/magefile/mage`
2. There will be a `mage` binary sitting in `$GOPATH/bin`. Make sure to add it to your path or alias it, you could probably move it to another folder already in your path but I didn't want to move it
3. Change into the folder `x-pack/<beat-name>` where `<beat-name>` is the name of the beat you wish to build.
4. Run `mage build`. Wait as this may take a little while with no real indication of it working as it doesn't show output
5. If done correctly, you should have the beats binary generated which will be the `x-pack` version

# Changes to this repo
If you have had a go at building a `x-pack` beat yourself, you will notice that the process isn't completely different to the OSS one. The main changes are:
* Using `mage` instead of `make`, which requires `go get github.com/magefile/mage` and sourcing the `mage` binary
* We are using `mage build` as a generic function, not needing to specify beat name
* We are building from the `x-pack/<beat-name>` folder instead of `<beat-name>` folder

The rest stays the same as the Dockerfile and Makefile of the `awsbeats` repo does the other parts

With that said, the quick changes to get it to work properly with this repo is changing the following in the `Makefile`
```
endif
	@cd "$$GOPATH/src/$(BEAT_GO_PKG)" &&\
	make $(BEAT_NAME) &&\
	mv $(BEAT_NAME) "$(CURDIR)/target/$(BEAT_NAME)-$(BEATS_VERSION)-go$(GO_VERSION)-$(GO_PLATFORM)"
else
```
```
endif
	go get github.com/magefile/mage
	export mage="/root/go/bin"
	@cd "$$GOPATH/src/$(BEAT_GO_PKG)" &&\
	mage build &&\
	mv $(BEAT_NAME) "$(CURDIR)/target/$(BEAT_NAME)-$(BEATS_VERSION)-go$(GO_VERSION)-$(GO_PLATFORM)"
else
```

And then build by using a command such as this
```
make dockerimage BEATS_VERSION=7.4.0 AWSBEATS_VERSION=0.3.5 BEAT_NAME=metricbeat BEAT_GO_PKG=github.com/elastic/beats/x-pack/metricbeat
```
