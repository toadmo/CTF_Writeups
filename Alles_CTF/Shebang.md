ALLES CTF 2020

Shebang
Category: Bash
Difficulty: Easy

Process:
The start of this challenge was analyze the python source code to understand what the program was doing. The program initialized with the characters #!d, which looked like a shebang. Normally, however, a shebang would be in the form of #!bin/sh or #!bin/bash, which would tell the file or user that the file is to be run in either a shell or bash. Unfortunately, the challenge provides an immutable string to start out the shebang, which included a d. 

This d did not fit into either of the shebang combinations we were expecting, so our first thought was to bypass the d by sending a backspace character. This did not work, as the backspace character did not properly register as a backspace until after it was sent to the challenge server to be run, rendering the commands broken. To make this even more difficult, the challenge would not return any error messages after we sent in our inputs, leading us to do more local testing. 

Next, we tried to send a .. command, that would pop out a shell of directories, allowing us to change directories into bin and then bash. This method did not work either, as the source code had a function that automatically exited the program if a '.' character was detected in the input. We tried to trick the system by perhaps sending in hex values for the periods and then using a python command to decode the hex into ASCII, but that didn't work either due to the 16 byte limit imposed on the input. 

After doing some more digging through the source code, we realized that the problem had to do with the file descriptors. The code opened two files, one containing the bin/bash that we needed, and one containing the flag text that we needed. Since the file descriptors 0, 1, and 2 are reserved for input, output, and error respectively, we knew that 3 would correspond to the first file opened, the bash shebang we need, and 4 would correspond to the flag file. We then noticed that later in the source code, the file descriptor for the flag file was copied into file descriptor 9, so we took note of that. 

The golden point of realization was that the flag descriptors were located in dev, which we could access with the d in the given script. For this, we sent the rest of dev and a newline character along with cat to try to open the flag file. Unfortunately, the problem we ran into was that we were unable to cat the file, since once the bash was enabled, privileges for the remote user were set to the lowest setting, disallowing us from reading the file. The workaround was to map to the file directory 9. The <& points the file descriptor to a file name, allowing us to cat the flag file and find the flag.

python -c "print('ev/fd/3\ncat <&9')" | ncat --ssl 7b000000ad865f5f197fef9f.challenges.broker2.allesctf.net 1337

