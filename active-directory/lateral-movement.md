# Lateral Movement

In Active Directory, becoming an administrator on one machine doesn't mean much. If we are simply a Domain User and we are admin on `WKSTN` for example, we still haven't 'pwned' it. The goal here is to escalate privileges until we reach the DC and become a Domain / Enterprise Admin (depending on what is needed) before pwning the domain.

For example, in HTB machines there could be multiple machines in one box, requiring us to move all over the place before becoming a Domain Admin and capturing the root flag.

In order to do this, we need to understand how Windows Authentication and how User Impersonation can be abused.&#x20;

WIP!&#x20;
