import SetupConnectionStrings from "versionContent/_partials/_migrate_live_setup_connection_strings.mdx";
import MigrationSetupDBConnectionTimescaleDB from "versionContent/_partials/_migrate_set_up_align_db_extensions_timescaledb.mdx";
import TuneSourceDatabaseMST from "versionContent/_partials/_migrate_live_tune_source_database_mst.mdx";
import MigrateSetupTargetEnvironment from "versionContent/_partials/_migrate_live_setup_environment_target_config.mdx";

## Set your connection strings

<SetupConnectionStrings />

## Align the version of TimescaleDB on the source and target
<Procedure>

<MigrationSetupDBConnectionTimescaleDB />

</Procedure>

## Tune your source database

<Procedure>

<TuneSourceDatabaseMST />

</Procedure>

## Tune the target Timescale Cloud service

<Procedure>

<MigrateSetupTargetEnvironment />

</Procedure>


[modify-parameters]: /use-timescale/:currentVersion:/configuration/customize-configuration/#modify-basic-parameters
[mst-portal]: https://portal.managed.timescale.com/login