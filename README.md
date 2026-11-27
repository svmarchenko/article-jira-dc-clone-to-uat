# Cloning Production Jira Data Center to a Test Server

Bash scripts that automatically clone Jira Data Center (DC) from PROD to UAT: files, the PostgreSQL database, restoring test-environment settings, and starting reindexing from a snapshot.

Detailed description of the approach

## What it does

- Copies Jira files from PROD to UAT (`rsync`, without stopping PROD)
- Dumps the PROD database (`pg_dump`) and restores it on DB UAT (`pg_restore`)
- Saves UAT-specific settings from the old UAT before cloning and restores them afterwards (integrations, Crowd, `baseurl`, Jira color)
- Patches configuration files for UAT (`crowd.properties`, `setenv.sh`, `server.xml`, `dbconfig.xml`)
- Uses an index snapshot from PROD to speed up the first reindexing
- Writes dated logs and checkpoints: after an error, the process resumes from the failed step

## My environment

- OS: Linux
- Jira Software DC 10
- Crowd DC
- PostgreSQL
- Java

For a different environment, the scripts will most likely need adjustments.

## Structure

```text
├── backup/
├── completed/
├── logs/
├── result/
│   └── UAT_tbl_dump/
├── scripts/
│   ├── chg_owner_UAT_db_jira.sh
│   ├── createPROD_data_rsync.sh
│   ├── createPROD_db_dump.sh
│   ├── createUAT_tbl_dump.sh
│   ├── drop_create_UAT_db_jira_empty.sh
│   ├── patch_UAT_file.sh
│   ├── patch_UAT_tbl_db.sh
│   ├── prepare_UAT_index_file.sh
│   ├── restore_UAT_db_jira.sh
│   ├── rsync_jira.exclude
│   ├── start_UAT_reindex.sh
│   └── stop_UAT_jira.sh
├── jconfig.env.example
└── startclone.sh
```

## Prerequisites

1. SSH key-based access from Jira UAT to Jira PROD and to DB UAT, with no interactive password prompts
2. A `syncuser` account on DB UAT
3. A Personal Access Token (PAT) of a service user, used to check Jira readiness and trigger reindexing
4. Network access: SSH (22) and PostgreSQL (`6432`) between the hosts
5. A filled-in `jconfig.env` (copy it from `jconfig.env.example`)
6. An already configured test Jira. If you don't have one, skip the reconfiguration scripts and adjust the settings manually on the first run

## Usage

Run on Jira UAT as `root`:

```bash
cp jconfig.env.example jconfig.env
# fill in jconfig.env
./startclone.sh
```

Helper scripts should not be run manually. To repeat a step, delete its checkpoint `completed/<script_name>.sh.done` and run `startclone.sh` again.

## Important

- **Review the scripts before running them.** Some steps may be critical for your environment (replacing the database, deleting data, changing the owner).
- The scripts drop and recreate the Jira database on DB UAT. Run them against a test server only.
- Not all variables are moved to `jconfig.env`; some values need to be checked directly in the scripts.
- Because PROD is not stopped, differences accumulate between the start and the end of synchronization. `rsync` errors 23 and 24 are ignored for this reason.
- Do not commit tokens or passwords to the repository.

