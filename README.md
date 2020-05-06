# mailpiler-recovery
Public Information for the  commercial / non-free eXtremeSHOK.com Piler / Mailpiler Archive Recovery

## eXtremeSHOK.com Piler/Mailpiler Archive Recovery
## Copyright (c) Adrian Jon Kriel :: admin@extremeshok.com
```
This software is furnished under a license and may be used and copied only in accordance with the terms of such license and with the inclusion of the above copyright notice. This software or any other copies thereof may not be provided or otherwise made available to any other person or entity. No title to and ownership of the software is  hereby transferred. You may not reverse engineer, decompile, defeat license encryption mechanisms, or disassemble this software product or software product license. eXtremeSHOK.com may terminate this license if you do not comply with any of the terms and conditions set forth in our end user license agreement (EULA). In such event, licensee agrees to return licensor or destroy all copies of software upon termination of the license. This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

## Recommended Guide for Recovery with De-duplication (Recovery and rebuild of Piler/Mailpiler Archive)

### Stage 1 (Recover)
```
xshok-piler-archive-recovery.sh.x -o /datastore/export -w /var/xshok/work_dir --recover --continue
```

### Stage 2 (Process)
```
xshok-piler-archive-recovery.sh.x -o /datastore/export -w /var/xshok/work_dir --process --continue
```
### Stage 3 (Import)
Import the '/datastore/export' directory into your new Piler/Mailpiler installation --process --continue

## Usage
Always run this program as the root user, this is due to the various OpenSSL and permission requirements needed.
Run the program on the piler host machine.
Launch the program via "screen -R -D" or "tmux", to prevent the console being closed and the program stopping. ([crtl + a] and then [ctrl + d]) will detach the screen, the command "screen -R -D" will re-attach the console.

## Requires
GNU Bash 4, MySQL (command line client), OPENSSL, zlib-flat (provided by the qpdf package)
If zlib-flate is not installed the program will attempt to use OpenSSL for zlib if it is a zlib compatible release

## Permissions/Ownership:
chmod 775 xshok-piler-archive-recovery.sh.x
chown root:root xshok-piler-archive-recovery.sh.x

## Help
For any problems, issues and suggestions, contract work, etc, please contact us via admin@eXtremeSHOK.com

## About the Author
Full Stack Dev-Ops and Senior Systems Engineer
20+ years Experience
Managing over 1000 servers via idempotent configuration management for various clients
BSc in Computing and Information Systems
SAP Certified Business Intelligence Architect

## Assumptions/Notes
The piler archive data on the disk and is not going to change for the entire duration of the program.
DO NOT CHANGE / MOVE / ALTER the piler archive on the disk in any way.
The piler database tables are not used, we do create 2 tables to aid in attachment de-duplication recovery in the piler database.
All configuration options are set on the command line and via the piler.conf (userconf)
This program is non destructive to the piler archive and piler database, ie it will never delete any files inside the piler archive and the piler database tables.
DO NOT set the export and work dirs to the piler archive dir.
DO NOT set the export and work dirs to sub-dirs of the piler archive dir.
Tested on CentOS and Debian, a live in production Piler Archive of 5.5million mails was used for testing.
The program runs with multiple jobs/threads/sub-processes. The program will manage and keep them running. It was designed to be as fast as possible while ensuring data integrity.
The MySQL attachment recovery tables are created with MyISAM as this is the fastest for updating/writing millions of records. The index generation with InnoDB is simply too slow for our use.
All MySQL database queries will be retried 50 times, if there is a database failure, ie due to table/row locking.
Emails are saved to the export folder in the same multi-path arrangement as the piler store, this was done to prevent issues with folder/file limits and to have consistency with the piler archive.

## Note about attachment de-duplication recovery
I have done my best (processed over 100 million emails) to ensure that I have the most viable method for finding unique filenames and their associated file sizes where possible.
Due to emails being created with various mail clients, the data differs across the various clients, this was an extremely tedious task with sampling and testing as many messages as possible.
The word dir "attachment_recovery_map.unknown" file will list all not-found/unknown filename records and their associated message id. This is to allow further improvements of the script.
Some attachment names will always be unknown as the email client did not add any headers/lines containing the attachment name and/or file size or anything which can be considered uniquely identifiable.

## Optional Development
This script can be enhanced to connect to a live piler database and then provide a highly efficient way to decrypt and export all the piler messages.
Ideal for backup, migrations, rebuilding the archive, mass decrypting the archive or changing from encrypted to decrypted and decrypted to encrypted.
Adding skipped attachments into the recovered mails, ie add the attachments to rescued mails when the "--skip" option was used.
Please contact us if this is required.


# Guide
Calling the program with the "--help" or "-h" will display the help and available options

Database (MySQL) and Encryption settings are read from the piler config (piler.conf) file aka userconfig.

The program will search for the piler config in the following locations:  /etc/mailpiler/piler.conf, /etc/piler/piler.conf, /etc/piler.conf, /usr/local/etc/piler.conf

You can specify the userconfig location directly with the option: "--usercfg /usercfg/file" or "-u /usercfg/file"

If Encryption is enabled, the program will decrypt the mails

To enable verbose (developer/debug) output, I do not recommend this as it can slow the script down, use the option: "--verbose" or "-v"

If you want to log all the output, use the option: "--log /log/file" or "-l /log/file"

If you would like to suppress the output to the terminal, use the option: "--quiet" or "-q"

An example use case would be to enable verbose (debug) and have all the output written to the log file, and suppress output to the terminal, use the options "--log /log/file --quiet"

To simulate/test, pretty much will just navigate the archive directory, test the initial settings and not actually do anything, use the option: "--dry" or "-d"

#### Jobs
This program was designed to run multiple concurrent jobs/processes/sub-processes. The default is 1 job per vcpu (vpu = cores + ht).

#### Output / Export / Recovery Directory
"--output /output/path" or "-o /output/path" is always required. This is the directory to place the recovered messages.

#### Temporary Directory
The script will automatically set the temporary directory to /dev/shm/xshok_temp (memory) if this location is not available it will fall back to /tmp/xshok_tmp (disk).
Optionally the temp dir can be overridden with the following option: "-temp /temp/path" or "-t /temp/path"

#### Work Directory
The script will automatically place the programs working files into the work dir, this defaults to the directory specified in "--output /output/path"
Optionally the work dir can be overridden with the following option: "-work /work/path" or "-w /work/path"

#### Limiting the number/amount of recovered messages
This is ideal for testing, to recover messages in small batches or to manually resume a non-continued recovery.
eg. to only recover 1000 mails, use the following option: "--number 1000" or "-n 1000"
To start the recovery at position 4001, ie. skip the first 4000 mails, use the following option: "--ignore 4000" or "-i 4000"

#### Skipping Attachments
Piler removes attachments from emails and then de-duplicates them.
By default all found not de-duplicated/missing attachments are automatically added to the message during the mail recovery.
Using the following option: "--skip" or "-s"
Allows the program to skip the re-adding/embedding of the removed attachments into the recovered mails.
This is ideal if you need to export a large number for emails quickly and do not require the attachments or want you want to save disc space.

#### Continue
This is basically a safety net, it allows you to resume a crashed/failed/terminated recovery operation.
If the continue option is not used, when the program is run a second time / restarted it will zero the database de-duplication recovery tables, working files and start the recovery from scratch.
A continue counter is saved into the workdir as the file "continue.tmp", the counter is saved every 50 messages and when [ctrl + c] is used the counter is saved at the last successful message.
To enable, use the following option: "--continue" or "-c", this will prevent the database from being emptied and the working files being zeroed

#### Xisting / skip eXisting
The default is to process all mails for recovery and overwrite any mails in the output direcory.
Skip messages which are already located in the output dir, ie they could have already been recovered, it will completely skip all processing for the message.
To enable, use the following option "--xisting" or "-x"

#### Recover and Export (attachment de-duplication recovery)
The option: "--recover" or "-r" enables the de-duplication recovery operations, ie checking and scanning. this is the first pass of a 2 pass de-duplication recovery.
This has to be run on the entire archive to be valid, as de-duplication recovery needs to know every single messages attachments in the archive which is then used to identify unique attachments.
By default the "--recover" option will create optimised MySQL tables ("xshok_piler_archive_recovery_mails_found" and "xshok_piler_archive_recovery_missing") in the piler database, which stores all the found and missing attachment data for every message and attachment in the entire archive.
These tables are updated via the MySQL command line during processing in real-time, we have full deadlock protection and retries for every failed sql transaction. If a sql query still fails after 50 retries the query will be saved into the work dir as a file called "attachment_recovery_limit.sql" which can be imported and/or fixed manually.

The "--export" or "-e" will stop MySQL the MySQL queries being run, and instead save all these queries into the work dir in 2 files called "attachment_recovery_missing.sql" and "attachment_recovery_found.sql"
These 2 files can be imported later. This was done for instances where a MySQL server is not available or for debugging.

#### Process de-duplicated attachments
After the entire archive has been recovered with the "--recover" option.
You will need to re-run the program with the same options, except the "--recover" or "-r" will be replaced with "--process" or "-p"
This causes the missing de-duplicated attachments to be added back into the recovered mails.
I will release an update which enables this once I am confident it is 100% reliable and has no issues/bugs.


# Usage ########################

#### Recovering mails with the attachments
```
./xshok-piler-archive-recovery.sh.x -o /datastore/export
```

#### Recovering mails without attachments
```
./xshok-piler-archive-recovery.sh.x -o /datastore/export -s
```

#### Recovering mails with the attachments and preparing for de-duplication recovery with continue and skip eXisting
```
./xshok-piler-archive-recovery.sh.x -o /datastore/export -r -c -x
```

#### Recovering a certain number (1000) of mails without attachments
```
./xshok-piler-archive-recovery.sh.x -o /datastore/export -m 1000 -s
```

#### Resuming a recovery task when you have forgotten to use the "-c" or "--continue" option
```
./xshok-piler-archive-recovery.sh.x -o /datastore/export -c
```
Another alternative is to just re-run the command with the "-c" or "--continue" and "-x" or "--xisting" options this will skip any mails which have already been recovered and will make sure a record is not processed 2x if they already exist in the output directory.
```
./xshok-piler-archive-recovery.sh.x -o /datastore/export -c -x
```

## Suggested Initial Command to test the system
* After the command has run, navigate to the output dir (/datastore/test) and check the sub dirs for the messages, you can view them in any email cleint or via a text editor.
* Verifiy the sql files.
* if you encounter any failures or issues, which you are unable to recify, please contact us

* (-o) output dir /datastore/test
* (-w) working dir /datastore/test
* (-e) export the SQL query
* (-r) enable attachment de-duplication recovery
* (-n) rescue 100 mails
* (-v) be verbose
```
./xshok-piler-archive-recovery.sh.x -o /datastore/test -w /datastore/test -e -r -n 100 -v
```

## Suggested Piler Recovery Command
* After the command has run, navigate to the output dir and check the sub dirs for the messages, you can view them in any email cleint or via a text editor.
* Verifiy the sql files.
* if you encounter any failures or issues, which you are unable to recify, please contact us.
* MAKE SURE THE OUTPUT DIR IS DIFFERENT FROM THE PREVIOUS TEST COMMAND

* (-o) output directory /datastore/export dir
* (-w) working directory /var/xshok/work_dir
* (--recover) enable attachment de-duplication recovery
* (--continue) enable continue, in-case you need to restart the command
```
./xshok-piler-archive-recovery.sh.x -o /datastore/export -w /var/xshok/work_dir --recover --continue
```

## Suggested Piler Post Process (Processing) Command
* After the Piler Recovery Command has completed with the -r (enable attachment de-duplication recovery), runt he command to add the missing (de-duplicated) attachments.
* if you encounter any failures or issues, which you are unable to recify, please contact us.
* MAKE SURE THE OUTPUT DIR IS SAME AS THE ONE USED FOR THE RECOVERY COMMAND

* (-o) output directory /datastore/export dir
*  (-w) working directory /var/xshok/work_dir
* (--process) enable processing mode
* (--continue) enable continue, in-case you need to restart the command
```
xshok-piler-archive-recovery.sh.x -o /datastore/export -w /var/xshok/work_dir --process --continue
```


## Real Time Statistics
The stats are calculated for every 500 messages during the recovery. eg.
```
Cnt: 12500 | Msg: 676 | Ign: 11823 | Time: 0:00:00:40 | Rate: 33.33 m/s | Avg: 16.90 m/s | 1014.0 m/m | 60840 m/h | 1460160 m/d
```
* Cnt: current mail counter (processed mail)
* Msg: current number of recovered mails
* Ign: current number of ignored mails
* Time: total runtime of the program since it was started
* Rate: number of mails per 1 second which have been processed/recovered in the last 500 mails
* Avg: average amount of mails per 1 second/minute/hour/day(24hours) for all the processed/recovered mails since the program was started


# Current Version Help output
```
################################################################################
 eXtremeSHOK.com Piler/Mailpiler Archive Recovery
 Version: v4.0.5 (6 May 2020)
 Copyright (c) Adrian Jon Kriel :: admin@extremeshok.com
################################################################################

-h, --help	Display this program's help and usage information

--version	Output programs version and date information

-d, --dry\Dry run, will not actually do anything

-v, --verbose	More verbose Output

-q, --quiet	Quiet Programs Output

-j, --jobs XX	Optional: Maximum simultaneous Jobs to run, defaults to number of cpu cores (vcpu)

-l, --log /log/file	Optional: Set a logfile to enable logging, defaults to disabled

-k, --key /path/file	Optional: Location of the key file

-a, --archive /archive/dir	Optional: Path to the archive directory / piler store directory, defaults to the queuedir in piler config

-t, --temp /temp/path	Optional: Set a temporary directory, defaults to /dev/shm/xshok_temp, /run/shm/xshok_temp, /tmp/xshok_temp

-w, --work /work/path	Optional: Used for the continue and recover options: defaults to /output/path

-u, --config /piler/config	Optional: Location of the Piler config, defaults to /etc/mailpiler/piler.conf, /etc/piler/piler.conf, /etc/piler.conf, /usr/local/etc/piler.conf

-e, --export	Optional: Export the SQL queries to files instead of running them on the MySQL server, only used with --recover option

==== Filtering ====

-c, --continue	Optional: Continue processing after the last number of processed mails are skipped, ignores from. Note: Advisable to enable when the the recover option is enabled

-x, --xisting	Optional: skip eXisting messages, ie. Skip messages which are already located in the output dir, ie they could have already been recovered, it will completely skip all processing for the message

-s, --skip	Optional: Skip attachments, ie. Attachments will not be added to the emails. Do this if you only want the message bodies.

-n, --number XX	Optional: Maximum number of mails to process, defaults to all

-i, --ignore XX	Optional: Ignore XX number of mails for processing ie. skip the first XX mails

==== Commands ====

-o, --output /output/path	Required: Set the directory to place the recovered messages

-r, --recover	Optional: Enable Processing required to recover de-duplicated attachments, requires mysql database access, credentials taken from the piler config

-p, --process	Process the missing / de-duplicated attachments and add them to the recovered emails

NOTE: ONLY PROCESSS (--process) FOR MISSING/DE-DUPLCIATED ATTACHMENTS ONCE ALL THE EMAILS HAVE BEEN RECOVERED WITH THE --recover OPTION

==== Recommended Guide for Recovery with De-duplication (Recovery and rebuild of Piler/Mailpiler Archive) ====

Stage 1 (Recover)	xshok-piler-archive-recovery.sh.x -o /datastore/export -w /var/xshok/work_dir --recover --continue

Stage 2 (Process)	xshok-piler-archive-recovery.sh.x -o /datastore/export -w /var/xshok/work_dir --process --continue

Stage 3 (Import)	Import the '/datastore/export' directory into your new Piler/Mailpiler installation --process --continue

################################################################################
```
