include ../common/Makefile.env
SIM_SOURCES?=sources.txt
#SIM_SOURCES?=instrument.txt
SIM_TOP_MODULE?=test_axis_async_fifo
TEST_DIR?=test
RTL_WORK_DIR?=work

VERILATOR=../../veripass/verilator/bin/verilator
VERILATOR_OPT?=
VERILATOR_OPT+= -cc -timescale-override 10ps/10ps
VERILATOR_OPT+= -Wno-WIDTH -Wno-LITENDIAN -Wno-UNPACKED -Wno-BLKANDNBLK -Wno-CASEINCOMPLETE -Wno-CASEX -Wno-UNOPTFLAT -Wno-SPLITVAR -Wno-PINMISSING -Wno-ALWCOMBORDER -Wno-TIMESCALEMOD -Wno-VLTAG
VERILATOR_OPT+= -trace-fst -trace-structs -comp-limit-syms 0
VERILATOR_OPT+= -assert -trace-max-array 65536 -trace-max-width 65536
VERILATOR_ROOT=$(CURDIR)/../../veripass/verilator

CXX?= g++
CXX_OPT?=
CXX_OPT+= -g
CXX_OPT+= -I$(VERILATOR_ROOT)/include -I$(RTL_WORK_DIR)
CXX_OPT+= -lz
VERILATOR_CXX_FILES?= $(VERILATOR_ROOT)/include/verilated.cpp
VERILATOR_CXX_FILES+= $(VERILATOR_ROOT)/include/verilated_vcd_c.cpp
VERILATOR_CXX_FILES+= $(VERILATOR_ROOT)/include/verilated_fst_c.cpp
TEST_CXX_FILES?=$(shell find $(TEST_DIR) -name '*.cpp')
TEST_RTL_SIMLIB?=$(RTL_WORK_DIR)/V$(SIM_TOP_MODULE)__ALL.a
TEST_BIN?= test_axis_async_fifo

all: clean verilator sw

verilator:
	$(VERILATOR) $(VERILATOR_OPT) -F $(SIM_SOURCES) -top-module $(SIM_TOP_MODULE) --Mdir $(RTL_WORK_DIR)
	$(MAKE) -C $(RTL_WORK_DIR) -f V$(SIM_TOP_MODULE).mk VERILATOR_ROOT=$(VERILATOR_ROOT)

sw: verilator
	$(CXX) $(CXX_OPT) $(VERILATOR_CXX_FILES) $(TEST_CXX_FILES) $(TEST_RTL_SIMLIB) $(CXX_OPT) -o $(TEST_BIN)

clean:
	rm -rf $(RTL_WORK_DIR) $(TEST_BIN) *.vcd *.fst
	rm -rf build_* *info.txt

sim:
	./$(TEST_BIN)

wave:
	gtkwave *.fst >/dev/null 2>/dev/null &

# for resource-util synthesize
TOP_MODULE=axis_fifo_wrapper
TOP_MODULE_SYNTH=axis_fifo_wrapper
RTL_SOURCES=sources-veripass.txt
TCLGEN_SRCS=
SV2V_OUT_FILES+=withtask
CODE_GEN+=withtask.v bug_step2.v

withtask.v: ${RTL_SOURCES} n6.instrument.cfg
	${TOOLS} --top ${TOP_MODULE} -F ${RTL_SOURCES} --config n6.instrument.cfg --reset "async_rst"

DEPTH_SWEEP_PREFIX+=depth_sweep_n6
depth_sweep_n6_d%.v: ${RTL_SOURCES} n6.instrument.cfg
	${TOOLS} --top ${TOP_MODULE} -F ${RTL_SOURCES} --config n6.instrument.cfg --reset "async_rst" --config-override="{\"LOG2_SAMPLE_DEPTH\":$*, \"SV2V_OUTPUT\":\"$@\", \"ILA_OUTPUT\":\"$(basename $@).ila.tcl\"}"
include ../common/Makefile.ILA.rules
