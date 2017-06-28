Vectors
=======
There are several vectors in security test plan, and all C source files are maintained in our VIP. We will make a brief introduction about them.


Aboout Test Plan
----------------

There are two test plans for security subsystem.
One for **PA** [#PA]_ ::
   
   $DESIGN_DIR/testbench/blocks/soc_tb/tool_data/razor/sanity/iMX8QX_Security_PM_Verification_Plan.xlsx

One for normal::
   
   $DESIGN_DIR/testbench/blocks/soc_tb/tool_data/razor/iMX8QX_SS_SECURITY_Verification_Plan.xlsx

Basically these two test plans originate from i.MXQM and with some updates for design change. Vectors and cases listed in the plan are exactly what we are focusing on, regardless of what else we have under the server directory .

Life Cycle
----------
Short description quoted from Security Subsystem ADD : 
"*The Chip Life Cycle applies to all cores and to both the silicon vendor (NXP) and OEM stakeholders. Secrets may be installed onto a device at different stages of the chip life cycle including from when the chip is first fused and when firmware and secure boot processes are enforced. Because of the possibility of failure, the chip may need to be returned to the OEM and/or to the silicon manufacturer. The Chip Life Cycle controls help ensure that secrets provisioned in one stage are not accessible to another as the chip progresses to the field and also as the chip returns back to the OEM and to the silicon manufacturer.*" See details in section 5.3.6.17 of ADD

Test cases in this vector are designed to cover transitions between life cycle states and some effects like access restrictions in different life cycle states.

Remember, life cycle is tightly coupled with ADM and SNVS. In fact, it's part of ADM, so some corners are going to be covered in other vectors . 

Since there is no key changes for life cycle, all test cases are copied from QM, and almost no modification. 


scu_sec_caam
------------
CAAM is one of the **Big 3** IPs under security subsystem, another two are SNVS and ADM. From function point of view, these three IPs are coupled together tightly. You can get some necessary info from security subsystem ADD and block guide of CAAM . In general , it's , quoted from section 5.2.2 of SEC ADD ,"*CAAM is to support a set of industry standard hardware accelerators, boot time acceleration of the hashing function, crypto key protection, HDCP 2.X authentication and protected video path (from software) support,manufacturing protection and public key cryptographic acceleration, and interoperate with TrustZone, Resource Domain and system virtualization access controls*"

In A0 version of i.MXQXP, there is no significant design change of CAAM compared with i.MXQM, so few updates are needed for test cases. But since, as we say, it's not a isolated IP and it interacts with other parts of subsystem very often , a lot of issues are revealed from this vector. 

From chip level verification point of view, you don't need to get very familiar with CAAM's inner details, well, it would be nice to have, but not necessary since we are focusing on the port connections around CAAM and are not going to check all the features itself in this vector, that should be done through standalone DV [#DV]_. 

Besides of general verification targets like register access, clock control, LPM [#LPM]_ , etc., the key responsibility of this vector is to cover the aspect of security control.

scu_sec_int
-----------
This is about several exceptions from M0+ inside security subsystem. All reused from i.MXQM .

scu_sec_m4_acc
--------------

Access resources of security subsystem from M4 inside SCU subsystem. All C source files are updated due to memory map change and increase of IPs. Please notice do not override them with i.MXQM versions.

scu_sec_mmcau
-------------
Description about mmcau: quote, "*The Memory-Mapped Cryptographic Acceleration Unit (mmCAU) is a coprocessor that is connected to the processor's private peripheral bus (PPB). It supports the hardware implementation of a set of specialized operations to improve the throughput of software-based security encryption/decryption operations and message digest functions.*"

This is a new vector in i.MXQXP, as this mmcau is added in this project to SECO, test cases follow the vector in SCU subsystem, but since these are for different cores, some workaround are needed for specific case. This will be introduced later . 

Generally speaking, design changes always come with bugs, it's no exception here. You can refer to tickets **TKT314206** and **TKT317747** for detailed info.


scu_sec_mu & sec_mu
-------------------
Not sure why two vectors in QM, just reused.

scu_sec_romcp
-------------
ROMCP is short for *ROM Controller with Patch*, this IP is used almost in every i.MX series products for ROM access and patching.You can find detailed info in RM in any i.MX chips.

This vector is following i.MXQM except that the size of ROM for SECO increased to 80K from 64K, so updates are needed. Also, bug is found for this design change, you can check in designPDM for details. 

scu_sec_snvs
------------

Another **Big 3** member. SNVS plays quite an important role inside Security Architecture of i.MX series chips. Generally speaking, it's used to detect attacks and potential threaten ,then react inside and output violations to system. You can see details in section 5.3.2 of SEC ADD and it's block guide. 

An important design change is about it's tampers. In i.MXQXP, we have 10 external tampers and 5 active tampers, while in i.MXQM it's 4 and 2. Besides of the numbers, IOMUX also changes since we don't have enough outer pads for tamper only, so those tampers in i.MXQXP are sharing pads with other IPs like CSU . And we need to communicate with design guys into details to cover such changes. 

Regardless of external/active tampers, other three analog tampers need more attention, which are for voltage/temperature/clock attack detection . The whole path should be covered, that is from the true violation detected to the reaction inside SNVS. We have learned a lesson in i.MXUL , in which project, a big issue with clock tamper is ignored and results in re-tapout of the chip. 

There is another IP VIP for SNVS , where some resources are maintained, you will need to look into it when meeting some problems::
   
   $DESIGN_DIR/testbench/common_blocks/v_ms_snvs_vip

scu_sec_wdog
------------

Following scu_wdog , but some **low power mode** relevant cases are removed. We can not be sure whether it's a deliberate design change or just a **Leave it being** bug. We do have some emails talking about this, and there is no problem with Architecture guys. 

sec_adm
-------

To be honest, I should put this vector at the beginning ,due to its significant priority. As a member of **Big 3**, lots of attention is payed to ADM for three reasons:

+ It's a new IP and still in developing.
+ It's untested , as far as we know, there is no standalone DV ENV for ADM right now. I think it's due to aggressive schedule.
+ It plays a significant role in security subsystem. It's akin to a center control unit that affects almost every aspect of security subsystem .
  
Below description is quoted from section 5.2.4 in SEC ADD : "*The ADM is a module that works in conjunction with the debug system and fuse configuration to provide security measures. It receives control signals from various sources such as pins, OTP fuses, and the SCU, DAP, and JTAG at boot and during run time to determine, restrict and indicate use of the debugging components. Certain debugging features are allowed or disallowed based on NXP and OEM requirements*" . 

In fact, it's not just about debug components, as I mentioned before, it also affects many aspect of security subsystem through its GPRs. Whatever, you will need to put it under high priority.

sec_jtagc
---------

Covering some corners of interacts of Jtagc and ADM , copied from i.MXQM. More corners are covered by **debug** vector.

sec_mscm
--------

This vector is used to cover **ECC** function of SECO RAM, including single bit error and multi bits error. Both these two types of ECC errors are generated using driver ram_ecc_corrupt. 

sec_ocotp
---------

Covering basic OCOTP program function and CRC test. For usage of all fuses, they are distributed in different vectors under security subsystem.
Please notice there is one case that is still not passing: **ocotp_ctrl_ded_irq_16k** , but this is already covered by IP standalone DV and should be ok.

sec_seco_access
---------------

Only one case exists. It's used to check those reserved address spaces from SECO point of view. Should be updated due to address space change in i.MXQXP.



.. rubric:: Footnotes

.. [#PA]  Power Aware, CPF involved .
.. [#DV]  Short for design verification .
.. [#LPM] Low Power Mode .
