
PreWords
========
This doc is written by a verification engineer per his limited understanding of security architecture in i.MX series **Especially For i.MX8 Series** products of Freescale Corporation and is aiming to help those who are not very familiar that. 
We can not guarantee that every aspects are mentioned or detailed, and any suggestion or correctness is welcome.

About security subsystem
------------------------
Brief introduction about security subsystem in i.MX series projects

Security SS in i.MX6 & i.MX7 projects
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Security related features are quite an important component in i.MX series chips . In some old projects like i.MX 6 series or i.MX7 ult1 , security subsystem is actually  not a subsystem , since we don't have any subsystems there from architecture point of view, but some domains divided as MIX, like **MEGAMIX** , **FASTMIX** , **WAKEUPMIX** .
So when we talk about security features in those projects, we are actually referring to those security related IPs distributed in different MIX which act as specific roles or affect the whole chip. Even though sometimes the bounds are not that clear, I'm still trying to distinguish them in this reference doc  according to their sphere of influence.


Security SS in i.MX8 Series
~~~~~~~~~~~~~~~~~~~~~~~~~~~

In i.MX8 series projects, we finally have a real security subsystem , which is embedded in **SCU Subsystem** and has almost all related IPs included or referred to so that we can have a more clear overview of the security features of our chip. 

Unfortunately, we don't have a separate ADD for **i.MXQXP** , well, not yet at least. But we can still refer to the one of **i.MXQM** for now. 

See `SECURITY SUBSYSTEM ADD OF QM`_.

.. _SECURITY SUBSYSTEM ADD OF QM: http://compass.freescale.net/livelink/livelink/235791998/Security_Subsystem_QM_V0_1_17.docx?func=doc.Fetch&nodeid=235791998
