.. index::
   single: Recipes for work with orders

Recipes for work with orders
============================

This section contains scripts for common tasks often found in work with orders

Get order props
---------------

.. code-block:: php

   <?php
   \Bitrix\Main\Loader::includeModule('sale');

   //get needed order properties
   $db_vals = CSaleOrderPropsValue::GetList(
       array('SORT' => 'ASC'),
       array(
           'ORDER_ID' => $ORDER_ID,
           'CODE' => array(
               'PROP_CODE',
               'PROP_CODE_1',
           ),
       ),
       false,
       false,
       array(
           'VALUE',
           'CODE'
       )
   );
   while ($arVals = $db_vals->Fetch()) {

   }

Copy order
----------
This function shows how you can copy order by it's ID. This function do NOT copies basket properties

.. code-block:: php

   <?php
   /**
    * Make copy of order with $orderId. Increments value of COPIED_ORDERS_COUNT order property
    * @param $orderId
    */
   function doCopyOrder($orderId)
   {
       \Bitrix\Main\Loader::includeModule('sale');

       $arOrder = CSaleOrder::GetByID($orderId);

       $newOrderPrice = $arOrder['PRICE'];

       $arNewOrderFields = array(
           'LID' => $arOrder['LID'],
           'PERSON_TYPE_ID' => $arOrder['PERSON_TYPE_ID'],
           'PAYED' => 'N',
           'CANCELED' => 'N',
           'STATUS_ID' => 'N',
           'EMP_STATUS_ID' => 1,
           'PRICE_DELIVERY' => $arOrder['PRICE_DELIVERY'],
           'ALLOW_DELIVERY' => 'N',
           'PRICE' => round($newOrderPrice, 0, PHP_ROUND_HALF_DOWN),
           'CURRENCY' => $arOrder['CURRENCY'],
           'DISCOUNT_VALUE' => $arOrder['DISCOUNT_VALUE'],
           'USER_ID' => $arOrder['USER_ID'],
           'PAY_SYSTEM_ID' => $arOrder['PAY_SYSTEM_ID'],
           'DELIVERY_ID' => $arOrder['DELIVERY_ID'],
           'USER_DESCRIPTION' => $arOrder['USER_DESCRIPTION'],
           'ADDITIONAL_INFO' => $arOrder['ADDITIONAL_INFO'],
           'COMMENTS' => 'Order copy',
           'TAX_VALUE' => $arOrder['TAX_VALUE'],
           'AFFILIATE_ID' => $arOrder['AFFILIATE_ID'],
       );

       $newOrderId = CSaleOrder::Add($arNewOrderFields);

       if ($newOrderId) {
           //copy needed order properties
           $db_vals = CSaleOrderPropsValue::GetList(
               array('SORT' => 'ASC'),
               array('ORDER_ID' => $orderId),
               false,
               false,
               array()
           );
           while ($arVals = $db_vals->Fetch()) {
               unset($arVals['ID']);
               $arVals['ORDER_ID'] = $newOrderId;

               CSaleOrderPropsValue::Add($arVals);
           }

           //copy order basket items
           $dbBasket = CSaleBasket::GetList(
               array('ID' => 'ASC'),
               array('ORDER_ID' => $orderId),
               false,
               false,
               array(
                   'SET_PARENT_ID', 'TYPE', 'ID',
                   'PRODUCT_ID', 'PRODUCT_PRICE_ID', 'PRICE', 'CURRENCY', 'WEIGHT', 'QUANTITY', 'LID',
                   'NAME', 'CALLBACK_FUNC', 'MODULE', 'NOTES', 'PRODUCT_PROVIDER_CLASS', 'CANCEL_CALLBACK_FUNC',
                   'ORDER_CALLBACK_FUNC', 'PAY_CALLBACK_FUNC', 'DETAIL_PAGE_URL', 'CATALOG_XML_ID', 'PRODUCT_XML_ID',
                   'VAT_RATE'
               )
           );

           $item = new \CSaleBasket;
           while ($arBasket = $dbBasket->Fetch()) {
               if (\CSaleBasketHelper::isSetItem($arBasket)) {
                   continue;
               }

               $arFields = array(
                   'ORDER_ID' => $newOrderId,
                   'PRODUCT_ID' => $arBasket['PRODUCT_ID'],
                   'PRODUCT_PRICE_ID' => $arBasket['PRODUCT_PRICE_ID'],
                   'PRICE' => $arBasket['PRICE'],
                   'CURRENCY' => $arBasket['CURRENCY'],
                   'WEIGHT' => $arBasket['WEIGHT'],
                   'QUANTITY' => $arBasket['QUANTITY'],
                   'LID' => $arBasket['LID'],
                   'NAME' => $arBasket['NAME'],
                   'CALLBACK_FUNC' => $arBasket['CALLBACK_FUNC'],
                   'MODULE' => $arBasket['MODULE'],
                   'NOTES' => $arBasket['NOTES'],
                   'PRODUCT_PROVIDER_CLASS' => $arBasket['PRODUCT_PROVIDER_CLASS'],
                   'CANCEL_CALLBACK_FUNC' => $arBasket['CANCEL_CALLBACK_FUNC'],
                   'ORDER_CALLBACK_FUNC' => $arBasket['ORDER_CALLBACK_FUNC'],
                   'PAY_CALLBACK_FUNC' => $arBasket['PAY_CALLBACK_FUNC'],
                   'DETAIL_PAGE_URL' => $arBasket['DETAIL_PAGE_URL'],
                   'CATALOG_XML_ID' => $arBasket['CATALOG_XML_ID'],
                   'PRODUCT_XML_ID' => $arBasket['PRODUCT_XML_ID'],
                   'VAT_RATE' => $arBasket['VAT_RATE'],
                   'PROPS' => array(),
                   'TYPE' => $arBasket['TYPE']
               );

               $item->Add($arFields);
           }
       }
   }

Get basket props
----------------

.. code-block:: php

   <?php
   /**
    * Function obtains all properties of a basket item
    * @param int $id Basket item Id to search for
    * @return mixed[] List of basket item properties
    */
   protected function getBasketItemProps($id)
   {
       \Bitrix\Main\Loader::includeModule('sale');

       $arProps = array();
       $dbBasketProps = CSaleBasket::GetPropsList(
           array("SORT" => "ASC"),
           array("BASKET_ID" => $id),
           false,
           false,
           array("ID", "BASKET_ID", "NAME", "VALUE", "CODE", "SORT")
       );

       if ($arBasketProps = $dbBasketProps->Fetch())
       {
           do
           {
               $arProps[] = array(
                   "NAME" => $arBasketProps["NAME"],
                   "CODE" => $arBasketProps["CODE"],
                   "VALUE" => $arBasketProps["VALUE"]
               );
           }
           while ($arBasketProps = $dbBasketProps->Fetch());
       }

       return $arProps;
   }
