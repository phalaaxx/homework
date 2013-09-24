3. Create a configuration that will allow you to delegate the reverse(PTR) records to another server for IP range smaller then /24 for example /26. Show the named.conf and the zone file.
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


В примерното решение на задачата се използват следните начални условия:

 * network 91.230.230.0/24
 * subnet 91.230.230.224/27
 * nameservers - a.ns.deck17.com, b.ns.deck17.com


За решаването на задачата се създава файл named.conf със следното съдържание (файла не е пълен но е достатъчен за функционирането на bind9):

#### named.conf

	options {
	        directory "/var/cache/bind";
	        dnssec-validation auto;
	        auth-nxdomain no;
	        listen-on { any; };
	};

	zone "." {
	        type hint;
	        file "/etc/bind/db.root";
	};

	zone "230.230.91.in-addr.arpa" {
	        type master;
	        file "/etc/bind/zones/ptr/230.230.91.in-addr.arpa.db";
	};

	zone "224-255.230.230.91.in-addr.arpa" {
	        type master;
	        file "/etc/bind/zones/ptr/224-255.230.230.91.in-addr.arpa.db";
	};

	;
	; forward zones (like for deck17.com, not shown here)
	;



Файла (зоната) с PTR записите има следното съдържание:

#### 230.230.91.in-addr.arpa.db

	;
	; BIND data file for 230.230.91.in-addr.arpa zone
	;
	$TTL    3600
	@       IN      SOA     a.ns.deck17.com. root.delchev-lawfirm.com. (
	                        2       ; Serial
	                        3600    ; Refresh
	                        1800    ; Retry
	                        604800  ; Expire
	                        7200 )  ; Negative Cache TTL
	;
	@       IN      NS              a.ns.deck17.com.
	@       IN      NS              b.ns.deck17.com.
	
	; Sample PTR records
	$GENERATE 1-223 $                       IN      PTR     primary$.deck17.com.
	
	; Delegate 91.230.230.224/27 to a.ns.deck17.com and b.ns.deck17.com
	; Zone name is 224-255.230.230.in-addr.arpa
	224-255.230.230.91.in-addr.arpa.        IN      NS      a.ns.deck17.com.
	224-255.230.230.91.in-addr.arpa.        IN      NS      b.ns.deck17.com.
	$GENERATE 224-255 $                     IN      CNAME   $.224-255.230.230.91.in-addr.arpa.



Файла със зона 224-255.230.230.91.in-addr.arpa съдържа примерно съдържание за делегираната зона.

#### 224-255.230.230.91.in-addr.arpa.db

	;
	; BIND data file for 224-255.230.230.91.in-addr.arpa zone
	;
	$TTL    3600
	@       IN      SOA     a.ns.deck17.com. root.delchev-lawfirm.com. (
	                        2       ; Serial
	                        3600    ; Refresh
	                        1800    ; Retry
	                        604800  ; Expire
	                        7200 )  ; Negative Cache TTL
	;
	@       IN      NS              a.ns.deck17.com.
	@       IN      NS              b.ns.deck17.com.
	
	; Sample PTR records
	$GENERATE 224-255 $                       IN      PTR     secondary$.deck17.com.
