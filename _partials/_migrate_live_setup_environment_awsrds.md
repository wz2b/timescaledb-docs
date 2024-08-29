import SetupConnectionStrings from "versionContent/_partials/_migrate_live_setup_connection_strings.mdx";
import MigrationSetupDBConnectionPostgresql from "versionContent/_partials/_migrate_set_up_align_db_extensions_postgres_based.mdx";
import TuneSourceDatabaseAWSRDS from "versionContent/_partials/_migrate_live_tune_source_database_awsrds.mdx";
import MigrateSetupTargetEnvironment from "versionContent/_partials/_migrate_live_setup_environment_target_config.mdx";

## Set your connection strings

<SetupConnectionStrings />

## Align the extensions on the source and target
<Procedure>

<MigrationSetupDBConnectionPostgresql />

</Procedure>

## Tune your source database

<Procedure>

<TuneSourceDatabaseAWSRDS />

</Procedure>


## Tune the target Timescale Cloud service

<Procedure>

<MigrateSetupTargetEnvironment />

</Procedure>


[modify-parameters]: /use-timescale/:currentVersion:/configuration/customize-configuration/#modify-basic-parameters
[mst-portal]: https://portal.managed.timescale.com/login