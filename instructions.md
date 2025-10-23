## Overview
GOAL: Participants in this lab will construct a 'digital twin' of a turbine generator, consisting of a PLC, HMI, and dummy Python load.

## Detailed Instructions
0. Install prerequisites:
    - Docker
    - Git
1. First, load OpenPLC in Docker (instructions from https://github.com/thiagoralves/OpenPLC_v3)

        git clone https://github.com/thiagoralves/OpenPLC_v3.git
        cd OpenPLC_v3
        docker build -t openplc:v3 .
        docker run -it --rm --privileged -p 8080:8080 openplc:v3

    If you receive the following error:

        docker: Error response from daemon: failed to create task for container: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "./start_openplc.sh": stat ./start_openplc.sh: no such file or directory: unknown.

    then back out, delete the folder, and run the steps again with this git clone command:

        git -c core.eol=lf -c core.autocrlf=false clone https://github.com/thiagoralves/OpenPLC_v3.git

    once done, should say that it's running on TCP 8443. open a web browser and go to http://127.0.0.1:8080. you should see the following:

        picture of good webserver

    log in with default creds 'openplc:openplc'

2. Load an HMI in Docker
    https://openplc.discussion.community/post/alternative-hmi-12512639
3. Dummy Python load
4. Program the whole thing
5. Replay attack???
