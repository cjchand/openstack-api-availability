# openstack-api-availability

**Note:** Currently in "it's working" stage (aka: MVP). Caveat emptor! :)

This repo is a collection of scripts that perform atomic, read-only tasks to gauge OpenStack API availability and response time. Results are fed into statsd for reporting and analysis, with Logstash ingestion coming soon.

Also includes a Vagrantfile that deploys all the required packages and pip installs the Pythonic bits, using cron to hit your APIs once per minute, streaming glorous metrics love to your statsd server.

# Requirements

Should be inferred from the above, but to spell it out:

* Reachable, working OpenStack environment with at least one object for any of the APIs you care to test (e.g. an instance for Nova, a network for Neutron). Shared/public objects are fine. All it's doing is looking for existence of any object (and ensuring there are no exceptions thrown when it tries to look.
* Reachable, working statsd server (or anything masquerading as such)
	* ... and by extension, a carbon/Graphite setup for statsd to ship its magic to
* If using the Vagrantbox:
	* Of course, Vagrant installed on wherever you plan to run it
	* Access to standard RHEL repos and EPEL (basically, can reach the Internet)
	* Access to PiPy (for pip modules)
* When implemented, ElasticSearch server(s) to shoot Logstash outputs into, Kibana to receive dashboards

Todo's include:

* Add lock files to ensure multiple instances are not running simultaneously (and add metric for runs blocked by this)
* Adding logging and Logstash config to log results
* Adding small CRUD operations; current ones are read-only
* Remove config not needed as a result of refactoring

# To use with the provided Vagrantfile:

1. Clone the repo: `git clone git@github.com:cjchand/openstack-api-availability.git`
2. cd to the repo and review the settings in `ansible/roles/os\_api\_availability/vars/main.yml`. There are settings for things like:
	* statsd server, port, and metric prefix
	* OpenStack URLs (e.g.: auth)
	* Test user credentials, tenant, etc
3. Once set, type: `vagrant up`
4. Watch the wall of text as packages and pip modules are installed, and so on
5. Once done, type `vagrant ssh` to access the VM
6. The scripts live in `/opt/openstack-api-availability` and will be run by a cron job as soon as the server is up
7. (Optional) If you want to change the code, automation, or variables, make those changes, then type `vagrant provision` to push them to the Vagrantbox

# Limitations/Sub-optimals

This is just an MVP at this point, so it comes with some caveats that aren't in the Todo section:

* Ansible is built for CentOS/RHEL7; no Debian-based automation at this time and might not work seamlessly on prior Fedora-based OS'
* No log rotation
* No timestamps in the logs (yet, will get this soon)
* Very little defensive code
* Some of the tests are - to be brutally honest - a bit lame:
	* Swift: While it exercises the API to a certain extent, the Swift test only gets the info for the account (which does indeed include info on any containers owned by this tenant)
	* These are intentionally read-only for now, as I have seen that OpenStack can suffer a bit from DB bloat if your're doing CRUD 24/7
		* Of course, this isn't really ideal - reading stuff is one thing, being able to do CRUD is another
		* When/if this gets implemented, it there will be a flag to control whether to run in read-only or CRUD mode
	* There are not currently any reachability-/sanity-type checks, for example:
		* Ping a permanently-spawned instance
		* Read an object in a Swift container to validate its contents
		* Iterate and confirm expected networks and subnets in Neutron

# An Aside

Note: The general flow and logic parts were cloned from [https://github.com/rakesh-patnaik/nagios-openstack-monitoring.git](). I'm leaving in the "OK,CRITICAL" output for now as I iterate on the scripts. I might ultimately refactor these to be Nagios plugins and/or check\_mk local checks later.
