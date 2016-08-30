# pdns2cloudflare
Tool, to sync your PowerDNS configuration to CloudFlare

IT Can Copy All your existing records to CloudFlare automatically.
Also It will sync them everytime you update something on PowerDNS.

It can be usefeul if you are migrating from PDNS to CloudFlare DNS Protection to sync all your records.
And also If you only keep copy of your records there, and still uses PDNS as your master DNS server.

# INSTALL
Copy Script to /etc/powerdns/cloudflare_sync

It uses other scripts too. Download them.
Download this one: https://github.com/dominictarr/JSON.sh and copy it to the same directory.

Also Install this one:
https://github.com/joemiller/dns_compare

After That login to mysql server as root, or pdns database allowed user, use pdns database (pdns is default, use your name if you use other) and Paste this configuration:
```
CREATE TABLE `cf_sync_new` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `domain_id` int(11) DEFAULT NULL,
  `name` varchar(255) DEFAULT NULL,
  `type` varchar(10) DEFAULT NULL,
  `content` varchar(255) DEFAULT NULL,
  `ttl` int(11) DEFAULT NULL,
  `prio` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
 
 
 
CREATE TABLE `cf_sync_updated` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `domain_id` int(11) DEFAULT NULL,
  `name` varchar(255) DEFAULT NULL,
  `type` varchar(10) DEFAULT NULL,
  `content` varchar(255) DEFAULT NULL,
  `ttl` int(11) DEFAULT NULL,
  `prio` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
 
 
 
CREATE TABLE `cf_sync_deleted` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `domain_id` int(11) DEFAULT NULL,
  `name` varchar(255) DEFAULT NULL,
  `type` varchar(10) DEFAULT NULL,
  `content` varchar(255) DEFAULT NULL,
  `ttl` int(11) DEFAULT NULL,
  `prio` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
 
 
delimiter &&
 
 
create trigger cf_after_new after insert on records for each row begin insert into cf_sync_new (domain_id,name,type,content,ttl,prio) values (NEW.domain_id, NEW.name, NEW.type, NEW.content, NEW.ttl, NEW.prio);END&&
 
 
create trigger cf_after_update after update on records for each row begin IF NEW.type <> 'SOA' THEN insert into cf_sync_updated (domain_id,name,type,content,ttl,prio) values (NEW.domain_id, NEW.name, NEW.type, NEW.content, NEW.ttl, NEW.prio);END IF;END&&
 
 
create trigger cf_after_delete after delete on records for each row begin insert into cf_sync_deleted (domain_id,name,type,content,ttl,prio) values (OLD.domain_id, OLD.name, OLD.type, OLD.content, OLD.ttl, OLD.prio);END&&
 
 
delimiter ;
```

Now You should edit script file, and privide your Cloudflare login credentials there. Cloudflare API is accessible from cloudflare website.

Make sure both files in /etc/powerdns/cloudflare_sync are executable, or just exec:

```chmod +x /etc/powerdns/cloudflare_sync/*```

NOW, You can use this tool.

# USAGE:
All is explained when added --help as a command line parameter:

```

CloudFlare Synchronization Script
 
Works without any additional Options
 
 
Additional Options Available:
 
 
--help - Shows this help screen
--compare domain.com - Checks domain.com for MIS-MATCHES between our, and CloudFlare server.
--compare-all - Checks all domains for MIS-MATCHES. You can add second parameter for delay time between checks per domain
--check domain.com - Checks if domain.com is properly handled by CF servers
--flush-queue - Removes all pending operations queued in mysql, You can add second parameter with domain name to delete only queue for this domain
--show-queue - Show all pending operations stored in mysql
--add - Runs only adding new Records to CF servers
--update - Runs only updating Records on CF servers
--delete - Runs only deletion of Records in CF servers
--restore-all - Copies all PowerDNS records to sync table for resync
--restore-domain domain.com- Restore records of specified domain
--specified domain.com - Do all the tasks for one domain only, specified one, as a second parameter
```
Normal work - full sync of queued records are done when executed without any parameter.

If You would like to automatically sync your records from PDNS to CF just add script to your crontab

To do this edit /etc/crontab and add line like this (adjust crontab settings for yourself)
```0 */2   * * *   root    /etc/powerdns/cloudflare_sync/CloudFlare_sync```

# LIMITATIONS:
Tool is written for basic records. A, TXT, etc. 
It will not work properly with SRV for example. Feel free to edit record exclusion line inside script to add additional records.


