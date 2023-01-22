# Delivery

## Gaining Access

Nmap scan:

<figure><img src="../../../.gitbook/assets/image (24) (1).png" alt=""><figcaption></figcaption></figure>

### Port 80

The website itself was rather uninteresting. However, when viewing the Contact Us portion, we can find a domain and mention of a helpdesk and a MatterMost server:

<figure><img src="../../../.gitbook/assets/image (5) (11).png" alt=""><figcaption></figcaption></figure>

### Port 8065

This was the port where the MatterMost instance was hosted, and credentials are needed for this.

<figure><img src="../../../.gitbook/assets/image (9) (4).png" alt=""><figcaption></figcaption></figure>

Based on the hints given in the Contact Us page, I registed an email with the @delivery.htb domain at the back. However, this was not possible because we had to verify the email, and we don't have any email clients on this machine.

<figure><img src="../../../.gitbook/assets/image (20) (7).png" alt=""><figcaption></figcaption></figure>

### Helpdesk

`helpdesk.delivery.htb` was a subdomain present on port 80. Visiting it reveals an OSTicket instance.

<figure><img src="../../../.gitbook/assets/image (16) (1).png" alt=""><figcaption></figcaption></figure>

I submitted a test ticket and received this:

<figure><img src="../../../.gitbook/assets/image (30) (1) (1).png" alt=""><figcaption></figcaption></figure>

Also, trying to create an account on this was not possible because we had to verify our email address, which was not possible without an internal client.&#x20;

### Email Client

All of our deadends so far were the result of not having an email client to view the confirmation emails.&#x20;

That's when I realised the email given to add more information can be used as a 'client'. This is because we can check the status of tickets without needing an account or verification on OSTicket via the `Check Ticket Status` tab. So, I registered `2431755@delivery.htb` on the MatterMost server and went to view my ticket again.&#x20;

This time, there was an email confirmation there.

<figure><img src="../../../.gitbook/assets/image (38) (3).png" alt=""><figcaption></figcaption></figure>

Then we can login and view the forums to find an SSH credential:

<figure><img src="../../../.gitbook/assets/image (35) (3).png" alt=""><figcaption></figcaption></figure>

We can then use these credentials to SSH in as the user.

<figure><img src="../../../.gitbook/assets/image (34) (1).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### MySQL Credentials

When looking at the MatterMost files in the `/opt` directory, we can find some SQL credentials:

<figure><img src="../../../.gitbook/assets/image (125).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (13) (3).png" alt=""><figcaption></figcaption></figure>

Then, we can use `netstat` to find that there is a MySQL instance running on the machine.

<figure><img src="../../../.gitbook/assets/image (122).png" alt=""><figcaption></figcaption></figure>

We can then access the `mattermost` database via the `mysql` command.

<figure><img src="../../../.gitbook/assets/image (8) (1) (6).png" alt=""><figcaption></figcaption></figure>

Then, we can read the Users table data to find the hash for the root user.

<figure><img src="../../../.gitbook/assets/image (130) (2).png" alt=""><figcaption></figcaption></figure>

The hash can be cracked via `hashcat -m 3200` to give `Password!21` as the password. We can then `su` to root.

<figure><img src="../../../.gitbook/assets/image (14) (6).png" alt=""><figcaption></figcaption></figure>
