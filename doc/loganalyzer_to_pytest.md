#### Requirements
Move existed loganalyzer to be supported by pytest framework.

#### Overview
At this approach all the work can be done on the Ansible host if script will download extracted syslog file to the Ansible host.

Current loganalyzer implementation description:
https://github.com/Azure/SONiC/wiki/LogAnalyzer

#### Development
Module "loganalyzer.py" with class "Loganalyzer". Class is intendent to do:
- Add start marker to the DUT syslog
- Extract DUT syslog based on the markers. Download extracted logs to the Ansible host
- Perform analize of downloaded syslog and return the result

- Configure regular expressions which will be used to found matches in the logs

"Loganalyzer" class interface:
- __init__(ansible_host: ansible_host)
- load_common_config() - Load regular expressions from common configuration files: match, expect and ignore which are located in some configuration directory or with current module. Clear previous configured match, expect and ignore. Save loaded configuration.
- parse_regexp_file(file_path) - read and parse regular expressions from specified file. Return list of strings of defined regular expressions.
- run_cmd(callable, *args, **kwargs) - call function and analyze DUT logs, return the same result as "analyze" function
- init() - Add start marker to the DUT syslog.
- analyze() - Extract syslog and copy it to ansible host. Analyze extracted syslog files localy. Return python dictionary object.
Return example:
{"counters": {"match": 1, "expected_match": 0, "expected_missing_match": 0},
 "match_files": {"/tmp/syslog": {"match": 0, "expected_match": 32},
					"/tmp/syslog1": {"match": 0, "expected_match": 15}
						},
 "match_messages": {"/tmp/syslog1": ["Message 1", "Message 2", "Message n"],
						 "/tmp/syslog2": ["Message 1", "Message 2", "Message n"]
						 }
}
- save_full_log(file_path) - save downloaded DUT syslog logs to the Ansible host folder specified in 'path' input parameter.

Python properties with defined setter and getter:
- match - list of regular expression strings to match
- expect - list of regular expression strings to expect
- ignore - list of regular expression strings to ignore

Usage example:

	from ansible_host import ansible_host
	from loganalyzer import Loganalyzer

	def test(localhost, ansible_adhoc, testbed):
		hostname = testbed['dut']
		ans_host = ansible_host(ansible_adhoc, hostname)

		loganalyzer = Loganalyzer(ans_host)

		# If it is a need to load common search regular expressions. It will load common: match, expect, ignore regular expressions.
		loganalyzer.load_common_config()

		# Add start marker to DUT syslog
		loganalyzer.init()

		# Example: If test need specific search marker
		# Add search marker
		loganalyzer.match.append("Runtime error: can't parse mac address 'None'")

		# Example: Get current match regular expressions
		loganalyzer.match

		# Example: Remove specific match regular expression
		loganalyzer.match.remove("Runtime error: can't parse mac address 'None'")
		
		# Example: read test specific match file and add read strings to the existed match list
		loganalyzer.match.extend(loganalyzer.parse_regexp_file("PATH_TO_THE_FILE/FILE.txt"))

		# Execute test steps here...

		result = loganalyzer.analyze()
		assert result["counters"]["match"] == 0, "Failure message\n{}\n{}".format(result["counters"], result["match_messages"])
