4. Make an Deb package for an application written in Perl, Python or Ruby. The exepected result is the package and its control files.
-------------------------------------------------------------------------------------------------------------------------------------

За създаване на пакет за python модул използваме модул python-agentx от sourceforge.net:

	http://sourceforge.net/projects/python-agentx/

Необходимо за създаването на модул за python е да имаме инсталирани следните пакети:

	apt-get install devscripts python-all-dev python-stdeb

След като бъде изтеглен модула от sf.net, изпълняваме следните команди:

	py2dsc -m 'Bozhin Zafirov <bozhin@abv.bg>' python-agentx_r7.tar.gz
	cd deb_dist/agentx-0.7
	debuild

Създадения пакет за Debian е във файл python-agentx_0.7-1_all.deb и може да бъде инсталиран по следния начин:

	dpkg -i python-agentx_0.7-1_all.deb

За проверка дали модула е правилно инсталиран използваме следната команда:

	echo "import agentx" | python

Липсата на съобщение за грешка означава, че съществува модул с име agentx и се зарежда успешно.
Резултата от изпълнението на горепосочените команди води до създаването на този пакет за Debian: [python-agentx_0.7-1_all.deb](https://github.com/phalaaxx/homework/blob/master/python-agentx_0.7-1_all.deb?raw=true "python-agentx_0.7-1_all.deb")
