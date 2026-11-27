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
