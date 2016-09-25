.. index::
   single: Examples of Bitrix ORM using

Examples of Bitrix ORM using
============================

This section contains scripts with example of using Bitrix ORM

Select with Query object
------------------------

.. code-block:: php

   <?php
   use \Bitrix\Main\Loader;
   use \Bitrix\Main\Entity\Query;
   use \Bitrix\Crm\DealTable;

   Loader::includeModule('crm');

   $query = new Query(DealTable::getEntity());
   $query->setFilter(array(
       'STAGE_ID' => 'WON',
   ));

   $query->setSelect(array(
       'ID',
       'LEAD_DATE_CREATE' => 'LEAD_BY.DATE_CREATE',
       'CONTACT_DATE_CREATE' => 'CONTACT_BY.DATE_CREATE',
       'USER_ASSIGNED' => 'CONTACT_BY.ASSIGNED_BY_ID',
       'CLOSEDATE' => 'CLOSEDATE',
       'MONEY' => 'OPPORTUNITY',
       'CURRENCY' => 'CURRENCY_ID'
   ));

   $result = $query->exec();

   $deals = array();
   while ($arDeal = $result->fetch()) {

   }

Select with Entity object
-------------------------

.. code-block:: php

   <?php
   use \Bitrix\Main\Loader;
   use \Bitrix\Crm\DealTable;

   Loader::includeModule('crm');

   $deals = DealTable::getList([
       'filter' => [
           'ID' => $dealId
       ]
   ])->fetch();

Join with ORM
-------------

.. code-block:: php

   <?php

   use \Bitrix\Main\Loader;
   use \Bitrix\Main\Entity\Query;
   use \Bitrix\Crm\DealTable;

   Loader::includeModule('crm');

   $arResult = array();
   $query = new Query(LeadTable::getEntity());
   $query
       //JOIN for b_crm_status table
       ->registerRuntimeField(
           'STATUSES',
           array(
               'data_type' => '\\Bitrix\\Crm\\StatusTable',
               'reference' => array(
                   '=this.STATUS_ID' => 'ref.STATUS_ID',
                   '=ref.ENTITY_ID' => new SqlExpression('"STATUS"')
               ),
           )
       )
       ->setSelect(array('STATUS_NAME' => 'STATUSES.NAME'))
       ->setFilter(
           array(
               'ID' => $dealId,
           )
       )
       ->setOrder(array('DATE_CREATE' => 'ASC'));

   $result = $query->exec();

   while ($arLead = $result->fetch()) {
       $arResult[] = $arLead;
   }
