ifndef OBJCOPY
OBJCOPY := $(CROSS_COMPILE)objcopy
endif

OUTPUT ?= ./
HOST_CFLAGS := -Wall -Werror -g -fPIC -z noexecstack \
	       -Wno-unused-const-variable -std=gnu11
ENCL_CFLAGS := -Wall -Werror -static -nostdlib -nostartfiles -fPIC \
	       -fno-stack-protector -mrdrnd -std=gnu11
HOST_LDFLAGS := -fPIC -shared -Wl,-Bsymbolic

IS_OOT_DRIVER := $(shell [ ! -e /dev/isgx ])
IS_SGX_FLC := $(shell lscpu | grep -q sgx_lc)

TEST_CUSTOM_PROGS := $(OUTPUT)/encl.bin $(OUTPUT)/encl.ss $(OUTPUT)/liberpal-skeleton-v1.so $(OUTPUT)/liberpal-skeleton-v2.so $(OUTPUT)/signing_key.pem

ifeq ($(IS_OOT_DRIVER),1)
    TEST_CUSTOM_PROGS += $(OUTPUT)/encl.token
else ifeq ($(IS_SGX_FLC),)
    TEST_CUSTOM_PROGS += $(OUTPUT)/encl.token
endif

all: $(TEST_CUSTOM_PROGS)

$(OUTPUT)/liberpal-skeleton-v1.so: $(OUTPUT)/sgx_call.o $(OUTPUT)/liberpal-skeleton-v1.o $(OUTPUT)/liberpal-skeleton.o
	$(CC) $(HOST_LDFLAGS) -o $@ $^

$(OUTPUT)/liberpal-skeleton-v1.o: liberpal-skeleton-v1.c liberpal-skeleton.c
	$(CC) $(HOST_CFLAGS) -c $< -o $@

$(OUTPUT)/liberpal-skeleton-v2.so: $(OUTPUT)/sgx_call.o $(OUTPUT)/liberpal-skeleton-v2.o $(OUTPUT)/liberpal-skeleton.o
	$(CC) $(HOST_LDFLAGS) -o $@ $^

$(OUTPUT)/liberpal-skeleton-v2.o: liberpal-skeleton-v2.c liberpal-skeleton.c
	$(CC) $(HOST_CFLAGS) -c $< -o $@

$(OUTPUT)/liberpal-skeleton.o: liberpal-skeleton.c
	$(CC) $(HOST_CFLAGS) -c $< -o $@

$(OUTPUT)/sgx_call.o: sgx_call.S
	$(CC) $(HOST_CFLAGS) -c $< -o $@

$(OUTPUT)/encl.bin: $(OUTPUT)/encl.elf $(OUTPUT)/sgxsign
	$(OBJCOPY) -O binary $< $@

$(OUTPUT)/encl.elf: encl.lds encl.c encl_bootstrap.S
	$(CC) $(ENCL_CFLAGS) -T $^ -o $@

# If you want to sign a production encalve, you need add '-p' args in sgxsign. In addition, for Intel SGX1 without FLC, please replace signing_key with the product signature key applied to Intel.
$(OUTPUT)/signing_key.pem:
	openssl genrsa -3 -out $@ 3072

$(OUTPUT)/encl.ss: $(OUTPUT)/encl.bin $(OUTPUT)/signing_key.pem
	$(OUTPUT)/sgxsign signing_key.pem $(OUTPUT)/encl.bin $(OUTPUT)/encl.ss

$(OUTPUT)/encl.token: $(OUTPUT)/encl.ss
	sgx-tools gen-token --signature encl.ss --token $@

$(OUTPUT)/sgxsign: sgxsign.c
	$(CC) -I../include -o $@ $< -lcrypto

EXTRA_CLEAN := \
	$(OUTPUT)/encl.bin \
	$(OUTPUT)/encl.elf \
	$(OUTPUT)/encl.ss \
	$(OUTPUT)/sgx_call.o \
	$(OUTPUT)/sgxsign \
	$(OUTPUT)/liberpal-skeleton*.o \
	$(OUTPUT)/liberpal-skeleton*.so \
	$(OUTPUT)/signing_key.pem \
	$(OUTPUT)/encl.token

clean:
	rm -f ${EXTRA_CLEAN}

.PHONY: clean all
