# Surf

Surf

Friday, 15 April 2022

5:56 am

1. Bypass login with some cookie replacements using base64 and simple JSON manipulation
2. Determine that the web application is running phpfusion, which is vulnerable to a certain RCE exploit.
3. We can use that exploit and combine it with the website that is known to have SSRF, which can be used to reach the internal server.
4. From there, we can have foothold as www-data.
5. There are credentials for the user 'james' inside one of the config folders, and we can su.
6. James can one php script, which is owned by www-data.
7. Echo in some PHP code that would execute bin/bash upon being executed.
8. Go back to james and sudo exploit this. We are now root.

> &#x20;
>
> &#x20;
