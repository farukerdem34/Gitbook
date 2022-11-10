# Bug Review

Bug Review

Thursday, July 14, 2022

12:45 PM

The very first bug is this line here:

{width="6.3125in" height="0.9791666666666666in"}

&#x20;

Basically, token is something that is user controlled, and of course insecure.

If we were to put token as an empty array as specified by the website, the function for $secret would just return false. Which is great.

&#x20;

The second bug lies here.

!\[\[65\_Bug Review\_image002.png]]

&#x20;

We can see that strict comparison is used, so nothing to do with exploiting magic hashes. However, since we can control the $secret value, making it false, we can basically control the $hm value, and as such always be able to bypass this extra security check to reach the rest of this function.

&#x20;

All we need to do is supply any random host name that we control, then send the thing through hash\_hmac to find a valid h value. Simple enough.

&#x20;

However, after we visit this, we

&#x20;
