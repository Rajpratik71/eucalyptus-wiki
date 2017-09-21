
### Features available in iscsitarget_common.pl
parse_devstringReturns an array containing the tokens from the first argument, split on commas.

Sample:parameter: "foo,bar,baz"

returns:('foo', 'bar', 'baz')

sanitize_pathRemoves dots - '.' from its input, but only if it is more than two characters in. Checks every 3rd character after that.

Sample:

run_cmdA simple wrapper around executing commands. Returns the output from running the command. Through parameters, the sub can print output to STDERR or cause the script to exit prematurely, if the command does not execute cleanly.

Arguments:

1 => flag to tell whether to print output to stderr when the command fails2 => flag to tell whether to exit the script when command fails3 => string containing command and parameters to executeSample:parameters: true, false, "ls -ltr"

returns:

total 16

-rwxrwxr-x. 1 wesw wesw 9434 Aug 19 11:09 [iscsitarget_common.pl](http://iscsitarget_common.pl)

-rw-rw-r--. 1 wesw wesw 93 Aug 19 11:10 [test.pl](http://test.pl)

check_and_log_faultReads the output of commands and calls euca-generate-fault if it is appropriate to do so.

Arguments:

1 => command executed2 => array of strings, the output of the command (argument 1) executedSample:do_exitcalls exit system call with parameter (really useless because do_exit($foo) is the same as exit($foo))

Arguments: 1 => string to pass to exit system call

Sample:

untaintruns parameter through an arbitrary regular expression that removes text unless it looks like a command. Allows input to begin with a space, an ampersand, a colon, a hash, an \@ or any word character.

Arguments: 1 => string to munge

Sample:parameter:::&#@-/usr/bin/iscsiadmreturns:::&#@-/usr/bin/iscsiadmlookup_mpathcreates a hash representing the multipath topology by running the `multipath` command

Arguments:

noneSample:parameter:returns:lookup_ifacecreates a hash representing the iscsi interfaces by running the `iscsiadm` command

Arguments:

noneSample:parameter:returns:retry_until_truetakes a ref to a subroutine, a ref to an array of parameters and a number representing the number of times to execute the subroutine with the parameters. The subroutine will then be executed until the subroutine returns 1 or the number of executions reaches the number of retries specified by the caller.

Arguments:

1 => reference to subroutine to be executed2 => reference to array of arguments to subroutine (argument 1) that will be executed3 => number of times to execute subroutine (argument 1)Sample:parameter:returns:match_iscsi_sessioncan be used to match up a session for a connection, such as when deleting a lun.

Arguments:

1 => reference to the session hash2 => string, indicates the network device in use in for the iscsi session3 => string, indicates IP address of the iscsi target4 => string, store?Sample:parameter:returns:get_iscsi_devicegiven the network device, IP address, store and LUN, returns the device name

Arguments:

1 =>string, indicates the network device in use in for the iscsi session2 => string, indicates IP address of the iscsi target3 => string, store?4 => LUNSample:parameter:returns:rescan_all_sessionsexecutes `scsiadm -m session -R`

Arguments:

noneSample:parameter:returns:get_first_lunuses a regular expression to see which of the arguments is the first to contain "lun-" followed by one or more digits.

Arguments:

1 => array of strings to search againstSample:parameter:returns:get_mpath_deviceChecks each argument as a key in the hash returned by get_mpath_device. If the hash has a value that is not null or an empty string, it will be returned

Arguments:

1 => array of stringsSample:parameter:returns:is_null_or_emptymakes sure argument has been assigned a value other than undef and that the length of the value, as a string, is 1 or greater

Arguments:

1 =>string, the string to checkSample:parameter:returns:get_conf_iface_mapextracts the list of interfaces from the eucalyptus.conf configuration file by examining the value forSTORAGE_INTERFACES.

Arguments:

noneSample:parameter:returns:get_netdev_by_confchecks the argument to see if it is a value in the conf_iface_map hash. If it is, and the value is not undef or an empty string, it will be returned.

Arguments:

1 =>stringSample:parameter:returns:get_ifacechecks the argument and to see if it is a value in the hash returned by lookup_iface, returns the value referred to with the argument as the key

Arguments:

1 =>stringSample:parameter:returns:lookup_sessiongiven the network device, IP address, store and LUN, returns the device name

Arguments:

noneSample:parameter:returns:

*****

[[category.storage]] 
[[category.confluence]] 
