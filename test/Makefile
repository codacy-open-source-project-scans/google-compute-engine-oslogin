TOPDIR = $(realpath ..)

ifeq ($(GTEST_DIR),)
GTEST_DIR = /usr/src/googletest/googletest/
endif

TEST_RUNNER = ./test_runner --gtest_output=xml
NEW_TEST_RUNNER = ./new_test_runner --gtest_output=xml
SSHCA_TEST_RUNNER = ./sshca_runner --gtest_output=xml --gtest_filter="SSHCATests.*"
CPPFLAGS += -I$(TOPDIR)/src/include -I$(TOPDIR)/third_party/include -I/usr/include/json-c -I$(GTEST_DIR) -isystem $(GTEST_DIR)/include
CXXFLAGS += -g -Wall -Wextra
LDLIBS = -lcurl -ljson-c -lpthread

.PHONY: all clean alltests ping reset
.PHONY: gtest prowtest non_network_tests network_tests
.DEFAULT_GOAL := all

all: test_runner new_test_runner non_network_tests

clean :
	rm -f test_runner new_test_runner test_detail.xml *.o

gtest-all.o: $(GTEST_DIR)/src/gtest-all.cc
	$(CXX) $(CXXFLAGS) $(CPPFLAGS) -c $^

test_runner: oslogin_utils_test.o $(TOPDIR)/src/oslogin_utils.o gtest-all.o
	$(CXX) $(CXXFLAGS) $(CPPFLAGS) $^ -o $@ $(LDLIBS)

sshca_runner: oslogin_sshca_test.o $(TOPDIR)/src/oslogin_utils.o $(TOPDIR)/src/oslogin_sshca.o gtest-all.o
	$(CXX) $(CXXFLAGS) $(CPPFLAGS) $^ -o $@ $(LDLIBS)

sshca_tests: sshca_runner
	$(SSHCA_TEST_RUNNER)

new_test_runner: oslogin_test.o gtest-all.o
	$(CXX) $(CXXFLAGS) $(CPPFLAGS) $^ -o $@ $(LDLIBS)

new_tests: new_test_runner $(TOPDIR)/src/nss/new_nss_oslogin.c
	$(NEW_TEST_RUNNER) ${GTESTARGS}

non_network_tests: test_runner new_test_runner
	$(TEST_RUNNER) --gtest_filter=*-GetGroupByTest.*:GetUsersForGroupTest.*
	$(NEW_TEST_RUNNER) --gtest_filter=ParserTest.*

network_tests: test_runner ping reset
	$(TEST_RUNNER) --gtest_filter=GetGroupByTest.*:GetUsersForGroupTest.*

# run as $ make tests GTESTARGS="--gtest_filter=GetGroupByTest.*"
alltests: test_runner
	$(TEST_RUNNER) ${GTESTARGS}

ping:
	nc -vzw2 169.254.169.254 80 >/dev/null 2>&1

reset:
	curl -Ss http://169.254.169.254/reset >/dev/null 2>&1
