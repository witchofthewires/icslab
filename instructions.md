GOAL: Participants in this lab will construct a 'digital twin' of a turbine generator, consisting of a PLC, HMI, and dummy Python load.

0. Install prerequisites:
    Docker
    Git
1. First, load OpenPLC in Docker (instructions from https://github.com/thiagoralves/OpenPLC_v3)
    git clone https://github.com/thiagoralves/OpenPLC_v3.git
    cd OpenPLC_v3
    docker build -t openplc:v3 .
    docker run -it --rm --privileged -p 8080:8080 openplc:v3
2. Load an HMI in Docker
    https://openplc.discussion.community/post/alternative-hmi-12512639
3. Dummy Python load
4. Program the whole thing
5. Replay attack???
