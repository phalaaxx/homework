3. On grizzly, parse /var/spool/mail/root and give me the timestamps of every e-mail that has 'SECURITY information' in its subject.
------------------------------------------------------------------------------------------------------------------------------------

За решаване на задачата използваме awk. Търсим ред, който съдържа "SECURITY information", и когато намерим такъв - взимаме следващите два реда. Търсената дата и час е в последния от трите реда. Пропускаме първата колона, тъй като тя съдържа само текст "Date:" и показваме всичко останало. Решението изглежда по следния начин:

	awk '/SECURITY information/ {getline; getline; $1=""; print}' /var/spool/mail/root


Забележка: Необходими са root права за четене на посочения файл.
