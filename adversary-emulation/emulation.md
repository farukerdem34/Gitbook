# Emulation

## Threat-Informed Defence

This is a strategy used to advance global understanding of adversary tradecraft, measure evolving adversary behaviour via playbooks or software, enable continuous assessment of defence and to continuously idenfiy and catalyse development and/or research new ways to thwart ATT\&CK techniques.

> Threat-Informed Defence applies a deep understanding of adversary tradecraft and technology to protect against, detect and mitigate cyber-attacks. It's a community-based approach to a worldwide challenge.
>
> \-MITRE

In layman terms, we are using existing adversary tactics, techniques and procedures (TTPs) from the MITRE ATT\&CK Framework to **emulate** attacks on corporate infrastructure. The definition of **emulation** in this case is to replicate the effects or behaviour of a given technique by executing the actual process which produces them.&#x20;

Commonly, a threat emulation exercise would involve some kind of **cyber range**, which is a simulated corporate network that may or may not mirror actual production infrastructure, then running attacks on it using adversary emulation tools like Caldera (most are close-source).

{% embed url="https://github.com/mitre/caldera" %}

Caldera comes with its own C2 server so it can have Beacons, as well as plugins emulating current APTs like OilRig.

## Red Team vs Pentesting

The red team is meant to **emulate adversary attacks.** They provide an adversarial perspective by challenging the assumptions made by an organisation and defenders. By challenging these assumptions, a red team can identify vulnerabilities in an organisation's OPSEC.&#x20;

There is some degree of overlap with penetration testing, but they have fundamentally different objectives. A typical pentest would focus on one technology stack within a company's entire infrastructure, such as their website. The goal is to **identify as many vulnerabilities as possible, demonstrate how those may be exploited, and provide some risk ratings**. The output is a report detailing each vulnerability and needed remediation actions, such as installing a patch or configuring software. In a pentest, **there is no focus on detection or response**, and the only goal is to hack the system.

On the other hand, the red team has a clear objective laid out, which is to gain access to a particular system (such as the backend database) instead of finding all the bugs within the infrastructure. They also have to emulate a **real-life threat** to the organistaion (For example, banks may be attacked by FIN APT groups). So for this case, we need to use specific TTPs based on the adversary we are trying to emulate.&#x20;

A penetration test is not focused on stealth, evasion or the ability for the blue team to detect and respond. Rather, the blue team might be aware that the penetration test is even happening, and let it happen because they are here to find out all the vulnerabilities and weak endpoints, so very noisy scans such as `autorecon` and `nmap` can be used for this.&#x20;

A red team operation is **very focused on stealth and evasion**. As the goal is to emulate a real-life threat, OPSEC and not getting caught is the most important aspect of a red team operation. The blue team is actively trying to detect the red team operations and block them instead of letting it happen.&#x20;

The tools used are different as well. Penetration testers might have more scanners and other tools which focus more on testing individual systems within the scope. A red team would use tools such as Cobalt Strike to again, simulate a real APT.&#x20;
