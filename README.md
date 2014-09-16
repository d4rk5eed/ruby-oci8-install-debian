ruby-oci8-install-debian
========================

### Step-by-step howto install ruby-oci8 with Oracle instant client on *debian*

This guide describes installation process of Oracle instant Client required
for [kubo/ruby-oci8](https://github.com/kubo/ruby-oci8).
It is required if gem installation is featured with error message as follows:

	---------------------------------------------------
	Error Message:
	  Set the environment variable ORACLE_HOME if Oracle Full Client.
	  Append the path of Oracle client libraries to LD_LIBRARY_PATH 
	  if Oracle Instant Client.

My configuration: *3.2.0-4-amd64 #1 SMP Debian 3.2.60-1+deb7u3 x86_64 GNU/Linux
ruby-oci8 (2.1.7), Oracle Instant Client x86_64 version 12.1.0.2.0*

This manual is applicable to other linux distribution and oracle releases.

* Fetch Oracle distribs from [this location](http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html). Oh, you should sign up on site first. Eventually you need 
3 archives:
	* **instantclient-basic-linux.x64-12.1.0.2.0.zip**
	* **instantclient-sqlplus-linux.x64-12.1.0.2.0.zip**
	* **instantclient-sdk-linux.x64-12.1.0.2.0.zip**


Unzip the files:

	unzip -qq instantclient-basic-macos.x64-12.1.0.2.0.zip
	unzip -qq instantclient-sqlplus-macos.x64-12.1.0.2.0.zip
	unzip -qq instantclient-sdk-macos.x64-12.1.0.2.0.zip


Make application dirs inside sudo:

	cd instantclient_12_1
	ORACLE_HOME=/usr/local/oracle/product/instantclient_64/12.1.0.2.0
	mkdir -p $ORACLE_HOME/bin
	mkdir -p $ORACLE_HOME/lib
	mkdir -p $ORACLE_HOME/jdbc/lib
	mkdir -p $ORACLE_HOME/rdbms/jlib
	mkdir -p $ORACLE_HOME/sqlplus/admin


Move files:

	mv ojdbc* $ORACLE_HOME/jdbc/lib/
	mv x*.jar $ORACLE_HOME/rdbms/jlib/

	# rename glogin.sql to login.sql
	mv glogin.sql $ORACLE_HOME/sqlplus/admin/login.sql

	# Move lib & sdk
	mv *.so* $ORACLE_HOME/lib/
	mv sdk $ORACLE_HOME/lib/sdk

	mv *README $ORACLE_HOME/
	mv * $ORACLE_HOME/bin/
	Setup TNS Names

	mkdir -p /usr/local/oracle/admin/network
	touch /usr/local/oracle/admin/network/tnsnames.ora


Put in your tnsnames, example:

	tnsnames.ora
	 ORADEMO=
	 (description=
	   (address_list=
	     (address = (protocol = TCP)(host = 127.0.0.1)(port = 1521))
	   )
	 (connect_data =
	   (service_name=orademo)
	 )
	)


Setup your environment

Create a file to store your oracle client environment variables with `touch ~/.oracle_client`

Add the following to it:

	export ORACLE_BASE=/usr/local/oracle
	export ORACLE_HOME=$ORACLE_BASE/product/instantclient_64/12.1.0.2.0
	export PATH=$ORACLE_HOME/bin:$PATH
	export DYLD_LIBRARY_PATH=$ORACLE_HOME/lib:$DYLD_LIBRARY_PATH
	export TNS_ADMIN=$ORACLE_BASE/admin/network
	export SQLPATH=$ORACLE_HOME/sqlplus/admin


Then run:

	echo "source ~/.oracle_client" >> ~/.bash_profile
	source ~/.bash_profile


Try gem installation and you may get:

	---------------------------------------------------
	Error Message:
	  cannot get Oracle version from sqlplus


Possibly it fails without some libraries, check it running sqlplus and you 
get shared libraries error.
For me libaio failed:

	sudo apt-get install libaio1 libaio-dev


Also gem installation can give you hint about symlinking some libraries:

	---------------------------------------------------
	Error Message:
	  Could not compile with Oracle instant client.
	  /usr/local/oracle/product/instantclient_64/12.1.0.2.0/lib/libclntsh.so could not be found.
	  You may need to make a symbolic link.
	     cd /usr/local/oracle/product/instantclient_64/12.1.0.2.0/lib
	     ln -s libclntsh.so.12.1 libclntsh.so


So do it!:

	cd /usr/local/oracle/product/instantclient_64/12.1.0.2.0/lib
	ln -s libclntsh.so.12.1 libclntsh.so


Now install gem:

	$ gem install ruby-oci8 -v '2.1.7'

	Fetching: ruby-oci8-2.1.7.gem (100%)
	Building native extensions.  This could take a while...
	Successfully installed ruby-oci8-2.1.7
	Parsing documentation for ruby-oci8-2.1.7
	Installing ri documentation for ruby-oci8-2.1.7
	Done installing documentation for ruby-oci8 after 3 seconds
	1 gem installed

Wow! We did it

-----------------------------------------
Source links: 
* A huge part of man is taken from http://blog.codiez.co.za/2013/09/setup-oracle-instant-client-ruby-oci8-gem-mac/
* http://stackoverflow.com/questions/17712501/cannot-install-ruby-oci8-on-ubuntu-12-04lts
