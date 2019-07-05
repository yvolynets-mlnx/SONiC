#### Approach 1

#### Requirements
Move existed loganalyzer to be supported by pytest framework.

#### Overview
Current loganalyzer implementation consists of the following parts:
Init - copy loganalyzer.py to the DUT. Add start marker to the syslog.

Analyze - copy loganalyzer.py and all match,expect,ignore files to the DUT. Extract logs from syslog. Add end marker. Analyze logs based on regexp described in match,expect,ignore files.
Generate summary and result files.

End - Check if "TOTAL MATCHES != 0" then generate system dump. Store system dump, result and summary files to the ansible host.

In proposed approach script copies loganalyzer.py and files with regexp to the DUT and runs syslog analyze on the DUT.

Current loganalyzer implementation description:
https://github.com/Azure/SONiC/wiki/LogAnalyzer

**Development**
**Implement:**
	Module "loganalyzer.py" with class "Loganalyzer".
	"Loganalyzer" class - executes shell commands on DUT.
	**"Loganalyzer" class interface:**
- __init__(match: list, expect: list, ignore: list)
- exec_cmd(cmd) - execute shell command on the DUT
- extract_syslog() - calls "extract_syslog" from "log_processor.py"
- upload_search_files(dest) - upload match, expect, ignore files to the DUT
- parse_summary_file() - convert generated on DUT "result.loganalysis.{TEST_NAME}..." file to json.
- parse_result_file() - convert generated on DUT "summary.loganalysis.{TEST_NAME}" file to json.
- init() - copy log_processor.py to the DUT. Add start marker to the DUT syslog.
- analyze() - calls extract_syslog(), upload_search_files(dest). Copy log_processor.py and all match, expect, ignore files to the DUT. Calls "log_processor.py" on the DUT with "--action analyze", which - extract syslog logs by start/end markers. Analyze logs based on regexp described in match, expect, ignore files. Generate summary and result files. Calls parse_summary_file(), parse_result_file().  Return converted to dictionary "result.loganalysis.{TEST_NAME}..." and "summary.loganalysis.{TEST_NAME}" files.

**Implement**:
Module "log_processor.py" with class "LogProcessor".
"log_processor.py" - python module which is running on the DUT. Module perform some actions based on script input parameters. It will be based on the legacy loganalyzer.py.
Supported actions: init, analyze, extract_syslog
- init and analyze are described above (legacy).
- extract_syslog - Extract all syslog entries since the latest start marker (new).

"LogProcessor" class interface:
- Includes legacy "LogAnalyzer" class from loganalyzer.py module.
- New:
- extract_syslog(dst="/tmp/syslog") - Extract all syslog entries since the latest start marker and store them to "/tmp/syslog" folder on the DUT.

Usage example:

	def test(localhost, ansible_adhoc, testbed):
		loganalyzer = Loganalyzer(match="{PATH}/loganalyzer_common_match.txt", expect="{PATH}/loganalyzer_common_expect.txt", ignore="{PATH}/loganalyzer_common_ignore.txt")
		loganalyzer.init()
		# Perform test steps
		result = loganalyzer.analyze()
		if result["summary"]["TOTAL MATCHES"] != 0:
			loganalyzer.fetch_fail_artifacts(dst="test/{}".format(loganalyzer.unic_test_name))
		pytest.fail("SOME_MESSAGE\n{}\n{}".format(result["summary"], result["match_messages"]))

#### Approach 2
#### Requirements
Move existed loganalyzer to be supported by pytest framework.

#### Overview
At this approach all the work can be done on the Ansible host if script will download extracted syslog file to the Ansible host.

Current loganalyzer implementation description:
https://github.com/Azure/SONiC/wiki/LogAnalyzer

#### Development
Module "loganalyzer.py" with class "Loganalyzer".

"Loganalyzer" class interface:
- load_common_config() - Load regexps from common configuration files: match, expect and ignore which are located in some configuration directory or with current module.
- get_search_regexp() - returns dictionary with defined regular expressions which are used for logs processing.
- update_match(action="pop/add", type="match/expect/ignore", *args) - add or remove regexp for syslog processing.
Supported parameters values:
action: "pop", "add"
type: "match", "expect", "ignore"
args: *args
- init() - Add start marker to the DUT syslog.
- analyze() - Extract syslog and copy it to ansible host. Analyze extracted syslog files localy. Return python dictionary object.
Return example:
		{"summary":
		"counters": {"match": 1, "expected_match": 0, "expected_missing_match": 0},
		"match_files": {"/tmp/syslog": {"match": 0, "expected_match": 32},
						"/tmp/syslog1": {"match": 0, "expected_match": 15}},
						"match_messages": {"/tmp/syslog1": ["Message 1", "Message 2", "Message n"],
						 "/tmp/syslog2": ["Message 1", "Message 2", "Message n"]}
	}
- run_cmd(callable) - call function and analyze DUT logs, return the same result as "analyze" function

Usage example:

	from loganalyzer import Loganalyzer
	def test(localhost, ansible_adhoc, testbed):
		loganalyzer = Loganalyzer()

		# If it is a need to load common search regular expressions. It will load common: match, expect, ignore regular expressions.
		loganalyzer.load_common_config()

		# Add start marker to DUT syslog
		loganalyzer.init()

		# Add search marker specific for None MAC test
		loganalyzer.update_match(action="add", type="match", "Runtime error: can't parse mac address 'None'")
		
		# Execute test steps here...

		result = loganalyzer.analyze()
		assert result["summary"]["counters"]["match"] == 0, "Failure message\n{}\n{}".format(result["summary"], result["match_messages"])
