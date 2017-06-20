General Architecture
====================
In this chapter, we will introduce some general concepts of security architecture.
Actually , these concepts are not absolutely isolated , but some subtle interactions with each other.

Fuse Control
------------
Fuse is kind of one time programmable resource that can be used to control many aspects of our chips.
Fuses are programmed through **OCOTP**. Generally speaking, they can make effects in two ways.

* **HW FLOW** Direct hardware connections between OCOTP and specific targets. (This method is almost the most popular in our i.MX series chips)

* **SW FLOW** Indirect connections through special GPRs. Specific core takes charge of reading out the fuses' values and writes to special GPRs, of which the output will be routed to the targets. (This is mainly applied in i.MX8 projects since we have **SCU Subsystem** as system control unit).

Even though fuses are used in many aspects of our chip, we are only going into some of them since our topic is about security.
You can refer to the block guide of **OCOTP** and the **FuseMap** for more information.

See `FuseMap Of i.MXQXP`_

.. _FuseMap Of i.MXQXP: https://nxp1.sharepoint.com/sites/MCUlibrary/iMX8QXP/05_Systems/fuse map/i MX8QXP_Fuse_Map_v1.0_test.xlsx?d=w1261529ba8874a63b60716131f78cf5a


Resource Limitation
~~~~~~~~~~~~~~~~~~~

Description
"""""""""""
With different customer targets , we will have different resource limitation resolutions. For example , we may have some IPs like PCIe not usable, or have 1,2 or 3 cores of 4 in one core platform disabled. Besides, performance limitation(set the maximum frequency of CPU, GPU, etc.) is often achieved by using fuses, too.

Verification Strategy
"""""""""""""""""""""

These are typical scenarios that cover chip level connections . The start point of the path is **OCOTP** and the end point depends on the target. They may belong to different **MIX Domains** or **Subsystems**, so there will exists some top level path. Thus we have several strategies to cover such features.

* If the finial effect is easy to check, we can cover the full path in our side either in SW flow or by monitoring target with assertions or anything else. For example, to check PCIe can be disabled, we can :
  
  + 1. Program relevant fuse bit through **OCOTP** using our **Core**

  + 2. Check the target. If we can do that easily with SW method, like trying to access the target and expect some exceptions, it will be the very best solution.  If not,  we can directly check into the final control logic in HW way. Take PCIe in **UTL1** for example, we disable it by gating its clock, and of course, enable input of gate logic is driven by our fuse bit. In this situation,  trying to access target will never return a response since the clock is gated, and the **Core** will hang for that access. That's not what we want during simulation. Well, we can have **WDOG** to reset the **Core** once get hang, but this will increase complexity of the case and load for case writer. So we can monitor crucial clock instead, or monitor the gate enable signal if gate logic can be trusted. 

* Sometimes the effect is not that visible from our side, we will take that as a black box and our duty is to make sure the control signals are driven correctly to the boundary of target domain. In this situation, the end side verification engineer must guarantee the function of those input boundary signals either with stand along **DV** or in their chip level **VECTORs**.  It is very important to **Make Clear about Both Sides' Duites** since bugs are often hidden when both sides over-depend on the other side. We should keep in mind: **Overlap Is Better Than Gap**
  
* For some fuse control bits in **SW FLOW** especially those would be routed to other subsystems besides of **SCU** itself, we may rely on those subsystem verification owners to cover the paths from **GPRs** to destinations. In this situation, we **MUST** make sure both sides' awareness and agreement of such division. 

Learn a Lesson
""""""""""""""
In this article, we are going to talk about a bug that are not discovered by our verification work in i.MX8QM. 

In this project, our control fuse bits are not directly connected to the target. **SW FLOW** is involved to enable resource control. The expectancy is that **SCU** read out those fuse bits, and write to those **GPRs** distributed in different subsystems , and each subsystem integration owner should make sure those **GPRs** are connected correctly to their specific targets according to **ADDs**. 

But verification cases only cover the flow from **OCOTP** to **GPRs**, and the paths from **GPRs** to destinations are deemed to be covered by those subsystem verification owners. But those owners think the owner for fuse would cover all these. 

Well, Unfortunately, for some reason, those outputs of **GPRs** are not connected to the right targets in each subsystem, and verification work fail to dig it out.

This kind of bug is very harmful since it's very difficult to correct by **ECO** when the flow is pushed to the **Back End** already since so many subsystems are affected. 
So, for such situations, communication are very important for verification work to reduce such masked bugs.

