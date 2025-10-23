## Overview
This repository provides instructions for creating a 'digital twin' ICS lab which behaves identically to production ICS/OT gear at the network level. This lab will mimic a turbine generator and consist of a PLC, HMI, and dummy Python load. It contains no modern security measures and is vulnerable to well-understood attacks such as replays and MitM, making it ideal for cybersecurity education and training.

This guide may be understood as instructions for building a series of [Black Triangles](https://rampantgames.com/blog/?p=7745). The term was coined by Jay Barnson while working at the video game company SingleTrac, which made such classics as Twisted Metal (emphasis added, as well as some commas):
> It was sometime in my first week, possibly my first or second day. In the main engineering room, there was a whoop and cry of success.
>
> Our company financial controller and acting HR lady, Jen, came in to see what incredible things the engineers and artists had come up with. Everyone was staring at a television set hooked up to a development box for the Sony Playstation. There, on the screen, against a single-color background, was a black triangle.
>
> “It’s a black triangle,” she said in an amused but sarcastic voice. One of the engine programmers tried to explain, but she shook her head and went back to her office. I could almost hear her thoughts… “We’ve got ten months to deliver two games to Sony, and they are cheering over a black triangle? THAT took them nearly a month to develop?”
>
> What she later came to realize (and explain to others) was that the black triangle was a pioneer. It wasn’t just that we’d managed to get a triangle onto the screen. That could be done in about a day. It was the journey the triangle had taken to get up on the screen. It had passed through our new modeling tools, through two different intermediate converter programs, had been loaded up as a complete database, and been rendered through a fairly complex scene hierarchy, fully textured and lit (though there were no lights, so the triangle came out looking black). <em><strong>The black triangle demonstrated that the foundation was finally complete, the core of a fairly complex system was completed, and we were now ready to put it to work doing cool stuff</em></strong>. By the end of the day, we had complete models on the screen, manipulating them with the controllers. Within a week, we had an environment to move the model through.
>
> Afterwards, we came to refer to certain types of accomplishments as “black triangles.” These are important accomplishments that take a lot of effort to achieve, but upon completion you don’t have much to show for it only that more work can now proceed. It takes someone who really knows the guts of what you are doing to appreciate a black triangle.

The construction of the lab thus proceeds in stages. First, an OpenPLC Docker image must be created as a virtual PLC. This can be programmed with OpenPLC editor to blink an LED (the classic cyber-physical 'hello world'), and this behavior can then be validated in the OpenPLC web console. Tools such as the Radzio Modbus Master Simulator and Wireshark can then be used to verify that the new PLC responds to Modbus queries. This is Black Triangle 1.

Next, an HMI must be built in Docker. This lab will use the FOSS HMI Fuxa. Once the HMI is constructed, it may be configured to display data from the PLC, send queries, and display responses. Wireshark can again be used to verify that this network traffic is proceeding as expected - this network communication is one of the most vulnerable to malicious interference, and so it must be thoroughly understood. The HMI should now also provide an intuitive interface to monitor and control the PLC - this is Black Triangle 2. Note that an attacker can manipulate Black Triangle 1 (network traffic) while leaving Black Triangle 2 (HMI display) unaffected - this is a common technique used by attackers to hide compromise from system operators, such as the authors of [Stuxnet](https://www.wired.com/2014/11/countdown-to-zero-day-stuxnet/).

## Detailed Instructions
### Install prerequisites:
- Docker
- Git
- Wireshark
- Radzio
### Install OpenPLC
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

    <img src="static/good_webserver.png" alt="Fuxa Editor" width="800"/>

    log in with default creds 'openplc:openplc'

    (TODO) talk about master:slave
    (TODO) overview of this portal

### Install OpenPLC Editor
Install OpenPLC editor (TODO)

### Start OpenPLC Editor
1. Overview of this program (TODO)
2. Load Blink
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
    vi. View traffic
        1. If you turned off the PLC in the last step, turn it on again.
        2. Load up Wireshark, capture on 'Adapter for loopback traffic capture' with filter 'port 502'
        3. Open Radzio, go to Connection settings, ensure Modbus TCP is selected, ensure the IP address is '127.0.0.1' and port is 502, then click OK. See screenshot below.

    <img src="static/radzio.png" alt="Fuxa Editor" width="800"/>

4. In Wireshark, click the blue fin in top left, 3rd pane, to start packet capture. From the top pane in Radzio, select Connection > Connect. You should see traffic in Wireshark that resembles the following (capture was stopped following 4 query-response cycles).

    <img src="static/wireshark1.png" alt="Fuxa Editor" width="800"/>

### Install HMI

#### Fuxa
1. PROS - 
    - english language docs
    - more recently updated

2. Install
            
        // from https://github.com/frangoteam/FUXA
        // ---------------------------------------
        // basic
        docker pull frangoteam/fuxa:latest
        docker run -d -p 1881:1881 frangoteam/fuxa:latest

        // persistent storage of application data (project), daq (tags history), logs and images (resource)
        docker run -d -p 1881:1881 -v fuxa_appdata:/usr/src/app/FUXA/server/_appdata -v fuxa_db:/usr/src/app/FU

        // with Docker compose
        // persistent storage will be at ./appdata ./db ./logs and ./images
        wget https://raw.githubusercontent.com/frangoteam/FUXA/master/compose.yml
        docker compose up -d
            

3. Navigate to http://127.0.0.1:1881 in browser. Screen should be mostly blank, with blue circle in bottom left and orange circle in bottom right. Click the blue circle, then click Editor. It should look like the following.

    <img src="static/fuxa_editor.png" alt="Fuxa Editor" width="800"/>

4. Click the gear in the top left, and click Plugins. Drat, Modbus-TCP is not installed. We can add that by:
    a. Go to terminal, and type "docker ps" to view running images.
    b. Find the fuxa container in the output. Note its UID at the start of the line, or rather, the first 4 characters of this UID (eg: d931).
    c. "docker exec -it $ID bash"
    d. Execute 'whoami; hostname'. You should get output like "root\n$FULL_UID".
    e. "npm install modbus-tcp". Should get 'added 1 packages in 2s' or similar.
    f. 'exit'. Then, 'docker restart $ID'.
    g. In Fuxa, click the gear, click Connections. Click the Plus in the bottom right. Select type 'ModbusTCP', and fill out the connection as follows (note that Docker IP, not localhost, must be used; TODO how get this info)
    ![Connections Property Screen](static/openplc_fuxa_connection.png)
    h. Click the 'link' icon on the connection to open the Connections Settings screen. Click the Plus in the bottom right. Fill out the connection form to match the Tag Property popup in the screenshot below (note that the address offset is 4, not 3; Fuxa counts the first coil as 1, not 0). Click OK. After a few seconds, the Connection settings page should resemble the screenshot below.

    <img src="static/openplc_fuxa_coil_connection.png" alt="Fuxa Editor" width="800"/>
    i. Go to other screen and make blinking red light (TODO)
    j. You can use tcpdump to get an internal packet capture and show it in Wireshark (TODO)

#### ScadaBR
b. ScadaBR
    i. PROS -
        - integrates very well with OpenPLC
https://openplc.discussion.community/post/alternative-hmi-12512639

### Writing Ladder Logic
We're almost done! We now have an OpenPLC container talking to a Fuxa container over the industry-standard Modbus protocol. However, at this point, the HMI can only query the output of the PLC. Let's write a new program for the PLC that will turn the light on and off based on input from the HMI. 

### Replay Attack
The lab is complete! Let's use it to test out a classic ICS/OT pentesting technique, the replay attack!

### Conclusion

## Acknowledgments
- Huge thanks to Dr. Thiago Alves for all his work on the OpenPLC project.
- Thanks to seafoxc for their Youtube tutorial on [connecting Fuxa to OpenPLC](https://www.youtube.com/watch?v=OQA5eVge0l8).