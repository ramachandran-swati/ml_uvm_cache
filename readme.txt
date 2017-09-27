GITHUB DOWNLOAD DESIGN:
	git clone <url of design database>
	Example: git clone https://github.tamu.edu/venkateshwar-k/design_update
	It will ask your username and password. Enter your Unix account credentials.

SIMULATION README
cd design_update
source setup.bash //It will set both UVM and Jasper environment
cd project 
---There are 5 sub-directories: 
	design : All the design files
	gold : golden memory and arbiter files
	prop : Jasper Gold properties and bind files
	sim : Simulation Directory
	test : Simulation tests
	uvm : Simulation testbench

cd sim
--Important files:
	file_list.f : Files to be compiled
	all.bash: It will run every test once with seed 1
	regress.bash: It will run every test 20 times, with random seed
	gen_cov_html_report.bash :It will coverage obtained from all regression runs

	Vmanager Files: ProjectVerificationPlanSpring2017_May23.vplanx , run_vm.vsif, run_vm.f
		ProjectVerificationPlanSpring2017_May23.vplanx : verification plan executable file
		run_vm.f: Compile time options and files for vmanager
		run_vm.vsif: This is regression file, number of tests to be run, how many times a test should run, seeds all things are defined in this file.

1. For compilation: Run the below command
irun -f cmd_line_comp_elab.f

2. Simulate: 
a. Simulate single testcase:
	in gui mode: 
		run command:  irun -f run_mc.f //Set the name of your test in run_mc.f
	else: 
		run command: irun -f sim_cmd_line.f +UVM_TESTNAME=<test_name> //test_name are there in file /design_update/project/test/test_lib.svh

b. Run regression:
	------Without vmanager:
		There are 2 scripts:
					run command: ./all.bash
					or run command: ./regress.bash
		A new directory cov_work will be formed
		To get coverage: 
					run command: ./gen_cov_html_report.bash
		To analyze coverage:
			Launch imc.
					run command: imc
			Once imc opens, load your coverage data formed in /design_update/project/sim/cov_work/scope/ALL 
			Analyze coverage.

	------With vmanager:
		Launch vmanager from sim directory
			When launching for first time from your unix account:
					run command: vmanager  -cs  -planner  -server  hera.engr.tamu.edu:8080
			When not launching first time:
					run command: vmanager -cs
			Once vmanager opens:
				1. Click on Regression Center
				2. Click on Launch icon. 
				3. Load vsif file kept in sim directory
				4. Regression will start.
				5. Once it is done, you will see completed in Session Status.
				6. Double click on the name of the session, Analysis Window will open.
				7. Click on vPlan option and load vplanx file kept in sim directory.
				8. Now analyze the coverage.

			If you want to edit your vplan,
				1. Click on Edit vPlan.
				2. You can see all your testcases, assertions, coverage in Right hand side pane under Implementation -> Metrics.
				3. Map whatever you want to map, by selecting the test/assertion,coverage in Right pane and bullet point in Left Plan pane.
				4. Click on Map to existing metric port or Map.
				5. Save your vPlan and Reload.
			

FORMAL VERIFICATION:

All properties are defined in directory design_update/project/prop/
 --Unicore files:
	v_cache_lv1_unicore.sva
	bindings.sva
 --Multicore files:
	mc_v_cache_lv1_unicore.sva
	mc_v_multicore.sva
	mc_bindings.sva

cd design_update/project/sim
Useful scripts:
  --Unicore files:
	jg_file_list.f: file list to compile files in database
	jg_run.tcl: Main source script	
  --Multicore files:
	mc_jg_file_list.f: file list to compile files in database
	mc_jg_run.tcl: Main source script	
Steps:
	1. Invoke tool, type below command on unix terminal
		jg
	2. Once tool opens, source your run script
		For Unicore: source jg_run.tcl
		For Multicore: source mc_jg_run.tcl

Commands inside source script:
	clear -all  //To cleasr previous database
	analyze -sv -f jg_file_list.f //Compiling the list of files	
	elaborate -bbox_a 10000 -auto_hr_info //elaboration: 
					      // -bbox_a 10000 : it suggests that as long as array size is not greater than 10000 tool won't blackbox
	clock core_if.clk //Declaration of clk
	reset rst //Declaring reset
	task -create //By default all the properties and covers are in default task "embedded"
		     //To manage data better, it is good practice to create tasks
	set_prove_time_limit 5000000 //To define maximum time for proving a prrpert, after this time tool will stop the engine.
				     //We are defining it to be maximum, so that tool doesn't stop itself.
	set_prove_orchestration on //It helps in selecting different engines based on which engine is better for your current property
	set_engine_mode {Hp Ht B N Tri} //To select different engines. 
	set_proofgrid_per_engine_max_local_jobs 3 //Running multiple jobs in parallel

Some Important Notes:
	1. We can convert any assertion on-the fly to assumption, by right clicking on the property in property table.
	2. We can exclude any assertion on the fly by right clicking on the property in property table.
	3. To check for dead-ends and conflicts,  run below commands on Jasper console:
		check -deadend
		check -conflicts