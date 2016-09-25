.. index::
   single: Tasks with cron utility

Tasks with cron utility
=======================

This section contains recipes for using cron

Agents on cron
--------------
Example of file that runs agents and events on cron. Place it in ``local/php_interface`` folder.

.. code-block:: php

   <?php
   $_SERVER['DOCUMENT_ROOT'] = realpath(dirname(__FILE__) . '/../..');
   $DOCUMENT_ROOT = $_SERVER['DOCUMENT_ROOT'];

   define('NO_KEEP_STATISTIC', true);
   define('NOT_CHECK_PERMISSIONS', true);
   define('BX_NO_ACCELERATOR_RESET', true);
   define('BX_CRONTAB_SUPPORT', false);

   require($_SERVER['DOCUMENT_ROOT'] . '/bitrix/modules/main/include/prolog_before.php');
   if ('N' != COption::GetOptionString('main', 'agents_use_crontab')) {
       COption::SetOptionString('main', 'agents_use_crontab', 'N');
   }
   if ('N' != COption::GetOptionString('main', 'check_agents')) {
       COption::SetOptionString('main', 'check_agents', 'N');
   }

   @set_time_limit(0);

   CAgent::CheckAgents();
   CEvent::CheckEvents();

   if (CModule::IncludeModule('subscribe')) {
       $cPosting = new CPosting;
       $cPosting->AutoSend();
   }

