.. index::
   single: Common event handlers

Common event handlers
=====================

This section contains common Bitrix event handlers

Minimal and maximal price props
-------------------------------

.. code-block:: php

   <?php

   AddEventHandler('iblock', 'OnAfterIBlockElementAdd', 'DoIBlockAfterSave');
   AddEventHandler('iblock', 'OnAfterIBlockElementUpdate', 'DoIBlockAfterSave');
   AddEventHandler('catalog', 'OnPriceAdd', 'DoIBlockAfterSave');
   AddEventHandler('catalog', 'OnPriceUpdate', 'DoIBlockAfterSave');

   /**
    * Runs on events:
    * - OnAfterIBlockElementAdd
    * - OnAfterIBlockElementUpdate
    * - OnPriceAdd
    * - OnPriceUpdate
    *
    * Calculates min and max prices and saves them into propd
    * @param      $arg1
    * @param bool $arg2
    */
   function DoIBlockAfterSave($arg1, $arg2 = false)
   {
       $ELEMENT_ID = false;
       $IBLOCK_ID = false;
       $OFFERS_IBLOCK_ID = false;
       $OFFERS_PROPERTY_ID = false;

       //Check for catalog event
       if (is_array($arg2) && $arg2["PRODUCT_ID"] > 0) {
           //Get iblock element
           $rsPriceElement = CIBlockElement::GetList(
               array(),
               array(
                   "ID" => $arg2["PRODUCT_ID"],
               ),
               false,
               false,
               array("ID", "IBLOCK_ID")
           );
           if ($arPriceElement = $rsPriceElement->Fetch()) {
               $arCatalog = CCatalog::GetByID($arPriceElement["IBLOCK_ID"]);
               if (is_array($arCatalog)) {
                   //Check if it is offers iblock
                   if ($arCatalog["OFFERS"] == "Y") {
                       //Find product element
                       $rsElement = CIBlockElement::GetProperty(
                           $arPriceElement["IBLOCK_ID"],
                           $arPriceElement["ID"],
                           "sort",
                           "asc",
                           array("ID" => $arCatalog["SKU_PROPERTY_ID"])
                       );
                       $arElement = $rsElement->Fetch();
                       if ($arElement && $arElement["VALUE"] > 0) {
                           $ELEMENT_ID = $arElement["VALUE"];
                           $IBLOCK_ID = $arCatalog["PRODUCT_IBLOCK_ID"];
                           $OFFERS_IBLOCK_ID = $arCatalog["IBLOCK_ID"];
                           $OFFERS_PROPERTY_ID = $arCatalog["SKU_PROPERTY_ID"];
                       }
                   } //or iblock wich has offers
                   elseif ($arCatalog["OFFERS_IBLOCK_ID"] > 0) {
                       $ELEMENT_ID = $arPriceElement["ID"];
                       $IBLOCK_ID = $arPriceElement["IBLOCK_ID"];
                       $OFFERS_IBLOCK_ID = $arCatalog["OFFERS_IBLOCK_ID"];
                       $OFFERS_PROPERTY_ID = $arCatalog["OFFERS_PROPERTY_ID"];
                   } //or it's regular catalog
                   else {
                       $ELEMENT_ID = $arPriceElement["ID"];
                       $IBLOCK_ID = $arPriceElement["IBLOCK_ID"];
                       $OFFERS_IBLOCK_ID = false;
                       $OFFERS_PROPERTY_ID = false;
                   }
               }
           }
       } //Check for iblock event
       elseif (is_array($arg1) && $arg1["ID"] > 0 && $arg1["IBLOCK_ID"] > 0) {
           //Check if iblock has offers
           $arOffers = CIBlockPriceTools::GetOffersIBlock($arg1["IBLOCK_ID"]);
           if (is_array($arOffers)) {
               $ELEMENT_ID = $arg1["ID"];
               $IBLOCK_ID = $arg1["IBLOCK_ID"];
               $OFFERS_IBLOCK_ID = $arOffers["OFFERS_IBLOCK_ID"];
               $OFFERS_PROPERTY_ID = $arOffers["OFFERS_PROPERTY_ID"];
           }
       }

       if ($ELEMENT_ID) {
           static $arPropCache = array();
           if (!array_key_exists($IBLOCK_ID, $arPropCache)) {
               //Check for MINIMAL_PRICE property
               $rsProperty = CIBlockProperty::GetByID("MINIMUM_PRICE", $IBLOCK_ID);
               $arProperty = $rsProperty->Fetch();
               if ($arProperty)
                   $arPropCache[$IBLOCK_ID] = $arProperty["ID"];
               else
                   $arPropCache[$IBLOCK_ID] = false;
           }

           if ($arPropCache[$IBLOCK_ID]) {
               //Compose elements filter
               $arProductID = array($ELEMENT_ID);
               if ($OFFERS_IBLOCK_ID) {
                   $rsOffers = CIBlockElement::GetList(
                       array(),
                       array(
                           "IBLOCK_ID" => $OFFERS_IBLOCK_ID,
                           "PROPERTY_" . $OFFERS_PROPERTY_ID => $ELEMENT_ID,
                           "ACTIVE_DATE" => "Y",
                           "ACTIVE" => "Y",
                           "<CATALOG_QUANTITY" => AVALIABLE_VAL,
                           ">CATALOG_QUANTITY" => 0
                       ),
                       false,
                       false,
                       array("ID")
                   );
                   while ($arOffer = $rsOffers->Fetch())
                       $arProductID[] = $arOffer["ID"];
               }

               $minPrice = false;
               $maxPrice = false;
               //Get prices
               $rsPrices = CPrice::GetList(
                   array(),
                   array(
                       "BASE" => "Y",
                       "PRODUCT_ID" => $arProductID,
                   )
               );
               while ($arPrice = $rsPrices->Fetch()) {
                   $PRICE = $arPrice["PRICE"];

                   if ($minPrice === false || $minPrice > $PRICE)
                       $minPrice = $PRICE;

                   if ($maxPrice === false || $maxPrice < $PRICE)
                       $maxPrice = $PRICE;
               }

               //Save found minimal price into property
               if ($minPrice !== false) {
                   CIBlockElement::SetPropertyValuesEx(
                       $ELEMENT_ID,
                       $IBLOCK_ID,
                       array(
                           "MINIMUM_PRICE" => $minPrice,
                           "MAXIMUM_PRICE" => $maxPrice,
                       )
                   );
               }
           }
       }
   }

Before mail sent
----------------

.. code-block:: php

   <?php

   AddEventHandler('sale', 'OnOrderNewSendEmail', 'processOrderMailTemplate');

   function processOrderMailTemplate($ID, &$eventName, &$arFields)
   {
       if ($eventName == 'SALE_NEW_ORDER') {

       }
   }

Restrict 1C from updating element fields
----------------------------------------

.. code-block:: php

   AddEventHandler(
      'sale',
      'OnBeforeIBlockElementUpdate',
      array('OnBeforeIBlockElementUpdate', 'restrictFieldsUpdateFrom1C')
   );

   class OnBeforeIBlockElementUpdate
   {
       public static $group1cId = 23;

       public static function restrictFieldsUpdateFrom1C(&$arFields)
       {
           global $USER;
           if(in_array(self::$group1cId, $USER->GetUserGroupArray()))
           {
               unset($arFields["PREVIEW_TEXT"]);
               unset($arFields["DETAIL_TEXT"]);
               unset($arFields["CODE"]);
           }
       }
   }
