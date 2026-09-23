CC = mpicc
CFLAGS = -Wall -O2
PROGS = sum_bcast sum_scatter sum_gather sum_reduce sum_allreduce sum_scan

all: $(PROGS)

%: %.c
	$(CC) $(CFLAGS) -o $@ $<

run: all
	@for p in $(PROGS); do echo "=== $$p ==="; mpirun -np 4 ./$$p; echo; done

clean:
	rm -f $(PROGS)

.PHONY: all run clean
