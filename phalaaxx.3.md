7. List all opened files using only the /proc fs.
------------------------------------------------

	ls -l /proc/*/fd 2>/dev/null

Този вариант не е абсолютно точен, тъй като /proc/self сочи към информацията за текущия процес (в случая - ls), и в резултат на това файловете отворени от този процес ще се покажат два пъти - в /proc/<pid>/fd и /proc/self/fd. 
Втори вариант за решаване на задачата:

	find /proc -type d -regextype sed -regex "/proc/[0-9]\+/fd" -exec ls -l '{}' \; 2>/dev/null | grep -v total

За да покажем броя на отворените файлове и по двата начина:

	ls -l /proc/*/fd 2>/dev/null | grep "\->" | wc -l
	find /proc -type d -regextype sed -regex "/proc/[0-9]\+/fd" -exec ls -l '{}' \; 2>/dev/null | grep -v total | wc -l
