
Resources
=================
We will look into detailed useful resources provided by the VIP in this article .


Fuse Load
---------
Backdoor initializing fuse need predefined fuse values matched with fuse map of our project. The fuse values can be specified through args . Let's take a look at this.

Related Files:

* C work

  + $(DESIGN_DIR)/testbench/blocks/soc_tb/tool_data/gnu/cm4/SCU _cm4.lnk

  + $(DESIGN_DIR)/testbench/imx_subsys_vip/v_sc_imx_security_vip/tool_data/compiler/include/fuse_map.h

* V work

  + $(DESIGN_DIR)/testbench/imx_subsys_vip/v_sc_imx_security_vip/testbench/modules_v/ocotp_preloader.sv

  + $(DESIGN_DIR)/testbench/blocks/soc_tb/testbench/top_instances_v/memory_interface_inst.sv

Ok, here is the thing, in **fuse_map.h** , we declared an static array of which the size is matched to our design.

::
   
   static const uint32_t fuse_map_array[800] __attribute__ ((section(".efuse"))) __attribute__ ((used)) = { ...};

This array is initialized with values defined with macros, which can be specified in your pattern's arg file as below::
   
   SCU_C_ARG += OTP_MFG_KEY
   SCU_C_ARG += OTP_MFG_KEY_0=0x64234237 
   SCU_C_ARG += OTP_MFG_KEY_1=0x43432321 
   SCU_C_ARG += OTP_MFG_KEY_2=0x57567565
   SCU_C_ARG += OTP_MFG_KEY_3=0xEADA1231
   SCU_C_ARG += OTP_MFG_KEY_4=0xCADE3453
   SCU_C_ARG += OTP_MFG_KEY_5=0xBAAB8898
   SCU_C_ARG += OTP_MFG_KEY_6=0x56562321
   SCU_C_ARG += OTP_MFG_KEY_7=0x7880EADA

Well, as we can see , we are using **SCU_C_ARG** to add macros, in fact, we must use this due to the compiler flow in our top TB. 
Only during **SCU**'s compiling, this file will be picked and compiled , as you may notice that in our declaration of **fuse_map_array**, it's bonded with section **.efuse** . Then in our link script **SCU_cm4.lnk**, it is allocated to specific memory address::

  //memory declare
  efuse_mem           : org = EFUSE_BASE              , len = EFUSE_SIZE
  ...
  ...
  ...
  //allocate section .efuse to memory block efuse_mem declared before
  .efuse               : { KEEP(*(.efuse*));                                       } > efuse_mem    
   
Then after compiling, you will be able to see our predefined fuse values in fixed location of our generated **HEX** file.    

On the other side, we declare and instantiate **ocotp_preloader** in **ocotp_preloader.sv** and **memory_interface_inst.sv**.

At the beginning of our simulation, we will call the build-in task **load_mem** to load the fuse values to our fuse array model through **BACKDOOR**, just like how we do with other normal ROMs and RAMs.  


Monitors
--------
There are several monitors for some key IPs inside the security subsystem.
Please notice,some monitors declared are not instantiated for design changes or just useless anymore.
Most prototype modules are declared under security VIP, and a few are under some common VIPs ::
   
   $DESIGN_DIR/testbench/imx_subsys_vip/v_sc_imx_security_vip/testbench/modules_v
   or
   $DESIGN_DIR/testbench/common_blocks/**IP_VIP**/testbench/modules_v

But their instances may be in security VIP::
   
   $DESIGN_DIR/testbench/imx_subsys_vip/v_sc_imx_security_vip/testbench/instances_v

   
or TOP level ::
   
   $DESIGN_DIR/testbench/blocks/soc_tb/testbench/top_instances_v

+----------------------+--------------+--------------+--------------------------------------------------------------+
| Monitors             | Instance DIR | Modules DIR  | Description                                                  |
+----------------------+--------------+--------------+--------------------------------------------------------------+
|   adm_monitor        |    TOP       | SEC VIP      | Checker [#Checker]_ and Reporter for inner logic of ADM.     |
|                      |              |              | e.g.,Life cycle transition, Jtagc control, etc..             |
+----------------------+--------------+--------------+--------------------------------------------------------------+
|   caam_monitor       |    SEC VIP   | SEC VIP      | Checker for Keys inside CAAM, Please notice,                 |
|                      |              |              | CAAM is a complex IP, detailed logic are supposed            |
|                      |              |              | be covered in standalone ENV                                 |
+----------------------+--------------+--------------+--------------------------------------------------------------+
|   caam_rand_cg_mon   |    TOP       | SEC VIP      | Checker the ADM CAAM random clockgen enable                  |
+----------------------+--------------+--------------+--------------------------------------------------------------+
|   sec_wdg_mon        |    SEC VIP   | SEC VIP      | Checker and Reporter for wdog under security subsystem       |
+----------------------+--------------+--------------+--------------------------------------------------------------+
|   snvs_trim_mon      |    SEC VIP   | **SNVS VIP** | Checker for Voltage, Clock, Temperature Tampers Trim         |
+----------------------+--------------+--------------+--------------------------------------------------------------+
|   snvs_trim_qmax_mon |    SEC VIP   | SEC VIP      | Additional Checker for SNVS&ANALOG trim, like OSC, VBG,etc.. |
+----------------------+--------------+--------------+--------------------------------------------------------------+

Basically, monitors are used in security vip behave like a golden model to check and report some expected behavior under certain condition.
They can be  **Always-Check** [#A-C]_ or **Auto-Trigger** [#A-T]_ statements.
Another common scenario is using CAPI methods to communicate and trigger Checker.

A simple flow may be:

#. **Front-door setup, either by SW or driver through Pads.**
#. **Trigger mailbox or trigger from C side.**
#. **Receive mailbox or trigger in monitor and execute checker.**


Drivers
-------
There are two kinds of drivers in security vip. 

#. Module-Instance based driver. Usually for pads like **ONOFF**, **TAMPER** .
#. Procedure-Block based driver. This is kind of **BackDoor** method to operate internal signals. Not recommended but sometimes useful.

See summary below.

+----------------------+------------+--------------+-----------------+----------------------------------------------------------------------+
| Drivers              | Module DIR | Instance DIR | Type            | Description                                                          |
+----------------------+------------+--------------+-----------------+----------------------------------------------------------------------+
| snvs_onoff_drv_qmax  | SEC VIP    | SEC VIP      | Module-Instance | Drive **OnOFF** pad, mostly based on mailbox and trigger             |
|                      |            |              |                 | Please notice, there is an internal initial-block used to            |
|                      |            |              |                 | accelerate 32k clk.                                                  |
|                      |            |              |                 | Add **SNVS_LP_SPEED_32kHZ_CLK** to **sim_arg** to enable this        |
+----------------------+------------+--------------+-----------------+----------------------------------------------------------------------+
| snvs_tamper_drv_qmax | SEC VIP    | SEC VIP      | Module-Instance | Drive **SNVS TAMPER** pads. We have 10 external tamper and five      |
|                      |            |              |                 | of which can be used as active tamper. It differs with QM where      |
|                      |            |              |                 | only 4 external tamper and 2 active tamper exist.                    |
+----------------------+------------+--------------+-----------------+----------------------------------------------------------------------+
| adm_gpr_test_drivers | NULL       | SEC VIP      | Procedure-Block | Drive inner logic of ADM . This driver is the most important one in  |
|                      |            |              |                 | security VIP since several Vectors and many test cases are involved  |
|                      |            |              |                 | and lots of force assignments are used inside . Need special care    |
+----------------------+------------+--------------+-----------------+----------------------------------------------------------------------+
| ram_ecc_corrupt      | NULL       | SEC VIP      | Procedure-Block | Specific driver to generate signal-bit and multi-bits ECC error      |
|                      |            |              |                 | in SECO RAM.                                                         |
+----------------------+------------+--------------+-----------------+----------------------------------------------------------------------+
| sec_wdg_drv          | NULL       | SEC VIP      | Procedure-Block | Specific driver to some internal wdog signals.E.g. ipg_clk, wdg_rsto |
|                      |            |              |                 | cm0p_sleeping,etc..  Not Recommended in my personal opinion          |
+----------------------+------------+--------------+-----------------+----------------------------------------------------------------------+
| snvs_msic            | SNVS VIP   | SEC VIP      | Module-Instance | SNVS msic control                                                    |
+----------------------+------------+--------------+-----------------+----------------------------------------------------------------------+

For any **force assignment** insides the drivers, we need to make sure it won't affect other vectors. Usually we have **MailBox**, **Trigger**,  **Macro**, or **sim_arg** to block [#block]_ those assignments . As with those that are not blocked, rigorous review is needed to avoid side-effect.

C ENV
---------
Besides of verilog&systemverilog component, another big part of resource provided by our VIP is about C . 
As we know, out TB consists of **C** and **Verilog**, it's quite a typical style for verification TB if any components akin to CPU exist in the chip. Here  **C** refers to the complete compile toolsuits and related resource like included files,lib,etc. to build the final **HEX** file for specific RAM, ROM, or Fuses as mentioned before. Well, apparently, Fuses are not part of memory of code for our core. That's just a trick for convinience . See section Fuse Load for details.
The so called C ENV provided by our VIP can be divided into two parts.

#. Contexts and APIs 
#. Memory definition 

Contexts and APIs   
~~~~~~~~~~~~~~~~~~~~
Context is a key concept in our C side of TB. Basically for every IP inside the subsystem, there would be a relative struct, where some basic items are declared, like REGS_BASE, TRIG_BASE, MBOX_BASE, etc.. 

In our security VIP , contexts are declared or created under ::
   
   $DESIGN_DIR/testbench/imx_subsys_vip/v_sc_imx_security_vip/tool_data/compiler/include

Please notice, not all context prototype are declared under our **SEC VIP**, since for many of them are already done under **IP VIP**, what we need to do is define the API to create such an instance and add it to the top TB. Let's take **ROMCP**  as an example to briefly illustrate this.

In our chip, we have three ROMCP: 

+ ROMCP under **LSIO** subsystem, it's for other masters like **CA35**
+ ROMCP under **SCU** subsystem, it's for **SCU_CM4**
+ ROMCP under **Security** subsystem, it's for our **SECO** and this is the one we need to care.

The context for ROMCP is declared under ROMCP VIP::
   
   $DESIGN_DIR/testbench/common_blocks/v_ms_imx_romcp_vip/tool_data/compiler/include/romcp.h 
   
   typedef struct
   { char*          NAME;
     address_t      REGS_BASE;
     uint16_t       TRIG_BASE;
     uint16_t       MBOX_BASE;
     vector_t       ROMCP_IPS_ERROR_VECTOR;
     uint32_t       IPS_SLOT_SIZE;
     uint8_t        ips_err_int_cnt;  
     uint32_t*      ROMCP_REG_ADDR_OFFSET_LST;
     uint32_t*      ROMCP_REG_RST_LST;
     uint32_t*      ROMCP_REG_BIT_MASK_LST;
     uint32_t*      ROMCP_REG_RW_BIT_MASK_LST;
   } ROMCP_CONTEXT;

Then, we just create the creation API under our SEC VIP ::
   
   $DESIGN_DIR/testbench/imx_subsys_vip/v_sc_imx_security_vip/tool_data/compiler/include/ss_sec_romcp_context.h 
   
   void ss_sec_create_context_romcp0(ROMCP_CONTEXT** context){
      static ROMCP_CONTEXT static_context;  
      static_context.NAME                    = "SEC_ROMCP";
      #ifndef SECO_ACCESS
      static_context.REGS_BASE               =  SEC_ROMCP_BASE; //NEED TO CHANGE THE NAME TO ROMCP(IT IS FLEXBUS NOW in sec_base_address file)
      #else 
      static_context.REGS_BASE               =  SECO_ROMCP_BASE;
      #endif
      static_context.ips_err_int_cnt           = 0;
      static_context.ROMCP_REG_ADDR_OFFSET_LST = (uint32_t*)&sec_romcp_reg_addr_offset_lst[0];
      static_context.ROMCP_REG_RST_LST         = (uint32_t*)&sec_romcp_reg_rst_lst[0];
      static_context.ROMCP_REG_BIT_MASK_LST    = (uint32_t*)&sec_romcp_reg_bit_mask_lst[0];
      static_context.ROMCP_REG_RW_BIT_MASK_LST = (uint32_t*)&sec_romcp_reg_rw_bit_mask_lst[0];
      //static_context.TRIG_BASE             = SCU_ROMCP_TRIG_BASE; //FUTURE USAGE
      //static_context.MBOX_BASE             = SCU_ROMCP_MBOX_BASE; //FUTURE USAGE
      static_context.ROMCP_IPS_ERROR_VECTOR  = HARD_FAULT_XCP;
      *context=&static_context;
    }

Last step, we modify the top_level file::
   
   $DESIGN_DIR/testbench/blocks/soc_tb/tool_data/compiler/include/context_romcp.h
   
   void romcp_populate_request(ip_request_t* ip_request){
      #ifdef ROMCP_ID
        ip_request->context_id            = ROMCP_ID;
      #endif  
        ip_request->context_array_length  = NUM_ROMCP_IN_SYSTEM;
        ip_request->init_context          = romcp_init_context;
      }

   void romcp_init_context(ip_request_t* ip_request){
     switch (ip_request->context_id){
       case (LSIO_ROMCP_0):   INFO("romcp_init_context","Initialize LSIO ROMCP context");
                           ss_lsio_create_context_romcp0((ROMCP_CONTEXT**)(&ip_request->context_ptr));
                           //b48029 - Define added temporarily to avoid errors when the Security controller
                           //Core M0 parse these lines , seco_ip_request needs to add this filed to ip_request strunct
                           #ifndef SECO_ACCESS
                           ip_request->clock_config   = DSC_ENABLE_LSIO_DSC;
                           #endif
               INFO("STIM", "Initialize LSIO DSC");
                           INFO("STIM", "Initialize LSIO ROMCP clock");

                           break;
       case (SCU_ROMCP):   INFO("romcp_init_context","Initialize SCU ROMCP context");
                           ss_scu_romcp_context((ROMCP_CONTEXT**)(&ip_request->context_ptr));
                           #ifndef SECO_ACCESS
                           DSC_INIT_SCU_SYS();
                           #endif 
                           break;

        case(SEC_ROMCP_0): INFO("romcp_init_context","Initialize LSIO ROMCP context");
                           ss_sec_create_context_romcp0((ROMCP_CONTEXT**)(&ip_request->context_ptr));
                           //ip_request->clock_config   = DSC_ENABLE_LSIO_DSC;slice ?
               INFO("STIM", "Initialize SEC DSC");
                           INFO("STIM", "Initialize SEC ROMCP clock");

                           break;
       default:            ERROR("populate_romcp","trying to populated an unsupported ROMCP");
                           break;
     }
   }

Finally, all set done. Just to reminder, if you are building a new context, remember to add necessary information in **context_api.h** and **ip_request.h**, just like how we do with **MMCAU** [#MMCAU]_ for SECO, which is added in i.MXQXP .   

To specify which ROMCP is involved in your test case , just assign correct ID to the IP in your arg file like below :: 
   
   SCU_C_ARG += ROMCP_ID=SEC_ROMCP_0

Based on the context, some APIs are created for some specific tasks, and it would be very convenient for reuse if the same type IP are integrated with more than one instances .
For any function used in the test cases, you'd better look into our SEC VIP or IP VIP for detailed information. Personally ,I recommend using `CSCOPE`_ to aid you for quick lookup.





.. _CSCOPE: http://cscope.sourceforge.net/
.. rubric:: Footnotes
   
.. [#Checker] Checker here refers to the statements declared using **assert**, **cover** , **check**, or any TB related Pass/Fail trigger like **`ERROR**, **UVM_ERROR**
.. [#A-C] Checks that execute every fix interval time, e.g. Triggered by clock or time slot, etc..
.. [#A-T] Automatically Triggered under such conditions that are consist of original internal hardware signals. 
.. [#block] Disable statements with special switches .
.. [#MMCAU] This is the design change in QXP, a new MMCAU is added to SECO. To avoid touching MMCAU VIP, we re-declare the specific context under our SEC VIP.

