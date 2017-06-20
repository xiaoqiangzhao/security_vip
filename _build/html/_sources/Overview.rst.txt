Overview
========

VIP is quite an important concept in i.MX8 series projects' DV TB, precisely speaking, in i.MXQM and i.MXQXP . We intend to improve re-usability through different projects by encapsulate some relevant testbench resources according to either **IP** or **Subsystem** . 

We are going to talk about our security subsystem VIP here , of course, in a most precise way as we can. You can also take this article as an typical entry to snoop at general structure of VIPs in our project if you are not that familiar with them.

Directory Structure 
-------------------
Below is the directory of our VIP. Please notice that it may differ with some other VIP in details, like we don't have **forces_v** under **testbench**. And of course, VIP owner can add if needed.

.. literalinclude:: dir_tree

