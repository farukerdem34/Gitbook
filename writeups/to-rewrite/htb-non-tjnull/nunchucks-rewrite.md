# Nunchucks

Nunchucks

Saturday, 26 March 2022

2:01 pm

This was an interesting box. There was a hidden Vhost on one of the web ports, and from there, it was an email input box that would print on screen.

&#x20;

When the response was intercepted, it was determined that there was actually an SSTI vulnerability that would have allowed for some reverse shelling.

&#x20;

From there, once we had the SSTI figured out, it was using Nunchuks engine, and we can get an RCE from there through calling a shell.

&#x20;

Once we get a reverse shell in, we discover that perl has the SetUID parameter on, meaning we can create a script that we execute allowing for perl to execute and get us the root shell.

&#x20;

All in all, cool box. Took about 20 minutes to root.
