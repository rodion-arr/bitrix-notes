.. index::
   single: Recipes for SEO tasks

Recipes for SEO tasks
=====================

This section contains scripts for common SEO tasks

301 Redirect from old URL
-------------------------

Save current URL into property

.. code-block:: php

    <?php
    CModule::IncludeModule('iblock');
    $arSelect = array('ID', 'NAME', 'CODE', 'DETAIL_PAGE_URL', 'ACTIVE');
    $arFilter = array('IBLOCK_ID' => 4);
    $res = CIBlockElement::GetList(
        array('TIMESTAMP_X' => 'ASC'),
        $arFilter,
        false,
        array('nPageSize' => 1000),
        $arSelect
    );
    while ($ob = $res->GetNextElement()) {
        $arFields = $ob->GetFields();
        CIBlockElement::SetPropertyValuesEx(
            $arFields['ID'],
            false,
            array('OLD_URL' => $arFields['DETAIL_PAGE_URL'])
        );
    }

Release redirect logic in ``init.php``

.. code-block:: php

    <?php
    AddEventHandler('main', 'OnBeforeProlog', 'MyOnBeforePrologHandler');

    function MyOnBeforePrologHandler()
    {
        global $APPLICATION;
        $curPage = 'http://example.com' . $APPLICATION->GetCurPage();
        if ($newUrl = getNewUrlFromOld($curPage, 2)) {
            LocalRedirect($newUrl, false, '301 Moved permanently');
        }
        if ($newUrl = getNewUrlFromOld($curPage, 1)) {
            LocalRedirect($newUrl, false, '301 Moved permanently');
        }
    }

    /**
     * returns a new url from an old stored in IBLOCK element/section property 'OLD_URL'/'UF_OLD_URL'
     * or false if no such element/section in IBLOCK
     * @param string $oldUrl
     * @param integer $iBlockId
     * @param string $elementFieldName (optional)
     * @param string $sectionFieldName (optional)
     * @return mixed
     */
    function getNewUrlFromOld($oldUrl, $iBlockId, $elementFieldName = 'OLD_URL', $sectionFieldName = 'UF_OLD_URL')
    {
        if (CModule::IncludeModule('iblock')) {
            $elementList = CIBlockElement::GetList(
                array('SORT' => 'ASC'),
                array(
                    'IBLOCK_ID' => $iBlockId,
                    'PROPERTY_' . $elementFieldName => $oldUrl
                ),
                false,
                false,
                array(
                    'DETAIL_PAGE_URL'
                )
            );
            if ($arElement = $elementList->GetNext()) {

                return $arElement['DETAIL_PAGE_URL'];
            }
            $sectionList = CIBlockSection::GetList(
                array('SORT' => 'ASC'),
                array(
                    'IBLOCK_ID' => $iBlockId,
                    $sectionFieldName => $oldUrl
                ),
                false,
                array(
                    'SECTION_PAGE_URL'
                )
            );
            if ($arSection = $sectionList->GetNext()) {

                return $arSection['SECTION_PAGE_URL'];
            }
        }
        return false;
    }



Find and fix sections duplicates
--------------------------------

For all section codes duplicates concat section code of it's parent

.. code-block:: php

    <?php
    CModule::IncludeModule('iblock');

    $arFilter = array('IBLOCK_ID' => 4);
    $by = 'ID';
    $order = 'ASC';
    $db_list = CIBlockSection::GetList(array($by => $order), $arFilter, false);
    while ($ar_result = $db_list->GetNext()) {
        $arSect[] = $ar_result;
        $arCode[] = $ar_result['CODE'];
    }

    $arCodes = array_count_values($arCode);

    foreach ($arCodes as $key => $arCd) {
        if ($arCd > 1) {
            $arEndCode[] = $key;
        }
    }

    foreach ($arSect as $arSection) {
        if (in_array($arSection['CODE'], $arEndCode)) {
            $resa = CIBlockSection::GetByID($arSection['IBLOCK_SECTION_ID']);
            if ($ar_resf = $resa->GetNext()) {
                $name = $ar_resf['CODE'];
            }

            $code = $name . '-' . $arSection['CODE'];

            $bs = new CIBlockSection;

            $arFields = array(
                'CODE' => $code,
            );

            $res = $bs->Update($arSection['ID'], $arFields);
        }
    }

.htaccess SEO redirects
-----------------------

.. code-block:: bash

    #remove index.(php|html|htm)
    RewriteRule ^(.*)\/index\.(php|html?)$ /$1/ [R=301,NC,L]
    RewriteRule ^index\.(php|html?)$ / [R=301,NC,L]

    #www. to no www
    RewriteCond %{SERVER_PORT}s ^(443(s)|[0-9]+s)$
    RewriteRule ^(.*)$ - [env=askapache:%2]
    RewriteCond %{HTTP_HOST} ^www\.(.*)$ [NC]
    RewriteRule ^(.*)$ http%{ENV:askapache}://%1/$1 [R=301,L]

    #add trailing slash
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_URI} !(.*)(?:\/|\.html?|\.php)$
    RewriteRule ^(.*)$ %{REQUEST_URI}/ [R=301,L]

Fix pictures URL changing
-------------------------

.. code-block:: php

    <?php

    AddEventHandler('iblock', 'OnBeforeIBlockElementUpdate', 'checkImagesSize');

    /**
     * Restrict image update if same image size given
     * @param $arFields
     */
    function checkImagesSize(&$arFields)
    {
        $arImageInfoDetail = \CFile::MakeFileArray($arFields['DETAIL_PICTURE']['old_file']);
        if ($arFields['DETAIL_PICTURE']['size'] == $arImageInfoDetail['size']) {
            unset($arFields['DETAIL_PICTURE']);
        }

        $arImageInfoPreview = \CFile::MakeFileArray($arFields['PREVIEW_PICTURE']['old_file']);
        if ($arFields['PREVIEW_PICTURE']['size'] == $arImageInfoPreview['size']) {
            unset($arFields['PREVIEW_PICTURE']);
        }
    }

Change elements code separator from '_' to '-'
----------------------------------------------

.. code-block:: php

    <?php
    CModule::IncludeModule('iblock');
    $arSelect = array('ID', 'NAME', 'CODE');
    $arFilter = array('IBLOCK_ID' => 10, 'ACTIVE' => 'Y');
    $res = CIBlockElement::GetList(
        array('TIMESTAMP_X' => 'ASC'),
        $arFilter,
        false,
        array('nPageSize' => 1000),
        $arSelect
    );
    while ($ob = $res->GetNextElement()) {
        $arFields = $ob->GetFields();
        $newCode = str_replace('_', '-', $arFields['CODE']);

        $el = new CIBlockElement;

        $arLoadProductArray = array(
            'MODIFIED_BY' => $USER->GetID(),
            'CODE' => $newCode,
        );

        $res2 = $el->Update($arFields['ID'], $arLoadProductArray);
    }

-------------------------

Change sections code separator from '_' to '-'
----------------------------------------------

.. code-block:: php

   <?php
   CModule::IncludeModule("iblock");
   $arFilter = array("IBLOCK_ID" => 11);
   $db_list = CIBlockSection::GetList(array(), $arFilter, true);
   while ($ar_result = $db_list->GetNext()) {
       $newCode = str_replace("_", "-", $ar_result["CODE"]);

       $bs = new CIBlockSection;
       $arFields = array(
           "CODE" => $newCode,
       );

       $res = $bs->Update($ar_result["ID"], $arFields);
   }
