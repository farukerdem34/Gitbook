# Vector (Hardest)

Vector (Hardest)

Wednesday, 23 March 2022

7:59 pm

These ports were open:

!\[\[100\_Vector (Hardest)\_image001.png]]

&#x20;

!\[\[100\_Vector (Hardest)\_image002.png]]

&#x20;

So on the HTTP server of 2290, there was this interesting part.

!\[\[100\_Vector (Hardest)\_image003.png]]

Hmm.

&#x20;

Within the pace source, there was this interesting bit.

!\[\[100\_Vector (Hardest)\_image004.png]]

C probably stood for cipher, let's key this thing in.

!\[\[100\_Vector (Hardest)\_image005.png]]

&#x20;

Interesting, it seems that all this gives me is a 1.

&#x20;

!\[\[100\_Vector (Hardest)\_image006.png]]

&#x20;

Appending anything results in 0 being given. Perhaps this was a hash I could pass the hash authenticate with.

The user was called victor too.

&#x20;

This was interesting indeed. So this website prints stuff based on the encryption.

There was some brute forcing needed to be done to this thing, which serves as a verification of whether or not I have identified successfully.

&#x20;

So in this case, we basically have an oracle of sorts, telling us what we are doing.

We have an encryption oracle of sorts, and we can perform the Padding Oracle attack.

&#x20;

[https://github.com/mpgn/Padding-oracle-attack](https://github.com/mpgn/Padding-oracle-attack)

&#x20;

This is a good repo to use.

&#x20;

{width="10.041666666666666in" height="1.0208333333333333in"}

&#x20;

Then we just wait around while it does its thing.

&#x20;

After a while, it finds the password.

!\[\[100\_Vector (Hardest)\_image008.png]]

&#x20;

Then we can RDP in as the user.

{width="6.78125in" height="0.8229166666666666in"}

&#x20;

Flag:

!\[\[100\_Vector (Hardest)\_image0010.png]]

&#x20;

We can transfer to a netcat shell and then run winPEAS.

&#x20;

PE:

!\[\[100\_Vector (Hardest)\_image0011.png]]

&#x20;

We have a password and a hash.

I don't think I need to show what needs to be done next.

&#x20;

Pwned.

&#x20;
