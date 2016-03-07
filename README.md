# Redmine on OpenShift

**CURRENT VERSION ON THIS REPO:** Redmine 3.2.0

Redmine is a flexible project management web application. Written using Ruby on Rails framework, it is cross-platform and cross-database.

Redmine is open source and released under the terms of the GNU General Public License v2 (GPL).

More information can be found at www.redmine.org

## Running on OpenShift

Create an account at https://www.openshift.com

Create a Ruby 1.9 application with MySQL 5.1

	rhc app create redmine ruby-1.9 mysql-5.1

Make a note of the username, password, and host name as you will need to use these to login to the database.

Add this upstream Redmine quickstart repo

	cd redmine
	git remote add upstream -m master git://github.com/dgacitua/openshift-redmine-quickstart.git
	git pull -s recursive -X theirs upstream master

Then push the repo upstream

	git push

That's it, you can now checkout your application at:

	http://redmine-$yournamespace.rhcloud.com


Use the following to login to your new Redmine application running on OpenShift:

	username: admin
	password: admin


## Changing the default admin password

Once your installation is complete, it is highly recommended that you change the password for the Redmine admin user - see the Change password link at:

	http://redmine-$yournamespace.rhcloud.com/my/account

## Procedure to create this Redmine instance
	
	# The Openshift application must be configured as follows:
	# Ruby 1.9 - MySQL 5.1 - No scaling
	# (there's a bug with Ruby 2.0 while deploying)

	# Begin the installation by navigating to the app runtime directory
	cd ~/app-root/runtime/repo/
	# download redmine
	wget http://www.redmine.org/releases/redmine-3.2.0.tar.gz
	tar xvzf redmine-3.2.0.tar.gz
	rm redmine-3.2.0.tar.gz

	# extract it to its new environment
	rm -rf public
	rm -rf tmp
	mv redmine-3.2.0/* ~/app-root/runtime/repo/
	rm -rf redmine-3.2.0*

	# make sure rails, mysql plugin and bundler are installed
	gem install rails mysql2
	gem install bundler -no-ri -no-rdoc

	# get the configuration files
	cd ~/app-root/runtime/repo/config
	wget —no-check-certificate https://raw.githubusercontent.com/chriswirz/openshift-redmine-3.0.1-quickstart/master/config/database.yml
	cp database.yml database.yml.openshift
	cp database.yml database.yml-1.9
	cp configuration.yml.example configuration.yml.openshift
	cd ~/app-root/runtime/repo/

	# bundle install
	bundle install --no-deployment

	# populate the database
	rake generate_secret_token
	RAILS_ENV=production rake db:migrate
	RAILS_ENV=production rake redmine:load_default_data

	# restart the app/gear
	gear stop
	gear start
