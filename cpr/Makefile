# FPGA variables
SOURCES= src/cpr.v

# COCOTB variables
export COCOTB_REDUCED_LOG_FMT=1
export PYTHONPATH := test:$(PYTHONPATH)
export LIBPYTHON_LOC=$(shell cocotb-config --libpython)

all: test_cpr

# if you run rules with NOASSERT=1 it will set PYTHONOPTIMIZE, which turns off assertions in the tests
test_top:
	echo "the top module was renamed to rgb_mixer. Please run make test_rgb_mixer instead"

test_cpr:
	rm -rf sim_build/
	mkdir sim_build/
	iverilog -o sim_build/sim.vvp -s cpr -s dump -g2012 src/cpr.v test/dump_cpr.v
	PYTHONOPTIMIZE=${NOASSERT} MODULE=test.test_cpr vvp -M $$(cocotb-config --prefix)/cocotb/libs -m libcocotbvpi_icarus sim_build/sim.vvp
	! grep failure results.xml

test_gate_level:
	rm -rf sim_build/
	mkdir sim_build/
	iverilog -o sim_build/sim.vvp -s cpr -s dump -g2012 gl/cpr.v test/dump_gl.v -I $(PDK_ROOT)/sky130A
	PYTHONOPTIMIZE=${NOASSERT} MODULE=test.test_gl vvp -M $$(cocotb-config --prefix)/cocotb/libs -m libcocotbvpi_icarus sim_build/sim.vvp

show_%: %.vcd %.gtkw
	gtkwave $^

# FPGA recipes

show_synth_%: src/%.v
	yosys -p "read_verilog $<; proc; opt; show -colors 2 -width -signed"

%.json: $(SOURCES)
	yosys -l fpga/yosys.log -p 'synth_ice40 -top rgb_mixer -json $(PROJECT).json' $(SOURCES)

%.asc: %.json $(ICEBREAKER_PIN_DEF) 
	nextpnr-ice40 -l fpga/nextpnr.log --seed $(SEED) --freq 20 --package $(ICEBREAKER_PACKAGE) --$(ICEBREAKER_DEVICE) --asc $@ --pcf $(ICEBREAKER_PIN_DEF) --json $<

%.bin: %.asc
	icepack $< $@

prog: $(PROJECT).bin
	iceprog $<

# general recipes

lint:
	verible-verilog-lint src/*v --rules_config verible.rules

clean:
	rm -rf *vcd sim_build fpga/*log fpga/*bin test/__pycache__

.PHONY: clean
