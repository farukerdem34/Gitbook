# Graph

Graph

Sunday, June 26, 2022

12:54 AM

Nmap scan:

!\[\[58\_Graph\_image001.png]]

&#x20;

!\[\[58\_Graph\_image002.png]]

&#x20;

Take note of node.js framework.

&#x20;

Port 80:

!\[\[58\_Graph\_image003.png]]

&#x20;

Let's take a look at the page source, because there is nothing notable about this page.

&#x20;

Right away, we can notice something like this:

!\[\[58\_Graph\_image004.png]]

&#x20;

There seems to be some form of graphing for the elements going on, combined with the fact that the machine is called Graph, gives me an indication that there could be some form of node.js graph module being used here.

&#x20;

There are few things we can do from here, which are:

* Research more about the graph concept of this page, because the box does not give us much to work with
* Experiment with HTTP Smuggling or something
* Attempting to SSRF to find some unique hidden page elsewhere.
* Gobusting the page.

&#x20;

Gobuster:

!\[\[58\_Graph\_image005.png]]

&#x20;

!\[\[58\_Graph\_image006.png]]

&#x20;

We can gobuster /static directory to find nothing much.

&#x20;

Wfuzz:

!\[\[58\_Graph\_image007.png]]

&#x20;

Tried this method with multiple wordlists.

When using the graphql.txt, we get some hits.

!\[\[58\_Graph\_image008.png]]

&#x20;

!\[\[58\_Graph\_image009.png]]

&#x20;

We can notice that this kind of stuff stands out.

!\[\[58\_Graph\_image0010.png]]

&#x20;

And we are right, because it does ask us for a query.

&#x20;

Querying GraphQL:

!\[\[58\_Graph\_image0011.png]]

&#x20;

!\[\[58\_Graph\_image0012.png]]

&#x20;

We can enumerate this database:

We can see that the name query takes an argument of users or something:

!\[\[58\_Graph\_image0013.png]]

&#x20;

So it takes a string basically.

We can test this.

!\[\[58\_Graph\_image0014.png]]

&#x20;

After a bit of testing, we can see this:

!\[\[58\_Graph\_image0015.png]]

&#x20;

Adding another quote results in an error:

!\[\[58\_Graph\_image0016.png]]

&#x20;

After more fucking around, we can find an SQL Error:

!\[\[58\_Graph\_image0017.png]]

&#x20;

This is done by appending ' at the end of the searchTerm, which is 1 in this case. Then, we can fuzz some more.

Successful injection:

!\[\[58\_Graph\_image0018.png]]

&#x20;

We have successfully appended a query onto this.

If we change this to OR, we find out the database!

&#x20;

!\[\[58\_Graph\_image0019.png]]

&#x20;

So we have 3 users here, and let's see if we can do something funny with this by being able to dump out the credentials. We now know there are 3 columns perhaps.

&#x20;

So let's try to enumerate this database.

!\[\[58\_Graph\_image0020.png]]

&#x20;

We can also figure out there is no database() function.

!\[\[58\_Graph\_image0021.png]]

&#x20;

We can try to SQLMap this to gain some form of exploitation easily.

!\[\[58\_Graph\_image0022.png]]

&#x20;

We can use the wildcard parameter to specify where to begin injection.

!\[\[58\_Graph\_image0023.png]]

&#x20;

And it appears we have a UNION injection exploit here.

We can proceed to enumerate this database, or use SQLMap to gain some passwords and stuff.

&#x20;

Manual Method:

!\[\[58\_Graph\_image0024.png]]

&#x20;

We can see that there is this users table.

Then we can enumerate the columns:

!\[\[58\_Graph\_image0025.png]]

&#x20;

So from here, it appears we have identified the database to use and we need to now enumerate this table called users.

I guessed a few parameters.

!\[\[58\_Graph\_image0026.png]]

&#x20;

We have username.

&#x20;

Then we have something better, and it's the hashes.

!\[\[58\_Graph\_image0027.png]]

&#x20;

We can john all of these.

!\[\[58\_Graph\_image0028.png]]

&#x20;

We have a crack!

&#x20;

SSH Shell:

!\[\[58\_Graph\_image0029.png]]

&#x20;

!\[\[58\_Graph\_image0030.png]]

&#x20;

PE:

{width="9.885416666666666in" height="1.3333333333333333in"}

&#x20;

It seems we have access to this binary.

!\[\[58\_Graph\_image0032.png]]

&#x20;

This seems to update some password. From there, it seems we are able to change the password of Jane.

Interesting. Perhaps this could be exploited further...

&#x20;

I ran LinPeas anyway to double check on whether there is a cronjob being manipulated for this binary.

&#x20;

We can see that josh is part of the shadow group.

!\[\[58\_Graph\_image0033.png]]

&#x20;

Other directories:

!\[\[58\_Graph\_image0034.png]]

&#x20;

Jane has this .py file, which is the basically the same script that is used in the pass-gen file.

Code Analysis:

&#x20;

| <p>#! /usr/bin/env python2</p><p> </p><p><strong>import</strong> sys</p><p><strong>import</strong> os</p><p><strong>import</strong> subprocess</p><p><strong>import</strong> crypt</p><p><strong>import</strong> string</p><p><strong>from</strong> random <strong>import</strong> randint<strong>,</strong> randrange<strong>,</strong> SystemRandom</p><p> </p><p>"""</p><p>generate hash</p><p>"""</p><p><strong>def</strong> gen_passwd<strong>(</strong>password<strong>):</strong></p><p><strong>return</strong> crypt<strong>.</strong>crypt<strong>(</strong>password<strong>,</strong> '$6$' <strong>+</strong> salt<strong>)</strong></p><p> </p><p>"""</p><p>update matching user entry</p><p>"""</p><p><strong>def</strong> check_entry<strong>(</strong>line<strong>):</strong></p><p>arr <strong>=</strong> line<strong>.</strong>split<strong>(</strong>":"<strong>)</strong></p><p><strong>if</strong> arr<strong>[0] ==</strong> user<strong>:</strong></p><p>arr<strong>[1] =</strong> gen_passwd<strong>(</strong>password<strong>)</strong></p><p>arr<strong>[4] = str(</strong>valid_days<strong>)</strong></p><p><strong>return</strong> ":"<strong>.</strong>join<strong>(</strong>arr<strong>)</strong></p><p> </p><p>"""</p><p>return password for user</p><p>"""</p><p><strong>def</strong> rand_passwd<strong>():</strong></p><p>chars <strong>=</strong> string<strong>.</strong>ascii_letters <strong>+</strong> string<strong>.</strong>digits <strong>+</strong> '!@#$%^&#x26;*()'</p><p>rnd <strong>=</strong> SystemRandom<strong>()</strong></p><p><strong>return</strong> ''<strong>.</strong>join<strong>(</strong>rnd<strong>.</strong>choice<strong>(</strong>chars<strong>) for</strong> i <strong>in range(15))</strong></p><p> </p><p># main def</p><p><strong>if</strong> __name__ <strong>==</strong> '__main__'<strong>:</strong></p><p> </p><p><strong>try:</strong></p><p># randomly generate password + salt</p><p>password <strong>=</strong> rand_passwd<strong>()</strong></p><p>salt <strong>= str(</strong>randrange<strong>(1000, 10000) *</strong> randrange<strong>(1000, 10000))</strong></p><p> </p><p>valid_days <strong>= 7</strong> # password is valid for max 1 week, but give user option to decide the exact duration</p><p><strong>if len(</strong>sys<strong>.</strong>argv<strong>) > 1:</strong></p><p>valid_days <strong>=</strong> sys<strong>.</strong>argv<strong>[1]</strong></p><p> </p><p># get actual user</p><p>user <strong>=</strong> subprocess<strong>.</strong>check_output<strong>(</strong>"who am i | awk '{print $1}'"<strong>,</strong> shell<strong>=True).</strong>strip<strong>()</strong></p><p> </p><p># overwrite shadow</p><p>new_content <strong>=</strong> ""</p><p>shadow_file <strong>=</strong> "/etc/shadow"</p><p> </p><p>with <strong>open(</strong>shadow_file<strong>,</strong> "r"<strong>) as</strong> fd<strong>:</strong></p><p>lines <strong>=</strong> fd<strong>.</strong>readlines<strong>()</strong></p><p> </p><p><strong>for</strong> line <strong>in</strong> lines<strong>:</strong></p><p>parsed <strong>=</strong> check_entry<strong>(</strong>line<strong>)</strong></p><p>new_content <strong>+=</strong> parsed</p><p> </p><p>fd <strong>= open(</strong>shadow_file<strong>,</strong> "w"<strong>)</strong></p><p>fd<strong>.</strong>write<strong>(</strong>new_content<strong>)</strong></p><p> </p><p><strong>print</strong> "%s: password updated successfully to %s " <strong>% (</strong>os<strong>.</strong>path<strong>.</strong>basename<strong>(</strong>sys<strong>.</strong>argv<strong>[0]),</strong> password<strong>)</strong></p><p><strong>except:</strong></p><p><strong>print</strong> "%s: error" <strong>%</strong> os<strong>.</strong>path<strong>.</strong>basename<strong>(</strong>sys<strong>.</strong>argv<strong>[0])</strong></p><p> </p><p>We can see that this file happens to take one system argument, which is the number of weeks this password is valid for. Afterwards, it checks on who is the user executing this, which should be root due to how sudo works.</p><p> </p><p> </p><p>So the root user is running this, which seems to run as jane here, which is weird because if root runs this command, it should theoretically rewrite the password of the root user.</p><p> </p><p> </p><p>The system takes in one argument, which is the number of days this password is valid for. Which is strange.</p><p>Since the input is directly put into the /etc/shadow file, perhaps we can add another user to this mess.</p><p> </p><p>We can test this on our local machine.</p><p> </p><p>I changed the script to just write to some random test file, and we can see that our text made it in.</p><p>The goal here is to either access the josh user through changing his password and then read the /etc/shadow group since he's part of the shadow group and then crack root's hash, or we change root's hash directly.</p><p> </p><p>By doing this, we can do something like this:</p><p> </p><p> </p><p>From here, it's simple, we just need to add a new entry to the /etc/shadow file that is a new root user or something.</p><p>I tried this out:</p><p> </p><p> </p><p> </p><p>Cool, we can append new entries to this, and in this case, I tested with josh first to see, and it worked.</p><p> </p><p> </p><p>Now, this only works once, because in the /etc/shadow file, the 'jane' user is likely one line higher than josh, and when using this to change passwords, we can</p><p>From here, we can grab the root hash.</p><p> </p><p> </p><p>Cracking:</p><p> </p><p>We can now SSH in as root and grab the flag.</p><p> </p><p>Flag:</p><p> </p> |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
