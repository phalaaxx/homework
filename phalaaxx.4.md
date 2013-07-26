8. Write a bash script which will parse a text file with the following format
-----------------------------------------------------------------------------

	64.31.49.26|2013-07-24-06:51:01|Blocked by pesho
	108.171.243.146|2013-07-24-08:17:01|Blocked by niki
	183.13.241.95|2013-07-24-11:51:00|Blocked by mm
	183.240.177.169|2013-07-24-11:51:01|Blocked by joro
	112.111.172.106|2013-07-24-13:00:00|Blocked by toni
	220.113.166.194|2013-07-24-13:00:01|Blocked by pesho
	82.59.74.82|2013-07-24-13:51:00|Blocked by pesho
	103.6.237.126|2013-07-24-16:51:00|Blocked by misho
	111.250.97.231|2013-07-24-18:34:00|Blocked by lubo
	209.208.27.41|2013-07-24-18:34:00|Blocked by vlado
	201.184.44.184|2013-07-24-18:51:01|Blocked by joro
	79.19.133.36|2013-07-24-20:00:00|Blocked by toni
	
	The script has to output only the lines that have dates older then 14 days. Sort the output by date. 


За решаване на задачата създаваме файл parser.sh със следното съдържание:

	#!/bin/bash
	
	# Date 14 days ago (14 days = 1209600 seconds)
	PastDate=$( date +"%Y-%m-%d-%H:%M:%S" --date="@$(( $(date +%s) - 1209600 ))" )
	
	# read stdin
	while read line
	do      
		LineDate=$(echo $line | cut -d\| -f2)
		if [[ $LineDate < $PastDate ]]
		then    
			echo $line
		fi      
	done | sort -t \| -k 2,3


Ако данните са във файл input.txt, скрипта може да се извиква по един от следните два начина:

	./parser.sh < input.txt
	cat input.txt | ./parser.sh
