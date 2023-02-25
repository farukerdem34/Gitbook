# Node

Node

Thursday, 24 February 2022

4:36 pm

Did this box at work. So not a nice writeup.

&#x20;

Anyways, I determined that there's an API making calls for the web application, which through looking and fuzzing, we can get passwords.

&#x20;

Log in to the page and from there we get this backup

&#x20;

Decrypt using base64 and find out it's a zip file.

&#x20;

Zip2John and brute force it.

&#x20;

Afterwards, retrieve Password and Username 'mark' from low user.

&#x20;

SSH in.

&#x20;

Afterwards, we need to manipulate mongodb that provides an RCE as high user.

&#x20;

Log in to mongodb with the same credentials and then inject commands into the code.

Gain a reverse shell

&#x20;

Afterwards, this part I looked up a guide.

&#x20;

We had to somehow determine the possibility of a script that we can execute.

&#x20;

This part was a BOF, which I have yet to look through the solution.

&#x20;

Basically it takes 3 inputs, and it checks on certain keys that we can take from the directory.

From here, it's a libc kind of thing, where we have to find the directory of which system is stored in memory, and then find our offset and then include the return address which would execute the shell code to give us root.

&#x20;
