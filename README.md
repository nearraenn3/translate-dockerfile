# TOPIC : Best practices for writing Dockerfiles -TH
## General guidelines and recommendations
### Create ephemeral containers
### Understand build context
### Pipe Dockerfile through stdin
  * BUILD AN IMAGE USING A DOCKERFILE FROM STDIN, WITHOUT SENDING BUILD CONTEXT
  * BUILD FROM A LOCAL BUILD CONTEXT, USING A DOCKERFILE FROM STDIN	
  * BUILD FROM A REMOTE BUILD CONTEXT, USING A DOCKERFILE FROM STDIN
### Exclude with .dockerignore //Short
### Use multi-stage builds
### Don’t install unnecessary packages //Short
### Decouple applications
### Minimize the number of layers
### Sort multi-line arguments //Short
### Leverage build cache
### Dockerfile instructions //Short
### FROM //Short
### LABEL
### RUN  //Short
### APT-GET //Long
### USING PIPES
### CMD
### EXPOSE
### ENV
### ADD or COPY
### ENTRYPOINT
### VOLUME  //Short
### USER
### WORKDIR //Short
### ONBUILD 
