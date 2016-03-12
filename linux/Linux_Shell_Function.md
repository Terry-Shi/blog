Linux Shell Function
====================

Basic structure
---------------
User defined function:

	function create_jail(){
	   # d is only visible to this function
	   local d=$1  
	   echo "create_jail(): d is set to $d"
	}

the *function* and *()* is optional.

How to define local variable
----------------------------
local variable_name

How to pass and handle arguments
---------------------
$$：(关于本 shell 的 PID)
$0 Shell本身的文件名
$? last command's return value
$1 first input argument
$# the number of input argument
$* all the input argument
eg:

	function execCmdForInstanceList(){	
		for x in $*;  do
			if [ "$x" != "start" -a "$x" != "stop" ]; then
				validateInstanceName $x;
				if (( "$?" == 0 )); then
					if [ "$OPERATION" == "stop" ]; then
						stopTomcat $x;
					else
						startTomcat $x;
					fi
				fi
			fi
		    shift
		done
	}





Return value
------------
return 0;
return 1;

exit a shell
------------
exit 0;


