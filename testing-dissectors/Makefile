
SHELL=/usr/bin/env bash -o pipefail -o errexit

.PHONY: test actual

all: test

data:
	# Depends on wireshark-lua repo, set LUA_BASE to the path, otherwise we
	# clone in current directory.
	test -d "$(LUA_BASE)" || \
	  test -d wireshark-lua || \
	    git clone --depth=1 \
	      https://github.com/Open-Markets-Initiative/wireshark-lua.git
	# Depends on pcap files, set PCAP_BASE if you need to specify
	test -d "$(PCAP_BASE)" || \
	  test -d Directory || \
	    git clone --depth=1 \
	      https://github.com/Open-Markets-Initiative/Directory.git

actual: data pcap-to-lua.txt
	# The `actual` directory is generated on each test run and compared
	# against `fixtures` which is committed to the repo.
	rm -rf actual
	./gen-data ./pcap-to-lua.txt actual

tests: data pcap-to-lua.txt
	# The `tests` file is generated based on the `fixtures` directory, and
	# both are committed to the repo, so this target typically only runs
	# when pcap-to-lua.txt changes.
	./gen-data ./pcap-to-lua.txt fixtures
	./gen-tests fixtures > tests

test: actual
	# generate `actual` directory and run the tests
	bats tests

clean:
	rm -rf actual

reset: clean
	rm -rf wireshark-lua fixtures
