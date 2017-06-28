Patterns_Example
================
We are going to take some examples to illustrate how they work.

scu_snvs_tamper_active3_fail
----------------------------
This pattern locates at vector **scu_sec_snvs**. It's a new pattern deriving from scu_snvs_tamper_active1_fail since we have 5 active tamper instead of 2 now. 

Take a look at the arg file::
   
   $DESIGN_DIR/testbench/blocks/soc_tb/vectors/scu_sec_snvs/ stimulus/arg/scu_snvs_tamper_active3_fail.arg
   
   SCU_STIM   = snvs_tamper_active_fail
   SCU_C_ARG += SNVS_ID=SNVS0

   SCU_C_ARG += SKIP_DB_CONFIG=1

   sim_arg        := +SIMULATION_TIMEOUT=3000000 $(sim_arg)
   sim_arg        := +SNVS_ACTIVE_TAMPER_ENABLE=1 $(sim_arg) 

   SCU_C_ARG += ACT3_TAMPER=1
   SCU_C_ARG += EXT5_TAMPER=1 
   sim_arg   := +SNVS_EXTERNAL_TAMPER_ENABLE=1 $(sim_arg)
   sim_arg        := +SNVS_LP_SPEED_32KHZ_CLK $(sim_arg)

For all these active tamper patterns, we are using the same c file with different SCU_C_ARG to indicate which tampers are used. 

For another three sim_args , you can find their function in driver snvs_tamper_drv_qmax.

Regarding the design changes, we will need to expand the driver to support 10 external tampers and 5 active tampers. Also, since we are using Trigger and MailBox to communicate between C and V, we also need to expand C source file ::
   
   $DESIGN_DIR/testbench/imx_subsys_vip/v_sc_imx_security_vip/tool_data/compiler/include/snvs_util.h
   
Ok, almost done. Consult with design team, we know that a register bit is added to switch the shared pads to tamper mode , so we need to configure that at appropriate location in our C file::
   
   // enable tamper_en_1p8v
   BSET32(SNVS_LP_TRIM_TEST, BIT29);

For these active tamper patterns , we encountered a small issue with tamper pad model. You may meet such an error info ::
   
   SNVS_PADS_ERROR @ 2236957.446000 ns --: [testbench.top.ss_scu.snvs_lp_top_qx.snvs_pads_drivers.tamper_out_04] Tamper disable && pad floating && none pull is not allowed!!!
   SNVS_PADS_ERROR @ 2236957.446000 ns --: [testbench.top.ss_scu.snvs_lp_top_qx.snvs_pads_drivers.tamper_out_03] Tamper disable && pad floating && none pull is not allowed!!!
   SNVS_PADS_ERROR @ 2236957.446000 ns --: [testbench.top.ss_scu.snvs_lp_top_qx.snvs_pads_drivers.tamper_out_02] Tamper disable && pad floating && none pull is not allowed!!!
   SNVS_PADS_ERROR @ 2236957.446000 ns --: [testbench.top.ss_scu.snvs_lp_top_qx.snvs_pads_drivers.tamper_out_01] Tamper disable && pad floating && none pull is not allowed!!!
   SNVS_PADS_ERROR @ 2236957.446000 ns --: [testbench.top.ss_scu.snvs_lp_top_qx.snvs_pads_drivers.tamper_out_00] Tamper disable && pad floating && none pull is not allowed!!!

These actually is not a true issue , confirmed with design team. Bus such error info will result in failure when collected by Regression, even may print a big PASS at the end. So we request the design team to add a switch to disable such error reporter. ::
   
   `ifndef DISABLE_TAMPER_PULL_CHECK
       if ((TAMPER_EN_1P8 === 0) && (PAD===1'bz) && (PUN===1'b1) && (PDN===1'b1))
          `SNVS_PADS_ERROR($sformatf("SNVS_PADS_ERROR @ %0t: [%m] Tamper disable && pad floating && none pull is not allowed!!!", $realtime));
   `endif

You can try add this *DISABLE_TAMPER_PULL_CHECK* to comp_arg to disable this error reporter. Here exists an typo in version 1.2 of block.arg of our SEC VIP resulting that it's still not disabled. You can re-add it if needed.

seco_mmcau_cmd
--------------
This pattern is picked up as an example for two reasons.

+ MMCAU is newly added to SECO 
+ Some extra work is needed to get it work for SECO.
  
Well, let's get to the point directly. This pattern originally derives from the one under vector scu_mmcau, but that is for CM4 alike core, and what we have under security subsystem is a M0+ core. The key difference is that we don't have a FPU in M0+ core . As the result, M0+ fails to execute those float point operations existing in our pattern. To make it work, we need to use SW lib instead of FPU hardware decoding. Fortunately I find the appropriate gcc lib for ARMv6-M core in i.MX6UL database and add it to our link process ::
   
   $DESIGN_DIR/testbench/blocks/soc_tb/vectors/scu_sec_mmcau/stimulus/makefile.defs
   
   LDFLAGS       = -L /proj/imx6ul/arm_compiler/arm-2013.05-23/lib/gcc/arm-none-eabi/4.7.3/armv6-m -lgcc

Here we can create a makefile.defs under the stimulus directory to bypass the one provided by the TB [#TB_MAKE]_ . Also we create another link script to allocate memory for sections induced by added gcc lib . Then the pattern is working.

Our top TB set some entries for user-define makefiles. For details, please take some time and read the makefiles in our TB .

seco_adm_glitch_det
-------------------

This pattern is quite unique due to its special design method . It's designed to check the clock glitch detector inside ADM. To work correctly , it fully depends on the precise delay value between two signals . But during RTL stage they directly connect these two signals and rely on the BackEnd to insert delay cells . 
So we must take different strategies to check this in both RTL and GLS views.

For RTL view, there are two things we need to complete through BackDoor .

+ Set appropriate delay and override the driven relationship between those two signals.
+ Insert glitch on the ADM clock 

For GLS view, we only need to insert the glitch at appropriate time point. GLS simulation for this pattern is absolutely necessary 
These are all done in driver *adm_gpr_test_drivers*.

Then to check the result, firstly we set assertion in the driver, and on the other hand, we check the FrontDoor IRQ flow.





.. rubric:: Footnotes

.. [#TB_MAKE] They are under $DESIGN_DIR/testbench/blocks/imx8_common_tb/testbench
