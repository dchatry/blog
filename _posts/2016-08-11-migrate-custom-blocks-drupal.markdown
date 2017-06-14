---
layout: post
title: Migrate custom blocks from D6 to D7
date: 19:57 06-05-2015
headline: Move blocks from Drupal 6 to 7 in a heartbeat
taxonomy:
    category: blog
    tag: [drupal]
---
[Migrate][1] and [migrate_d2d][2] projects make it super easy to transfer content from one *Drupal* installation to another regardless of versions. Recently, the possibility was added to handle custom blocks migrations. With a minimal setup we're going to see how we can import these from a D6 to a D7 installation (all the while conserving the region they were originally setup in). 

## Requirements

Of course you will need to download and enable: 

*   [migrate][1]
*   [migrate_d2d][2]

If you have drush installed, just copy and paste this in your terminal: 

    drush dl migrate migrate_d2d && drush en migrate migrate_d2d -y
    

## Step 1 : Database definition

On your D7 installation, **update** the `settings.php` so that Drupal can interact with the D6 database (here I named it `legacy`). 

<pre class="prettyprint">
<code>
$databases = array (
  'default' => 
  array (
    'default' => 
    array (
      'database' => 'd7_database',
      'username' => 'login',
      'password' => 'password',
      'host' => 'localhost',
      'driver' => 'mysql',
      'prefix' => '',
    ),
  ),
  'legacy' => 
  array (
    'default' => 
    array (
      'database' => 'd6_database',
      'username' => 'login',
      'password' => 'password',
      'host' => 'localhost',
      'driver' => 'mysql',
      'prefix' => '',
    ),
  ),
);
</code>
</pre>

## Step 2 : Module creation 

Now **create** a custom module, let's name it `migration_from_d6` for example. Here is the content of `migration_from_d6.info`: 

    name = Migrate from Drupal 6
    description = Migrate content from Drupal 6 to Drupal 7
    core = 7.x
    package = Migration
    version = "7.x-1.0"
    core = "7.x"
    project = "migration_from_d6"
    
    dependencies[] = migrate
    dependencies[] = migrate_d2d
    
    files[] = migration_from_d6.migrate.inc
    files[] = blocks.inc
    
    
**Create** the `migration_from_d6.module` but leave it empty. The *migrations settings* are going to be defined in `migration_from_d6.migrate.inc`, create the file and paste the following inside: 

<pre>
<code>
&lt;?php

/**
 * @file
 * Declares our migrations.
 */

/**
 * Implements hook_migrate_api().
 */
function migration_from_d6_migrate_api() {
  // Define api settings and common arguments.
  $api = array(
    'api' => 2,
    'groups' => array(
      'd6' => array(
        'title' => t('D6 Migrations'),
      ),
      'migrations' => array(),
    )
  );

  $common_arguments = array(
    'source_connection' => 'legacy',
    'source_version' => 6,
    'group_name' => 'd6',
  );

  // Register the blocks migration.
  $api['migrations']['BlockMigration'] = $common_arguments + array(
    'title' => t('Blocks Migration'),
    'description' => t('Migration of blocks from Drupal 6'),
    'class_name' => 'BlockMigration',
    'machine_name' => 'Block',
    'source_type' => 'block',
    'destination_type' => 'block',
  );

  return $api;
}
</code>
</pre> 

Now that we have *registered* our migration, we need to define the `BlockMigration` class used to import our custom blocks. **Create** a `blocks.inc` file and copy/paste this: 

<pre class="prettyprint">
<code>
class BlockMigration extends DrupalCustomBlock6Migration {
  public function __construct(array $arguments) {
    parent::__construct($arguments);
  }

 /**
   * Implementation of MigrateDestination::complete().
   */
  public function complete($block, stdClass $row) {
    // Rebuild block table.
    block_flush_caches();

    // Retrieve the source region.
    $region = Database::getConnection('default', 'legacy')->query('SELECT region FROM {blocks} WHERE bid = :bid', array('bid' => $row->bid))->fetchField();

    // If a region is set, update created block.
    if(!empty($region)) {
      db_update('block')
        ->fields(array(
          'region' => $region,
          'status' => 1,
        ))
        ->condition('delta', $block->bid)
        ->execute();
    }
  }
}
</code>
</pre> 

Here we **extend** the existing `DrupalCustomBlock6Migration` class (provided by `migrate_d2d`) and we tell `migrate` that each time a block is *imported* into our D7 installation (`complete` method), we check if the block had a region and **update** the newly created block if that's the case. 

## Step 3 : Migration

Okay we are now ready to import our custom blocks! First **enable** your custom migration module. 

Then, **go to** `admin/content/migrate/configure` and **click** on "Register statically defined classes", this will force `migrate` to register the `BlockMigration` class we just created. **Click** on your `D6 Migrations`, you should see the task `BlockMigration` and the number of blocks left to migrate. 

**Check** the row for `BlockMigration`, **select** "Import" and **launch** the migration.

Everything should go smoothly and you should soon see your custom blocks imported in their respective regions.

 [1]: http://www.drupal.org/project/migrate
 [2]: http://www.drupal.org/project/migrate_d2d