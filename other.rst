.. index::
   single: Other

Other
=====

This section contains various recipes thant not fit to other categories

Dates on russian
----------------

.. code-block:: php

   <?php
   setlocale(LC_ALL, 'ru_RU.UTF-8');
   strftime('%d %B %Y', strtotime("+1 day"))

Phinx config
------------

.. code-block:: php

   <?php
   define("NOT_CHECK_PERMISSIONS", true);
   define("NO_AGENT_CHECK", true);
   $GLOBALS["DBType"] = 'mysql';
   $_SERVER["DOCUMENT_ROOT"] = realpath(__DIR__ . '/..');
   include($_SERVER["DOCUMENT_ROOT"] . "/bitrix/modules/main/include/prolog_before.php");
   // manual saving of DB resource
   global $DB;
   $app = \Bitrix\Main\Application::getInstance();
   $con = $app->getConnection();
   $DB->db_Conn = $con->getResource();
   // "authorizing" as admin
   $_SESSION["SESS_AUTH"]["USER_ID"] = 1;


   $config = include realpath(__DIR__ . '/../bitrix/.settings.php');

   return array(
       "paths" => array(
           "migrations" => realpath(__DIR__ . '/migrations/')
       ),
       "environments" => array(
           "default_migration_table" => "phinxlog",
           "default_database" => "dev",
           "dev" => array(
               "adapter" => "mysql",
               "host" => $config['connections']['value']['default']['host'],
               "name" => $config['connections']['value']['default']['database'],
               "user" => $config['connections']['value']['default']['login'],
               "pass" => $config['connections']['value']['default']['password']
           )
       )
   );

Errors in bitrix core
---------------------

Get last bitrix error

.. code-block:: php

   <?php
   $error = $APPLICATION->GetException();
   echo $error->GetString();

Examples of DateTime class
--------------------------

.. code-block:: php

   <?php
   $curTime = time();
   //get current day of week
   $curDayOfWeek = date("l", $curTime);

   //date calculations
   $intervalStartDate = date("Y-m-d", strtotime('+2 days'));
   $end = new DateTime();
   $end = $end->modify('+6 weeks');

   //date intervals
   $begin = new DateTime($intervalStartDate);

   //set interval for 1 day in step
   $interval = new DateInterval('P1D');
   $dateRange = new DatePeriod($begin, $interval, $end);

   //loop on days intervals
   foreach($dateRange as $date)
   {
       $dayTimestamp = $date->getTimestamp();

       //formatting
       $printDate = $date->format("j.n.Y");
       $dayOfWeek = $date->format('l');
   }

Logging examples
----------------

Using file_put_contents
~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: php

   <?php
   $logFile = $_SERVER['DOCUMENT_ROOT'].'/local/logs/log-'.strftime("%Y-%m-%d").'.log';

   $line = strftime("%d.%m.%Y | %H:%M:%S")." | LOG MESSAGE\n";
   file_put_contents($logFile, $line, FILE_APPEND);


Using Monolog
~~~~~~~~~~~~~
.. code-block:: bash

    $ composer require monolog/monolog

.. code-block:: php

   <?php
   use Monolog\Logger;
   use Monolog\Handler\StreamHandler;
   use Monolog\Formatter\LineFormatter;

   $stream = new StreamHandler('logs/solutions.log');
   $formatter = new LineFormatter("[%datetime%] %message%\n");
   $stream->setFormatter($formatter);

   $log = new Logger('logger_name');
   $log->pushHandler($stream);

   $log->addDebug('LOG MESSAGE');

Update 1C exchange module error
-------------------------------

.. code-block:: php

   COption::SetOptionString("catalog", "DEFAULT_SKIP_SOURCE_CHECK", "Y");
   COption::SetOptionString("sale", "secure_1c_exchange", "N");

Reset module files
------------------
Head to ``/bitrix/admin/update_system.php?lang=ru&BX_SUPPORT_MODE=Y`` page and find 'System area' block