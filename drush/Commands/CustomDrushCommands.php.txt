<?php

namespace Drush\Commands;

use Consolidation\OutputFormatters\StructuredData\RowsOfFields;
use Consolidation\SiteAlias\SiteAliasManagerAwareInterface;
use Consolidation\SiteAlias\SiteAliasManagerAwareTrait;
use Consolidation\SiteProcess\ProcessManagerAwareInterface;
use Consolidation\SiteProcess\ProcessManagerAwareTrait;

/**
 * General policy commands and hooks for the application.
 */
class CustomDrushCommands extends DrushCommands implements SiteAliasManagerAwareInterface, ProcessManagerAwareInterface {
  use SiteAliasManagerAwareTrait;
  use ProcessManagerAwareTrait;

  /**
   * Configuration that should be sanitized.
   *
   * @var array
   */
  protected $sanitizedConfig = [];

  /**
   * Show the database size.
   *
   * @command cdc:database:size
   *
   * @aliases cdc:ds
   *
   * @field-labels
   *   database: Database
   *   size: Size
   *
   * @return string
   *   The size of the database in megabytes.
   */
  public function databaseSize() {
    $selfRecord = $this->siteAliasManager()->getSelf();

    /** @var \Consolidation\SiteProcess\SiteProcess $process */
    $process = $this->processManager()->drush($selfRecord, 'core-status', [], [
      'fields' => 'db-name',
      'format' => 'json',
    ]);

    $process->run();
    $result = $process->getOutputAsJson();

    if (isset($result['db-name'])) {
      $db = $result['db-name'];
      $args = ["SELECT SUM(ROUND(((data_length + index_length) / 1024 / 1024), 2)) AS \"Size\" FROM information_schema.TABLES WHERE table_schema = \"$db\";"];
      $options = ['yes' => TRUE];
      $process = $this->processManager()->drush($selfRecord, 'sql:query', $args, $options);
      $process->mustRun();
      $output = trim($process->getOutput());
      return "Database {$db} size is : {$output} MB";
    }
  }

  /**
   * Show tables larger than the input size.
   *
   * @param int $size
   *   The size in megabytes of table to filter on. Defaults to 1 MB.
   * @param mixed $options
   *   The command options.
   *
   * @command cdc:table:size
   *
   * @aliases cdc:ts
   *
   * @field-labels
   *   table: Table
   *   size: Size
   *
   * @return \Consolidation\OutputFormatters\StructuredData\RowsOfFields
   *   Tables in RowsOfFields output formatter.
   */
  public function tableSize(int $size = 1, $options = ['format' => 'table']) {
    $size = $this->input()->getArgument('size') * 1024 * 1024;
    $selfRecord = $this->siteAliasManager()->getSelf();
    $args = ["SELECT table_name AS \"Tables\", ROUND(((data_length + index_length) / 1024 / 1024), 2) \"Size in MB\" FROM information_schema.TABLES WHERE table_schema = DATABASE() AND (data_length + index_length) > $size ORDER BY (data_length + index_length) DESC;"];
    $options = ['yes' => TRUE];
    $process = $this->processManager()->drush($selfRecord, 'sql:query', $args, $options);
    $process->mustRun();
    $output = $process->getOutput();

    $rows = [];

    $output = explode(PHP_EOL, $output);
    foreach ($output as $line) {
      if (!empty($line)) {
        [$table, $table_size] = explode("\t", $line);

        $rows[] = [
          'table' => $table,
          'size' => $table_size . ' MB',
        ];
      }
    }

    $data = new RowsOfFields($rows);
    $data->addRendererFunction(function ($key, $cellData) {
      if ($key == 'first') {
        return "<comment>$cellData</>";
      }

      return $cellData;
    });

    return $data;
  }

}
