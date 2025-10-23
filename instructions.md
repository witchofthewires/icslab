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

    (TODO) talk about master:slave
    (TODO) overview of this portal

2. Install OpenPLC editor (TODO)
3. Start OpenPLC Editor
    a. Overview of this program (TODO)
    b. Load Blink
        i. File > Tutorials and Examples > 5. Blink
        ii. (Double) click 'Blink' in Project pane on top left
        iii. (TODO) overview of ladder logic
        iv. Run simulation
            1. In the second pane from top, click the running person icon, 'Start PLC Simulation'
            2. In bottom left pane, click sunglasses next to 'blink_led (BOOL)'
            3. In Debugger (right pane), watch the value of blink_led switch from True to False to True etc
            4. In the second pane from top, where the running person icon was, click the stop sign to stop simulation
        v. Program OpenPLC
            1. Next to the running person icon from the previous step, click the down arrow, 'Generate Program for OpenPLC Runtime'
            2. Navigate to your project folder and save the compiled output as 'blink.st' or similar.
            3. In OpenPLC, click Programs in the left pane.
            4. Under Upload Program, click Browse, and select 'blink.st'. Then click Upload Program.
            5. In the next page, give your program a title. The other fields are optional.
            6. Once done, click 'Start PLC' in the bottom left.
            7. Navigate to 'Monitoring' and watch the LED blink on and off.
            8. When done, you may click the button in bottom left to 'Stop PLC'.

3. Load an HMI in Docker
    https://openplc.discussion.community/post/alternative-hmi-12512639
4. Dummy Python load
5. Replay attack???
