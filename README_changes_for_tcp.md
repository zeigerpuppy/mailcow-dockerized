# Documentation of changes required for use of TCP inter-container comms.

> no method is proposed to replace function of `mailcow-dockerapi`

## Introduction

This is a list of changes made to allow compatability with TCP (and retain compatability with sockets0
All changes attempt to use the variable `${CONNECT_METHOD}="tcp|socket"`

The default is set as `socket` so all previous docker builds should execute as before.
The choice of tcp/socket has been added to `generate_config.sh`

Hopefully merging will be relatively painless...

Change in build files are generally of the form:

originally tried with this method:

```bash
if [[ "$CONNECT_METHOD" == "socket" ]]; then
	<old code>
else
	<new, TCP compatible code>
fi
```

however, a simpler method is just to have a general replacement in the mysql connect string:

```
DBOST="mysql"
DBCONN="-h ${DBHOST}"
OR
DBCONN="--socket=/var/run/mysqld/mysqld.sock"
```

## Files modified:

`data/Dockerfiles/acme/docker-entrypoint.sh`
  - [x] 3 changes made with generic `${DBCONN}` variable

`data/Dockerfiles/dovecot/docker-entrypoint.sh`
  - [x] 2 blocks changed for `--socket=...` to `${DBCONN}`
  - [x] host config generalised for dovecot config files creation

`data/Dockerfiles/dovecot/rspamd-pipe-ham`
`data/Dockerfiles/dovecot/rspamd-pipe-spam`
`data/Dockerfiles/postfix/rspamd-pipe-ham` 
`data/Dockerfiles/postfix/rspamd-pipe-spam` 
  - [ ] change will be needed for `curl --unix-socket` options in these files
  - not sure what appropriate change is yet
 
`/Dockerfiles/phpfpm/docker-entrypoint.sh` 
  - [x] changed to generic `${DBCONN}`

`data/Dockerfiles/sogo/bootstrap-sogo.sh`
  - [x] multiple replacements of connect sting and some reparsing of URLs
  - [x] using generic `${DBCONNURL}`

`data/Dockerfiles/watchdog/watchdog.sh`   
  - [ ] another instance of `curl --unix-socket` pointing to rspamd.sock

`data/assets/mysql/docker-entrypoint.sh`
  - [x] although a socket is defined here, no change made as mysql networking is currently exposed with default config

`data/conf/rspamd/dynmaps/settings.php`
  - [x] change to `$dsn` connection settings
    - note that the old host config was already here as a comment, uncommented and wrapped in DBCONN if statement

`data/conf/rspamd/meta_exporter/pipe.php`
 - [x] same as above

`data/web/autodiscover.php`
  - [x] same as above

`data/web/inc/functions.inc.php`
  - [ ] review `curl_setopt` as is currently connecting to `rspamd.sock`

`data/web/inc/functions.mailbox.inc.php`
`data/web/json_api.php`
  - [ ] review `curl_setopt`
    - interesting that we have an option which looks like it is for TCP connection:
    ```
          $curl = curl_init();
          curl_setopt($curl, CURLOPT_UNIX_SOCKET_PATH, '/var/lib/rspamd/rspamd.sock');
          curl_setopt($curl, CURLOPT_URL,"http://rspamd/actions");
    ```
same issues in:
  - [ ] 2 instances of `curl_setopt` in `data/web/inc/functions.quarantine.inc.php`

`data/web/inc/init_db.inc.php`
`data/web/inc/prerequisites.inc.php`
  - [x] edited `$dsn` connection settings

## ToDo

- [ ] check for other instances of `mysqld.sock`
- [ ] check if we have defined the variables correctly in .php files
