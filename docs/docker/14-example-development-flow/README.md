# Example Docker Development Flow

In this section we're going to see an example development flow for Dockerized images.

We'll build the Docker image, push it to a remote registry and then run a Docker container based on this image. Then we'll modify the application codebase and repeat the procedure.

We're going to use the same application code that was used in the build section.

## Prerequisites

- You understand basic [Docker concepts](https://docs.docker.com/get-started/overview/).
- Clone this Git repository for fetching the web application code and files needed to build our Docker image.
- Have access and can login to [CERN's IT remote registry web interface](https://registry.cern.ch/).

## Login to CERN's IT remote Registry

To login to CERN's IT remote registry:

```bash
docker login registry.cern.ch
```

!!!note
    If you login for the first time, it will prompt for credentials. These credentials can be found in the [web interface of the registry](https://registry.cern.ch/), under the user's profile section from the main menu.

## Initial Application

### Build the image

To build the initial Docker image (**make sure you replace `USERID` with your CERN username**):

```bash
$ docker build --tag registry.cern.ch/cms-daq-workshop/USERID/flow-1:latest \
    -f docs/docker/07-build/dockerfile/Dockerfile.alpine-base \
    app/


[+] Building 1.5s (12/12) FINISHED
 => [internal] load build definition from Dockerfile.alpine-base                                                                          
 => => transferring dockerfile: 259B                                                                                                      
 => [internal] load .dockerignore                                                                                                         
 => => transferring context: 2B                                                                                                           
 => [internal] load metadata for docker.io/library/golang:1.19.3-alpine3.16                                                                      1.4s
 => [internal] load build context                                                                                                         
 => => transferring context: 81B                                                                                                          
 => [1/7] FROM docker.io/library/golang:1.19.3-alpine3.16@sha256:dc4f4756a4fb91b6f496a958e11e00c0621130c8dfbb31ac0737b0229ad6ad9c         
 => CACHED [2/7] WORKDIR /app                                                                                                             
 => CACHED [3/7] COPY go.mod ./                                                                                                           
 => CACHED [4/7] COPY go.sum ./                                                                                                           
 => CACHED [5/7] RUN go mod download                                                                                                      
 => CACHED [6/7] COPY *.go ./                                                                                                             
 => CACHED [7/7] RUN CGO_ENABLED=0 go build -o /cms-daq-simple-app                                                                        
 => exporting to image                                                                                                                    
 => => exporting layers                                                                                                                   
 => => writing image sha256:4ec040bb3c4279b4fd72625c2aa0268b0f5c2ff981a9bb6c3403656c95424994                                              
 => => naming to registry.cern.ch/cms-daq-workshop/thrizopo/flow-1:latest
```

After the above command finishes successfully, Docker has successfully built our image locally and assigned the `registry.cern.ch/cms-daq-workshop/flow-1/USERID:latest` tag to it.

### Push the Docker Image to the remote Registry

Once you're logged in to the remote registry, you can push the local image to it (**make sure you replace `USERID` with your CERN username**):

```bash
$ docker push registry.cern.ch/cms-daq-workshop/USERID/flow-1:latest

The push refers to repository [registry.cern.ch/cms-daq-workshop/thrizopo/flow-1]
a95b38e10c86: Pushed
fffe423b3246: Pushed
9188977abd30: Pushed
c27ea7a239a9: Pushed
d258abd81330: Pushed
9e0bb54ef79d: Pushed
daba023ab15f: Pushed
469fd49452ac: Pushed
5bed14ed4195: Pushed
17bec77d7fdc: Pushed
latest: digest: sha256:89abecafe7fa5d20892f37b7d1846bce0958059a0f8368a311e824cc7f30c69e size: 2408
```

### Run a Container based on the initial built image

After building the container image and uploading it to the remote registry, we can use this image to run/start a container from any host that has access to the remote registry (**make sure you replace `USERID` with your CERN username**):

```
$ docker run -d --name cms-daq-flow-1 \
    -p 8080:8080 registry.cern.ch/cms-daq-workshop/USERID/flow-1:latest

bc7a9367aeefb281ab4f7da6aaa09f50b14307a669aa43d235e7e2f3d96ff653
```

The above command should create a new container named `cms-daq-flow-1` (if it doesn't already exist in your system), based on the `registry.cern.ch/cms-daq-workshop/USERID/flow-1:latest` image.

As we're mapping the port `8080` from the container to the port `8080` of our machine, our Golang web application should be reachable from our local machine via:

```
$ curl http://localhost:8080

Hello CMS DAQ Group!
```

Alternatively, [http://localhost:8080/](http://localhost:8080/) should be also accessible from the browser.

### Clean up

Now, let's stop and remove the container:

```bash
$ docker container stop cms-daq-flow-1

$ docker container rm cms-daq-flow-1
```

## Modified Application

### Modify the application code

Now, let's modify the web application code and re-built the Docker image:

```diff
-fmt.Fprintf(rw, "Hello CMS DAQ Group!")
+fmt.Fprintf(rw, "My modified code!")
```

### Build the Docker image

To re-build the Docker image (**make sure you replace `USERID` with your CERN username**):

```bash
$ docker build --tag registry.cern.ch/cms-daq-workshop/USERID/flow-2:latest \
    -f docs/docker/07-build/dockerfile/Dockerfile.alpine-base \
    app/


[+] Building 1.1s (12/12) FINISHED
 => [internal] load build definition from Dockerfile.alpine-base                                                                                                                   
 => => transferring dockerfile: 49B
 => [internal] load .dockerignore
 => => transferring context: 2B
 => [internal] load metadata for docker.io/library/golang:1.19.3-alpine3.16
 => [1/7] FROM docker.io/library/golang:1.19.3-alpine3.16@sha256:d171aa333fb386089206252503bc6ab545072670e0286e3d1bbc644362825c6e
 => [internal] load build context
 => => transferring context: 1.91kB
 => CACHED [2/7] WORKDIR /app
 => CACHED [3/7] COPY go.mod ./
 => CACHED [4/7] COPY go.sum ./
 => CACHED [5/7] RUN go mod download
 => CACHED [6/7] COPY *.go ./
 => CACHED [7/7] RUN CGO_ENABLED=0 go build -o /cms-daq-simple-app
 => exporting to image
 => => exporting layers
 => => writing image sha256:aa9331b0c14499b3ca98c9915df8cf08cf37dc1c4f14ddfccb19fc46d784c1d3
 => => naming to registry.cern.ch/cms-daq-workshop/thrizopo/flow-2:latest
```

### Push the Docker Image to the remote Registry

Push the local image to it (**make sure you replace `USERID` with your CERN username**):

```bash
$ docker push registry.cern.ch/cms-daq-workshop/USERID/flow-2:latest

The push refers to repository [registry.cern.ch/cms-daq-workshop/thrizopo/flow-2]
a95b38e10c86: Pushed
fffe423b3246: Pushed
9188977abd30: Pushed
c27ea7a239a9: Pushed
d258abd81330: Pushed
9e0bb54ef79d: Pushed
daba023ab15f: Pushed
469fd49452ac: Pushed
5bed14ed4195: Pushed
17bec77d7fdc: Pushed
latest: digest: sha256:7b4644c56cd3f419ddea115cd7aa3c7356106dadeca9171ca661318d031afc73 size: 2408
```

### Run a Container based on the initial built image

Let's run a new container with the modified application code (**make sure you replace `USERID` with your CERN username**):

```
$ docker run -d --name cms-daq-flow-2 \
    -p 8081:8080 registry.cern.ch/cms-daq-workshop/USERID/flow-2:latest

07a638a6a6ec553148dbc654c55634c41b546f224bf83f9b560f4f19ce530283
```

The above command should create a new container named `cms-daq-flow-2` (if it doesn't already exist in your system), based on the `registry.cern.ch/cms-daq-workshop/USERID/flow-2:latest` image.

As we're mapping the port `8081` from the container to the port `8080` of our machine, our web application should be reachable from our local machine via:

```
$ curl http://localhost:8081

My modified code!
```

As you can see, the new container returned the modified text response.

### Clean up

Let's stop and remove the container:

```bash
$ docker container stop cms-daq-flow-2

$ docker container rm cms-daq-flow-2
```

## Notes

- As mentioned in previous sections, one can use a few best practices in order to minimize the build time of our Docker image in order to save CPU cycles, network traffic and storage space. Some of these practices include using minimal base images and using Docker's build cache.
