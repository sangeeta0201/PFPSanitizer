CC = clang -pg
CCE = clang -pg -fno-inline -emit-llvm 
OPT = opt -load $(FPSAN_HOME)/llvm-pass/build/FPSan/libFPSanitizer.so -fpsan
GCC := clang -pg

SRC_DIR := .
OBJ_DIR := .
SRC_FILES := $(wildcard $(SRC_DIR)/*.c)
OBJ_FILES := $(patsubst $(SRC_DIR)/%.c,$(OBJ_DIR)/%.o,$(SRC_FILES))
OBJ_FILES_FP := $(patsubst $(SRC_DIR)/%.c,$(OBJ_DIR)/%.bc,$(SRC_FILES))
CPPFLAGS := -g -std=c11  -lmpfr -lm  -lpthread
CFLAGS= -fno-slp-vectorize -std=c11 -L$(FPSAN_HOME)/runtime/obj -lfpsanitizer -lmpfr -lm  -lstdc++ -L$(FPSAN_HOME)/tbb/build -ltbb -I$(MPFR_HOME)/include
INCLUDE_CFLAGS = -g -O1 -DHYPRE_SEQUENTIAL=1
CCFLAGS =\
 -I..\
 -I../utilities\
 -I../seq_mv\
 -I../parcsr_mv\
 -I../parcsr_ls\
 ${INCLUDE_CFLAGS}
RANLIB = ranlib

HEADERS =\
 csr_matrix.h\
 headers.h\
 HYPRE_seq_mv.h\
 seq_mv.h\
 vector.h

all: seq_mv.fp.bc libseq_mv.a

libseq_mv_fp.a: seq_opt_fp.o ${HEADERS} 
	@echo  "Building $@ ..."
	ar -rcu $@ $^
	${RANLIB} $@

seq_opt_fp.o: seq_opt_fp.bc 
	llc $< -filetype=obj -o $@

seq_opt_fp.bc: seq_mv.fp.bc  
	$(OPT) <seq_mv.fp.bc> $@ $^

seq_mv.fp.bc: $(OBJ_FILES_FP)
	llvm-link $^ -o $@ 

$(OBJ_DIR)/%.bc: $(SRC_DIR)/%.c
	$(CCE) $(CCFLAGS) -c -o $@ $< 

libseq_mv.a: $(OBJ_FILES)
	@echo  "Building $@ ..." 
	ar -rcu $@ $^
	${RANLIB} $@

$(OBJ_DIR)/%.o: $(SRC_DIR)/%.c
	$(GCC) $(CCFLAGS) -c -o $@ $<

clean:
	rm -rf  libseq_mv.a libseq_mv_fp.a *.out *.o *.dSYM *.bc seq_opt_fp
